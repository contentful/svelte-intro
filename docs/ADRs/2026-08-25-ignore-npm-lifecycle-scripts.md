# Ignore npm lifecycle scripts on install

- **Date:** 2026-08-25 (record written); decision made 2025-11-26
- **Status:** Accepted — in effect on `main`

> This record was written on 2026-08-25 from the commit history. It documents an
> existing decision rather than a new one, and the rationale below is
> reconstructed from the change itself — the original commit message and PR
> title record no reasoning. Where intent could not be established, this record
> says what is true and stops.

## Context

`svelte-intro` is a private, unpublished Svelte + Vite demo app. Its dependency
tree is small — three direct devDependencies (`svelte`,
`vite`, `@sveltejs/vite-plugin-svelte`) — but the transitive tree resolved
through `package-lock.json` is not, and any package in it can declare
`preinstall`, `install`, or `postinstall` scripts that npm executes on the
machine running the install.

By default npm runs those scripts. Nothing in this repository needs them: the
three devDependencies are pure JavaScript, there is no native addon to compile,
and there is no build step wired to a lifecycle hook. The only npm scripts
declared in `package.json` are `dev`, `build`, and `preview`, all invoked
explicitly by a developer.

## Decision

An `.npmrc` at the repository root sets:

```
ignore-scripts=true
```

Evidence: commit
[`36331ef`](https://github.com/contentful/svelte-intro/commit/36331efdfffcde5a5013f44af2cc68049cee3a68)
("chore: [] ignore npm scripts (#5)", 2025-11-26), which created `.npmrc` as a
single-line file. That is the entire diff — one insertion, no other files
touched. The empty `[]` in the subject looks like an unfilled slot in a
templated commit message, which suggests the change arrived as part of a batch
across repositories rather than in response to something specific in this one;
that reading is inference from the message format, not something the history
confirms.

The mechanical effect is what the npm setting does: `npm install` and `npm ci`
resolve and unpack dependencies but do not execute any package's lifecycle
scripts. No package in the current tree requires one, so installs succeed
without them.

## Consequences

- Installs in this repository do not execute third-party install-time code.
  That removes install-time script execution as a path for anything in the
  transitive tree to run on a developer's machine or in a future CI job.
- If a dependency is ever added that genuinely requires a postinstall step —
  a native module, a binary downloader — the install will succeed while leaving
  that dependency unusable, and the failure will surface later and less
  obviously than an install error would. Run the required step explicitly
  rather than removing this setting.
- Do not work around a failing install with `--ignore-scripts=false`. Identify
  the dependency that needs the hook and handle it directly, or open a PR that
  changes this decision on purpose.
- The setting is repository-local. Anyone installing this project's
  dependencies outside a checkout of this repo will not inherit it.
