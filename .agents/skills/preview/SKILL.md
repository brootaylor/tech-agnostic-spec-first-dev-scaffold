---
name: preview
description: "Read-only preview of a spec before you build it - the next Ready spec, or a named one whatever its status: what it involves, depends on, and touches, how big it is, and what still blocks it. Use when the user runs /preview, asks what the next feature involves, wants to preview a spec before /feature, asks why a Draft spec is not ready, or is deciding what to build next."
---

# preview - understand a spec before you build it

Where this sits in the workflow:

    docs/specs/*.spec.md  ->  [preview]  ->  /feature  ->  /implement
    (the contract)            (read-only      (sequence     (build it)
                               explainer)      it)

This skill answers one question: *what does this spec actually involve, before I
commit to building it?* It reads the spec and its surrounding context and prints a
short preview so you can decide whether to build it now, reorder it, split it, or
clear a blocker first. It is the read-only precursor to `/feature`.

It is also the right tool for a `Draft` spec. `/feature` refuses to act on one;
`/preview` will happily read it and tell you what is missing before it can be
promoted to `Ready`.

It never writes anything: no spec edits, no status changes, no branch, no commit.

How it differs from its neighbors:

- `/status` reports the *whole project*: the spec queue, current work, git, next
  action. `/preview` zooms into *one spec* and explains it in depth.
- `/feature` *writes* the work order at `current-feature.md`. `/preview` shows
  what `/feature` would tackle, changing nothing.

## Input

A spec, by name or path - e.g. `/preview "dark mode"`, `/preview button`, or
`/preview docs/specs/components/button.spec.md`.

**With no argument, preview the next one** - the first `Ready` spec in
`docs/features/`, then `docs/specs/`, the same target `/feature` would pick.

Unlike `/feature`, this skill reads a spec at **any** status. Briefing a `Draft`
is one of its most useful jobs. Always state the status up front, and for a
`Draft` say what would have to change for it to be promoted.

`docs/features/_feature-template.md` and `docs/specs/_component-template.spec.md`
are not briefable targets. Both carry a `**Status:** Draft` line of their own, and
neither is a work item - skip them when picking a target, and say so if one is
named directly.

If no spec exists yet, or every spec is a bare copy of one of those templates, say
so plainly and point at writing one rather than inventing a preview.

## What it reads

Gather these, then synthesize. Don't dump file contents; explain.

1. **The target spec** - in full: its status, interface, behaviour, states,
   accessibility requirements, and test cases, plus the specs it lists as
   dependencies.
2. **Dependency statuses** - the `**Status:**` line of every spec this one depends
   on, whatever its type. A `Ready` spec resting on a `Draft` one - a feature on a
   component, a page on a layout, a component on another component - is the single
   most useful thing this preview can surface.
3. **Project context** - `docs/project-brief.md` for stack, conventions, browser
   targets, and accessibility standard; the parent feature spec in
   `docs/features/` for product context when the target is a component spec.
4. **What already exists** - specs already marked `Complete`, the state of `src/`,
   and if useful git history, to ground the dependency read (what must be in place
   first, what this unblocks later).
5. **Design reference** - if `prototypes/` exists and the spec is UI-facing, note
   which mockups apply. `docs/design-tokens.md` is the durable source for colour,
   spacing, and type - but it ships complete, so read its `**Last updated:**`
   line before reporting that the values exist. A placeholder comment there means
   the palette is the scaffold's baseline and nobody has settled it, which is a
   blocker worth naming rather than a reference to lean on.

## Output

A short, scannable preview, not a wall of text. Aim for something like:

    docs/features/dark-mode.md - Dark mode
    Status: Draft - /feature will refuse this until it is promoted to Ready.
    What: users switch between light and dark themes; the choice persists and
      respects the operating system setting on first visit.
    Depends on: theme-toggle.spec.md (Draft) - the one component in its
      Components required table. Two specs need promoting, not one.
    Unblocks: nothing else currently specced.
    Touches: the dark palette in tokens.css, a toggle component in the header,
      and localStorage under the color-scheme key.
    Size: medium - one reviewable cycle once theme-toggle is settled.
    Reference: docs/design-tokens.md defines both palettes, but its Last updated
      line is still the placeholder - those are the scaffold's baselines, not
      settled values. The toggle only swaps data-theme and defines no colours,
      so it is not blocked; the first spec that styles anything will be.
    Missing before Ready: one unticked row in the Draft -> Ready checklist -
      "Every required component has reached Ready".

    Next: finish and promote docs/specs/components/theme-toggle.spec.md, then
    this one, then /feature.

Note what the shape of that preview is doing: it traces every line back to a
file. The dependency list comes from the spec's own table - **Components
required** in a feature spec, **Dependencies** in a component, page, or layout
spec - not from guessing what the spec sounds like it needs, and the
"Missing before Ready" line quotes the spec's checklist rather than paraphrasing
what looks unfinished.

Adapt the lines to the spec; drop any that don't apply. Always end with a single
**Next** action - usually `/feature <spec>` to build it, but "finish and promote
the spec" when it is `Draft`, `/prototype` if it's UI-facing and the look isn't
locked, or "clear X first" when a dependency blocks it.

## Rules

- **Read-only, always.** Never write a file, never touch a spec's `**Status:**`
  line, never edit `current-feature.md`, never branch, commit, install, or build.
  To act on the preview, the user runs `/feature` next.
- **Explain, don't sequence.** Size, dependencies, and a likely split are the
  value here; the actual build steps are `/feature`'s job. Don't write step lists.
- **Trace to the spec.** Everything in the preview comes from the spec, its
  dependencies, and `docs/project-brief.md`. Don't invent scope; if something is
  underspecified, say so and name the section that needs it.
- **Always report the status, and lead with it.** It determines whether anything
  can be built at all.
- **Be honest about gaps.** If a spec is `Draft`, a dependency is `Draft`, design
  tokens are missing, or a prerequisite isn't built yet, say that plainly.
  Catching a blocker before building is half the value.

## Formatting

Output follows `AGENTS.md` - Output conventions.
