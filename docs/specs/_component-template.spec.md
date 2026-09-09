# Component Spec: [ComponentName]

**Status:** Draft
**Last updated:** <!-- e.g. 2025-01-15 -->

> See `docs/project-brief.md` → Spec conventions for the full status key and agent behaviour rules.

---

## Is this the right template?

| You are describing | Use |
|--------------------|-----|
| A reusable piece of user interface | **This template** → `docs/specs/components/<name>.spec.md` |
| A **whole screen** built from several components | **This template** → `docs/specs/pages/<name>.spec.md` |
| A **page structure** that wraps other content | **This template** → `docs/specs/layouts/<name>.spec.md` |
| Something a **user can do**, and why it matters to them | `docs/features/_feature-template.md` → `docs/features/<name>.md` |

The test: can you write it as *"As a user, I want… so that…"*? If yes, it belongs
in `docs/features/` instead. A component is a unit of code; a feature is a unit
of user value. One component often serves several features.

If this component belongs to a feature, read that feature spec first — its
Implementation notes table fixes any value the two must agree on, and this spec
must reference those values rather than restate them.

> _Delete this section once you know which template you are in._

---

## Purpose

> A single paragraph describing what problem this component solves, who uses it,
> and when. Avoid describing how it works — describe why it exists.

_Replace this with your component's purpose._

---

## Dependencies

Components, utilities, assets, or design token categories this component relies on.
List anything an agent would need to locate or implement before building this component.

Every dependency that has a spec of its own must reach `Ready` before this
component can be implemented — a contract cannot be built on an unfinished
contract. Record each one's status in the Status column so the blocker is visible
without opening the other file. Dependencies with no spec (assets, tokens,
third-party utilities) take `n/a`.

| Type | Name | Location | Status |
|------|------|----------|--------|
| Component | `[ComponentName]` | `src/components/[ComponentName]/` — spec at `docs/specs/components/[name].spec.md` | Draft — must be `Ready` first |
| Utility | `[helperName]` | `src/lib/[helperName].js` | n/a |
| Asset | `[filename]` | `src/assets/[filename]` | n/a |
| Tokens | Colour | `docs/design-tokens.md#colour` | n/a |
| Tokens | Spacing | `docs/design-tokens.md#spacing` | n/a |

> _Remove any rows that don't apply. Add rows as needed._

---

## Interface

The props, attributes, and events that define how this component is used.
The agent will derive the implementation interface directly from this section.

### Props / Attributes

| Name | Type | Required | Default | Description |
|------|------|:--------:|---------|-------------|
| `prop1` | string | ✓ | — | … |
| `prop2` | boolean | | `false` | … |

### Events / Callbacks

| Event | Fires when | Payload |
|-------|------------|---------|
| change / onChange / on:change | … | `{ value: string }` |

### Public methods _(if applicable)_

Some components expose methods that a parent can call directly — for example, a `Modal` that exposes `open()` and `close()`, an `Input` that exposes `focus()`, or a `Form` that exposes `reset()`. How these are accessed depends on the active framework:

- **Vanilla / plain JS** — returned from the component's constructor or initialisation function
- **React** — exposed via `useImperativeHandle` and accessed through a `ref`
- **Svelte** — exported functions accessed via `bind:this`

If this component exposes no public methods, write `None` so it is clear the question was considered. Omit the section only if this is a layout or page spec where the concept does not apply.

**Example — methods present:**

| Method | Description | Parameters | Returns |
|--------|-------------|------------|---------|
| `open()` | Opens the modal | — | `void` |
| `close()` | Closes the modal | — | `void` |
| `focus()` | Moves focus to the first input | — | `void` |

**Example — no methods:**

None.

---

## Behaviour

How the component behaves on first render, across its different states, and in
response to user interaction.

### Default / initial state

Describe exactly what the component renders and does on first mount with only
the required props supplied.

### States

| State | Trigger | Visual / functional result |
|-------|---------|---------------------------|
| state-a | … | … |
| state-b | … | … |

### Interaction rules

- When the user does **X** → **Y** should happen
- When condition **Z** is true → component should …

---

## Accessibility

The accessibility requirements this component must meet. The agent must not
consider the component complete until all of these are satisfied.

- Keyboard navigation requirements
- Required Accessible Rich Internet Applications (ARIA) roles, labels, and live regions
- Text alternatives for any images or icons the component renders
- Focus management after actions
- Contrast and motion requirements
- Minimum target size for anything operated by pointer, and whether a focused
  control can be obscured by sticky content

---

## Error handling

How the component should behave when something goes wrong.

- What happens if required props are missing or invalid?
- What happens when an async operation fails?

---

## Styling notes

Token categories and CSS patterns specific to this component.
The agent must read `docs/design-tokens.md` before writing any styles.

- **Tokens used** — list the token categories this component draws from (e.g. colour, spacing, typography)
- **CSS patterns** — note any conventions specific to this component (e.g. Block-Element-Modifier (BEM) class names, custom properties)
- **Dark mode** — note any theme-specific overrides if applicable

---

## Test cases

The agent will generate one test function per entry, **once the project has a
test runner** — a real `Test` command under Commands in `AGENTS.md` is the
switch, and `/tests` is what adds one. Unit testing ships as `None`, so on a
fresh clone these entries are the contract for what must eventually be proven,
not work the agent can do yet; it will say so rather than claim a case was
tested. IDs must be unique within this spec and must match the test file exactly.

### Render

- [ ] **TC-01** — Renders with only required props
- [ ] **TC-02** — …

### Interaction

- [ ] **TC-03** — …

### Accessibility

- [ ] **TC-04** — …

### Edge cases

- [ ] **TC-05** — …

---

## Out of scope

Explicitly listing what this component does NOT cover prevents scope creep and
helps the agent avoid building more than is required.

- …

---

## Example usage

Include only the example for your active framework. Check `docs/project-brief.md` → Stack
for the active selection.

**Vanilla:**

```html
<div class="component-name">…</div>
```

<!--
**Astro:**

```astro
<ComponentName prop1="value" />
```

**React:**

```jsx
<ComponentName prop1="value" onChange={handleChange} />
```

**Svelte:**

```svelte
<ComponentName prop1="value" on:change={handleChange} />
```
-->

---

## Notes for "Ai" implementation

Additional context, constraints, and implementation guidance that the agent
should be aware of before writing any code.

- **CSS** — import from `./[ComponentName].css` or `./[ComponentName].scss` depending on the active styles selection in `docs/project-brief.md`
- **Assets** — note any SVGs or other assets to inline rather than reference via `<img>`
- **Related specs** — link to any specs this component appears in or depends on
- **Gotchas** — anything non-obvious about the implementation that the agent should know upfront

---

## Draft → Ready checklist

Complete every item before changing the status to `Ready`.
**Agents must not begin implementation until the status is `Ready`.**

- [ ] Purpose describes the *why*, not the *how*
- [ ] All props are named, typed, and have a default where applicable
- [ ] All events include a payload description
- [ ] Every meaningful state is listed in the States table
- [ ] Interaction rules cover all non-obvious behaviours
- [ ] Accessibility requirements are specific, not generic
- [ ] At least one test case exists per behaviour and state
- [ ] Out of scope section is filled in
- [ ] Every dependency that has a spec of its own has reached `Ready`
- [ ] Example usage matches the active framework
- [ ] Notes for "Ai" implementation are complete
