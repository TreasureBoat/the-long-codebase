---
layout: post
title: "The typewriter argument"
subtitle: "A manual typewriter, three decades of reverse engineering other people's code, and the year I finally turned it on my own"
date: 2026-08-27 09:00:00 -0400
author: paul
series: modernization
description: "I managed engineers who used AI for years without touching it myself. Then a new job removed every excuse, and I turned what I learned on my own fifteen-year-old codebase."
excerpt: >-
  I managed engineers who used AI for years without once using it myself. Then a new job
  removed every excuse — and left me with a year of accumulated code that ran fine and was
  quietly bad.
cta_hook: "If you are the only person who can break your own product, that is the position all of this is written from."
---

At the risk of dating myself, I wrote my graduate school papers on a manual typewriter.

If you have not used one, the thing to understand is that a mistake in the third paragraph
of a twelve-page paper meant retyping from the third paragraph. Not correcting. Retyping.
You learned to think a sentence all the way through before you committed it to the page,
because the page charged you for every change.

Then the IBM PC arrived, and a dot-matrix printer with it, and the cost of being wrong
collapsed. Fix the third paragraph, print the whole thing again. Twenty minutes instead of
an evening.

I do not think I would have finished the MBA without that. I was working, the papers were
long, and the arithmetic of retyping was beating me. The machine did not make me smarter. It
removed a tax on revision I could not afford to keep paying.

## Hands off the keyboard

I have been in software since around then. Most of it as some flavor of technical project
manager — technical enough to work through the details with the engineers, senior enough to
be held responsible when the decision turned out to be wrong. Consulting, engineering
leadership, solutions architecture. Hands on the keyboard for stretches of it, hands off for
longer stretches.

I reverse engineered the US Navy's ammunition system off COBOL and IDMS by hand. There was
no other way to do it then.

For the last fifteen years I have also had a side application of my own. A teacher
evaluation product used by school districts. This application has a very broad feature set
and substantial workflow between administrators and their staff. I wrote the first version,
I maintain the system as I have time and I still answer the support email, and every
architectural decision in it is mine, including the ones I regret. As a side business, I am
a sole proprietor, like Ken, switching between job functions, trying to ship the best product
the constraints allowed.

## All of it built by hand

This fifteen-year-old application was built by me or a small group of contractors with an
integrated development environment (IDE). We tried to apply the best industry practices at
the time, but technical and functional debt just builds over time like any other system. I
kept adding features over the years based on customer feedback, which introduced variations
in design patterns and coding styles. Just like every long-lived system.

My exposure to AI before last year was secondhand, and it was management exposure. I ran a
very strong engineer who used both hosted models and a locally hosted RAG setup, and I
directed his efforts with it. I understood the work well enough to brief executives on it. I
never once used it myself to write a line of code. That is a slightly embarrassing thing to
put in public, but it is true, and I suspect it describes a great many people at my level.

## Every excuse went at once

Then I changed jobs, and every excuse went at once.

The new role put me in a technical domain I had no background in. I had to build a large,
complex system with many parts — test bench software, a data ingest engine, a data
visualization application, Jupyter notebooks — in Python, which I had never written. I had
to design a database schema for a subject domain I was learning as I went, with no subject
matter expert to ask.

As the applications came to reality, I also had to become the Azure cloud deployment and
operations engineer. Not the manager of my Azure cloud engineers but actually do the work.
There was no team to hand any of it to. I was the team.

At the time I knew I was well over my head. I do not think I would have succeeded without
Claude Code. I am aware of how that sentence reads. It is not enthusiasm. It is an
assessment of the distance between what the job needed in month one and what I actually
knew.

So — the typewriter again, thirty-five years later. Same shape. A tax I could not have
afforded, lifted.

## Where the comparison breaks

Except the comparison breaks in one place, and the place it breaks is the interesting part.

The PC never wrote a paragraph for me. It only made my own paragraphs cheap to change. This
writes the paragraph, and it writes a wrong one with exactly the same confidence as a right
one. Ken has already documented [four separate occasions in a single working day]({{ site.baseurl }}{% post_url 2026-08-27-six-bugs-that-never-said-a-word %}) when it told him something plainly and
incorrectly. My experience matches his, and I have a second problem he did not have: over a
year of accumulated code that ran fine and was quietly bad. Not broken — bad. It took two
major review passes to deal with it, and I will write about both, because the cost of
cleaning up after this tool is the part nobody publishes.

What got me through a domain I did not know was forty years of recognizing when a system is
not behaving. Enough to catch the things that looked right and weren't. Someone without that
would have shipped them.

That is a real difference from the printer, and I would rather say it here than have it
discovered later.

I will write about that year properly another time. It needs its own series.

This one is about what I did with the lessons afterward.

## The codebase that was sitting there

Because the whole time I was learning this at work, my own fifteen-year-old codebase was
sitting there. Java 8. MySQL. No regression coverage worth the name. An interface that had
aged the way interfaces do when the only person who can change it is also doing sales,
support and hosting. Ken had helped me try to modernize it by hand, part-time. We did not
finish. I will write about why we did not finish, because that failure is
the most useful thing I have to measure against.

Over the past year I went back at it. Java 8 to Java 21. MySQL to PostgreSQL. A Selenium
suite built from nothing on a codebase that had never had one. Interface work. The
unglamorous business of making Google notice the site again. The next stretch is a proper
hardening assessment of fifteen years of Java, which has never had one, and I will write
that up as it happens rather than after I know how it ended.

None of it is novel. Thousands of people have done every one of those migrations. What is
worth writing down, I think, is the position I was in while doing them: one person, no team,
a live product with paying customers, a codebase too large to hold in my head, and nothing
to catch me if I broke it.

One problem per post, roughly in the order I hit them. Real numbers where I have them. The
parts that went badly included, because the parts that went badly are where the information
is.

The first one is about why I stopped sending invoices.
