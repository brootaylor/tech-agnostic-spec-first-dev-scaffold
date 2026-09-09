---
name: feature
description: Turn a Ready spec into a buildable work order. With no argument, picks the next spec marked Ready in docs/features/ or docs/specs/; given a name or path, uses that one. Refuses Draft specs and will not re-implement Complete ones. If a request has no spec yet, offers to draft one as Draft for the human to review and promote. Sizes the work, writes small reviewable build steps to context/current-feature.md, then red-teams its own draft for gaps, oversized steps, and scope creep before stopping at a review gate. Use when the user runs /feature, names a spec or feature, asks to start the next feature, or asks to break down or start work on a spec.
---

# feature - turn a Ready spec into a buildable work order

Where this sits in the workflow:

    docs/specs/*.spec.md  ->  [this skill]  ->  current-feature.md  ->  /implement
    (human-owned contract,     (sequence it        (disposable work        (build it)
     Status: Ready)             into steps)         order)

**The work queue is the `**Status:**` line.** Every spec in `docs/features/` and
`docs/specs/` carries `Draft`, `Ready`, or `Complete`. A `Ready` spec is work
waiting for an engine, and this skill is that engine: it reads one `Ready` spec
and sequences it into small, reviewable build steps.

The spec says *what* and *why*. This skill decides *in what order*, and writes
that ordering to `context/current-feature.md` as a disposable work
order. The spec is the contract; the work order is scaffolding that gets archived
at `/complete`.

## Input

A spec, by name or path - e.g. `/feature "dark mode"`,
`/feature button`, or `/feature docs/specs/components/button.spec.md`.

The request may also describe something with no spec yet. That goes through the
**no-spec intake** in Step 1. Never silently invent scope.

**With no argument, take the next Ready spec.** Scan `docs/features/` first, then
`docs/specs/` (components, pages, layouts, hooks), and pick the first spec whose
`**Status:**` is `Ready`. Feature specs come first because they name the
components they depend on, so a `Ready` feature usually implies the component
work under it.

If more than one spec is `Ready` and the order is not obvious, list them and ask
which to build rather than guessing.

## Step 1 - pick the target spec

- Given a name or path matching a spec -> use it, subject to the status gate
  below.
- No argument -> take the first `Ready` spec, per the Input rules above.
- Given a request with no matching spec -> follow **no-spec intake** below.

### The status gate

This is not advisory. It is the project's core rule, stated in
`docs/project-brief.md`.

| Status | What this skill does |
|--------|----------------------|
| `Draft` | **Stop.** Do not spec, do not build. Report which sections look incomplete and ask the human to finish the spec and set it to `Ready`. |
| `Ready` | Proceed. |
| `Complete` | **Stop.** Do not re-implement or overwrite. Tell the user the human must update the spec and reset it to `Ready` first. |

If the target spec depends on another spec that is still `Draft`, stop and say
which one blocks it. This applies to every spec type, not just features: a page
resting on a `Draft` layout, or a component resting on a `Draft` component, is as
blocked as a feature resting on a `Draft` component. Nothing can be built on an
unfinished contract.

Read the dependencies from the target's own table — **Components required** in a
feature spec, **Dependencies** in a component, page, or layout spec — and check
the `**Status:**` line of each spec named there. Rows with no spec of their own
(assets, tokens, third-party utilities) don't gate anything.

If no spec is `Ready` anywhere, say so and point at the `Draft` specs that are
closest to done, rather than inventing work.

### No-spec intake

Use this path for a new capability with no spec, not a bug or a small unplanned
change. Those still belong in `/fix`.

1. Search `docs/features/` and `docs/specs/` for an existing or near-duplicate
   spec. If the wording may simply be a mistaken name, show the closest matches
   instead of creating new scope.
2. If it is genuinely new, **stop and say a spec is needed first.** Specs are
   human-owned contracts; this skill does not get to invent one and then build
   against it in the same breath.
3. Offer to draft it, from the template for its kind - the two are different
   shapes and the wrong one produces the wrong document:

   | It is | Template | It lands in |
   |-------|----------|-------------|
   | Something a user can do, and why it matters | `docs/features/_feature-template.md` | `docs/features/<name>.md` |
   | A reusable component, page, or layout | `docs/specs/_component-template.spec.md` | `docs/specs/<kind>/<name>.spec.md` |

   The test: can it be written as *"As a user, I want... so that..."*? If yes it
   is a feature; if nobody wants it on its own, only the thing it enables, it is
   a component. Only draft it on explicit approval.
4. If approved, write the draft with `**Status:** Draft`, never `Ready`. Then
   stop. The human reviews it, fills the gaps, and promotes it to `Ready`. Only
   then does this skill run again against it.
5. If the new work materially changes product direction, users, data, stack,
   or deployment, say which sections of `docs/project-brief.md` need updating.
   Do not edit that file yourself.

**Promoting a spec to `Ready` is a human act.** That is the whole point of the
status machine: it is the human's signal that the contract is settled. An agent
that writes a spec and marks it `Ready` has removed the gate the project is
built around.

State which spec you're building, and its status, before going further.

## Step 2 - size it, and split if too big

Read the target spec in full. Its interface, behaviour, states, accessibility
requirements, and test cases are the contract, and they carry more detail than a
roadmap line ever would. Then pull surrounding context:

- `docs/project-brief.md` - stack, conventions, browser targets, accessibility
  standard, agent rules
- the parent feature spec in `docs/features/`, when the target is a component,
  page, or layout spec - for product context, and for the Implementation notes
  table that fixes any value the components must agree on
- any specs the target names as dependencies - the **Components required** table
  in a feature spec, the **Dependencies** table in a component, page, or layout spec

Decide how big the work is:

- **Small enough to build and review as one unit** -> one spec. Continue to
  Step 3.
- **Too big for one reviewable cycle** -> it usually means the spec covers
  several independent contracts. Say so, name the natural split (title + one line
  each), and let the user decide. Do not split the spec file yourself: propose
  either separate specs for the human to write, or an agreed ordering where you
  build one slice of this spec now and the rest in later runs. Record the slice
  boundary in the work order's scope section so the deferred part is explicit.

Two levels of breakdown - don't confuse them:

- **Separate specs** - each is a standalone contract with its own status line,
  review cycle, and archive entry.
- **Build steps** (in the work order, Step 3) - small diffs *within* one spec.

Worked example - a feature spec for "Authentication" covers registration, login,
and route protection. Those are three contracts, not one. Propose three specs
under `docs/features/`, build the first once the human marks it `Ready`. Then
*within* registration, the build steps are small: first "registration page UI",
then "register action + validation + redirect". The page and its logic are steps,
not separate specs.

This sizing call is the skill's job. The spec states the contract; how many
reviewable diffs it takes to satisfy it is a build decision.

## Step 3 - write the work order

For the one spec being built now, write a work order to
`context/current-feature.md` (create `context/` if needed), following
`reference/feature-work-order-template.md`. Fill every section: goal, design reference
(when the feature has a visual target), in/out of scope, the build loop, small
build steps as a checklist (`- [ ]`, each with an observable "done when" -
`/implement` ticks them off and resumes from the first unchecked one),
files/areas, data/contracts, testing, and notes for the "Ai".

**Record the source spec at the top of the work order**, as a `Spec:` line with
its path. `/implement` reads it as the contract, `/check` proves the work against
its acceptance criteria, and `/complete` writes `Status: Complete` back to it.
Without that line the chain breaks.

Do not restate the whole spec here. The work order carries sequencing, files, and
done-whens; the spec remains the authority on interface, behaviour, states,
accessibility, and test cases. Where the two would disagree, the spec wins and
the work order is wrong.

**Derive the done-whens from the spec's own acceptance criteria and test cases**
rather than writing fresh ones. That is what makes `/check` able to prove the work
against the contract instead of against your paraphrase of it.

**Visual or replication features need a reference image.** If the feature is
"make it look like X" - recreating an existing design, matching a mockup, or
rebuilding a Canva/Figma artifact - prose underspecifies the target and the build
will approximate it wrong. Ask the user for a screenshot or image if one isn't
already provided, save it under `docs/reference/` (create the folder if
needed), and link it from the **work order's** Design reference section. That
section is in the work order, not in the spec - this skill never edits a spec
file, and `/implement` reads the work order to find the visual target. Don't
write a visual spec from words alone when an image could exist.

**If `prototypes/` exists, that is your design reference.** When `/prototype` has
run, the repo holds `prototypes/theme.css` (the locked design tokens) and
`prototypes/*.html` (the visual mockups). For a UI-facing feature, link the
relevant mockups from the work order's Design reference section instead of
asking for a screenshot - they beat a flat image, since they carry the exact
tokens. Treat `theme.css` as the source of truth for colors, type, and spacing,
and make the
feature's **first build step** port those tokens into **both** places before
building components against the mockups:

1. `docs/design-tokens.md` - the durable token document. Every agent is told to
   read it before writing any CSS, so a theme that never lands here leaves that
   instruction pointing at the scaffold's shipped baseline palette for the rest
   of the project. The file arrives complete, so nothing looks wrong when this
   step is skipped - the step's "done when" has to name matching values and a
   dated `**Last updated:**` line, not merely a non-empty file.
2. the app's global stylesheet - `src/styles/tokens.{css|scss}`, where the values
   actually resolve at runtime.

Write that step's "done when" so it names both files. `/complete` deletes
`prototypes/` once the look is built, and `theme.css` lives nowhere else until
this step has run - the mockups are throwaway, the tokens are not.

This is a draft. Don't present it yet - critique it first.

## Step 4 - red-team the draft, then tighten

Before the user reads it, turn on the spec yourself and try to break it. The
cheapest place to catch a scope problem or an oversized step is here, before any
code exists. Run the draft against these questions:

- **Coverage.** What does this feature need that no step delivers? Push on the
  unhappy paths the happy-path spec skipped: empty / missing / malformed input,
  the error / loading / empty states, the first-run case, failure of anything
  external it calls.
- **Visual fidelity.** If this is a look-alike or replication feature, is a
  reference image linked in the work order's Design reference - or are we about
  to build a design blind from prose? If `prototypes/` exists, are the relevant
  mockups linked there, and does the first build step port `theme.css` into *both*
  `docs/design-tokens.md` and the app's global stylesheet?
  If a real design exists and nothing is captured, get it before building, not
  after the approximation lands.
- **Step size.** Would any step's diff be too big to read in one sitting? If so,
  split it - oversized steps defeat the review gate.
- **Order.** Does each step leave the app working, and depend only on earlier
  steps, never a later one? Resequence if not.
- **Contracts.** Is any type, route, or stored shape that a later feature will
  touch left undefined here? Lock it now and flag it load-bearing.
- **Scope honesty.** Is anything creeping in that belongs to a later feature? Is
  anything pushed to "out of scope" that this feature actually can't ship without?
- **Done-whens.** Is each one observable and checkable by `/check`, or is it a
  vague "it works"? Make it concrete.
- **Testing.** Does the predicted coverage match the gate - in-scope logic gets a
  test when a `Test` command is declared in `AGENTS.md`, UI/integration rides on
  screenshot + build? Does it cover the spec's own **Test cases** section?
- **Fidelity to the contract.** Walk the spec's interface, behaviour, states, and
  accessibility requirements one by one. Is each one satisfied by some step? A
  state the spec names and no step delivers is the most common miss.

Apply the fixes to `current-feature.md`. Then stop and present the work order, leading
with a short **"what the critique changed"** note - the splits, gaps, or scope
cuts you made (or "nothing - the draft held up"). That note is the point: it shows
the gate working before a line of code is written.

Tell the user to review and adjust. This skill plans; it never starts building.

## Rules the spec must follow

- **Small, reviewable steps.** Each step ends with something working and a diff
  small enough to read in full. If a step's diff would be too big to review, the
  step is too big - split it. This review gate is the point.
- **Build in order.** Sequence the steps so each builds on the last and leaves
  the app working.
- **Lock data contracts early.** If a shape (type, API response, stored field) is
  used by a later feature, define it now and flag it as load-bearing.
- **Flag client vs server** and any conventions from `docs/project-brief.md`
  (for example, filtering user-scoped queries by the authenticated user's id).
- **Scope honestly.** State what is deferred so the feature stays contained.

## When the work is done

`/complete` handles it: it sets the source spec's `**Status:**` to `Complete`,
updates its `**Last updated:**` line, archives the finished work order to
`context/history/features/`. Then run `/feature` again for the next `Ready` spec.

**This skill never edits a spec file.** Not the status line, not the content. The
only exceptions in the whole workflow are `/complete` writing the status back, and
this skill drafting a brand-new spec as `Draft` under the no-spec intake, on
explicit approval.

## Formatting

Format the output to match the project's conventions in `AGENTS.md`: concise,
scannable markdown, with lists for enumerations and tables for matrices rather
than dense paragraphs.
