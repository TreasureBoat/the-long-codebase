---
layout: post
title: "Never bump one pin alone"
subtitle: "A one-line version change took production down, and the second time it happened the crash sat armed for two days without firing"
date: 2026-08-28 22:30:00 +0900
author: ken
description: "Bumping a single framework version in an application POM compiled cleanly and killed the app at boot. The same field killed a second app two days later — except that one had no users, so nothing happened."
excerpt: >-
  One framework version bumped from 21.0.3 to 21.0.4 to ship a one-line NPE fix. It compiled
  cleanly and the application died at startup. Two days later the same field was armed on a
  second app for two days without firing, because nobody logged in.
cta_hook: "If you maintain a set of frameworks that reference each other, the constant-pool check at the bottom is the part worth stealing."
---

I bumped one version number in one file to ship a one-line fix. It compiled cleanly. The
application died at boot and production was down until I rolled it back.

```
NoSuchFieldError: Class ETBFFrameworks does not have member field 'ETBFFrameworks tb_pro_keen'
    at TBBaseModelBridge.addToDashboard(TBBaseModelBridge.java:58)
```

Two days later the identical field was sitting armed on a second application, and it had been
for two days, and nothing had happened. That second part is the more interesting one, so I will
come back to it.

## What I changed

We ship about fifteen frameworks. Applications pin the versions they want in their own POM:

```xml
<tb.fw.core>21.0.4</tb.fw.core>
<tb.fw.pro>21.0.3</tb.fw.pro>
<tb.fw.sangria>21.0.3</tb.fw.sangria>
...
```

There was a one-line null-pointer fix in the core framework, so I moved `tb.fw.core` from
21.0.3 to 21.0.4 in the two applications that needed it. That is exactly the change you would
make. It is also the change I would review and approve without much thought.

One of the two applications was fine. The other had missed the previous coordinated release and
was quietly behind on **six** frameworks. Moving core forward on its own left it holding one
current framework and six old ones — and our frameworks reference each other's internals.

Specifically: core 21.0.4 removed a constant, `ETBFFrameworks.tb_pro_keen`, as part of an
unrelated refactor. The base-model bridge at 21.0.3 still read that constant. A field that no
longer exists is not a compile error in a prebuilt jar; it is a `NoSuchFieldError` the first
time that class initializes. Which is during startup.

## Two things made this worse than it needed to be

**Applications do not inherit the framework pins.** Ours parent off an application POM, not the
framework POM, so each application carries its own copy of the whole pin table. That is a
deliberate choice — an application should decide when it moves — but the consequence is that
there is no single place where the set is kept consistent. Nothing stops one pin drifting
forward while the rest stay behind, because as far as the build is concerned they are just
independent properties.

**The fix release carried an unrelated breaking change.** I cut 21.0.4 for a one-line NPE fix.
It also shipped a refactor that had been sitting unreleased on the main branch for four days,
and that refactor is what removed the constant. Nobody lied; I just did not read
`git log <lastTag>..HEAD` before cutting the release, so the release notes said "NPE fix" and
were true and incomplete.

## Why compiling proved nothing

This is the part I would most want to pass on.

`mvn compile` succeeded. Of course it did — the code being compiled is the application's, and
the application never mentions the missing constant. The reference lives inside a *prebuilt jar*
that was compiled a month earlier against a version of core where the field still existed. The
jar's constant pool holds a symbolic reference that is resolved lazily, at class initialization,
at runtime.

So the successful build was not weak evidence of health. It was **no evidence at all**, and it
felt like evidence, which is worse.

What actually answers the question is looking in the jar:

```sh
javap -v -p -cp <extracted> org.treasureboat.TBBaseModelBridge \
  | grep -oE "ETBFFrameworks\.[a-zA-Z_]+"
```

That reads the constant pool and lists the fields the class will demand at runtime, whether or
not any test happens to reach them. Pair it with `mvn dependency:list` to see which version
actually resolved after mediation — which is not always the one you pinned.

## The second one, which did not go off

Two days later, aligning a third application, the same constant turned up again — this time
reached through a different framework that had not yet been repointed.

Four classes still read it. In one of them the instruction that fetches the missing field is
**instruction 0** of the method: the very first thing it does. There is no branch that could
skip it, no null check in front of it, no configuration that avoids it. Any call to that method
throws, without exception.

It had been deployed for two days.

Nothing had happened, and the reason nothing had happened is that the method is on the login
path, and nobody had logged in. The login *page* had rendered thousands of times — bots, mostly
— but rendering the page and completing a login are different code. The application looked
perfectly healthy. The logs were clean.

Zero errors in the log was not evidence that the code was correct. It was evidence that nobody
had run it.

I have written the same sentence about a
[memory leak that took fourteen months to become visible]({{ site.baseurl }}{% post_url 2026-08-28-the-heap-floor-lies %}),
where a version we believed was clean turned out never to have been observed long enough to
show the problem. It is the same mistake in a different costume, and I have now made it twice
in one week: **ask what a system has exercised, not how long it has been up.**

## What I do now

**Diff the whole table before touching one line of it.** Pick an application already running
the target version, print both pin tables, and move every laggard together. The bad version of
this task is deciding which pins "should" matter; the good version is making the two tables
identical and not thinking about it.

**Do not reason about whether a path is reachable — scan the constant pool.** I spent time on
that second one working out whether the login path could be hit. That was wasted effort with a
wrong answer available at the end of it. `javap` tells you what the class demands. Reachability
arguments tell you what you hope.

**Keep the rollback close.** The previous deployment bundle is still on disk. Repointing to it
and restarting took under a minute, and I reached for it later than I should have because I was
still trying to understand the failure. Understand it afterwards.

**Read `git log <lastTag>..HEAD` before cutting a release.** If something unrelated is riding
along, either hold it or say so in the notes. A version number that means "one-line NPE fix" to
its author and "removes a public constant" to its consumer is not a version number, it is a
trap with a changelog attached.

None of this is clever. All of it is the sort of thing you write down after the second time.
