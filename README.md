# claude-skills

A small, curated set of [Claude Code](https://code.claude.com/docs/en/skills) **skills** I use daily — for note-taking that survives `/compact`, stress-testing plans, handing work between agents, step-by-step learning, auditing numeric documents, retros on what went wrong, authoring new skills, and keeping SQL consistent.

Skills are folders of Markdown that Claude Code loads on demand: only each skill's `name` + `description` sit in context until it's triggered, then the full `SKILL.md` loads. Cheap to keep installed, powerful when they fire.

## Install

Each top-level folder is one skill (`agents/` holds the one agent `retro` needs). Drop the ones you want into `~/.claude/skills/`:

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
**Invoke:** `/notes-M <topic>`, or auto on "take notes" / a topic that runs beyond ~5 turns

The backbone skill. Each topic becomes a self-contained folder — `<parent>/<YYYY-MM-DD>-<slug>/<slug>-notes.md` plus any side artifacts (reports, exports, diagrams). The notes file is **curated, not a transcript**: it captures decisions, rationale, ruled-out options and open flags, and applies collapse/supersession/pivot rules on every update so it stays a clean deliverable you could derive a PRD or summary from. Survives `/compact`, session end, and multi-day work.

> Several other skills here (`grill-me-M`, `pre-compact`, `html-report-M`) delegate their file mechanics to this one.

### `html-report-M` — findings as a self-contained HTML report
**Invoke:** `/html-report-M [title]`, or auto when you ask for an "HTML report" / "write this up as HTML"

Turns findings, analysis, or data into **one self-contained HTML file** — all CSS/JS inline, images as data URIs, so it opens offline and drops into a claude.ai Artifact unchanged. Responsive and theme-aware (light/dark). Saves locally so you can version it in git (into the active `notes-M` topic folder if one is open, else wherever you say), then offers to publish a shareable claude.ai **Artifact** replica for non-technical stakeholders (PO, leadership), with a one-line warning first if the content looks private — same file, so the link stays stable across edits.

> Local, versionable deliverable first; optional shareable copy second. (For throwaway "just make this terminal output readable" HTML, that's a separate, disposable-by-design skill.)

### `handoff-M` — pass work to another agent
**Invoke:** `/handoff-M <topic description>`

Writes a handoff document (to the OS temp dir, never your workspace) for **one sub-thread** of the conversation — a bug you spotted on the side, a tangent worth its own agent — so a *different* agent can pick it up while you keep working here. It references PRDs/commits/issues by path rather than duplicating them. Whole-session saves belong to `pre-compact`.

> "Spin this side-thing off, I'll stay on the main thread."

### `pre-compact` — checkpoint before `/compact`
**Invoke:** `/pre-compact <what you want to do next>`

The ritual before you compact or close a session. Saves the full resume state durably (into the active `notes-M` topic if one is open, else a temp file), then prints a **self-sufficient, ready-to-paste "come-back prompt"** — goal, current state, load-bearing decisions, gotchas, and your sharpened next task — so a post-compact (or next-day) agent resumes cleanly with no re-litigating. It does *not* run `/compact` — you do that after.

> The only whole-session save. `handoff-M` hands off a single topic; this resumes *your own* line of work.

### `skill-M` — author well-structured skills
**Invoke:** `/skill-M`, or auto when you're drafting or reviewing a skill

A short checklist to run before saving a skill: who invokes it, a narrow description that matches that setting, length, dead references, accidental shell injection. For field-by-field reference it points to the official docs; for evals and trigger tuning it points to Anthropic's `skill-creator`.

> The checklist, not the manual.

### `sql-style-M` — consistent SQL, Snowflake-flavoured
**Invoke:** auto-applies to `**/*.sql`, `dbt_project.yml`, and `dagster/**/*.py`

Style and structure rules for Claude-written SQL: layout, aliasing (descriptive initials, never single letters), clause order, CTE conventions (source CTEs, `_cte` suffix, numbered sectioning), JOIN formatting, `WHERE 1=1` neutral-element prefixes, audit-column lineage naming, and a Snowflake section (`QUALIFY`, `IFF`, `* EXCLUDE`, `HASH` surrogate keys). General rules are dialect-neutral; the Snowflake section is clearly fenced off so it's easy to swap for another warehouse.

> Vendor-neutral in the general section; the Snowflake-specific rules are opt-out.

### `stepwise-M` — learn a procedure one step at a time
**Invoke:** `/stepwise-M <what to walk through>`, or auto on "guide me step by step"

Breaks a procedure into T steps, sends one per message with an `N/T` counter, and waits for `next`. Every step carries its why and a "how you know it worked". Keywords: `curious` (short deep dive on the current step), `recap`, `plan`, `back`, `skip`.

> You set the pace; Claude holds the map.

### `audit-prouve-M` — audit, fix, then prove every fix landed (French)
**Invoke:** `/audit-prouve-M <file> <sections>`

For long documents full of numbers (reports, financial models). Parallel read-only auditors on a frozen snapshot, each returning a table that includes the exact wrong string. Fixes go through a throwaway Python script that asserts every anchor before writing. Then every reported wrong string is replayed against the file, so half-fixes (corrected in the table, left in the prose) can't hide.

> "Did the right value arrive?" is not enough — check the old one left.

### `readable-M` — make a long terminal answer readable
**Invoke:** `/readable-M`, or auto on "make this readable"

Renders the last long answer as a temporary, self-contained HTML page with numbered `[N]` sections and a table of contents, then opens it. Files are prefixed `temp-` and saved outside your working tree, so they're easy to clean up.

> Disposable by design; `html-report-M` is the durable one.

### `clipboard-M` — put it straight in the clipboard
**Invoke:** `/clipboard-M <file>`, or auto on "copy to clipboard"

Pipes a file, text or command output into `pbcopy` and confirms the line count, instead of printing a wall of text to copy by hand. macOS; Linux fallbacks noted.

### `jira-formatting` — Jira descriptions that render
**Invoke:** auto when writing a Jira issue or comment in Markdown through the Atlassian MCP

Three rules: real newlines (never `\n`), standard Markdown, clear sections.

### `retro` + `garden` — turn frustrations into permanent fixes
**Invoke:** `/retro <what bothered you>` (user-only) · `/garden` weekly (user-only)

`retro` hands your complaint and the session transcript to the read-only `retro-analyst` agent, which proposes **one** fix at the strongest layer that works (hook > skill > agent > CLAUDE.md > your own process). Nothing is applied without your yes; every retro is logged. `garden` reviews that log plus your CLAUDE.md, skills and agents weekly: repeats, duplicates, stale rules, contradictions.

**Setup:** copy `agents/retro-analyst.md` to `~/.claude/agents/`, and create `~/.claude/retro/log.md` with a `## Entries` heading (put your entry template above it).

> Retro fixes one incident; garden keeps the whole rule set small.

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
