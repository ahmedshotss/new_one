# Driftline — Summer Collection (static site)

A dependency-free static site: plain HTML/CSS/JS, no build step. Seven
products, rendered from a single data file, with a product-detail modal
and a "buy 2 get 3rd free" promo calculator.

## Run locally

No build tooling required. Pick one:

```bash
# Python (built in on most machines)
cd summer-collection-site
python3 -m http.server 8000
# open http://localhost:8000

# Node, if you have it
npx serve .
```

You can also just double-click `index.html` — everything works from
`file://` except that some browsers block `loading="lazy"` image
prefetching slightly differently; a local server is closer to production.

## Project structure

```
summer-collection-site/
├── index.html            Home: hero, product grid, promo calculator, about/contact
├── css/
│   └── styles.css        All styles — design tokens at the top of the file
├── js/
│   ├── products.js       Product data — the only file you edit to change the catalog
│   └── main.js           Renders the grid + modal, runs the promo calculator, mobile nav
├── assets/
│   └── products/         Product photography (see naming convention below)
└── README.md
```

## Editing products

Everything about a product lives in one object in `js/products.js`:

```js
{
  id: "tidewave-runner",        // stable, URL/DOM-safe slug — used as the modal key
  name: "Tidewave Runner",
  price: 100,                   // number, USD — formatted as $100 by main.js
  image: "assets/products/shoe-01-tidewave-runner.webp",
  alt: "Navy knit sneaker with a textured mesh upper and a chevron-tread outsole",
  tagline: "Everyday knit, built for miles.",
  description: "…",
  details: ["Breathable knit mesh upper", "…"]
}
```

To add, remove, or reorder products, edit this array — the grid, the
modal, and the promo calculator's dropdowns all render from it
automatically. No other file needs to change.

**Placeholder prices:** the 7 prices currently in `products.js` are the
ones supplied ($100 / $300 / $150 / $400 / $250 / $500 / $300). Replace
the `price` field per item when final numbers are set — nothing else
depends on the values being final.

## Asset organization & naming

Images live in `assets/products/`, one file per product, named:

```
shoe-<two-digit-index>-<product-slug>.webp
```

e.g. `shoe-01-tidewave-runner.webp`. The index keeps files sorted in a
directory listing in catalog order; the slug matches the product's `id`
in `products.js` so the mapping is obvious at a glance.

**Optimization applied:** source photos were flattened to RGB, resized to
a max width of 1000px (the grid never renders them larger than ~360px,
and the modal never larger than ~440px, so 1000px covers retina
displays with headroom), and re-encoded as WebP at quality 82 — this cut
file sizes by roughly 90–95% with no visible quality loss. If you add
new product photos:

- Export at ≤1000px on the long edge.
- Use WebP (fall back to optimized JPEG only if a tool in your pipeline
  doesn't support WebP).
- Keep a plain, light, or white background so items read consistently
  in the grid (the grid frames each photo on a white card regardless of
  the source background).
- Write a real `alt` describing the shoe's color and type — not the
  filename — for screen reader users and SEO.

## Styling approach

Plain CSS with a small token system at the top of `styles.css` (colors,
fonts, spacing, radius) — no framework. Two breakpoints:

- **1024px** — grid drops from 3 to 2 columns, promo panel and about
  section stack to one column.
- **720px** — grid drops to 1 column, nav collapses behind the menu
  button, section padding tightens for mobile.

Fonts are loaded from Google Fonts (Big Shoulders Display for headings,
Work Sans for body copy, IBM Plex Mono for prices/labels) — swap the
`<link>` in `index.html` and the `--font-*` variables in `styles.css` to
change the type system.

## Accessibility notes

- Semantic landmarks: `<header>`, `<nav>`, `<main>`, `<footer>`.
- Skip-to-content link, visible on keyboard focus.
- Every product image has a descriptive `alt`.
- Product cards are real `<button>` elements — reachable and
  operable by keyboard, not `<div onclick>`.
- The product modal traps Escape-to-close, moves focus to its close
  button on open, and returns focus to the triggering card on close.
- Focus states use a visible outline everywhere (`:focus-visible`),
  not just on hover.
- `prefers-reduced-motion` disables transitions/smooth-scroll for users
  who've asked for it.

## The promo: "buy 2, get the 3rd free"

There's no cart, so the promo is demonstrated two ways:

1. **Ribbon banner** — always visible under the header, states the offer.
2. **Promo calculator** (`#promo` section) — three dropdowns let a
   visitor pick any three products; `main.js` sorts the three prices and
   subtracts the lowest one from the subtotal, live. This is the exact
   rule a real cart/checkout would apply — the calculator is that logic
   extracted into a static-site-friendly demo, not a different rule.

To wire this into a real cart later, reuse `updatePromoTotal()`'s logic
(sort cart items by price, subtract the minimum for every group of 3)
against the actual cart contents instead of the three `<select>` values.

## Deploying

It's static output — any static host works with zero config:
drag-and-drop the folder onto Netlify or Cloudflare Pages, or run
`vercel` / `netlify deploy` from inside the folder, or push to a repo
and enable GitHub Pages on it. No environment variables, no server.

## Optional next steps (not built here)

- **Real cart/checkout** — track selections in `localStorage` or a
  backend cart, apply the same "cheapest of every 3 is free" rule
  server-side at payment time (never trust a client-side discount for
  real money).
- **CMS for images/prices** — swap `products.js` for a fetch from a
  headless CMS (e.g. a simple JSON endpoint, or something like
  Contentful/Sanity) so non-developers can update the catalog.
- **Per-product static pages** — if you want shareable URLs per shoe
  (`/shoes/tidewave-runner.html`) instead of the modal, generate one
  HTML file per entry in `products.js` with a small script; the modal's
  markup is already close to what each page's `<main>` would contain.
- **React/Vite version** — the same `products.js` array drops straight
  into a `<ProductGrid products={PRODUCTS} />` component if the project
  later needs client-side routing, a real cart, or a component library.
  Say the word and I'll scaffold that version against this same data
  file and design tokens.
