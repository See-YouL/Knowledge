---
status: pending
depends_on: [01-Vault-Scaffold]
---

# 阶段 03：Git 版本控制规范

## 目标

Knowledge Vault 用 Git 做版本控制，作为知识出错时的安全网。Agent 对 Vault 做大规模修改
前后应能够对比、可以恢复，且未经明确授权不得进行危险的大规模删除。

## 范围（做什么）

- [ ] 为 Vault 补一份 `.gitignore`（排除 `.obsidian/workspace*.json` 等本地状态文件、
      未来 `.knowledge/` 下不应入库的缓存类文件，具体清单在实施时按当时目录内容确定）。
- [ ] 在 `90-Agent/DEVELOPMENT-RULES.md`（阶段 02 产出）中补充一节 Git 工作流规则：
  - 大规模修改前后必须 `git status` / `git diff` 核对变更范围
  - 出错时可通过 Git 历史恢复
  - 禁止未经明确授权的危险大规模删除（`git clean -f`、批量 `rm`、`git reset --hard` 等）
- [ ] 明确「大规模修改」的判断标准（例如：一次改动涉及 N 篇笔记以上，或涉及目录结构调整），
      超过阈值时 Agent 应先展示计划变更范围，等待人工确认。

## 明确不做（Out of Scope）

- 不设置 Git hooks 或 CI 自动化校验 —— 如需要，属于阶段 04 CLI 里 `kb validate` 之后再考虑
  是否接入 pre-commit。
- 不处理 Notion 同步相关的 Git 状态（例如 `notion-map.json` 是否入库）—— 属于阶段 05。

## 来源

Plan.md 章节：「Git」。
