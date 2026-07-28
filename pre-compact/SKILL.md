---
name: pre-compact
description: Pre-compact ritual — before you run /compact (or end the session), save the full session state durably and hand back a polished, ready-to-paste "come-back prompt" that resumes the work with an explicit next task baked in. Use when the user says "I want to compact", "save everything and give me the come-back prompt", "prep for compaction", "wrap this up so I can resume later", especially when followed by "next I want to: <X>".
when_to_use: User is about to /compact or end the session and wants state saved plus a resume prompt, usually stating what they want to do next.
argument-hint: "<what you want to do next>"
---

# Compact prep + come-back prompt (compact-M)

Two deliverables, in order: (1) save the full session state to a durable file, (2) print a self-sufficient come-back prompt the user can paste to resume. This skill does NOT run `/compact` — the user runs it after.

Distinct from `handoff-M full` (which briefs a *fresh* agent, temp doc only). Here the point is resuming *this* line of work after compaction/session-restart, with the user's next task driving it.

## Step 1 — Save full state (durable)

Write a `resume state` capturing everything needed to continue:

- **Goal** — what this work is ultimately for (one line)
- **Where we are** — what's done, current position, what just happened
- **Key decisions** — with the load-bearing *why* (so they aren't re-litigated)
- **Constraints / gotchas** — discovered facts, dead-ends ruled out, things to avoid
- **Pointers** — relevant `file:line`, paths, PRs, tickets, queries (reference, don't paste)
- **Open threads** — unresolved items
- **Next** — the user's stated next task (verbatim from args), sharpened into concrete first steps

Location:
1. **notes-M topic active** → checkpoint into that topic's notes file (update `## Summary` + a `## Current state` entry, bump `Last updated`). This IS the notes-M "before /compact" checkpoint. Print the absolute path.
2. **No active topic** → save to the OS temp dir (e.g. `compact-resume-<slug>.md`), print the path, and note: "temp copy — want a versioned copy in your repo instead?" Don't block on the answer; the come-back prompt (Step 2) is self-sufficient regardless.

Redact secrets (keys, tokens, PII).

## Step 2 — Print the come-back prompt

Output a single fenced block the user can copy wholesale. Make it **self-sufficient** — enough inline context to resume even if the saved file is gone. Sharpen the user's raw "next" into an actionable directive, not a restatement.

```
Resume: <one-line goal>

## Where we are
<concise state: done / current position / just happened>

## Key context
<load-bearing decisions + why · constraints · gotchas · file:line pointers>

## Skills to (re)invoke
<e.g. notes-M topic at <path>; any others in play>

## Do next
<the sharpened next task → concrete first step(s)>

Full saved state: <path>
```

Keep it tight — favor pointers over pasted content. After printing, tell the user they can now run `/compact` (or start a fresh session and paste the block).

## Rules

- Come-back prompt is the primary deliverable; the saved file is the durable backup.
- Don't duplicate content that already lives in PRDs/plans/ADRs/commits — reference by path/URL.
- Sharpen "next", don't parrot it. Point to `file:line`, don't paste files.
