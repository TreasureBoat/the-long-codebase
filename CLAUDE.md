# Working in this repo

A Jekyll blog on the `minima` theme, served by GitHub Pages from `main`. Two authors, Ken and
Paul. The repository is **public** — assume anything committed here is readable by anyone.

## First thing in a fresh clone

```sh
git config core.hooksPath .githooks
```

Without it the pre-commit hook does not run and the protections below are off.

## Identity

Commit and act as the personal account only. Paul has a second, work-linked GitHub
account with pull-only access to this repo; anything done under it leaves an
employer's name on a public repository that names no employer anywhere on the site.

- Git identity is pinned per-repo, and the pre-commit hook blocks anything else.
- **`gh` is not covered by that.** It acts as whichever account is globally active,
  so check before opening a PR or issue:

```sh
gh auth status          # confirm the active account
gh auth switch --user pyu13821
```

## Hard rules

**1. No HTML comments in posts.** `<!-- KEN: ... -->` notes shipped to the live page source
once already, including one asking whether to name a client, and the build reported success
throughout. Editorial notes go in front matter as YAML `#` comments, which are stripped at
parse time and can never reach a page. The pre-commit hook blocks any `_posts/` file
containing `<!--`.

**2. Draft gaps use double-bracket markers,** never single brackets — a single `[` collides
with markdown link syntax, so a `grep '\['` check returns false positives and stops being
trusted.

```
[[FACT   a number or name to drop in]]
[[ASK    a question whose answer becomes the prose]]
[[DECIDE a choice between options]]
[[NOTE   commentary — delete, never publish]]
```

Find them all with `grep -n '\[\['`. The hook blocks any `_posts/` file still containing one.

**3. Drafts live in `_drafts/`, which is gitignored.** Jekyll does not build drafts, but this
repo is public, so an unfinished post committed here would still be readable on GitHub. Move
a draft into `_posts/` only when it is ready to be seen.

**4. Internal links need the baseurl explicitly.**

```liquid
[text]({{ site.baseurl }}{% post_url 2026-08-27-the-typewriter-argument %})
```

`{% post_url %}` on its own omits the baseurl on the Jekyll version Pages actually runs, which
produced a live 404 once. Never hardcode `/the-long-codebase/...`.

**5. Every post needs `excerpt` in front matter.** Without it Jekyll takes the first
paragraph, which has twice been the wrong thing. `author` must be the **key** from
`_data/authors.yml` (`paul`, `ken`) — not the display name, or the byline silently vanishes.
`cta_hook` is optional, one post-specific sentence above the standing call to action.

**6. American spelling** — skeptic, modernize. It is what the existing text uses.

## The editorial rule

The skepticism is the asset. These posts earn attention by being candid about failure,
including the machine's. Never soften or remove a specific failure to make something read
better. Praise is only allowed attached to a specific incident with a number in it —
"Claude is great at tedious work" is worthless; "it read 84 table definitions against 18
models without getting bored, which I will not" is evidence.

`.claude/agents/post-review.md` reviews drafts against this and nine other dimensions. It is
read-only and proposes rewrites rather than applying them.

## Layout

| Path | What it is |
|---|---|
| `_data/authors.yml` | Bylines and contact addresses. The CTA shows the post author's address. |
| `_includes/cta.html` | End-of-post call to action, rendered by the post layout. |
| `work.md` | The `/work/` services page. |
| `assets/main.scss` | Overrides minima at its own path. |
| `.githooks/pre-commit` | Blocks placeholders and HTML comments in `_posts/`. |

## Verifying changes

**The site cannot be built locally as configured.** The Gemfile pins Jekyll 4.3, which needs
Ruby ≥ 2.7; the machine this was set up on has 2.6.10. Production also ignores that pin —
Pages runs its own Jekyll 3.10. So verify against the live site after pushing rather than
trusting a green build:

```sh
# Wait for the build whose commit matches HEAD. Polling only for status "built"
# returns instantly on the PREVIOUS build and reports a stale success.
HEAD=$(git rev-parse HEAD)
gh api repos/TreasureBoat/the-long-codebase/pages/builds/latest \
  --jq '.status + " " + (.commit // "-")'

# Then bust the CDN. Pages serves with max-age=600, so a plain curl straight
# after a push can hand back a ten-minute-old page and look like a failed deploy.
curl -s "https://treasureboat.github.io/the-long-codebase/<path>?cb=$RANDOM" \
  -H 'Cache-Control: no-cache' | grep ...
```

Both of those have already produced false readings here — once reporting success
against an older build, once showing corrected text as still wrong.

Switching the Gemfile to the `github-pages` gem would make local preview match production.
The line is already in the Gemfile, commented out.

## Open decisions — 27 Aug 2026

**For Ken:**

- **Review `/work/`.** It went live making commercial commitments in your name, including a
  characterisation of your availability ("Ken maintains a framework and the servers it runs
  on… we would rather tell you that than take the work and be slow") and using your country
  enum story as a credential. Change anything you do not stand behind.
- **Do you want this repo private?** Your org, your call. Note that on GitHub Free for
  organisations, Pages does not serve private repos — going private would take the site down
  unless the org moves to Team first. Also: the editorial notes stripped from the posts are
  still readable in git history.
- **Enable the hook** (the config line at the top).

**For Paul:**

- **The modernization was not one attempt, it was three summers,** and it is still going.
  Ken's account: summer 2024, the database structure was reworked onto BaseModel; summer
  2025, the UI moved onto Keen; summer 2026, Java 10 to 21 plus bug fixes. It runs on that
  cadence because it is a school application — changes can only land during the summer
  holiday — and because it is a side project for both of you on top of other work.
  So "we did not finish" may be the wrong frame: nothing failed, it is a constrained
  cadence that is still running. Your post, your call, but the facts are now available.

## Settled — 28 Aug 2026

- **The "two years" figure was wrong and is corrected.** Paul's push began around November
  2025; he built the plugin prototype at the start of December 2025 and sent screenshots
  over Discord; Ken bought his IntelliJ licences and started on the free Claude plan that
  same month. So the gap between first pitch and first use was about a month, and the whole
  story is nine months old, not two years. It had reached `/work/` as "Ken refused for two
  years", which was a factual overstatement on a page selling services.
- **"A person who does not write code" is gone from the country-enum post.** It contradicted
  Paul's own post on the same site. What was actually remarkable is stated instead: he had
  never written an IntelliJ plugin, does not know the framework internals, and did it in the
  evenings alongside a full-time job.
- **The repo stays public and single.** The controls added on 27 Aug — the pre-commit hook,
  notes in front matter, `_drafts/` gitignored — address the cause. A private drafting repo
  with copy-out to a public one is process that will be bypassed the first busy week, and it
  would take the site down on GitHub Free for organisations. The standing rule is the one at
  the top of this file: assume anything committed here is public.
