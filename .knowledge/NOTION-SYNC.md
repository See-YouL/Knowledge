# Notion 同步设计

对应 [`Plans/05-Notion-Sync.md`](../Plans/05-Notion-Sync.md)。这里是同步机制的权威设计文档；
`kb sync notion` 的实现（KnowledgeEngine 仓库）照此文档执行。

## 硬性原则（不可违反）

- Markdown 是唯一 Source of Truth，同步方向默认 **Markdown → Notion**。
- Notion 正文永远不会覆盖本地 Markdown 正文。
- Notion → Vault 的回写仅限于少数管理属性（见下），不涉及正文。

## Notion Database

- Database 名称：`Knowledge Database`
- 所在页面：`Knowledge知识库`（工作区顶层页面，已分享给 `KnowledgeAPI` 集成）
- `parent_page_id` / `database_id` 记录在同目录下的 [`notion-config.json`](notion-config.json)
  （非敏感信息，可入库；Token 本身绝不出现在这里，只存在本地未入库的 `.env`）。

### 字段与 YAML frontmatter 的映射表

| Notion Property | 类型 | 对应 YAML 字段 | 方向 | 说明 |
|---|---|---|---|---|
| Name | title | `title` | Markdown → Notion | |
| KB ID | rich_text | `kb_id` | Markdown → Notion | 用于查找/去重，不在 Notion 里改 |
| Domain | select | `domain` | Markdown → Notion | |
| Subdomain | rich_text | `subdomain` | Markdown → Notion | |
| Type | select | `type` | Markdown → Notion | |
| Status | select | `status` | Markdown → Notion | |
| Tags | multi_select | `tags` | Markdown → Notion | |
| Source | rich_text | `source_type` | Markdown → Notion | |
| Created | date | `created` | Markdown → Notion | |
| Updated | date | `updated` | Markdown → Notion | |
| Confidence | select | `confidence` | Markdown → Notion | |
| Local Path | rich_text | （笔记相对 Vault 的路径，非 YAML 字段） | Markdown → Notion | 方便在 Notion 里定位对应本地文件 |
| Sync Status | select | （同步工具内部状态，非 YAML 字段） | Markdown → Notion | `synced`，由 `kb sync notion` 每次同步后写入 |
| Review Status | select | 无对应 YAML 字段（Vault 内不存这个状态） | **Notion → Markdown**（写回 frontmatter，不新建字段名冲突，见下） | 学习/复习状态，Notion 是权威来源 |
| Favorite | checkbox | 同上 | **Notion → Markdown** | |
| Priority | select | 同上 | **Notion → Markdown** | |
| Rating | number | 同上 | **Notion → Markdown** | |
| Next Review Date | date | 同上 | **Notion → Markdown** | |

`Review Status` / `Favorite` / `Priority` / `Rating` / `Next Review Date` 这五个属性是
"Notion 是权威来源"的例外：`kb sync notion --pull` 会把它们写回对应笔记 YAML frontmatter
里同名（小写+下划线）的字段：`review_status` / `favorite` / `priority` / `rating` /
`next_review_date`。这些字段**不在** [`90-Agent/Templates/Frontmatter-Spec.md`](../90-Agent/Templates/Frontmatter-Spec.md)
的必填规范里，是可选的学习管理扩展字段，`kb validate` 不会因为它们缺失而报错。

## `.knowledge/notion-map.json`

```json
{
  "kb-3f9a21c4": {
    "notion_page_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
    "content_hash": "sha256:xxxx..."
  }
}
```

- key 是 `kb_id`。
- `content_hash` 是笔记整个文件内容（frontmatter + 正文）的 SHA-256，用于判断是否需要更新。
- 由 `kb sync notion` 自动维护，不需要手工编辑。

## 同步算法（`kb sync notion`）

```text
扫描 10~80 目录下 sync.notion: true 的正式笔记
  ↓
对每篇笔记：
  读取 kb_id，计算 content_hash
  ↓
  查 notion-map.json：
    ├── 无记录 → 创建 Notion Page（写入全部属性 + 正文转换出的 blocks），记录 notion_page_id + content_hash
    ├── 有记录，hash 相同 → 跳过
    └── 有记录，hash 不同 → 更新已有 Notion Page 的属性和正文 blocks，更新 content_hash
  ↓
更新 notion-map.json
```

`--dry-run`：只打印将要创建/更新/跳过的笔记列表，不调用 Notion 写接口。

`--pull`：反方向，只读取 Notion 里已同步页面的 `Review Status/Favorite/Priority/Rating/
Next Review Date` 五个属性，写回对应笔记 frontmatter，正文和其余字段不动。

## WikiLink 处理

正文里的 `[[标题]]` 在转换成 Notion blocks 时：

1. 在当前 Vault 内按标题找到目标笔记，取其 `kb_id`。
2. 查 `notion-map.json` 看目标笔记是否已经同步过、有没有 `notion_page_id`。
3. 有 → 转换成指向该 Notion 页面的链接；没有（目标还没同步，或找不到）→ 保留为纯文本
   `[[标题]]`，不报错。

这意味着 WikiLink 之间的互相链接可能需要跑两次 `kb sync notion` 才能全部生效
（先把两边的页面都创建出来，第二次跑才能互相解析到对方的 `notion_page_id`）——这是
增量同步的自然结果，不是 bug。

## 数学公式

正文里的 `$...$`（行内）和 `$$...$$`（独立成行的公式块）转换成 Notion 的 `equation`
富文本/block 类型，保持 LaTeX 源码，不截图、不转图片。

## 图片与 PDF（本阶段推迟）

`![[附件]]` 目前只会在 Notion 正文里保留一行文字提示（`[附件未同步: 文件名]`），不做真正的
文件上传。原因：Notion 文件上传是独立的 API（先上传拿 file id，再引用），涉及更多边界情况
（大小限制、MIME 类型等），Plan.md 本身也把这部分列为"后期通过文件上传机制解决"。等正文/
属性同步稳定运行一段时间后再补。
