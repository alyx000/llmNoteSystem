---
description: 把一份 source 提炼成 distill 笔记（精确 2 文件操作 + 强制输出概念候选）
argument-hint: <source-slug>
---

# /distill

**精确两文件**操作：把 `sources/.../<slug>.md` 提炼成 `notes/distill/<slug>.md` + 改原 source 状态。**绝不动 entities/concepts/index**——那是 `/ingest` 的职责。

## 输入

`$ARGUMENTS` 是 source 的 slug（不含 `.md`）。例如 `karpathy-llm-wiki`。

## 步骤

1. **定位 source**：在 `sources/articles/`、`sources/books/`、`sources/courses/`、`sources/podcasts/`、`sources/videos/` 中找 `<slug>.md`。找不到 → 报错并建议先 `/ingest`
2. **读 source 全文**（含 frontmatter）
3. **生成 distill 内容**（按以下结构）：
   ```markdown
   ---
   title: <source 的中文标题>
   type: note
   tags: [distill, <source 的领域 tag>]
   created: YYYY-MM-DD  # 今天
   updated: YYYY-MM-DD  # 今天
   status: active
   summary: <一行 ≤ 80 字概括>
   source: [[<source-slug>]]
   concept_candidates: []  # 第 5 步会填
   ---
   
   # <标题>
   
   ## 核心论点
   <2-4 句话，材料的中心观点>
   
   ## 关键洞察
   - <洞察 1>
   - <洞察 2>
   - ...
   
   ## 与已有知识的连接
   <这份材料如何呼应/挑战 wiki 中已有的 entity/concept；用 [[wikilink]] 引用>
   
   ## 个人评价
   <你的判断、保留意见、何时该用何时不该用>
   
   ## 来源
   - [[<source-slug>]]
   ```
   音视频且 `transcript_status: none` 时：仅产出"摘要级"提炼，并在末尾加一行 ⚠️ 提示用户该 distill 质量受 URL 存活制约
4. **构造修改后的 source 内容**：在原 source 文件末尾追加：
   ```markdown
   
   ## 提炼
   - [[<distill-slug>]]
   ```
   并修改 frontmatter：`status: distilled`、`distilled_at: YYYY-MM-DD`、`updated: YYYY-MM-DD`
5. **生成概念候选建议**（强制步骤，不可跳过）：
   - 从 distill 正文中识别 ≤ 5 个反复出现的名词性短语
   - 每个候选生成合法 slug（按 §3 命名规范：ASCII 小写 + 数字 + 连字符）
   - **碰撞检查**：每个候选 slug 在 `concepts/` 目录下查是否已存在
     - 已存在 → 在建议列表里标"已有页面 [[<slug>]]，请直接 wikilink 到 distill 正文里"，**不写入 concept_candidates**
     - 不存在 → 列给用户，问 accept/skip/rename
   - 用户 accept 的 slug 写入 distill frontmatter `concept_candidates: [...]`
6. **展示两文件 diff** 给用户：
   ```
   将创建：notes/distill/<slug>.md
   将修改：sources/<media>/<slug>.md
   
   <diff 内容>
   
   概念候选建议：
   - [accept] ai-augmented-refactoring (新)
   - [skip] prompt-caching (已有 [[concepts/prompt-caching]]，请在正文 wikilink 引用)
   ```
7. **等用户 yes 后写入两文件**
8. **写入 log**：`log.md` append `## [YYYY-MM-DD] distill <slug> — <一句话概括>`

## 不变量（违反 = bug）

- **恰好 2 个内容文件变动**（log.md append 不计入；它是所有命令的共享副作用）
- 不动 `entities/`、`concepts/`、`index.md`、其他 distill 页
- distill 页 frontmatter 必含 `source: [[<source-slug>]]`
- source 页 status 必须从 `raw`/`distilled` 更新（已是 distilled 时仅追加章节）
- 概念候选**必须**输出（即使 0 个候选也要明确告知用户"无候选"）
