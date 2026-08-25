# Contributing

`svelte-intro` is a small Svelte + Vite demo app owned by
`@contentful/team-devrel`. It is not published to npm (`package.json` is
`"private": true` at version `0.0.0`) and it is not deployed from this
repository. Changes here are almost always in service of a demo, a workshop, or
keeping the dependencies from rotting.

The repo has been quiet since its first commit in June 2023 — everything after
that is repository plumbing. Please keep changes small and self-contained.

## Prerequisites

- Node.js with npm. No engine range is pinned in `package.json`; a Node version
  new enough for Vite 3 works.
- Nothing else. There are no environment variables, no `.env` file, and no
  Contentful credentials involved — the app renders hardcoded data.

## Getting set up

```bash
npm install       # or npm ci, if you want the lockfile enforced
npm run dev       # Vite dev server with HMR
```

`.npmrc` sets `ignore-scripts=true`, so dependency lifecycle scripts do not run
during install. This is deliberate; see
`docs/ADRs/2026-08-25-ignore-npm-lifecycle-scripts.md`. Do not pass
`--ignore-scripts=false` to work around a failing install — figure out which
dependency needs a postinstall step and run it explicitly instead.

## Verifying a change

There is no test suite and no linter in this repository. The available checks
are:

```bash
npm run build     # vite build — must complete without errors
npm run preview   # serve dist/ and click through the page
```

Because `jsconfig.json` sets `"checkJs": true`, your editor will typecheck the
plain JavaScript and `.svelte` files. Those diagnostics are the closest thing
to a static analysis gate, so install the recommended
`svelte.svelte-vscode` extension (`.vscode/extensions.json`) and clear
warnings in the files you touch.

Please state in your PR what you actually ran and what you saw. Do not claim a
test suite passed — there isn't one.

## Making a change

1. Branch from `main`.
2. Keep the change scoped. If you are adding a component, put it in `src/lib/`
   alongside `Card.svelte` and `Counter.svelte`.
3. Match the existing style: two-space indentation, single quotes in `.js`
   files, no semicolon discipline enforced either way, Svelte 3 idioms
   (`export let` props, `on:click`, `new App({ target })`). Do not introduce
   Svelte 4 or 5 syntax without bumping the `svelte` dependency in the same PR
   and saying so.
4. Write commit subjects as Conventional Commits — `fix:`, `chore:`, `docs:`,
   `feat:` — matching the existing history. There is no semantic-release setup
   in this repo, so the prefix is for readability rather than versioning.
5. Open a PR against `main`. `.github/CODEOWNERS` requests review from
   `@contentful/team-devrel` automatically. No CI runs, so a human reading the
   diff is the entire quality gate — make the diff easy to read.

## Things to leave alone

- `.gitignore` — do not loosen it. `dist/`, `node_modules`, and `*.local` must
  stay ignored.
- `.npmrc` — see the ADR above.
- `README.md`'s "Technical considerations" section is inherited verbatim from
  the `create-vite` template and explains upstream template choices. Rewriting
  it loses that provenance; add repo-specific notes to `ARCHITECTURE.md` or
  `AGENTS.md` instead.

## Known rough edges

If you are looking for something small to fix, these are real and documented in
`AGENTS.md`:

- `src/app.css` is empty; all styling is `@import`ed from `unpkg.com` at
  runtime, so the app is unstyled offline.
- `Counter.svelte` is imported by `App.svelte` but never rendered, and
  `App.svelte`'s `hasVat` import is unused.
- Nothing writes to the `hasVat` store, so VAT is permanently on with no way to
  toggle it.
- The demo product titles are misspelled `"Prodcut"`.
- `catalog-info.yaml` is still the unfilled Backstage template — `description`,
  `type`, `lifecycle`, and `system` are all `unknown`, and the file carries a
  warning comment saying it must be filled in before merging to the default
  branch. It was merged anyway in `63198f5`.
