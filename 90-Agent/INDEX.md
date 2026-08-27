# Agent 入口索引

Codex、Claude Code 等 Agent 开始任务前，应先读这个文件，再决定去哪里找知识——
**不要递归加载整个 Vault**。

## 第一步：识别领域

| 目录 | 用途 |
|---|---|
| `10-Embedded/` | 嵌入式开发：MCU、外设驱动、RTOS、中断/DMA、调试与烧录工具链 |
| `20-Motor-Control/` | 电机控制：PMSM、无感（sensorless）控制、观测器、驱动电路 |
| `30-Control-Theory/` | 控制理论：PID、状态空间、观测器、稳定性分析 |
| `40-MATLAB-Simulink/` | MATLAB / Simulink 建模、仿真、代码生成 |
| `50-Toolchains/` | 编译器、调试器、构建系统、IDE 配置 |
| `60-System/` | 操作系统、Linux、WSL2、网络、Shell 环境 |
| `70-Papers/` | 论文阅读笔记（`type: paper`） |
| `80-Projects/` | 具体项目相关知识与决策记录 |

领域索引页在 [`01-Index/`](../01-Index/Home.md)（随内容增长逐步建立）。

## 第二步：了解规则

- 如何把任务成果提炼成知识、什么内容禁止直接保存、按什么优先级读取知识——见
  [`KNOWLEDGE-RULES.md`](KNOWLEDGE-RULES.md)。
- 知识类型（`type`）怎么选、`domain`/`subdomain` 怎么填——见 [`TAXONOMY.md`](TAXONOMY.md)。
- 作为 Knowledge Curator 时如何审核/去重/合并 Candidate，如何维护 Canonical Note——见
  [`DEVELOPMENT-RULES.md`](DEVELOPMENT-RULES.md)。

## 第三步：新建或修改笔记

- 新建笔记：复制 [`Templates/Note-Template.md`](Templates/Note-Template.md)。
- YAML frontmatter 字段含义、取值范围、`kb_id` 生成方式：
  [`Templates/Frontmatter-Spec.md`](Templates/Frontmatter-Spec.md)。

## 第四步：搜索已有知识

当前阶段（见 [`Plans/README.md`](../Plans/README.md)）尚未提供 `kb search` CLI（阶段 04），
先用 `ripgrep` 在 Vault 内检索关键词，例如：

```bash
rg -i "PLL" --glob '!Plans/**' --glob '!.git/**'
```

## 整体架构参考

完整设计见 [`Plan.md`](../Plan.md)；分阶段执行计划见 [`Plans/README.md`](../Plans/README.md)。
