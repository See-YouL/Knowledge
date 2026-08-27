---
status: done
depends_on: []
---

# 阶段 01：Vault 目录骨架与知识规范

## 目标

搭建 Knowledge Vault 的物理目录结构，并定稿知识条目的 YAML frontmatter 规范与正文模板。
这是整个系统的地基——没有稳定的目录结构和统一的元数据格式，后续的 Agent 规则、Curator、
CLI、Notion 同步都无法可靠运作。遵循 Plan.md 的原则：Markdown 是唯一的 Source of Truth，
且必须 Human-readable first, Agent-retrievable by design。

## 范围（做什么）

### 目录结构

- [x] 创建 `00-Inbox/Manual/`、`00-Inbox/Agent-Candidates/`、`00-Inbox/Unsorted/`
- [x] 创建 `01-Index/`（先放 `Home.md` 作为入口，其余索引文件随内容增长再建）
- [x] 创建 `10-Embedded/`、`20-Motor-Control/`、`30-Control-Theory/`、
      `40-MATLAB-Simulink/`、`50-Toolchains/`、`60-System/`、`70-Papers/`、`80-Projects/`
- [x] 创建 `90-Agent/`（含 `Templates/` 子目录，具体文件在阶段 02 填充）
- [x] 创建 `99-Archive/`
- [x] 创建 `.knowledge/`（机器数据：同步状态、索引、映射；先建空目录，具体文件由后续阶段
      按需产出，例如阶段 05 的 `notion-map.json`）

### YAML Frontmatter 规范

- [x] 定稿字段集合与取值范围，落地为 `90-Agent/Templates/` 下的一个 frontmatter 模板文件
      （`Frontmatter-Spec.md`）：
  - `kb_id`：永久唯一 ID，不随文件移动/改名变化，不使用文件路径作为标识
  - `title`
  - `type`：`concept | procedure | troubleshooting | decision | reference | paper | project | experiment`
  - `domain` / `subdomain`
  - `tags`
  - `status`：`candidate | verified | experimental | deprecated`
  - `source_type`
  - `created` / `updated`
  - `confidence`：`high | medium | low`
  - `sync.notion`：布尔值，是否允许同步到 Notion
- [x] 明确 `kb_id` 的生成方式（例如短随机 ID 或基于内容的 slug + 序号），并记录在模板文件中，
      供人和 Agent 一致地生成新 ID。（`kb-` + 8 位十六进制，见 `Frontmatter-Spec.md`）

### 笔记正文模板

- [x] 落地一个 Markdown 正文模板文件到 `90-Agent/Templates/`（`Note-Template.md`），结构为：
      `# 标题` → `## 问题 / 定义` → `## 核心原理` → `## 详细解释` → `## 实现或推导` →
      `## 常见问题` → `## 相关知识`（含 WikiLink 示例）→ `## 来源`

## 明确不做（Out of Scope）

- 不填充 `90-Agent/INDEX.md`、`KNOWLEDGE-RULES.md`、`TAXONOMY.md`、`DEVELOPMENT-RULES.md`
  的具体规则内容 —— 属于阶段 02。
- 不写任何实际的知识条目内容到 `10~80` 目录下 —— 目录建好即可，内容由日常使用逐步产生。
- 不涉及 Git 提交策略 —— 属于阶段 03。
- 不实现任何 CLI 工具 —— 属于阶段 04。

## 来源

Plan.md 章节：「Knowledge Vault 目录设计」「知识类型」「YAML Frontmatter」「知识正文要求」。
