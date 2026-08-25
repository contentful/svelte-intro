# Decision records

Architectural decision records for `svelte-intro`. One file per decision, named
`YYYY-MM-DD-<kebab-case-title>.md`.

- [2026-08-25 — Ignore npm lifecycle scripts on install](./2026-08-25-ignore-npm-lifecycle-scripts.md):
  `.npmrc` sets `ignore-scripts=true`, so `npm install` and `npm ci` do not run
  dependency lifecycle scripts.
