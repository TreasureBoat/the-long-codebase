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

[[NOTE  KEN — one correction and one flag for "The thing that lasted was not the code".
        The correction: the framing credits me with inventing the specs, and they were extracted
        from your Eclipse plugin. The flag: the two counts in that section both run backwards
        against the repo — 145 of the 207 spec files were created by me, and about a third of
        src/main/java still blames to me. I am NOT asking you to credit me with more; I am fine
        with the code being disposable and have said so above. But both numbers are checkable by
        anyone you show the repo to, so they are worth re-running before publish. ]]

[[NOTE  KEN — two things for you to check in this section, both about your own facts.

        (1) The "about two weeks" in "160 countries" and the "forty-eight hours" in this title
        are not in conflict, and the section above now separates them: the weekend of 29-30
        November was the initial prototype, December was the rest of the prototyping. Nothing
        needs correcting in the published post.

        (2) I have said you went free -> Pro -> Max in December. That is your spending and your
        story, so cut it if you would rather not have it on the page.

        (3) Bigger one: I have written that YOU challenged me to build the first plugin module.
        Your subtitle and opening currently read as though I started it unprompted, which makes
        the whole thing more surprising than it was and gives me credit that belongs to you. If
        my memory matches yours, your half probably wants to say so. ]]

[[DECIDE  the byline. Front matter carries one author key and currently says Ken. Either the
          layout gets a coauthor field, or the section headings carry it in the text. ]]

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
