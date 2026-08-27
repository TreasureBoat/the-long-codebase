---
layout: post
title: "Six bugs that never said a word"
subtitle: "A day converting a five-year-old production database with Claude Code — including the four times it was confidently wrong"
date: 2026-08-27
author: ken
---

<!-- PAUL: if you want a few lines at the top, the useful angle is the one only you have —
     you had already used Claude across a dozen projects while Ken was still refusing to,
     so you watched this kind of day from the outside long before he did. -->

## The job

In October I have to replace a shop system that has been running since about 2013. The
old one is on Java 8 and a version of our framework that is five years behind. The new
one is written, deployed and mostly tested. What was left was the part that actually
frightens me: the database.

The client has agreed to a two-day window. That sounds generous until you remember what
it replaces — for years my only slot for this kind of work was an hour on a Saturday
night, because that was when the shop was closed.

The important decision was made a few days earlier, and it wasn't a technical one. The
goal was **not** to convert the database. The goal was to produce a *script* that
converts the database, because it has to run at least three times: now, again after our
tester comes back from holiday and her bug list moves the model, and finally on the
cutover day itself. I had done this conversion once by hand, about a year ago. Those
notes were stale on both ends — the app's entities have changed, and the live database
has a year more data in it.

Everything below happened in one working day.

## What went fast

The first useful thing was not code, it was a question I hadn't thought to ask: how far
behind is the old database *actually*?

Our framework tracks migrations in a table. The old production database said the
application model was at version 40. The new application ships 43 migration classes. So
the gap wasn't five years — it was **two steps**. The framework's own migration engine
could do most of the work, and all we had to write by hand was the part migrations can't
do: renaming tables, rebuilding constraint names, and clearing the way so the framework's
own chains could replay without colliding.

That reframing came from reading the migration table and the migration classes together,
which is exactly the sort of tedious cross-referencing I'd have skipped and guessed at.

The second useful thing was measurement. We're likely to move from FrontBase to
PostgreSQL later, and PostgreSQL truncates identifiers at 63 characters. Thirteen of our
constraint names were already over. Shortening the prefix wasn't enough — seven were
still too long — because the real problem was structural: foreign key names repeated the
destination table, which the source column already implies. Dropping the redundancy took
the longest name from 84 characters to 48. That's the kind of thing you either measure or
argue about.

## The trap that says nothing

Then the first real bug, and it's the one I keep thinking about.

To rename a table cleanly you also rebuild its primary key constraint. So: drop the old
constraint, rename, add the new one. Perfectly reasonable.

In FrontBase, `drop constraint <primary key> cascade` **silently drops every foreign key
that references that primary key.** No error. No warning. The script reported success.

We lost eight foreign keys that way, including `OneOrderLine → Product` and
`OneOrder → InvoiceGroup` — the referential backbone of the order system. Nothing would
have told us. The application would have run perfectly. We'd have found out months later,
when orphaned rows started appearing in a shop that had been quietly accepting them.

We only caught it because the next step failed for an unrelated reason and, while looking
at that, we counted the constraints and found 50 where there had been 58.

That is the whole argument for rehearsing a cutover, in one bug.

## Where the machine was wrong

I want to be exact about this part, because it's the part that usually gets left out.

Claude was confidently wrong four times in one day.

It told me the application tables had **no** NOT_NULL constraints. They had about 170 —
just named with a prefix it hadn't searched for. It corrected itself once it looked
properly, but the first answer was stated plainly and would have shaped the script.

It dropped three columns from the person table because the model no longer defined them.
Reasonable — except the pending migrations *delete those columns themselves*, and now
they had nothing to delete. The application refused to start. My reaction at the time was
"looks like a simple drop mistake", which it was; the useful correction was that the test
isn't *"is this column dead?"* but *"does a migration that is about to run remove it?"*

It spent six rounds building a theory about a framework-wide bug in a lookup function,
complete with a written-up memo about how every guarded migration across all my apps
might be silently skipping. The lookup was fine. It had been comparing the running
application — which was connected to one database — against SQL it was running against a
*different* database. I noticed the two numbers couldn't both be true and said so, and the
whole theory evaporated in one line.

And it flagged a qualifier format as the likely culprit for that same non-bug, when the
framework's own documentation shows that exact form as correct usage.

Four wrong calls. Every one caught by knowing the system, not by being cleverer.

## Where I was wrong

Fair is fair.

I assumed the old application predated our policy system entirely, and that converted
users would land with no roles or permissions — I had it written down as a cutover task
that needed app-specific knowledge nobody else had. It wasn't true. The old database
already had the policy tables populated, and the framework migrations created 223
policies and 9 roles on their own. A task I'd been dreading for months didn't exist.

I also had a year-old script with a section that drops fourteen columns. Copying it would
have destroyed live columns — a framework we still use has since claimed them. And it was
missing one that has appeared since. Re-deriving beat copying, and I'd have copied.

## Four bugs in the framework, all of them quiet

While testing the conversion, one broken migration file produced ten errors across
completely unrelated files. My guess was that the first one triggered the rest, and that
turned out to be right: the failed object stayed in the editing context, so every later
save picked it up and failed the same way.

Fixing that turned up worse. This, in our own framework, for years:

```java
} catch (final Exception e) {
    log.error(...);
} finally {
    runSuccessfully = true;      // set even when the catch just fired
    ...
    lock.unlockXML(...);         // records the migration as done regardless
}
...
if (!runSuccessfully) {          // unreachable
    throw new MigrationFailedException(...);
}
```

`runSuccessfully = true` sits in the `finally`. So it is true whether the migration
succeeded or threw. The failure branch below it has been dead code since it was written,
and the marker that says "this migration has run" was written even when it hadn't.

On this conversion, three migrations failed, were marked done, and would never have run
again. Their roles, policies and a report would simply have been missing — while the
application logged a clean start.

We fixed it so a failed migration is loud, is retried, and never blocks a boot. Then
found two more of the same shape: a migration skipped by its condition logged *nothing at
all*, and a lookup that matched several rows was indistinguishable from one that matched
none.

## The one that would have hit on cutover day

The last bug is my favourite, because it only appears on the *second* boot.

The application creates its default roles at startup, before the data migrations run, and
it looks them up by a short identifier — `A` for admin. Our 2013 admin role had a name but
no identifier. So the first boot found nothing, created a *second* admin role, and
twenty-seven seconds later a data migration matched the original by *name* and filled in
the identifier. Two roles with identifier `A`. Every boot after that died before it
started.

Boot once: fine. Boot twice: dead.

If we had converted the database on the Sunday and started the app once to check it, we'd
have shipped that. The symptom — *"more than one role matched"* — gives no hint that the
cause is a thirteen-year-old row missing a column value.

## What I take from it

By the end of the day the conversion ran clean from a fresh dump in about three minutes,
and we'd run it seven times. 204,820 order lines intact. The app boots and logs in.

Six real bugs. **Five of them were silent** — work that didn't happen, reported as if it
had. That's the actual lesson, and it has nothing to do with AI: the dangerous failures
aren't the ones that throw. They're the ones that return success.

On the AI part, the honest summary is narrower than the marketing and more useful. The
machine was very good at the things I avoid because they're tedious and exact — reading
84 table definitions against 18 models, counting constraint name lengths, noticing that a
flag was set in a `finally`. It was bad at knowing when its own evidence didn't fit, and
it stated wrong conclusions with the same confidence as right ones.

I was the opposite. I couldn't have found the dead code. But I knew that a table with no
matching row and a lookup claiming otherwise meant somebody was looking at the wrong
database, and I knew what a duplicate admin role smelled like before we had any evidence.

Neither half would have finished this alone. That's not a slogan, it's just what the log
shows.

<!-- KEN: things I might still add —
     - a line about what the two-day window means for the actual cutover plan
     - whether to name FrontBase openly (I think yes, it's part of why this is unusual)
     - Paul may want a closing paragraph tying back to his intro -->
