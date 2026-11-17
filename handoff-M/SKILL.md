---
name: handoff-M
description: Hand off ONE topic or sub-thread of the conversation to another agent while you keep working in the current session — a bug spotted on the side, a tangential idea, a follow-up that deserves its own agent. Not for wrapping up the whole session; that is `/pre-compact`.
argument-hint: "<topic description>"
---

# Handoff (one topic)

Write a handoff document so another agent can pick up **one sub-thread** of this conversation. **The current session continues** — do not summarize unrelated content, do not "wrap up" the current work.

Save to the **OS temp directory** — never the current workspace.

## Arguments

- The arguments are the topic to extract. A leading word `topic` is accepted and ignored (old syntax).
- No topic given → ask which topic to extract. The skill cannot proceed without knowing.
- First word is `full`, or the user wants the whole session handed over → stop and tell them to run `/pre-compact <what they want to do next>` instead. That skill owns whole-session saves; this one no longer does them.

## What goes in the document

- **Goal** — what the receiving agent should accomplish (verbatim from user args when provided)
- **Context** — only what the receiving agent needs: the topic's history within this conversation, what's known/unknown, current state of that thread, why it was set aside
- **Suggested skills** — skills the receiving agent should invoke for this work
- **Next steps** — concrete first moves the receiving agent can take

## Rules

- **Do not duplicate** content already in PRDs, plans, ADRs, issues, commits, diffs — reference them by path or URL
- **Redact sensitive info** — API keys, passwords, PII, internal secrets
- **Save to OS temp dir**, print the absolute path back to the user
- After producing the doc, return cleanly so the user can resume the main thread — do not narrate next steps for the current session
