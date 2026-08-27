---
status: done
depends_on: [01-Vault-Scaffold]
---

# 阶段 03：Git 版本控制规范

## 目标

Knowledge Vault 用 Git 做版本控制，作为知识出错时的安全网。Agent 对 Vault 做大规模修改
前后应能够对比、可以恢复，且未经明确授权不得进行危险的大规模删除。

## 范围（做什么）

- [x] 为 Vault 补一份 `.gitignore`（排除 `.obsidian/workspace.json` 等本地状态文件、
      `.fw-context/` 这类与知识内容无关的工具本地缓存）。额外把已经被历史误跟踪的
      `.obsidian/workspace.json`、`.fw-context/*` 用 `git rm --cached` 从索引移除（工作区
      文件保留，只是不再纳入版本控制）。
- [x] 在 `90-Agent/DEVELOPMENT-RULES.md`（阶段 02 产出）中补充一节 Git 工作流规则：
  - 大规模修改前后必须 `git status` / `git diff` 核对变更范围
  - 出错时可通过 Git 历史恢复
  - 禁止未经明确授权的危险大规模删除（`git clean -f`、批量 `rm`、`git reset --hard` 等）
  - 只在用户明确要求时才 `git commit`
- [x] 明确「大规模修改」的判断标准：一次改动涉及 5 篇以上正式笔记、涉及目录结构调整、
      涉及批量删除、或涉及修改 `90-Agent/` 规则文档本身；超过阈值时 Agent 应先展示计划
      变更范围，等待人工确认。

## 明确不做（Out of Scope）

- 不设置 Git hooks 或 CI 自动化校验 —— 如需要，属于阶段 04 CLI 里 `kb validate` 之后再考虑
  是否接入 pre-commit。
- 不处理 Notion 同步相关的 Git 状态（例如 `notion-map.json` 是否入库）—— 属于阶段 05。

## 来源

Plan.md 章节：「Git」。
