---
status: pending
depends_on: [01-Vault-Scaffold, 02-Agent-Rules, 03-Git-Workflow, 04-Knowledge-Engine-CLI, 05-Notion-Sync]
---

# 阶段 06：远期扩展路线图（非近期执行范围）

## 目标

记录 Plan.md 中明确标注为"后续扩展"的能力，作为路线图保留，但**明确不在近期执行范围内**。
Plan.md 原文强调："第一阶段不要直接引入复杂 Vector Database"，先用
`Markdown + 目录 + YAML + WikiLink + ripgrep + 文件搜索` 把系统跑起来，等知识规模明显
增长之后，再考虑升级。

## 范围（远期路线图，仅记录不实施）

- [ ] Keyword Search（阶段 04 已实现的 ripgrep 检索，作为升级前的基线）
- [ ] Embedding
- [ ] Vector Database
- [ ] Reranker
- [ ] Knowledge MCP Server —— 让 Codex、Claude Code、Gemini CLI、OpenCode 等多个 Agent
      通过统一的 MCP Server 访问 Knowledge Engine，而不是各自实现协议

目标终态（远期）：

```text
Codex / Claude Code / Gemini CLI / OpenCode / 其他 Agent
       │
       ▼
Knowledge MCP Server
       │
       ▼
Knowledge Engine
       │
       ▼
Markdown Vault
       │
   ┌───┴────┐
   ▼        ▼
Obsidian   Notion
```

## 触发升级的判断标准（待观察，非立即执行）

- [ ] 何时判定为"知识规模明显增长"：建议以 `kb status`（阶段 04）汇总的笔记总数、或纯关键词
      检索开始出现明显召回不足/结果太多难以筛选为信号，而不是设一个固定数字提前规划。

## 明确不做（Out of Scope，当前阶段）

- 不在阶段 01–05 完成前启动本阶段任何一项。
- 不因为"未来可能需要"而提前在 CLI 或 Vault 结构中为 Vector DB / MCP 预留接口——遵循
  Plan.md「优先采用简单、透明、可维护的方案，再逐步增加 MCP、RAG、Embedding 等复杂能力」。

## 来源

Plan.md 章节：「后续扩展」。
