# MoMA Exhibition Pages — Prototype

Mid-fidelity prototypes of MoMA exhibition detail pages. The project produces multiple HTML files — one per exhibition — each sharing the same stylesheet foundation. New exhibition variants are created by duplicating `index.html`.

---

## File Structure

```
exhibition-pages/
├── CLAUDE.md
├── index.html                     ← base exhibition detail template
├── [exhibition-name].html         ← additional exhibition variants (duplicated from index.html)
├── styles/
│   ├── tokens.css                 ← CSS custom properties from moma-design-tokens.tokens.json
│   └── main.css                   ← @font-face, reset, base styles, shared components
└── font/                          ← local MoMA Sans font files (.woff2, .woff)
```

---

## Coding Conventions

Derived from analysis of existing MoMA prototype work in this repository.

### HTML
- **2-space indentation** throughout
- **Semantic HTML5** elements: `<nav>`, `<main>`, `<section>`, `<aside>`, `<footer>`
- **BEM class naming**: `.block__element--modifier`
- **Section comments** marking each major region: `<!-- Hero -->`, `<!-- Site Navigation -->`, etc.
- No inline styles; all styling via class names and CSS custom properties

### CSS
- **CSS custom properties** for all design decisions (colors, spacing, type sizes) — never hardcode values
- All tokens defined in `styles/tokens.css`; component styles in `styles/main.css`
- **Property order**: layout → box model → typography → visual → animation
- Responsive via `@media (min-width: ...)` — mobile-first
- No frameworks; no utility classes

### JavaScript
- Minimal vanilla JS only (e.g. mobile menu toggle)
- No libraries or build tools

---

## Design Tokens

Source file: `moma-design-tokens.tokens.json`

### Breakpoints
| Name | Value |
|---|---|
| Mobile (default) | < 768px |
| Desktop | ≥ 768px |

### Grid
| | Mobile | Desktop |
|---|---|---|
| Columns | 2 | 6 |
| Gutter | 24px | 48px |
| Offset | 24px | 0 (centered) |
| Column width | — | 156px |

### Typography Scale (Desktop)
| Token | Size |
|---|---|
| `--type-scale-sm` | 16px |
| `--type-scale-md` | 18px |
| `--type-scale-body` | 18px / line-height 22px |
| `--type-scale-lg` | 22px |
| `--type-scale-ml` | 30px |
| `--type-scale-xl` | 40px |
| `--type-scale-2xl` | 60px |
| `--type-scale-hero` | 80px |

### Typography Scale (Responsive/Mobile)
| Token | Size |
|---|---|
| `--type-sm` | 14px |
| `--type-md` | 16px |
| `--type-lg` | 20px |
| `--type-ml` | 24px |
| `--type-xl` | 28px |
| `--type-2xl` | 32px |
| `--type-hero` | 36px |

### Font Weights
| Weight | File suffix |
|---|---|
| 300 | `moma-sans__light` |
| 400 | `moma-sans__regular` |
| 500 | `moma-sans__medium` |
| 600 | `moma-sans__semibold` |
| 700 | `moma-sans__bold` |

---

## Page Structure

```html
<nav class="site-nav">              <!-- placeholder; will be replaced with Figma design -->
<main class="exhibition">
  <section class="exhibition__hero">
  <section class="exhibition__intro page-container">
  <section class="exhibition__body page-container">
  <aside class="exhibition__sidebar page-container">
  <section class="exhibition__related">
<footer class="site-footer">
```

---

## Working Notes

- Nav is a placeholder — will be replaced with Figma reference
- `exhibition__sidebar` may be repositioned alongside `exhibition__body` once Figma layout is confirmed
- `page-container` handles horizontal padding and max-width; apply to inner wrappers, not full-bleed sections

---

## Collaboration Rules

- **Show a plan before making changes.** Confirm approach with the user before editing files, especially for structural changes.
- **Prefer small, approvable changes.** Break large tasks into reviewable steps rather than making sweeping edits in one pass.
- **Preserve hand-coded edits.** Do not remove or alter the user's manual changes unless they directly conflict with the requested outcome.
- **Figma MCP first.** Always attempt to fetch design context from Figma before implementing UI. If the MCP is unavailable, say so explicitly — do not approximate from other sources.
- **r1-b.html is the active accordion prototype.** Changes to the accordion system go here only unless specified otherwise.

## r1-b Accordion Architecture

Accordion sections are wrapped in `<div id="accordion-list">` for JS-driven reordering. Each section has a `data-accordion-id` attribute used by the visitor-choice filter.

### Visitor Choice Filter (`data-filter` values)

Each filter defines an explicit order and open/closed state. Member promo is repositioned after the specified anchor accordion.

| Button | Order (top → bottom) | Open | Promo after |
|---|---|---|---|
| `visit` | resources, events, catalogues, artists, on-view, videos, articles, history, images, research-resources | resources, events, catalogues, artists, on-view | `events` |
| `explore` | artists, catalogues, videos, articles, resources, events, on-view, history, images, research-resources | artists, catalogues, videos, articles | `artists` |
| `research` | artists, history, images, videos, articles, catalogues, research-resources, resources, events, on-view | artists, history, images, videos, articles, catalogues, research-resources | `artists` |

Deselecting a button restores all accordions to open and resets DOM order to original.

### Accordion ID Map
| `data-accordion-id` | Section |
|---|---|
| `artists` | Artists |
| `events` | Events |
| `resources` | Resources and guides (placeholder) |
| `audio` | Audio |
| `catalogues` | Shop / Catalogues and more |
| `videos` | Videos and podcasts |
| `articles` | Articles (placeholder) |
| `history` | Exhibition history (placeholder) |
| `images` | Installation images |
| `on-view` | On view now (placeholder) |
| `thanks` | Special thanks |
| `research-resources` | Research resources (placeholder) |
| `reading` | Further reading |
