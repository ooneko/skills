---
name: push-to-icode-review
description: 一键完成「创建 iCafe 卡片 → git commit 绑卡 → push iCode 评审 → 追踪 iPipe 构建结果」全流程。手动触发，通过 `/$push-to-icode-review` 调用。
---

# push-to-icode-review

把代码改动变成一个可评审的 iCode Change，并追踪 iPipe 构建结果。四步：建卡 → 提交 → 推送评审 → 追踪构建。

## 前提

- 依赖 `icafe-official` skill（创建/搜索卡片）。
- 依赖 `ipipe-official` skill（追踪 iPipe 构建结果，第 4 步用到）。
- 远端是 iCode（Gerrit），分支受保护，必须走 `HEAD:refs/for/<branch>`。
- 工作区有未提交的改动（`git status` 非干净）。

## 目标分支

默认推送到 `master` 评审分支。如果用户指定了分支（如"推到 develop 分支评审"），则推送到该分支。

## 全流程

### 1. 创建卡片（委托 icafe-official）

按 `icafe-official` 的零交互版 Git 绑定工作流执行：

1. 收集本地信号：`git log --oneline -10`、`git rev-parse --abbrev-ref HEAD`、`git remote get-url origin`、`git diff --stat`
2. `icafe-cli card smart-find` 搜索已有卡片
3. 有明确相关的卡片 → 直接复用；都没有 → `icafe-cli card create` 新建
4. 得到 `{space}-{sequence}`（如 `baidu-ACG-CXD-CXD-AI-86`）和类型（如 `Story`）

### 2. 提交

```bash
git add <相关文件>
git commit -m "{space}-{sequence} [{type}] {简述}"
```

- commit message 第一行严格遵守 `{space}-{sequence} [{type}] {title}` 格式。
- 不要加 `Co-authored-by` 或其他 co-author trailer（仓库规范禁止）。
- 只 `git add` 本次改动相关的文件，不要 `git add .`。

### 3. 推送评审

iCode 分支禁止直接 push，必须用 Gerrit 评审推送。目标分支默认 `master`，用户指定了则用指定分支：

```bash
# ❌ 禁止：直接 push 会被 HCK_F603 拒绝
git push origin master

# ✅ 正确：推到评审分支（TARGET_BRANCH 默认 master）
git push origin HEAD:refs/for/${TARGET_BRANCH}
```

推送成功后，远端会返回形如 `CHECK_SUCC` 的状态和评审链接：

```
http://icode.baidu.com/myreview/changes/c/{repo-path}/+/{change-id}
```

把评审链接和卡片链接一起输出给用户。

### 4. 追踪 iPipe 构建结果（委托 ipipe-official）

push 成功只是第一步——iCode 收到 Change 后会触发 `ci.yml` 定义的 iPipe
流水线。输出评审链接和卡片链接后，**主动询问用户是否要观测 iPipe 构建结果**：

> 已推送到 iCode 评审，是否要追踪 iPipe 构建结果？

- 用户**确认**（是 / 要 / 观测 …）→ 继续下方追踪流程。
- 用户**拒绝**（否 / 不用 / 跳过 …）→ 流程到此结束，不启动观测。

仅当用户确认后，依赖 `ipipe-official` skill（提供 `ipipe-cli`）按以下步骤走。

#### 准备

1. 取本次提交的**完整 commit SHA**（40 位，用 `git rev-parse HEAD`）。
   注意：`--revision` 只认完整 SHA，短 SHA 和 Change-Id 都会返回空。
2. 从 `git remote get-url origin` 提取 module 名（去掉 `ssh://...:8235/` 前缀）。
   如 `ssh://huabinhong@icode.baidu.com:8235/baidu/ACG-CXD/CXD-AI` → `baidu/ACG-CXD/CXD-AI`。
3. 确认 `ipipe-cli login status` 已登录；`auth_failed` 则见 `ipipe-official` 登录流程。

#### 后台轮询收集结果

一次 push 会触发多条流水线（评审阶段至少 ChangePipeline）。用后台脚本轮询，
避免长时间阻塞。**等待约 1–2 分钟让流水线启动**，然后查询：

```bash
# 从 commit SHA 反查触发的所有流水线构建
ipipe-cli build commit --revision <完整commit-SHA> --module <模块名>
```

构建状态看返回 JSON 每条记录的 `status` 字段：

| status | 含义 | 处理 |
|--------|------|------|
| `SUCCEEDING` | 成功 | 流程结束，输出结果 |
| `FAILING` | 失败 | 拉日志定位，见下方 |
| `RUNNING` | 进行中 | 稍等后重新查询 |
| `WAITTING` | 等待中 | 稍等后重新查询 |

若仍有 `RUNNING`/`WAITTING`，每 30–60 秒重查一次，直到全部终态。
**建议用后台脚本轮询**（`run_in_background`），结束时统一汇报。

#### 判定与收尾

- **全部 `SUCCEEDING`** → 构建通过，流程结束。把每条流水线的状态和构建页链接
  （返回 JSON 的 `buildUrl`）输出给用户。
- **有 `FAILING`** → 拉日志定位报错，见下方「失败构建排障」。

#### 失败构建排障

1. 从失败记录的 `stageBuilds[]` 里找到 `status: "FAIL"` 的阶段，取 `id`（stageBuildId）。
2. 查日志链接：

   ```bash
   ipipe-cli build stage --stage-build-id <stageBuildId>
   ```

   返回 JSON 的 `realJobBuilds[].logs[]` 里有日志 URL，通常两条：
   - `key: "summary"` → 摘要日志
   - `key: "build"` → 详细日志（报错根因在这里）

3. curl 拉取详细日志（logonline 无需认证，本地直接拉）：

   ```bash
   curl -sS "https://logonline.baidu-int.com/<build-日志UUID>" | tail -80
   ```

4. 定位报错根因（日志尾部的 `Result: FAIL` + `return code: 1` 区域往上翻），
   评估是否能在本次改动范围内修复。能修则修；超出本次改动边界的缺陷，记为
   follow-up，不要扩大本次 PR 范围。

## 失败处理

| 现象 | 原因 | 解决 |
|------|------|------|
| `HCK_F603 Pushing directly to ... is forbidden` | 用了 `git push` 而非 `HEAD:refs/for/<branch>` | 改用 `git push origin HEAD:refs/for/<目标分支>` |
| `CHECK_FAIL`（提交信息校验） | commit message 没绑卡或格式不对 | 确认第一行是 `{space}-{sequence} [{type}] {title}` |
| `auth_failed` (icafe-cli) | UUAP token 失效 | 见 `icafe-official` 登录流程 |
