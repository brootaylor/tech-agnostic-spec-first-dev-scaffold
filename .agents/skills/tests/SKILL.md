---
name: tests
description: Add or normalize unit testing for a spec-first project. Detects the stack, reuses an existing test runner when present, or installs the stack-native unit test runner when missing, then adds one small example test, updates AGENTS.md commands, runs build and tests, and reports the diff. Use when the user runs /tests, invokes $tests, asks to add unit tests, set up unit testing, configure tests, or make tests part of the build workflow.
---

# tests - add unit testing to the project

Where this sits in the workflow:

    any time  ->  [tests]  ->  test command in AGENTS.md  ->  /feature + /implement use it
                  (setup)     (the opt-in testing gate)      (logic steps get tests)

Testing is optional until the project declares a real test
command in `AGENTS.md`. This skill is the explicit setup path. It adds or
normalizes **unit testing** only; browser automation and end-to-end testing are
separate setup work.

## Input

No argument is required. If the user names a runner or stack preference, treat it
as a preference and verify it against the project files.

## Step 1 - inspect the project

Read enough files to identify the real setup:

- `AGENTS.md` Commands section
- package or language manifest (`package.json`, `pyproject.toml`, `go.mod`,
  `Cargo.toml`, and similar)
- existing test config (`vitest.config.*`, `jest.config.*`, `pytest.ini`,
  `phpunit.xml`, language-native config, and similar)
- existing test files
- package manager lockfile
- an existing `Verify` command and `.github/workflows/verify.yml`, when present
- `docs/project-brief.md`
- `docs/setup.md` -> Stack compatibility notes -> Testing setup, for the
  framework-specific wiring a test config needs beyond a standard install

Do not assume Next.js. Detect the stack from files.

## Step 2 - choose the smallest test setup

Prefer the existing runner if one is already present. If none exists, check
whether `docs/project-brief.md` marks one `[active]` under Stack -> Unit testing.
That is the human's own selection and it wins over every default below. Only
when it marks `None`, or the project is not one the brief covers, choose the
stack-native unit test runner:

- TypeScript or JavaScript app: Vitest by default, unless the project already
  clearly uses Jest or another runner.
- Python: pytest.
- Go: built-in `go test`.
- Rust: built-in `cargo test`.
- Ruby: the runner already implied by the project, or Minitest when nothing else
  is present.
- PHP: PHPUnit when the project is Composer-based.

If the stack is unclear, stop and ask what runner to use instead of guessing.

Keep the setup minimal. Do not add coverage, browser testing, continuous
integration (CI), snapshots, mock-service layers, or a large test architecture
unless the user explicitly asks.

## Step 3 - make the setup changes

Apply the smallest practical diff:

1. Add missing test dependencies or config.
2. Add or normalize package/script commands.
3. Add one small example test for real project logic if a suitable function
   exists; otherwise add a tiny helper and test that proves the runner works.
4. Update the Commands section of `AGENTS.md` with the real test command and,
   when available, the test watch command.
5. If a `Verify` command already exists, add the real test command to it between
   typecheck and build while preserving any established project checks. Do not
   create verification or CI only because `/tests` was invoked.
6. Set the runner you installed or adopted `[active]` under Stack -> Unit testing
   in `docs/project-brief.md`, clearing the mark that was there. **Step 2 reads
   that mark as the human's own selection and lets it override every default**,
   so leaving it on `None` after installing Vitest means the next run of this
   skill reads a selection the project's own config contradicts. If the runner is
   not one of the table's rows, say so and leave the marks alone rather than
   adding a row - `/tests` detects stacks the table does not cover, by design.
7. Update the testing section of `docs/project-brief.md` only if the project
   needs a stack-specific testing note it does not already carry.

Do not write a broad test suite for existing app code. This skill proves the
testing path and turns on the gate; feature work adds focused tests later.

If adding dependencies requires network access, ask for the needed install command
through the current tool's approval flow. Use the project's package manager.

## Step 4 - verify

If `AGENTS.md` documents a `Verify` command, run the focused test command and
then run `Verify` as the final gate. The combined command should now exercise the
new tests along with its existing typecheck and build checks.

If no `Verify` command exists, run the relevant commands from `AGENTS.md`:

- the test command
- the build command, when one exists
- lint or typecheck only if they are already standard commands and the diff
  touches config or types that should satisfy them

An empty suite must not be treated as a pass. If the runner reports no tests, add
or fix the example test.

## Step 5 - report

Stop with a concise report:

- runner chosen or reused
- the Stack -> Unit testing mark now set in `docs/project-brief.md`, or why it
  was left alone
- commands added or updated
- existing verification command updated, or confirmation that none exists
- example test added
- verification commands run and whether they passed
- any follow-up the user should consider

Show the diff summary. Do not commit, merge, push, or start product feature work.

## Rules

- Unit testing only. Do not set up Playwright, Cypress, browser end-to-end
  (E2E) testing, CI, or coverage unless the user explicitly asks.
- Reuse existing project conventions before adding new tools.
- Preserve existing CI. This skill may update an existing verification command,
  but it never creates a GitHub workflow on its own.
- Keep the first test boring and small. It exists to prove the workflow.
- Once `AGENTS.md` has a test command, later `/feature` and `/implement` runs
  treat tests as the gate for logic-bearing changes.
- Do not hide install or verification failures. Report exactly what failed and
  what to fix next.

## Formatting

Format the output to match the project's conventions in `AGENTS.md`: concise,
scannable markdown, with lists for enumerations and tables for matrices rather
than dense paragraphs.
