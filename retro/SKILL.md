---
name: retro
description: Run a retro on something that just went wrong in this session (rabbit hole, needless question, repeated mistake) and turn it into a permanent fix at the right layer.
disable-model-invocation: true
argument-hint: <what bothered you>
---

# /retro: turn a frustration into a permanent fix

The user is unhappy with how the work just went. Complaint: **$ARGUMENTS**

If the complaint is empty, ask ONE short question ("What bothered you?") and wait.

## Steps

1. **Pause the current task.** Don't defend or explain what you did. Don't try to fix the task itself yet.

2. **Hand off to the `retro-analyst` agent** (Agent tool, subagent_type `retro-analyst`). Pass ONLY:
   - the user's complaint, word for word
   - the current working directory
   - the transcript folder: `~/.claude/projects/<cwd with every "/" replaced by "-">/` (the analyst picks the most recently modified `.jsonl`)

   Do NOT give your own account of what happened. The analyst reads the transcript itself, so its view isn't biased by yours.

3. **Show the analyst's report to the user** as it is, in its fixed format. Then ask: *"Apply this fix? (yes / adjust / log only)"*.

4. **On "yes" or an adjusted version:**
   - Apply the fix exactly as approved, in its layer (hook, skill, agent, CLAUDE.md or note for the user).
   - If the fix is big (new test suite, refactor, multi-file change), don't do it now: log it as 🟡 to do.
   - Never write anything the user didn't approve.

5. **Append an entry to `~/.claude/retro/log.md`**, newest first under `## Entries`, using the template at the top of that file. Status: 🔧 fix applied, or 🟡 to do.

6. **Resume the original task**, now following the fix. Say in one line what changed.

Be extremely concise throughout.
