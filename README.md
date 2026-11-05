# Astro + Leaflet Map Examples

Four small examples of putting an interactive [Leaflet](https://leafletjs.com/) map on an [Astro](https://astro.build/) page.
They build on each other: **each example changes one thing from the one before it**, so the best way to learn from them is to compare neighboring examples side by side.

## Getting started

```sh
npm install        # install Astro and Leaflet
npm run dev        # start the dev server at http://localhost:4321
```

Before you deploy, also try the production build. Some problems only show up there (see "Things to watch for" below).

```sh
npm run build      # build the site into ./dist/
npm run preview    # serve ./dist/ locally
```

## Where things are

```text
src/
├── pages/            ← one file per page; the filename becomes the URL
│   ├── index.astro                      (list of examples)
│   ├── harvard-yard.astro               (Example 1)
│   ├── new-england-national-parks.astro (Example 2)
│   ├── new-england-national-parks-leaflet-cdn.astro               (Example 3)
│   └── new-england-national-parks-leaflet-cdn-with-variable.astro (Example 4)
├── components/       ← the map code lives here, one component per example
│   ├── MapOnePoint.astro
│   ├── MapNps.astro
│   ├── MapNpsCdn.astro
│   └── MapNpsCdnVariable.astro
├── layouts/
│   └── Layout.astro  ← shared <html>, header, and footer for every page
└── data/
    └── ne-nps.json   ← New England National Park Service sites (name, lat, lng)
```

Each **page** is short: it loads data and passes it to a **component** as props.
The **component** draws the map. Most of the learning is in `src/components/`.

## The key idea: server code vs. browser code

An `.astro` file has two kinds of JavaScript that run in two different places:

| Part of the file | Runs where / when | Can it see the other part's variables? |
| :-- | :-- | :-- |
| Frontmatter (between the `---` lines) | On the server, at **build time** | No |
| `<script>` tag | In the **browser**, when the page loads | No |

Leaflet runs in the browser, but the data (props, JSON) lives in the frontmatter.
**Every example is about getting data from the frontmatter into the browser script.** Watch how each one does it.

## The examples

| # | Page | Component | Leaflet comes from | Data reaches the browser via |
| :-- | :-- | :-- | :-- | :-- |
| 1 | `harvard-yard` | `MapOnePoint` | npm (`import L from "leaflet"`) | `data-*` attributes (one value each) |
| 2 | `new-england-national-parks` | `MapNps` | npm | `data-places` attribute holding a JSON string |
| 3 | `...-leaflet-cdn` | `MapNpsCdn` | CDN `<script>` in `<head>` | `data-places` attribute holding a JSON string |
| 4 | `...-leaflet-cdn-with-variable` | `MapNpsCdnVariable` | CDN `<script>` in `<head>` | `define:vars` (a real JavaScript variable) |

### What stays the same in every example

- A `<div id="map">` with a fixed height (Leaflet maps are invisible without one).
- `L.map(...)` to create the map, then `L.tileLayer(...)` for the OpenStreetMap background.
- `L.marker(...).bindPopup(...)` for each point.
- The shared `Layout.astro`.

### What changes, step by step

**1 → 2: one point → many points.**
Single values (lat, lng, zoom, title) fit easily into `data-*` attributes.
An array doesn't, so `MapNps` turns it into a string with `JSON.stringify` and turns it back with `JSON.parse` in the browser.
It also uses `fitBounds` to zoom so every marker is visible.

**2 → 3: npm → CDN.**
Instead of `import L from "leaflet"` (bundled by Astro), the page loads Leaflet from unpkg.com with a `<script>` tag in `<head>`, which creates a global variable `L`.
Compare `MapNps.astro` and `MapNpsCdn.astro`: only the top of the `<script>` is different.

**3 → 4: data attribute → script variable.**
`define:vars={{ places }}` makes Astro write `const places = [...]` directly into the script, so there's no `JSON.parse`.
The trade-off: a `define:vars` script is **not processed** by Astro, so you can't use TypeScript or `import` from npm in it.
That's why this example only works with the CDN version of Leaflet.

### Tip: compare files directly

```sh
diff src/components/MapNps.astro src/components/MapNpsCdn.astro
diff src/components/MapNpsCdn.astro src/components/MapNpsCdnVariable.astro
```

In VS Code: select two files in the Explorer, right-click, and choose **Compare Selected**.

## Things to watch for

- **Broken marker images after `npm run build` (npm examples only).** Bundling breaks Leaflet's guess about where its marker images are. `MapOnePoint` and `MapNps` include a short fix that imports the images directly; the comments explain it. The CDN examples don't need it.
- **Data in the browser console.** Each component calls `console.log` on its data. Open your browser's developer tools to see exactly what arrived from the server.
- **One map per page.** Every component uses `id="map"`, and an id must be unique on a page.

## Learn more

- [Astro: Scripts and event handling](https://docs.astro.build/en/guides/client-side-scripts/) (including passing data with `data-*` and `define:vars`)
- [Leaflet Quick Start Guide](https://leafletjs.com/examples/quick-start/)

Examples are from [CSCI E-12](https://cs12.net/), Harvard Extension School.
