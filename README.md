# skills

Personal collection of [ZCode](https://z.ai) agent skills. Each subdirectory is a self-contained skill with a `SKILL.md` entry point.

## Skills

| Skill | What it does |
|-------|--------------|
| [chrome-bookmark-reorganize](chrome-bookmark-reorganize/) | Reorganize Chrome bookmarks on macOS, handling the local `Bookmarks` file, Google sync conflicts, and Chromium checksum recomputation. |

## Usage

Skills here are loaded from `~/.agents/skills/`. To install one, copy or symlink its directory:

```bash
# Symlink (recommended — stays in sync with this repo)
ln -s "$(pwd)/chrome-bookmark-reorganize" ~/.agents/skills/

# Or copy
cp -R chrome-bookmark-reorganize ~/.agents/skills/
```

## Layout convention

```
skill-name/
├── SKILL.md          # entry point — frontmatter + workflow (keep < 100 lines)
├── scripts/          # reference implementations / templates
└── references/       # deep-dive docs, linked from SKILL.md
```

See [chrome-bookmark-reorganize](chrome-bookmark-reorganize/) as a reference example.

## License

Private repository. For personal use.
