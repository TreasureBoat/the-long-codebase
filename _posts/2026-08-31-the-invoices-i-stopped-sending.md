---
layout: post
title: "The invoices I stopped sending"
subtitle: "I shipped a version I knew was not finished, and then I could not bill for it"
author: paul
series: modernization
excerpt: >-
  Nobody complained. Nobody would have known. Two years of evenings had rebuilt 161 of the
  165 screens in a fifteen-year-old product by hand, and I deployed it knowing it was not
  clean — after which I could not send the invoice.
cta_hook: "If you are one person carrying a product that people pay for, you already know the arithmetic in this post."
date: 2026-08-31 09:00:00 -0400
#
# Self assessments are a SUBTYPE inside the 26,526 evaluations, not a separate count.
# "of all types" is load-bearing — do not reword it into a narrower claim.
---

I chose not to invoice my customers last school year.

None of my customers complained about the service. There were some extra support emails, but
nothing extraordinary. I could have invoiced them and they would have paid. But I knew there
was quality debt in what I had shipped them, and I could not have told them how much.

That decision has more to do with AI than it sounds like. But the story starts earlier.

## What it actually is

A teacher evaluation product used by school districts. Administrators and their staff
perform observations, evaluations, professional development planning, and self assessments.
Each work product has a complex workflow between the administrators and staff member. Every
district wants the workflow and wording slightly differently. Each district has to comply
with their state's laws and requirements.

The SaaS product has been in production for fifteen years. Over that time we have had
between thirty and forty paying districts. Six are active today, and every one of them has
been a customer for more than five years. It holds 42,109 observations, 26,526 evaluations
of all types with self assessments among them, and 8,422 professional development plans.

Nobody has ever heard of it. It has never had a launch, a funding round, or a mention
anywhere. It has simply run, every school day, for fifteen years, for people who need it on
Tuesday morning.

That is the kind of software most software actually is.

## It was always the side thing

I have never done it full-time. This was a work of passion and a desire to help school
administrators get some time back. There has always been a day job. Most small software
products are somebody's second job, and that sets the budget for everything else: the
product got evenings and weekends, after the day job had taken the best hours.

For years that was enough, because I was not doing all of it. I had a partner doing sales
and the product was good enough.

He did it the way I did, around his own day job. He found the districts and had the
conversations, which is a specific skill and not one of mine. I built and maintained and
supported and hosted; he brought in the customers. Two people, four jobs, both of us working
evenings.

Then he took a job in the UAE, and moved there.

There is no villain in this. People's lives move and his did. What it exposed was a
dependency I had never once had to look at, because for fifteen years it had simply worked —
and selling to American school districts turns out to be done by someone who can be in the
room, on their time zone, in their week. From most of a day ahead it is not a smaller
version of the same job. It is a different one.

So selling came to me. And selling is the one job where nothing breaks if you skip it. A
support email has a deadline. A hosting problem has a deadline. A bug a school hits on a
Tuesday morning has a very sharp deadline. Finding the next district has none, so it quietly
stopped happening.

The base did not collapse. It eroded. No new districts after that, ordinary attrition, and
the number above is where it landed.

## Two years, four months of work

Ken was modernizing and refactoring TreasureBoat by hand — the framework my product sits on,
which he has maintained for twenty years.

We decided to follow him. Not because anyone asked us to, but because I had already been
left behind once by a framework version and I knew exactly what that costs. Staying close to
his refactoring meant my application would keep working with the framework as it moved.
Falling further behind meant one day facing a jump too large to make at all.

It was the right call. I still think it was the right call.

I knew the size of the effort. I built most of these features myself over fifteen years, so
I knew exactly how large the system was, and I knew where the object model was awkward — the
places with subclasses, and the places that should have had them and did not. 95 database
tables. Aligning with the new design patterns meant building a new component for every screen
— 165 of them, each an HTML template and the Java class behind it, 330 files. We did not
delete the old ones. The new UI went on top and the old components stayed underneath, so that
if we ran out of time we could still ship the old application without a large change. All of
it by hand, in evenings and weekends, on a fifteen-year-old codebase with no regression tests.

Ken started in June 2023. I joined him in September 2024 — fifteen months later, which tells
you most of what you need to know about how much of my evenings the day job was taking.

I never put anyone else on it, for two reasons. The first is arithmetic: six districts does
not pay for a development team, and it does not come close. The second is less comfortable.
I did not trust anyone else with it — fifteen years of my own decisions, including the ones
I regret, and the only two people I would have let near it were the man who wrote the
framework and me.

I went back through the commit history to write this, and it is not the story I remembered.
I remembered a hard spring and summer. What actually happened is that Ken worked through the
second half of 2023, then nothing at all for three quarters — not one component between
October 2023 and June 2024. Then a burst in the autumn of 2024 that put in most of the work,
then nothing again, then a second burst at the start of 2025 that finished it. Two years of
calendar time containing perhaps four months of actual work.

That is what a side project alongside two day jobs looks like when you chart it. Not steady
progress. Bursts, with months in between where life took the evenings, and each burst
starting with a week of remembering how any of it worked.

We are both good developers. Between us we converted 161 of the 165 screens. What we never
did was make it clean.

## The trade I made

We cut over to a new school year in August, in the weeks before the schools come back. That
is not a date I choose. It is the date the product has, it is the same every year, and it
does not move.

So at the end of that summer the choice was not whether to cut over. It was which version to
cut over on: the old one, which worked, or the one we had spent two years building.

I deployed what I had.

I knew there were issues still in it. Not catastrophic ones — nothing that took the product
down, nothing that lost anybody's data. Just the accumulated small wrongness of a very large
manual change that ran out of road before it ran out of scope. Things that did not look
right. Things I would have found if I had been able to test it properly, which I could not,
because there was nothing to test it with.

I deployed it anyway because the new version was better. Not marginally better — the
architecture underneath and the UI elements on top of it were both a long way ahead of what
fifteen years of incremental change had left me with. That part had worked.

What I could not do was tell you how much was still wrong. Every screen in the application
had a new component behind it, and one person clicking through on a weeknight does not cover
that surface. So I
made a trade: ship something I believed was substantially better, and accept an amount of
residual wrongness I had no way to measure.

I would find out how much, slowly, across the year that followed. This system has two heavy
stretches: the weeks before winter break, when formative evaluations are due, and the end of
the school year, when the summatives are. Whatever I had shipped in August had to hold
through both.

I had to make that call one way or the other. I still think I made it the right way, and I
would make it again.

I was embarrassed anyway.

That is the whole reason the invoices stopped. Not a decision I announced, and not one
anybody else was party to. I had put a version of my own product in front of paying
customers without being able to say what state it was in, and after that I could not send
them a bill for it. So I did not.

It is worth being exact about what that is and is not. It is not that the product got worse.
It got better, and those districts kept using it every day. It is that I had shipped
something I could not verify, and billing for work I could not vouch for did not sit right.

I invoice by school year, and I send the invoices in October. That is probably not best
practice, but it is what I have always done, and it means this was not a drift of small
deferrals. It was one decision, on one day in October, covering a whole school year that six
districts used every day and none of them were billed for.

## The same summer

That summer I also changed jobs, into a role in a technical domain I knew nothing about,
which is where I finally started using Claude Code properly. I have written about that part
[here]({{ site.baseurl }}{% post_url 2026-08-27-the-typewriter-argument %}).

So the same few months contained the end of it: a two-year refactor finally deployed, a
deploy I was not proud of, the invoices stopping in October, and the first time I used the
thing that would eventually let me fix all of it. I did not see any connection between those at the time.
They were just a bad summer.

## Why I am writing this now

Because I am about to send those invoices.

Over the past year I took the codebase apart properly. Java 8 to Java 21. MySQL to
PostgreSQL. A regression suite on a system that had never had one. The UI work I ran out of
road on last time, finished. The bar I set for sending an invoice again was not that the
code is perfect. It was that I can change it and know what I have broken.

I am not going to tell you a tool did that. I am going to tell you what it cost, what it got
wrong, and what I had to go back and fix afterwards — because I have something almost nobody
writing about this has. I tried the same job by hand first, and I have the deploy to prove
it.

That is the next post.
