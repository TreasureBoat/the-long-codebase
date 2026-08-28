---
layout: post
title: "The heap floor lies"
subtitle: "A memory leak that hid for fourteen months, two wrong suspects, and why the standard instrument kept telling us the opposite of the truth"
date: 2026-08-28 20:00:00 +0900
author: ken
description: "A modernization sweep replaced two synchronized weak maps with ConcurrentHashMap. Strictly better on every axis anyone had written down, except the one that mattered. It took fourteen months and a bot to surface."
excerpt: >-
  Four applications, heap climbing about 195 MB a day, straight line, until the cap. The cause
  was two lines changed fourteen months earlier in a cleanup that was correct on every axis
  anyone had written down.
cta_hook: "If you have a long-running JVM and no idea whether it is healthy, the measurement half of this is the part worth stealing."
---

Four applications. Heap climbing about 195 MB a day, in a straight line, until it hit the cap.
No exception, no stack trace, nothing in the logs that pointed anywhere.

The cause was two lines, changed fourteen months earlier, in a cleanup that was correct on
every axis anyone had written down.

## Why it took fourteen months

The maps involved are original code and had been fine for years. What changed was a sweep in
June 2025 that replaced this:

```java
Collections.synchronizedMap(new WeakHashMap<>())
```

with this:

```java
new ConcurrentHashMap<>()
```

If you are reviewing that in a pull request it reads as a strict improvement. Thread-safe
without a wrapper. Faster under contention. Modern. There is no axis on which
`ConcurrentHashMap` is the worse choice — except that the old map had **weak keys**, and that
was load-bearing, and nobody had written down why.

Even then it stayed invisible, because you need a specific set of conditions to see it: an
internet-facing application, left running more than a week, taking real traffic. Our internal
apps are login-only and never announced, so they barely showed it. The first place it became
obvious was a public homepage that bots crawl. It needed a bot to find a bug that had been
there since the previous summer.

That is the honest answer to "how did this survive fifteen years of the framework". It didn't
survive fifteen years. It survived fourteen months, and it needed the right kind of host to
become visible at all.

## Two wrong suspects

Before any of that was clear, we were wrong twice, and both are worth stating because the
reasoning was plausible each time.

**First suspect: a `finalize()` removal.** We had recently replaced finalizers with `Cleaner`
as part of the JEP 421 work. Timing fits, memory-related, obvious candidate. Both of us
believed it.

It is not the cause, and the reason it cannot be is the useful part: the objects being
retained never become unreachable in the first place. A finalizer would never have run on
them. Neither would a `Cleaner`. If something is still strongly referenced from a static
field, nothing in that family of mechanisms is ever going to be asked about it. Once you say
that out loud the suspect eliminates itself — but we spent time there first.

**Second suspect: the notification center.** Claude proposed it, I found it plausible, and it
was wrong too. It holds its observers weakly throughout — `WeakHashMap`, weak arrays,
`WeakReference`. It retains nothing. Reading the code carefully killed the theory in about ten
minutes, which is ten minutes better than the first suspect managed.

## The red herring that cost the most

The expensive mistake was not a wrong suspect. It was a correlation.

Two versions, one apparently clean and one apparently leaking. That is a strong signal, and it
sent us looking at everything that changed between them.

The clean one was not clean. An application that had been running on it for thirty-nine days
was holding **18,995 editing contexts**. The earlier observations had simply been too short to
show anything, and I had looked at logs from those runs, seen no memory errors, and read that
as evidence of health.

It isn't. A short log with no errors tells you the failure has not had time to happen yet.
Ask what a system has *exercised*, not how long it has been *up*. A confirming instance of
that, found later: an old application on pre-sweep code held 249 editing contexts after **423
days** — five hundred times lower a rate than the post-sweep app we were staring at. That was
the control we needed and had not thought to look for.

## There is no tool for the actual question

`jmap` will tell you there are 3,224 instances of a class. It will not tell you who is holding
them, and that is the only question that matters. The JDK ships nothing that answers it.

So Claude wrote one: about 150 lines that parse an HPROF dump — strings, class loads, class
dumps, instance dumps, object arrays, GC roots, statics — and then walk *inbound* references,
level by level, from a suspect object toward whatever is keeping it alive.

That is the part of this I would not have done. Not could not — would not. It is an afternoon
of fiddly binary-format work to answer one question, on a Saturday, with three applications
misbehaving. I would have kept guessing, and my guesses were already 0 for 2.

Two traps in it produced confidently wrong answers first, and both would catch anyone:

**`referent` looks like a normal reference.** In a heap dump, `java.lang.ref.Reference.referent`
is just a field. Walk it and every weak map in the system looks like a leak, because you are
following exactly the edge that weakness is defined by. Filter it, or your tool reports the
opposite of the truth.

**Cycles look like roots.** A login component whose chain refers to a set that refers back to
the component is self-referential, and a naive walker stops there and declares it the culprit.
The garbage collector collects cycles perfectly well. You have to keep walking until you reach
a static field or a real GC root.

With those fixed it produced a chain in one pass:

```
TBWResponseRewriter._ajaxPageUserInfos   (static, 3,571 entries)
  → key = TBKeenLogin × 3,407
      → .editingContext
          → the entire registered enterprise-object graph
```

A static map, keyed by page component, that nothing ever removes from. Every page that emitted
a script or a stylesheet tag was retained for the life of the process, and each one dragged its
editing context and every object registered in it along behind.

## The second one

The first fix shipped and the numbers still did not fully add up. A second set of static maps,
from the same June sweep, keyed by editing context this time.

That one behaves differently in a way worth knowing: it accrues on a **timer, not on traffic**.
About fourteen entries an hour on two unrelated applications, whether or not anybody was using
them. So an idle internal application leaked at the same rate as a busy public one, which is
precisely the pattern that had made the earlier evidence so confusing. Two bugs, one origin,
and the two applications had each been dominated by a different one.

## The instrument was lying

Here is the part I would most want another team to take.

The standard way to watch for a leak is the **heap floor**: the minimum used heap in each
window. Garbage comes and goes, so the floor should track live data, and a floor that climbs
means a leak.

On the final day, after both fixes, with object counts proving all four applications were
completely clean, the floors read:

```
Boise      +1.35 MB/h
ESPGLAKE   +1.21 MB/h
ESPGCSCW   +0.39 MB/h
AppGWare   −1.08 MB/h
```

Three of them still "leaking", one apparently reclaiming memory from nowhere. All four
verifiably fine. Look at the actual shape of the hour-by-hour floor:

```
122 129 137 145 153 162 169 177 184 193 202 210 218 │ 124 130 138 146 154 161 169
                                                    ↑ a collection happened here
```

A sawtooth. Climb, collect, climb. On a heap with plenty of headroom the JVM has no reason to
collect often, so "the minimum this hour" is measuring **allocation between collections**, not
live data. Fit a straight line through that and you manufacture a slope with no physical
meaning — and its sign depends on nothing more than where your window happened to fall
relative to the last collection.

Four rules that survived the exercise:

1. **The floor means something only if a full GC happened inside the window.** Otherwise you
   are measuring garbage.
2. **Object counts are the instrument.** `jcmd <pid> GC.class_histogram` forces a full
   collection before counting, so what it returns is *live* objects. That is what turns "the
   heap is big" into "these specific things are being retained".
3. **Ignore the first two hours of any run.** Warmup overshoots on healthy and sick builds
   alike. A known-healthy run read +59 MB/h at 86 minutes.
4. **Watch for returns to baseline.** A leak floor never comes back down. One application
   returning to ~124 MB twice was the tell that it was fine, well before the counts confirmed
   it.

## The proof

Because `GC.class_histogram` forces a collection first, these are live objects — things the
JVM tried to reclaim and could not.

| application | editing contexts before | after ~20h on the fix |
|---|---|---|
| internal, 39 days uptime | **13,152** | **4** |
| public, bot-hit, 36.8h | **1,384** | **5** |
| public homepage, 11.5h | **3,224** | **4** |

Not reduced. Gone. At the first application's old rate you would have expected about 283 by
that point; there were four. Undo stacks went from thousands to twelve. One application had
served roughly 2,700 transactions during the window, so this is under real traffic, not an idle
box flattering the result.

## What came out of it

Two things worth more than the fix.

**A heartbeat in the framework.** Every application now writes its own liveness and heap history
without cron jobs or JDK tools on the box. Its first act in production was to *disprove* a crash
report: the monitor declared an application dead during a slow boot, and an uptime of 145
seconds proved the JVM had never restarted.

**JDK on every box, not JRE.** One of the four applications had only a JRE, which meant no
`jcmd`, no `jmap`, and nothing to diagnose with. The entire diagnosis depended on a different
box happening to have the tools installed. That is not a margin I want to rely on again.

## The honest split

I have written before about
[four occasions in one day]({{ site.baseurl }}{% post_url 2026-08-27-six-bugs-that-never-said-a-word %})
where this tool told me something plainly and incorrectly. This was the other kind of day.

The wrong suspects were shared — one of them was mine first. The red herring was a
misreading of short logs, and I have made that mistake for thirty years without help. What the
machine did was write a heap-dump parser on a Saturday afternoon to answer one question, and
then notice that a sawtooth had been fitted with a straight line.

Neither of those is clever. Both are the kind of patient, tedious, exact work that I avoid when
three applications are misbehaving and I want an answer now. Left alone I would have kept
guessing, and I do not think I would have found this at all.

That is a narrower claim than the ones usually made for these tools. It is also the one I keep
finding evidence for.
