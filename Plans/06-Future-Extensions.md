---
status: done
depends_on: [01-Vault-Scaffold, 02-Agent-Rules, 03-Git-Workflow, 04-Knowledge-Engine-CLI, 05-Notion-Sync]
---

# 阶段 06：语义检索 + Knowledge MCP Server

## 目标

原本记录在这里的是"远期路线图，明确不在近期执行范围内"——触发条件是"知识规模明显增长"，
而这个条件在启动本阶段时并未满足（`kb status` 当时显示 0 篇正式笔记）。**用户在知情的情况下
明确要求提前执行**，所以这不是"规模驱动的自然升级"，而是一次主动的能力建设。据此调整了
实现方式：跳过"等内容长起来"这个前提，但仍然贯彻 Plan.md「优先采用简单、透明、可维护的
方案」的原则——没有引入 sqlite-vec / FAISS / Chroma 这类专门的 Vector Database 组件，
用一个 JSON 文件 + numpy 线性扫描代替，量级上对个人知识库完全够用；等量级真的撑不住了，
再单独升级存储层，不影响上层调用接口。

## 范围（已完成）

- [x] Keyword Search（阶段 04 已实现的 ripgrep 检索，继续作为 `kb search` 的基线，不受影响）。
- [x] Embedding：本地模型（`kb/embeddings.py`），离线运行，不需要 API Key。
- [x] "Vector Database"：**没有引入专门的 Vector DB 引擎**，用
      `.knowledge/embeddings.json`（`kb_id → 向量 + content_hash + 标题/路径/正文摘要`）
      + `kb/vector_store.py`（numpy 向量化点积做余弦相似度线性扫描）代替
      （`kb/commands/index.py` 负责增量构建，逻辑跟 `notion_sync.py` 的 content-hash
      增量比较是同一套思路）。这是本阶段唯一偏离原始清单字面表述的地方，特此说明。
- [x] Reranker：本地 cross-encoder（`kb/embeddings.py` 里的 `rerank()`），`kb semantic-search
      --rerank` 时才会加载调用，默认关闭。
- [x] Knowledge MCP Server —— `kb/mcp_server.py`，官方 `mcp` SDK（2.x，MCPServer/原
      FastMCP），stdio transport，工具集：`kb_search / kb_semantic_search / kb_read /
      kb_related / kb_status / kb_capture`（有意不暴露 `kb_curate`/`kb_validate`/
      `kb sync notion`——前两个是给人/Curator 用的维护工具，sync 涉及外部付费 API 和凭证，
      仍然只走 CLI）。

### 实现细节

- **模型选择走过一次弯路**：一开始用了通用英文模型
  `sentence-transformers/all-MiniLM-L6-v2` + `cross-encoder/ms-marco-MiniLM-L-6-v2`，
  端到端测试时发现中文查询"容器和主机之间网络不通怎么办"检索不到语义上明显相关的
  "WSL2 网络配置排查"笔记——这两个模型主要是英文语料训练的，中文语义表示质量不够。
  换成中文优化的 `BAAI/bge-small-zh-v1.5`（embedding）+ `BAAI/bge-reranker-base`
  （rerank）后，同一批测试查询（含之前失败的那条）全部正确命中。Vault 内容基本是中文，
  这个选择比"通用/知名"更重要。
- **索引会在换模型后自动整体重建**：`.knowledge/embeddings.json` 顶层记录了
  `model` 字段，`kb index` 发现存的模型名和当前 `EMBED_MODEL_NAME` 不一致就整体作废
  重算，不会把"用旧模型算的向量"误当成"内容没变、还能用"（不同模型的向量不在同一个
  语义空间，直接比较毫无意义）。
- **依赖是可选的**：`sentence-transformers`（含 torch）和 `mcp` 都放进
  `pyproject.toml` 的 `[project.optional-dependencies]`（`semantic` / `mcp` /
  `all`），core `kb` 命令（search/read/related/capture/curate/validate/status/sync）
  完全不需要装这些重依赖。
- **端到端验证**：用三篇主题明显不同的临时测试笔记（PID / WSL2 网络 / 状态观测器）测试过
  `kb index`、`kb semantic-search`（含/不含 `--rerank`）、以及通过真实 MCP stdio 协议
  （官方 `mcp` client SDK 起子进程连接）依次调用全部 6 个工具，确认 tool listing、
  `kb_status`、`kb_semantic_search`、`kb_search`、`kb_related`、`kb_capture` 都正常工作。
  测试完已删除全部临时笔记、Candidate 和 `.knowledge/embeddings.json`，Vault 恢复到
  0 篇正式笔记的状态，未在 Vault 或代码仓库留痕。

目标终态（已实现到这一步）：

```text
Codex / Claude Code / Gemini CLI / OpenCode / 其他 Agent
       │
       ▼
Knowledge MCP Server  ← kb/mcp_server.py（stdio）
       │
       ▼
Knowledge Engine  ← kb CLI（search/semantic-search/read/related/capture/...）
       │
       ▼
Markdown Vault
       │
   ┌───┴────┐
   ▼        ▼
Obsidian   Notion
```

## 用法

```bash
cd /home/eric/Personal/KnowledgeEngine
source .venv/bin/activate            # venv 已建好，装了 semantic + mcp 两个 extra
kb index                             # 建 / 增量更新语义索引（首次会下载模型，之后离线）
kb semantic-search "自然语言问题" --top-k 5 [--rerank]
kb mcp-server                        # 启动 MCP Server（stdio），供 Claude Code/Codex 等接入
```

把 `kb mcp-server` 接入 Claude Code / Codex 的 MCP 配置时，`command` 指向
`/home/eric/Personal/KnowledgeEngine/.venv/bin/kb`，`args` 是 `["mcp-server"]`，并设置
环境变量 `KB_VAULT_PATH`——具体的 `mcpServers`/`hooks.json` 配置留给需要接入的那次任务去做
（本阶段只交付了 Server 本身，还没有把它注册进任何一个 Agent 的配置里）。

## 明确没做的事（本阶段范围之外）

- 没有把 `kb mcp-server` 注册进 `~/.claude` 或 `~/.codex` 的配置——服务器建好了，但接入哪个
  Agent、以什么权限接入，需要单独决定。
- 没有做「Agent 会话结束自动化」（Plan.md 里已经写了设计，还没建 `Plans/07-*.md` 执行）。
- 没有引入图片/多模态 embedding，只对文本（标题+正文）编码。
- 触发升级到本阶段的"标准"（知识规模明显增长）不再适用——是用户主动要求提前做的，不是
  规模驱动的；如果未来出现真正的规模问题（几千篇笔记、线性扫描明显变慢），再考虑升级
  存储层（sqlite-vec/FAISS 等），不是现在。

## 来源

Plan.md 章节：「后续扩展」。
