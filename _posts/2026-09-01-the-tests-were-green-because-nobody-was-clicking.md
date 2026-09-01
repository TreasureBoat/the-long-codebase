---
layout: post
title: "The tests were green because nobody was clicking"
subtitle: "My suite passed 113 of 115 by clicking buttons nobody could reach, and one class was green 22 of 22 while saving empty transcripts. A pass rate cannot tell that from real coverage, and I have been reporting pass rates for years."
date: 2026-09-01 10:00:00 -0400
author: paul
excerpt: >-
  A small shop can never buy full regression coverage, so we went fifteen years without it. Now the
  tests get written — and most of the suite turned out to click with executeScript instead of the
  mouse, passing on controls nobody can reach. Pass rate said it was fine, because pass rate cannot
  say anything else. What catches it is where the assertion points.
cta_hook: "If you are scaling test writing with an agent and reporting it by pass rate, the audit in this post is the part worth stealing."
#
# Editorial notes — front matter only, never HTML comments.
#
#   Standalone technical vignette, program-manager perspective. Not a series entry.
#   Deliberately generic: no district, client or production record identifiers.
---

Functional regression testing is not a controversial idea. It is on every list of what a
competent shop does, next to code review and a pipeline that will not deploy a red build. I have
been the person writing those lists for other people's programs.

It is also the first thing a small shop gives up, and we gave it up for fifteen years. Not
because anyone disagreed. Because the arithmetic never worked. A browser suite covering a product
with 165 screens is months of work that ships no features, and it does not stay written — every
screen change breaks a selector, so the maintenance is permanent. On a product that is somebody's
second job, that is not a trade-off, it is a non-starter. So we tested by hand, in the places we
remembered to look, and shipped on the strength of knowing the code.

What changed in the last nine months is that the writing got cheap. Not free, and not automatic,
but cheap enough that a suite covering the real workflows is now a thing I can ask for on a
Tuesday and have by Thursday. That is the first time in fifteen years the arithmetic has
worked.

So I set out to close the gaps, which were not subtle. One feature with 9,082 records in
production had zero tests and not one test hook in its markup. Nine reports are offered to users
and not one was exercised, including the one that hung for 72.5 seconds in production earlier this
year. An evaluation is five different things depending on who performs it on whom. One of the
five was tested as itself. The end-to-end test covers the two highest-usage types without ever
asserting which of them it is in, so a change that broke one and left the other working would
pass.

The plan was to add coverage. The first thing I actually did was audit the 115 tests I already
had, because adding to a suite means trusting it, and I had never checked what its passes were
worth.

They were worth very little. Most of the suite had never clicked anything.

Ken made the same argument from the other end of the stack a week ago: five of the six bugs in a
database cutover were [silent, work that did not happen reported as if it had]({{ site.baseurl }}{% post_url 2026-08-27-six-bugs-that-never-said-a-word %}).
His were in a conversion script. Mine were in the tests that were supposed to catch mine.

The tests were written by a Claude session in January and were good, sincere work against the
instruction I gave. The audit that found the problem was a Claude session in August, against a
better one. The difference between those two instructions is the whole post, and no pass rate could have
told them apart.

## The clicks that were not clicks

The suite drove the browser like this, 75 times across five test classes, 62 of them in one
class alone:

```java
((JavascriptExecutor) driver).executeScript("arguments[0].click();", element);
```

That line fires the element's click handler. It does not click. Everything the browser checks
between a person and a button is skipped:

- whether the element is **covered** by a modal, a sticky header, or the sidebar
- whether it is **disabled**
- `pointer-events: none`, `visibility: hidden`, zero size
- whether it is **scrolled out of the viewport** entirely

So a JavaScript click succeeds against a control nobody can reach. The handler runs, the server
responds, the assertion passes, and the feature is broken for every actual user. It is also the
reasonable thing to reach for when a click is flaky, which is exactly why it spreads. Every one of
those 75 calls was added to make a red test go green, and every one of them worked.

Replacing them is not subtle: a helper that scrolls to the element, waits for it to be genuinely
clickable, and then clicks it with the mouse. Scrolling stays, because a real user scrolls too and
Selenium's own scrolling is unreliable with our sticky header. Only the click had to become real.

**Result: 2 failures became 9, across 115 tests.** Every new failure was a control a user cannot
reach. Three distinct causes:

```
<input> displayed=false, BLOCKED BY: SPAN
    A hidden radio behind its styled span. The framework's radio matrix renders
    <label class="option" for="..."><input type="radio"></label> and the CSS hides
    the input. Users click the label. That assertion had been worth nothing.

<a> displayed=false, BLOCKED BY: DIV.printHide.aside-primary...
    A create link, covered by the sidebar.

<a> displayed=true, BLOCKED BY: DIV#...Sign.modal.fade.show
    Logout blocked by a signing modal that was still open.
```

The third is the one to look at twice. Nothing failed when that modal failed to close. Every step
after it ran against a page with an invisible overlay across it, and the JavaScript clicks went
through regardless. The old code waited for the modal to close with `Thread.sleep(2000)`,
which waits without ever checking, and located its submit button with a selector loose enough to
pick Cancel, or a submit button elsewhere on the page entirely.

For a while I took that modal as the explanation for two signing tests that had been failing
since January while the database showed the signatures landing. It was the wrong answer, and the
right one is in the next section. The modal was a symptom: the sign component only closes it after
the save succeeds, so a modal that stays up is the server saying no. The old code slept two
seconds and clicked past it, throwing away the reason.

The message it throws now says what a stuck modal actually means:

```
Modal 'PDPlanSign' did not close, which means the server REJECTED the submission.
  app says: Components Addressed must be selected for all goals before signing
  The action reached the Java class and failed validation. Fix the cause; do not
  retry or dismiss the modal to get past it.
```

The last sentence is there because the fastest way to make that test green is to dismiss the
modal, and it would have worked.

The helper now reports what is covering an element when a click fails, using `elementFromPoint`.
"Element not interactable" on its own tells you nothing, and the thing on top is usually the actual
bug.

## The typing that was not typing

The same shape had already played out in text, and it is worth the detour because I shipped the
wrong fix for it first.

`ObservationNavigationTest` was passing 22 of 22 while every classroom observation it created had
an **empty transcript column** in the database. Three silent skips in a row, each logging something
that read like success:

1. The Transcript tab locator matched only `<a>` text. The label sits inside an `<li>`. Not found,
   so the test logged `Skipping Transcript tab click` and moved on.
2. With no tab clicked there was no editor on screen, so: `No TinyMCE iframe found... Skipping.`
3. The alignment step then reported it had `found component references (2a, 3a)` on a page where
   nothing had been typed. It was almost certainly matching the on-screen help text, which uses 2a
   and 3a as its examples.

Both skips are now hard failures, and the failure message names the consequence, so that nobody
restores them for tidiness.

The typing itself was broken for a separate reason, and the first fix offered to me was
`tinymce.setContent()`. Set the editor's content through its API, save, assert it persisted. It
works. The column fills. The test goes green for real.

I asked whether that misses what the UI actually does.

It does. Measured on the same editor in one pass:

```
sendKeys to the #tinymce body        FAILS  -- editor model unchanged
click to focus, then sendKeys        WORKS
Actions click + sendKeys             WORKS
```

**A single missing click was the entire problem.** `setContent` would have proven the editor
initializes and the save path persists, while never showing that a person typing into the box
produces a saved record. Keyboard shortcuts, paste handling, and our paste cleanup all live on the
path `setContent` skips. It would have persisted, passed, and buried a fixable harness bug under a
permanent workaround.

I want to be precise about how that question came up, because it is the transferable part and it
is not a virtue of mine.

I did not review a diff. I was clicking through permission prompts — the approvals an agent asks
for before it runs something — and I noticed the calls going into the editor were API calls. Not
clicks, not keystrokes. That is the only reason I asked. If I had been approving those prompts the
way people mostly approve prompts, or had them turned off, there would have been nothing to
notice: the tests passed, the columns filled, the diff was small and plausible, and `setContent`
had already gone into all five editor sites. It sat there for two days looking exactly like a fix.

The approval prompt turned out to be the highest-signal review surface in the whole loop, and it is
the one everybody optimizes away first. It shows the mechanism. The diff shows the result, the test
report shows the result, and the result was fine — that was the problem.

The reply when I pushed was immediate agreement: yes, you are right, that is using the API instead
of actual user clicks. I have learned not to take that for much. Agreement arrives just as fast
when the pushback is wrong, so on its own it is a response to pressure rather than to evidence.
What settled it was the measurement that followed — three input methods, same editor, one run,
under a minute — which is something I could not have produced as quickly and which happened to
prove me right. The question was worth nothing without the measurement, and the measurement would
never have been run without the question.

## The one that was not a test bug

With both fixed, the suite started failing at something that was not its fault.

One of our workflows has a grid of rows, each with a free-text field and a multi-select beside
it. The document cannot be signed until every row has something selected in that multi-select.
The tests kept failing to sign, and every document the suite created had exactly one row with an
empty selection, and not always the same row. Selections per row, three rows each:

```
document A   3 / 3 / 0
document B   3 / 3 / 0
document C   0 / 3 / 3
```

The first fix I was offered made the test re-read the page and confirm the selection was there.
It passed every time. The database still showed the row empty, which is the same lesson twice in
one week: the DOM is the test's own output, and asserting on it proves nothing about what was
saved.

The selection was made in the browser and never reached the server. The reason is in the template:
the text field sits in an in-place editor with a real save action, and the multi-select sits in a
**separate table cell outside it**, with no save action, no change handler, and no observer. It is
bound straight to the row's object and nothing submits it.

That is also why it was exactly one row and never all three. The test picked the selection for
each row after saving that row, so every row's selection rode along on the next row's save, and
the last one had nothing behind it.

So those selections persist only when something else submits the enclosing form, and the only
submit in the row is the text editor's save. **A user who makes a selection and then does anything
other than edit-and-save the text beside it loses it silently.** No warning, no dirty state. They
then cannot sign, and the app tells them to fill in the thing they believe they just filled in.

That has been in production, and it is the kind of defect a hand-tested release will never catch,
because the person testing knows to save before they navigate. It was sitting behind 75 clicks
that did not care whether the form had submitted.

The first fix was an explicit Save button for the grid, so the selection has a deliberate commit.
That gives users a way to save. It does not stop them abandoning an unsaved one, and the obvious
second half — a warning on navigate-away — would have done nothing, because the way people lose
the selection is moving from the Goals tab to the Sign tab, and that tab panel is AJAX. No unload
event fires. A `beforeunload` handler would have sat there silent for the exact case that produced
the bug.

So I auto-saved the selection on change instead, and broke something worse. The observer I added
pointed at the whole goal grid, so every auto-save repainted all three rows — and a save firing
while the next goal's editor was opening destroyed that editor. Goal text came back 118 / 0 / 118
characters, against 118 / 118 / 118 on every run before I touched it.

I had fixed a silent data loss by shipping a silent data loss, into the same grid, one day later.
Nothing failed. The run log is the only thing that showed it:

```
--- Editing Goal 0 ---   Found 1 TinyMCE iframes   saved
--- Editing Goal 1 ---   Found 0 TinyMCE iframes   DESTROYED
--- Editing Goal 2 ---   Found 1 TinyMCE iframes   saved
```

The bug was granularity, not timing. The observer now points at a small save-status container
instead of the grid, and that has not been verified against a run yet.

## We are what we measure

Years ago I was on a project next to one where the manager was assessed on unit test pass rate and
coverage. The story I got, secondhand and never verified, was that his team wrote tests asserting
`1 == 1`. Hundred percent passing, hundred percent covered, every build green, bonus paid. Nobody
lied about anything. The tests genuinely passed.

I cannot vouch for any of it, and I am repeating it because I no longer need to. I have the
firsthand version now, in my own suite, and it does not require anyone to have been cynical.

My January suite is that story with the incentive removed. There was no bonus. Nothing was being
gamed. A capable agent was told "make these tests pass" and did exactly that, competently, 75
times, and every one of those JavaScript clicks was a correct answer to the question I asked. You
can fix the `1 == 1` case by changing what the manager is paid on. There is no equivalent lever
here. The specification is the only thing in the loop.

And pass rate cannot separate any of them. A test asserting `1 == 1`, a test clicking a control
nobody can reach, and a test driving a real browser all emit the same signal. That is not a
threshold problem. It is what the measurement is, and it is why the number improves while the
product does not.

**What separates them is two things, and a pass rate reports neither: whether a real input path
produced the change, and whether the assertion reads state the test could not have written
itself.** Neither is sufficient alone, and this post has a counterexample to each.

- `ObservationNavigationTest` ran 22 of 22 green and the transcript column in the database was
  empty. Right assertion target, no real input — the keystrokes never reached the editor.
- The goal grid's post-condition re-read the page and confirmed the selection every time. The
  database showed the row empty every time. Real input, wrong assertion target.
- And `setContent` would have satisfied the persisted-state check perfectly while bypassing the
  input path entirely, which is exactly why I nearly kept it.

A test that drives the DOM and then asserts on the DOM is grading its own work. A test that
reaches past the UI to set up the state it then verifies is grading a different application. You
need both ends real, and the pair is countable: for any suite, how many tests both enter data the
way a person does and assert on something the server wrote.

Those are the two numbers I am moving to. They are worse than pass rate in every way that makes a
metric popular: harder to compute, never 100%, and they go down when someone adds a convenience
helper. Those are the properties that make them worth reporting.

For fifteen years my constraint here was capacity — I knew what good coverage looked like and
could not pay for it. That constraint is gone, and what replaced it is that I can generate more
test code in a week than I can meaningfully read. A suite that is 90% honest is indistinguishable
from one that is 100% honest by every signal a dashboard gives me. Both are green.

So the job changed. It is now writing acceptance criteria that name the mechanism rather than the
outcome. "The transcript persists" is satisfied by an API call. "A person typing into the editor
produces a saved record" is not. The second is barely longer and it is the entire difference.
Every qualification of that kind I have added since came from catching a specific wrong answer, not
from foresight. Which is the argument for reading the approval prompts instead of clearing them:
they are the only place in the loop that shows you the mechanism rather than the result, and the
result is not the thing that goes wrong.

Three hygiene checks, none of them clever, that I would run against any browser suite before
quoting a number from it:

- **Grep for `executeScript` with `click` in it.** Every hit is a control that may be unreachable.
  Mine had 75.
- **Grep for `sleep`.** Mine hid an overlay that stayed open for the rest of the run.
- **Check what your text-entry helper does.** If it sets values through an API instead of typing,
  it is testing your save path and calling it a UI test.

Ten minutes of that is now the expensive half of the job rather than the cheap one. That inversion
is worth more attention than the tooling.

One last number, because it is the one I got wrong for months. Before this work the suite ran 115
tests with 2 failures, and I would have called that healthy. Two permanent known failures is not
healthy — it is a suite people have stopped reading, which is how the other 113 got away with
whatever they were doing.

Making the clicks real took it to 9 failures: every one a control a user cannot reach, two of the
three causes the test's fault and one the product's. One closed within the hour, as soon as the
test clicked the label instead of the hidden radio it was styled behind. The rest are on a list
with dates, ahead of the new coverage rather than behind it.

That count will be wrong by the time you read it, and I am not going to pretend otherwise. This is
a report from the middle of the work, not from the end of it — I do not have a finished method, a
clean suite, or a number I would defend in a quarterly review. What I have is the shape of the
mistake, which arrived twice in one week in two different forms and will arrive again in a third.

A worse number and a better suite, and no dashboard I own will ever say so.
