---
description: "[Step 3] Build a TDD implementation plan from the feature spec."
disable-model-invocation: true
---

## Setup

Follow `${CLAUDE_PLUGIN_ROOT}/references/setup.md`. It reads the project and machine config,
scaffolds either if missing, and resolves `{vault_base}`, `{vault_folder}` and
`{principles}`.

Also read `test_dir`, `test_single` and `test_all` from `.claude/dbbon-sdd.md` — the plan
writes real test commands into its steps. Where a project has `_frontend` variants, use
those for frontend work.

This command never invokes `gh`.

---

## Step 1: Identify the issue

If the user provided an issue number when invoking this command, use it.
Otherwise ask: "Which issue are we planning? (provide the number)"

Find the Obsidian folder:
```powershell
Get-ChildItem "{vault_base}\{vault_folder}" -Directory | Where-Object { $_.Name -match "^{N} - " }
```

If no folder found: "Run `/dbbon-sdd:new-issue` first."

Read the spec:
```powershell
Get-Content "{vault_base}\{vault_folder}\{N} - {Folder Name}\spec.md" -Raw
```

If no spec found: "Run `/dbbon-sdd:design` first."

---

## Step 2: Read the constitution

Read:
1. `{principles}` — global principles
2. `{vault_base}\{vault_folder}\Constitution.md` — project rules (skip if absent)

These constrain how the plan is written — test style, layering rules, naming conventions.

---

## Step 3: Explore the codebase

Before writing the plan, understand the existing structure. This exploration is read once here and reused by `/dbbon-sdd:implement` (via the Conventions section below), so do it in one thorough pass rather than a skim.

First, search broadly to identify what's relevant — where tests live, existing patterns for the layer(s) this feature touches (identify what this project's layers actually are rather than assuming a standard set), and any shared test infrastructure (base classes, fixtures, factories). Then read all identified representative files together in a single batch — at minimum one existing test and one existing implementation file from the same layer — rather than discovering and reading them one file at a time across multiple turns.

Summarize what was found as a short **Conventions** list (package/class names as concrete examples, annotation patterns, test setup approach). This goes into `plan.md` in Step 4 so `/dbbon-sdd:implement` doesn't need to re-explore the same ground.

If the project has no existing code yet, skip this step and omit the Conventions section in Step 4.

---

## Step 4: Draft the plan

Write a step-by-step TDD plan. Each step is a red-green-commit cycle. Describe **scenarios and approach, not code** — the actual test and implementation code is written during `/dbbon-sdd:implement`, with full codebase context at that point. Writing runnable code twice (once here, once for real) is slow to generate and wastes the review checkpoint on a wall of code instead of the shape of the plan.

```markdown
# Plan: {Title}

> Issue #{N}

## Conventions
[Short bullet list from Step 3 — one line per convention, each naming a real example file. E.g. "Controllers: `@WebMvcTest` + `@MockitoBean` repository — see `OrderControllerTest`." Omit this section if the project had no existing code to explore.]

## Steps

### {N}. [ ] {What this step delivers}

**Test scenario** (`{test file path}`):
- {Method / endpoint / unit under test}
- Input: {specific input value or state}
- Expected: {specific expected output or behaviour}
- [additional scenario lines if this step covers more than one case]

Run: `{test_single command with this test substituted in}`
Expected: RED

**Implementation approach** (`{implementation file path}`):
{Prose — key classes/methods to add or change, and any non-obvious logic. Not code.}

Run: `{test_single command}`
Expected: GREEN

Commit: `{short conventional commit message}`

---
```

Rules for the plan:
- Every step heading starts unchecked (`[ ]`) — `/dbbon-sdd:implement` checks it off as steps complete, so re-runs can resume without asking you to remember where you left off.
- Every scenario needs a specific input and a specific expected output — no "verify it works" placeholders.
- Every step must be independently committable — the repo must compile and all existing tests must pass after each commit.
- Tests come first. Never implement before the failing test.
- Implementation is the minimum to make the test pass. No speculative code.
- If a step requires multiple files (e.g. entity + migration + repository), group them under one test scenario that covers the observable behaviour, not one step per file.
- Use the actual test command from `.claude/dbbon-sdd.md` (`test_single` and `test_all`), substituting the test name where needed. Note `test_dir` if set — the commands run from there, not the repo root.
- At the end of the plan, add a final step: run `{test_all}` to verify the full suite is green.

---

## Step 5: Present for confirmation

Show the plan and ask: "Does this look right? Any steps to add, remove, or reorder?"

Wait for the user to confirm or adjust before writing.

---

## Step 6: Write to Obsidian

Once confirmed:

```powershell
Set-Content -Path "{vault_base}\{vault_folder}\{N} - {Folder Name}\plan.md" -Value "{plan content}" -Encoding UTF8
```

---

## Step 7: Report

Tell the user:
- `plan.md` written to `{vault_base}\{vault_folder}\{N} - {Folder Name}\plan.md`

Then: "Run `/dbbon-sdd:implement` to execute the plan step by step."
