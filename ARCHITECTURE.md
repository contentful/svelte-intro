# Architecture

`svelte-intro` is a single-page Svelte 3 application built with Vite. There is
no server, no router, no persistence layer, and no external API client — the
entire app is three components and one store rendering hardcoded data.

## Directory layout

```
index.html            Vite HTML entry; loads /src/main.js as a module
vite.config.js        Vite config — @sveltejs/vite-plugin-svelte, no other plugins
jsconfig.json         Editor/TS settings; checkJs enabled, includes src/**
.npmrc                ignore-scripts=true
.vscode/extensions.json  Recommends svelte.svelte-vscode
catalog-info.yaml     Backstage descriptor (still the unfilled template)
public/vite.svg       Static asset served at /vite.svg (the favicon)
src/
  main.js             Mounts App into #app
  app.css             Present but empty (0 bytes)
  vite-env.d.ts       Triple-slash refs for svelte and vite/client types
  assets/svelte.svg   Imported by App.svelte
  App.svelte          Root component: hardcoded product list
  lib/
    Card.svelte       Renders one product, applies VAT
    Counter.svelte    Click counter (imported but not rendered)
    store.js          Exports the `hasVat` writable store
```

## Boot sequence

1. Vite serves `index.html`, which contains `<div id="app">` and a module
   script tag pointing at `/src/main.js`.
2. `src/main.js` imports `./app.css` (currently a no-op — the file is empty),
   imports `App.svelte`, and instantiates it with
   `new App({ target: document.getElementById('app') })`. This is the Svelte 3
   client component API.
3. The instance is exported as the module's default. Nothing consumes that
   export; it exists so the module has a stable entry value.

## Component graph and data flow

`App.svelte` is the only page. It declares a module-local `data` array of three
objects (`title`, `description`, `price`) and renders one `Card` per entry via
`{#each}`, passing each field as a separate prop.

`Card.svelte` receives `title`, `description`, and `price` (defaulting to `0`)
through `export let`. It imports the `hasVat` store from `./store` and calls
`hasVat.subscribe(...)` directly, copying the value into a local
`hasVatValue`. The rendered price is `hasVatValue ? price * 1.2 : price`.

`src/lib/store.js` is the only shared state: a single
`writable(true)` exported as `hasVat`. No code in the repo ever calls
`hasVat.set` or `hasVat.update`, so the store is effectively a compile-time
constant and the 1.2x multiplier always applies. There is no UI control for it.

`Counter.svelte` holds its own `count` and an `increment` handler. `App.svelte`
imports it but does not place it in the template, so it never mounts. It is
leftover scaffolding from the `create-vite` template.

Data flow is therefore one-directional and shallow: `App` → props → `Card`,
plus one read-only store subscription in `Card`. There are no events bubbling
back up (`createEventDispatcher` is not used anywhere).

## Styling

There is no local stylesheet in use. `App.svelte` declares a
`<style global>` block whose entire content is two `@import` rules pointing at
`https://unpkg.com/open-props` and
`https://unpkg.com/open-props/normalize.min.css`. Those are resolved by the
browser at runtime, not bundled.

Consequences worth being aware of:

- The app depends on `unpkg.com` being reachable to look correct. Offline or
  behind a restrictive CSP it renders unstyled.
- `Card.svelte`'s scoped styles reference `var(--font-weight-8)` and
  `var(--font-weight-4)`, which are Open Props custom properties. Without the
  CDN import they resolve to nothing and the browser falls back to defaults.
- `src/app.css` is the conventional place for global styles and is already
  wired up through `main.js`, so it is the natural home for anything that
  should be bundled rather than fetched.

## Build and output

`npm run build` runs `vite build`, which bundles from `index.html` into `dist/`.
`dist/` and `dist-ssr/` are gitignored. `npm run preview` serves that output.
`npm run dev` runs the Vite dev server with HMR.

Note that HMR does not preserve component state — this is a documented
`@sveltejs/vite-plugin-svelte` default and is discussed in `README.md`. State
that must survive a hot update belongs in an external store like
`src/lib/store.js`.

There is no CI in this repository: no `.github/workflows` directory exists, so
nothing builds, lints, or tests on push. `.github/CODEOWNERS` assigns all paths
to `@contentful/team-devrel`, which is the only automated gate on changes.
