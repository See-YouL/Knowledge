---
status: pending
depends_on: [01-Vault-Scaffold, 02-Agent-Rules, 04-Knowledge-Engine-CLI]
---

# 阶段 05：Notion 单向同步

## 目标

让 Notion 承担远程展示、移动端阅读、Database 管理和学习状态跟踪的角色，但绝不成为
Source of Truth。默认同步方向是 Markdown → Notion；只允许少量管理属性做 Notion → Vault
的回写，正文永远不允许 Notion 覆盖本地 Markdown。

**需要 Notion API Token 等凭证，属于用户需要主动提供输入的阶段**——实施前应与用户确认
Token 获取方式，凭证本身遵循 Plan.md「禁止简单保存密码/API Token」的原则，绝不写入 Vault
或代码仓库，只放在环境变量或本地未入库的配置文件中。

## 范围（做什么）

### Notion Database 设计

- [ ] 建立核心 Knowledge Database，字段对齐：`Name / KB ID / Domain / Subdomain / Type /
      Status / Tags / Source / Created / Updated / Confidence / Local Path / Sync Status /
      Review Status`。
- [ ] 产出一份 YAML frontmatter ↔ Notion Property 的映射表（文档形式，落在本阶段文件或
      `.knowledge/` 下的一份说明文件中）。

### 同步机制

- [ ] 设计 `.knowledge/notion-map.json` 结构：`kb_id → { notion_page_id, content_hash }`。
- [ ] 实现增量同步流程（对接阶段 04 的 `kb sync notion`）：
  1. 扫描 `sync.notion: true` 的 Markdown
  2. 读取 `kb_id`
  3. 计算 content hash
  4. 与 `notion-map.json` 中记录的上次 hash 比较
  5. 未变化则跳过；变化则更新对应 Notion Page；新知识则创建 Page
  6. 更新 `notion-map.json`
- [ ] 实现 `kb sync notion --dry-run`：只打印将要执行的创建/更新列表，不实际调用 Notion API。
- [ ] 设计有限的 Notion → Vault 回写：仅限 `Review Status / Favorite / Priority / Rating /
      Next Review Date` 等管理属性，写回对应笔记的 YAML frontmatter，正文不动。

### Obsidian 特殊语法处理

- [ ] `[[WikiLink]]` → 查 `kb_id` → 查 `notion_page_id` → 转换成 Notion 页面链接。
- [ ] 数学公式保持 LaTeX，不转换成图片。
- [ ] 图片、PDF 的同步机制（先记录方案，具体文件上传实现可以是本阶段的子任务，视实际需要
      决定是否推迟）。

## 明确不做（Out of Scope）

- 不允许 Notion 正文自动覆盖本地 Markdown 正文 —— 这是硬性原则，任何实现都不能违反。
- 不做完全双向同步。
- 不引入 Vector Database 或语义检索 —— 属于阶段 06。

## 来源

Plan.md 章节：「Notion 定位」「Notion Database」「Notion 同步机制」「Obsidian 特殊语法」。
