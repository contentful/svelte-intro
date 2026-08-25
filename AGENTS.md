# AGENTS.md

Guidance for coding agents working in `contentful/svelte-intro`.

## What this repo is

A minimal Svelte 3 + Vite demo application. It was scaffolded from the
`create-vite` `svelte` (JavaScript) template — `README.md` is still largely the
template's own text — and then extended with a small hardcoded product-card
example.

It is a teaching/intro app, not a maintained product:

- `package.json` is `"private": true` at version `0.0.0`. Nothing is published.
- There is no test suite, no linter, and no `.github/workflows` directory.
- The only substantive application commit is `fa76730` ("First commit",
  2023-06-20). Every commit since then touches `catalog-info.yaml`,
  `.github/CODEOWNERS`, or `.npmrc` — repository plumbing, not features.
- Despite living in the `contentful` org, the app does not use any Contentful
  SDK and makes no network calls to Contentful. Do not add a Contentful
  integration unless a ticket explicitly asks for one.

Owner: `@contentful/team-devrel` (`.github/CODEOWNERS`).

## Commands

These are the only scripts defined in `package.json`:

```bash
npm run dev      # vite        — dev server with HMR
npm run build    # vite build  — production build into dist/
npm run preview  # vite preview — serve the built dist/ locally
```

There is no `npm test` and no `npm run lint`. Do not invent one in a report;
if a task needs verification, run `npm run build` and check it completes.

`.npmrc` sets `ignore-scripts=true`, so `npm install` / `npm ci` will not run
dependency lifecycle scripts. Leave that setting alone — see
`docs/ADRs/2026-08-25-ignore-npm-lifecycle-scripts.md`.

## Where things live

- `index.html` — Vite's HTML entry point; loads `/src/main.js`.
- `src/main.js` — mounts `App.svelte` into `#app`.
- `src/App.svelte` — the whole page: a hardcoded `data` array and a `Card` per item.
- `src/lib/` — `Card.svelte`, `Counter.svelte`, `store.js`.
- `vite.config.js` — Vite config; the Svelte plugin is the only plugin.
- `jsconfig.json` — `checkJs: true`, so editors typecheck the plain JS and
  `.svelte` files. Type errors surface in the editor, not in a build gate.

See `ARCHITECTURE.md` for how these fit together.

## Gotchas worth knowing before you edit

- `src/app.css` exists but is empty (0 bytes). All styling comes from the
  `@import` statements in `App.svelte`'s `<style global>` block, which pull
  `open-props` from `unpkg.com` at runtime. The app therefore looks unstyled
  offline, and `Card.svelte`'s `var(--font-weight-8)` / `var(--font-weight-4)`
  resolve to nothing without that CDN.
- `Counter.svelte` is imported by `App.svelte` but never rendered. So is the
  `hasVat` store import in `App.svelte` — only `Card.svelte` actually reads it.
- Nothing ever writes to the `hasVat` store, so the 1.2x VAT multiplier in
  `Card.svelte` is always applied. There is no UI to toggle it.
- The demo product titles read `"Prodcut 1"`, `"Prodcut 2"`, `"Prodcut 3"` —
  a typo in the original commit, not a fixture other code depends on.
- `catalog-info.yaml` is still the unfilled Backstage template: `description`,
  `type`, `lifecycle`, and `system` are all `unknown`, and the tags include
  `update-me` and `tier-unknown`. Treat its metadata as unreliable.

## Conventions

- Commit subjects follow Conventional Commits (`fix:`, `chore:`, `docs:`), as
  seen in `git log`. There is no release automation configured in this repo, so
  the prefix is a readability convention only.
- Svelte 3 syntax throughout (`new App({ target })` in `main.js`,
  `on:click`, `export let` props). This is not Svelte 4 or 5 — check the
  `svelte` version in `package.json` before applying newer idioms.
