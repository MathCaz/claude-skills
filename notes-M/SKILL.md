---
name: notes-M
description: Persistent, distilled notes for a multi-turn topic (investigation, design discussion, data analysis, grill-me session) in a dated topic folder (`<parent>/<YYYY-MM-DD>-<slug>/<slug>-notes.md` + side artifacts) that survives `/compact` and multi-day work. Use when the user says "take notes", "start a topic", "keep notes on this", or a topic clearly runs beyond ~5 substantive turns. Not for one-shot Q&A or scoped code edits.
disable-model-invocation: false
---

# Notes (notes-M)

Each topic = one folder: `<slug>-notes.md` (folder name minus the `YYYY-MM-DD-` prefix) plus side artifacts. The folder is the source of truth across sessions.

## When to use

- A topic will span more than ~5 substantive turns
- User says "take notes", "keep notes on this", "start a topic", or invokes `/notes-M`
- Another skill (e.g. `grill-me-M`) delegates file mechanics here

Not for: one-shot Q&A, well-scoped code edits, tasks with their own deliverable.

## Step 1 — Resolve the location

```
<parent>/                        ← default: notes/
└── <YYYY-MM-DD>-<slug>/
    ├── <slug>-notes.md
    └── <side artifacts>
```

1. **Slug** — short kebab-case from the topic (e.g. `mart-freshness-drift`).
2. **Existing match** — `ls` likely parents (`notes/`, `brainstorms/`, `topics/`), grep folder names for topic tokens, `head -5` candidates' notes files. If one looks relevant, ask: "Continue `<folder>` (Status, Last updated), or start a new topic?" If continuing, read the notes file fully first.
3. **No match** — propose `<cwd>/notes/<YYYY-MM-DD>-<slug>/<slug>-notes.md` and wait. The user may name another parent instead. Create the folder only after the answer.

Print the absolute path of the notes file when creating or reopening.

## Step 2 — Skeleton

```markdown
# <Topic>

**Goal:** <one line>
**Mode:** investigation | interview | analysis | generic
**Status:** in progress
**Started:** <YYYY-MM-DD>
**Last updated:** <YYYY-MM-DD>

---

## Summary
*(running synthesis — readable standalone)*

## <Body — shaped by mode, see below>

## Artifacts
*(side files in this folder, one-line purpose each)*

## Open flags
*(pending input + owner)*
```

## Step 3 — Body shape by mode (hints, not rules)

- **Investigation / debug** — `## Findings` (claim · confidence confirmed|probable|speculative · evidence `file:line`/query/commit) · `## Reasoning chains` · `## Dead-ends explored` · `## Open threads`
- **Interview / grill** (used by `grill-me-M`) — `## Decisions log`: Decision · Why · Dead-ends · Flags
- **Data analysis** — `## Hypotheses` · `## Results` (finding + pointer, not full output) · `## Surprises`
- **Generic** — pick from `## Facts`, `## Decisions`, `## Pointers`, `## TODO`

## Step 4 — Side artifacts

Non-text outputs (HTML report, CSV, diagram, scratch SQL) go in the topic folder with a descriptive name. Add `- file.ext — purpose` under `## Artifacts`.

## Step 5 — Checkpoint after every meaningful turn

New finding, decision, ruled-out hypothesis or open thread → update the file and `**Last updated:**` right away, applying the curation rules. Never batch.

## Step 6 — Curation rules

The file is a curated deliverable, rich enough to derive a PRD, plan or summary from — never a transcript.

- **Capture**: decisions, load-bearing why, constraints, rejected options + why, flags with owners, illuminating dead-ends.
- **Drop**: back-and-forth, restated context, duplication, exact wording unless it IS the decision.
- **Collapse**: a thread that resolves → one consolidated entry.
- **Supersede**: a later finding replaces the earlier entry; note "changed from X because Y" only if decision-relevant.
- **Pivot**: stale entries become one dead-end entry if there's a lesson, else are deleted. Decisions tied to the old direction get re-asked. Surface pivots: "These entries assumed X; this pivots to Y — drop, keep as dead-end, or re-ask?"

`## Summary` must stand alone.

## Step 7 — Status

- `in progress` — active; sessions resume against the same folder
- `parked` — add `## Parked because`
- `concluded` — add `## Outcome` (answer + links to PR, ticket, ADR)

End of session → stays `in progress`. End of topic → reconcile contradictions, set status, short recap.

## Rules

- Append cheaply, distill aggressively. Point to `file:line`, don't paste files.
- Redact secrets (keys, tokens, passwords, PII) in notes and artifacts.
- Before `/compact`, `/pre-compact` checkpoints into this file — don't duplicate it.
