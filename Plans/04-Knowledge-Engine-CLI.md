---
status: done
depends_on: [01-Vault-Scaffold, 02-Agent-Rules]
---

# 阶段 04：Knowledge Engine CLI 骨架

## 目标

提供一个与 Vault 数据分离的 `kb` 命令行工具，让人和 Agent 通过统一接口访问知识系统，
而不是各自实现一套检索/整理逻辑。第一阶段只做本地检索（ripgrep + 文件搜索），不引入
Vector Database，也不含 Notion 同步（同步逻辑属于阶段 05，应作为 `kb sync notion` 子命令
接入同一个 CLI，但本阶段先把子命令占位、不实现内部逻辑）。

## 范围（做什么）

- [x] 在 `/home/eric/Personal/KnowledgeEngine` 建立独立代码仓库（与 Vault 数据目录
      `/home/eric/Personal/Obsidian/Knowledge` 分开，符合「程序与数据必须分开」原则）。
- [x] 确定实现语言与最小依赖：Python 3（标准库 + PyYAML）。检索优先用系统里真实安装的
      `ripgrep`，检测不到时自动退回纯 Python 扫描——因为发现当前 Shell 里的 `rg` 只是
      Claude Code 注入的函数，子进程调不到真正的可执行文件，系统实际没有装 ripgrep 二进制
      （`apt list --installed` 确认过），所以不能把它当硬依赖。
- [x] 实现命令骨架，均已跑通并对着真实 Vault 做过端到端验证（含清理测试数据）：
  - [x] `kb search "<query>"` —— 关键词检索（ripgrep 优先，纯 Python 兜底）
  - [x] `kb read "<title或kb_id>"` —— 定位并输出对应笔记内容
  - [x] `kb related "<title或kb_id>"` —— 正向/反向 WikiLink 相关笔记
  - [x] `kb capture` —— 将一段结构化 Candidate 写入 `00-Inbox/Agent-Candidates`
  - [x] `kb curate` —— 只读检查：重复标题 / Broken Link / 待审核 Candidate 数量
  - [x] `kb validate` —— 校验 YAML frontmatter（必填字段、枚举取值、kb_id 格式与唯一性、
        日期格式）
  - [x] `kb status` —— 汇总 Vault 现状（笔记数量、按 type/status/confidence 分布、
        Inbox 待处理数量）
  - [x] `kb sync notion` / `kb sync notion --dry-run` —— 子命令占位，实现留给阶段 05
- [x] 编写 README，说明安装方式（`pip install -e .` 或直接 `python3 -m kb ...`）、
      `KB_VAULT_PATH` 配置、各命令用法。

补充说明：`pip install -e .` 需要 venv（`python3 -m venv` 因缺少 `python3.14-venv`
系统包而失败，未擅自 `apt install` 或 `--break-system-packages`），或走 `pipx`；
在两者都不可用的当前环境下，也可以在仓库目录直接用 `python3 -m kb ...` 运行，无需安装
（依赖只有标准库 + 已装好的 PyYAML）。README 中两种方式都有说明。

## 明确不做（Out of Scope）

- 不实现 Embedding / Vector Database / Reranker —— 属于阶段 06，明确是「知识规模明显增长
  以后」才升级的能力。
- 不实现 `kb sync notion` 的真实同步逻辑 —— 属于阶段 05。
- 不实现 Knowledge MCP Server —— 属于阶段 06。

## 来源

Plan.md 章节：「Knowledge Engine」。
