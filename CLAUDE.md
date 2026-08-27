# Working in this repo

A Jekyll blog on the `minima` theme, served by GitHub Pages from `main`. Two authors, Ken and
Paul. The repository is **public** — assume anything committed here is readable by anyone.

## First thing in a fresh clone

```sh
git config core.hooksPath .githooks
```

Without it the pre-commit hook does not run and the protections below are off.

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
gh api repos/TreasureBoat/the-long-codebase/pages/builds/latest --jq .status
curl -s https://treasureboat.github.io/the-long-codebase/<path> | grep ...
```

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

- The typewriter post says Ken helped modernize "some years back", but the manual attempt was
  spring/summer 2025. Two separate attempts, or does that line need correcting on a live page?
