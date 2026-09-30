# Feature: [Feature Name]

**Status:** Draft
**Last updated:** <!-- e.g. 2025-01-15 -->

> See `docs/project-brief.md` → Spec conventions for the full status key and agent behaviour rules.

---

## Is this the right template?

| You are describing | Use |
|--------------------|-----|
| Something a **user can do**, and why it matters to them | **This template** → `docs/features/<name>.md` |
| A reusable piece of user interface that **helps deliver** that | `docs/specs/_component-template.spec.md` → `docs/specs/components/<name>.spec.md` |
| A **whole screen** built from several components | `docs/specs/_component-template.spec.md` → `docs/specs/pages/<name>.spec.md` |
| A **page structure** that wraps other content | `docs/specs/_component-template.spec.md` → `docs/specs/layouts/<name>.spec.md` |

The test: can you write it as *"As a user, I want… so that…"*? If yes, it is a
feature. If the honest answer is *"nobody wants this on its own, they want the
thing it enables"*, it is a component.

A feature is not a unit of code — it is a unit of user value. One feature
usually needs several components; one component often serves several features.

> _Delete this section once you know which template you are in._

---

## Overview

> A short paragraph describing what this feature lets users do and why it is
> worth building. Describe the outcome, not the implementation.

_Replace this with your feature's overview._

---

## Key

A reference for the prefixes used throughout this document.

| Prefix | Stands for | Description |
|--------|------------|-------------|
| `US-##` | User Story | A feature requirement from the user's perspective |
| `AC-##` | Acceptance Criteria | A condition that must be met for the story to be complete. Numbered continuously across the whole feature, not restarted per story, so each ID names one criterion wherever a report cites it |

---

## User stories

Each user story describes a requirement from the user's perspective, followed by
the acceptance criteria that must be met for it to be considered complete.
`/check` proves the finished work against these criteria, so write them as
things that can be observed in the running application — not as internal
implementation details.

### US-01 — [Short title]

> As a [type of user], I want [capability], so that [benefit].

- [ ] **AC-01** — [An observable condition that must be true]
- [ ] **AC-02** — …

### US-02 — [Short title]

> As a [type of user], I want [capability], so that [benefit].

- [ ] **AC-03** — …

---

## Out of scope

Explicitly listing what this feature does NOT cover prevents scope creep and
helps the agent avoid building more than is required.

- …

---

## Components required

The following must have a spec in `docs/specs/` before an agent can begin
implementation. Every row must reach `Ready` first — a feature cannot be built
on a `Draft` component.

| Component | Spec | Status |
|-----------|------|--------|
| `[ComponentName]` | `docs/specs/components/[name].spec.md` | Draft — must be `Ready` before implementation begins |

> _If this feature needs no new components, write "None" so it is clear the
> question was considered._

---

## Implementation notes

Authoritative decisions that component specs and implementation code must
follow. This is the single source of truth for any value more than one component
has to agree on — attribute names, storage keys, precedence order, route paths,
event names.

| Decision | Value |
|----------|-------|
| [What is being decided] | `[the value]` |

> [!IMPORTANT]
> Component specs must **reference** this table, never restate its values. A key
> or attribute name copied into a component spec is a silent failure: both files
> read as correct, nothing errors, and they drift apart the first time one is
> edited alone.

---

## Notes for "Ai" agents

Additional context, constraints, and guidance the agent should be aware of
before writing any code.

- **Related specs** — link to features or components this one interacts with
- **Constraints** — anything the implementation must or must not do
- **Gotchas** — anything non-obvious the agent should know upfront

---

## Draft → Ready checklist

Complete every item before changing the status to `Ready`.
**Agents must not begin implementation until the status is `Ready`.**

- [ ] Overview describes user-visible outcome, not implementation
- [ ] Every user story follows the "As a… I want… so that…" shape
- [ ] Every acceptance criterion is observable in the running application
- [ ] Out of scope section is filled in
- [ ] Every required component is listed, with a spec path
- [ ] Every required component has reached `Ready`
- [ ] Any value shared across components is fixed in Implementation notes
- [ ] Notes for "Ai" agents are complete
