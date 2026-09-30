# Component Spec: ThemeToggle

**Status:** Draft
**Last updated:** <!-- e.g. 2025-01-15 -->

> See `docs/project-brief.md` → Spec conventions for the full status key and agent behaviour rules.

---

## Purpose

> A single paragraph describing what problem this component solves, who uses it,
> and when. Avoid describing how it works — describe why it exists.

A toggle control that allows users to switch between light and dark colour
schemes. It reflects the current scheme and updates it immediately when
activated. Used in the main navigation so it is accessible from every page.

---

## Dependencies

Components, utilities, assets, or design token categories this component relies on.
List anything an agent would need to locate or implement before building this component.

| Type | Name | Location | Status |
|------|------|----------|--------|
| Asset | `icon-sun.svg` | `src/assets/icons/icon-sun.svg` | n/a |
| Asset | `icon-moon.svg` | `src/assets/icons/icon-moon.svg` | n/a |
| Tokens | Colour | `docs/design-tokens.md#colour` | n/a |
| Tokens | Spacing | `docs/design-tokens.md#spacing` | n/a |

> The icon assets above are placeholders — replace with the actual filenames once chosen. Both must be inlined, not referenced via `<img>`.

---

## Interface

The props, attributes, and events that define how this component is used.
The agent will derive the implementation interface directly from this section.

### Props / Attributes

| Name | Type | Required | Default | Description |
|------|------|:--------:|---------|-------------|
| `initialTheme` | `light` / `dark` | | `light` | Sets the final tier of the Priority order in `docs/features/dark-mode.md` — the theme applied when neither a stored nor an operating system preference is available |

### Events / Callbacks

| Event | Fires when | Payload |
|-------|------------|---------|
| change / onChange / onchange | The user activates the toggle | `'light' \| 'dark'` — the new theme value as a plain string |

### Public methods _(if applicable)_

None.

---

## Behaviour

How the component behaves on first render, across its different states, and in
response to user interaction.

### Default / initial state

On first render the component resolves the theme by the Priority order in
`docs/features/dark-mode.md` — that file owns the order and this spec does not
restate it. `initialTheme` supplies its final tier, the default used when neither
a stored preference nor an operating system preference is available. The resolved
scheme is reflected visually in the toggle.

### States

| State | Trigger | Visual / functional result |
|-------|---------|---------------------------|
| light | Theme is `light` | Toggle reflects light mode is active |
| dark | Theme is `dark` | Toggle reflects dark mode is active |

### Interaction rules

- When the user activates the toggle → the theme switches immediately
- When the theme switches → `data-theme` on the root `<html>` element is updated
- When the theme switches → the new preference is saved to `localStorage` under the key `color-scheme`
- When the theme switches → the change event fires with the new theme value

---

## Accessibility

The accessibility requirements this component must meet. The agent must not
consider the component complete until all of these are satisfied.

- Must use a native `<button>` element
- Must have an `aria-label` that reflects the current state, e.g. `"Switch to dark mode"` or `"Switch to light mode"`
- Must be keyboard operable — activatable with `Enter` and `Space`
- Focus ring must meet Web Content Accessibility Guidelines (WCAG) 2.2 AA contrast requirements
- Target size: at least 24 by 24 CSS pixels — an icon-only control is the easiest one to draw too small
- Icon or visual indicator must not be the only means of conveying the current state

---

## Error handling

How the component should behave when something goes wrong.

- If `localStorage` is unavailable, fall back to OS preference or `initialTheme` silently — do not throw
- If an invalid `initialTheme` value is passed, fall back to `light` and emit a `console.warn` in development only

---

## Styling notes

Token categories and CSS patterns specific to this component.
The agent must read `docs/design-tokens.md` before writing any styles.

- **Tokens used** — colour, spacing
- **CSS patterns** — Block-Element-Modifier (BEM): `.theme-toggle` (block); state communicated via `aria-label` and icon swap, not CSS class modifiers
- **Dark mode** — all colours must reference the semantic custom properties defined in `docs/design-tokens.md`: the light theme on `:root`, the dark theme as a `[data-theme="dark"]` override. This component sets the attribute and does not define any colour values. It still writes both values from the feature spec's Implementation notes table — `data-theme="light"` is what tells the operating-system media query to stand down when the user has explicitly chosen light, so omitting it because "light is the default" breaks light mode on a dark-themed machine

---

## Test cases

The agent will generate one test function per entry. IDs must be unique within
this spec and must match the test file exactly.

### Render

- [ ] **TC-01** — Renders with no props and defaults to `initialTheme`
- [ ] **TC-02** — Applies stored `localStorage` preference on first render
- [ ] **TC-03** — Applies OS `prefers-color-scheme` when no stored preference exists

### Interaction

- [ ] **TC-04** — Activating the toggle switches the theme
- [ ] **TC-05** — Activating the toggle updates `data-theme` on the root `<html>` element
- [ ] **TC-06** — Activating the toggle saves the new preference to `localStorage`
- [ ] **TC-07** — Change event fires with the correct theme string (`'light'` or `'dark'`) when toggled

### Accessibility

- [ ] **TC-08** — `aria-label` reflects the current state correctly (e.g. `"Switch to dark mode"` when light is active)

### Edge cases

- [ ] **TC-09** — Falls back gracefully when `localStorage` is unavailable

---

## Out of scope

Explicitly listing what this component does NOT cover prevents scope creep and
helps the agent avoid building more than is required.

- Displaying a text label alongside the toggle — icon only
- Animating the theme transition
- Supporting more than two themes

---

## Example usage

How this component is implemented depends on the active framework selection in
`docs/project-brief.md`. This is one of the scaffold's shipped examples, so every
framework is shown — the stack has not been chosen yet, and the `[active]` marks
in the brief are a placeholder default, not a recommendation. **In your own specs,
keep only the example matching your active selection and delete the rest.** That
is what the Example usage row in the checklist below is asking for.

**Vanilla:**

```html
<button class="theme-toggle" aria-label="Switch to dark mode">
  <!-- icon -->
</button>
```

**Astro:**

```astro
<ThemeToggle initialTheme="light" />
```

**React:**

```jsx
<ThemeToggle initialTheme="light" onChange={(theme) => console.log(theme)} />
```

**Svelte:**

```svelte
<ThemeToggle initialTheme="light" onchange={(theme) => console.log(theme)} />
```

---

## Notes for "Ai" implementation

Additional context, constraints, and implementation guidance that the agent
should be aware of before writing any code.

- **Read first** — `docs/features/dark-mode.md` for full feature context before implementing
- **Theme application** — set `data-theme` on the root `<html>` element; this component does not define colour values, only toggles the attribute
- **localStorage key** — the canonical key is `color-scheme`, defined in `docs/features/dark-mode.md`
- **CSS** — import from `./ThemeToggle.css` or `./ThemeToggle.scss` depending on active styles selection in `docs/project-brief.md`
- **Assets** — icon SVGs must be inlined, not referenced via `<img>`; see Dependencies above for asset locations
- **Related specs** — used in `docs/specs/layouts/main-layout.spec.md`
- **Gotchas** — TC-03 requires mocking `window.matchMedia` in tests; no animation libraries — CSS transitions only

---

## Draft → Ready checklist

Complete every item before changing the status to `Ready`.
**Agents must not begin implementation until the status is `Ready`.**

- [x] Purpose describes the *why*, not the *how*
- [x] All props are named, typed, and have a default where applicable
- [x] All events include a payload description
- [x] Every meaningful state is listed in the States table
- [x] Interaction rules cover all non-obvious behaviours
- [x] Accessibility requirements are specific, not generic
- [x] At least one test case exists per behaviour and state
- [x] Out of scope section is filled in
- [x] Every dependency that has a spec of its own has reached `Ready` — none of these have specs
- [x] Example usage matches the active framework
- [x] Notes for "Ai" implementation are complete

> Every row is ticked and this spec is still `Draft` — that is not an oversight.
> **Promotion is a human act**, so a spec stays `Draft` until you have read it and
> decided the contract is settled, checklist or no checklist. Nothing here
> promotes itself. `docs/features/dark-mode.md` is `Draft` in turn because it
> depends on this one.
