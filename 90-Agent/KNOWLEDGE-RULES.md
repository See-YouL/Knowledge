# 知识规则

面向 Agent（也适用于人工）的三条核心规则：怎么把任务成果变成知识、什么绝对不能直接存进
知识库、按什么顺序读取已有知识。

## 一、知识生成流程

完成一个具有长期价值的任务后，**不应直接把聊天记录保存进知识库**，必须先做 Knowledge
Extraction：

```text
完成任务
   ↓
判断是否存在长期可复用知识
   ↓
提炼知识
   ↓
生成结构化 Candidate（用 Templates/Note-Template.md 的结构）
   ↓
放入 00-Inbox/Agent-Candidates/
   ↓
Knowledge Curator 审核（见 DEVELOPMENT-RULES.md）
   ↓
搜索已有知识
   ↓
判断：
├── 已存在 → 更新已有笔记（保留 kb_id）
├── 部分重复 → 合并进已有笔记
├── 新知识 → 在对应 10~80 目录下新建正式笔记
├── 内容冲突 → 标记人工审核，不要自行覆盖
└── 无长期价值 → 丢弃，不进入正式知识库
   ↓
正式知识
```

判断"是否有长期价值"的粗略标准：这条信息在未来类似任务中会被再次用到吗？如果只对
当前这一次任务有意义（例如某次具体的报错堆栈、某个临时环境变量），就不是长期知识。

## 二、会话级同意开关（只管当前这一次对话）

每次 Codex / Claude Code 开启新会话时，Hook 会让 Agent 在第一条回复的开头问一次：
「这次对话如果产生了值得长期保留的知识，结束时要不要自动整理保存？」用户的回答记在
`.knowledge/session-consent/<session_id>.json`，**只对当前这一次会话生效**，不替上一次或
下一次对话做决定。

- 当前会话答**「是」**：按上面第一节的流程，主动把有长期价值的内容提炼成 Candidate 写进
  `00-Inbox/Agent-Candidates/`。会话结束时自动整理、校验、同步并在本地提交（见
  [`DEVELOPMENT-RULES.md`](DEVELOPMENT-RULES.md) 的 Git 工作流规则）。
- 当前会话答**「否」**，或者根本没问到答案：这次对话**不主动写 Candidate**，会话结束时
  也不动 Vault。

这个开关只管「要不要主动帮你存」，不改变别的任何规则：

- 用户随时可以手动 `kb capture` 写 Candidate，不受开关影响。
- 下面第三节「禁止直接保存的内容」照常生效——答了「是」不等于放宽敏感信息和原始输出的限制。
- 会话结束时的自动整理，仍然受 `DEVELOPMENT-RULES.md`「什么算大规模修改」的阈值约束：
  命中阈值就把改动留在工作区等人工确认，不自动提交。

分阶段执行记录见 [`Plans/07-Session-End-Automation.md`](../Plans/07-Session-End-Automation.md)。

## 三、禁止直接保存的内容

以下内容不允许原样堆进知识库（Candidate 或正式笔记都不行）：

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

如果这些内容里包含有价值的结论，应该提炼出结论本身（例如"WSL 下 npm 网络错误的根因和
修复方法"），而不是保留原始输出或推理过程。敏感信息（密码/Token/Key/Cookie）无论如何都
不写入 Vault，即使是作为"如何配置"的示例也要用占位符替代。

## 四、Agent 知识读取原则

不应在每次任务开始时递归加载整个 Vault：

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

当多个知识来源冲突时，按以下优先级判断：

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

对应到 YAML `status` 字段：优先信任 `verified`，其次 `experimental`，`candidate` 仅作为
参考、不作为结论依据，`deprecated` 只在需要了解历史决策时才查阅。
