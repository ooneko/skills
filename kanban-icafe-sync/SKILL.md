---
name: kanban-icafe-sync
description: 从 iCafe 拉取卡片状态,更新 Obsidian Kanban 看板,上传知识库。
disable-model-invocation: true
---

# iCafe → 看板 → 知识库

拉 iCafe 卡片 → 组织 → 写看板 → 上传知识库。走 `icafe-official` skill,不手写 HTTP。

## 前置确认

缺一就问:

- **看板路径** —— `.md` 绝对路径。不确定用 `ls` 找。
- **iCafe 空间** —— prefixCode(如 `baidu-ACG-CXD-CXD-AI`)。`icafe-cli space latest` 推断。
- **范围** —— "上周完成" → 已完成卡按完成时间过滤;"本周需求" → 去掉 Bug 只留 Story;"按主题" → 按业务域归并。

## 第一步:拉取

```bash
~/.icafe-cli/bin/icafe-cli card query \
  --space <空间> \
  --iql "流程状态 = 已完成 AND 创建时间 > \"<起始日期>\"" \
  --brief --max-records 50
```

**完成时间以 `status-changes` 为准,不看 `lastModifiedTime`** —— 后者被评论/字段编辑刷新,不能判断完成时机。对每张卡补查精确时间:

```bash
~/.icafe-cli/bin/icafe-cli card status-changes \
  --space <空间> --sequences <逗号分隔编号>
```

取每个 sequence 最后一条 `targetStatusName = "已完成"` 的 `operationTime`。

**完成:** 区间内流转到「已完成」的卡全部拿到,无 `lastModifiedTime` 假报。

## 第二步:组织

按用户要的方式:

- **按天** —— `status-changes` 时间分。适合日报/周报。
- **按主题** —— 按业务域归并(平台迁移、登录门禁、首页、基建…),不预设固定分类。
- **筛选** —— 去掉 Bug、只要某负责人/类型。按用户说的筛。

**完成:** 每张卡归到正确分组,筛选条件已执行。

## 第三步:写入看板

先 `Read` 看板文件看清栏结构。铁律:

- **不碰 frontmatter(`---`…`---`)和文末 `%% kanban:settings` 块** —— 改坏看板就废。
- **卡片格式** —— `- [ ]`/`- [x]` 开头,状态跟 iCafe 一致。
- **保留卡号和负责人** —— `#负责人 [#编号](链接)`。
- **标题去技术黑话** —— 讲"交付什么",不讲组件类名。规则见 [references/card-title-style.md](references/card-title-style.md)。
- **新建栏** —— `## 栏名` + 空行 + 卡片列表。日期区间用 `M.DD–M.DD`。
- **更新栏** —— Edit 精确替换,`old_string` 必须唯一。Edit 报 "File has been modified" → 重新 Read 再改,别硬写。

**完成:** frontmatter 和 Kanban 块完好,卡片格式正确,卡号和链接不丢。

## 第四步:上传知识库

用 `ku-doc-manage` 的 `create-doc --md-file`。默认知识库 `8Xh2-LmLim`(设计领航平台),用户另行指定时从 URL path3 提取 repo-id。

**上传前剥掉 Obsidian 专属结构:**

- 删 frontmatter(`---` + `kanban-plugin: board` + `---`)
- 删文末 `%% kanban:settings` 块
- 不加"从 Obsidian 同步"之类的导入说明行
- emoji 原样保留
- `#` 只用于文档标题,各栏 `##`

写入临时文件上传,原 Obsidian 文件不动:

```bash
export SANDBOX_USERNAME=huabinhong
~/.agents/skills/ku-doc-manage/bin/ku create-doc \
  --repo-id 8Xh2-LmLim \
  --md-file "/tmp/进度看板.md" \
  --title "进度看板"
```

**完成:** 文档创建成功(returnCode=200),无 frontmatter/kanban settings 残留,栏结构和卡片完整。

## 避坑

- `status-changes` 的 opType 有 `UPDATE`/`API_UPDATE`/`COMMITGIT` 三种,都算有效完成流转。
- IQL 日期不支持 `>=`,用 `> "前一天日期"` 捕获当天。
- 卡片可能多次流转(已完成 → reopened → 再完成),取最后一次到「已完成」的时间。
- 多人空间 query 返回的不只是当前用户的卡。用户没说"我的"就不按负责人过滤。
- 跨周卡片以完成时间归属,不以创建时间归属。
