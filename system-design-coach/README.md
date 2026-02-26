# 🧰 OpenClaw Skills Collection

My personal collection of [OpenClaw](https://github.com/openclaw/openclaw) skills.

## Skills

| Skill | Description |
|-------|-------------|
| [system-design-coach](skills/system-design-coach/) | 🏗️ AI-powered system design learning coach. Interactive exercises, architecture teardowns, design reviews, and study planning. |

## Structure

```
skills/
├── system-design-coach/     # System design learning companion
│   ├── SKILL.md             # Main skill definition
│   └── references/          # Topic guides, exercises, teardowns
└── <future-skills>/         # More skills to come...
```

## How to Use

### With OpenClaw

Copy or symlink a skill directory into your OpenClaw workspace:

```bash
# Copy
cp -r skills/system-design-coach ~/.openclaw/workspace/skills/

# Or symlink
ln -s $(pwd)/skills/system-design-coach ~/.openclaw/workspace/skills/system-design-coach
```

### Skill Format

Each skill follows the [OpenClaw Skill spec](https://docs.openclaw.ai):

- `SKILL.md` — Main entry point with frontmatter metadata + instructions
- `references/` — Supporting docs loaded on demand
- `scripts/` — Executable automation (if needed)
- `assets/` — Templates, images, etc. (if needed)

## License

MIT
