# /pblog - Work Logging Skill

Log daily work to Obsidian vault using the [pblog CLI tool](https://github.com/patrickmburrell/pblog).

## Requirements

- **pblog CLI** installed (`uv tool install --editable ~/Projects/PMB/pblog` or similar)
- Obsidian vault configured in `~/.config/pblog/config.toml`

## Installation

```bash
# Clone or copy to Claude Code skills directory
git clone https://github.com/patrickmburrell/pblog-skill.git ~/.claude/skills/pblog
```

## Usage

Use when:
- User wants to log work, accomplishments, or commits
- User mentions pblog or PB Logger
- User asks to capture or document today's work

See [SKILL.md](SKILL.md) for complete guide.

## Features

- Log work entries with optional git commit integration
- Add detailed sub-bullets atomically with `--details`
- Retroactive entries with `--date YYYY-MM-DD`
- Auto-detect project from `.pblog` file
- Structured markdown output to Obsidian daily notes

## Important

**Always use `--date YYYY-MM-DD`** to avoid UTC date rollover issues. pblog uses UTC internally, so entries without an explicit date may appear on the wrong day in your local timezone.

## Files

- `SKILL.md` — Main skill guide
- `evals/` — Skill evaluation test cases

## License

MIT
