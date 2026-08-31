---
layout: post
title: "Nobody announced anything"
subtitle: "The last release of the framework everything I had was built on came out in 2008. The official end-of-life notice came in 2016. I announced a replacement in 2013"
date: 2026-09-01 09:00:00 +0900
author: ken
description: "A framework can be dead for years before anyone says so. Mine was, and rebuilding it meant shadowing a closed-source product class by class, with the live applications as the test suite."
excerpt: >-
  The last release came out in September 2008. The official end-of-life announcement came in
  May 2016. In between, everybody just kept going — and the interesting question is what you
  do when the thing holding up your work has stopped moving and nobody will say it out loud.
cta_hook: "If you are running on something the vendor has quietly stopped maintaining, the shape of that decision is work we take on."
---

The last release of the framework I had built my working life on came out on 15 September 2008.

Nobody said anything. There was no announcement, no deprecation notice, no migration guide. The
release simply turned out to have been the last one, which is a thing you can only know
afterwards.

The official end-of-life statement arrived in **May 2016**. Eight years later.

I announced my replacement for it in **2013**, three years before the vendor admitted there was
anything to replace.

## How you find out without being told

At the developer conference in 2010, the vendor did not turn up.

That is a smaller signal than it sounds and a bigger one than it looks. Vendors miss conferences.
But this one had always come, and the people who came from it were the ones who answered the hard
questions, and that year there was nobody to ask. A room full of people noticing the same absence
at the same time is its own kind of announcement.

We also had a back channel. Some of us knew people inside the team running the single largest
deployment of the framework in the world — the vendor's own. What we heard was not ambiguous. It
was over. There would be no more releases, and no statement either.

So: a mature framework, a real community, production systems everywhere, and the certain
knowledge that it had stopped — with nothing official to point at. You cannot take "a friend
told me" to a client as a reason to spend their money.

## Three doors

Everyone I knew took one of three.

**Leave.** Pick something with a future, rewrite, move on. Entirely rational. What I watched
happen to a number of those applications is that the rewrite was estimated, found to be large,
deferred, and the application was eventually just switched off. For a lot of people the decision
to leave was, in practice, the decision to end.

**Freeze.** Keep running the last release forever. A community formed around exactly that and it
still exists — there are people today running the version from 2008. That is not stupid. The code
works. It does not stop working because a press release happened, or fails to.

But it is a slow door. Every year the world moves and you do not: the Java version you can run
on, the libraries you can use, the servers your host will still sell you. Eventually you are not
maintaining an application, you are maintaining a museum with users in it.

**Rebuild.** Which is the one nobody was taking.

## The talk

At the conference in 2013 I stood up and gave a one-hour talk about it.

Not a technical talk. The argument was: this is over, we all know it is over, and there are
enough of us in this room that if we build a replacement together it is achievable. The ideas are
good. The ideas are not the vendor's property in any sense that matters. What we lose if we do
nothing is not a product, it is everything all of us know how to do.

A few people were interested.

Most of the answers were one of two things. The first was **impossible** — too large, too much
for the people available, why would you try. The second I had not expected, and it came up more
than once: people were **frightened**. Not of the work. Of the vendor. Reimplementing a
commercial framework, even from the outside, even from behavior, felt to them like inviting a
letter from somebody's lawyers.

I do not think either group was being unreasonable. Reimplementing the foundation, persistence
and web layers of a mature framework, without the source, with almost no one, while still
shipping client work to pay for the time, is a genuinely unreasonable plan. If someone described it to me today I
would have questions.

But everyone advising me had an escape route I did not have. A bigger team. A different stack. An
employer who would absorb a rewrite. I had a one-person business, applications with real
customers, and twenty years of specific expertise that would become worthless the moment I walked
away from it.

## Why the name

If you rebuild something, you cannot keep its name.

Partly that is the legal caution everyone in the room was feeling. Mostly it is commercial: no
client wants to hear that the thing their system runs on is a version of a product that was
discontinued. An end-of-life name is a liability in every conversation you will ever have about
it, forever.

So — **TreasureBoat**.

The treasure is two things: the framework, and the knowledge. Those were the assets actually at
risk, and the second one mattered more than the first.

The boat is the community, all sitting in it together. That was the whole proposal of the 2013
talk.

And it is a reference. In Japan, where I have lived since I was eighteen, the Seven Lucky Gods —
七福神 — arrive on a treasure ship, the 宝船, bringing fortune with them. A ship that carries what
matters to a new place, which is exactly what I was proposing.

I will be honest that the boat ended up emptier than the name suggests. Most of the room did not
get in. A few did, and some of them are still here, and one of them turned up thirteen years
later with
[a prototype that changed my mind about something else entirely]({{ site.baseurl }}{% post_url 2026-08-20-160-countries %}).

## The closed box

The obvious plan is: reimplement the same ideas on a runtime that is still alive.

The obstacle is that the original was **not open source**. I could not read it. There was no
repository, no design document, no internals specification. There was documentation for people
*using* it, which tells you what a method is called and what it returns and almost nothing about
how the thing behaves when you push on it.

And the behavior is the entire product. A framework is not its API. It is ten thousand small
decisions underneath: what happens on a fault, what order things are notified in, which identity
comparison is used, exactly what happens to an object graph at save time. Get the API right and
the behavior wrong and every application built on top breaks in ways that look like their own
bugs.

So the job was not "write a framework". It was: work out how something behaves when nobody is
going to explain it to you, and then build a thing that behaves the same way.

## How you rebuild something you cannot read

Not alone, first of all. The hard part was two people — me and one other developer in Europe,
who runs his own business and would rather not be named. Everything below is as much his as
mine.

The method was not "write a replacement and switch over". It was to **shadow the original, one
class at a time, while the applications kept running on it.**

Phase one kept the original class names — internally, privately, never shipped — so that
existing applications continued to work untouched. Then we overrode the class loading order, so
that for one particular class our implementation was found before the original. Now you have a
live system where exactly one class is yours and everything else is still theirs. If it behaves,
you move to the next one. If it does not, the failure is unambiguous, because only one thing
changed.

That is the whole trick, and it solves the problem that makes this sort of work usually
impossible: **you do not need a specification, because you have a live system that already
behaves correctly, and real applications exercising it.** The applications were the test suite.
Not tests we wrote — a decade of accumulated production behavior, pushing on the framework in
ways no test author would have thought of.

Decompilation helped, but less than you would hope. The decompilers of the time were poor;
sometimes the output was unusable, and it was never something you could compile. What it gave
was *the idea* — roughly what a method was doing, which fields it touched, what order things
happened in. From there it was inference, and testing against real application code, and
implementing as we went.

I work bottom to top, so we started with the foundation layer — collections, dates, key-value
coding, the primitives everything else stands on — and then moved up the chain. That order is
not aesthetic. The lower layers are where the invisible behavior lives, and every mistake there
would have surfaced later as a mystery in a layer we had not written yet.

One unexpected advantage: the original was written for Java 4 to 6. We were writing on Java 8.
So we were not transcribing, we were re-expressing — the same behavior in the language as it
now was, rather than as it had been a decade earlier. That made the code shorter and clearer
than what it replaced, which mattered enormously later, because we had to keep reading it.

Phase one took about **eighteen months**.

## Phase two, which has no end date

Then the names changed, and the diet started.

Classes and methods renamed to ours. Unused code cut. Features added that the old framework had
never had — much of that owed to the work of the wider community around the original, which by
then had built a great deal on top of it.

And primitives replaced outright once we could see them clearly. The original's date and time
type was used absolutely everywhere, and its handling of time zones did not survive contact with
the modern world. That is not a thing you patch. It was replaced with new code, written
properly, and then every use of it was migrated.

Phase two is difficult to put a date on because it never really stopped — that is what the last
decade has been. But the thing I would most want understood about both phases is this:

**The applications never stopped running.** There was no cutover, no freeze, no big-bang
weekend. Real systems with real customers ran continuously on a framework that was being
replaced underneath them, a class at a time, for years. Sometimes an application needed
refactoring to keep up. That was never the hard part.

## What phase one actually was

I still have it. Seven layers, **663 classes, about 290,000 lines**, carrying the original names
so the applications would not notice.

The distribution is the part I would point at:

| layer | classes | lines |
|---|---|---|
| web | 265 | 69,131 |
| **foundation** | **206** | **137,699** |
| enterprise objects — control | 63 | 30,691 |
| enterprise objects — access | 58 | 38,337 |
| web services | 52 | 5,049 |
| database adaptor | 18 | 9,669 |
| xml | 1 | 7 |

The xml layer is one class and seven lines. The original had a layer there, so the shadow had
one too.

The foundation layer is not the biggest by class count and is nearly twice the biggest by
volume. That is the collections, the dates, the key-value coding — the things everything else
stands on, and the things with the most behavior per class that nobody ever wrote down. It is
also the layer we did first, when we knew least.

Two people. Eighteen months. Around 290,000 lines of somebody else's design, re-derived from the
outside.

Today, after a decade of phase two, it is a little under five thousand source files across
fifteen frameworks — most of that is not reimplementation any more, it is everything built since.

## Why this is the name of the blog

Most writing about software assumes the code is new, or that rewriting is normal when something
gets old. Both assumptions come from a world with more people and more time than mine has ever
had.

The code I work on is long. It has outlived the framework it was written against, the language
versions it was compiled for, the servers it ran on, and the company that made the tools. What I
have mostly learned is that leave-or-freeze is a false choice, and that the third door is harder
than it looks and much shorter than the rewrite.

That is what I mean by a long codebase. Not old. Long — still going, still being paid for, and
expected to still be here in another decade.
