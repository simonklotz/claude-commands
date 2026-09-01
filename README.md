# claude-commands

Development home for my custom [Claude Code slash commands](https://code.claude.com/docs/en/slash-commands).
The markdown files in this repo are the tracked source of truth; Claude Code itself reads them
from `~/.claude/commands/`, so a command goes live by copying (or symlinking) it there:

```bash
ln -s "$(pwd)/init-claude.md" ~/.claude/commands/init-claude.md
```

Each `<name>.md` file becomes the user-level command `/<name>`, available in every repo. The
frontmatter `description` is what the `/` picker shows.

## Commands

| Command | Purpose |
| --- | --- |
| `/init-claude` | Replacement for `/init`: sets up minimal Claude Code memory + git/gh guardrails in a fresh repo (Angular, TS/JS, or any stack via a project-type picker) |
