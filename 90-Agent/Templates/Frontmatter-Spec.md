# YAML Frontmatter 规范

正式知识笔记（`10~80` 目录下）应使用以下 YAML frontmatter 字段。新建笔记时直接复制
[`Note-Template.md`](Note-Template.md)，其中已包含这份 frontmatter 骨架。

```yaml
---
kb_id: kb-xxxxxxxx

title: 标题

type: concept

domain: motor-control
subdomain: sensorless

tags:
  - PMSM
  - STO
  - Observer

status: candidate

source_type: documentation

created: 2026-08-27
updated: 2026-08-27

confidence: medium

sync:
  notion: false
---
```

## 字段说明

| 字段 | 说明 | 取值 |
|---|---|---|
| `kb_id` | 永久唯一 ID，生成后终身不变，与文件路径、标题无关。文件移动、改名、内容大改都不改变 `kb_id`。**不使用文件路径作为知识的唯一标识**——路径可以变，`kb_id` 不能变。 | `kb-` + 8 位小写十六进制，见下方「`kb_id` 生成方式」 |
| `title` | 笔记标题，应与 H1 标题一致 | 自由文本 |
| `type` | 知识类型 | `concept` \| `procedure` \| `troubleshooting` \| `decision` \| `reference` \| `paper` \| `project` \| `experiment` |
| `domain` | 所属领域，建议对齐 `10~80` 目录（如 `embedded`、`motor-control`、`control-theory`、`matlab-simulink`、`toolchains`、`system`、`papers`、`projects`） | 自由文本（小写、连字符分隔） |
| `subdomain` | 领域下的子分类，可选 | 自由文本 |
| `tags` | 关键词标签，便于检索 | 字符串数组 |
| `status` | 知识的成熟度/可信状态 | `candidate` \| `verified` \| `experimental` \| `deprecated` |
| `source_type` | 知识的来源类型 | 如 `documentation`、`experiment`、`paper`、`agent-extraction`、`manual` 等，自由文本 |
| `created` | 创建日期 | `YYYY-MM-DD` |
| `updated` | 最后更新日期，每次修改正文时同步更新 | `YYYY-MM-DD` |
| `confidence` | 内容可信度 | `high` \| `medium` \| `low` |
| `sync.notion` | 是否允许被同步到 Notion（阶段 05 实现同步逻辑前，此字段先声明意图） | `true` \| `false` |

## `status` 取值含义

- `candidate`：刚从 Agent-Candidates 或 Inbox 提炼出来，未经 Curator 审核确认。
- `verified`：已确认可靠，可作为 Agent 检索优先级中的高优先知识源。
- `experimental`：内容基本可信但尚在验证/实验阶段，可能调整。
- `deprecated`：已过时或被 Canonical Note 取代，通常连同笔记一起移入 `99-Archive/`。

## `kb_id` 生成方式

格式：`kb-` + 8 位小写十六进制字符，例如 `kb-3f9a21c4`。

生成命令（WSL2 中任选一种）：

```bash
# 方式一：openssl
echo "kb-$(openssl rand -hex 4)"

# 方式二：/dev/urandom
echo "kb-$(head -c4 /dev/urandom | xxd -p)"
```

生成后必须检查唯一性（极低概率碰撞，但仍需确认），例如：

```bash
rg "kb_id: kb-3f9a21c4" .
```

若已存在，重新生成。人和 Agent 都应遵循同一套生成 + 查重流程，避免 `kb_id` 冲突。
