---
description: 跑 11 项自检（fast 默认；--slow 追加 LLM 矛盾检测），输出报告到 notes/lint-reports/
argument-hint: [--slow]
---

# /lint

跑全库自检。**所有项输出候选报告，不自动改文件**——lint 只发现问题，不修。

## 输入

`$ARGUMENTS` 可选：`--slow` 启用 LLM 矛盾检测（项 7）。默认 fast，不带 LLM 调用。

## 扫描范围

包含：`entities/**` `concepts/**` `notes/**`（除 `lint-reports/`）`sources/**` `index.md`

排除：`CLAUDE.md` / `AGENTS.md` / `.claude/**` / `notes/lint-reports/**` / `.git/**` / `.obsidian/**`

orphan 检查另外排除：`tags: [orphan-ok]` 的页 / `status: archived` 的页

## Fast 检查（默认 10 项，预期 < 2 秒 / 500 页）

### 1. stale raw sources
扫 `sources/**`，找 `status: raw` 且 `created` > 14 天前的页。列出 → 建议 `/distill`。

### 2. orphan pages
**实现 wikilink 解析器**（按 §5.3）：
- 跳过 fenced code block（```/~~~ 之间）、inline code（` 之间）、frontmatter（首尾 --- 之间）、HTML comment（<!-- --> 之间）
- 在剩余内容中匹配 `[[<内容>]]`，且 `\]\]` 不算终止符
- `<内容>` 拆解：`<slug>`、`<slug>|<alias>`、`<slug>#<heading>`
- slug 中不允许 `[]|#\`

收集所有 wikilink 目标 slug，计算每页的入链数。入链 = 0 的页列为 orphan（除上述排除外）。

### 3. broken wikilinks
对每个 wikilink 目标 slug，检查磁盘是否有 `<slug>.md`（在合法路径下：entities/_/, concepts/, notes/_/, sources/*/）。找不到 → 列出。

### 4. index drift
对比磁盘文件集 vs `index.md` 中登记的 slug 集（按 `[[<path>/<slug>]]` 解析）。
- 磁盘有 + index 无 → "未登记"
- index 有 + 磁盘无 → "孤儿登记"

### 5. tag hygiene
聚合所有页 frontmatter `tags`，统计每个 tag 的出现次数。出现 = 1 的列为"拼写或合并候选"。

### 6. log gaps
扫 git log（或文件 mtime）找近 7 天内有改动的页。对每页检查 `log.md` 中是否有以该 slug 结尾的条目。无 → "log 缺记"。

### 8. schema symlink 完整性
跑 `test -L AGENTS.md && [ "$(readlink AGENTS.md)" = "CLAUDE.md" ]`。退出码非 0 → "schema symlink 损坏"。

### 9. frontmatter 必填字段
对每页解析 YAML frontmatter，按 CLAUDE.md §4 校验：
- 通用必填：`title, type, tags, created, updated, status`
- source 类追加：`source_url, media_type`
- distill 类追加：`source`
缺 → 列出。

### 10. 升格候选
聚合所有 distill 页 frontmatter `concept_candidates` 字段，对每个 slug：
- 在 `concepts/` 下查是否已存在
- 已存在 → 跳过
- 不存在 → 计数
- 计数 ≥ 3 → 列为"建议 `/promote <slug>`"

### 11. slug 违规
扫所有 `.md` 文件名（去 `.md`），按 §3 命名规范校验：
- 仅允许 `[a-z0-9-]`
- 不连续 `--`
- 长度 ≤ 60
违反任一条 → 列出，并建议 `/rename <old> <new>`。

## Slow 检查（仅 --slow 触发）

### 7. contradiction candidates
- 筛选近 7 天改动的页
- 每对页计算 tag overlap（≥ 2）+ token-level Jaccard（≥ 0.3）
- 按 tag overlap 降序排，取**前 5 对**（硬上限）
- 对每对让 LLM 输出："可能矛盾的句子片段（A 页 vs B 页）+ 置信度"
- **不做断言**——只列候选给用户判断

## 输出

写到 `notes/lint-reports/YYYY-MM-DD-lint.md`：

```markdown
---
title: Lint Report YYYY-MM-DD
type: note
tags: [lint, generated]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active
---

# Lint Report YYYY-MM-DD

## 摘要
- Fast: N issues
- Slow: M candidate pairs (if --slow)
- 检查项分布: [项 1: N, 项 2: M, ...]

## 1. Stale raw sources
- ...

## 2. Orphan pages
- ...

(逐项列出，每项包含问题清单 + 建议命令)
```

并在 `log.md` append：
```
## [YYYY-MM-DD] lint — N issues across M categories
```

## 不变量

- **永不自动修文件**（即使是明显错误，也只列报告让用户决定）
- contradictions 永远只输出"候选片段 + 置信度"，不写"A 矛盾于 B"的断言
- 每次跑生成一份新 lint-report，不覆盖旧的（按日期命名）
- log 摘要必须明确写出 N 和 M
