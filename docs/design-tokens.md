# Design Tokens

> **This is a design reference, not a spec.** It defines the values every
> component references, but it carries no `**Status:**` line and is never
> promoted to `Ready`. Nothing gates it: it simply has to be filled in before any
> styles are written (`docs/workflow.md` Step 6).

Design tokens are the named values that define the visual language of this project.
They are defined once and referenced everywhere — in components, pages, and layouts.

> *The values below are sensible baselines — adjust them to match your project's visual language before building components.*

---

## Token architecture

This project uses a **two-layer token system**:

1. **Primitive tokens** — raw, named values. Never referenced directly in components.
   Define the full palette here.
2. **Semantic tokens** — purposeful aliases that components reference. Defined on
   `:root` as the light theme, then overridden under `[data-theme="dark"]` on the
   root `<html>` element, so switching themes only requires toggling that
   attribute.

```css
/* ✗ Don't do this in a component — the value is opaque and won't adapt to the theme */
color: var(--color-slate-900);

/* ✓ Do this — the semantic token resolves to the correct primitive per theme */
color: var(--color-text-body);
```

No hardcoded colour values anywhere in stylesheets. Always reference a semantic token.
See `docs/features/dark-mode.md` for theming implementation details.

> [!IMPORTANT]
> **Define the light theme on bare `:root`, never on `[data-theme="light"]`.** *If
> every semantic token lives inside an attribute selector, then a page with no
> `data-theme` attribute matches neither block and **no colour token resolves at
> all** — text falls back to the browser's default, backgrounds go transparent.
> That is the state of every page before the theme script runs, and the permanent
> state of every page where it never runs: JavaScript disabled or still loading, a
> script error, a crawler, a view-source reader. Nothing errors and nothing warns;
> the page simply renders unstyled. `:root` is what guarantees a defined value
> before any attribute exists, which is why the dark block is an override rather
> than a twin.*

The shape that follows from that:

```css
:root {
  /* primitives, then the light theme's semantic aliases */
}

[data-theme="dark"] {
  /* only the semantic aliases whose primitive changes */
}
```

To follow the operating system when the user has expressed no preference, add a
`prefers-color-scheme` block guarded so an explicit light choice still wins:

```css
@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    /* the same overrides as the [data-theme="dark"] block */
  }
}
```

The guard matters: without `:not([data-theme="light"])`, a user on a dark
operating system who deliberately picks light mode gets dark anyway, and the
toggle appears broken.

---

## How to use tokens

Tokens live in `src/styles/tokens.{css|scss}` and are imported by
`src/styles/main.{css|scss}`, which is loaded once at the top level of the project.
Neither file is pre-created in the scaffold. They are created at **Step 6 of
`docs/workflow.md`**, not during initial project setup — this document has to be filled
in first, since its values are what the files implement. The file extension follows
the active Styles selection in `docs/project-brief.md`.

**If using an agent:** ask it to read this document and create the token and main
style files. It will stop and ask you to fill this document in if it is still empty.

**If building by hand:** create both files before writing any styles. The token file
should follow the structure in this document.

Either way, the files must exist before an agent writes any styles — Step 6 can
happen any time before that, and can be re-run as the project's visual language
evolves.

Reference tokens in any stylesheet like this:

```css
.example {
  color: var(--color-text-body);
  padding: var(--space-sm) var(--space-md);
  font-size: var(--text-base);
  transition: color var(--duration-base) var(--easing-default);
}
```

---

## Colour

### Primitive palette

Raw values. Define the full range here — these are the only place actual colour
values appear. Never reference primitives directly in component styles.

| Token | Value | Notes |
|-------|-------|-------|
| `--color-slate-50` | `#f8fafc` | Lightest neutral |
| `--color-slate-100` | `#f1f5f9` | |
| `--color-slate-200` | `#e2e8f0` | |
| `--color-slate-300` | `#cbd5e1` | |
| `--color-slate-500` | `#64748b` | Mid neutral |
| `--color-slate-700` | `#334155` | |
| `--color-slate-900` | `#0f172a` | Darkest neutral |
| `--color-brand-300` | `#93c5fd` | Light brand tint |
| `--color-brand-500` | `#3b82f6` | Primary brand |
| `--color-brand-700` | `#1d4ed8` | Dark brand shade |
| `--color-red-500` | `#ef4444` | Error / danger |
| `--color-green-500` | `#22c55e` | Success |
| `--color-amber-500` | `#f59e0b` | Warning |

> *Rename the scale to match your palette — `slate`, `blue`, `zinc`, etc.
> Add or remove steps as needed. The names don't matter; the two-layer pattern does.*

---

### Semantic tokens — Light theme

Defined on bare `:root` in the token file, so these values apply before any
`data-theme` attribute exists — see Token architecture above for why this one is
not attribute-scoped. Replace the example primitives with your actual palette
tokens once the primitive table above is filled in.

| Token | Maps to (example) | Usage |
|-------|-------------------|-------|
| `--color-bg-page` | `--color-slate-50` | Page background |
| `--color-bg-surface` | `--color-slate-100` | Card, panel, input backgrounds |
| `--color-bg-elevated` | `--color-slate-200` | Overlays, dropdowns |
| `--color-text-body` | `--color-slate-900` | Primary body text |
| `--color-text-muted` | `--color-slate-500` | Secondary / helper text |
| `--color-text-disabled` | `--color-slate-300` | Disabled labels |
| `--color-text-inverse` | `--color-slate-50` | Text on dark/coloured backgrounds |
| `--color-border-default` | `--color-slate-200` | Default borders and dividers |
| `--color-border-focus` | `--color-brand-500` | Focus ring colour |
| `--color-action-primary` | `--color-brand-500` | Primary buttons, active links |
| `--color-action-primary-hover` | `--color-brand-700` | Primary hover state |
| `--color-action-secondary` | `--color-slate-200` | Secondary actions |
| `--color-action-danger` | `--color-red-500` | Destructive actions |
| `--color-action-danger-hover` | `--color-red-500` | Destructive hover — darken via filter or swap primitive |
| `--color-feedback-error` | `--color-red-500` | Error text and icons |
| `--color-feedback-error-bg` | `--color-slate-100` | Error message backgrounds — use a light tint |
| `--color-feedback-success` | `--color-green-500` | Success text and icons |
| `--color-feedback-success-bg` | `--color-slate-100` | Success message backgrounds — use a light tint |
| `--color-feedback-warning` | `--color-amber-500` | Warning text and icons |
| `--color-feedback-warning-bg` | `--color-slate-100` | Warning message backgrounds — use a light tint |

---

### Semantic tokens — Dark theme

Scoped to `[data-theme="dark"]` in the token file, as an override of the `:root`
values above rather than a second complete set. Usage is identical to the light
theme — only the primitive each token points to changes. The mappings below show
the pattern: backgrounds invert toward the dark end of the scale, text toward the
light end, and brand colours shift to a lighter tint to maintain contrast against
dark surfaces.

Replace the example primitives with your actual palette tokens once the primitive
table above is filled in.

| Token | Maps to (example) | Notes |
|-------|-------------------|-------|
| `--color-bg-page` | `--color-slate-900` | Darkest surface — the page canvas |
| `--color-bg-surface` | `--color-slate-700` | Cards and panels sit one step lighter |
| `--color-bg-elevated` | `--color-slate-500` | Dropdowns and overlays, lighter still |
| `--color-text-body` | `--color-slate-50` | Primary text must contrast against `bg-page` |
| `--color-text-muted` | `--color-slate-300` | Secondary text, still legible |
| `--color-text-disabled` | `--color-slate-500` | Disabled — visually receded but present |
| `--color-text-inverse` | `--color-slate-900` | Text placed on light/brand-coloured surfaces |
| `--color-border-default` | `--color-slate-700` | Subtle — just enough to separate surfaces |
| `--color-border-focus` | `--color-brand-300` | Lighter tint so the ring reads on dark backgrounds |
| `--color-action-primary` | `--color-brand-300` | Lighter than light-theme primary — same reason |
| `--color-action-primary-hover` | `--color-brand-500` | Deepens on hover, not lightens |
| `--color-action-secondary` | `--color-slate-700` | Matches surface colour; border provides definition |
| `--color-action-danger` | `--color-red-500` | Can often stay the same as light theme |
| `--color-action-danger-hover` | `--color-red-500` | Adjust if contrast against dark bg is insufficient |
| `--color-feedback-error` | `--color-red-500` | Check contrast; may need a lighter shade |
| `--color-feedback-error-bg` | `--color-slate-700` | Use surface colour + border rather than a tint |
| `--color-feedback-success` | `--color-green-500` | Check contrast against dark backgrounds |
| `--color-feedback-success-bg` | `--color-slate-700` | Same pattern as error-bg |
| `--color-feedback-warning` | `--color-amber-500` | Amber often works on dark without adjustment |
| `--color-feedback-warning-bg` | `--color-slate-700` | Same pattern as error-bg |

> **Checking contrast:** before marking any component spec `Complete`, verify that
> text/background token pairs in both themes meet the Web Content Accessibility
> Guidelines (WCAG) 2.2 AA minimums —
> **4.5:1** for body text, **3:1** for large text and UI components.
> [Colour Contrast Checker](https://webaim.org/resources/contrastchecker/) is a
> quick way to do this.

---

## Spacing

All values in `rem`. The scale uses a 4px base (`0.25rem` steps) to align with standard design tool grids.

| Token | Value | px equivalent | Usage |
|-------|-------|--------------|-------|
| `--space-xs` | `0.25rem` | 4px | Tight internal padding, icon gaps |
| `--space-sm` | `0.5rem` | 8px | Component internal padding |
| `--space-md` | `1rem` | 16px | Default component padding, small gaps |
| `--space-lg` | `1.5rem` | 24px | Section gaps, card padding |
| `--space-xl` | `2rem` | 32px | Large section gaps |
| `--space-2xl` | `3rem` | 48px | Page-level vertical rhythm |
| `--space-3xl` | `4rem` | 64px | Hero sections, major layout gaps |

---

## Typography

### Font families

| Token | Value | Usage |
|-------|-------|-------|
| `--font-sans` | `system-ui, -apple-system, BlinkMacSystemFont, 'Segoe UI', sans-serif` | Body text, UI elements |
| `--font-mono` | `ui-monospace, 'Cascadia Code', 'Fira Code', monospace` | Code blocks, technical labels |

### Font sizes

All values in `rem`.

| Token | Value | px equivalent | Usage |
|-------|-------|--------------|-------|
| `--text-xs` | `0.75rem` | 12px | Labels, captions |
| `--text-sm` | `0.875rem` | 14px | Secondary text, helper copy |
| `--text-base` | `1rem` | 16px | Body text |
| `--text-lg` | `1.125rem` | 18px | Subheadings, lead text |
| `--text-xl` | `1.25rem` | 20px | Section headings |
| `--text-2xl` | `1.5rem` | 24px | Page headings |
| `--text-3xl` | `2rem` | 32px | Hero headings |

### Font weights

| Token | Value | Usage |
|-------|-------|-------|
| `--weight-regular` | `400` | Body text |
| `--weight-medium` | `500` | Emphasis, labels |
| `--weight-bold` | `700` | Headings, strong emphasis |

### Line heights

| Token | Value | Usage |
|-------|-------|-------|
| `--leading-tight` | `1.25` | Headings |
| `--leading-normal` | `1.5` | Body text |
| `--leading-loose` | `1.75` | Long-form / prose |

### Letter spacing

| Token | Value | Usage |
|-------|-------|-------|
| `--tracking-tight` | `-0.025em` | Large display headings |
| `--tracking-normal` | `0em` | Body text |
| `--tracking-wide` | `0.05em` | Uppercase labels, badges |

---

## Border

| Token | Value | Usage |
|-------|-------|-------|
| `--radius-sm` | `0.25rem` | Buttons, inputs |
| `--radius-md` | `0.375rem` | Cards, panels |
| `--radius-lg` | `0.5rem` | Modals, large surfaces |
| `--radius-full` | `9999px` | Pills, avatars |
| `--border-width-default` | `1px` | Standard borders |
| `--border-width-focus` | `2px` | Focus ring border width |

> Border *colours* live in the Colour → Semantic section above
> (`--color-border-default`, `--color-border-focus`) so they adapt to the active theme.

---

## Focus ring

Centralised focus ring values used across all interactive elements. These feed
directly into the accessibility requirements in component specs.

| Token | Value | Usage |
|-------|-------|-------|
| `--focus-ring-width` | `2px` | Width of the focus outline |
| `--focus-ring-offset` | `2px` | Gap between element and ring |
| `--focus-ring-color` | `var(--color-border-focus)` | Alias `--color-border-focus` |

> The focus ring must meet a minimum **3:1 contrast ratio** against adjacent colours
> (WCAG 2.2 AA). Verify both light and dark theme values before marking any
> interactive component spec as `Complete`.

---

## Shadows

| Token | Value | Usage |
|-------|-------|-------|
| `--shadow-sm` | `0 1px 2px 0 rgb(0 0 0 / 0.05)` | Subtle lift — inputs, buttons |
| `--shadow-md` | `0 4px 6px -1px rgb(0 0 0 / 0.1), 0 2px 4px -2px rgb(0 0 0 / 0.1)` | Cards, dropdowns |
| `--shadow-lg` | `0 10px 15px -3px rgb(0 0 0 / 0.1), 0 4px 6px -4px rgb(0 0 0 / 0.1)` | Modals, overlays |

> If the project uses dark mode, consider reducing shadow opacity in the dark
> theme — heavy shadows on dark surfaces can look harsh.

---

## Transitions

Shared duration and easing values. All animated components must use these tokens
rather than raw values, so motion can be tuned globally.

| Token | Value | Usage |
|-------|-------|-------|
| `--duration-fast` | `100ms` | Micro-interactions (hover, focus) |
| `--duration-base` | `200ms` | Standard transitions |
| `--duration-slow` | `400ms` | Larger, more deliberate animations |
| `--easing-default` | `cubic-bezier(0.4, 0, 0.2, 1)` | General-purpose easing |
| `--easing-out` | `cubic-bezier(0, 0, 0.2, 1)` | Elements entering the screen |
| `--easing-in` | `cubic-bezier(0.4, 0, 1, 1)` | Elements leaving the screen |

> All animated components must respect `prefers-reduced-motion`. Override the
> duration tokens inside a single `@media (prefers-reduced-motion: reduce)` block in
> the token file rather than scattering the media query across component files.

> [!IMPORTANT]
> **Set the reduced-motion durations to `0.01ms`, not `0ms`.** *Per the Mozilla
> Developer Network (MDN), if transition duration and delay are both zero
> [there is no transition and none of the transition events fire](https://developer.mozilla.org/en-US/docs/Web/API/Element/transitionend_event) —
> not `transitionrun`, `transitionstart`, `transitionend` or `transitioncancel`.
> Any component that removes an element, restores focus, or advances state in a
> `transitionend` handler therefore stalls forever, and only for users who asked
> for reduced motion, who are the least likely to be in anyone's test matrix.
> `0.01ms` is imperceptible and still fires the events.*

```css
@media (prefers-reduced-motion: reduce) {
  :root {
    --duration-fast: 0.01ms;
    --duration-base: 0.01ms;
    --duration-slow: 0.01ms;
  }
}
```

> **Duration overrides cover transitions, not loops.** A looping animation
> (`animation: … infinite`) does not stop when its duration drops to `0.01ms` —
> it cycles faster than the display can draw, so it reads as jitter rather than
> stillness. Stop those with `animation: none` in the component's own
> `@media (prefers-reduced-motion: reduce)` rule. That is the one sanctioned
> exception to keeping the media query out of component files.

---

## Z-index

A named scale prevents magic numbers accumulating across components and layouts.

| Token | Value | Usage |
|-------|-------|-------|
| `--z-base` | `0` | Default stacking context |
| `--z-raised` | `10` | Sticky elements, floating labels |
| `--z-dropdown` | `100` | Menus, autocomplete panels |
| `--z-overlay` | `200` | Backdrop / scrim layers |
| `--z-modal` | `300` | Dialogs, drawers |
| `--z-toast` | `400` | Notifications, toasts |

---

## Notes

> *Document decisions here — why a particular scale was chosen, any tokens that
> deliberately alias others, or constraints on adding new ones. For example:*
>
> - *"The spacing scale uses a 4px base (`0.25rem` steps) to align with the design
>   tool grid."*
> - *"`--color-action-primary` in the dark theme intentionally uses a lighter brand
>   tint to meet contrast requirements against dark backgrounds."*
> - *"Do not add new primitive colour tokens without a corresponding semantic alias —
>   raw primitives must never be referenced in component stylesheets."*
