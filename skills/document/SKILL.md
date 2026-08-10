---
description: "[Step 6] Finalize the feature — write approved patterns, update the constitution, close out the GitHub Issue."
disable-model-invocation: true
---

## Setup

Follow `${CLAUDE_PLUGIN_ROOT}/references/setup.md`. It reads the project and machine config,
scaffolds either if missing, and resolves `{vault_base}`, `{vault_folder}` and
`{principles}`.

Also read `issue_body` from `.claude/dbbon-sdd.md`. It is `spec` unless stated otherwise,
and it changes Step 4's closing comment.

No remote check here — an issue number implies a remote.

---

## Step 1: Identify the issue and read the CR

If the user provided an issue number when invoking this command, use it.
Otherwise ask: "Which issue are we closing out? (provide the number)"

Find the Obsidian folder:
```powershell
Get-ChildItem "{vault_base}\{vault_folder}" -Directory | Where-Object { $_.Name -match "^{N} - " }
```

If no folder found: "Run `/dbbon-sdd:new-issue` first."

Read:
```powershell
Get-Content "{vault_base}\{vault_folder}\{N} - {Folder Name}\cr.md" -Raw
```

If no CR file found: "Run `/dbbon-sdd:review` first."

Also read the current state of:
- `{principles}`
- `{vault_base}\{vault_folder}\Constitution.md` (skip if absent)
- `{vault_base}\{vault_folder}\Patterns.md` (skip if absent)

---

## Step 2: Write approved patterns

Present the pattern candidates from the CR one at a time. For each:

```
Pattern: {name}
{description}

Write to: [Patterns.md / Principles.md / skip]?
```

- **Patterns.md** — project-specific patterns (Testcontainers setup for this stack, this project's validation approach, etc.)
- **Principles.md** — universal patterns that apply across all projects (tech-agnostic TDD rules, general architecture principles)
- **Skip** — noted but not worth preserving

For each approved pattern, append to the appropriate file:

```markdown
## {Pattern name}

> Added after issue #{N} — {Feature Name}

{Description. What the pattern is, why it exists, an example if helpful.}
```

If `Patterns.md` doesn't exist yet, create it with a header:
```markdown
# {Project} — Patterns
```

---

## Step 3: Update the constitution

Ask: "Did this feature establish any project-wide rule worth adding to the constitution?"

Give a suggestion based on what was seen during review — a decision that came up and will likely come up again. The user can accept, reword, or skip.

If accepted, append one line to `{vault_base}\{vault_folder}\Constitution.md`:

```markdown
- {Rule}. _(established in issue #{N})_
```

If `Constitution.md` doesn't exist yet, create it with a header:
```markdown
# {Project} — Constitution
```

---

## Step 4: Add a GitHub Issue comment

Add a closing comment to the issue summarising what was built and what was learned:

```
gh issue comment {N} --body "{comment}"
```

Comment structure:
```markdown
## Done

{One paragraph: what was implemented, any notable deviations from the original spec.}

## Learnings
{Bullet points: patterns extracted, stack discoveries, decisions made. Omit if nothing worth noting.}
```

**If `issue_body: brief`**, read `${CLAUDE_PLUGIN_ROOT}/references/brief.md` and apply the disclosure test to this comment before posting. The structure is unchanged, but both sections stay at the level of the project's own capabilities and techniques — never the source material the work came from. A framework or stack learning publishes; anything that only makes sense by describing the underlying task does not. Show the comment to the user and get confirmation before posting.

Then ask: "Close the issue?"

If yes:
```
gh issue close {N}
```

---

## Step 5: Report

Tell the user:
- Patterns written: list what was added and where
- Constitution: updated / unchanged
- GitHub Issue #{N}: commented + closed / commented only

Feature complete.
