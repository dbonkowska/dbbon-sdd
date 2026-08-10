---
description: "[Step 1] Create a structured GitHub Issue and Obsidian working folder for a new feature."
disable-model-invocation: true
---

## Setup

Follow `${CLAUDE_PLUGIN_ROOT}/references/setup.md`. It reads the project and machine config,
scaffolds either if missing, and resolves `{vault_base}` and `{vault_folder}`.

Also read `title_prefixes` and `gh_project` from `.claude/dbbon-sdd.md`. `gh_project` is optional — when blank or absent, issues are created without a project board.

This command needs a GitHub remote — it is the only one that does. Confirm one exists:
```
gh repo view --json name -q .name
```
If this fails, stop: "No GitHub remote. `new-issue` needs one — the issue number names the
feature folder every later step looks for. Publish the repo first."

---

## Step 1: Gather feature description

If the user provided a description when invoking this command, use it as the starting point.
Otherwise ask: "What do you want to build or fix?"

From their input, draft a structured issue. Choose the type that fits:
- **feature** — new capability
- **bug** — something broken
- **experiment** — exploring a technology, pattern, or approach
- **refactor** — improving structure without changing behaviour

Draft:

```
[type: feature | bug | experiment | refactor]

## Why
[One paragraph — the motivation and problem being solved. For experiments: what you want to learn.]

## Acceptance Criteria
- [ ] [specific, observable, testable — one item per behaviour]
- [ ] 

## Out of Scope
- [explicit exclusions — things that could be assumed but won't be done here]

## Tech Notes
[Optional — only if the approach is already clear before design]
```

Also suggest:
- A **title** — prefixed with one of the project's `title_prefixes`, in brackets, per the type of work, followed by a concise specific description (e.g. with prefixes `backend, frontend, setup`: "[backend] Order Domain: model, repository, REST endpoints"). If `title_prefixes` is blank or absent, suggest a plain descriptive title with no prefix.
- A **folder name** — short version for the Obsidian folder, without the prefix (e.g. "Order Domain")

Present both to the user and ask: "Does this look right?" Wait for confirmation or adjustments before continuing.

---

## Step 2: Create the GitHub Issue

**If `issue_body: spec`** (the default) — create the issue with the structured body drafted in Step 1:

```
gh issue create --title "{title}" --body "{formatted body}"
```

**If `issue_body: brief`** — the structured body is *not* published. Read `${CLAUDE_PLUGIN_ROOT}/references/brief.md` and draft a short capability brief instead, covering Capability, Technique and Model. Show it to the user and get explicit confirmation, then:

```
gh issue create --title "{title}" --body "{confirmed brief}"
```

**If `gh_project` is set** in `.claude/dbbon-sdd.md`, append `--project "{gh_project}"` to whichever `gh issue create` runs above, so the issue lands on the board rather than needing to be added by hand. The flag takes the project **title**, not an id or number.

If that call fails with a missing-scope error, the token lacks project access. Stop and tell the user to run `gh auth refresh -s project` themselves — it opens a browser and cannot be run for them. Do not retry without the scope, and do not silently drop the flag: an issue created off the board is easy to miss.

Either way, capture the issue number from the output (e.g. `#5`).

---

## Step 3: Create the Obsidian folder

```powershell
New-Item -ItemType Directory -Force "{vault_base}\{vault_folder}\{N} - {Folder Name}"
```

Where `{N}` is the issue number and `{Folder Name}` is the confirmed short name.

**Order matters:** the folder path depends on `{N}`, so the issue must be created first — create issue → capture `{N}` → create folder → write `issue.md` (next step, `brief` only).

---

## Step 3b: Write `issue.md` — `brief` mode only

Under `brief`, GitHub no longer holds the structured body, so the vault must. Write the full Step 1 draft — Why, Acceptance Criteria, Out of Scope, Tech Notes — to:

```powershell
Set-Content -Path "{vault_base}\{vault_folder}\{N} - {Folder Name}\issue.md" -Value "{structured body}" -Encoding UTF8
```

`design` reads this instead of `gh issue view`. Skip this step entirely under `spec`.

---

## Step 4: Report

Tell the user:
- Issue created: `{URL}`
- Obsidian folder ready: `{vault_base}\{vault_folder}\{N} - {Folder Name}`

Then: "Run `/dbbon-sdd:design` to expand this into a technical spec."
