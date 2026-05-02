---
description: 拉取 URL 或本地剪藏，做 Karpathy 风格的 fan-out 跨页更新（entities/concepts/sources/index）
argument-hint: <url-or-path>
---

# /ingest

执行 Karpathy 风格的 fan-out 操作：把一份新材料"摊开"到知识图，可能触发 10-15 个页面的创建/更新。**不是单页操作**——如果只想做单页提炼，用 `/distill`。

## 输入

`$ARGUMENTS` 是 URL（http/https）或本地路径（相对 `/Users/alyx/note` 或绝对）。

## 步骤

1. **判断输入类型 + 准备 source 页**：
   - 如果是 URL：拉取内容，生成 slug（按 `/Users/alyx/note/CLAUDE.md` §3 命名规范），创建 `sources/<media>/<slug>.md`，frontmatter 含 `source_url, media_type, status: raw, created, updated, title, type: source, tags`
   - 如果是已有 source 路径：直接读取
   - media_type 判断：URL 含 youtube/vimeo → video；含播客平台 → podcast；含 amazon book → book；其他默认 article
2. **读 schema**：读 `/Users/alyx/note/CLAUDE.md`，确认协议（特别是 §5、§8）
3. **读现状**：读 `/Users/alyx/note/index.md` 和 `log.md tail -50`，了解已有 entity/concept
4. **分析材料**：从 source 内容中识别——
   - 提到的具体工具/人/项目（候选 entities）
   - 提到的抽象概念/方法论（候选 concepts）
   - 是否需要新建 decision/pitfall（如果材料触发了新的判断）
5. **复用优先**：每个候选先与现有 entity/concept 比对，**优先复用**而非新建
6. **生成 plan**：列出该次 ingest 的全部文件变动——
   ```
   将创建：
     - sources/articles/<slug>.md (新)
     - entities/tools/cursor.md (新) [如果材料引入了未存在的工具]
   将更新：
     - concepts/ai-augmented-refactoring.md (追加引用 + 一段总结)
     - index.md (新增 N 行)
   ```
7. **等用户 yes 后写入**——不要直接写入！必须显式确认
8. **写入完成后**：在 `log.md` append `## [YYYY-MM-DD] ingest <slug> — <一句话>`，YYYY-MM-DD 用今天日期

## 不变量（违反 = bug）

- 所有新页文件名遵守 §3 slug 命名规范
- 所有新页 frontmatter 含 §4 通用最小集
- 所有 wikilink 按 §5.3 三种形式之一
- 不修改已有 entity/concept 的核心论述（只追加引用、补充段落、更新 updated 字段）
- 不动 `notes/distill/*`（distill 是 source 提炼层，不应被 ingest 覆盖；distill 由 `/distill` 维护）
- 不动 `CLAUDE.md` / `AGENTS.md` / `.claude/**`
