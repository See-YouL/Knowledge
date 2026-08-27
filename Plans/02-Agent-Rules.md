---
status: done
depends_on: [01-Vault-Scaffold]
---

# 阶段 02：Agent 规则、知识生成流程与 Curator 职责

## 目标

让 Codex、Claude Code 等 Agent 有一套明确、可长期依赖的规则：如何读取知识库而不递归加载
整个 Vault，如何把任务中产生的临时信息提炼成正式知识而不是直接堆聊天记录，以及由谁
（Knowledge Curator 角色）负责审核、去重、合并，维持「一个概念一个 Canonical Note」。

## 范围（做什么）

### 90-Agent/INDEX.md

- [x] 作为 Agent 任务开始时第一个要读的文件：列出各领域目录（`10~80`）的用途一句话说明、
      指向 `01-Index/` 下各索引文件、指向 `KNOWLEDGE-RULES.md` / `TAXONOMY.md` /
      `DEVELOPMENT-RULES.md`。

### 90-Agent/KNOWLEDGE-RULES.md

- [x] 落地「知识生成流程」：完成任务 → 判断是否有长期可复用知识 → 提炼 → 生成结构化
      Candidate → 放入 `00-Inbox/Agent-Candidates` → Curator 审核 → 更新/合并/新建/标记冲突/丢弃。
- [x] 落地禁止直接保存的内容清单：整段聊天记录、大量终端输出、临时调试信息、Agent 推理
      过程、未验证的猜测、密码/API Token/Cookie/Private Key 等敏感信息。
- [x] 落地「Agent 知识读取原则」：任务 → 读 `INDEX.md` → 识别领域 → 搜索相关知识 → 只加载
      少量相关 Markdown → 执行任务；以及知识优先级顺序（当前项目明确规则 > 已验证项目知识 >
      verified 技术知识 > 官方文档 > 论文/实验资料 > experimental > Agent 推测）。

### 90-Agent/TAXONOMY.md

- [x] 落地知识类型定义与示例（`concept/procedure/troubleshooting/decision/reference/
      paper/project/experiment`，各配一个 Plan.md 中给出的示例）。
- [x] 落地 `domain`/`subdomain` 与 `10~80` 目录的对应关系，作为分类时的参考表。

### 90-Agent/DEVELOPMENT-RULES.md

- [x] 落地 Knowledge Curator 角色职责清单：Candidate 审核、去重、合并、分类、修正 YAML、
      建立 WikiLink、检查冲突知识、检查 Broken Links、检查重复笔记、维护 Canonical Note。
- [x] 落地「一个概念一个 Canonical Note」原则，明确反例（`PLL.md` / `PLL基础.md` /
      `PLL总结.md` / `PLL新版.md` / `PLL-final.md` 这类应避免的命名膨胀），要求更新已有笔记
      而不是新建变体。

## 明确不做（Out of Scope）

- 不实现自动化的 Curator 脚本或去重算法 —— 这些规则先作为人工/Agent 执行时遵循的文字规范，
  自动化留给阶段 04 的 `kb curate` 命令去逐步实现。
- 不涉及 Git 提交前后的检查约定 —— 属于阶段 03（虽然「禁止未授权大规模删除」在 Plan.md 的
  Git 一节，但落地到 `DEVELOPMENT-RULES.md` 时应与阶段 03 的产出交叉引用，而不是重复定义）。

## 来源

Plan.md 章节：「知识生成流程」「Agent 知识读取原则」「Knowledge Curator」。
