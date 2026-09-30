# Feature: Dark Mode

**Status:** Draft
**Last updated:** <!-- e.g. 2025-01-15 -->

> See `docs/project-brief.md` → Spec conventions for the full status key and agent behaviour rules.

---

## Overview

Allow users to switch between light and dark colour schemes. The preference should persist across sessions and respect the user's operating system setting by default.

---

## Key

A reference for the prefixes used throughout this document.

| Prefix | Stands for | Description |
|--------|------------|-------------|
| `US-##` | User Story | A feature requirement from the user's perspective |
| `AC-##` | Acceptance Criteria | A condition that must be met for the story to be complete |

---

## User stories

Each user story describes a requirement from the user's perspective, followed by the acceptance criteria that must be met for it to be considered complete.

### US-01 — System preference

> As a user, I want the application to match my operating system colour scheme
> by default, so I don't have to configure it manually.

- [ ] **AC-01** — On first visit, the application detects the OS colour scheme via `prefers-color-scheme` and applies it automatically
- [ ] **AC-02** — No preference is stored until the user makes a manual selection

### US-02 — Manual toggle

> As a user, I want to manually switch between light and dark mode, so I can
> override my OS setting when I choose to.

- [ ] **AC-03** — A toggle is accessible from every page
- [ ] **AC-04** — Switching applies immediately without a page reload
- [ ] **AC-05** — The selected preference is saved and restored on return visits

### US-03 — Persistence

> As a returning user, I want my colour scheme preference to be remembered, so
> I don't have to set it every time I visit.

- [ ] **AC-06** — Preference is stored in `localStorage` under the key `color-scheme`
- [ ] **AC-07** — Stored preference takes priority over the OS setting on return visits
- [ ] **AC-08** — Clearing browser storage resets the preference to OS default

---

## Out of scope

Explicitly listing what this feature does NOT cover prevents scope creep and helps the agent avoid building more than is required.

- Per-page or per-component colour scheme overrides
- High contrast mode
- Custom colour themes beyond light and dark

---

## Components required

The following components must have a spec in `docs/specs/components/` before an agent can begin implementation.

| Component | Spec | Status |
|-----------|------|--------|
| `ThemeToggle` | `docs/specs/components/theme-toggle.spec.md` | Draft — must be `Ready` before implementation begins |

---

## Implementation notes

These are authoritative decisions that component specs and implementation code must follow. Do not redefine these values elsewhere — reference this file instead.

| Decision | Value |
|----------|-------|
| Theme attribute | `data-theme` on the root `<html>` element |
| Attribute values | `"light"` and `"dark"` |
| localStorage key | `color-scheme` |
| Priority order | Stored preference → OS preference → default |
| Default theme | `light` — a component may make this tier configurable, as `theme-toggle.spec.md` does with its `initialTheme` prop; the order above is not negotiable |

---

## Notes for "Ai" agents

Additional context, constraints, and implementation guidance that the agent should be aware of before writing any code.

- Colour scheme is applied via a `data-theme` attribute on the root `<html>` element, toggled between `"light"` and `"dark"`
- All colour values must reference CSS custom properties — no hardcoded colours anywhere in the codebase
- The toggle must be keyboard accessible and meet Web Content Accessibility Guidelines (WCAG) 2.2 AA contrast requirements
- The `color-scheme` localStorage key and the Priority order are defined here and are the single source of truth — component specs must reference this file rather than restate either

---

## Draft → Ready checklist

Complete every item before changing the status to `Ready`.
**Agents must not begin implementation until the status is `Ready`.**

- [x] Overview describes user-visible outcome, not implementation
- [x] Every user story follows the "As a… I want… so that…" shape
- [x] Every acceptance criterion is observable in the running application
- [x] Out of scope section is filled in
- [x] Every required component is listed, with a spec path
- [ ] Every required component has reached `Ready`
- [x] Any value shared across components is fixed in Implementation notes
- [x] Notes for "Ai" agents are complete

> This spec is `Draft` for exactly one reason: `ThemeToggle` is still `Draft`, so
> the unchecked row above is the blocker. Promote that component spec first, then
> this one. `/preview docs/features/dark-mode.md` reports the same thing.
