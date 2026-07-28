# claude-skills

A small, curated set of [Claude Code](https://code.claude.com/docs/en/skills) **skills** I use daily — for note-taking that survives `/compact`, stress-testing plans, handing work between agents, authoring new skills, and keeping SQL consistent.

Skills are folders of Markdown that Claude Code loads on demand: only each skill's `name` + `description` sit in context until it's triggered, then the full `SKILL.md` loads. Cheap to keep installed, powerful when they fire.

## Install

Each top-level folder is one skill. Drop the ones you want into `~/.claude/skills/`:

```bash
git clone https://github.com/MathCaz/claude-skills.git
# copy the ones you want (repeat per skill):
cp -R claude-skills/notes-M       ~/.claude/skills/
cp -R claude-skills/html-report-M ~/.claude/skills/
# …or symlink instead so `git pull` keeps them current:
ln -s "$(pwd)/claude-skills/notes-M"       ~/.claude/skills/
ln -s "$(pwd)/claude-skills/html-report-M" ~/.claude/skills/
```

Restart Claude Code (or start a new session) and the skills are available. Invoke an explicit one with `/<skill-name>`; the auto-invoked ones fire on their own when their triggers match.

## A note on the `-M` suffix

The `-M` is just my personal namespace — it marks these as *my* variants so they don't collide with built-ins or other people's skills of the same name. Rename freely; nothing depends on the suffix.

---

## The skills

### `grill-me-M` — get your plan interrogated
**Invoke:** `/grill-me-M` (user-only; won't auto-fire)

Relentlessly interviews you about a plan, design, or decision — one question at a time, walking every branch of the decision tree, resolving upstream choices before the ones that depend on them. Every question comes with lettered options and a recommended answer so you can reply with a single letter. Captured decisions are written to a durable notes file (delegated to `notes-M`, in `interview` mode) and checkpointed after *every* answer, so nothing is lost if the session dies.

> Use it when you catch yourself saying "I think the plan is fine" — it finds the parts that aren't.

### `notes-M` — persistent, distilled notes per topic
**Invoke:** auto-fires on any multi-step topic; or `/notes-M`

The backbone skill. Each topic becomes a self-contained folder — `<parent>/<YYYY-MM-DD>-<slug>/<slug>-notes.md` plus any side artifacts (reports, exports, diagrams). The notes file is **curated, not a transcript**: it captures decisions, rationale, ruled-out options and open flags, and applies collapse/supersession/pivot rules on every update so it stays a clean deliverable you could derive a PRD or summary from. Survives `/compact`, session end, and multi-day work.

> Several other skills here (`grill-me-M`, `pre-compact`, `html-report-M`) delegate their file mechanics to this one.

### `html-report-M` — findings as a self-contained HTML report
**Invoke:** `/html-report-M [title]`, or auto when you ask for a "report" / "dashboard"

Turns findings, analysis, or data into **one self-contained HTML file** — all CSS/JS inline, images as data URIs, so it opens offline and drops into a claude.ai Artifact unchanged. Responsive and theme-aware (light/dark). Saves locally so you can version it in git (into the active `notes-M` topic folder if one is open, else wherever you say), then offers to publish a shareable claude.ai **Artifact** replica for non-technical stakeholders (PO, leadership) — same file, so the link stays stable across edits.

> Local, versionable deliverable first; optional shareable copy second. (For throwaway "just make this terminal output readable" HTML, that's a separate, disposable-by-design skill.)

### `handoff-M` — pass work to another agent
**Invoke:** `/handoff-M full <focus>` or `/handoff-M topic <description>`

Writes a handoff document (to the OS temp dir, never your workspace) so a *different* agent can pick up. Two modes: `full` compacts the entire conversation for a fresh agent taking over (this session usually ends); `topic` extracts just one sub-thread — a bug you spotted on the side, a tangent worth its own agent — while you keep working here. It references PRDs/commits/issues by path rather than duplicating them.

> `full` = "someone else takes over." `topic` = "spin this side-thing off, I'll stay on the main thread."

### `pre-compact` — checkpoint before `/compact`
**Invoke:** `/pre-compact <what you want to do next>`

The ritual before you compact or close a session. Saves the full resume state durably (into the active `notes-M` topic if one is open, else a temp file), then prints a **self-sufficient, ready-to-paste "come-back prompt"** — goal, current state, load-bearing decisions, gotchas, and your sharpened next task — so a post-compact (or next-day) agent resumes cleanly with no re-litigating. It does *not* run `/compact` — you do that after.

> Distinct from `handoff-M full`: that briefs a *fresh* agent; this resumes *your own* line of work.

### `skill-M` — author well-structured skills
**Invoke:** `/skill-M`, or auto when you're building/refining a skill

A meta-skill: everything about writing good Claude Code skills — frontmatter fields (`description`, `disable-model-invocation`, `paths`, `context: fork`, `allowed-tools`, …), invocation control, dynamic context injection with `` !`command` ``, string substitutions, performance/character-budget tips, and a final checklist. Use it to create a new skill or tighten an existing one.

> The skill that builds skills. Start here if you're new to authoring them.

### `sql-style-M` — consistent SQL, Snowflake-flavoured
**Invoke:** auto-applies to `**/*.sql`, `dbt_project.yml`, and `dagster/**/*.py`

Style and structure rules for Claude-written SQL: layout, aliasing (descriptive initials, never single letters), clause order, CTE conventions (source CTEs, `_cte` suffix, numbered sectioning), JOIN formatting, `WHERE 1=1` neutral-element prefixes, audit-column lineage naming, and a Snowflake section (`QUALIFY`, `IFF`, `* EXCLUDE`, `HASH` surrogate keys). General rules are dialect-neutral; the Snowflake section is clearly fenced off so it's easy to swap for another warehouse.

> Vendor-neutral in the general section; the Snowflake-specific rules are opt-out.

---

## How they fit together

```
grill-me-M    ──delegates files──►  notes-M  ◄──checkpoints into── pre-compact
html-report-M ──saves artifacts──►    ▲
                                      │ (durable topic folders)
skill-M  (author any of these)        │
sql-style-M  (auto on SQL)            │
handoff-M  (temp doc, hand to another agent)
```

`notes-M` is the shared substrate; the interview, compact, and report skills all write through it. `handoff-M` deliberately writes *outside* your workspace (temp) since it targets a different agent.

## Credits

These are shaped by ideas from people who've done a lot of the thinking out loud on agent skills — thank you:

- **[Matt Pocock](https://github.com/mattpocock)** — the skill-structure conventions and the broader `mattpocock/skills` ecosystem that several of my other skills track and sync against.
- **Nate Herk** — the original "grill me" concept that `grill-me-M` builds on.

Any rough edges are mine, not theirs.

## License

MIT — do what you like with them. See [LICENSE](LICENSE).
