---
layout: post
title: "The token count measures your codebase, not your work"
subtitle: "5.1 billion tokens across sixteen working days, 0.11% of it output. By that measure it was the most effective I have ever been. The most valuable thing it bought was a colleague changing his mind, which appears on no dashboard anywhere."
date: 2026-09-04 09:00:00 -0400
author: paul
excerpt: >-
  Token usage is the easiest number to collect about AI-assisted development, so it is the one
  that ends up on dashboards. I pulled mine apart: 91.95% cache reads, 0.11% output, and a
  window that misses every episode I have written about. A token count is a measure of how much
  context you carried, and context scales with your codebase.
cta_hook: "If your organization has started reporting token usage, run this breakdown on your own numbers before anyone attaches a target to it — I would like to know whether your ratio looks like mine."
#
# Editorial notes — front matter only, never HTML comments.
#
#   Short companion to the two September posts. Not a series entry.
#   All figures are Paul's own local stats cache, verified digit by digit. Deliberately makes
#   NO claim about what other companies measure, and NO dollar figures: costUSD is 0 in the
#   file (subscription), so a computed bill would be a number he never paid.
---

Token usage is the easiest number to collect about AI-assisted development. It arrives for free,
it is denominated in something that looks like effort, and it can go on a dashboard without
anyone having to decide what it means.

I have no data on what your company measures and I am not going to guess. I have my own numbers,
so I pulled them apart to see what a token count is actually made of.

## The breakdown

```
TOTAL                    5,140,846,107
  cache read             4,727,054,864   91.95%
  cache creation           406,197,588    7.90%
  input                      1,976,212    0.04%
  output                     5,617,443    0.11%
```

Twenty-five sessions across sixteen working days between 14 December 2025 and 10 February 2026.
Two caveats before anything else, because this is a post about honest measurement. The 92,856
messages behind those tokens count every tool call and tool result, not turns I typed. And six
days in December carry 80% of them — this is not a tidy sample, it is one developer's actual
lumpy usage on one codebase.

**Output is one tenth of one percent.** Ninety-two percent is cache reads, and the ratio that
explains them is in the same block: 4,727,054,864 read against 406,197,588 written, so for every
token put into the cache, 11.6 came back out.

That is not waste. It is how a conversation with a large codebase works — every turn re-sends
what came before, and the cache is what stops that being ruinous. But it does mean the headline
number is very nearly a measurement of how many turns I took and how much code they had to
carry. It scales with the size of the repository and the length of the session. What it does not
scale with is whether any of it was worth doing.

## What that winter actually was

There is a second problem with the number, and it is not a rounding error. I do not fully know
what it was spent on.

Most of that window is one push: building an IntelliJ plugin for our framework, started at the
end of November and now 766 commits and a Marketplace listing later. Some of the rest is
probably day-job work that landed on the same account, and I cannot cleanly separate the two
from this file. So the 5.1 billion is not even attributable to a single product, let alone to an
outcome.

I remember that winter mainly as running out of tokens. Constantly. If token consumption were
the measure, that was the most effective I have ever been.

The usual version of the mixed-account story runs the other way. When people worry about this,
they mean company resources going into somebody's side project, and that does happen. Mine went
the other direction: a personal subscription, paid for by me, producing a plugin our product
uses and an argument that changed how Ken works.

I am not after credit for that. The point is that the meter cannot tell the two apart. It sees
tokens on an account. It does not see which way the value flowed, and a per-seat dashboard built
on it would have been wrong about me twice over — counting spend against me personally, and
showing nothing at all on the ledger that actually got the benefit.

Here is what the plugin was actually for. It was an argument aimed at one person. Ken had been
unconvinced about Claude Code, and I did not think I could talk him into it, so I built something
in his own territory — a plugin for the framework he maintains, in the IDE he uses — and sent him
screenshots. He bought his IntelliJ licenses and started on the free plan that same month.

Nine months later Ken writes the other half of this blog, and it is the deeply technical half:
six posts in the last sixteen days, on a database cutover, a memory leak that hid for fourteen
months across four applications, an eight-year gap between a framework's last release and its
end-of-life notice, and
[the migration published this morning]({{ site.baseurl }}{% post_url 2026-09-04-finished-and-thrown-away %})
that he finished and then abandoned on purpose.

That is the conversion I actually care about, and it is the reason this blog has two bylines. Not
a tool being adopted — a skeptic becoming the person you would ask.

Now trace that back to a token count. You cannot. Not the spend that produced it, which is
tangled up with a day job in a file that stops in February, and certainly not the outcome, which
is a person changing his mind and then becoming good at something. There is no dashboard on
which that appears, and it is the most valuable thing in the window by a distance.

## What the meter did not see

That file stopped updating on 10 February. Every piece of work I have written about on this blog
happened after it. The Selenium suite that
[passed 113 of 115 by clicking buttons nobody could reach]({{ site.baseurl }}{% post_url 2026-09-01-the-tests-were-green-because-nobody-was-clicking %})
was written in the first week of January — and the file has no entry for those days either. The
August audit that found out it was hollow is not in there. Neither is a day of the
[optimization episode]({{ site.baseurl }}{% post_url 2026-09-02-ninety-four-entities-none-of-them-read-only %}),
which ran from April to August.

So I have a five-billion-token record of sixteen days in winter, and no record at all of the
seven months containing everything I would actually want to account for. Nobody noticed the
number had stopped, which tells you how much weight it was carrying.

## The two episodes, since the meter cannot speak to them

Both are about the same thing, and neither is visible in a token count.

The test suite produced 115 tests and a green pipeline while 75 of its clicks called the
element's JavaScript handler directly, which passes on controls no user can reach. That is worse
than nothing, because it occupied the slot where real coverage would have gone, and it held that
slot from January to August. Whatever it cost in tokens, the number would have looked like
healthy activity.

The slow page got five correct optimizations in sixteen days, every one measured before and
after and three of them `EXPLAIN`-verified against a copy of production. None of them fixed the
page, because the cost was not in the queries. They were not wasted — several still keep other
pages fast — they just were not the thing. What eventually fixed it was one question, and the
answer came back in a single pass.

Token consumption tracked effort faithfully in both. Effort was not the variable that mattered
in either.

## What I would rather count

I do not have a clean replacement, and I distrust anyone who offers one quickly. None of these
three is ungameable — nothing is, that is the whole lesson of the pass rate. What they have in
common is that the shortcut shows up *in* the number rather than hiding behind it.

**Defects the work surfaced in production code.** Not bugs it introduced, bugs it found. Making
those Selenium clicks real took the failure count from 2 to 9, and every one of the seven new
failures was a control no user could reach. The defect with a person on the other end of it
surfaced after that: a multi-select that never reached the server, so a user who made a selection
and then did anything but edit the text beside it lost it silently, in production, for months.

**How many tests have both ends real.** This is the one from the first post, and it is a pair,
not a single check: did a real input path produce the change, and does the assertion read state
the test could not have written itself. Either half alone fails. A test that drives the UI and
then checks the page it rendered is grading its own work. A test that checks the database but
reaches past the UI to set up the state is grading a different application — the `setContent`
shortcut I nearly kept would have passed the persisted-state half perfectly. The count is how
many do both, and it is never 100%.

**Work deleted.** The optimization episode ends with the old screen scheduled for deletion rather
than made faster. Nothing has been removed yet — the replacement is behind a flag that still
defaults to `false` — so this is a number I expect to count, not one I have. Nothing about a
token count distinguishes 500 lines added from 500 lines deleted, and in a fifteen-year-old
codebase the deletion is frequently the better outcome.

None of those is as easy to collect as a token count. That is the trade. The easy number is easy
precisely because it does not require anyone to decide what good looks like.

## The obvious objection

A token count is an input measure and nobody claims otherwise — you track it for cost, the way
you track any bill.

Two things about that. The raw count is not the bill: cache reads are billed at a steep discount
to input and cache writes at a premium, so the four lines above rank completely differently by
money than by volume. The 91.95% line is the cheap one per token and the 0.11% line is the
expensive one. And my own file records a cost of zero for both models in it, because this was
subscription usage. The count was attached to no bill at all.

So control cost if you are paying per token — but control cost, not the count. They are different
numbers in the same units and they disagree about which line matters.

What I am actually worried about is the step after, where a number that exists gets a target
attached to it. I have done the equivalent: I reported pass rates for years on a suite that was
113 of 115 green and hollow. Both directions fail here. Reward high usage and you get long
conversations. Penalize it and people avoid the tool on the hard problems, which are exactly the
ones with the most context to carry — and context is 99.9% of the count.

Ken's version of this is that cheap execution changes which questions get asked. The estimate
used to be a forcing function because it arrived *before* the work and made someone decide
whether to start. A token count arrives afterwards and decides nothing. It just tells you the
meter ran.

Which leaves the part nobody sells you. Every metric in this post that would have been worth
having — did the work surface a real defect, does the test touch both ends, did the screen get
deleted, did a skeptical colleague become effective — is a judgment about what we are trying to
achieve, and somebody has to sit down and make it. The tool will not. It will report its own
consumption with great precision, indefinitely, and that number will keep looking like a
measurement.

The vendor can sell you the capability. Deciding what counts as it working is still ours.
