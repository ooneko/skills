# 卡片标题风格

看板是给团队拉通用的,标题只讲"交付什么"——用户或系统能看到的行为变化,不讲代码怎么实现。

## 改写规则

从 iCafe 拉到的卡标题经常混着三层信息:功能名 + 技术实现 + 英文类名。写入看板时只留第一层。

| iCafe 原标题 | 看板标题 |
|---|---|
| `生成设计说明/自动保存/文件位置改（autosave BOS+元数据）` | 生成设计说明 + 自动保存到云存储 + 文件位置调整 |
| `只读态+viewerCapabilities 渲染+双击拦截` | 只读预览态:正确渲染内容 + 拦截双击编辑 |
| `ProgressBoard(role) 统一组件+两看板（看板抽象+聚合接口）` | 统一进度看板组件(项目看板 + 审核看板共用) |
| `首页创作框+分类 Tab+项目位置选择器（CreationComposer+项目树）` | 首页创作区:选项目位置 + 分类切换 + 发起创作 |
| `KB 链接粘贴→卡片+权限/授权（KnowledgeBaseProvider）` | 粘贴知识库链接生成卡片 + 权限校验 |

## 要清掉的

- 英文组件/类名:`CreationComposer`、`PlatformBridge`、`KnowledgeBaseProvider`、`ProgressBoard`
- 技术协议/缩写:`SSE`→场景名(如"预览推送")、`CNAP`→"灰度发布"、`BOS`→"云存储"
- 实现细节:`autosave`、`adapters`、`guard`、`pending prompt`、`--input-format stream-json`

## 要保留的

- 负责人 `#username`
- 卡号 `[#编号](链接)`
- 完成状态 `[x]` / `[ ]`
- 域标签,统一用加粗 + 全角分隔:`**创作**｜`、`**审核**｜`、`**基建**｜`

## 例外

约定俗成、比中文更清晰的技术术语可以保留:`API`、`CR`、`CI`、`P95`、`SSE`(当用户明确要保留时)、产品名(`tldraw`、`Figma`)。

拿不准某个术语该不该留时,问用户。
