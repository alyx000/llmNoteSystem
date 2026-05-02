---
description: 把 distill 中的 concept_candidates 升格成 concepts/<slug>.md（原子操作，含全库 wikilink 替换）
argument-hint: <candidate-slug>
---

# /promote

把一个反复在多份 distill 中出现的概念候选 slug，升格成 `concepts/<slug>.md` 的实际页面。**原子操作**——要么全部成功，要么全部回滚。

## 输入

`$ARGUMENTS` 是候选 slug（按 §3 命名规范）。通常由 `/lint` 第 10 项推荐而来。

## 前置检查

1. **校验 slug**：按 §3 命名规范（仅 `[a-z0-9-]`、不连续 `--`、长度 ≤ 60）
2. **碰撞检查**：`concepts/<slug>.md` 已存在 → 报错"已存在，无需升格；如要更新内容请直接编辑"
3. **支持度检查**：扫所有 `notes/distill/*.md` frontmatter `concept_candidates`，统计 `<slug>` 出现的页数。**< 3** → 警告用户"仅 N 页提到该候选，是否仍要升格"。≥ 3 → 直接进行

## 步骤

### 1. 创建 concepts/<slug>.md

```markdown
---
title: <slug 的中文显示标题，由用户输入或 LLM 提议>
type: concept
tags: [<从相关 distill 页继承的领域 tag>]
created: YYYY-MM-DD  # 今天
updated: YYYY-MM-DD  # 今天
status: active
summary: <一行 ≤ 80 字概括，可由 LLM 基于相关 distill 提议>
---

# <标题>

> 由 `/promote <slug>` 从 N 份 distill 中升格而来：
> [[<distill-1-slug>]] [[<distill-2-slug>]] ...

## 定义
<待补>

## 关键性质
<待补>

## 在 wiki 中的连接
<列出引用此概念的 entity / 其他 concept / distill>
```

详细内容留空（"待补"）——升格只是建立"锚点页"，深入内容由后续手动或 `/ingest` 补。

### 2. 全库扫描 distill 页，移除 candidate

对每个 `notes/distill/*.md`：
- 读 frontmatter `concept_candidates: [...]`
- 移除 `<slug>` 元素
- 如果移除后列表为空，**保留** `concept_candidates: []`（不删字段，明确表达"已处理"）
- 写回（更新 `updated: YYYY-MM-DD`）

### 3. 全库正文扫描，首次出现处替换为 wikilink

对每个**包含该 slug 候选的 distill 页**（即第 2 步处理过的页）：
- 在正文中查找该 slug 对应的中文/英文表述（用户在升格时提供"表述变体"清单，例如 `["AI-augmented refactoring", "AI 增强重构", "AI augmented refactoring"]`）
- **按文件 mtime 升序遍历**这些 distill 页
- 在每页中查找**首次出现**的位置（按 byte offset，不论变体）
- 替换为 `[[<slug>|原表述]]`（保留原显示文本作为 alias）
- **仅替换首次出现，后续不动**——避免暴力 replace-all 破坏行文
- 如果某页找不到任何变体（例如表述与 slug 字面差异太大），**跳过该页**并在 plan 中告知用户"该页未找到表述变体，请手动加 wikilink"

### 4. 更新 index.md

在 `## concepts` section 下追加一行：
```markdown
- [[concepts/<slug>]] — <summary>
```

### 5. 列出全部变动 diff，等用户 yes

```
将创建：concepts/<slug>.md
将修改 distill 页 (N 个)：
  - notes/distill/<a>.md
    - frontmatter: concept_candidates 移除 <slug>
    - 正文: line 42 "AI 增强重构" → "[[<slug>|AI 增强重构]]"
  - ...
将修改：index.md（追加 1 行）
```

### 6. 写入

按 plan 严格执行。失败任一步 → 回滚（恢复所有已写文件到操作前状态）。

### 7. log

`log.md` append：`## [YYYY-MM-DD] promote <slug> — 从 N 份 distill 升格`

## 不变量

- **绝不动旧 distill 页正文除"首次出现"那一处替换之外的任何字符**
- frontmatter 修改严格限于 `concept_candidates` 移除元素 + `updated` 更新
- 失败必须回滚（用 git stash 或事先备份）
- 永不修改 `entities/`、`sources/`、`notes/decisions/`、`notes/pitfalls/`、`notes/career-log/`、`notes/lint-reports/`
