---
description: "[Step 2] Expand a GitHub Issue into a detailed technical spec in Obsidian."
disable-model-invocation: true
---

## Setup

Follow `${CLAUDE_PLUGIN_ROOT}/references/setup.md`. It reads the project and machine config,
scaffolds either if missing, and resolves `{vault_base}`, `{vault_folder}` and
`{principles}`.

Also read `issue_body` from `.claude/dbbon-sdd.md`. It is `spec` unless stated otherwise,
and it changes Step 1 and Step 7.

No remote check here — an issue number implies a remote, and `new-issue` already enforced
it.

---

## Step 1: Identify the issue

If the user provided an issue number when invoking this command, use it.
Otherwise ask: "Which issue are we designing? (provide the number)"

Find the matching Obsidian folder first — under `brief` the source lives inside it:
```powershell
Get-ChildItem "{vault_base}\{vault_folder}" -Directory | Where-Object { $_.Name -match "^{N} - " }
```

If no folder found: "Run `/dbbon-sdd:new-issue` first to create the issue and folder."

Now fetch what the spec expands from.

**If `issue_body: spec`** (the default) — the GitHub Issue is the source:
```
gh issue view {N} --json title,body
```

**If `issue_body: brief`** — GitHub holds only a capability brief, so the source is in the vault:
```powershell
Get-Content "{vault_base}\{vault_folder}\{N} - {Folder Name}\issue.md" -Raw
```

If `issue.md` is absent (an issue created before this mode existed), fall back to `gh issue view {N} --json title,body`.

---

## Step 2: Read the constitution

Read these files before designing — they are non-negotiable constraints:

1. `{principles}` — global principles, applying across every project
2. `{vault_base}\{vault_folder}\Constitution.md` — project-specific rules (if it exists; skip silently if not)

Note any rules that are directly relevant to this feature. You will reference them while drafting the spec.

---

## Step 3: Draft the spec

Using the source from Step 1 as the starting point, draft a `spec.md`. The spec must be precise enough that an agent can implement from it alone.

Under `issue_body: spec` it must also be clean enough to stand as the GH Issue body. Under `brief` that constraint does not apply — the spec is vault-only and can carry whatever detail the work needs, including material that must never be published.

Structure:

```markdown
# {Title}

> Issue #{N}

## Why
[What problem this solves, or what is being learned. One or two paragraphs.]

## Scope
**In scope:**
- [explicit list of what will be built]

**Out of scope:**
- [explicit exclusions — things that could be assumed but won't be done here]

## Design

[This section varies by what the feature needs. Use only the subsections that apply:]

### Data model
[Entity fields with types. For Java: include annotations. For TypeScript: interfaces.]

### API
[Table: Method | Path | Request | Response | Notes]

### Behaviour
[State transitions, rules, edge cases. Numbered or bulleted.]

### UI / UX
[Only if relevant. Wireframe description or interaction notes.]

## Testing approach
[What kinds of tests, what they cover. No code here — plan.md will have the test code.]

## Tech notes
[Non-obvious implementation details, library choices, version constraints. Omit if empty.]
```

Use only the subsections that apply. A refactor might only have Why, Scope, and Behaviour. An experiment might have Why, Scope, and a Learning Goals subsection instead.

---

## Step 4: Check against the constitution

Review the draft spec against the rules read in Step 2. For each conflict or extension, add an inline callout directly in the spec under the relevant section:

```
> ⚠ **Constitution note:** [rule being extended or contradicted] — [brief reasoning for the deviation or extension]
```

If the spec is fully consistent with the constitution, say so briefly: "No constitution conflicts."

---

## Step 5: Present open questions

Before writing any files, list questions that could affect the design — things that are genuinely unclear, not just implementation details. Examples:
- Should this share infrastructure with X, or be independent?
- Is Y a strict constraint or a preference?
- Does this interact with Z in a way that needs to be spec'd now?

Format:
```
Open questions before we proceed:
1. [question] — [why it matters for the design]
2. ...
```

If there are no open questions, say so and skip this step.

Wait for the user to answer or confirm before continuing.

---

## Step 6: Write to Obsidian

Once the user confirms the spec (and any open questions are resolved):

Write the final spec:
```powershell
Set-Content -Path "{vault_base}\{vault_folder}\{N} - {Folder Name}\spec.md" -Value "{spec content}" -Encoding UTF8
```

---

## Step 7: Update the GitHub Issue

**If `issue_body: spec`** (the default) — replace the GH Issue body with the refined spec, omitting any constitution callouts, which stay in Obsidian only:

```
gh issue edit {N} --body "{spec content without constitution callouts}"
```

**If `issue_body: brief`** — the spec is never published. Read `${CLAUDE_PLUGIN_ROOT}/references/brief.md` and draft a capability brief instead, firming up its Scope section now that the design is settled. **Show it to the user and get explicit confirmation before pushing** — that confirmation is the safety mechanism, not the config key. Then:

```
gh issue edit {N} --body "{confirmed brief}"
```

**Do not pass `--title`.** Never rename the issue — the spec's `# {Title}` heading is for the Obsidian file only. If the drafted title is meaningfully better than the original, mention it to the user and let them rename it themselves via `gh issue edit {N} --title "..."`.

---

## Step 8: Report

Tell the user:
- `spec.md` written to `{vault_base}\{vault_folder}\{N} - {Folder Name}\spec.md`
- GH Issue #{N} updated

Then: "Run `/dbbon-sdd:plan` to build the TDD implementation plan."
