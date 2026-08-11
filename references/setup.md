# Shared setup

Every `dbbon-sdd` command begins here. Follow these steps, then return to the command's
own Setup section for anything specific to it.

---

## Step 1: Read the project config

Read `.claude/dbbon-sdd.md` from the current project.

**If it does not exist**, write the template below, substituting the repository's
directory name for `{repo dir}`, then **stop**:

> Created `.claude/dbbon-sdd.md`. Check the values — `vault_folder` was guessed from the
> directory name, and the test commands are blank. Re-run when it's right.

````markdown
# dbbon-sdd — {repo dir}

```
vault_folder:   {repo dir}
title_prefixes:
issue_body:     spec
gh_project:
test_dir:
test_single:
test_all:
```

**`title_prefixes`** — comma-separated, offered by `new-issue` as title prefixes.
Leave blank to suggest a plain descriptive title.

**`issue_body`** — `spec` publishes the full spec as the GitHub Issue body. `brief`
publishes only a short capability brief and keeps the spec in the vault, for repos
whose source material can't be republished.

**`gh_project`** — GitHub Project board **title** (not id or number) that `new-issue`
adds issues to. Leave blank to create issues without a board. Requires the `project`
token scope: `gh auth refresh -s project`.

**`test_dir`** — directory the test commands run in, relative to the repo root.
Leave blank if that's the root.

**`test_single`** — `{test}` is substituted with the test name.
````

Do not proceed on a file you just created. The user has to see the values first.

---

## Step 2: Read the machine config

Read `~/.claude/dbbon-sdd-local.md`.

**If it does not exist**, write this template and **stop**:

> Created `~/.claude/dbbon-sdd-local.md`. Set `vault_base` to your vault's root and
> re-run.

````markdown
# dbbon-sdd — machine-local paths

Machine-specific, and points at a private vault. Never committed to any repo.

```
vault_base:
principles:
```

`principles` — the global rules file read by `design`. Leave blank to use
`{vault_base}\Principles.md`.
````

**If `vault_base` is blank**, stop with the same message. It's the one value that
cannot be guessed.

**If `principles` is blank**, use `{vault_base}\Principles.md`.

---

## Step 3: Resolve paths

The project's vault path is `{vault_base}\{vault_folder}`.

Feature folders inside it are named `{N} - {Folder Name}`, where `{N}` is the GitHub
issue number. Commands that work on an existing feature locate its folder with:

```powershell
Get-ChildItem "{vault_base}\{vault_folder}" -Directory | Where-Object { $_.Name -match "^{N} - " }
```

---

## Notes

**No project detection.** Earlier versions ran `gh repo view --json name -q .name` and
matched the result against a block in a shared config file. That is gone — the config is
in the project, so the project is already known. A consequence worth relying on: `plan`,
`implement`, `review` and `recall` never invoke `gh`, and work on a repo with no remote.

**Config is read, never written** — apart from the two scaffold cases above, which always
stop rather than continue.