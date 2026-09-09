# Feature: <name>

**Type:** Feature
**Spec:** `docs/specs/components/<name>.spec.md`
**Base commit:** <filled in by /implement before the first step>
**Work status:** not started

> `Type:` is what `/complete` reads to decide which folder under
> `context/history/` this archives to, and what `/status` reports. Declare it
> rather than leaving it to be inferred from the absence of `Type: Fix` and
> `Type: Rollback`.
>
> `Spec:` is load-bearing. `/implement` reads that file as the contract, `/check`
> proves the work against its acceptance criteria, and `/complete` writes
> `**Status:** Complete` back to it. A `Type: Fix` work order has no source spec
> and omits this line entirely, rather than setting it to `none` - see
> `../../fix/reference/fix-work-order-template.md`.
>
> `Base commit:` is the commit this work started from. `/implement` records it
> before the first product edit; `/audit` uses it to find what this work changed.
>
> `Work status:` tracks this work order only. It is not the spec's `**Status:**`
> line and must never be confused with it. `AGENTS.md` lists its five values
> under "The work order's own status"; nothing else is a valid value.

## Goal

What this feature delivers, in a sentence or two. Why it matters.

## Design reference

For a visual or replication feature (recreating a design, matching a mockup),
link the reference image(s) here, stored in `docs/reference/`. A screenshot
pins down what prose can't, so build against it, not a guess. Omit this section
when the feature has no visual target.

## In scope

- The specific things this feature includes.

## Out of scope

- What it deliberately doesn't touch (deferred to a later feature).

## Build loop

Build one step at a time, never the whole feature at once.

1. Plan mode lays out the step before any code.
2. The "Ai" implements just that step.
3. It shows the diff (not full files); you read it and understand it.
4. You approve, then choose whether to commit a checkpoint or roll straight on.
   Checkpoints are optional; `/complete` makes the real feature-level commit at the end.

Never accept a step you haven't read. If a diff is too big to review, the step was too big, so split it.

## Build steps

Small, reviewable units. Each ends with something working. `/implement` checks
these off as it finishes them, so progress survives a context clear: a fresh
session reads which boxes are ticked and resumes from the first unchecked step.

- [ ] **Step 1 - <step>** - what you build. *Done when:* <observable criteria>.
- [ ] **Step 2 - <step>** - what you build. *Done when:* <observable criteria>.

## Files / areas

- The files or modules this will create or change.

## Data / contracts

- Schema, types, or API shapes involved, or "none yet."

## Testing

- How to verify: what to click through, and the observable done-when per step.
- If a test runner is configured, name the in-scope logic that needs a test
  (parsers, formatters, validators, data transforms - not components or
  integration/render routes), so each logic-bearing step ships its test. If no
  runner is configured, say so and rely on screenshot plus build evidence. The
  testing gate is on only when `AGENTS.md` lists a real `Test` command under
  Commands; `/tests` adds one.

## Notes for the "Ai"

- Conventions and constraints to respect (e.g. client vs server, filter user-scoped queries by the authenticated user's id, match an existing data shape).
