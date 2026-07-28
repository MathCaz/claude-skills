---
name: html-report-M
description: Generate a self-contained HTML report (dashboard, analysis write-up, results summary, status report) saved locally so it can be versioned in GitHub, with an optional shareable claude.ai Artifact replica for non-technical stakeholders (PO, leadership). Use whenever the user asks for an "HTML report", "report", "dashboard", "visual summary", "write this up as HTML", or wants findings/data presented as a viewable page.
when_to_use: User asks to produce an HTML report/dashboard/visual summary, or to turn findings, analysis, or data into a shareable viewable page.
argument-hint: "[report title or topic]"
---

# HTML Report (html-report-M)

Produce **one self-contained HTML file** (all CSS/JS inline, images as data URIs — no external requests). Self-contained is mandatory: the same file must open locally AND drop into a claude.ai Artifact unchanged.

## Step 1 — Build the report

- Load the **dataviz** skill BEFORE writing any chart, graph, KPI tile, or table styling. Follow it.
- If the report is design-heavy (stakeholder-facing), load **artifact-design** for layout/typography calibration.
- Make it **responsive** (relative units, flex/grid, `max-width:100%` on media; wide tables/diagrams scroll inside their own `overflow-x:auto` container — body never scrolls sideways).
- Make it **theme-aware** (light + dark via `prefers-color-scheme`) unless a single look is intentional.
- Write the page content directly under a normal HTML skeleton. Keep a concise, stable `<title>`.
- No external fonts, CDN scripts, or remote images — inline/embed everything.

## Step 2 — Save locally (versionable)

Determine where the file goes:

1. **A notes-M topic is active** (a `<slug>-notes.md` path is known in this session) →
   - Save into that topic folder, descriptive filename (e.g. `mart-freshness-report.html`).
   - Add/refresh the entry under `## Artifacts` in the notes file: `- <file> — <one-line purpose>`.
   - Print the absolute path back.

2. **No active notes-M topic** → ASK the user where to save, offering:
   - Root of the current working repo (default), or
   - Another path they name.
   - Use a descriptive kebab-case filename. Create the folder if needed (never delete/overwrite an existing file without confirming). Print the absolute path back.

Reports are meant to be committed to GitHub — do not commit unless the user asks; just save the file where their repo can track it.

## Step 3 — Offer the claude.ai Artifact replica

After the local file is saved, ALWAYS ask:

> "Also publish a shareable copy as a claude.ai Artifact (for non-technical folks — PO, stakeholders)?"

- **If yes** → publish the SAME file via the `Artifact` tool (load **artifact-design** first if not already). Set a matching `<title>`, a one-sentence `description`, and a stable `favicon` emoji. Return the artifact URL to the user, noting it is private until they share it.
- **If no** → stop after the local save.
- On later edits: re-save the local file first, then redeploy to the same Artifact URL (same file path) so the shared link stays stable.

## Rules

- Local file and Artifact are the **same self-contained HTML** — build once, reuse.
- Never fabricate data — report only what's in the conversation/sources; label any estimates.
- Redact secrets (keys, tokens, PII) from the report.
- Keep the local save and the artifact question as two distinct steps — local first, artifact second, always ask.
