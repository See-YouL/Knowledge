# Agent-Candidates

Agent（Codex、Claude Code 等）完成任务后提炼出的结构化知识 Candidate 存放于此，
尚未经过 Knowledge Curator 审核确认，不视为正式知识。

对应 [`Plan.md`](../../Plan.md) 「知识生成流程」一节：

```text
完成任务 → 判断是否有长期可复用知识 → 提炼 → 生成结构化 Candidate → 本目录
   → Knowledge Curator 审核 → 更新 / 合并 / 新建 / 标记冲突 / 丢弃 → 正式知识
```

具体的 Curator 审核规则见 `90-Agent/`（[`Plans/02-Agent-Rules.md`](../../Plans/02-Agent-Rules.md) 中产出）。
