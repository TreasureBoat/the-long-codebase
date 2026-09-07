---
layout: post
title: "Forty-eight hours"
subtitle: "A product manager who does not write framework code started an IDE plugin for my framework. I made my first commit to it two days later, and I have not stopped since"
date: 2026-09-08 09:00:00 +0900
author: ken
description: "Paul started a plugin for a framework he does not maintain, in a plugin API he had never used. Two days later I joined. Nine months on it is 476 files and 122,000 lines, and the thing that lasted was not the code."
excerpt: >-
  Paul's first commit is dated 29 November. Mine is dated 1 December. I had spent a month
  ignoring everything he said about AI, and it took forty-eight hours from seeing the thing to
  committing to it.
cta_hook: "Both halves of this were written by the people who did them. If you want the same on your own stack, that is the work we take on."
---

[[NOTE  Joint post. Paul writes the first section, I write the rest. Front matter only takes one
        author key, so the byline currently says Ken — either add a coauthor field to the layout
        or make it explicit in the text. DECIDE which. ]]

## Paul

[[ASK  PAUL — this is your section, and it is the half I cannot write.

       What I know from the outside: your first commit is 2025-11-29, "Initial commit from
       Specify template". You wrote 136 commits over three weeks and stopped around 21 December.
       You had never written an IntelliJ plugin. You do not maintain the framework it is for.

       What would be worth having:
        - why a plugin, rather than continuing to send Ken prompts
        - the spec-first method — was that deliberate from day one, or did you arrive at it?
        - where you got properly stuck, and what you did about it
        - what you could not judge, being outside the framework
        - why you stopped in December ]]

## Ken

Paul's first commit is dated 29 November. Mine is dated 1 December.

I want to sit on that for a second, because it is the most surprising fact in this whole story
to me, and I am the one it happened to. I had spent about a month not answering him about AI. I
had quit the previous assistant outright after it
[handed me a list of 160 countries and told me it was all of them]({{ site.baseurl }}{% post_url 2026-08-20-160-countries %}).
And then I looked at a half-working prototype and was committing to it inside forty-eight hours.

## What the prototype actually was

Not much, honestly. It opened. Bugs everywhere, no detail finished, nothing you would ship.

But it was a plugin for *my* framework, doing the thing I had wanted for years and never had a
spare fortnight for — getting off an IDE I had been complaining about for a decade. And it had
been built by someone who does not write framework code, in a plugin API he had never touched,
in a couple of weeks, in the evenings.

The argument I had been ignoring for a month was "this tool is good". The thing that landed was
a working artifact that existed and should not have.

## Not a handover

I described this to someone recently as Paul building a prototype and handing it over, and then
I looked at the history and found that is not what happened at all.

December 2025:

| | commits |
|---|---|
| Ken | 207 |
| Paul | 129 |

Three weeks, side by side, both of us in the same repository. He was not handing me anything —
he was still going, on his features, while I started on mine. He tapered off around the 21st and
I have carried it alone since.

That distinction matters because it is the difference between "someone gave me a head start" and
"we worked on it together and then I kept going". The second is what happened, and it is a
better description of what these tools make possible for two people who are not in the same
company, the same country, or the same time zone.

## The thing that lasted was not the code

Almost none of Paul's original code survives. That is not a criticism — it was a prototype, and
prototypes are meant to be replaced. Nine months on, the plugin is 476 Java files, about 122,000
lines, 544 tests, and 28 separate feature areas: model editors, rule editors, component editors,
a deploy tool, a localization editor, code generation.

What survived is his **method**, and I did not notice I had adopted it until I went looking at
the history for this post.

His very first commit is called "Initial commit from Specify template". His commits then run in a
pattern, over and over:

```
Add specification for D2W Rule Generation from EOModel Popup
Clarify Docker spec: Gradle build, Azul Zulu base image
Add implementation plan for Docker Deployment Support
Add implementation tasks for Docker Deployment Support
Implement Docker deployment support with property file workarounds
```

Specification, then clarification, then plan, then tasks, then code. He did not become a
developer for two weeks. He used the thing he is actually expert at — turning a vague want into
a written, agreed, decomposed statement of what should exist — and let that drive the
implementation.

There are now 207 spec files in that repository. Most of them are mine. I am still working the
way a product manager set up in his first fortnight, on a codebase he has not touched since
December.

If you want one transferable idea out of this post, that is it. The prototype was the thing that
convinced me. The method is the thing that was worth having.

## What nine months of it looks like

Not a straight line, and I want to record the two least glamorous parts.

**The plugin was rejected by the marketplace.** Not for quality — for using internal platform
APIs, the kind that work today and break on some future IDE release. Perfectly fair, and it took
a cleanup pass across a dozen usages to clear. The submission after that has now been sitting in
the review queue for weeks, which is its own lesson about shipping into somebody else's process.

**The test suite was dead for two months and reported nothing.** A build tool upgrade quietly
dropped an implicit dependency, and without it the test task failed *before running a single
test* — while looking like infrastructure noise rather than a failure. Nothing was red. Nothing
was green either; there was simply nothing, and I did not notice, because a build that says
nothing looks like a build that passed.

I have written about
[a memory leak that hid for fourteen months]({{ site.baseurl }}{% post_url 2026-08-28-the-heap-floor-lies %})
and about
[a version bump that took production down while compiling cleanly]({{ site.baseurl }}{% post_url 2026-08-28-never-bump-one-pin-alone %}).
This is the same shape a third time. If a test task ever looks *broken* rather than *failing*,
check that first — and treat two months of silence as the alarming thing it is.

## Where it is now

Version 1.2.7 shipped yesterday, fixing three small things, one of which was an empty file
producing a stack trace instead of an editor.

The framework it supports is one I rebuilt by hand because its original was discontinued. The
plugin that now edits its models was started by someone who does not write framework code, using
a tool I had refused to use, and I joined two days later.

I would not have predicted any part of that sentence a year ago.

[[ASK  KEN — worth deciding:
        - is naming the marketplace and the review wait fine, or too pointed?
        - do you want the 122,000-line figure in? It is accurate but invites "lines of code" arguments
        - should we link the plugin itself, or is it not public enough yet? ]]
