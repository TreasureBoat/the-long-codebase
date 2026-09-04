---
layout: post
title: "Finished, and thrown away"
subtitle: "We rewrote the whole AJAX engine to remove jQuery. It worked. Then we found the UI skin ships its own copy of jQuery, and the goal had been unreachable from the beginning"
date: 2026-09-04 09:00:00 +0900
author: ken
description: "A migration that succeeded technically and was abandoned anyway, because the thing it was trying to achieve turned out to be impossible for reasons that had nothing to do with the code we wrote."
excerpt: >-
  In one day we rewrote every verb of our AJAX engine to drop jQuery. It worked. Three days
  later a real application broke, and a week after that we found the UI skin bundles its own
  copy of jQuery — so "jQuery-free" had never been available at any price.
cta_hook: "If you are partway through a migration and quietly unsure it was a good idea, this one is for you."
---

We removed jQuery from our AJAX engine in a single day. Every verb, the serialization, the
autocomplete widget rewritten from scratch, a third-party plugin deleted outright. It worked.

Three days later it broke a real application. A week after that we abandoned the whole idea —
not because the rewrite was bad, but because the goal it was aiming at had never been
achievable, for reasons that had nothing to do with any code we wrote.

I think that is a more useful story than a migration that went well.

## What we were doing and why

The framework has an in-session AJAX engine — the thing that updates part of a page without
reloading it. It was built on jQuery, as everything of its generation was. jQuery is not a
problem exactly, but it is a large dependency carried for a shrinking set of reasons, and the
browser APIs it originally papered over have been standard for years.

So: rewrite it in plain JavaScript. Well-understood work, clear finish line.

It took a day. Five verbs converted, each element's emitted JavaScript rewritten, the
autocomplete widget rebuilt from nothing so a plugin could be deleted, and a property to switch
jQuery loading off entirely so the result could be tested honestly.

That speed matters to the rest of this story, and not in the way I expected.

## The break

Three days later, testing an application on a new Java version, editors stopped working —
markdown editor, date-time picker, rich text — but *only* inside AJAX-loaded overlays. On an
ordinary page they were fine. Open the same component in an overlay and the field was inert.

I found it with a manual bisect across sixty commits. That is not a clever technique, it is an
afternoon of `git checkout` and clicking, and it was the only thing that would have worked
because the failure was silent: no console error, no exception, no failed request. The editor
simply never initialized, which looks identical to an editor that has not been configured.

The cause is worth knowing if you ever replace jQuery's DOM insertion with your own.

When jQuery injects HTML, it does something particular with `<script>` tags: it pulls them out,
loads external ones **in order**, and runs inline ones **after** those have loaded. Our
replacement set `innerHTML` and then re-ran the scripts it found. Same idea, different
semantics — external scripts started loading asynchronously while the inline initializers ran
immediately. So the initializer for a plugin ran before the plugin itself existed. On a normal
page load the browser's own parser handles the ordering and everything is fine. Only inside an
AJAX-injected fragment does the difference appear.

That is not a bug in the sense of a mistake. It is a behavior of jQuery that nobody had written
down as a requirement, because nobody knew it was one until it was gone.

## The fix, and then the real question

The fix was small: put jQuery back for *content injection only* — `.html()` for one case,
`.load()` for the other — and keep every other piece of the vanilla rewrite. Shipped as a patch
release.

That left the interesting question. If we were going to keep jQuery for injection, could we at
least get rid of it everywhere else and stop loading it on the pages that did not need it? So I
flipped the switch we had built, turned jQuery loading off, and looked at what happened.

The console filled with `Can't find variable: jQuery`.

Not from our code. From the **UI skin** — the thing that draws every page in the application.
Its bundled JavaScript contains **its own copy of jQuery**, version 3.4.1, compiled in. The
skin is not a jQuery *user*; it is a jQuery *distributor*.

Which meant the situation was this: turn our jQuery off and the skin's copy is still there, so
nothing is saved. Leave ours on and there are now two jQuery instances loading in an
unpredictable order, which produces exactly the race the console was complaining about. There
was no configuration in which the application ran jQuery-free. There never had been.

## The part that is actually about how we work now

Here is what I keep thinking about.

The rewrite took a day. It took a day because I was working with a tool that is extremely good
at mechanical, well-specified conversion — five verbs, a known target, obvious tests. That is
the best case for it and it performed exactly as advertised.

And because it took a day, **nobody asked whether the goal was reachable.**

If that rewrite had been a two-week job, the first thing I would have done is spend an hour
working out what actually loads jQuery in a rendered page, because two weeks is expensive
enough to justify checking. One day is not. One day is cheap enough to just do it and see.

The check I skipped would have taken about ten minutes: open the skin's bundle and search for
`jQuery`. That is it. That is the whole investigation that would have prevented all of it.

I do not think the answer is "be more careful", which is the sort of resolution that never
survives contact with a busy week. I think the answer is that **cheap execution changes which
questions get asked**, and the feasibility question is the one that quietly stops getting asked
first — because asking it now costs a meaningful fraction of just doing the work. When building
was expensive, the estimate itself forced the question. Now nothing does.

That is a genuinely new failure mode, and it is not the one people warn you about. The warnings
are all about the machine writing bad code. This code was fine.

## What we kept

We did not revert it. The outcome is a split, decided by what is actually on the page:

- **Public, session-less pages** — no skin, no plugins, nothing bundling its own copy — use the
  vanilla engine. That is where the dependency was genuinely removable, and there it is gone.
- **In-session pages behind the skin** stay on jQuery, permanently and deliberately. Not as
  technical debt to be paid off later. As the correct answer, because the skin ships jQuery and
  fighting that gains nothing and costs a load-order race.

The vanilla injection code is still in the tree, unused, as the starting point if the skin is
ever replaced. It cost a day. Leaving it there costs nothing.

## The uncomfortable version

The honest summary is that we did a piece of work competently and it turned out to have been
pointless, and we found out by breaking a production application under deploy pressure.

The recoverable part is that abandoning it was cheap, and that we narrowed the scope instead of
choosing between "finish it" and "revert it". Migrations get framed as binary far too often. The
useful question at the end was not *did this work*, it was *where does this work* — and the
answer turned out to be a real, defensible subset rather than nothing.

But I would still rather have spent the ten minutes.
