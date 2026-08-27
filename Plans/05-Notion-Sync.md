---
status: done
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

- [x] 建立核心 Knowledge Database，字段对齐：`Name / KB ID / Domain / Subdomain / Type /
      Status / Tags / Source / Created / Updated / Confidence / Local Path / Sync Status /
      Review Status`（外加 `Favorite / Priority / Rating / Next Review Date` 四个 Notion→
      Vault 回写属性，见「同步机制」）。已在 Notion 里真实建好，位于顶层页面
      「Knowledge知识库」（用户手动创建并分享给 `KnowledgeAPI` 集成）下，
      `database_id` 记录在 `.knowledge/notion-config.json`。
- [x] 产出一份 YAML frontmatter ↔ Notion Property 的映射表：见
      [`.knowledge/NOTION-SYNC.md`](../.knowledge/NOTION-SYNC.md)（权威设计文档，本文件不
      重复贴表格）。

### 同步机制

- [x] 设计 `.knowledge/notion-map.json` 结构：`kb_id → { notion_page_id, content_hash }`
      （见 NOTION-SYNC.md）。
- [x] 实现增量同步流程，接入 `kb sync notion`（KnowledgeEngine 仓库
      `kb/notion_sync.py` + `kb/notion_client.py` + `kb/notion_blocks.py`）：
      扫描 `sync.notion: true` 的正式笔记 → 算 content hash → 对比
      `notion-map.json` → 未变化跳过 / 变化则更新 / 新笔记则创建 → 回写
      `notion-map.json`。已用一篇临时测试笔记做过完整端到端验证（创建 → 改动后更新到
      同一个 Page、无重复 → 验证完归档删除，未在 Vault 或 Notion 留痕）。
- [x] 实现 `kb sync notion --dry-run`：只打印计划动作，不调用 Notion 写接口（已验证）。
- [x] 实现有限的 Notion → Vault 回写：`kb sync notion --pull`，仅限 `Review Status /
      Favorite / Priority / Rating / Next Review Date` 五个属性写回对应笔记 YAML
      frontmatter（新增字段 `review_status/favorite/priority/rating/next_review_date`，
      不在 Frontmatter-Spec.md 的必填规范内，是可选扩展字段），正文和其余字段不动。已验证。

### Obsidian 特殊语法处理

- [x] `[[WikiLink]]` → 查 `kb_id` → 查 `notion_page_id` → 转换成 Notion 页面链接；查不到
      （目标笔记还没同步过）就保留纯文本 `[[标题]]`，不报错——已验证（测试笔记里链接到
      一个不存在的笔记，同步后在 Notion 里正确保留为纯文本）。
- [x] 数学公式保持 LaTeX：行内 `$...$` 转成 Notion `equation` 富文本，独立成行的 `$$...$$`
      转成 Notion `equation` block，均不转图片——已验证行内公式转换正确。
- [x] 图片、PDF 的同步机制：记录方案（`![[附件]]` 目前只在正文里替换成
      `[附件未同步: 文件名]` 提示文字，不做真正的文件上传），有意推迟真正实现，见
      NOTION-SYNC.md「图片与 PDF」一节的理由。

## 明确不做（Out of Scope）

- 不允许 Notion 正文自动覆盖本地 Markdown 正文 —— 这是硬性原则，任何实现都不能违反。
- 不做完全双向同步。
- 不引入 Vector Database 或语义检索 —— 属于阶段 06。

## 来源

Plan.md 章节：「Notion 定位」「Notion Database」「Notion 同步机制」「Obsidian 特殊语法」。
