# Knowledge

一个本地优先、以 Markdown 为唯一真相源（Source of Truth）的自生成知识库。由 Obsidian 提供阅读与管理界面，Codex、Claude Code 等 AI Agent 作为知识的生产者与使用者协同维护，并可选地向 Notion 单向同步以便移动端阅读和轻量学习管理。

## 核心原则

- **Markdown 是唯一的 Source of Truth**，所有知识以纯文本 Markdown + YAML frontmatter 存储。
- **Obsidian** 是本人长期阅读和管理知识的主要界面。
- **Codex / Claude Code 等 Agent** 既读取知识，也在完成任务后提炼、补充知识——但必须经过结构化的知识提取和审核流程，不能直接把聊天记录或调试输出堆进知识库。
- **Notion 不是 Source of Truth**，只作为远程展示、移动端阅读、Database 管理和学习状态跟踪层；正文默认单向同步（Markdown → Notion），不允许无控制的双向覆盖。
- 知识必须**人类可读优先，同时对 Agent 友好**（Human-readable first, Agent-retrievable by design）。
- 尽量使用**开放格式**，不与任何特定 Agent 或软件强绑定，保证未来可以替换 Obsidian、Notion、Codex 或 Claude Code 而不丢失知识。

## 详细架构

完整的目标架构、目录设计、知识生成流程、YAML frontmatter 规范、Knowledge Curator 职责、Knowledge Engine CLI 设计、Notion 同步机制等，见 [`Plan.md`](./Plan.md)。

## 当前状态

`Plan.md` 已拆分为分阶段任务，见 [`Plans/README.md`](./Plans/README.md)。阶段 01～05 均已完成
（Vault 骨架、`90-Agent/` 规则、Git 工作流、`kb` CLI、Notion 单向同步）；阶段 06（远期扩展：
Embedding / Vector DB / MCP Server）明确保留为路线图，暂不执行。

Notion 同步已经在真实工作区跑通：Database 建在「Knowledge知识库」页面下，设计细节见
[`.knowledge/NOTION-SYNC.md`](./.knowledge/NOTION-SYNC.md)。目前 Vault 里还没有任何笔记把
`sync.notion` 设为 `true`，所以 `kb sync notion` 现在运行不会同步任何东西——需要同步的笔记
要自己在 frontmatter 里打开这个开关。

## 给人类和 Agent 的入口提示

- 想了解整体设计，先读 [`Plan.md`](./Plan.md)。
- 想知道当前进度和下一步该做什么，读 [`Plans/README.md`](./Plans/README.md)。
- 日常阅读入口是 [`01-Index/Home.md`](./01-Index/Home.md)。
- 新建知识笔记时，复制 [`90-Agent/Templates/Note-Template.md`](./90-Agent/Templates/Note-Template.md)，
  YAML frontmatter 字段说明见 [`90-Agent/Templates/Frontmatter-Spec.md`](./90-Agent/Templates/Frontmatter-Spec.md)。
- Agent 执行任务前应优先读取 [`90-Agent/INDEX.md`](./90-Agent/INDEX.md)，而不是递归加载整个 Vault（原则见 `Plan.md` 的「Agent 知识读取原则」一节）。
- 检索/整理知识优先用 `kb` CLI（`kb search` / `kb read` / `kb related` / `kb capture` / `kb curate` / `kb validate` / `kb status`），代码在 `/home/eric/Personal/KnowledgeEngine`，用法见该仓库的 README。
