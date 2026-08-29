---
layout: page
title: About
permalink: /about/
---

## Who writes here

**Ken Ishimoto** has been shipping software since 1988, starting as a one-man team writing a
sales company's database on a Sharp X68000 under OS/9. He builds and maintains TreasureBoat,
a web application framework in the WebObjects tradition, along with the applications and
servers running on it.

When the closed-source framework it descends from was discontinued, the people around him
split between abandoning their applications and freezing on a version that would never move
again. He did neither: he reverse-engineered the Foundation, Enterprise and WebCore layers by
hand, because that was the only way to keep fifteen years of work alive. Everyone he asked
for help told him it was impossible.

He left Vienna at eighteen and has lived near Tokyo ever since. Japanese, English and
programming he taught himself — which is the same skill the work needs, applied earlier: work
out how something behaves when nobody is going to explain it to you.

**Paul Yu** is a business-oriented project, product and program manager who can actually
program — enough depth to work with the engineers, enough functional knowledge to work with
the subject matter experts, and a career spent becoming the domain expert by building the
system. He has built and run the same production SaaS for fifteen years: ninety-five tables,
most of the features, and all of the support email.

He had been using Claude across a dozen projects while Ken was still refusing to, and he is
the reason Ken started at all — though not by arguing.

## Why this blog

Almost everything written about AI and programming is about new code: a greenfield app, a
fresh repository, a demo built in an afternoon. That is not the code most companies have.

Most companies have something old, load-bearing and unfashionable, maintained by people
who know exactly what it will cost if it breaks. That is the interesting case, and almost
nobody writes about it.

So we write up real work — a database conversion, a framework migration, a bug that hid
for years — with the failures left in. Including the ones that were ours, and the ones
that were the machine's.

Ken was the skeptic, and had good reasons. That story is
[here]({{ site.baseurl }}{% post_url 2026-08-20-160-countries %}).

## What we keep finding

Software does not fail according to the model you have of it. It fails according to what it
actually does.

A map replaced with one that was strictly better on every axis anyone had written down, except
that the old one's weak keys were what kept the memory down. A build that compiled cleanly and
then could not resolve a field that was no longer there. An application deployed for two days
without failing, because nobody had logged into it. A list of every country on Earth, missing
dozens of them, with no note that it was partial. Migrations that recorded themselves as
having run, and had not. A deploy its own author believed was better, and could not verify.

That is the thread through all of this, and it is why AI belongs in the story rather than
being the story. It produces a convincing model of correctness faster than anything else we
have used — which makes checking that model against what the software actually does the
entire job.
