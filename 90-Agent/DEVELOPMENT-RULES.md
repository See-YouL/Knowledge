# 开发规则：Knowledge Curator

Knowledge Curator 是知识库的守门人角色——不产生新知识，负责把 Candidate 变成高质量的
正式知识，并保持整个 Vault 长期整洁、不重复、不冲突。人和 Agent 都可以承担这个角色。

Git 工作流规则见下方一节；分阶段执行记录见
[`Plans/03-Git-Workflow.md`](../Plans/03-Git-Workflow.md)。

## 职责清单

处理 `00-Inbox/Agent-Candidates/` 里的每一条 Candidate 时，Curator 应依次检查：

- [ ] **Candidate 审核**：内容是否真的有长期价值？是否已经违反
      [`KNOWLEDGE-RULES.md`](KNOWLEDGE-RULES.md) 中"禁止直接保存"的清单？
- [ ] **去重**：Vault 里是否已经有覆盖同一概念的笔记（用 `rg` 按标题/关键词搜索）？
- [ ] **合并**：如果是对已有笔记的补充或局部修正，合并进已有笔记而不是新建。
- [ ] **分类**：确认 `type`、`domain`/`subdomain` 是否准确（参考 [`TAXONOMY.md`](TAXONOMY.md)）。
- [ ] **修正 YAML**：确认 frontmatter 符合
      [`Templates/Frontmatter-Spec.md`](Templates/Frontmatter-Spec.md)，`kb_id` 已生成
      且唯一，`status`/`confidence` 取值合理。
- [ ] **建立 WikiLink**：在"相关知识"一节中，把能关联到的已有笔记用 `[[...]]` 链接起来。
- [ ] **检查冲突知识**：新内容是否和已有 `verified` 笔记矛盾？矛盾则不要自行覆盖，保留
      双方说法并标记为需要人工审核（例如在笔记中加一句"⚠ 与 [[XXX]] 存在冲突，待确认"）。
- [ ] **检查 Broken Links**：确认笔记中引用的 `[[WikiLink]]` 目标真实存在或值得先创建为
      占位笔记。
- [ ] **检查重复笔记**：定期扫描 Vault，发现标题相近、内容重叠的笔记应合并。
- [ ] **维护 Canonical Note**：确保一个概念只有一份权威笔记（见下）。

## 一个概念，一个 Canonical Note

不要为同一个概念反复新建变体笔记，例如：

```text
PLL.md
PLL基础.md
PLL总结.md
PLL新版.md
PLL-final.md
```

这类命名膨胀会让 Agent 和人都无法判断哪一份是权威版本。正确做法：**找到已有的那一份，
更新它**（保留其 `kb_id`，更新 `updated` 日期），而不是新建一份内容相近的笔记。

只有当新内容明确是另一个不同的概念（哪怕相关）时，才新建笔记，并在"相关知识"里用
WikiLink 互相关联，而不是把不同概念硬塞进同一篇笔记。

如果发现历史遗留的重复笔记，Curator 应该：合并成一份 Canonical Note（保留最合适的
`kb_id`），把其余的重定向说明或直接移入 `99-Archive/` 并标记 `status: deprecated`。

## Git 工作流规则

整个 Vault 用 Git 做版本控制，是知识出错时的安全网。

- **改动前后核对范围**：对 Vault 做修改前后，运行 `git status` / `git diff` 核对实际改动
  的文件和内容，确认没有意外触碰到计划外的笔记。
- **出错可恢复**：任何一次错误的批量修改，都可以通过 Git 历史（`git log`、`git diff`、
  `git checkout <commit> -- <path>`）找回旧版本，因此发现问题时优先想"恢复"而不是"手工
  重写"。
- **禁止未经明确授权的危险操作**：`git clean -f`、批量 `rm`、`git reset --hard`、
  `git push --force` 等具有破坏性或不可逆倾向的操作，未经用户明确授权不得执行。
- **提交时机**：只在用户明确要求时才 `git commit`；日常整理、批量修改后默认停在工作区，
  由用户查看 `git diff` 后决定是否提交。

### 什么算「大规模修改」

满足以下任一条件时，视为大规模修改，Agent 应先展示计划改动的文件范围（而不是直接执行），
等待人工确认后再动手：

- 一次改动涉及 **5 篇以上**正式笔记（`10~80` 目录下的 `.md` 文件）；
- 涉及目录结构调整（新增/删除/重命名 `00~90` 下的任意目录）；
- 涉及批量删除（无论文件数量）；
- 涉及修改 `90-Agent/` 下的规则文档本身。

单篇笔记的新建、更新、`00-Inbox/` 内的日常整理不算大规模修改，可以直接执行，但仍应在
完成后展示 `git diff --stat` 供用户查看。
