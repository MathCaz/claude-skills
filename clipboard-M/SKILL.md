---
name: clipboard-M
description: Copy content to the macOS clipboard (pbcopy) so the user can paste it directly. Use when the user says "copy to clipboard", "put it in the paperclip", "so I can just paste", or wants a file / snippet / command ready to paste elsewhere (SQL worksheet, editor, chat).
---

# Clipboard (clipboard-M)

Pipe content to `pbcopy` so the user pastes instead of hand-copying.

## Recipes

- **A whole file:**
  `cat "<path>" | pbcopy`
- **A file with lines prepended** (e.g. `USE ROLE ...;` before a SQL runbook):
  `{ printf '<line1>\n<line2>\n\n'; cat "<path>"; } | pbcopy`
- **Arbitrary text** (heredoc keeps it exact, no shell expansion):
  `pbcopy <<'EOF'` … `EOF`
- **Command output:** `<cmd> | pbcopy`

## Rules

- Always confirm with a byte/line count so the user knows it worked:
  `... | pbcopy && echo "copied ($(pbpaste | wc -l | tr -d ' ') lines)"`
- Never copy secrets/credentials to the clipboard unless the user explicitly asks.
- macOS only (`pbcopy`/`pbpaste`). On Linux fall back to `xclip -selection clipboard` / `wl-copy`.
- Don't echo the full content back into the chat after copying — the point is to avoid the wall of text.
