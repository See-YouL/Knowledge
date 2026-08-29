---
status: done
depends_on: [02-Agent-Rules, 03-Git-Workflow, 04-Knowledge-Engine-CLI, 05-Notion-Sync]
---

# 阶段 07：Agent 会话结束自动化（开头问，管这一次）

## 目标

每次 Codex、Claude Code 开启一个新会话时，一开始就主动问用户"这次对话结束时要不要自动
整理保存产生的知识"——问的是"这次"，答案也只管"这次"，不是替上一次或下一次对话做决定。
完整设计已经写在 Plan.md 的「Agent 会话结束自动化」一节，本阶段负责落地实现。

依赖 02（复用 `KNOWLEDGE-RULES.md`/`DEVELOPMENT-RULES.md` 规则）、03（遵循 Git 安全规则）、
04（调用 `kb validate`/`kb curate`）、05（调用 `kb sync notion`）。不依赖阶段 06（语义检索 +
MCP Server）——SessionEnd 端的整理调用直接用 `kb` CLI + 一次无交互 Agent 调用，不强制经过
MCP。

## 范围（做什么）

- [x] 写共享逻辑，**SessionStart 端**：通过 Hook 往新会话注入指令，让 Agent 在第一条回复
      里主动问用户"这次对话结束时要不要自动整理保存知识"，把回答（是/否）连同这次会话的
      session id 写入 `.knowledge/session-consent/<session_id>.json`。只管当前这一次
      会话，不涉及之前或之后的会话。
      → `kb session start`（KnowledgeEngine 仓库 `kb/session.py` + `kb/commands/session.py`）；
      Agent 记录回答走 `kb session consent --session <id> --answer yes|no`。
- [x] 在 `90-Agent/KNOWLEDGE-RULES.md` 补一句：只有当前会话的 consent 是"是"，Agent 才
      在过程中主动把长期价值内容写成 Candidate；是"否"则这次对话不主动写（用户仍可以
      随时手动 `kb capture`）。→ 新增「二、会话级同意开关」一节，原二/三节顺延为三/四。
- [x] 写共享逻辑，**SessionEnd 端**：读取 `.knowledge/session-consent/<session_id>.json`。
      没有记录或是"否" → 直接清理退出，不动 Vault。是"是" → 记录 `transcript_path` 并拉起
      一次无交互 Agent 调用（固定用 `claude -p`）；先从仅含用户/Assistant自然语言、已排除
      系统提示/推理/工具输出并脱敏的 transcript 视图中补建漏掉的 Candidate，再复用
      `90-Agent/KNOWLEDGE-RULES.md`/`DEVELOPMENT-RULES.md` 处理本次 Candidate
      （去重/合并/修 YAML/建 WikiLink，冲突就标记待人工审核）→ `kb validate` +
      `kb curate` 兜底 → `kb sync notion` → 有改动就 `git commit`（本地，不 push，消息
      标注是这次确认后的整理）。审计确认没有长期知识 → 不修改 Vault、记录结果后退出。
      最后清掉这次会话的 consent 记录文件。→ `kb session end` + `kb session run`
      （`kb/session_pipeline.py`）。
- [x] Claude Code 接入：`hooks.SessionStart` + `hooks.SessionEnd`（`~/.claude/settings.json`，
      全局生效）。
- [x] Codex CLI 接入：`~/.codex/hooks.json` 的 `SessionStart` + `SessionEnd`。
- [x] "大规模修改"阈值检测：复用 `90-Agent/DEVELOPMENT-RULES.md` 已有标准（改动超过 5 篇
      正式笔记、涉及目录结构调整、批量删除、改动 90-Agent 规则文档本身）——命中阈值时不
      自动 commit，留给用户自己确认。
- [x] 端到端验证：见下「验证记录」。

## 实现要点与踩到的坑

### 一份逻辑，两个触发层

Claude Code 2.1.251 和 Codex 0.149.1 送进 Hook stdin 的 JSON **字段名完全一致**
（`session_id`/`cwd`/`hook_event_name`/`transcript_path`，SessionStart 多 `source`，
SessionEnd 多 `reason`），SessionStart 的响应格式（`hookSpecificOutput.additionalContext`）
也一致；Codex 的 `hooks.json` 就是 Claude 那套 schema（它自带的 external-agent-migration
模块正是把 `.claude` 配置转成 `hooks.json`）。所以共享逻辑只写了一份解析代码，两边的配置
都只是 `<venv>/bin/kb session start|end` 这一行——换掉任意一个 Agent 只需替换触发层。

Claude Code 的 SessionStart 还支持 `initialUserMessage`（能在用户开口前就让 Agent 发问），
Codex 没有对应字段，**有意不用**，保持两边对称。

### SessionEnd 必须秒回，重活自己 detach

Claude Code 是同步 `await` SessionEnd Hook 的（`AbortSignal.timeout(getSessionEndHookTimeoutMs())`
加一个 failsafe），而且 SessionEnd **不在**它支持 async 回执的事件白名单里；Codex 更直接，
会把 SessionEnd 的 timeout 钳到 3s（每次会话打一条 warning，所以 `hooks.json` 里那一条
直接按 3 声明）。因此 `kb session end` 只做廉价判断，需要整理时用
`Popen(..., start_new_session=True)` 把 `kb session run` 甩到后台，宿主退出后继续跑。

### 防递归

`claude -p` 会重新触发 Claude Code 自己的 SessionStart/SessionEnd Hook。守卫是环境变量
`KB_SESSION_AUTOMATION=1`，两个 Hook 入口第一行检查并立即退出——一个变量同时覆盖两条链路。
**没有用 `claude --bare`**：它虽然跳过 hooks，但同时强制"认证只走 `ANTHROPIC_API_KEY`，
永不读 OAuth/keychain"，而本机是 OAuth 登录，用了会直接认证失败。

### git 提交身份（本阶段最容易踩的坑）

这个 Vault 平时是从 **Windows 侧**提交的，身份配在 `C:\Users\jff_1\.gitconfig`；WSL 的
HOME 不同，`git config user.email` 是空的，所以流水线里直接 `git commit` 会
`fatal: empty ident name`。处理办法：WSL 侧有配置就用配置，没有就沿用**这个仓库自己历史
提交**的作者（`git log -1 --format=%an/%ae`）临时用 `-c` 指定，署名和用户平时的提交一致；
两者都取不到就不提交、把改动留在工作区并在日志里写明。**不去写用户的 git config。**

### 只提交这次流水线自己产生的改动

流水线开始前先记 `git status --porcelain -z` 基线（用 `-z` 是因为 Vault 全是中文文件名，
默认输出会被引号转义）。基线里已有的改动——例如用户正在 Obsidian 里写的别的笔记——不纳入
自动提交，`git add` 只加这次新出现的路径，不用 `git add -A`。这次会话自己产生的 Candidate
排除在基线之外（它们是本次整理的输入/产物，该跟着一起提交）。

### 整理 Agent 需要一个"清 Candidate"的合法出口

第一次真实端到端跑的时候，整理 Agent 想 `rm` 掉已经转正的 Candidate，被权限拒了
（受限工具集里 `rm` 是 deny 的），结果 Inbox 永远排不空。加了
`kb candidate resolve <文件名> --as merged|promoted|discarded`：按构造只能删
`00-Inbox/Agent-Candidates/` 下的文件，不需要给整理 Agent 开放通用删除能力。
另外整理 Agent 的权限白名单必须同时覆盖 `kb` 的绝对路径和裸 `kb`，并把 venv 的 bin
目录加进它的 PATH——否则它调 `kb` 一样会被拒。

### 权限模式

无交互整理用 `claude -p --permission-mode acceptEdits --tools Read,Write,Edit,Glob,Grep,Bash`
加一份通过 `--settings` 传入的 JSON 权限配置（allow 只有 `kb`/`git status`/`git diff`/
`git log`，deny 掉 `git commit`/`push`/`reset`/`clean`/`checkout`/`rm`/`mv`/`WebFetch`/
`WebSearch`），再加 `--strict-mcp-config`（关掉全部 MCP）、`--max-budget-usd`、超时。
**没有用 `--dangerously-skip-permissions`**。权限规则写进 JSON 而不是 `--allowedTools`，
是因为后者按逗号/空格切分，`Bash(git status*)` 这种带空格的模式会被切坏。

## 验证记录

在一个复刻真实骨架的临时 Vault 里做端到端验证（避免污染真实 Vault），加上真实 Vault 上的
Hook 触发验证：

| 场景 | 结果 |
|---|---|
| consent=是 + 会话中产生 Candidate | Candidate 被整理成 `60-System/` 下的正式笔记、Inbox 排空、validate 通过、只提交本次路径；`git revert HEAD` 干净撤销 ✓ |
| consent=是 + 无新 Candidate | SessionEnd 仍启动 transcript 审计；无长期知识才无改动退出 ✓ |
| consent=否（中文"否"也识别） | 关闭后不整理、不提交 ✓ |
| 未回答（consent 为空） | 关闭后直接清理退出 ✓ |
| 两个不同 session id | consent 文件按 session id 隔离，互不影响 ✓ |
| resume / compact 来源 | 不重复提问 ✓ |
| 子 Agent 会话（`agent_type`） | 不提问、不写记录 ✓ |
| 防递归（`KB_SESSION_AUTOMATION`） | 两个 Hook 入口静默退出 ✓ |
| 大规模修改阈值 | 四条标准逐条单测通过；端到端造 6 篇正式笔记改动 → 不提交、改动留工作区、日志写明命中哪条 ✓ |
| Notion 凭证缺失 | 跳过同步、不阻断整理与提交 ✓ |
| 用户已有的无关未提交改动 | 不被自动提交带走 ✓ |
| Claude Code 真实 Hook | `claude -p` 会话里 SessionStart 写入 consent 记录、注入内容送达模型（模型原样回显了 session id）、SessionEnd 清理干净 ✓ |
| Codex 真实 Hook | `codex exec` 正确解析 `hooks.json`（并报出 SessionEnd 3s 钳制），hooks 已登记进 `[hooks.state]`；仍需人工信任一次，见下 ⚠ |

验证用的临时笔记、Candidate、提交和日志均已清理。

## 已知限制

- **Codex 侧需要用户手动信任一次**：Codex 首次加载新的 `hooks.json` 会把它记成
  `enabled = false`（信任状态在 `~/.codex/config.toml` 的 `[hooks.state]`，带
  `trusted_hash`），要在 Codex TUI 里确认一次才会真正执行。这是 Codex 的安全设计，
  不该由脚本代劳。改动 `hooks.json` 后哈希变化，需要重新确认。
- **终端被强杀时 SessionEnd 不保证触发**（关窗口、WSL 关机），那一次整理就不会发生，
  会留下孤儿 consent 文件。按 Plan.md「不设计跨会话的待处理队列」，只做 7 天过期清理
  （下次 SessionStart 顺手做），**不自动补跑、不在下次会话里提醒**；需要补跑就人工
  `kb session run --session <id>`。
- **Claude Code 的 `/clear`** 会触发 `SessionEnd(reason=clear)` 加 `SessionStart(source=clear)`，
  即正常走一次整理并重新问一次。这符合"只管这一次"的语义。
- **每次触发整理都是一次真实的付费模型调用**（默认上限 `--max-budget-usd 1.00` 加
  900s 超时，可用 `KB_SESSION_AGENT_BUDGET`/`KB_SESSION_AGENT_TIMEOUT` 调）。
- **全局生效**意味着任何项目下的会话开头都会多一个确认问题——这是 Plan.md 的既定取舍
  （知识可能来自任何一次任务）。

## 明确不做（Out of Scope）

- 不 push——保持「Git」一节的既有规则，push 需要用户明确授权。
- 不做正文层面的 Notion → Vault 回写——超出 Plan.md 已定的硬性边界（正文永远不被 Notion
  覆盖），只有既定的五个管理属性通过 `kb sync notion --pull` 回写，这条不变。
- 不强制这条流水线经过 Knowledge MCP Server——SessionEnd 的整理调用直接用 `kb` CLI，
  MCP Server 是给交互式 Agent 用的检索入口，不是这条自动化流水线的依赖。
- 不做"上一次对话"或"下一次对话"的追溯/预约——每次会话只对自己的开头提问和自己的结尾
  行为负责，不设计跨会话的待处理队列。
- 不做成"两个 Agent 可切换"的整理后端——按 Plan.md「固定用一个」，固定 `claude -p`
  （`KB_SESSION_AGENT_BIN` 只是给排查和测试留的口子，不是产品化的切换开关）。

## 来源

Plan.md 章节：「Agent 会话结束自动化」。
