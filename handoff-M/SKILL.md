---
name: handoff-M
description: Hand off either the full conversation OR a single topic/sub-thread to another agent. Use `full` to compact the entire session for a fresh agent (current session typically ends). Use `topic` to extract one sub-thread (a bug spotted on the side, a tangential idea) for another agent while you keep working in the current one.
argument-hint: "full <next session focus> | topic <topic description>"
---

# Handoff

Write a handoff document so another agent can pick up specific work. Save to the **OS temp directory** — never the current workspace.

## Modes

The first arg word selects the mode:

- **`full <focus>`** — compact the **entire** conversation. Use when you're wrapping up this session and want a fresh agent to continue the work. The current session is typically ended after.
- **`topic <topic description>`** — extract **only the sub-thread** matching the topic. Use when a side-issue surfaced (a bug, a tangential idea) that deserves its own agent while you stay on the main thread here. **The current session continues** — do not summarize unrelated content, do not "wrap up" the current work.

If no mode is given, ask the user which one. For **`topic`** without a description, ask which topic to extract — the skill cannot proceed without knowing. For **`full`** without a description, proceed with a comprehensive summary; a description is optional and only biases the doc's framing.

## What goes in the document (both modes)

- **Goal** — what the receiving agent should accomplish (verbatim from user args when provided)
- **Context** — only what the receiving agent needs to be effective
  - **Full mode:** comprehensive — decisions made, current state, open items, blockers
  - **Topic mode:** focused — the topic's history within this conversation, what's known/unknown, current state of that thread, why it was set aside
- **Suggested skills** — skills the receiving agent should invoke for this work
- **Next steps** — concrete first moves the receiving agent can take

## Rules (both modes)

- **Do not duplicate** content already in PRDs, plans, ADRs, issues, commits, diffs — reference them by path or URL
- **Redact sensitive info** — API keys, passwords, PII, internal secrets
- **Save to OS temp dir**, print the absolute path back to the user
- **Topic mode only:** after producing the doc, return cleanly so the user can resume the main thread — do not narrate next steps for the current session
