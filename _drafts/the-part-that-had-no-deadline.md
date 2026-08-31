---
layout: post
title: "The part that had no deadline"
subtitle: "We converted 161 of 165 screens by hand and still failed, because finishing the conversion and finishing the work turned out to be different things"
author: paul
series: modernization
excerpt: >-
  Two experienced developers, a codebase one of us wrote and the other's framework runs, a
  plan good enough to survive its own failure — and it still ended in slop. Not because we
  were slow. Because only one of the competing jobs ever had a date on it.
cta_hook: "If you have a modernization that has been nearly done for a year, this post is about why."
#
# DRAFT — part 3 of the modernization series.
# Markers:  [[FACT]] number/name  ·  [[ASK]] answer becomes prose  ·  [[NOTE]] delete before publishing
#
# Post 1 already spent: the January-to-August framing (now corrected), 95 tables / 165
# components / 330 files, build-alongside strategy, June 2023 and September 2024 dates,
# the three dead quarters, 161 of 165, the trust boundary. Do not re-explain those. This
# post owes the HOW and the WHY.
---

Post one said we ran out of time. That is what I believed when I wrote it, and the commit
history says something more awkward: we did not run out of time on the conversion. We
finished the conversion. What we ran out of time on was everything that makes a conversion
safe, and none of that had a date attached to it.

## The plan was good

We did not modernize the application in place. We branched it.

Every one of the 165 screens got a new component built beside the old one — a new HTML
template, a new Java class — and the old component stayed exactly where it was. The new UI
went on top. Nothing was deleted.

That was deliberate, and it was the right call. It meant that at any point, right up to the
last week, we could have shipped the old application without a large change. A modernization
you can abandon safely is worth more than one that is faster and cannot be stopped.

I want to be clear about this because of where the post ends up: the plan was not the
problem. If I ran it again I would branch it the same way.

## Ken went first, and took the worst of it

Ken started in June 2023, on the observation components.

There are 45 of them, and they are the hardest part of the system. An observation comes in
three subtypes — classroom, non-classroom, and formal — which share a great deal of logic and
then diverge in UI behaviour, with formal diverging most. Fifteen years of that had produced
exactly the kind of object model you would expect: shared where somebody had noticed, copied
where nobody had.

He converted them. I reviewed his work, and later my own, and as far as I could tell it was
correct.

That last clause is carrying more weight than it looks. I did review. What I could not do
was review all of it, in the same way each time, and then do it again after the next change.
One person clicking through screens on a weeknight is sampling. It is not coverage, and it
is not repeatable — so whatever I confirmed in the autumn told me nothing about what was
still true in the spring.

There was also nobody to ask. No QA, and no subject matter expert other than me: one person
who could look at a converted formal observation and know whether it still did what a formal
observation is legally required to do in that district's state. I was that person, I was
reviewing my own work as well as his, and I was busy.

I cannot tell you what those fifteen months were like for him. I was not there for them, and
I have never asked.

That is not an omission in this post. It is the same gap as everything else in it. I arrived
fifteen months late, adopted his patterns, and did not ask why they were the way they were —
and I have still never asked whether he knew, at eleven at night in Tokyo, that the only
person who would check what he had just written was going to do it in gaps, from memory, and
not twice.

## The quarter where a district came and went

Between October 2023 and June 2024 the conversion stopped completely. Not one component.

Not because we had lost interest. A friend introduced a school district who wanted to look at
the platform. So we did what you do: we built to their state's requirements, and we took our
contacts' suggestions about how classroom transcripts should be captured, and we put both
into the product.

We did not win the business.

The code stayed. It is in there now, a state's compliance rules and a transcript workflow
built for a district that said no, sitting on top of an object model that was halfway through
being replaced. That is not a story about wasted effort — you build to the prospect in front
of you, and I would do it again. It is a story about what a modernization actually competes
with.

## Then me, fifteen months late

I joined in September 2024.

I did not start from the design. I started from Ken's components, and I followed the pattern
he had established, because that was obviously the fastest way in and because he had already
solved the hard problems.

The trouble with following a pattern is that you can reproduce it without understanding it.
There were technical decisions in Ken's work that I did not fully appreciate — not errors on
his part, things he had reasons for that I had not asked about. Where the pattern did not
quite fit what I was converting, the correct move was to stop and find out why he had done it
that way.

I did not do that. I coded around the gap and kept moving.

That sentence is the honest centre of this post. Every developer reading it knows exactly
what it means and roughly how often they have done it.

I went looking for how many hours this took and found something better, because git
remembers what nobody does.

Across three years there are forty-eight days on which either of us committed a line of this
work. Not forty-eight weeks of evenings. Forty-eight days, scattered over thirty-eight
months.

Ken never worked two days running. Not once — across his twenty-four active days the
shortest gap between any two of them is two days. The longest unbroken run by either of us
was three days, in September 2024, and it was mine.

We were also on opposite schedules and opposite sides of the world. Nineteen of Ken's forty
commits landed after eight in the evening in Tokyo, six of them between midnight and six,
and his heaviest day of the week was Monday. Not one of my twenty-nine landed after eight.
Mine are weekdays, spread from seven in the morning to eight at night, because I was not
keeping evenings for this. I was taking whatever gap appeared.

I had been describing this to myself as evenings and weekends. It was Ken's evenings, and my
leftovers.

## What that produced

By the August 2025 cutover we had converted 161 of the 165 screens, and the application had
three separate kinds of debt in it.

**Refactor slop.** Coding patterns that drifted between Ken's components and mine. CSS
inconsistencies. Places where I had worked around something rather than understood it.

**Requirements built mid-flight.** The SDE rules and the transcript changes, layered onto
patterns that were themselves still moving.

**UX changes with good intent and incomplete design.** These are the expensive ones. Slop can
be cleaned by a careful developer. A half-conceived interface cannot — it needs somebody to
decide what was actually meant, and nobody had time to make that decision.

Everything generally worked. That is the part I keep coming back to. A full development team
would have cleaned all of it up as a matter of course, and we did not have one, and it all
shipped.

## Why the cleanup lost

Here is the mechanism, and it is the same one that killed the selling.

A state regulation has a date on it. A district's compliance requirement has a date on it. A
bug a school hits on a Tuesday morning has a very sharp date on it. Cleaning up a coding
pattern has no date at all, and neither does verifying that a formal observation still behaves
correctly after conversion.

So the work with dates got done, every single time, and the work without dates did not. Not
once did anybody decide the cleanup was less important. It simply never won a scheduling
contest, because it never entered one.

The obvious answer is a test suite. I knew that in 2023, and I have known it for fifteen
years. But a regression suite for 165 screens carrying this much workflow is a job of
exactly the same shape as the cleanup — enormous, mechanical, and with no date on it. So it
lost the same contest, every time, for the same reason. The thing that would have let us
finish was itself a thing we could never finish.

That is the trap, and I was in it for two years.

## What this is a control group for

This is the part of the series I need you to hold on to, because everything after it is
measured against this post.

Two developers. One of them wrote the application; the other wrote and has maintained the
framework for twenty years. A plan good enough to survive its own abandonment. Two years of
calendar time. And the outcome was 161 of 165 screens converted, three kinds of debt, and a
deploy I could not bring myself to invoice for.

Nobody was slow. Nobody was careless. What we did not have was any way to know whether what
we had built was correct — and without that, cleanup is not work you can prioritise, because
you cannot tell when it is done.

There were not two roads. There was one, and it was closed.

The road you are supposed to take is more people. A tester who could tell me whether a
converted formal observation still behaved. A second developer. Somebody to finish the UX
decisions nobody had time to make. I have costed that more than once in fifteen years. Six
districts has never paid for it and was never going to — and the money was only half of it,
because even with the money I would not have handed this codebase to someone I did not know.

The other road I did not reject. I did not know it was there.

I found it somewhere else entirely — on somebody else's problem, in a technical domain I
knew nothing about, at a job I started in the summer of 2025. I have written about that
[here]({{ site.baseurl }}{% post_url 2026-08-27-the-typewriter-argument %}). Then I spent a
while convincing Ken, who had better reasons than most to say no, and whose account of why
he refused is [here]({{ site.baseurl }}{% post_url 2026-08-20-160-countries %}).

And then I came back to this codebase, which had been sitting there the whole time.

The next post is about the first thing I did when I got back, which was not writing any new
code at all.
