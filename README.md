# skills

Personal collection of agent skills, compatible with any agent runtime that supports the open skills format — [Claude Code](https://claude.com/claude-code), [Codex](https://github.com/openai/codex), ZCode, and others.

Each subdirectory is a self-contained skill with a `SKILL.md` entry point.

## Skills

| Skill | What it does |
|-------|--------------|
| [chrome-bookmark-reorganize](chrome-bookmark-reorganize/) | Reorganize Chrome bookmarks on macOS — handles the local `Bookmarks` file, Google sync conflicts, and Chromium checksum recomputation. |
| [to-kanban](to-kanban/) | 把一份需求（PRD / 需求文档 / 一句口述）拆成一张可直接执行的 Obsidian kanban-plugin 看板——列管时间窗口、标签管功能模块、勾管状态，并预留 icafe 卡号回填接口。 |
| [voice-memo-manager](voice-memo-manager/) | 列出、转写、摘要、删除通过 iCloud 同步的 Apple 语音备忘录——封装了元数据提取、转写脚本与删除流程。 |
| [primary-contradiction](primary-contradiction/) | 从一段讨论或一件事里识别出主要矛盾与次要矛盾，锚定起决定作用的那处对立，并指出先动哪里。 |
| [publish-skill](publish-skill/) | 把本地 Skill 发布到 GitHub 仓库 ooneko/skills——复制目录、更新 README 表格、提交并推送。 |
| [review-until-clean](review-until-clean/) | 一个驱动代码库走向零 review 问题的紧循环：独立派发 reviewer，亲自逐一核实每条发现，只修必须修的，循环直到干净。 |
| [humanize-chinese-until-clean](humanize-chinese-until-clean/) | 通过 review→修改 循环把一个中文文件里的翻译腔改到零：每轮派 reviewer 挑机翻味，自己核实每条发现，改掉真问题，连续两轮零新发现即停。 |

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
