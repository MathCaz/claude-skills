---
name: notes-M
description: Persistent, distilled note-taking for any multi-turn topic — investigations, debugging, design discussions, data analyses, brainstorms, grill-me sessions. Each topic gets its own folder (`<parent>/<YYYY-MM-DD>-<topic-slug>/`) containing `<topic-slug>-notes.md` plus any side artifacts (reports, exports, diagrams). Survives `/compact`, session end, and multi-day work. Auto-invokes when starting a multi-step topic; can also be invoked explicitly.
disable-model-invocation: false
---

# Notes (notes-M)

A generic persistent-note primitive. Each topic = a self-contained folder (`<topic-slug>-notes.md` + supporting artifacts). The folder is the source of truth; topics live across sessions.

**Naming convention**: main file = `<topic-slug>-notes.md` where `<topic-slug>` is the folder name with the leading `YYYY-MM-DD-` stripped. Example: folder `2026-06-26-claude-setup/` → file `claude-setup-notes.md`. In legacy mode (d, single file no folder), file name = `<YYYY-MM-DD>-<topic-slug>.md`.

## When to use

- User starts a multi-step investigation, debugging session, design discussion, data analysis, or brainstorm — anything that will span more than ~5 substantive turns
- User explicitly invokes (`/notes-M`) or another skill (e.g. `grill-me-M`) delegates file mechanics here
- User mentions: "take notes", "let's analyze", "let's dig into", "let's design", "investigate", "debug", "trace"

Don't use for: simple one-shot Q&A, well-scoped code edits, tasks with their own deliverable.

## Step 1 — Resolve the location

**Default structure** (when parent exists or user accepts the proposal):

```
<parent>/                        ← default: notes/
└── <YYYY-MM-DD>-<topic-slug>/   ← topic folder
    ├── <topic-slug>-notes.md     ← main file (folder name minus `YYYY-MM-DD-` prefix)
    └── <side artifacts>          ← reports, exports, diagrams, etc.
```

### 1.a — Derive the slug

Short kebab-case from the topic (e.g. `auth-login-flow-investigation`, `mart-freshness-drift`, `sql-style-rules`).

### 1.b — Scan for existing match

- `ls <parent>/` (cheap, no file reads — try `notes/`, `brainstorms/`, `topics/`, `projects/`, or whatever the user uses)
- Grep folder names for tokens from the current topic
- For any candidates, `head -5` the inner `<topic-slug>-notes.md` (just the header — Topic / Goal / Status / Last updated)
- **Always ask the user** if a candidate looks relevant:
  > "Continue `notes/2026-06-20-auth-login-flow-investigation/` (Status: in progress, Last updated: 2026-06-22), or start a new topic?"
- If continuing → **read the existing `<topic-slug>-notes.md` fully** before adding anything. Print the folder path back.

### 1.c — No match found → ASK before creating

Default proposal: `<cwd>/notes/<YYYY-MM-DD>-<slug>/<slug>-notes.md`.

If `notes/` doesn't exist at cwd, present the user with 4 options:

```
No `notes/` parent folder at <cwd>. Where should the topic live?
  (a) Create `notes/2026-06-27-<slug>/<slug>-notes.md` (new parent folder)
  (b) Use a different parent name (you pick — e.g. `brainstorms/`, `docs/research/`, `topics/`)
  (c) No parent folder — create `2026-06-27-<slug>/<slug>-notes.md` at cwd root
  (d) No folders at all — single file `2026-06-27-<slug>.md` at cwd root (legacy mode)
```

Wait for the answer. Then:

- **(a)** → create `notes/<YYYY-MM-DD>-<slug>/` + `<slug>-notes.md`
- **(b)** → use the user's parent name + `<YYYY-MM-DD>-<slug>/` + `<slug>-notes.md`
- **(c)** → create `<YYYY-MM-DD>-<slug>/` directly at cwd + `<slug>-notes.md`
- **(d)** → create single `<YYYY-MM-DD>-<slug>.md` at cwd (no folder, no artifacts support)

Remember the chosen structure for the rest of the topic. Future saves go in the same place. Print the absolute path of the notes file back to the user.

## Step 2 — Skeleton

```markdown
# <Topic>

**Goal:** <one-line statement of what we're working on>
**Mode:** investigation | interview | analysis | generic
**Status:** in progress
**Started:** <YYYY-MM-DD>
**Last updated:** <YYYY-MM-DD>

---

## Summary
*(running synthesis — readable standalone)*

## <Body section(s) — see mode-specific guidance below>

## Artifacts
*(list of side files in this folder, with one-line purpose each. Skip in mode (d) — no folder.)*

## Open flags
*(pending input + owner)*
```

## Step 3 — Body shape by mode

The body is **mode-shaped**, not a fixed template. Pick a structure that fits the topic. These are hints, not rules — the goal is efficient notes, not preserved conversation structure.

**Investigation / debug**:
- `## Findings` — claim · confidence (confirmed | probable | speculative) · evidence (`file:line`, query, commit)
- `## Reasoning chains` — for non-trivial conclusions, the path from evidence to claim
- `## Dead-ends explored` — hypothesis + why ruled out
- `## Open threads` — what's unverified, what's next

**Interview / grill** (consumed by `grill-me-M`):
- `## Decisions log` — consolidated topics: Decision · Why · Dead-ends · Flags

**Data analysis**:
- `## Hypotheses` — what we're testing
- `## Results` — finding + pointer to query/figure (NOT the full output)
- `## Surprises` — things that contradicted assumptions

**Generic**:
- Free-form distilled notes — pick sections as needed: `## Facts`, `## Decisions`, `## Pointers`, `## TODO`

## Step 4 — Side artifacts

When the topic produces an artifact that isn't note-text — HTML report, CSV export, diagram, screenshot, query result, scratch SQL, sub-doc — **save it into the topic folder** alongside the notes file. Use a descriptive filename (`research-report.html`, `candidates.csv`, `architecture.png`).

Update the notes file's `## Artifacts` section: `- file.ext — one-line purpose`.

In mode (d) — single file, no folder — there's no place for artifacts. Note that limitation if the topic starts producing them; offer to migrate to a folder structure.

## Step 5 — Checkpoint discipline

After every meaningful turn (finding, decision, ruled-out hypothesis, new open thread):

- Update the file with the new content
- Update `**Last updated:**`
- Apply the curation rules below — don't blindly append

Never batch. If context is lost at any moment, the file already holds everything.

## Step 6 — Curation rules (apply when updating)

The file is a **curated, detailed deliverable** — rich enough to derive other documents from (PRD, plan, summary), but never a raw transcript.

- **Capture**: decisions, load-bearing rationale, constraints discovered, options considered + why each was rejected, flags with owners, illuminating dead-ends. Be generous — keep enough detail that this file alone is enough to later derive a PRD, plan, or summary without losing the reasoning.
- **Drop**: conversational back-and-forth. Restated context. Exact user wording unless that wording IS the decision. Obvious duplication.

**Collapse rule**: when a thread of several exchanges resolves to one decision/finding, replace the intermediate entries with ONE consolidated entry (decision + key why + dead-ends worth remembering).

**Supersession rule**: when a later finding overrides an earlier one, REPLACE the earlier entry. Don't keep both. Add a one-line "changed from X because Y" only if the change itself is decision-relevant.

**Pivot rule**: when new content makes earlier entries stale (not wrong — no longer relevant because direction changed):
- If the pivot itself is illuminating ("tried X, hit Y, switched to Z"), replace the stale entries with ONE dead-end entry capturing the lesson.
- If the pivot is just "we changed direction, no lesson", DELETE the stale entries silently.
- Decisions that depended on the old direction get re-asked under the new framing.

Detect pivots proactively. When new content invalidates earlier entries, surface it: "These earlier entries assumed X; this pivots to Y — drop, keep as dead-end, or re-ask?"

The `## Summary` should be readable standalone — if someone only reads that section, they have the full picture.

## Step 7 — Status & lifecycle

- **`in progress`** — active work; sessions end and resume against the same folder
- **`parked`** — set aside, may resume later; add a `## Parked because` note
- **`concluded`** — done; add a `## Outcome` section (answer/decision + links to PR, ticket, ADR)

**End of session vs end of topic**:
- End of *session* (stopping for now): file stays `in progress`. No reconcile beyond the last checkpoint.
- End of *topic* (resolved or parked): final reconcile — read the file for contradictions, fix them, set status, give a short recap. Consider archiving the whole folder.

## Rules

- **Append cheaply, distill aggressively** — efficient notes beat preserved conversation structure
- **Be concise** on context an agent could re-read — point to `file:line`, don't paste full files
- **Redact secrets** (API keys, tokens, passwords, PII) — in the notes file AND in side artifacts
- **Print the absolute path** of the notes file when creating or reopening
- **Before `/compact`**: offer to write a "current state" summary entry so the post-compact agent (or future you) can resume cleanly
