# skills

Personal collection of agent skills, compatible with any agent runtime that supports the open skills format — [Claude Code](https://claude.com/claude-code), [Codex](https://github.com/openai/codex), ZCode, and others.

Each subdirectory is a self-contained skill with a `SKILL.md` entry point.

## Skills

| Skill | What it does |
|-------|--------------|
| [chrome-bookmark-reorganize](chrome-bookmark-reorganize/) | Reorganize Chrome bookmarks on macOS — handles the local `Bookmarks` file, Google sync conflicts, and Chromium checksum recomputation. |
| [to-kanban](to-kanban/) | 把一份需求（PRD / 需求文档 / 一句口述）拆成一张可直接执行的 Obsidian kanban-plugin 看板——列管时间窗口、标签管功能模块、勾管状态，并预留 icafe 卡号回填接口。 |
| [voice-memo-manager](voice-memo-manager/) | 列出、转写、摘要、删除通过 iCloud 同步的 Apple 语音备忘录——封装了元数据提取、转写脚本与删除流程。 |

## Install

Use the [Skills CLI](https://skills.sh/) (`npx skills`) to install a skill globally:

```bash
# Install a single skill from this repo
npx skills add ooneko/skills@chrome-bookmark-reorganize -g -y

# Browse / search the wider ecosystem
npx skills find chrome bookmarks
```

The `-g` flag installs at the user level (works across all agent runtimes); `-y` skips confirmation.

**Prerequisite:** Node.js. The first `npx skills` invocation downloads the CLI on demand — no separate install step.

### Manual install

If you prefer not to use the CLI:

```bash
git clone https://github.com/ooneko/skills.git
ln -s "$(pwd)/skills/chrome-bookmark-reorganize" ~/.agents/skills/
```

## Layout convention

```
skill-name/
├── SKILL.md          # entry point — frontmatter + workflow (keep < 100 lines)
├── scripts/          # reference implementations / templates
└── references/       # deep-dive docs, linked from SKILL.md
```

See [chrome-bookmark-reorganize](chrome-bookmark-reorganize/) as a reference example. New skills should follow the same structure; details in [skill-creator](https://skills.sh/) or the Anthropic skills guide.

## Adding a new skill

1. Create `skill-name/` with at least a `SKILL.md`
2. Add it to the table above
3. Commit & push — it becomes installable as `npx skills add ooneko/skills@skill-name`

## License

Private repository. For personal use.
