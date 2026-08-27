# Plans 总览

本目录把 [`Plan.md`](../Plan.md) 中的架构愿景拆分成按阶段排序、可勾选执行的任务文件。

- `Plan.md` 保持不动，继续作为唯一的顶层架构参考（Source of Truth for 架构决策）。
- 本目录下的文件不复制 Plan.md 的大段原文，只做提炼、结构化、拆分依赖顺序。
- **先读这个文件，判断当前处于哪个阶段，再只打开对应的阶段文件** —— 不要一次性加载整个
  `Plans/` 目录。这与 Plan.md「Agent 知识读取原则」一节的思路一致：只加载当前任务需要的
  少量文件。

## 阶段总览

| 阶段 | 文件 | 一句话说明 | 依赖 | Status |
|---|---|---|---|---|
| 01 | [01-Vault-Scaffold.md](01-Vault-Scaffold.md) | Vault 目录骨架、YAML frontmatter、笔记正文模板、知识类型体系 | 无 | done |
| 02 | [02-Agent-Rules.md](02-Agent-Rules.md) | 90-Agent 规则文档、知识生成流程、Knowledge Curator 职责、Agent 读取原则 | 01 | done |
| 03 | [03-Git-Workflow.md](03-Git-Workflow.md) | Vault 的 Git 版本控制规范 | 01 | done |
| 04 | [04-Knowledge-Engine-CLI.md](04-Knowledge-Engine-CLI.md) | 独立的 `kb` CLI 骨架（本地检索，不含 Notion） | 01, 02 | done |
| 05 | [05-Notion-Sync.md](05-Notion-Sync.md) | Notion Database 设计 + 单向同步机制 | 01, 02, 04 | done |
| 06 | [06-Future-Extensions.md](06-Future-Extensions.md) | 本地语义检索（Embedding + Rerank）+ Knowledge MCP Server（用户主动要求提前做，非规模驱动） | 01–05 | done |
| 07 | [07-Session-End-Automation.md](07-Session-End-Automation.md) | 会话一开始就问"这次要不要自动保存知识"，只管这一次，结束时按当次回答执行 | 02, 03, 04, 05 | pending |

## Status 含义

- `pending`：尚未开始。
- `in-progress`：正在进行，阶段文件内的勾选清单部分完成。
- `done`：交付物全部完成并已核对，可以作为后续阶段的地基。

推进某个阶段后，请更新该阶段文件的 frontmatter `status` 字段，并同步更新本表。

## 阶段顺序的理由

01 是地基：没有目录结构和 YAML 规范，Agent 规则、CLI、Notion 同步都无从谈起。
02、03 是运行 Vault 所需的最小规则层，让 Agent 知道该怎么用、Git 该怎么把关。
04、05 是工具层，明确建立在 01/02 之上。
06 原本是 Plan.md 标注的「后续扩展」，按设计应该等知识规模明显增长后再做——但用户在知情
（Vault 当时 0 篇正式笔记）的情况下明确要求提前执行，所以已经完成，细节和"跳过触发条件"
的说明见 [06-Future-Extensions.md](06-Future-Extensions.md)。
07 建立在 02/03/04/05 之上（复用规则文档、Git 安全规则、`kb validate`/`kb curate`、
`kb sync notion`），不依赖 06——会话结束自动化直接调 `kb` CLI，不强制经过 MCP Server。
