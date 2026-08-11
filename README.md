# dbbon-sdd

A spec-driven development workflow for Claude Code, as a plugin.

Seven commands run a feature from idea to closed issue: GitHub Issue → spec → TDD plan →
implementation → review → recall → documented patterns. Specs, plans, reviews and
accumulated knowledge live in an Obsidian vault; the repo stays code-only.

## Commands

| Command | Produces |
|---|---|
| `/dbbon-sdd:new-issue` | Structured GitHub Issue + Obsidian working folder |
| `/dbbon-sdd:design` | `spec.md` — precise enough to implement from alone |
| `/dbbon-sdd:plan` | `plan.md` — TDD steps, derived from the spec |
| `/dbbon-sdd:implement` | Code, one red/green/commit cycle per step |
| `/dbbon-sdd:review` | `cr.md` — review against the spec, plus pattern candidates |
| `/dbbon-sdd:recall` | `recall.md` — the feature rebuilt from memory, corrected |
| `/dbbon-sdd:document` | Approved patterns, constitution line, issue comment + close |

Run them in that order. Each one tells you what to run next.

They are user-invoked only (`disable-model-invocation: true`) — Claude will not decide to
start `implement` on your behalf.

## Install

Loaded as a skills-directory plugin: anything under `~/.claude/skills/` with a
`.claude-plugin/plugin.json` auto-loads next session, with no marketplace and no install
step. To keep the working copy in `dev/` rather than under `~/.claude/`, link it —
a Windows **junction**, because symlinks need Developer Mode:

```powershell
New-Item -ItemType Junction `
  -Path   "$HOME\.claude\skills\dbbon-sdd" `
  -Target "C:\path\to\dbbon-sdd"
```

Restart Claude Code, or `/reload-plugins` to pick up edits without restarting. `/plugin`
has an **Errors** tab if the commands don't appear.

This keeps the working directory live — edits take effect immediately. Installing from a
marketplace instead (`claude plugin marketplace add`, then `claude plugin install`) copies
the plugin into `~/.claude/plugins/cache/`, which gives proper versioning at the cost of
needing a push-and-reinstall for every change.

## Configuration

Two files. Both are scaffolded for you on first run if missing, so there is nothing to
copy by hand — the command writes a commented template and stops so you can fill it in.

**`{project}/.claude/dbbon-sdd.md`** — committed, travels with the code.

```
vault_folder:     folder for this project inside the vault
title_prefixes:   comma-separated, offered by new-issue. Empty = plain titles
issue_body:       spec | brief   (see below)
gh_project:       GitHub Project board title. Empty = no board
test_dir:         directory the test commands run in. Empty = repo root
test_single:      command to run one test, with {test} substituted
test_all:         command to run the whole suite
```

`gh_project` takes the board's **title**, not an id, and needs the `project` token scope
(`gh auth refresh -s project`). Projects with `_frontend` variants of the test keys use
those for frontend work.

**`~/.claude/dbbon-sdd-local.md`** — machine paths, never committed.

```
vault_base:       root of the Obsidian vault
principles:       optional; defaults to {vault_base}\Principles.md
```

Only `vault_base` genuinely has to be typed. `vault_folder` is inferred from the repo
directory name and everything else has a working default.

### `issue_body`

`spec` publishes the full spec as the GitHub Issue body. `brief` publishes only a short
capability brief and keeps the spec in the vault — for repos whose source material can't
be republished, such as course exercises. Absent means `spec`.

## Requirements

- `gh`, authenticated. Only `new-issue` requires a GitHub remote to exist; `plan`,
  `implement`, `review` and `recall` never call `gh` at all.
- An Obsidian vault, or any directory tree — nothing depends on Obsidian itself.

## Why it's shaped this way

Personal tooling, developed by using it — every convention below exists because running
the previous version made something obvious.

**Specs live outside the repo.** A `specs/` folder in the repository is the thing this
deliberately avoids: it fills with generated documents that go stale the moment code
moves past them, and it turns `git log` into a record of paperwork. So the repo holds
code, and the vault holds the working material — one feature per folder, `spec.md`,
`plan.md`, `cr.md`. Two places, two jobs: the vault is where thinking happens, the GitHub
Issue is the clean public record.

**The spec drives the plan, not the issue.** `plan` reads `spec.md` and never the issue.
An issue captures intent before you understand the problem; the spec captures it after.
Planning from the issue means planning from the less-informed document, and `review` then
checks the implementation against the spec for the same reason.

**`recall` exists because the other six commands only train recognition.** In every other
step you read something Claude produced and approve it, which builds the ability to
recognise a good design and not the ability to produce one. So one step inverts: you answer
four prompts from memory with no priming, Claude marks them and corrects only what's wrong,
and `recall.md` is written in your words rather than its own. The page that matters is the
short list at the bottom of where you were off — that's your error pattern, and it's the
only part worth re-reading before the next feature.

**Config splits three ways.** Generic workflow in the plugin, project conventions in
`{project}/.claude/dbbon-sdd.md`, machine paths in `~/.claude/dbbon-sdd-local.md`. One
combined file forces the whole thing private as soon as any single value is — a vault
location, say — which drags ordinary facts like a project's test command out of version
control with it. Splitting by scope lets project config travel with the code, so a fresh
clone already knows how to run its own tests.

**`brief` mode exists because not everything can be republished.** Some repositories
build on material that isn't yours to publish — course exercises, licensed content, work
under NDA. The workflow still wants a public record, so `brief` publishes what the project
*gained* — capability, technique, model — while the specifics stay in the vault. The
config key only decides which text gets drafted; the actual safety mechanism is that
nothing is pushed without you confirming it first.

**Commands are user-invoked only.** All seven carry `disable-model-invocation: true`. These
create issues, write files and run test suites — nothing here should start because a
model inferred it was a good idea.

This repo holds only what is needed to run — no generated docs, no design folder. The
same rule it applies to the projects it manages.
