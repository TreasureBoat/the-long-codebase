---
layout: post
title: "Ninety-four entities, none of them read-only"
subtitle: "Five database optimizations in sixteen days, every one measured, EXPLAIN-verified and still holding. The database was never the cost. The fix was a read-only entity over a view — a technique with no example anywhere in the model."
date: 2026-09-02 10:00:00 -0400
author: paul
excerpt: >-
  A summary screen was slow enough to blow past a 60-second proxy timeout. We cached it, replaced
  in-memory counting with SQL counts, and added indexes verified with EXPLAIN against a production
  backup. Every step was a real improvement and none was wrong — they were answers to the question
  I had asked. The one that fixed it was a technique the codebase contained no instance of.
cta_hook: "If you have a page that has survived five rounds of optimization and is still slow, the question in this post is the one worth asking about it."
#
# Editorial notes — front matter only, never HTML comments.
#
#   Standalone technical vignette, program-manager perspective. Not a series entry.
#   Rule 7: no district, client or organization names, no production record identifiers.
#
#   Title deliberately NOT "SQL was two percent of it": the 2% / 99% profile is an April
#   local run that was retracted the next day as an IntelliJ artifact, and there has been no
#   production profile of this page since the July Postgres cutover. The post says so.
---

We have a screen that summarizes a customer across several entities: for each person, how many
observations this period, where they are in the evaluation cycle, whether a professional
development plan is open. It is the kind of thing that looks trivial on a whiteboard and is not,
because every column is a rollup over a different relationship and the answers have to agree with
the rules the rest of the application enforces.

It worked, and it was slow. How slow is a longer answer than I expected, and getting it wrong is
the first thing that happened.

In April a local profiling run recorded `132,946ms total / SQL 2% (84) / append 99%` for a
ten-row page. That number went into the project's notes. The next day I restructured my IntelliJ
workspace onto a Maven reactor, the render time dropped to normal, and the whole thing was
re-attributed: stale module graph, JAR re-resolution per request, an artifact of my machine and
not a real cost. The note was rewritten to say so, and it ended with a recommendation — the
rollups are fine, and per-row rollups can now be added to other pages "without the prior caution."

That was written down, in durable project memory, and it was wrong. It was also wrong in the
direction of doing more of the thing.

What dislodged it was production. In May a trace caught a superintendent's post-login render at
**78 seconds**. Apache's `ProxyTimeout` is 60, so what the user actually saw was a proxy error
page. A thread dump from the same session showed four more of their queued requests blocked in
`TBWSessionStore.checkOutSessionWithID`, waiting on the session lock the slow render was still
holding. One page was taking the whole session down with it. The commit that started the fixes had
to say the earlier conclusion was wrong before it could do anything else.

## Sixteen days of correct answers

What followed was, in fairness to everyone involved including the machine, good engineering.

**A per-render cache.** The four heavy accessors were each being re-walked on every pass of the
framework's multi-pass render cycle — N times per row, per render. Caching them per render, keyed
by person, collapsed that. Real fix, real improvement.

**SQL counts instead of in-memory counting.** Those accessors fetched a person's entire history
into the JVM and filtered it there. They were replaced with `SELECT COUNT(*)` helpers on the model
object. The cost moved off the to-many fetch entirely.

**Indexes, with the plans to justify them.** One table had no index covering the district column at
all, so the query was a full scan of 32,000-plus rows followed by a filesort. A composite index on
`(district, lastModified)` let the engine walk the rows already sorted and stop at the limit.
Verified against a production backup: 32,719 rows examined dropped to 827, filesort gone.

**A dead `OR` branch, removed.** One page qualifier carried a clause for future-period draft
records. Seventeen years of production history contained zero of those — nobody had ever used it.
Meanwhile the `OR` was forcing a nested-loop join for every district row and knocking the second
column off the composite index. Dropping it, plus a three-column composite, took that path from
5,942 rows and two nested-loop joins at 3-4 seconds to a single row with no filesort.

**A restructured qualifier.** The remaining slow path scattered the district equality inside an
`OR`, which meant the index could not seek on it. Pulling the district filter to the top and
leaving the visibility checks as a residual produced the same plan as the fast case.

Five rounds over sixteen days. Every one measured before and after, three of them verified with
`EXPLAIN` against a copy of the production database. None has been reverted and several still keep
other pages fast.

One of them quietly narrowed behavior. The qualifier restructure scoped principals to their own
district, where the old shape let them see evaluations from any district they happened to be named
on. The commit says so under its own heading, argues it matches the documented intent, and notes
cross-district assignments do not happen in practice. I agree with all of that and I still count it
as a behavior change that rode in on a performance ticket.

The summary screen stayed slow.

## The profile

When we came back to this during the current redesign, the only profile anyone had was that April
one — the retracted one. Nobody had ever re-measured in production, and nobody has since: we moved
the database from MariaDB to Postgres in July, which invalidates the query side of any older
number anyway. The plan document written in August carries the April figures forward as though
they were clean. I am not going to repeat that here.

So I will be careful about what survived the retraction, because it is not the duration.
`132,946ms` was contaminated. What was not contaminated is the **shape**: SQL was a rounding error
in that run, and no production trace since has suggested otherwise. Ten rows, four accessors each,
every one filtering a person's entire history in the JVM, re-walked on every pass of a multi-pass
render.

Whatever the true ratio is, the cost is not in the queries. Every index, every composite, every
`EXPLAIN` plan had been careful, verified work on the part that was already cheap.

I want to be exact about where that goes, because the obvious reading is the wrong one. Nothing in
those three weeks was a mistake. The work was profiled, measured, checked against real data, and
reported with honest before-and-after numbers, and it made other pages faster in ways that have
held up since May. It was answering "how do we make these queries faster."

That was my question. I opened the ticket, I wrote it as a database performance problem, and
everything downstream was a correct answer to it. A good answer to the wrong question is not a
failure of the person answering.

## The suggestion I turned down

Months later, starting a work-oriented list on the same data, the proposal that came back was to
reuse the existing staff board. It already renders a teacher list. It is already people-centric
rather than record-centric. It exists, it works, and building on it would have been the fast start.

Every reason to reuse it was visible in the code. The two reasons not to were not.

The first is that I had watched that page fail in production in May and I remembered what it cost.
The written record did not agree with me. The April note was still sitting in project memory saying
the rollups were fine — and it did not just say that in general, it named the page: *add per-row
rollups to other pages, e.g. the Personnel List, without the prior caution.* That note was never
amended after the production traces contradicted it. So the documented position, four months on,
was that reusing those accessors on exactly this kind of screen was safe.

I do not know whether that note was read. I know it said precisely the thing that would produce
this suggestion, that it was still there, and that the only thing arguing against it was my
recollection of a proxy error page in May.

The second reason is that I did not think the component was reusable here at all, independently of
speed. The existing list is administrative — login, email, invited, active, supervisor. That is
onboarding and access. A work-oriented list wants last observation, cycle position, plan status and
the create actions. Same people, different question, and the column sets barely overlap. Reuse
would have meant bending a screen built to answer one question into answering another.

Both objections were mine, and neither was clever. One was memory of an outage. The other was
knowing what the screen is for.

The rest followed easily once I said no. The reasoning got written up properly: everything the new
list wants to display is exactly what makes the old page slow, so building it the same way would
put a 78-second render on the landing page, where every user meets it at every login. That is the
right analysis and it is in the plan document. It arrived after the decision, not before it.

The alternatives offered at that point were still in the lane we had been driving in. The April
note had already laid out three options, ordered by effort: cache per component, one `COUNT(*)` per
rollup, or one `GROUP BY` per page across the ten ids — the last of which, it said, "would bring
post-login to well under a second." All three are correct. All three are *make the rollups
cheaper*. None of them is *stop computing them*.

## Changing direction

We had moved the production database from MariaDB to Postgres in July. Sitting with three correct
answers I did not want, the thing that occurred to me was not an optimization at all: **stop
computing the rollups and start reading them.**

I want to be honest about how long that took to arrive, because the gap between May and August is
part of the story. I had never mapped an entity onto a database view in this framework. Neither had
anyone else here. Three model files, ninety-four entities between them, and not one of them was
read-only — `grep isReadOnly` across the whole model returns a single hit today, and it is the one
this work added. There was no example to copy, no precedent to suggest the shape, and nothing in
fifteen years of this application that would have put the idea in front of me. It is an ordinary
technique and it was not in our vocabulary. Define a Postgres view that produces one row per person with the counts and
the last-observation date already resolved, then map a read-only entity onto that view and let the
ORM treat it as an ordinary table.

Claude's reaction was immediate and enthusiastic, and then it did the part I could not have done
quickly: confirmed that TreasureBoat actually honors a read-only entity at fetch time —
`TBEntity.isReadOnly()`, enforced at `TBEnterpriseDatabaseContext:2296` and
`TBEnterpriseSubDatabaseContext:977` — found the three existing migrations that already run raw SQL
through the migration runner, so the delivery mechanism was established rather than invented, and
worked out that the view has to produce exactly one row per person or the entity has no usable
primary key.

What I like about this fix is that it does not escape the framework. This is a strong MVC codebase
with a real object-relational layer, and the tempting move — drop to raw SQL inside the component,
hand-roll the query, bypass the model — would have traded the performance problem for a
maintenance problem. A read-only entity over a view is still an entity. The paging component
still pages it, the role-scoping filters still filter it, the column headers still sort it. We gave
the ORM a better table to map, and everything above it stayed ordinary.

## What it actually bought

The view ships with two indexes, one of which covers a foreign key that **had no index at all** —
only the primary key — so the plan had been sequentially scanning that table once per person. For a
twenty-person district the view goes from **43.7ms to 0.7ms**. The old screen is not being
optimized. It is scheduled for deletion, and the performance ticket closes by removing the page.

As of this writing none of that is live. The replacement ships behind
`boise.nextgeneration.staffWorkList.enabled`, which defaults to `false`, with a test asserting the
shipped default. What I am reporting is a view, an entity, five verification tests through the
save-path harness, and a plan — not a page anyone has used yet. The 43.7ms is the view's own query
time, not a page render. Nobody has timed the new screen end to end, because it is not on.

The speed is not the interesting part. This is:

> "Everyone with no observation this period, longest since observed first."

That is the query a principal actually wants in evaluation season, and with the rollups computed in
Java it was not slow, it was **not expressible**. The sort cannot go into the fetch, because the
value does not exist until the render has walked the graph — so answering it means fetching every
person in the district and walking each one's history before the first row can be ordered. Against
the view it is an ordinary qualifier and an ordinary sort, pushed down to SQL along with the
paging. The reframe did not just make the old screen faster. It made a screen answerable that was
not.

## What it cost

Stated plainly, because a post that only lists the wins is an advertisement.

The model is no longer portable — the view is Postgres SQL and it lives outside the model file. The
view must exist before the entity is used or the fetch fails at runtime, which makes migration
order a correctness requirement rather than a convenience. The entity is read-only, so it has no
write path and creating a record does not refresh a row; anything that creates has to invalidate
and refetch deliberately.

One column did not make the move, and it is the same column that did not become a `COUNT(*)` in
May. It depends on a weighted score computed in Java over two district preferences and a sum across
per-domain values, none of which are stored columns. It has now defeated both approaches for the
same reason: the number it displays is not in the database. Reproducing that
in SQL would mean maintaining the weighting model in two places forever, so it was left out on
purpose rather than half-ported. If we want it, the honest fix is to store the score at save time
first.

And the migration runner does not wrap a script in a transaction, despite its documentation saying
otherwise — it commits whatever succeeded even on the failure path. Every statement in the
migration had to be written to be re-runnable, and that was verified by running it twice rather
than assumed.

## What the human actually supplied

It is tempting to write this up as the machine getting stuck and a person rescuing it. That is not
what happened, and the more useful version is duller: I contributed three things that were not in
the code, and one question that was not in the codebase.

1. **A memory that outranked the written record.** I remembered a proxy error page in May. The
   project's own notes said the opposite, in writing, naming this exact kind of screen as safe to
   build. Nothing in the repository would have corrected that. I was the correction, and only
   because I happened to have been there.
2. **We changed database platforms two months ago.** A fact about July, in a repository the task
   had no reason to open, and what made a view a live option rather than a hypothetical.
3. **Knowing what the screen is for.** The old list answers an administrative question and the new
   one answers a working question. That judgment is not recoverable from the code, which shows you
   columns, not purposes.

Then the question, which was not expertise at all and is the one that mattered: **would the object
layer sit on top of a database view?** I had never done it. The model contained no read-only entity
to copy. I could not have told you whether the framework would honor one. What I had was a database
concept from outside the application and enough nerve to ask.

That is the uncomfortable part, and it is narrower than it first looks. The technique was not
unknown to the machine — the moment I asked, the confirmation came back in a single pass with a
class and a line number, plus three prior migrations that already ran raw SQL. The delivery
mechanism had precedent. Mapping an entity onto a view had none. So the technique was known and
would not be volunteered, because what gets volunteered is ranked by fit to the code in front of
it, and ninety-four entities said *this is not a thing we do*.

I have watched teams do the same for a year at a stretch: the page is slow, the page is the unit of
work, so every sprint produces another optimization to the page. A frame is invisible from inside
it. That is not a property of machines, and here it was written down — a note in project memory
saying the rollups were fine and should be spread to other pages, which took a production outage to
dislodge.

So the argument for staying in the loop is not that you need to check the work. The work was fine,
and better than mine would have been. It is that direction is a separate job from execution, it
runs on things that are not in the repository, and somebody has to be accountable for asking
whether the question is still the right one.

Which makes the useful prompt something other than "is this right." It will be right. The question
worth asking is: **what would you have suggested if this codebase were not the only thing you had
read?**
