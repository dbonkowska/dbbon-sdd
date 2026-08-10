---
description: "[Step 4] Execute the TDD plan step by step — red, green, commit."
disable-model-invocation: true
---

## Setup

Follow `${CLAUDE_PLUGIN_ROOT}/references/setup.md`. It reads the project and machine config,
scaffolds either if missing, and resolves `{vault_base}` and `{vault_folder}`.

Also read `test_dir`, `test_single` and `test_all` from `.claude/dbbon-sdd.md`. Run every
test command from `{test_dir}`, or the repo root if it is blank. Where a project has
`_frontend` variants, use those for frontend work.

(`principles`/`Constitution.md` are not read here — `plan` already applied them when
drafting the approach for each step.)

This command never invokes `gh`.

---

## Step 1: Identify the issue and read the plan

If the user provided an issue number when invoking this command, use it.
Otherwise ask: "Which issue are we implementing? (provide the number)"

Find the Obsidian folder:
```powershell
Get-ChildItem "{vault_base}\{vault_folder}" -Directory | Where-Object { $_.Name -match "^{N} - " }
```

If no folder found: "Run `/dbbon-sdd:new-issue` first."

Read the plan:
```powershell
Get-Content "{vault_base}\{vault_folder}\{N} - {Folder Name}\plan.md" -Raw
```

If no plan found: "Run `/dbbon-sdd:plan` first."

If the user did not specify a starting step, scan the step headings (`### {N}. [ ] ...` / `### {N}. [x] ...`) for the first unchecked one and propose it: "Steps 1–{M} are marked done — resume from step {N}?" Wait for confirmation. If every step is checked, tell the user the plan is already complete and stop.

---

## Step 2: Read conventions

The plan describes test scenarios and implementation approach, not code — actual code is written here, so it needs to follow real conventions.

Read the `## Conventions` section from `plan.md` (already loaded in Step 1) — it captures what `/dbbon-sdd:plan` already learned about existing patterns (test setup, layering, naming), so this step doesn't need to re-explore from scratch. If a convention references a file and the current step needs more detail than the summary gives (exact field names, exact assertion style), read that file now.

If `plan.md` has no Conventions section (project had no existing code when it was planned, or the plan predates this section), explore directly instead: read representative existing files for the layer(s) this feature touches — at minimum one existing test and one existing implementation file from the same layer.

If the project has no existing code at all, skip this step.

---

## Step 3: Execute each step

For each step in the plan, starting from the chosen step:

### 3a. Write the test

Using the test scenario from the plan (input → expected output), write the actual test code, following the conventions found in Step 2. If the file already exists, add the new test to it — do not overwrite existing tests.

### 3b. Run — expect RED

Run the test:
```
{test_single with the test name substituted in}
```

If the test passes immediately (no RED): stop and tell the user — a passing test before implementation means either the behaviour already exists or the test is not testing what it should. Do not proceed until resolved.

If the test fails for the wrong reason (compilation error, wrong class name, missing import): fix the structural issue and re-run. This is not a RED/GREEN problem — it's a setup problem. Fix it before treating it as RED.

Once the test fails for the right reason (assertion failure — the behaviour doesn't exist yet): confirm RED and continue.

### 3c. Write the implementation

Using the implementation approach from the plan and the conventions from Step 2, write the minimum code to make the test pass. Do not add behaviour not covered by the current test.

### 3d. Run — expect GREEN

```
{test_single with the test name substituted in}
```

If still failing: debug and fix. Do not move on until this test is GREEN.

### 3e. Run full suite — confirm no regressions

Before suggesting a commit, run the full suite, not just this step's test:
```
{test_all}
```

If anything fails: this step introduced a regression in a previously passing test, even though its own test is GREEN. Debug and fix before proceeding — do not suggest a commit while `{test_all}` is red. This is the point where a regression is cheapest to find, since it's isolated to the change just made.

### 3f. Suggest commit

Once `{test_all}` is green, tell the user which files changed and the commit message from the plan:

```
Files to stage: {list of files changed in this step}
Suggested commit: {commit message from the plan}
```

Wait for the user to commit before continuing.

### 3g. Mark done and pause

Mark this step complete in `plan.md`: change its heading from `### {N}. [ ]` to `### {N}. [x]` (that step only — leave everything else unchanged).

Tell the user what was just completed and what comes next. Ask: "Continue to step {N+1}?" Wait for confirmation before proceeding.

---

## Step 4: Final verification

`{test_all}` already ran green after the last step (3e), so this is a formality — re-run it once more to confirm nothing changed outside the step loop:
```
{test_all}
```

If anything fails: do not report success. Fix and re-run until the full suite is green.

---

## Step 5: Report

Tell the user:
- All {N} steps complete
- Full test suite green

Then: "Run `/dbbon-sdd:review` to review the implementation."
