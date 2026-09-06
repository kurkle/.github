# Contributing

Thanks for taking the time to contribute. This file applies as a default across the
`kurkle` Chart.js ecosystem repositories (chart types, plugins, `@kurkle/color`,
`@kurkle/configs`, `@kurkle/astro-chartjs-editor`) wherever a repository doesn't
already have its own `CONTRIBUTING.md`.

## Reporting a reproducible bug

Several of these repositories ship a documentation site built with
[`@kurkle/astro-chartjs-editor`](https://github.com/kurkle/astro-chartjs-editor), where
every sample chart on the site is **editable directly in the browser**. If you're
reporting a bug in a chart type or plugin, please:

1. Open the closest matching sample on the documentation site.
2. Edit it in place until it reproduces the problem.
3. Paste the resulting code into the issue.

This is by far the fastest way to get a bug looked at, and it avoids the single most
common problem with reports filed against these repos: an issue that can't be
reproduced from the description alone.

## Commit messages and what they release

Releases are automated with [`semantic-release`](https://github.com/semantic-release/semantic-release)
using its default configuration — no custom preset — which means the **Angular commit
convention** decides what happens on every merge to `main`:

| Prefix | Release |
| --- | --- |
| `fix:` | patch |
| `feat:` | minor |
| `perf:` | patch |
| `BREAKING CHANGE:` (in the body/footer) | major |
| `docs:`, `test:`, `ci:`, `chore:`, `refactor:`, `build:`, `style:` | no release |

**The scope doesn't change this.** `fix(ci): ...` releases a patch exactly like a bare
`fix: ...` would — the parenthesised scope has no special meaning to the release
tooling, it's only there for readability.

Because of this, please only use `fix`, `feat` or `perf` when the change actually
touches what gets published. In the chart and plugin packages, `package.json`'s
`files` field is `["dist/*", "!dist/docs/**"]` — the documentation site is explicitly
excluded from the published package. A documentation-only change is a `docs:` commit,
even if it happens to live in the same repository as the published code.

### Why both the PR title and the commit message matter

When a pull request is merged, GitHub decides what becomes the commit message on
`main` based on how it's merged:

- **Squash merge of a multi-commit PR** → the **PR title** becomes the commit message.
  The individual commits inside the PR are discarded.
- **Squash merge of a single-commit PR** → that one commit's own message is kept,
  and the PR title is ignored.
- **Merge commit** → every commit on the branch lands on `main` individually, and
  `semantic-release`'s commit analyzer reads each one separately.

In other words, depending on how a PR is merged, either the title or the commit
messages (or both) end up deciding the release — and if either one is wrong, an
unintended release goes out. This has already caused six unnecessary releases across
this fleet, usually a `docs:`-only PR titled or committed as `fix:` or `feat:` by
habit. Please make sure **both** are correct before merging, not just one.

## Development environment

Git hooks are installed with:

```bash
npx kurkle-install-hooks
```

(from [`@kurkle/configs`](https://github.com/kurkle/configs)). It copies `commit-msg`
and `pre-commit` into `.githooks/` and points `core.hooksPath` at them; `npm run
prepare` runs `git config core.hooksPath .githooks` so a fresh clone picks them up
automatically after `npm install`.

The `pre-commit` hook runs, in order, whichever of these scripts a repository defines
(each is invoked with `--if-present`, so it's a no-op where the script doesn't exist):

```
lint → test → typecheck → build → docs
```

The hook is a fast local smoke test, not the gate — it's there to catch obvious
problems before they leave your machine. The real gate is CI on the pull request,
which checks the actual merge result.

Commit messages are checked by the `commit-msg` hook against the same Angular-style
prefixes listed above.

### Browser tests

Tests run under [Karma](https://karma-runner.github.io/) and require **both Chrome and
Firefox** to be installed locally — the shared Karma config launches both browsers for
a test run.

### Documentation sites

Three repositories (the chart types) build their documentation with
[Astro](https://astro.build/) and [Starlight](https://starlight.astro.build/). Samples
live in `src/content/docs/samples/*.md` (or `.mdx`) files, inside fenced code blocks
tagged ` ```js chart-editor `. These blocks are picked up and rendered as interactive,
editable charts in the browser by `@kurkle/astro-chartjs-editor`.

Preview the documentation site locally with:

```bash
npm run docs:dev
```

## Pull requests

- Keep PRs focused on one change.
- Make sure the PR title (or, for single-commit PRs, the commit message) follows the
  commit convention above — it's what ends up in the changelog and decides the release.
