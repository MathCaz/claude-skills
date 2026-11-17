---
name: garden
description: Weekly review of Claude's rules: merge duplicates, remove stale rules, resolve contradictions, promote repeated retro patterns.
disable-model-invocation: true
---

# /garden: weekly cleanup of the rule set

Retro fixes one incident at a time. Garden keeps the **whole** rule set small, consistent and true.

## Read
- `~/.claude/retro/log.md`: entries since the last `GARDEN` marker
- `~/.claude/CLAUDE.md`, plus the `./CLAUDE.md` of the current project if there is one
- All skills and agents: `~/.claude/skills/`, `~/.claude/agents/` (name, description, body)
- `~/.claude/settings.json`: hooks only

## Check
1. **Repeats**: the same root cause 2+ times in the log → the fix isn't strong enough. Propose a stronger layer (hook > skill > agent > CLAUDE.md).
2. **Duplicates**: the same rule in several places → keep ONE canonical home.
3. **Stale**: rules about tools, paths or projects that no longer exist, or that are never relevant.
4. **Contradictions**: for each one, decide:
   - **Confusion** → propose one clarified rule, or
   - **Different behaviour per task** → split into separate skills or agents, each with a clear scope.
5. **Size**: `~/.claude/CLAUDE.md` should stay short (target < 50 lines). Push detail into skills or docs (progressive disclosure).
6. **Weak skill descriptions**: vague descriptions make skills trigger wrongly. Propose sharper ones.
7. **Open items**: 🟡 to-do entries in the log. Remind the user.

## Output
A one-screen report, extremely concise:
```
GARDEN <date>: <N> retro entries reviewed
Repeats:        ...
Duplicates:     ...
Stale:          ...
Contradictions: ... → clarify | split
Size:           CLAUDE.md = <n> lines
Open to-dos:    ...
Proposed changes (numbered):
1. <file>: <change>
...
```
Then ask: *"Which changes should I apply? (all / numbers / none)"*.

## Apply
- Apply only the approved changes.
- Then append a marker at the top of the log's `## Entries`:
  `### <date> — GARDEN: <n> entries reviewed, changes: <short list>`
