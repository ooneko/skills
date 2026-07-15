---
name: publish-skill
description: 把一个本地 Skill 发布到 GitHub 仓库 ooneko/skills——复制目录、更新 README 表格、提交并推送。
disable-model-invocation: true
---

# 发布 Skill

把本地写好的 Skill 推到 GitHub 仓库 `ooneko/skills`。仓库根目录是 `/Users/huabinhong/Code/bigcat/skills`（下称**仓库根**），remote 是 `git@github.com:ooneko/skills.git`。

## 0. 确认目标 skill

用户说"把 XXSkill 发布到我的 github"——把 XX 解析成一个 skill 名，并在以下来源里找到它：

- `/Users/huabinhong/.agents/skills/<name>/`
- `/Users/huabinhong/.zcode/skills/<name>/`

**完成标准**：定位到一个存在的 skill 目录，且里面有 `SKILL.md`。找不到就停下问用户。

## 1. 复制到仓库

把整个 skill 目录原样复制到**仓库根 / `<name>` /**。已存在则覆盖（用户在重新发布）。

复制后**清理 git 残留**：如果源目录或子目录里带了 `.git`，删掉它，否则会把子仓库嵌进来。

**完成标准**：`<仓库根>/<name>/SKILL.md` 存在，且目录里没有 `.git`。

## 2. 更新 README

README 表格维护着 skill 清单，每行一个。新 skill 加一行；重新发布就更新对应行。格式：

```
| [skill-name](skill-name/) | 一句话说清它干什么 |
```

描述从 skill 的 `description` 字段提炼，控制在表格里一行可读。中英文不限，贴合该 skill 自身的语言。

**完成标准**：README 表格里有这一行，格式与现有行一致；没有重复行。

## 3. 提交并推送

在**仓库根**执行：

```
git add <name>/ README.md
git commit -m "<一句话，中文>"
git push
```

提交信息点明这次加了或改了哪个 skill。push 前先 `git pull --rebase` 以防远端有新提交。

**完成标准**：`git push` 成功，远端 `ooneko/skills` 能看到这个 skill。

## 备注

- 这是私有仓库。push 后 skill 可通过 `npx skills add ooneko/skills@<name>` 安装（见仓库 README）。
- 仓库约定 `SKILL.md` 尽量 < 100 行、skill 自包含。若要发布的 skill 体积偏大，复制时照原样即可，但可在最后提醒用户一句。
- 若 push 因权限或网络失败，把 `gh` / `git` 的原始报错贴出来，别自己改写。
