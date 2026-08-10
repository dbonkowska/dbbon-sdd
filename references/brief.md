# The capability brief

Read this when `issue_body: brief`. It is the only thing that gets published to GitHub for
such a project — the full spec stays in the vault.

---

## The disclosure test

Apply this to every sentence before it goes to GitHub:

> **Write the brief as if the source material did not exist.** If a sentence can't be
> written without naming the task, its data, or its answer, it doesn't belong in the brief.

This is the actual safety mechanism, along with the user confirming the text before it is
pushed. The `issue_body` key only decides *which* text gets drafted — it cannot judge
content. Never push a brief the user hasn't seen.

Nothing reaches GitHub that wasn't deliberately written to be public. There is no
stripping step and no marker to forget: the brief is composed as a public document from
the start.

---

## The template

Filled progressively — `new-issue` writes Capability, Technique and Model; `design` firms
up Scope; `document` appends its usual `## Done` and `## Learnings`.

```markdown
[type: feature | bug | experiment | refactor]

## Capability
[What the project can now do that it couldn't before. Project vocabulary only.]

## Technique
[The named technique or pattern being exercised.]

## Model
[Model id + provider. Omit if not model-specific.]

## Scope
- [What gets built, in project terms]

Out of scope: [...]

_Ref: {reference}_
```

**The reference token appears in exactly one place** — the `_Ref:_` footer, plus
optionally the issue title. Never woven into prose, so there is one line to check rather
than a document to scan.

---

## Where the line falls

A worked example. This is publishable:

> Four application-side tools exercising the loop, defined in the application module
> rather than the shared library.

The count and the layering are architecture. What stays private: the tool names, their
semantics, the endpoints they call, and the problem domain — those describe the task, not
the capability.

The same split applies to `document`'s closing comment. A dependency's package namespace
moving between major versions is a framework learning and publishes. *"The upstream
endpoint returns X so you can join it against Y"* describes the shape of the task and
does not.

---

## What this costs

Under `brief`, the public issue reads as a changelog entry rather than a spec: capability,
technique, model, what changed, and a bare reference token — but not the problem it
solved. That is the accepted trade, not an oversight.