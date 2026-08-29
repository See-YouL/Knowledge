# 目标

我要搭建一个长期可维护、可扩展的本地自生成知识库系统，主要供我本人阅读和管理，同时让 Codex、Claude Code 等 AI Agent 能够长期读取、搜索、补充、整理和复用知识。

整个系统应遵循以下原则：

- Markdown 知识库是唯一的 Source of Truth。
- Obsidian 是我的主要知识管理和阅读界面。
- Codex、Claude Code 等 Agent 是知识的使用者和生产者。
- Notion 是远程展示、移动端阅读、数据库管理和学习管理层，不作为主要知识源。
- 不允许 Obsidian 与 Notion 无控制地双向覆盖。
- 知识库必须 Human-readable，同时 Agent-friendly。
- 尽量使用开放格式，不与某个特定 Agent 或软件强绑定。

# 当前环境

Windows 11 上的 Obsidian Vault 实际路径为：

```text
C:\Personal\Project\Obsidian\Knowledge
```

WSL2 中对应入口为：

```text
/home/eric/Personal/Obsidian/Knowledge
```

这个 WSL2 路径已经通过 ln 符号链接指向 Windows 中的实际 Knowledge 目录。

因此 Windows Obsidian、WSL2 Codex、Claude Code 实际访问的是同一套 Markdown 文件。

Codex 和 Claude Code 主要运行于 WSL2。

# 总体架构

目标架构：

```text
                        用户
                         │
                         ▼
                     Obsidian
                         │
                         ▼
C:\Personal\Project\Obsidian\Knowledge
              唯一真实知识库
                         ▲
                         │
/home/eric/Personal/Obsidian/Knowledge
                         │
                ┌────────┴────────┐
                │                 │
              Codex          Claude Code
                │                 │
                └────────┬────────┘
                         │
                  Knowledge Engine
                         │
       ┌─────────────────┼──────────────────┐
       │                 │                  │
     Search            Curator            Sync
       │                 │                  │
       ▼                 ▼                  ▼
 Markdown检索       自动整理知识          Notion
```

# Knowledge Vault 目录设计

知识库需要采用清晰、长期稳定的目录结构，例如：

```text
Knowledge/
├── 00-Inbox/
│   ├── Manual/
│   ├── Agent-Candidates/
│   └── Unsorted/
│
├── 01-Index/
│   ├── Home.md
│   ├── Embedded.md
│   ├── Motor-Control.md
│   ├── Control-Theory.md
│   └── System.md
│
├── 10-Embedded/
├── 20-Motor-Control/
├── 30-Control-Theory/
├── 40-MATLAB-Simulink/
├── 50-Toolchains/
├── 60-System/
├── 70-Papers/
├── 80-Projects/
│
├── 90-Agent/
│   ├── INDEX.md
│   ├── KNOWLEDGE-RULES.md
│   ├── TAXONOMY.md
│   ├── DEVELOPMENT-RULES.md
│   └── Templates/
│
├── 99-Archive/
│
└── .knowledge/
```

其中：

- `00-Inbox/Agent-Candidates` 用于存放 Agent 自动提取但尚未正式确认的知识。
- `10~80` 是正式知识主体。
- `90-Agent` 存放 Agent 如何使用知识库的规则。
- `.knowledge` 存放同步状态、索引、映射等机器数据。

# 知识生成流程

Agent 完成一个具有长期价值的任务后，不应直接把聊天记录保存进知识库。

必须进行 Knowledge Extraction。

正确流程：

```text
完成任务
   ↓
判断是否存在长期可复用知识
   ↓
提炼知识
   ↓
生成结构化 Candidate
   ↓
00-Inbox/Agent-Candidates
   ↓
Knowledge Curator
   ↓
搜索已有知识
   ↓
判断：
├── 已存在 → 更新
├── 部分重复 → 合并
├── 新知识 → 创建
├── 内容冲突 → 标记人工审核
└── 无长期价值 → 丢弃
   ↓
正式知识
```

禁止简单保存：

- 整段聊天记录
- 大量终端输出
- 临时调试信息
- Agent 推理过程
- 未验证的猜测
- 密码
- API Token
- Cookie
- Private Key
- 其他敏感信息

# 知识类型

每条知识应具有明确的 type，例如：

```text
concept
procedure
troubleshooting
decision
reference
paper
project
experiment
```

例如：

- `concept`：PLL 是什么
- `procedure`：如何配置 STM32 Clangd
- `troubleshooting`：WSL npm 网络错误排查
- `decision`：为什么 ADC 采用 TIM1 触发
- `paper`：论文问题、方法和结论
- `experiment`：实验条件和结果

# YAML Frontmatter

正式知识应尽量采用统一 YAML，例如：

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

status: verified

source_type: documentation

created: 2026-08-27
updated: 2026-08-27

confidence: high

sync:
  notion: true
---
```

要求：

- `kb_id` 是永久唯一 ID。
- 文件移动、改名后 `kb_id` 不改变。
- 不使用文件路径作为知识唯一标识。
- `status` 至少支持：
  - candidate
  - verified
  - experimental
  - deprecated
- `confidence` 至少支持：
  - high
  - medium
  - low

# 知识正文要求

知识首先应保证人类可读。

推荐结构：

```markdown
# 标题

## 问题 / 定义

## 核心原理

## 详细解释

## 实现或推导

## 常见问题

## 相关知识

- [[相关概念1]]
- [[相关概念2]]

## 来源
```

不要为了 Agent 检索而生成大量难以阅读的机器文本。

遵循：

Human-readable first, Agent-retrievable by design.

# Agent 知识读取原则

Codex、Claude Code 不应该在每次任务开始时递归加载整个 Vault。

正确流程：

```text
任务
 ↓
读取 90-Agent/INDEX.md
 ↓
识别当前领域
 ↓
搜索相关知识
 ↓
只加载少量相关 Markdown
 ↓
执行任务
```

优先级：

```text
当前项目明确规则
>
已验证项目知识
>
verified 技术知识
>
官方文档
>
论文/实验资料
>
experimental
>
Agent 推测
```

# Knowledge Curator

需要设计 Knowledge Curator 角色。

职责包括：

- Candidate 审核
- 去重
- 合并
- 分类
- 修正 YAML
- 建立 Obsidian WikiLink
- 检查冲突知识
- 检查 Broken Links
- 检查重复笔记
- 维护 Canonical Note

尽可能遵守：

一个概念只有一个 Canonical Note。

不要不断产生：

```text
PLL.md
PLL基础.md
PLL总结.md
PLL新版.md
PLL-final.md
```

而应更新已有 Canonical Note。

# Knowledge Engine

Knowledge Engine 程序与 Knowledge 数据必须分开。

建议代码目录：

```text
/home/eric/Personal/KnowledgeEngine
```

知识目录：

```text
/home/eric/Personal/Obsidian/Knowledge
```

Knowledge Engine 最终应提供统一 CLI，例如：

```bash
kb search "PLL"
kb read "PLL"
kb related "PLL"
kb capture
kb curate
kb validate
kb status
kb sync notion
kb sync notion --dry-run
```

Codex、Claude Code 等 Agent 后期应优先通过统一的 `kb` 接口访问知识系统，而不是各自实现一套知识管理逻辑。

# Notion 定位

Notion 不作为 Source of Truth。

默认同步方向：

```text
Obsidian / Markdown
        ↓
Knowledge Engine
        ↓
Notion
```

正文不做完全双向同步。

Notion 主要负责：

- 手机阅读
- Web 阅读
- Database
- 筛选
- 分类
- 学习状态
- Review
- 分享

可以考虑有限的：

```text
Notion → Knowledge
```

但只同步少量管理属性，例如：

- Review Status
- Favorite
- Priority
- Rating
- Next Review Date

不要让 Notion 正文自动覆盖本地 Markdown 正文。

# Notion Database

建议使用一个核心 Knowledge Database。

字段可包括：

```text
Name
KB ID
Domain
Subdomain
Type
Status
Tags
Source
Created
Updated
Confidence
Local Path
Sync Status
Review Status
```

Markdown YAML 与 Notion Database Property 应建立明确映射。

# Notion 同步机制

Knowledge Engine 应维护：

```text
.knowledge/notion-map.json
```

记录：

```text
kb_id
→ notion_page_id
→ content_hash
```

例如：

```json
{
  "kb-sto-001": {
    "notion_page_id": "xxxx",
    "content_hash": "sha256:xxxx"
  }
}
```

同步时：

1. 扫描允许同步的 Markdown。
2. 读取 `kb_id`。
3. 计算 content hash。
4. 与上次 hash 比较。
5. 未修改则跳过。
6. 修改则更新对应 Notion Page。
7. 新知识则创建 Notion Page。
8. 更新 notion-map。

必须支持增量同步，而不是每次全量上传。

# Obsidian 特殊语法

同步 Notion 时需要考虑：

```text
[[WikiLink]]
![[图片]]
YAML Frontmatter
数学公式
内部附件
```

后期应实现：

```text
[[PLL]]
↓
查找对应 kb_id
↓
查找 notion_page_id
↓
转换成 Notion 页面链接
```

图片和 PDF 后期通过文件上传机制解决。

数学公式应尽量保持 LaTeX，不转换成图片。

# Git

整个 Knowledge Vault 应使用 Git 版本控制。

Agent 对知识库进行大规模修改前后，应尽可能能够：

```bash
git diff
git status
```

发生错误时可以恢复历史版本。

禁止 Agent 在未经明确授权时进行危险的大规模删除。

# Agent 会话结束自动化

## 目标

每次 Codex、Claude Code 开启一个新会话时，一开始就主动问用户："这次对话如果产生了值得
长期保留的知识，结束时要不要自动整理保存？"——问的是"这次"，答案也只管"这次"，不是替
上一次或下一次对话做决定。

## 触发机制

Claude Code 和 Codex CLI 都提供 SessionStart 和 SessionEnd 两个 Hook（Claude Code：
`hooks.SessionStart` / `hooks.SessionEnd`；Codex：`hooks.json` 的 `session-start` /
`session-end`）。两边都指向同一套共享逻辑，不各自实现一遍——保证以后如果换掉 Codex 或
Claude Code 中的任意一个，只需要替换触发层，自动化逻辑本身不用重写（对应「当前任务要求」
里"所有组件尽量解耦"的原则）。

生效范围：**全局**——任意项目下开启的 Claude Code / Codex 会话都会被问，因为知识可能来自
任何一次任务。

## 三段式流程

### 第一段：SessionStart（会话一开始就问，问的是"这次"）

会话刚开始，共享逻辑通过 Hook 往当前会话注入一条指令：让 Agent 在第一条回复里主动问用户
"这次对话结束时要不要自动整理保存产生的知识"，并把用户的回答（是/否）连同这次会话的
session id 一起记到 `.knowledge/session-consent/<session_id>.json`。这一步问的、记的，
都只针对**当前这一次**会话，跟之前或之后的会话无关。

### 第二段：会话进行中（受当前会话的同意状态控制）

只有这次会话在开头得到了"是"，Agent 才会在过程中按「知识生成流程」一节已有的规则，主动
把有长期价值的内容整理成结构化 Candidate 写进 `00-Inbox/Agent-Candidates`；如果开头是
"否"，这次对话里 Agent 不主动写 Candidate（用户仍然可以随时手动 `kb capture`，这条开关
只管"要不要主动帮你存"）。

### 第三段：SessionEnd（无交互，补偿提炼并收尾）

会话关闭时，共享脚本读取 `.knowledge/session-consent/<session_id>.json`：

- 没有记录，或者是"否"：直接清理、退出，不动 Vault 任何东西。
- 是"是"：无论会话内是否已经产生 Candidate，都拉起一次无交互 Agent 调用（固定用
  `claude -p`）。先把本次 transcript 过滤成只含用户和 Assistant 自然语言的脱敏视图，排除
  系统提示、推理、工具调用和终端输出；Agent 据此审计漏记知识并补建 Candidate，再复用
  `90-Agent/KNOWLEDGE-RULES.md`/`DEVELOPMENT-RULES.md` 去重、合并、修正 YAML、建 WikiLink，冲突就
  标记待人工审核，不强行覆盖 → `kb validate` + `kb curate` 兜底 → `kb sync notion` →
  有改动就 `git commit`（本地，不 push，消息标注是这次确认后的整理）。
- 审计后确认没有长期知识：不修改 Vault，记录结果后清理、退出。

最后清掉这次会话的 consent 记录文件，不留状态。

## 安全阀

- 单次整理如果触发了 `90-Agent/DEVELOPMENT-RULES.md` 定义的"大规模修改"阈值（改动超过
  5 篇正式笔记、涉及目录结构调整、批量删除、改动 90-Agent 规则文档本身），这一次不自动
  commit，改动留在工作区，等用户自己确认。
- 敏感信息禁止清单（密码/Token/Cookie/Private Key/聊天记录原文等）同样生效，不因为用户
  开头同意了就放宽。
- `git commit` 只在用户对"这次对话"明确回答"是"之后才可能发生，属于「只在用户明确要求
  时才 commit」规则本身覆盖的正常情况，不需要单独开例外。
- 仍然不 push；仍然不做正文层面的 Notion → Vault 回写。
- `.knowledge/session-consent/` 下的文件按 session id 隔离，各会话互不影响，天然没有
  并发覆盖的问题（不需要像共享一个队列文件那样加锁）。

# 后续扩展

第一阶段不要直接引入复杂 Vector Database。

先使用：

```text
Markdown
+
目录
+
YAML
+
WikiLink
+
ripgrep
+
文件搜索
```

当知识规模明显增长以后，再升级：

```text
Keyword Search
+
Embedding
+
Vector Database
+
Reranker
+
Knowledge MCP Server
```

最终可以形成：

```text
Codex
Claude Code
Gemini CLI
OpenCode
其他 Agent
       │
       ▼
Knowledge MCP Server
       │
       ▼
Knowledge Engine
       │
       ▼
Markdown Vault
       │
   ┌───┴────┐
   ▼        ▼
Obsidian   Notion
```

# 当前任务要求

请基于以上架构帮助我继续设计或实现系统。

每次需要修改架构时，应优先保证：

1. Markdown Vault 仍然是唯一 Source of Truth。
2. Obsidian 中的知识仍然适合我本人长期阅读。
3. Agent 可以可靠检索和补充知识。
4. 不产生重复知识。
5. 不让 Notion 与本地 Markdown 产生不可控冲突。
6. 所有组件尽量解耦。
7. 后续可以替换 Codex、Claude Code、Obsidian 或 Notion，而不会导致知识丢失。
8. 优先采用简单、透明、可维护的方案，再逐步增加 MCP、RAG、Embedding 等复杂能力。

在设计或实现任何功能前，请先检查它是否符合以上原则。
