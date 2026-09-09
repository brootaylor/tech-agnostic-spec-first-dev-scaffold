---
name: check
description: Prove the current work actually does what its spec says by running the real app and observing behavior against the "done when" criteria in current-feature.md. Drives the app (browser, CLI, or server), captures evidence (screenshots, output, console/network errors), and reports pass/fail per criterion. Does not edit source or commit - it observes; fixing stays /implement's job. Use when the user runs /check, asks to confirm a step or feature works, wants proof before /complete, or wants to check a change in the running app rather than just the build.
---

# check - prove it works against the spec, with evidence

Where this sits in the workflow:

    /implement  ->  [check]  ->  /complete
    (built a       (run the app,    (only once the
     step or        prove each       done-whens are
     the feature)   done-when)       proven)

`/implement` builds and does a quick build-plus-screenshot check inline. `/check`
is the deeper, repeatable gate for when a "done when" needs the *real running app*,
not just a green build: a click that triggers a download, a route that returns a
file, a flow across screens. Run it on a single step whose done-when is
behavioral, or on the whole feature as the acceptance check before `/complete`.

The point is evidence. A passing build proves the code compiles; `/check` proves
the thing the spec promised actually happens. It changes no source and commits
nothing - it runs the app and reports what it saw.

## Input

Optional: a specific thing to check (a step, a flow, a URL). With no argument,
verify the whole current feature against every "done when" in
`context/current-feature.md`.

## Step 1 - build the checklist

`/check` runs automatically only when `/implement` or `/complete` judges that a
"done when" needs observed runtime behaviour. An explicit `/check` or `$check`
request always runs. Running it never grants permission to start a server or take
any other action the project instructions or user have not authorized.

Read `context/current-feature.md`, then **the spec named on its `Spec:`
line**. Pull the observable "done when" criteria from the build steps, and pull
the acceptance criteria, states, and **test cases** from the spec itself.

The spec is what the work is actually held to. A work order can paraphrase it
loosely or miss a state; proving only the paraphrase is how a feature passes
`/check` and still fails its contract. Prove both, and if the spec names a state,
behaviour, or accessibility requirement that no work-order criterion covers,
**check it anyway and report the gap**.

Include the spec's accessibility requirements in the checklist: keyboard
navigation, focus handling, and screen reader semantics are observable, and
`docs/project-brief.md` sets a project-wide minimum that applies whether or not
the spec restates it.

Turn all of this into a concrete checklist of claims to prove - each one a
specific, observable behavior, not "it works". If the user named one thing, scope
to that.

If there's no current feature spec, ask what to verify rather than guessing.

## Step 2 - get the app running

Use the project's real commands (see Commands in `AGENTS.md`). Match the project
type:

- **Web app** - start (or reuse) the dev/preview server, then drive a real browser
  to the relevant routes. Prefer reusing an already-running server over starting a
  duplicate. If Playwright is already installed or declared in `AGENTS.md`, prefer
  it for browser driving, screenshots, console errors, and failed request checks.
  If it is not installed, do not add it from `/check`; use another real-browser
  evidence path and report what you used.
- **Command-line interface (CLI)** - run the actual command(s) with
  representative inputs.
- **Server/API** - start it and hit the endpoints.
- **Library** - exercise the public API through an example or the test command.

If a `Test` command is declared in `AGENTS.md`, you may run it as *one* input, but
`/check` is broader than unit tests: it checks real behavior, which is exactly the
evidence UI and integration steps ride on instead of unit tests.

## Step 3 - exercise each claim

Drive the app to each checklist item and capture evidence as you go:

- Navigate and interact for real (click, type, submit, download) - don't assert
  from the code what the running app would do.
- Capture **screenshots** for visual/UI claims, **output** for CLI/API claims.
- Watch for **console errors and failed network requests**; a clean-looking screen
  with errors in the console is not a pass.

Use the strongest evidence available, and report any gap plainly. Where a
browser, CLI, or server can actually be driven, a UI claim needs direct
evidence - a screenshot plus the relevant console and network check. Where that
path is unavailable, mark the claim unverifiable rather than passing it from
build output.

## Step 4 - report

Give a short, honest verdict, one line per checklist item:

    [pass] Download PDF saves certificate-<slug>.pdf - file downloaded, opened to the cert
    [pass] Both buttons show a loading state - screenshot: loading-state.png
    [fail] PDF border missing - printBackground not set; screenshot: pdf-no-border.png
    [skip] Production redirect behaviour - can't verify locally (feature 9)

Then state the bottom line: are all the feature's done-whens proven, or not yet.

- All proven -> update only the `**Work status:**` line in
  `context/current-feature.md` to `verified`, then say it is ready for
  `/complete`.
- Anything failed -> update only that line to `verification failed`, then hand
  back to `/implement`; name what to fix. Do not fix it here.
- Anything unverifiable -> update only that line to `verification incomplete`,
  then say why; never report it as a pass.

`**Work status:**` is the work order's own field. The spec's `**Status:**` line
is a different thing in a different file, and `/check` never touches it. The
update above is generated workflow state, not a product-source edit. Do not
change the spec, checkboxes, findings, or product files from `/check`.

## Rules

- **Observe product behavior, don't repair it.** `/check` changes only the work
  order's `**Work status:**` line as described above. It never edits a spec,
  product source, commits, or merges. Fixing is `/implement`'s job.
- **Evidence or it didn't happen.** Every `pass` is backed by something observed -
  a screenshot, output, a response. No assumed passes from reading the code.
- **Honest over green.** "Couldn't verify" and "failed" are valid, useful results.
  Faking a pass defeats the entire gate.
- **Check the spec, not vibes.** Verify against the done-whens in
  `current-feature.md`, so "works" means what the spec said it would do.

## Formatting

Format the output to match the project's conventions in `AGENTS.md`: concise,
scannable markdown, with lists for enumerations and tables for matrices rather
than dense paragraphs.
