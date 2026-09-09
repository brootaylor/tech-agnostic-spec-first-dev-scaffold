# Fix: <the bug or change, in a few words>

**Type:** Fix
**Base commit:** <filled in by /implement before the first step>
**Work status:** not started
**Fixes:** `<finding id>`

> A fix work order carries **no `Spec:` line**. It has no source spec by
> definition, which is why `/complete` archives it to `context/history/fixes/`
> and writes no status back. Do not add `Spec: none` to fill the gap - a path
> that does not resolve is what `/complete` stops on.
>
> `Base commit:` is the commit this work started from. `/implement` records it
> before the first product edit; `/audit` uses it to find what this work changed.
>
> `Work status:` tracks this work order only. It is not a spec's `**Status:**`
> line and must never be confused with it. `AGENTS.md` lists its five values
> under "The work order's own status"; nothing else is a valid value.
>
> `Fixes:` is only for a fix that repairs a `context/findings.md` entry. The
> stamp makes the repair traceable: `/implement` marks that finding `fixed` when
> the repairing step lands, and `/audit` re-reviews it before it closes. Delete
> the line entirely for a fix that came from a bug report rather than the ledger.

## The problem

What is wrong, or what needs to change, and where. Include how to reproduce it
if it is a bug: the steps, and what happens instead of what should.

## The fix

The approach, in a sentence or two. Then what it must not break - the nearby
behaviour that has to still work when this lands.

## Build steps

Usually one small step. Split only if the diff would be too big to read.
`/implement` ticks these off as it finishes them, so progress survives a context
clear: a fresh session resumes from the first unchecked step.

- [ ] **Step 1 - <step>** - what you change. *Done when:* <observable criteria>.

## Files / areas

- The files this touches. A fix that reaches beyond two or three is probably a
  feature; stop and write a spec instead.

## Verification

- How to confirm it is fixed: what to click, run, or test.
- Regression path: `<a small nearby flow that must still work>`
- If a test runner is configured and this fix touches in-scope logic (parsers,
  formatters, validators, data transforms), the fix ships with a test that fails
  before it and passes after. The testing gate is on only when `AGENTS.md` lists
  a real `Test` command under Commands.

## Notes for the "Ai"

- Conventions and constraints to respect, and anything about this area of the
  codebase worth knowing before touching it.
- Fix the cause, not the symptom. If the real cause sits outside this scope, say
  so rather than patching over it.
