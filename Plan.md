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

Codex、Claude Code 关闭会话时，自动触发一轮知识整理（Knowledge Curator）和 Notion 同步，
不需要用户每次手动记得去跑 `kb curate` / `kb sync notion`。

## 触发机制

Claude Code 和 Codex CLI 都提供会话结束时执行任意命令的 Hook，事件名分别是
`SessionEnd`（Claude Code，`settings.json` 里的 `hooks.SessionEnd`）和 `session-end`
（Codex CLI，`hooks.json`）。两边都指向同一个共享脚本，而不是各自实现一套触发逻辑——
保证以后如果换掉 Codex 或 Claude Code 中的任意一个，只需要替换触发层，自动化逻辑本身
不用重写（对应「当前任务要求」里"所有组件尽量解耦"的原则）。

生效范围：**全局**——不限定在 Knowledge Vault 这一个项目里，任意项目下关闭 Claude Code
或 Codex 会话都会触发这个 Hook。共享脚本内部固定指向 Knowledge Vault 路径
（`KB_VAULT_PATH`），跟触发它的会话当时在哪个项目目录无关。

## 自动整理的动作范围

Hook 触发后，共享脚本依次执行：

1. **先做一次廉价检查**：Vault 里有没有东西值得处理（`00-Inbox/Agent-Candidates` 是否
   非空、`git status` 是否有未提交改动）。没有就直接退出，不浪费一次模型调用——绝大多数
   跟 Vault 无关的会话关闭都会在这一步就结束。
2. **有东西要处理时，才拉起一次无交互的 Agent 调用**（`claude -p` 或 `codex exec`，用哪个
   是实现细节，只要固定用一个），指令直接复用已有的 `90-Agent/KNOWLEDGE-RULES.md` 和
   `90-Agent/DEVELOPMENT-RULES.md`（不重新定义一套规则），让它扮演 Knowledge Curator：
   审核 `00-Inbox/Agent-Candidates`、去重、合并、修正 YAML、建 WikiLink、维护
   Canonical Note——即「知识生成流程」一节已经定义的判断逻辑，只是从"人工触发"变成
   "自动触发"。内容冲突仍然按现有规则标记待人工审核，不强行覆盖。
3. `kb validate` + `kb curate` 兜底检查一遍。
4. `kb sync notion`（只同步 `sync.notion: true` 的笔记，机制不变，见「Notion 同步机制」）。
5. 如果有改动：`git add` + `git commit`，提交信息明确标注是自动整理（例如
   `chore(auto-curate): session-end 自动知识整理`），方便日后在 `git log` 里筛选、或者
   单独 `git revert` 某一次自动提交。**只本地提交，不 push**——跟现有 Git 规则一致
   （见「Git」一节：Push 属于需要用户明确授权的操作）。

## 安全阀（即使全自动，也要保留的边界）

- 单次自动整理如果触发了 `90-Agent/DEVELOPMENT-RULES.md` 定义的"大规模修改"阈值（改动
  超过 5 篇正式笔记、涉及目录结构调整、批量删除、改动 90-Agent 规则文档本身），这一次
  **不自动 commit**，改动留在工作区，`git status` 能看到，等用户自己确认后再提交——防止
  自动化在无人看着的时候做出大范围、难以追溯的改动。
- 敏感信息禁止清单（密码/Token/Cookie/Private Key/聊天记录原文等，见「知识生成流程」）
  在自动模式下同样生效，不因为是自动触发就放宽。
- 并发保护：可能有多个 Claude Code / Codex 会话先后关闭，共享脚本需要一个简单的文件锁
  （例如 `.knowledge/session-end.lock`），避免多个自动整理同时跑、互相踩到对方改动或
  产生冲突的 git 提交。
- Hook 本身通常有执行超时（Claude Code、Codex 都对 SessionEnd Hook 设了超时），一次完整
  的模型调用可能跑不完；脚本要能优雅处理（例如异步执行、下次会话开始时报告上次是否正常
  完成），不能让超时导致改到一半的不一致状态。这属于实现阶段要解决的细节，这里不预先
  下定论。

## 与既有规则的关系

这是 `90-Agent/DEVELOPMENT-RULES.md` 里"只在用户明确要求时才 commit"规则的一个**明确
例外**：用户已经明确同意这一条自动化流水线可以本地自动提交。除了这条流水线之外，Agent
在正常交互会话里改动 Vault 仍然遵循原有规则——不能因为这里开了口子，就在其他场景也擅自
commit。真正落地实现时，需要把这条例外补写回 `90-Agent/DEVELOPMENT-RULES.md`，让规则
文档和实际行为保持一致（属于对应执行阶段的任务，不在本次 Plan.md 更新范围内）。

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
