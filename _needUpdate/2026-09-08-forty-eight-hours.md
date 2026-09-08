---
layout: post
title: "Forty-eight hours"
subtitle: "A product manager who does not write framework code started an IDE plugin for my framework. I made my first commit to it two days later, and I have not stopped since"
date: 2026-09-08 09:00:00 +0900
author: ken
coauthor: paul
description: "Paul started a plugin for a framework he does not maintain, in a plugin API he had never used. Two days later I joined. Nine months on it is 571 files and 135,000 lines, and the thing that lasted was not the code."
excerpt: >-
  Paul's first commit is dated 29 November. Mine is dated 1 December. I had spent a month
  ignoring everything he said about AI, and it took forty-eight hours from seeing the thing to
  committing to it.
cta_hook: "Both halves of this were written by the people who did them. If you want the same on your own stack, that is the work we take on."
---

## Paul

I had spent about a month telling Ken what I was getting out of Claude, without much progress,
and I want to be fair about why. He was not stonewalling. He had used Copilot, it had handed him
a plausible and wrong answer, and he had drawn a reasonable conclusion from his own evidence. He
tells that story below. It was not an argument I was going to win by making it again more loudly.

What broke it was not my persistence. It was that he challenged me to build the first module of
an IntelliJ plugin for his framework.

That is a good challenge, and I want to give him credit for it rather than take it. It put the
claim somewhere it could be checked, on ground he owned and I did not, at a size where failure
would have been obvious and quick. If I could not do it, there would have been nothing more to
discuss.

It took me that weekend to get it started. Ken's half opens with the forty-eight hours; here is
what was inside them: seven commits, from 08:21 on Saturday 29 November to 16:31 on the Sunday.
He committed on the Monday.

That weekend was the initial prototype, and it is only the first part. I kept going through
December, prototyping other components — 129 commits by the time I stopped on the 21st. The
weekend answered his challenge. December is what actually convinced him, and the measure of that
is not anything I said: he went from the free plan to Pro to Max inside those few weeks.

We were not building an IntelliJ plugin. We were **porting** one. There was already an Eclipse
plugin for the framework — 1,428 Java files, about 192,000 lines — and it worked, and people used
it. Which means the specification for the IntelliJ version already existed. It was just encoded
as behavior in a codebase rather than written down anywhere.

So the first thing I asked for was not code. It was a review of the Eclipse plugin, and an
extraction of the specifications for the IntelliJ equivalent from it.

Concretely, the first file in that repository that does any work is
`.claude/agents/eclipse-plugin.md`. Eighty-three lines. Its tool access is `Read, Grep, Glob,
WebSearch, WebFetch` — nothing that can write. Its entire job is to know the Eclipse codebase and
answer questions about it. It went in on 29 November at 19:45, in the same commit as the first
line of implementation, and it is still there today.

That is why someone who does not maintain the framework could produce something credible for it.
I did not need to understand EOModels — the database-to-object mappings TreasureBoat runs on — or
its D2W rules, which decide what a screen shows without anyone writing the screen, well enough to
specify tooling for them. The Eclipse plugin already understood them, in detail, correctly, and
reading 192,000 lines to extract what a replacement would have to do is precisely the tedious
cross-referencing I would have given up on by the second afternoon.

What I contributed was not the specification. It was noticing that the task everyone would have
described as "build a plugin" was actually "port a plugin". **A working system is a specification
nobody has bothered to write down, so before you specify the replacement, spend the first day
extracting the spec out of the thing you are replacing.** That is the part that transfers, and it
has nothing to do with this framework or this IDE.

**Where it ran out.** I could get the rough outline of a feature working. I could not get the
details of a subsystem right, and those are different problems.

Extraction gave me the shape of what an EOModel editor has to do — the screens, the operations,
the artifacts it reads and writes. It did not give me the requirements underneath: what every
field means, which combinations are valid, what the framework does with them at runtime. Those
are not recoverable by reading. They are known by people who have lived in the thing.

I had maintained some of the Eclipse plugin code myself over the years, but that was patching.
Fixing a defect in a codebase that size teaches you the few hundred lines around the defect. It
does not teach you the system, and I never had reason to learn the system, because I was a
consumer of it rather than its maintainer.

So when Ken says the prototype had bugs everywhere and no detail finished, that was the ceiling,
not carelessness. On 30 December he had to make complex rule expressions read-only in a class I
wrote, because the editor was loading them, letting you edit them, and corrupting them on save.
Nine days later he fixed my plist writer, which had been emitting model files with unquoted keys,
and my model save, which was dropping a field. All three of those opened. All three looked like
they worked.

That is the distinction I could not cross: the extraction gave me code that runs, and running is
not the same as right. I could tell you the editor opened. I could not tell you it was correct,
because I had no way to check the answers.

**What the prototype was for.** Ken says below that almost none of my original code survives, and
I have no argument with that as a description of what a prototype is for. It was never meant to
last. It was meant to answer one question — is this road passable — for one person who did not
believe it was.

That is the part I would defend, and it is not a coding achievement. Changing the mind of a
competent skeptic who had arrived at his position from evidence is hard, and arguing does not do
it. A half-working thing in his own territory did, in two days. Whether the lines survived nine
months is beside the point; they had already done their job by 1 December.

**Why I stopped.** My day job took the evenings back, which is the boring half. The other half is
that by then it did not matter. Ken had made more commits in December than I had, on his own
features, in the parts I could not have finished anyway. The thing I was worried about in
November — that this would be a demo he politely ignored — was long over.

[[NOTE  PAUL — (3) does not match my memory, and it goes the other way. I did not challenge you.
        You decided to build a prototype and you decided to start from the Eclipse plugin, and I
        did not know you were doing either until I saw it. So the subtitle stands.

        Take the credit rather than hand it to me. It was the first hard work on this and it was
        the harder road: reading a plugin you did not maintain, to work out what a replacement
        would have to do, before writing a line. The only word I would change is "challenge",
        because I never issued one. My half now says what the month before was actually like,
        which I think was the part missing from both our sections.

        Your other two notes are done rather than pending: the specs correction is in "The thing
        that lasted was not the code" — you extracted them from the Eclipse plugin and the idea
        you contributed was noticing the old plugin WAS the specification — and the free/Pro/Max
        line stays, with the timing added on my side, because it is the best evidence in the post
        and it is mine to give.

        One thing I have added rather than changed: my half now says the Eclipse plugin was the
        kickstart and not the driver. Your section is about the fortnight when it WAS the driver,
        so nothing there needs touching — but nine months on, most of what the plugin does has no
        Eclipse counterpart at all, and the post read as though we were still porting.

        Two numbers in your section while you are in there: the Eclipse plugin is 1,384 Java files
        and about 187,000 lines, not 1,428 and 192,000. And December was 131 commits for you, 130
        of them by the 21st, not 129. ]]

## Ken

Paul's first commit is dated 29 November. Mine is dated 1 December.

I want to sit on that for a second, because it is the most surprising fact in this whole story
to me, and I am the one it happened to. I had spent about a month not answering him about AI. I
had quit the previous assistant outright after it
[handed me a list of 160 countries and told me it was all of them]({{ site.baseurl }}{% post_url 2026-08-20-160-countries %}).
And then I looked at a half-working prototype and was committing to it inside forty-eight hours.

## The month before

I was not ignoring him out of stubbornness. Two things were in the way.

The first was Copilot. I had used it, it had handed me something plausible and wrong, and I had
drawn the obvious conclusion from my own evidence.

The second is the part I have not seen anyone write down. Paul was not only telling me the tool
was good — he was sending me prompts. Twenty pages of them.

I read them. That is honestly all I did with them, because I could not see how to get from that
document to my keyboard. Writing twenty pages of English about code you have not built yet is a
skill, and it is not mine. I think in code and I write code; I have been doing that since 1988.
Turning something I can already see into prose for somebody else to act on is harder work for me
than building it.

So the prompts did not fail because they were wrong. They failed because they asked me to work in
the medium I am worst in, before I had any reason to believe the output would be worth it.

Then Paul stopped arguing and built something instead. That was his decision, not mine — I did
not ask him for a prototype, and starting from the Eclipse plugin would not have occurred to me.
I have maintained that plugin for years and I never once thought of it as a specification.

It was also the harder road. He did not start by writing code; he started by reading a plugin he
does not maintain, in a framework he does not work on, to find out what a replacement would have
to do — 1,384 Java files of it. That is the part I would not have done, and it is why what landed
on my screen was credible instead of a demo. The first hard work on this project is his.

## What the prototype actually was

Not much, honestly. It opened. Bugs everywhere, no detail finished, nothing you would ship. He
had also worked from an older copy of the Eclipse plugin, so some features were not missing by
accident — they were not in the version he was reading.

But it was a plugin for *my* framework, doing the thing I had wanted for years and never had a
spare fortnight for — getting off an IDE I had been complaining about for a decade. And it had
been built by someone who does not write framework code, in a plugin API he had never touched,
in a couple of weeks, in the evenings.

The argument I had been ignoring for a month was "this tool is good". The thing that landed was
a working artifact that existed and should not have.

Paul mentions above that I went from the free plan to Pro to Max in those few weeks. The timing
is the part worth having. Free lasted about three days — against work this heavy it did not last
five minutes in a sitting. Pro lasted maybe two weeks. I have been on Max since and I am happy
there.

A month before that I would not have spent anything on this at all. That is the measure of what
the prototype did, and it is a better one than anything I could say about it.

## Not a handover

I described this to someone recently as Paul building a prototype and handing it over, and then
I looked at the history and found that is not what happened at all.

December 2025:

| | commits |
|---|---|
| Ken | 207 |
| Paul | 131 |

Three weeks, side by side, both of us in the same repository. He was not handing me anything —
he was still going, on his features, while I started on mine. He tapered off around the 21st and
I have carried it alone since.

That distinction matters because it is the difference between "someone gave me a head start" and
"we worked on it together and then I kept going". The second is what happened, and it is a
better description of what these tools make possible for two people who are not in the same
company, the same country, or the same time zone.

## The thing that lasted was not the code

Almost none of Paul's original code survives. That is not a criticism — it was a prototype, and
prototypes are meant to be replaced. Nine months on, the plugin is 571 Java files, about 135,000
lines, 544 tests, and 28 separate feature areas: model editors, rule editors, component editors,
a deploy tool, a localization editor, code generation.

[[FACT  File count and lines are measured and current (571 files, 134,990 lines across all Java;
        504 files and 124,548 lines if you count only src/main — say which you mean). The 544
        tests I could not reproduce: I count 490 @Test methods. If 544 comes off a test report,
        parameterized cases would explain the gap — but the number needs a source. Same for the
        207 spec files: 224 files match *spec*, and specs/ holds 34, so it depends what you are
        counting. Worth noting 207 is also exactly your December commit count. ]]

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

Two things need saying about that, and I only understood the second one while writing this.

The first is Paul's own correction, and he is right to make it: he did not invent those
specifications. He extracted them from the Eclipse plugin. The idea he contributed was that the
old plugin *was* the specification, which is not a small thing to notice, but the content came
out of code I had been maintaining for years without ever seeing it that way.

I should be equally careful in the other direction, because the story so far makes the Eclipse
plugin sound like it drove all nine months. It did not. It was the kickstart, and I have barely
looked at it since.

Most of what the plugin does now has no counterpart over there. The localization editor, the face
ID editor and most of the twenty-eight areas did not exist in Eclipse at all — they are not ports
of anything. The entity modeler is the one that resembles its ancestor, and even it has a great
deal in it that never existed there, plus the whole Model Data Creator integration, which
generates model files from annotated Java classes and had nothing to port from.

So the extraction was exactly right for the question it answered — what would a replacement have
to do — and it answered it inside the first fortnight. After that, what was being built stopped
being a port.

The second is that I did not keep his method as he wrote it. I copied it exactly at first, long
prompts and all, and I hit the same wall as I had with the twenty pages: it asked me to write
down, in advance and in full, something I do not know in advance. I get ideas while looking at
something. I am visual — put a picture in front of me and everything connected to it arrives at
once. Ask me to produce that picture in prose before it exists and I stall.

So I stopped writing them. Now I work in short prompts. Do this one thing, then test it. Do the
next thing, then test it. It is easier for me to write and it catches problems while they are
still small — a step that goes wrong is a small thing to find, and a twenty-page step that goes
wrong is not. And the feedback coming back on screen is itself the picture: I read two or three
messages and the next idea is already there. That is how the work in this post got done, today
included.

What survived, then, is the shape — write it down, plan it, break it into tasks, do them one at a
time. What I dropped is the length. There are now 207 spec files in that repository and most of
them are mine, so the structure clearly stuck; it is the twenty-page prompt that did not.

If you want one transferable idea out of this post, that is it. The prototype was the thing that
convinced me. The method is the thing that was worth having — and it was worth having in a
smaller size than it arrived in.

## What nine months of it looks like

Not a straight line, and I want to record the two least glamorous parts.

**The plugin was rejected by the marketplace.** Not for quality — for using internal platform
APIs, the kind that work today and break on some future IDE release. Perfectly fair, and it took
a cleanup pass across a dozen usages to clear. The resubmission went through, and it has been
[live on the JetBrains Marketplace](https://plugins.jetbrains.com/plugin/33292-treasureboat-coding-tools)
since 30 August.

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

Version 1.2.7 is what the marketplace serves — tagged on 27 August, published on the 30th. It
fixed three small things, one of which was an empty file producing a stack trace instead of an
editor.

The framework it supports is one I rebuilt by hand because its original was discontinued. The
plugin that now edits its models was started by someone who does not write framework code, using
a tool I had refused to use, and I joined two days later.

I would not have predicted any part of that sentence a year ago.

[[ASK  KEN — one left. Do you want the line-count figure in at all? It is accurate and current
       (571 files, about 135,000 lines) but it invites "lines of code" arguments, and the post
       does not need it: files, tests and 28 feature areas already carry the scale. Cut the
       number and nothing else changes.

       Settled: the marketplace is named and linked — it has been live since 30 August, so the
       "still in the review queue" line was false and is gone. The listing is public and our own
       install pages already link it, so naming it discloses nothing. The repository stays
       unlinked because it is private. The download count is deliberately absent. ]]
