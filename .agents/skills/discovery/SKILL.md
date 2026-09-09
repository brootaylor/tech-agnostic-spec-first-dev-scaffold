---
name: discovery
description: "Optional deep, multi-turn project discovery that helps the user fill in docs/project-brief.md and draft the first feature specs in docs/features/ through an adaptive conversation, then writes them only after the user says they are ready. Use when the user explicitly runs /discovery, asks for a guided planning interview, wants to think through a new product before writing anything down, or wants help deepening an existing brief. Do not use merely because the brief is unfilled or the project is new; users may always write these files directly, by hand or through any conversation they prefer."
---

# discovery - fill in the brief and the first specs through a deep conversation

Where this can sit in the workflow:

    workflow Step 2 + Step 4  ->  write them by hand  ->  Step 5 (component specs)
                              \
                               ->  [discovery]  ->  review and approve  ->  Step 5

`/discovery` is an optional planning partner, not a required workflow gate and
not a quick questionnaire. It can span as many turns as the project needs. Its
job is to help the user think through the product, preserve the depth and nuance
of that conversation, and write the user-owned planning files only when the user
asks for drafts.

It covers `docs/workflow.md` Steps 2 and 4: the setup checklist in
`docs/project-brief.md`, and the first feature specs in `docs/features/`. It
stops there. Component specs are Step 5, written one at a time against a settled
feature spec, and design tokens are Step 6.

An unfilled brief never requires this skill. A user who fills it in manually,
has another "Ai" conversation, or arrives with finished specs continues straight to
Step 5 exactly as before.

## Step 1 - establish the starting point

Read only what the conversation actually needs:

- `docs/project-brief.md`, especially its setup checklist
- any specs already in `docs/features/` and `docs/specs/`
- the root `README.md` and project manifest when they already contain real facts
- `docs/design-tokens.md` only when the user is revisiting visual direction

Classify each file as an untouched scaffold template, a partial draft, or
substantive user content. The scaffold ships worked examples - `dark-mode.md`,
`button.spec.md`, `theme-toggle.spec.md`, `home.spec.md`, `main-layout.spec.md`
- which are illustrative and safe to replace. Anything the user has written is
not. When either the brief or a spec has real content, summarize what it already
establishes and ask whether the user wants to deepen it, revise a specific
direction, or use it unchanged as conversation context. Never replace it with a
fresh generic version.

Start with a short working hypothesis about the project and name the most
important unknown. Then ask one focused question. Do not draft anything yet.

## Step 2 - run adaptive discovery

Ask one meaningful question at a time and let each answer shape the next one.
Prefer a likely interpretation the user can correct over a vague request for
more detail. Explain a tradeoff when the answer would materially change scope,
architecture, cost, or build order.

Cover the areas that matter to this project, not a fixed questionnaire:

- problem, desired outcome, and why the project should exist
- target users, their context, and their primary workflows
- Minimum viable product (MVP) capabilities, explicit non-goals, and later
  possibilities
- business rules, data, integrations, permissions, and important edge cases
- stack choices, constraints, dependencies, and technical unknowns
- UI/UX direction, accessibility needs, and useful references
- deployment shape, environments, background work, storage, and operations
- risks, assumptions, unresolved decisions, and how success will be judged
- feature boundaries, dependencies, and a sensible build order

The brief's `## Stack` section is a set of exclusive choices - one `[active]`
option per category - so treat an unsettled stack as a real question rather than
a detail to fill in later. The choice constrains every spec written afterward,
and `docs/setup.md` documents the compatibility traps for combinations
that fight each other. `## Browser support` and `## Accessibility standard` have
defaults worth confirming rather than interviewing at length.

Depth is the goal. Follow a consequential answer until its implications are
clear instead of racing to the next category. Do not ask the user to repeat facts
already established in the conversation or repository. Do not force irrelevant
topics merely to complete a checklist.

Periodically return a compact discovery snapshot with:

- confirmed decisions
- working assumptions that still need confirmation
- open questions or conflicts
- ideas explicitly deferred or excluded

The snapshot keeps a long conversation coherent. It is not permission to write.

## Step 3 - decide whether it is ready

Do not end discovery because a preset number of questions has been reached. It
is ready to draft when:

- the problem, users, and core workflows are concrete
- MVP scope and non-goals are distinguishable
- one stack option per category can be marked `[active]` with reasons
- the build order can be expressed as feature-sized outcomes
- important contradictions are resolved
- remaining unknowns are either safe to defer or explicitly accepted as TODOs
- the user says they are ready for the drafts

If the user asks for drafts while a material gap remains, name the gap and ask
whether to continue discovery or preserve it as an explicit TODO. Respect the
choice. The user may also stop at any time and write the files by hand.

## Step 4 - draft the brief and the feature specs

When the user asks for drafts, produce complete proposed contents without
writing them yet.

For `docs/project-brief.md`:

- fill in the four setup-checklist sections and leave the rest of the file alone.
  Everything below the checklist is reference material the scaffold ships, and it
  is not this skill's to rewrite
- under `## What this project is`, preserve the rationale, examples, tradeoffs,
  constraints, edge cases, and exclusions that will matter during later feature
  work. Be as detailed as the project needs
- mark exactly one `[active]` option per `## Stack` category, and say in a line
  why, so a later reader can tell a decision from a default
- update `## Browser support` and `## Accessibility standard` only when the
  conversation established something different from the defaults
- distinguish confirmed decisions from assumptions and TODOs

For each spec in `docs/features/`:

- follow `docs/features/_feature-template.md`: overview, user stories,
  acceptance criteria, the components the feature requires, and the
  Implementation notes table that fixes any value those components must agree
  on. Delete the template's "Is this the right template?" section from each
  draft, as it instructs. `docs/features/dark-mode.md` is a worked example of
  the same shape if one is useful
- describe what a user can do, not how it is built. Interface and behaviour
  detail belongs in the component specs written at Step 5
- set `**Status:** Draft`. Promoting a spec to `Ready` is a human review
  decision, and never one this skill takes
- keep each spec a feature-sized outcome, and order them by dependency and the
  earliest useful vertical slice
- include only agreed scope; leave deferred ideas out or note them explicitly as
  out of scope

If substantive content already exists, preserve its information. Clearly identify
proposed additions, removals, or changed decisions.

End by asking the user to review the full drafts. Do not write anything in the
same response that first presents them.

## Step 5 - write only after approval

Write the approved drafts only after the user explicitly approves them. If the
user requests changes, revise and show the affected sections again before
writing.

After writing:

- report which files changed
- list any retained TODOs or unresolved decisions
- remind the user that every one of these files remains theirs to edit directly
- point at `docs/workflow.md` Step 5 as the next step: a component spec for each
  item in the feature's "Components required" list, using
  `docs/specs/_component-template.spec.md`
- mention that `/brief <spec>` will read back any spec, at any status, and say
  what it still needs before it can be promoted to `Ready`

## Rules

- This skill is always optional. Never make it a prerequisite for `/feature`,
  `/brief`, or anything else.
- Never start it automatically because the brief is unfilled or the project is
  new.
- Never imply that a brief or spec written by hand is inferior merely because
  this skill was not used.
- Never overwrite substantive user content without showing the replacement and
  receiving explicit approval. The scaffold's own worked examples are the one
  safe exception.
- Never write during the interview, or after a vague signal such as "looks good."
  The user must explicitly approve the proposed file contents.
- Never set a spec's `**Status:**` to anything but `Draft`.
- Never scaffold the app, edit product code, write a component spec, define
  design tokens, commit, or push.
- Keep the depth in `docs/project-brief.md` and the feature specs, and keep the
  feature specs about user-visible outcomes.
- Keep the conversation adaptive. Depth comes from relevant follow-up questions,
  not from mechanically asking every possible one.

## Formatting

Format the output to match the project's conventions in `AGENTS.md`. During
discovery, ask one focused question per turn. For snapshots and draft reviews,
use concise headings and lists so confirmed decisions and remaining gaps are easy
to inspect.
