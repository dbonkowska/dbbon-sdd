---
description: "[Step 5] Review the implementation against the spec and surface pattern candidates."
disable-model-invocation: true
---

## Setup

Follow `${CLAUDE_PLUGIN_ROOT}/references/setup.md`. It reads the project and machine config,
scaffolds either if missing, and resolves `{vault_base}`, `{vault_folder}` and
`{principles}`.

This command never invokes `gh`.

---

## Step 1: Identify the issue and read context

If the user provided an issue number when invoking this command, use it.
Otherwise ask: "Which issue are we reviewing? (provide the number)"

Find the Obsidian folder:
```powershell
Get-ChildItem "{vault_base}\{vault_folder}" -Directory | Where-Object { $_.Name -match "^{N} - " }
```

If no folder found: "Run `/dbbon-sdd:new-issue` first."

Read from Obsidian:
```powershell
Get-Content "{vault_base}\{vault_folder}\{N} - {Folder Name}\spec.md" -Raw
Get-Content "{vault_base}\{vault_folder}\{N} - {Folder Name}\plan.md" -Raw
```

If either is missing: "Run `/dbbon-sdd:design` and `/dbbon-sdd:plan` first."

Read the constitution:
1. `{principles}` — global principles
2. `{vault_base}\{vault_folder}\Constitution.md` — project rules (skip if absent)

---

## Step 2: Read the implementation

From `plan.md`, extract the list of files that were created or modified. Read each one.

Also run:
```
git log --oneline -20
```

to get a sense of the commits made during this feature.

If the user mentions any additional files not in the plan, read those too.

---

## Step 3: Review

Evaluate the implementation across these dimensions:

**Spec compliance**
- Does the implementation match the acceptance criteria in spec.md?
- Are any out-of-scope items accidentally included?
- Are there behaviours in the spec that are missing or incomplete?

**TDD discipline**
- Does each test assert one specific behaviour?
- Are tests testing through the public API, or are they reaching into internals?
- Are there any tests that could not meaningfully fail?

**Code quality**
- Naming: clear, consistent, following project conventions?
- Layering: does each class/function stay within its responsibility?
- Duplication: anything that should be extracted (but only if it's already recurring)?
- Anything that will be confusing to read in 3 months?

**Constitution compliance**
- Does the implementation follow the rules from Principles.md and Constitution.md?
- Flag any deviation explicitly.

---

## Step 4: Surface pattern candidates

Actively scan for things worth keeping beyond this feature:

- Implementation approaches that solved a recurring problem (error mapping, validation, test setup)
- Architectural decisions that should be applied project-wide going forward
- Spec → implementation deviations that were accepted (these are often implicit rules)
- Non-obvious stack discoveries (framework quirks, version-specific behaviour)
- Explicit out-of-scope decisions that established a project-wide boundary

Present them as candidates — do not write them anywhere yet:

```
Pattern candidates from this feature:
1. [name] — [what it is and why it's worth keeping]
2. ...
```

If nothing worth extracting, say so.

---

## Step 5: Write the CR file

Write the review output to:
```
{vault_base}\{vault_folder}\{N} - {Folder Name}\cr.md
```

Structure:
```markdown
# CR — {Feature Name}

> Issue #{N} | {date}

## Summary
[One paragraph overall assessment.]

## Issues
[Numbered list of problems found. Each one: what it is, where it is, why it matters, and a concrete suggestion for fixing it. Omit this section if none.]

## Minor notes
[Small things — naming, style, non-blocking suggestions. Omit if none.]

## Constitution compliance
[Compliant / deviations noted: ...]

## Pattern candidates
[List from Step 4, or "None identified."]
```

---

## Step 6: Present findings

Show the user:
1. The issues found (if any) — these should be addressed before moving on
2. The pattern candidates — ask which to approve for `document` to write up

If there are blocking issues: "Fix these before running `/dbbon-sdd:recall`."
If the review is clean: "Run `/dbbon-sdd:recall` to rebuild the feature from memory before closing it out."
