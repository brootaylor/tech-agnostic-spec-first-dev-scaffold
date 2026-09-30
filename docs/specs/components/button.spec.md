# Component Spec: Button

**Status:** Draft
**Last updated:** <!-- e.g. 2025-01-15 -->

> See `docs/project-brief.md` → Spec conventions for the full status key and agent behaviour rules.

---

## Purpose

> A single paragraph describing what problem this component solves, who uses it,
> and when. Avoid describing how it works — describe why it exists.

A reusable, accessible button covering every interactive call-to-action across the
application. Must support loading and disabled states and be usable by any team
without additional configuration.

---

## Dependencies

Components, utilities, assets, or design token categories this component relies on.
List anything an agent would need to locate or implement before building this component.

| Type | Name | Location | Status |
|------|------|----------|--------|
| Asset | `spinner.svg` | `src/assets/icons/spinner.svg` | n/a |
| Tokens | Colour | `docs/design-tokens.md#colour` | n/a |
| Tokens | Spacing | `docs/design-tokens.md#spacing` | n/a |
| Tokens | Typography | `docs/design-tokens.md#typography` | n/a |

---

## Interface

The props, attributes, and events that define how this component is used.
The agent will derive the implementation interface directly from this section.

### Props / Attributes

| Name | Type | Required | Default | Description |
|------|------|:--------:|---------|-------------|
| `label` | string | ✓ | — | Visible button text |
| `variant` | `primary` / `secondary` / `danger` / `ghost` | | `primary` | Visual style |
| `size` | `sm` / `md` / `lg` | | `md` | Size preset |
| `disabled` | boolean | | `false` | Prevents interaction |
| `loading` | boolean | | `false` | Shows spinner; prevents interaction |
| `type` | `button` / `submit` / `reset` | | `button` | HTML button type |
| `ariaLabel` | string | | — | Overrides the accessible label when the visible text is insufficient |

### Events / Callbacks

| Event | Fires when | Payload |
|-------|-----------|---------|
| click / onClick / onclick | User clicks and button is neither disabled nor loading | `MouseEvent` |

### Public methods _(if applicable)_

None.

---

## Behaviour

How the component behaves on first render, across its different states, and in
response to user interaction.

### Default / initial state

Renders as a native `<button>` element with the `primary` variant and `md` size.
Fully interactive.

### States

| State | Trigger | Result |
|-------|---------|--------|
| default | — | Fully interactive |
| hover | Pointer over | Visual feedback; no layout shift |
| focus | Keyboard focus | Visible ring (3:1 contrast min) |
| disabled | `disabled` is true | HTML `disabled` attr + `aria-disabled="true"`; click blocked |
| loading | `loading` is true | Spinner inline; `aria-busy="true"`; click blocked |

### Interaction rules

- When `loading` is true → spinner shown; the label stays visible and unchanged. Do not mute, dim, or reduce the opacity of the label — on the filled variants the only inverse text token is `--color-text-inverse`, and lowering its contrast against a brand background is the failure mode the palette already has. The spinner and `aria-busy="true"` carry the state
- When `disabled` OR `loading` → click event must never fire
- `type="submit"` inside a `<form>` → form submits normally via the browser

---

## Accessibility

The accessibility requirements this component must meet. The agent must not
consider the component complete until all of these are satisfied.

- Must use a native `<button>` element — never a `<div>` or `<span>`
- `disabled`: set both the HTML attribute AND `aria-disabled="true"`
- `loading`: set `aria-busy="true"`
- Focus ring: minimum 3:1 contrast — Web Content Accessibility Guidelines (WCAG) 2.2 AA
- Target size: at least 24 by 24 CSS pixels, including the `sm` size
- Spinner: `aria-hidden="true"`

---

## Error handling

How the component should behave when something goes wrong.

- If the click handler throws, do not catch — let the error bubble
- Invalid `variant` falls back to `primary` with a `console.warn` in dev only

---

## Styling notes

Token categories and CSS patterns specific to this component.
The agent must read `docs/design-tokens.md` before writing any styles.

- **Tokens used** — colour, spacing, typography, motion
- **CSS patterns** — Block-Element-Modifier (BEM): `.btn` (block), `.btn--primary` / `.btn--secondary` / `.btn--danger` / `.btn--ghost` (variant modifiers), `.btn--sm` / `.btn--lg` (size modifiers), `.btn--loading` (state modifier)
- **Dark mode** — button colours must reference the semantic colour tokens in `docs/design-tokens.md`, which resolve per theme on their own; no hardcoded values, and no assumption here about which selector defines them
- **Motion** — transitions use the `--duration-*` and `--easing-*` tokens; `docs/design-tokens.md` owns the `prefers-reduced-motion` response for them and this spec does not restate it. The spinner's rotation is the exception, because a duration override cannot stop a loop: under `@media (prefers-reduced-motion: reduce)` the rotation stops and the spinner stays rendered but static. `aria-busy="true"` conveys the loading state, not the movement

---

## Test cases

The agent will generate one test function per entry. IDs must be unique within
this spec and must match the test file exactly.

### Render

- [ ] **TC-01** — Renders a `<button>` with only the required `label`
- [ ] **TC-02** — Applies the correct CSS class for each `variant` value
- [ ] **TC-03** — Applies the correct CSS class for each `size` value

### Interaction

- [ ] **TC-04** — Click event fires when the button is enabled
- [ ] **TC-05** — Click event does NOT fire when `disabled` is true
- [ ] **TC-06** — Click event does NOT fire when `loading` is true
- [ ] **TC-07** — `type="submit"` triggers form submission

### Accessibility

- [ ] **TC-08** — Spinner is rendered and `aria-busy="true"` is set when `loading` is true
- [ ] **TC-09** — `aria-disabled="true"` is present when `disabled` is true
- [ ] **TC-10** — `ariaLabel` prop sets `aria-label` on the element

### Edge cases

- [ ] **TC-11** — Falls back to `primary` variant and emits `console.warn` when an invalid `variant` value is passed

---

## Out of scope

Explicitly listing what this component does NOT cover prevents scope creep and
helps the agent avoid building more than is required.

- Icon-only buttons → use `IconButton` component
- Navigation / anchor behaviour → use `LinkButton` component
- Button groups → use `ButtonGroup` component

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
<button class="btn btn--primary" type="button">Save changes</button>
```

**Astro:**

```astro
<Button label="Save changes" variant="primary" />
```

**React:**

```jsx
<Button label="Save changes" variant="primary" onClick={handleSave} />
```

**Svelte:**

```svelte
<Button label="Save changes" variant="primary" onclick={handleSave} />
```

---

## Notes for "Ai" implementation

Additional context, constraints, and implementation guidance that the agent
should be aware of before writing any code.

- **CSS** — import from `./Button.css` or `./Button.scss` depending on active styles selection in `docs/project-brief.md`
- **Assets** — `src/assets/icons/spinner.svg` must be imported and inlined; do not use `<img>`
- **Related specs** — none. A purely visual component can be specified on its own, with no feature spec above it; see `docs/project-brief.md` → Features and components
- **Gotchas** — no animation libraries; CSS only for all transitions and the spinner animation. Every transition duration must come from the `--duration-*` tokens in `docs/design-tokens.md` — that file owns the reduced-motion response and overrides those tokens centrally, so a hardcoded duration escapes it silently. The spinner's rotation is a loop, not a transition, and the central override does not reach it; see Styling notes

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
