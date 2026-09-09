---
name: audit
description: Read-only code audit for a spec-first project, except for the findings ledger it maintains at context/findings.md. Reviews the active feature, changed files, a selected path, or the full project through all concerns or a focused quality, security, performance, accessibility, or tests lens. Use when the user runs /audit, invokes $audit, asks for a code or quality audit, security review, performance review, accessibility or WCAG review, test quality review, dead-code or duplication check, vibe-coded project cleanup, or standards review.
---

# audit - review code quality against the project standards

Where this sits in the workflow:

    /implement  ->  [audit]  ->  fixes or /complete
    (code exists)   (review +    (repair quality issues
                     ledger)      or close the feature)

`/check` proves behavior against the spec. This skill checks the code itself,
through either a broad review or one focused lens: quality, security,
performance, accessibility, or tests.

It reviews code without changing it: it never edits source files, installs
dependencies, commits, merges, pushes, or starts product work. Its one write is
the findings ledger at `context/findings.md` (Step 4), the durable
record of findings and their status.

`/complete` and `/autopilot` invoke this skill automatically in two cases, both
defined in `/complete`'s Quality gates: a full `/audit current` when the work
touches a security boundary - authentication, authorization, payments, secrets,
personal or user data, migrations, destructive operations, or external side
effects - and `/audit current accessibility` when the work added or changed
markup, styles, or design tokens, since that lens is where contrast and
semantics get measured. `/implement` calls it for one narrower reason: to
re-review a repair it marked `fixed`, since a repair never closes itself. An
explicit `/audit` or `$audit` request always runs it, whatever the work touches.

## Input

Treat scope and lens as separate controls. Arguments may appear in either order,
such as `/audit security current` or `/audit src/auth tests`.

Optional scope:

- no scope argument: use `current` when an active feature exists, otherwise use
  `changed` when local changes exist, otherwise use `full`
- `current`: audit the active `current-feature.md`, the commits made for this
  work order, staged and unstaged changes, untracked source files, and nearby
  code affected by the feature
- `changed`: audit staged, unstaged, and untracked source files plus nearby code
- `full`: audit all project-owned source, tests, and configuration while excluding
  dependencies, generated files, build output, coverage output, caches, vendored
  code, and minified assets unless the user explicitly includes them
- path or directory: audit that area and the tests or callers needed to understand it

Optional lens:

- no lens: review all five lenses
- `quality`: maintainability, duplication, dead code, consistency, complexity,
  and standards drift
- `security`: authorization, input trust, injection, data exposure, secret
  handling, and unsafe configuration
- `performance`: query, network, rendering, memory, payload, concurrency, and
  unbounded-work risks
- `accessibility`: the standard set in `docs/project-brief.md` - semantics,
  keyboard operability, focus, names and roles, and colour contrast
- `tests`: missing coverage for important logic, weak assertions, skipped or
  focused tests, poor isolation, brittle mocks, and likely flakiness

`full` is always the full-project scope, not a lens. `/audit full` therefore runs
all lenses across the full project. When only a lens is supplied, select scope
with the normal no-scope rules. A focused pass may name one or more lenses. If
the request names multiple lenses, review their union and report them separately.

If the requested scope is unclear, pick the smallest useful scope and state it.
If the lens is unclear, use all lenses and state that choice.

## Step 1 - gather context

Read:

- `AGENTS.md`
- `context/current-feature.md`
- the spec named on its `Spec:` line, in `docs/features/` or `docs/specs/`. **Audit
  the code against the spec, not only against the work order.** A behaviour,
  state, or accessibility requirement the spec names and the code does not deliver
  is a finding, whether or not any build step mentioned it
- `docs/project-brief.md`, for the conventions, secure-coding rules, browser
  targets, and accessibility standard that apply project-wide
- `docs/security.md`, for the security lens - the project's own declared headers,
  Content Security Policy (CSP) directives, and CSP exceptions table. Read it
  whenever the Security selection in `docs/project-brief.md` is active or the
  scope includes deployment config
- `context/findings.md`, for existing IDs and statuses
- git working tree status
- relevant source files, tests, and configs for the chosen scope

For `current`, resolve the comparison base without network access. The work
order records the commit the work started from; use it and inspect the delta
through `HEAD`, then add staged, unstaged, and untracked work. If the work order
records no starting commit, fall back to the commits made since the last
`/complete` archive, and say which base you used. If no reliable base exists, say
so and use the active spec plus local changes - never claim committed work was
fully covered in that case.

Do not fetch or pull to discover the base. For `full`, state the excluded paths
before reviewing so generated or third-party code does not consume the audit.

Prefer `rg` and targeted file reads. Do not dump large files into the response.

## Step 2 - run available signals

Use existing commands only. Do not install tools.

Run or inspect only the signals relevant to the selected lens and scope:

- lint and typecheck commands when declared and relevant
- test command for the tests lens or when it directly validates a suspected risk
- build command when the selected lens needs compilation or bundle evidence
- existing security command for the security lens, when declared and locally runnable
- existing performance command for the performance lens, when declared and locally runnable
- existing accessibility linter or checker for the accessibility lens, when
  declared and locally runnable. Do not install one; a project without one is a
  gap to report, and static review still applies
- targeted lightweight searches for the chosen lens, such as unused exports and
  copied logic for quality, unsafe trust boundaries for security, repeated or
  unbounded work for performance, non-semantic elements and missing names for
  accessibility, and skipped or weak tests for tests

Do not run broad checks unrelated to a focused lens. If a useful command is
missing, report that as a gap. Do not invent a pass or claim that a focused
review covered the other lenses.

## Step 3 - review the code

For all lenses, ground findings in reachable code and project-specific
expectations. Apply only the selected lens or lenses:

- **Quality:** duplicated logic, dead or unused code, unreachable paths,
  oversized modules, abstractions that do not pay for themselves, risky missing
  abstractions, inconsistent patterns, and drift from the standards or spec.
- **Security:** missing authentication or authorization, client-controlled
  ownership, injection, unsafe parsing or deserialization, sensitive-data
  exposure, secret handling, insecure defaults, and trust-boundary mistakes.
  Also measure the deployed configuration against the project's own contract in
  `docs/security.md`, when the Security selection is active: headers the code or
  provider config omits, weakens, or contradicts, and any external origin the
  code loads that its CSP exceptions table does not list. That file is the source
  of truth for those values, and generic secure-coding review does not reach
  them - a project whose shipped headers disagree with its own security document
  passes every other check in this lens.
  Inspect existing dependency or scanner output when available, but never imply
  that local manifest inspection is a current vulnerability scan.
- **Performance:** N+1 queries, repeated network or database work, unnecessary
  rendering, blocking work on hot paths, unbounded loops or collections, memory
  growth, oversized payloads, missing pagination, and unsafe concurrency. Mark
  hypotheses as unverified when runtime or profiling evidence is missing.
- **Accessibility:** measured against the standard named in
  `docs/project-brief.md`, which applies project-wide whether or not a spec
  restates it. Generic elements carrying behaviour a native element provides,
  interactive controls that are not keyboard operable, focus that is removed
  without replacement or lost after a state change, missing or wrong accessible
  names, state conveyed by colour or icon alone, Accessible Rich Internet
  Applications (ARIA) attributes that contradict or duplicate native semantics,
  text and non-text contrast below the standard's minimums, and animation that
  ignores `prefers-reduced-motion`. Where token
  values are readable, compute contrast rather than estimating it; where a
  judgement needs the running app, say so and hand it to `/check` rather than
  guessing.
- **Tests:** important logic without coverage when a test command exists, weak
  assertions, tests that only mirror implementation, excessive mocking, shared
  state, time or order dependence, skipped or focused tests, placeholder tests,
  swallowed failures, and missing browser or integration evidence where behavior
  crosses a real boundary. Never invent a coverage percentage.

Do not nitpick harmless style differences unless they signal drift from the local
patterns. Prefer a short list of real findings over a broad list of guesses.

Do not broaden a focused pass because another category might be interesting.
Do not report or call out non-critical concerns from omitted lenses, even as
suggestions for a later audit. If an obvious P0 is directly encountered outside
the selected lens, report and record it as an out-of-lens critical risk, but do
not continue searching that other lens.

If a possible secret is found, never quote its value, paste the matching source
line, or include raw command output containing it. Report only the redacted secret
category, file, line, risk, and remediation. Redact sensitive values from all
audit evidence before responding.

## Step 4 - update the findings ledger

`context/findings.md` is the durable record of findings. Chat reports
do not survive a context clear; the ledger does. It is the only file this skill
writes. If it is missing (an older install), create it with a `# Findings`
heading first.

**The ledger never scopes the review.** Review the code fresh in Step 3, then
record what the review found. Working from the open findings as a checklist and
verifying only those is the exact failure this file exists to prevent: a repair
can introduce a new defect that no existing entry points at.

One block per finding. The header line is the machine-readable contract and must
keep this exact shape; the prose below it is for humans and may vary:

    ### F-03 [P0] open - Retained auth volumes carry the run label

    **File:** ops/agent-proof/compose.yaml:86
    **Found:** 2026-07-21 by /audit (scope: current; lens: security)
    **Why it matters:** ...
    **Suggested fix:** ...
    **Resolution:**

IDs are sequential within the ledger (`F-01`, `F-02`, ...), never reused and
never renumbered while their entries live here, even after a finding closes.
Bare IDs are scoped to the live ledger: `/complete` archives resolved entries
under a work-item prefix (feature 12's `F-03` becomes `12/F-03`), and that
prefixed form is the permanent reference. A later ledger that has emptied and
reset starts at `F-01` again without colliding. Severity reuses the P0-P3
scheme from Step 5; only P0 and P1 block `/complete`. Status is one of:

| Status | Meaning | Blocks P0/P1 at /complete |
|---|---|---|
| `unverified` | Suspected, no confirming evidence yet | No |
| `open` | Confirmed, not yet repaired | Yes |
| `fixed` | Repaired, not yet re-reviewed | Yes |
| `closed` | Repaired and re-reviewed against the new code | No |
| `accepted` | Not fixing, by the user's explicit decision; reason recorded in Resolution | No |
| `invalid` | Re-examination proved the finding wrong; evidence recorded in Resolution | No |

After the review:

- Append each new confirmed finding as `open` with the next sequential ID, one
  past the highest ID present in the ledger (entries carried forward from
  earlier work count; a fresh ledger starts at `F-01`).
- Record an unverified risk worth tracking as `unverified`. It is a lead, not a
  defect, and never blocks completion.
- Update the entries this pass re-examined: correct the status or severity and
  note the evidence in **Resolution**.
- Move a `fixed` finding to `closed` only when all three hold: this pass's
  reviewed set included the finding's file, re-examining the repaired code
  confirmed the original defect is gone and the repair introduced no new one,
  and the report names the finding as closed. An unrelated new finding in the
  same file gets its own entry and does not keep the repaired one open. Never
  close a finding implicitly.
- Set `accepted` only on the user's explicit decision in the current session,
  and record their reason. Never accept a finding on their behalf.
- Set `invalid` only when re-examination shows the finding was wrong, and
  record that evidence in **Resolution**. It is a review verdict (or the
  user's explicit call), never a shortcut past the gate for blocked work.

`fixed` blocking `/complete` is deliberate: a repair is not done when the code
changes, it is done when a review has looked at the result. `/implement` marks
repairs `fixed`; only a review pass moves them to `closed`.

## Step 5 - report findings

Lead with findings, ordered by severity, using the IDs the ledger assigned:

    F-04 [P1] Title
    File: path:line
    Why it matters: ...
    Suggested fix: ...

Severity:

- `P0` - data loss, security break, or code that cannot ship
- `P1` - likely bug, broken contract, missing guard, or high-risk duplication
- `P2` - maintainability issue worth fixing before the feature closes
- `P3` - small cleanup, consistency issue, or follow-up candidate

Use P0 or P1 only when a concrete code path, violated contract or security
boundary, failing command or test, or reproducible behavior confirms the risk. If
the evidence is incomplete, list it under `Unverified risks` with the missing
validation instead of presenting it as a confirmed high-severity finding.

If there are no findings, say that clearly for the selected lens and name any
remaining risk or missing signal, such as "no test command declared" or
"browser flow not audited."

Then include:

- ledger changes: findings added, updated, or closed this pass, by ID
- commands run and results
- selected scope
- selected lens or lenses
- comparison base and commit range for `current`, when available
- files or directories reviewed
- generated, third-party, or otherwise excluded paths
- applicable standards checked
- browser or runtime evidence inspected, when relevant
- skipped, focused, or placeholder tests found, when the tests lens was selected
- accessibility findings that need the running app to settle, when the
  accessibility lens was selected, named as work for `/check`
- checks that were unavailable or could not run
- suggested repair order

For `full`, say whether coverage was complete or partial. Never label a partial
review as a full-project audit.

## Rules

- The findings ledger is the only file this skill writes. Never edit, format,
  install, commit, merge, push, or delete anything else.
- A focused lens is not a broad audit. State what was not reviewed and never
  imply that omitted lenses passed.
- The ledger reports status; it never defines what the review looks at. Do not
  turn open findings into the review checklist.
- Never fetch, pull, or run network-backed audit tools without explicit approval.
- Never reproduce secrets or sensitive values in findings or command output.
- Findings first. Keep summaries short.
- Ground every finding in a file path and line number when possible.
- Avoid speculative rewrites. Recommend the smallest fix that removes the risk.
- Respect existing project patterns over generic advice.
- Do not require perfection. The goal is code that is understandable, consistent,
  testable where it matters, and safe to keep building on.

## Formatting

Format the output to match the project's conventions in `AGENTS.md`: concise,
scannable markdown, with lists for enumerations and tables for matrices rather
than dense paragraphs.
