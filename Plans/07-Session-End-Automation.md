---
status: pending
depends_on: [02-Agent-Rules, 03-Git-Workflow, 04-Knowledge-Engine-CLI, 05-Notion-Sync]
---

# 阶段 07：Agent 会话结束自动化（开头问，管这一次）

## 目标

每次 Codex、Claude Code 开启一个新会话时，一开始就主动问用户"这次对话结束时要不要自动
整理保存产生的知识"——问的是"这次"，答案也只管"这次"，不是替上一次或下一次对话做决定。
完整设计已经写在 Plan.md 的「Agent 会话结束自动化」一节，本阶段负责落地实现。

依赖 02（复用 `KNOWLEDGE-RULES.md`/`DEVELOPMENT-RULES.md` 规则）、03（遵循 Git 安全规则）、
04（调用 `kb validate`/`kb curate`）、05（调用 `kb sync notion`）。不依赖阶段 06（语义检索 +
MCP Server）——SessionEnd 端的整理调用直接用 `kb` CLI + 一次无交互 Agent 调用，不强制经过
MCP。

## 范围（做什么）

- [ ] 写共享逻辑，**SessionStart 端**：通过 Hook 往新会话注入指令，让 Agent 在第一条回复
      里主动问用户"这次对话结束时要不要自动整理保存知识"，把回答（是/否）连同这次会话的
      session id 写入 `.knowledge/session-consent/<session_id>.json`。只管当前这一次
      会话，不涉及之前或之后的会话。
- [ ] 在 `90-Agent/KNOWLEDGE-RULES.md` 补一句：只有当前会话的 consent 是"是"，Agent 才
      在过程中主动把长期价值内容写成 Candidate；是"否"则这次对话不主动写（用户仍可以
      随时手动 `kb capture`）。
- [ ] 写共享逻辑，**SessionEnd 端**：读取 `.knowledge/session-consent/<session_id>.json`。
      没有记录或是"否" → 直接清理退出，不动 Vault。是"是"且这次会话确实新增了 Candidate
      （廉价检查 `00-Inbox/Agent-Candidates` 有没有新文件）→ 拉起一次无交互 Agent 调用
      （`claude -p` 或 `codex exec`，固定用一个），复用
      `90-Agent/KNOWLEDGE-RULES.md`/`DEVELOPMENT-RULES.md` 只处理这次新增的 Candidate
      （去重/合并/修 YAML/建 WikiLink，冲突就标记待人工审核）→ `kb validate` +
      `kb curate` 兜底 → `kb sync notion` → 有改动就 `git commit`（本地，不 push，消息
      标注是这次确认后的整理）。是"是"但没有新 Candidate → 直接清理退出。最后清掉这次
      会话的 consent 记录文件。
- [ ] Claude Code 接入：`hooks.SessionStart` + `hooks.SessionEnd`。
- [ ] Codex CLI 接入：`hooks.json` 的 `session-start` + `session-end`（具体字段名在实现时
      对照当时安装的 Codex 版本确认）。
- [ ] "大规模修改"阈值检测：复用 `90-Agent/DEVELOPMENT-RULES.md` 已有标准（改动超过 5 篇
      正式笔记、涉及目录结构调整、批量删除、改动 90-Agent 规则文档本身）——命中阈值时不
      自动 commit，留给用户自己确认。
- [ ] 端到端验证：
      - consent=是 + 会话中产生了 Candidate → 关闭会话后自动整理/校验/同步/提交，且这次
        提交可以被 `git revert` 干净撤销；
      - consent=是 + 会话中没有产生 Candidate → 关闭后不做任何改动；
      - consent=否 → 无论会话中发生什么，关闭后都不整理、不提交；
      - 两个不同 session id 的 consent 文件互不影响。

## 明确不做（Out of Scope）

- 不 push——保持「Git」一节的既有规则，push 需要用户明确授权。
- 不做正文层面的 Notion → Vault 回写——超出 Plan.md 已定的硬性边界（正文永远不被 Notion
  覆盖），只有既定的五个管理属性通过 `kb sync notion --pull` 回写，这条不变。
- 不强制这条流水线经过 Knowledge MCP Server——SessionEnd 的整理调用直接用 `kb` CLI，
  MCP Server 是给交互式 Agent 用的检索入口，不是这条自动化流水线的依赖。
- 不做"上一次对话"或"下一次对话"的追溯/预约——每次会话只对自己的开头提问和自己的结尾
  行为负责，不设计跨会话的待处理队列。

## 来源

Plan.md 章节：「Agent 会话结束自动化」。
