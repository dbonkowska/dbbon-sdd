---
description: "[Step 6] Rebuild the feature from memory before closing it out — the user answers, Claude corrects, the page stays in their words."
disable-model-invocation: true
---

## Setup

Follow `${CLAUDE_PLUGIN_ROOT}/references/setup.md`. It reads the project and machine config,
scaffolds either if missing, and resolves `{vault_base}` and `{vault_folder}`.

`{principles}` and `Constitution.md` are deliberately not read here — the user's memory is
the input, and the rules are not what is being measured.

This command never invokes `gh`.

---

## The gate

**The user answers first. Every prompt, every time.**

<EXTREMELY-IMPORTANT>

Do not write, draft, summarise, hint, or "just to orient you" your way into supplying an
answer before the user has given theirs. Do not offer multiple choice. Do not restate the
spec. A blank prompt is the entire mechanism — the moment you fill it in, this command
becomes one more artifact the user did not write, and the problem it exists to solve is
back.

If the user asks you to just write it: say no, and offer the hint ladder in Step 3 instead.
</EXTREMELY-IMPORTANT>

The rest of the pipeline trains recognition — the user reads what you produced and approves
it. Recognition is not recall, and recall is not design. This is the only step where the
user generates, so it is the only step where the learning happens.

---

## Closed book, and no revision first

The prompts are answered cold. Not from a reread, not from a skim of `spec.md` five minutes
earlier, not with the file open in another pane.

Revising first measures the last twenty minutes rather than what the user holds, and it
raises *fluency* — the feeling of recognising the material — without touching the ability to
produce it. That gap is the whole reason this command exists, so an answer produced with the
spec open is worse than useless: it reports a competence that isn't there and writes it into
`recall.md` as fact.

A wrong answer is the useful outcome. It is the only thing in the pipeline that locates where
the user's model is actually broken.

**The artifacts get read afterwards, not before.** Once a prompt is marked, reading the spec
and the code for whatever was wrong is worth more than an hour of reading to absorb — the
user now has a specific question, which turns reading into retrieval instead of more input.
Point them at the exact file and let them look.

**One exception.** If a prompt lands on something the user genuinely never saw — built while
they were elsewhere, or landed in a step they didn't follow — that is not a memory failure.
Take it straight to rung 3 and move on rather than letting them grind.

---

## Step 1: Identify the issue and read the material

If the user provided an issue number when invoking this command, use it.
Otherwise ask: "Which issue are we recalling? (provide the number)"

Find the Obsidian folder:
```powershell
Get-ChildItem "{vault_base}\{vault_folder}" -Directory | Where-Object { $_.Name -match "^{N} - " }
```

If no folder found: "Run `/dbbon-sdd:new-issue` first."

Read, skipping any that are absent:
- `{vault_base}\{vault_folder}\{N} - {Folder Name}\spec.md`
- `{vault_base}\{vault_folder}\{N} - {Folder Name}\plan.md`
- `{vault_base}\{vault_folder}\{N} - {Folder Name}\cr.md`

Then read the implementation itself — the diff for the issue's branch, or the files the plan
touched. The spec describes intent; only the code shows what was actually built.

`cr.md` is optional. This command can run before `review`, and can run mid-implementation.
**If the plan has unchecked steps**, say so and scope every prompt to what is built:

> Steps {N}–{M} aren't done yet, so I'll keep this to what exists. We can extend it later.

Say nothing else about what you read. No summary, no "here's what I found." The user's
memory is the input, and priming it destroys the measurement.

---

## Step 2: Four prompts, one at a time

Ask one. Wait. Mark it. Then ask the next.

One at a time matters: a wrong answer to prompt 1 gets corrected before it poisons prompt 3,
and four blank prompts at once is a wall.

Before the first prompt, set expectations once:

> Four questions, one at a time, from memory. Closed book — don't open the spec, and don't
> revise first; answering cold is the point, and a wrong answer is the useful outcome. Rough
> bullets are fine. We'll go read the artifacts together afterwards, for whatever you missed.

If the user has just been rereading the spec, or asks to skim it first, say no and explain
why in a sentence. Offer the hint ladder instead — that is what it is for.

**Prompt 1 — Compression.** Ask verbatim:
> In five lines or fewer: what did this issue add, and what couldn't be done before it?

**Prompt 2 — The name.** Ask verbatim:
> What is the main pattern here called, outside this repo? Where else would you meet it?

**Prompt 3 — The mechanism.** Ask verbatim:
> Write the core mechanism as pseudocode. Names don't matter; the order of operations does.

**Prompt 4 — The forks.** Ask verbatim:
> Which decisions here were judgment calls rather than forced? Pick one and say what breaks
> if you reverse it.

Each prompt does a job the others can't. 1 builds the zoom level the spec has no room for.
2 deflates the work — most of what feels bespoke has a name and a literature, and knowing
that is the difference between "I could never design this" and "I haven't learned this one
yet." 3 is the only prompt that cannot be answered by recognising something, which makes it
the one that builds design ability. 4 separates the handful of real decisions from the
mechanical consequence that surrounds them.

---

## Step 3: Mark each answer

Right / incomplete / wrong. Then:

- **Right** — say so and move on. **Add nothing.** Enriching a correct answer with the detail
  you would have written is how this step quietly reverts to being yours.
- **Incomplete** — name only what is missing, in a sentence.
- **Wrong** — give the correction and one line of why. Not a lecture.

Judge understanding, not vocabulary. A right idea in the wrong words is right; correct the
term in passing and move on.

**If the user is stuck**, climb the ladder one rung at a time, and only on request:

1. Name the file or type the answer lives in — nothing about what it does.
2. Give the shape: how many parts, what kind of thing each is.
3. Give the answer.

Rung 3 is a normal outcome, not a failure — but note which prompts needed it. A prompt that
needs rung 3 twice across two issues is the signal for what to study, and that is worth more
than a clean-looking page.

---

## Step 4: Write `recall.md`

**The user's words, patched.** Keep their phrasing, their bullets, their pseudocode style.
Fix what was wrong; do not rewrite what was right into better prose. A page that sounds like
the spec is a failed page — it means you wrote it.

```powershell
Set-Content -Path "{vault_base}\{vault_folder}\{N} - {Folder Name}\recall.md" -Value "{content}" -Encoding UTF8
```

```markdown
# Recall — {Feature Name}

> Issue #{N} | {date}

## In five lines
[Prompt 1, corrected.]

## Pattern
[Prompt 2 — the name, and where it exists outside this repo.]

## Mechanism
[Prompt 3 — pseudocode, in the user's own notation.]

## Judgment calls
[Prompt 4 — which decisions were choices, and the cost of reversing one.]

## Where I was off
- [what was said] → [what is true]

## Needed a hint
- [prompt] — [rung reached]
```

Omit `Where I was off` and `Needed a hint` if empty.

Those two sections are the point of the file. Everything above them the user already knows
by the time it's written; these are the only parts worth re-reading before the next issue,
because they are the user's actual error pattern rather than generic notes.

---

## Step 5: Read back what was missed

Now the artifacts earn their keep. For each entry in `Where I was off` and each prompt that
needed a hint, name the exact place the answer lives — file and type, or the spec section:

> You had the retry firing on any failure. That's `OrderService.submit`, the branch that
> checks the status code first — worth reading those six lines now.

Point, don't re-explain. The user has just failed to produce it, so they arrive at the file
with a live question, and reading is retrieval rather than more input. Re-narrating it here
converts that back into recognition and wastes the one moment the reading would have stuck.

Give them the list and stop. If they come back with questions, answer those.

Nothing to read back is a clean run — say so and move on.

---

## Step 6: Report

Tell the user:
- `recall.md` written to `{vault_base}\{vault_folder}\{N} - {Folder Name}\recall.md`
- Which prompts were clean, which needed correction, which needed a hint

If anything from prompt 2 or 4 looks like a durable project rule rather than a one-off,
mention it — `document` will ask about patterns and the constitution next, and this is where
those candidates surface honestly.

Then: "Run `/dbbon-sdd:document` to close out the feature."

---

## Notes

**`Patterns.md` is not this file.** That one holds project gotchas and load-bearing config —
things you would otherwise trip on twice. `recall.md` is per-issue and personal: what you
held, where you were off. Different jobs, and merging them loses both.

**Runs before `review` if needed.** The ordering is a default, not a dependency. Recalling
while the work is fresh beats recalling in the right slot.
