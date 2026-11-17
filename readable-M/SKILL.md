---
name: readable-M
description: Render the last long CLI answer as a temporary browser-readable HTML file (`temp-<date>-<time>-<slug>.html`). Each section is prefixed `[N]` and anchored `id="section-N"` for easy reference ("expand [3]"). Saved in the active notes-M topic folder if one is open, else `~/claude-temp-html/`. Triggers: "readable", "make this readable", "html this", "temp html", "hard to read", "open in browser", or `/readable-M`.
disable-model-invocation: false
---

# Readable (readable-M)

Ephemeral HTML for browser reading. Distinct from `html-report-M` (durable, versioned).

## 1. Cleanup pass first

`ls -lh ~/claude-temp-html/temp-*.html` + `ls -lh <notes-folder>/temp-*.html`. If any exist, list them (size, age) and ask which to delete before creating new. Same handling when user says "delete temp file(s)" / "clean up temp html".

## 2. Where + filename

- **Where**: active notes-M topic folder if `<slug>-notes.md` known this session, else `~/claude-temp-html/` (mkdir if missing). Never cwd.
- **Filename**: `temp-<YYYY-MM-DD>-<HHMMSS>-<short-slug>.html`. `temp-` prefix is non-negotiable.

## 3. Parse into sections

- `##` / `###` headers → one section each
- Plain paragraphs → group into 3–8 sections by topic
- < 40 lines → ask "seems short — still want it?"
- Number `[1]`, `[2]`, ... — prefix section titles AND set `id="section-N"` on the wrapping element

## 4. HTML output

Self-contained (no CDN), respects `prefers-color-scheme`, has phone viewport meta. Include:
- `<title>` = topic
- TOC at top linking to each `#section-N`
- `<main>` with `<section id="section-N"><h2>[N] Title</h2>...`
- Monospace + horizontal scroll for `<pre><code>`; bordered tables with padding
- Footer with `rm <full-path>` command

## 5. Open + report

`open <path>`. Then print: absolute path, section titles list, deletion hint.

## Cross-session nudge

During ANY conversation, if you notice `temp-*.html` files (via `ls` or user reference), mention ONCE if count > 2 or age > 7 days:
> "FYI N temp HTML files in <location>. Clean up when convenient."

## Rules

- `temp-` prefix always (disposability signal)
- Save to notes folder OR `~/claude-temp-html/` — never cwd
- Redact secrets (API keys, tokens, PII) in rendered content
- Self-contained HTML (works offline)
- Print absolute path so user can `rm` manually
