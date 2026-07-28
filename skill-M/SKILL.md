---
name: skill-M
description: Create well-structured Claude Code skills following official best practices. Use when building new skills, improving existing ones, or understanding skill architecture. Covers frontmatter, invocation control, and performance optimization.
when_to_use: When you want to create, refactor, or improve a Claude Code skill; need guidance on frontmatter fields; or understand when to use advanced features like subagents and dynamic context injection.
---

This skill will be invoked when you want to create or improve a Claude Code skill. You may skip sections if you don't consider them necessary.

## Process

When building a new skill, work through these steps with the user:

1. **Gather requirements** — ask the user:
   - What task or domain does the skill cover?
   - What specific use cases should it handle?
   - Does it need executable scripts, or just instructions?
   - Any reference materials to include?

2. **Draft the skill** — create:
   - `SKILL.md` with concise instructions
   - Additional reference files if content would exceed 500 lines (see *Performance Best Practices*)
   - Utility scripts if deterministic operations are needed (see *Write Effective Skill Content*)

3. **Review with the user** — present the draft and ask:
   - Does this cover your use cases?
   - Anything missing or unclear?
   - Should any section be more or less detailed?

   Then validate against the *Checklist for Good Skills* below before finalizing.

## Skill File Structure

Every skill needs a `SKILL.md` file with two parts:
1. **YAML frontmatter** (between `---` markers) — tells Claude when/how to use the skill
2. **Markdown content** — instructions Claude follows when the skill runs

Skills live in directories: `.claude/skills/<skill-name>/SKILL.md`

## Required and Recommended Frontmatter

```yaml
---
name: my-skill
description: What this skill does. Include key use case first — Claude uses this to decide when to apply the skill.
---
```

**Critical fields:**
- `name` (optional) — Skill identifier. Lowercase letters, numbers, hyphens only. Max 64 chars. Defaults to directory name.
- `description` (recommended) — What it does + when to use it. Claude uses this to decide whether to load the skill. Combined with `when_to_use`, capped at 1,536 characters. Put key use case FIRST.

## Control Who Invokes the Skill

- `disable-model-invocation: true` — Only you can invoke it (use for workflows with side effects: `/commit`, `/deploy`, `/send-slack-message`)
- `user-invocable: false` — Only Claude can invoke it (use for background knowledge users shouldn't run directly)

```yaml
---
name: deploy
description: Deploy the application to production
disable-model-invocation: true
---
```

## Advanced Frontmatter Fields

**Execution context:**
- `context: fork` — Run in isolated subagent (forked context, doesn't access conversation history)
- `agent: Explore` — Which subagent type (Explore, Plan, general-purpose, or custom)
- `allowed-tools: Bash(git *) Read` — Pre-approve tools for this skill without permission prompts

**Model and performance:**
- `model: claude-opus-4-7` — Override model for this skill
- `effort: high` — Override effort level (low, medium, high, xhigh, max)

**Content and arguments:**
- `when_to_use: "Additional trigger phrases or examples"` — Appended to description (counts toward 1,536 char cap)
- `argument-hint: "[issue-number]"` — Hint for autocomplete
- `arguments: [issue, branch]` — Named arguments for `$name` substitution in content

**Conditional activation:**
- `paths: "src/**/*.ts,*.js"` — Glob patterns. Skill only auto-loads when working with matching files.

**Shell and hooks:**
- `shell: bash` — bash (default) or powershell
- `hooks: {...}` — Lifecycle hooks scoped to this skill

## Write Effective Skill Content

Keep it concise — skill content stays in context across turns, so every line is a token cost.

**Types of content:**

1. **Reference content** (runs inline) — Conventions, patterns, domain knowledge
   ```yaml
   ---
   name: api-conventions
   description: API design patterns for this codebase
   ---
   
   When writing API endpoints:
   - Use RESTful naming conventions
   - Return consistent error formats
   ```

2. **Task content** (explicit instructions) — Steps for a specific action. Often paired with `disable-model-invocation: true`
   ```yaml
   ---
   name: commit
   description: Stage and commit the current changes
   disable-model-invocation: true
   ---
   
   1. Stage all modified files
   2. Write a conventional commit message
   3. Verify the commit
   ```

## Performance Best Practices

**Description optimization:**
- Put key use case FIRST — combined description + `when_to_use` capped at 1,536 chars
- Claude uses descriptions to decide when to load skills
- Real-world testing: 20% activation baseline → 50% with optimized description → 90% with examples

**Character budget:**
- Skill descriptions consume 1% of context window dynamically (fallback: 8,000 chars)
- Set `SLASH_COMMAND_TOOL_CHAR_BUDGET` environment variable to raise limit
- Use `skillOverrides` in settings to mark low-priority skills as `"name-only"` to free budget

**File structure:**
- Keep `SKILL.md` under 500 lines
- Move detailed reference material to separate files and reference them:
  ```markdown
  For complete API details, see [reference.md](reference.md)
  For usage examples, see [examples.md](examples.md)
  ```

## Advanced Patterns

**Dynamic context injection** — Run shell commands before Claude sees the prompt:
```yaml
---
name: pr-summary
description: Summarize a pull request
context: fork
agent: Explore
allowed-tools: Bash(gh *)
---

## PR Context
- Diff: !`gh pr diff`
- Comments: !`gh pr view --comments`
```

The `` !`<command>` `` executes immediately, output replaces the placeholder.

**String substitutions** in skill content:
- `$ARGUMENTS` — All arguments passed to the skill
- `$0`, `$1`, etc. or `$ARGUMENTS[0]` — Specific argument by position
- `$name` — Named argument (declared in `arguments` field)
- `${CLAUDE_SESSION_ID}` — Current session ID
- `${CLAUDE_SKILL_DIR}` — Directory containing SKILL.md

Example:
```yaml
---
name: fix-issue
description: Fix a GitHub issue
disable-model-invocation: true
---

Fix GitHub issue $ARGUMENTS following our coding standards.
```

Running `/fix-issue 123` renders as "Fix GitHub issue 123..."

## Skill Lifecycle

- When invoked, `SKILL.md` content enters conversation as a message and stays for the session
- Claude Code doesn't re-read the skill file on later turns — write standing instructions
- During [auto-compaction](https://code.claude.com/docs/en/how-claude-code-works#when-context-fills-up), skills are re-attached (first 5,000 tokens of each)
- If skill stops influencing behavior after first response, strengthen the description/instructions

## Checklist for Good Skills

- ✅ Description is specific and includes use case (key phrase first)
- ✅ Frontmatter declares whether Claude or you invokes it
- ✅ Content is concise (under 500 lines in SKILL.md)
- ✅ Reference material in separate files with links
- ✅ Arguments documented with examples if applicable
- ✅ For task skills: `disable-model-invocation: true` if user-controlled
- ✅ For knowledge skills: `user-invocable: false` if Claude-only
- ✅ Supporting files organized (templates, examples, scripts)
- ✅ Shell commands injected with `` !`command` `` when live data needed

## Resources

- [Official Claude Code Skills Docs](https://code.claude.com/docs/en/skills)
- [Agent Skills Overview](https://platform.claude.com/docs/en/agents-and-tools/agent-skills/overview)
