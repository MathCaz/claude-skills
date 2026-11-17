---
name: skill-M
description: Personal checklist for writing or reviewing a Claude Code SKILL.md — who invokes it, description wording, length, frontmatter traps. Use when drafting a new skill or reviewing one before saving. For evals, benchmarks or description-trigger tuning, use `anthropic-skills:skill-creator` instead.
---

# Skill checklist (skill-M)

For field-by-field reference, read the official docs: https://code.claude.com/docs/en/skills. For testing a skill (evals, trigger tuning), use `anthropic-skills:skill-creator`. This file is only the checklist.

## Process

1. Ask: what task, which concrete situations trigger it, scripts or instructions only, any reference material.
2. Draft `~/.claude/skills/<name>/SKILL.md` (or `.claude/skills/` in a repo).
3. Run the checklist below with the user before saving.

## Checklist

- **Who invokes it.** Side effects (creates tickets, sends, publishes, writes outside the repo) → `disable-model-invocation: true` (me only). Background knowledge Claude should pull in on its own → leave it model-invocable. Background knowledge I'd never type → `user-invocable: false`.
- **Description matches the setting.** A me-only skill must not say "use when…" phrases aimed at Claude; they do nothing and mislead.
- **Description is narrow.** Key use case first, concrete trigger phrases, plus one "Not for: …". Avoid generic words that every other skill also claims ("report", "dashboard", "investigate", "debug"). Check the live skill list for collisions.
- **Description budget.** `description` + `when_to_use` ≤ 1,536 chars.
- **Short.** SKILL.md well under 500 lines; it stays in context all session. Move long references to `references/*.md` and link them.
- **Scripts for deterministic steps**, not prose.
- **No dead references.** Every skill, tool, MCP operation, path or person named still exists. No colleague names or personal clones in examples.
- **No surprise shell injection.** An exclamation mark directly against a backticked command runs it when the skill loads. In prose or examples, keep a space between them.
- **No hardcoded model ids**; they go stale.
- **Arguments** shown with one example (`argument-hint`, `$ARGUMENTS`).
- **Name**: lowercase, hyphens, ≤ 64 chars; folder name = `name`.
