# Modern Platform Guide

---

This file defines which web platform APIs and CSS features this project uses.
Agents must read it before writing any HTML, CSS, or JavaScript.

Where a native platform solution exists, use it. Do not reach for a JavaScript
workaround, a utility library, or a preprocessor feature unless this file
explicitly says otherwise.

**Cross-reference `docs/project-brief.md` → Browser support before using any
API not in your training data's confident support range. Flag gaps rather than
silently polyfilling or falling back.**

---

## HTML

| Task | Use | Do not use |
|------|-----|------------|
| Modal / dialog | `<dialog>` with `.showModal()` and `.close()` | `<div role="dialog">` with a custom focus trap and backdrop |
| Disclosure / accordion | `<details>` + `<summary>` | `<div>` with a click handler toggling `display` or `height` |
| Lazy-load images | `<img loading="lazy">` | Intersection Observer scroll hack, JS lazy-load libraries |
| Form validation | Constraint Validation API (`required`, `pattern`, `min`, `max`, `setCustomValidity`) | Manual JS validation re-implementing what the browser already provides |
| Popovers / tooltips | `popover` attribute + `popovertarget` | Custom JS-positioned `<div>` overlays with manual show/hide logic |
| Inline SVG icons | Inline `<svg>` directly in the component | `<img src="icon.svg">` or CSS `background-image` for icons |
| Progress indicators | `<progress value="" max="">` | `<div>` with a `width` style or animation |
| Meter / gauge values | `<meter value="" min="" max="">` | Custom `<div>` bars with inline width calculations |
| Native date / time input | `<input type="date">`, `<input type="time">` | Third-party date picker libraries for standard date entry |
| Colour input | `<input type="color">` | Custom colour picker libraries for basic colour selection |
| Toggle / switch | `<input type="checkbox">` styled with CSS | Custom `<div role="switch">` with manual ARIA state management |
| Image with fallback | `<picture>` + `<source>` | JS-based format detection or `onerror` swapping |
| Responsive images | `srcset` and `sizes` attributes | JS that swaps `src` on resize |

---

## CSS

| Task | Use | Do not use |
|------|-----|------------|
| Two-dimensional layout | CSS Grid | Float layouts, `display: table` hacks, absolute positioning for layout |
| One-dimensional layout and spacing | Flexbox with `gap` | `margin` on child elements to simulate spacing, clearfix |
| Responsive fluid sizing | `clamp(min, preferred, max)` | JS `resize` listener computing sizes; separate breakpoint declarations for every step |
| Aspect ratio | `aspect-ratio` property | Padding-bottom percentage hack (`padding-top: 56.25%`) |
| Sticky positioning | `position: sticky` | JS `scroll` listener toggling a `.is-fixed` class |
| Theming and design tokens | CSS custom properties (`var(--token)`) | Sass variables for runtime theming, hardcoded values, JS class-swapping |
| CSS cascade management | `@layer` for ordering third-party vs project styles | Specificity hacks, `!important` to override cascade problems |
| Scroll snapping | `scroll-snap-type` / `scroll-snap-align` | JS scroll hijacking or animation loops |
| Subgrid alignment | CSS Subgrid (`grid-template-columns: subgrid`) | Nested grids with duplicated and manually synchronised track definitions |
| Component-level breakpoints | Container queries (`@container`) | Viewport `@media` queries used for component-level layout decisions |
| Parent state selectors | `:has()` | JS that walks the DOM to add a class to a parent element |
| CSS nesting | Native CSS nesting | Sass or PostCSS nesting plugins used solely to work around lack of native support |
| Logical / directional properties | `margin-inline`, `padding-block`, `inset-inline-start`, etc. | Physical properties (`margin-left`, `padding-top`) for layouts that need to support RTL |
| Smooth scrolling | `scroll-behavior: smooth` | JS animation loop incrementing `scrollTop` |
| Text truncation (single line) | `text-overflow: ellipsis` with `overflow: hidden; white-space: nowrap` | JS that measures text and truncates the string |
| Custom focus ring | `outline` styled via CSS (never `outline: none` without a replacement) | Removing the outline and relying on a `:hover` state as a substitute |
| Colour functions | `oklch()` or `color-mix()` where targets support it | JS colour manipulation libraries for static colour derivations |

---

## JavaScript

| Task | Use | Do not use |
|------|-----|------------|
| Intersection detection | `IntersectionObserver` | `scroll` event listener + `getBoundingClientRect()` on every frame |
| Element resize detection | `ResizeObserver` | `resize` window event listener querying element dimensions |
| DOM mutation detection | `MutationObserver` | `setInterval` polling the DOM for changes |
| Cancellable fetch | `AbortController` with a `signal` passed to `fetch` | Ignoring the resolved response after a race condition |
| Deep object clone | `structuredClone()` | `JSON.parse(JSON.stringify(...))` — loses `undefined`, `Date`, `Map`, `Set`, functions |
| Last array element | `array.at(-1)` | `array[array.length - 1]` |
| Safe own property check | `Object.hasOwn(obj, key)` | `obj.hasOwnProperty(key)` — fails on objects with no prototype |
| Optional chaining | `a?.b?.c` | `a && a.b && a.b.c` |
| Nullish fallback | `value ?? default` | `value \|\| default` — incorrectly treats `0` and `""` as falsy |
| Concurrent async (partial failure OK) | `Promise.allSettled()` | `Promise.all()` when partial failure should not abort everything |
| Concurrent async (all must succeed) | `Promise.all()` | Sequential `await` when operations are independent |
| URL construction | `URL` and `URLSearchParams` APIs | String concatenation or manual encoding for query parameters |
| Clipboard write | `navigator.clipboard.writeText()` | `document.execCommand('copy')` — deprecated and unreliable |
| Smooth scroll to element | `element.scrollIntoView({ behavior: 'smooth' })` | JS animation loop incrementing `scrollTop` |
| Unique IDs | `crypto.randomUUID()` | `Math.random()` string constructions for IDs that need to be unique |
| Type checking at runtime | `instanceof`, `typeof`, or `Array.isArray()` | String comparisons against `Object.prototype.toString` unless targeting older runtimes |
| Object immutability | `Object.freeze()` | Naming conventions or comments to signal "do not mutate" |
| Event delegation | Single listener on a parent with `event.target.closest()` | Attaching individual listeners to every child element in a list |
| Scheduling non-urgent work | `requestIdleCallback()` | `setTimeout(fn, 0)` for deferring low-priority work |
| Animation frame sync | `requestAnimationFrame()` | `setInterval` or `setTimeout` for visual updates |

---

## Patterns and conventions

### Prefer declarative over imperative

Reach for a CSS or HTML solution before a JavaScript one. JS is appropriate for
behaviour that cannot be expressed in markup or styles — not as a default.

**Example:** a disclosure widget should be `<details>` + `<summary>`, not a `<div>`
with a click handler and toggled class. The browser handles keyboard interaction,
ARIA state, and animation for free.

### Avoid re-implementing what the browser provides

Check whether the platform already solves the problem before writing custom logic.
Common areas where agents drift toward unnecessary custom code:

- **Focus trapping** — `<dialog>` traps focus natively; do not write a focus trap for a `<dialog>`
- **Form validation** — the Constraint Validation API covers required fields, patterns,
  min/max ranges, and custom messages via `setCustomValidity`; do not re-validate
  fields the browser already validates
- **Smooth scroll** — `scroll-behavior: smooth` and `scrollIntoView` cover the
  standard cases; JS animation loops are not needed

### No layout with JavaScript

Do not use JavaScript to calculate or apply layout — widths, heights, positions,
or column counts. Use CSS Grid, Flexbox, container queries, and `clamp()`.
JavaScript layout code breaks on resize, causes layout thrash, and is harder to
maintain than a CSS equivalent.

### Minimal dependencies

Do not install a package to solve a problem the platform already solves. Before
adding a dependency, check whether a native API exists. If a library is genuinely
needed, stop and ask rather than installing it unilaterally — see the agent
behaviour rules in `docs/project-brief.md`.

---

## Notes for AI agents

- **Check browser support first** — cross-reference `docs/project-brief.md` →
  Browser support before using any API. If support is insufficient for the project
  targets, flag it and propose an approach rather than silently falling back to a
  legacy workaround
- **The tables are not exhaustive** — if a native platform API exists for a task
  not listed here, use it and note it in the relevant component spec's Notes section
- **Specs take precedence on specifics** — if a component spec names a particular
  API or element to use, follow the spec. This file defines the default choice when
  the spec is silent
- **Native element, then ARIA** — always prefer a native semantic element over a
  generic element with ARIA roles. `<dialog>` is correct; `<div role="dialog">` is
  a fallback for environments where `<dialog>` cannot be used
- **If in doubt, ask** — if you are unsure whether a native solution covers a
  requirement fully, stop and ask rather than defaulting to a JS workaround
