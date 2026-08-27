---
status: pending
depends_on: [01-Vault-Scaffold, 02-Agent-Rules]
---

# 阶段 04：Knowledge Engine CLI 骨架

## 目标

提供一个与 Vault 数据分离的 `kb` 命令行工具，让人和 Agent 通过统一接口访问知识系统，
而不是各自实现一套检索/整理逻辑。第一阶段只做本地检索（ripgrep + 文件搜索），不引入
Vector Database，也不含 Notion 同步（同步逻辑属于阶段 05，应作为 `kb sync notion` 子命令
接入同一个 CLI，但本阶段先把子命令占位、不实现内部逻辑）。

## 范围（做什么）

- [ ] 在 `/home/eric/Personal/KnowledgeEngine` 建立独立代码仓库（与 Vault 数据目录
      `/home/eric/Personal/Obsidian/Knowledge` 分开，符合「程序与数据必须分开」原则）。
- [ ] 确定实现语言与最小依赖（例如 Python 或 Node，优先选择容易在 WSL2 中运行、易于调用
      `ripgrep` 的方案）。
- [ ] 实现命令骨架，先占位、逐步补实现：
  - [ ] `kb search "<query>"` —— 基于 ripgrep 在 Vault 内做关键词检索
  - [ ] `kb read "<title或kb_id>"` —— 定位并输出对应笔记内容
  - [ ] `kb related "<title或kb_id>"` —— 基于 WikiLink 反向查找相关笔记
  - [ ] `kb capture` —— 将一段结构化 Candidate 写入 `00-Inbox/Agent-Candidates`
  - [ ] `kb curate` —— 辅助 Knowledge Curator 流程（先实现检测重复标题/Broken Links 等
        只读检查，真正的合并/去重仍由人工或 Agent 判断）
  - [ ] `kb validate` —— 校验 YAML frontmatter 是否符合阶段 01 定稿的规范
  - [ ] `kb status` —— 汇总 Vault 现状（笔记数量、按 type/status/confidence 分布、
        Inbox 待处理数量等）
  - [ ] `kb sync notion` / `kb sync notion --dry-run` —— 子命令占位，实现留给阶段 05
- [ ] 编写最小 README，说明如何在 WSL2 中安装和调用 `kb`。

## 明确不做（Out of Scope）

- 不实现 Embedding / Vector Database / Reranker —— 属于阶段 06，明确是「知识规模明显增长
  以后」才升级的能力。
- 不实现 `kb sync notion` 的真实同步逻辑 —— 属于阶段 05。
- 不实现 Knowledge MCP Server —— 属于阶段 06。

## 来源

Plan.md 章节：「Knowledge Engine」。
