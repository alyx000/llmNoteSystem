# 个人 Wiki Schema（CLAUDE.md / AGENTS.md）

> 这是 alyx 个人知识库的 schema 文件。**`AGENTS.md` 是本文件的 symlink**——Claude Code、Codex、其他 agent 工具读到的内容完全一致。修改本文件 = 同步修改两处。

本文件定义：目录结构、命名规范、frontmatter 协议、wikilink 解析规则、6 个 slash command 的语义、自检（lint）流程、升格规则、Obsidian Web Clipper 集成约定。任何写入本仓库的 LLM 都必须遵守这些约定。

---

## 1. 目的与四类内容

本仓库承载四类知识：

1. **工程经验与技术判断**（架构决策、踩坑记录）
2. **AI 工具使用心得**（哪个工具适合什么任务）
3. **职业转型路径**（思考、目标、里程碑）
4. **学习输入提炼**（书、文章、课程、播客、视频）

设计哲学（来自 Karpathy gist 442a6bf555914893e9891c11519de94f）：把传统 RAG 的"查询时检索"改为"LLM 增量维护一个持久、复合的 markdown 仓库"。维护成本由 LLM 承担，人只负责判断与触发。

---

## 2. 顶层目录结构

```
note/
├── entities/         # 具体实例：工具、人、项目
│   ├── tools/        # AI 工具、开发工具：每工具一页
│   ├── people/       # 共事过的人、思想影响者
│   └── projects/     # 你做过 / 在做的项目
├── concepts/         # 抽象概念、方法论；平铺，可按需加 subdir
├── notes/
│   ├── decisions/    # ADR 风格架构决策
│   ├── pitfalls/     # 踩坑记录
│   ├── distill/      # 学习材料的提炼笔记（每页 ↔ 一份 source）
│   ├── career-log/   # 职业转型时间线思考
│   └── lint-reports/ # /lint 自动生成的自检报告
├── sources/          # 原始材料（不存二进制，存 URL + 元数据 + 可选转录）
│   ├── articles/     # Web Clipper 默认落点
│   ├── books/
│   ├── courses/
│   ├── podcasts/     # 音频
│   └── videos/       # 视频（YouTube / 课程录像 / 演讲）
├── CLAUDE.md         # 本文件（schema）
├── AGENTS.md         # → CLAUDE.md 的 symlink
├── index.md          # 全站目录（按 type 分组，每页一行）
├── log.md            # 时间线 append-only
└── .claude/
    ├── settings.json # SessionStart / Stop hooks
    └── commands/     # slash command 定义
        ├── ingest.md
        ├── distill.md
        ├── wiki-log.md
        ├── lint.md
        ├── promote.md
        └── rename.md
```

**领域用 tag 表达，不用目录**——`#engineering` `#ai-tools` `#career` `#learning`。同一页可有多个领域 tag。

---

## 3. 命名规范

### Slug 命名（强约束，`/lint` 第 11 项校验）

- **只允许**：ASCII 小写字母 `a-z`、数字 `0-9`、连字符 `-`（U+002D HYPHEN-MINUS）
- **禁止**：下划线 `_`、em-dash `—` (U+2014)、en-dash `–` (U+2013)、空格、任何非 ASCII 字符
- 多词用单个 `-` 连接，**不允许连续 `--`**
- 长度 ≤ 60 字符

### 各类页文件名

| 类型 | 路径模板 | 示例 |
|---|---|---|
| 实体 | `entities/<type>/<slug>.md` | `entities/tools/cursor.md` |
| 概念 | `concepts/<slug>.md` | `concepts/ai-augmented-refactoring.md` |
| 决策 | `notes/decisions/YYYY-MM-DD-<slug>.md` | `notes/decisions/2026-05-01-adopt-bun.md` |
| 踩坑 | `notes/pitfalls/<symptom-slug>.md` | `notes/pitfalls/postgres-replica-lag.md` |
| 提炼 | `notes/distill/<source-slug>.md` | `notes/distill/karpathy-llm-wiki.md` |
| 时间线 | `notes/career-log/YYYY-MM-DD-<slug>.md` | `notes/career-log/2026-05-01-pivot-thoughts.md` |
| 源 | `sources/<media>/<slug>.md` | `sources/articles/karpathy-llm-wiki.md` |

decision/career-log 保留日期前缀方便 supersede；其他按主题命名便于 grep。

---

## 4. Frontmatter 协议

### 通用最小集（所有页必填，`/lint` 第 9 项校验）

```yaml
---
title: 显示标题（中文）
type: entity | concept | note | source
tags: [engineering, ai-tools, ...]
created: YYYY-MM-DD
updated: YYYY-MM-DD
status: active | archived         # 默认 active；archive 是软退役，跳过 orphan 检查
---
```

### 可选通用字段

```yaml
summary: 一行 ≤ 80 字摘要，用于 index.md 自动生成
```

### source 类追加字段

```yaml
source_url: https://...
author: ...
media_type: article | book | course | podcast | video
status: raw | distilled | archived   # source 类取值域更宽
distilled_at: YYYY-MM-DD             # status 变 distilled 时填
duration: "1h23m"                    # 音视频用，可选
transcript_status: none | partial | full | external   # 音视频用，可选
transcript_path: ./transcripts/<slug>.md               # 可选
```

### distill 类追加字段

```yaml
source: [[<source-slug>]]            # 必填：指向被提炼的 source
concept_candidates: [<slug-1>, <slug-2>, ...]   # 可选：从正文抽取的、未来可升格成 concept 的候选 slug
```

### 音视频质量警告

⚠️ 当 `transcript_status: none` 时，`/distill` 只能产出"摘要级"提炼，质量受 URL 长期存活制约。**重要素材**（多次回看 / 在 entity/concept 中引用的）至少做 partial 转录，否则 URL 失效后该 source 即作废。

---

## 5. 通信协议（三端共用）

这是**协议级约束**，三端（Claude Code / Codex / Obsidian）必须按此解析与生成，否则 `index drift` / `log gaps` / orphan 检测失去 oracle。

### 5.1 `index.md` 行格式

```markdown
## entities
- [[entities/tools/<slug>]] — <一行 ≤ 80 字摘要>
- [[entities/people/<slug>]] — ...

## concepts
- [[concepts/<slug>]] — ...

## notes
- [[notes/decisions/YYYY-MM-DD-<slug>]] — ...

## sources
- [[sources/<type>/<slug>]] (raw|distilled) — ...
```

- 每页**恰好一行**，按 type 分组、组内按 `created` 降序
- 摘要 ≤ 80 字，从页 frontmatter `summary` 取（如无则取首段首句截断）
- 摘要中不含换行、不含 `|`（避免破坏未来表格化）

### 5.2 `log.md` 行格式

每条记录起一行新二级标题：

```markdown
## [YYYY-MM-DD] <action> [<slug>] — <一句话>
```

- `<action>` ∈ `{init, ingest, distill, log, lint, decision, pitfall, milestone, rename, promote, archive, delete}`
- `<slug>` 在 `log` action 时可省略；其他 action 必填
- `rename` 用 `<old-slug>→<new-slug>` 格式
- `promote` 用单个 slug（slug 本身不变，只是从 frontmatter `concept_candidates` 升格成 `concepts/<slug>.md`）
- 一句话**不换行、不含 markdown 链接**（用 slug 自己即足够，避免链接腐烂污染历史）

#### Action 语义与归属（消歧 + 写入者明确）

| action | 含义 | 由谁写入 log |
|---|---|---|
| `init` | 仓库初始化 | 初始化脚本（一次） |
| `ingest` | URL/source fan-out 跨页更新 | `/ingest` |
| `distill` | source → distill 页 | `/distill` |
| `log` | 闪念/无结构化页的记录 | `/wiki-log` |
| `lint` | lint 跑完 | `/lint` |
| `decision` | 创建/修改 `notes/decisions/...` | 创建者（用户 `/wiki-log` 或 `/ingest`） |
| `pitfall` | 创建/修改 `notes/pitfalls/...` | 同上 |
| `milestone` | 职业/项目里程碑 | 用户 `/wiki-log` |
| `rename` | 页面 slug 变更 | **`/rename`** 命令（mv 文件 + 全库 wikilink 更新 + index.md 行更新） |
| `promote` | distill 候选升格成 `concepts/` 实际页 | **`/promote`** 命令（创建 concept 页 + 全库 candidate 移除 + 首次出现处替换为 wikilink） |
| `archive` | 页面**保留但 status: archived**，不参与 lint orphan 检查 | 用户 `/wiki-log` |
| `delete` | 页面**从文件系统移除**（git 历史保留） | 用户 `/wiki-log` |

`archive` vs `delete` 必须分开：archive 是软退役（页面还在），delete 是硬移除（仅在确认完全无价值时用）。

### 5.3 反向链接 & wikilink 解析

distill ↔ source 用 `/distill` 创建的两固定章节（`## 来源` / `## 提炼`）。
其他 entity/concept 互引用 Obsidian 风格 wikilink，`/lint` 必须能稳定解析。

支持的三种 wikilink 形式：

| 形式 | 示例 | 含义 |
|---|---|---|
| 纯 slug | `[[cursor]]` | 链接到 slug=cursor 的页 |
| 带 alias | `[[cursor\|游标编辑器]]` | 同上，显示文本不同 |
| 带 heading | `[[cursor#定价]]` | 链接到 cursor 页的 "定价" 二级标题 |

#### 转义规则

- 标题或正文若需字面 `]]`，写成 `\]\]`（反斜杠转义）。`/lint` 解析器遇到 `\]\]` 不视为 wikilink 终止符。
- slug 中**禁止**出现 `[`、`]`、`|`、`#`、`\`（wikilink 元字符）
- alias 中允许任意字符（含中文标点），直到遇到未转义的 `]]`

#### Parser 扫描范围（实现者必守，否则文档代码示例会被误报）

- **跳过** fenced code block：```` ``` ````—```` ``` ```` 和 `~~~`—`~~~` 之间的所有内容
- **跳过** inline code：`` ` ``—`` ` `` 之间
- **跳过** YAML frontmatter：首尾两条 `---` 之间
- **跳过** HTML comment：`<!--`—`-->` 之间
- **包含** 普通正文、表格单元格、列表项、blockquote

---

## 6. Slash Commands

每个命令的详细指令在 `.claude/commands/<name>.md`。本节给出**契约级**摘要——LLM 执行命令时必须严格遵守这里的语义边界。

### `/ingest <url-or-path-or-log-ref>`

Karpathy 风格 fan-out：把一份输入摊开到知识图，可能触发多个页面的创建/更新。**列出 plan 让用户确认 → 写入 → log**。

输入支持三种类型：

| 类型 | 示例 | 行为 |
|---|---|---|
| **URL** | `/ingest https://...` | 拉取 → 在 `sources/<media>/` 创建 source 页（status: raw）→ fan-out 到 entities/concepts/index |
| **本地 source 路径** | `/ingest sources/articles/foo.md` | 直接读 → fan-out（不重建 source 页） |
| **log 心得引用**（无 source） | `/ingest log:2026-05-02` 或 `/ingest log:<slug>` | 读 `log.md` 中对应日期/slug 的条目作为输入 → **跳过创建 source 页** → 直接 fan-out 到 entities/concepts/index |

第三种情形的存在原因：很多有价值的 fan-out 起点是**用户自己的心得**（通过 `/wiki-log` 速记到 log.md），不是外部材料。这种场景没有 source 页可言，强制创建会污染 `sources/`（sources 的语义是"原始外部材料"）。

`log.md` append：`## [YYYY-MM-DD] ingest <slug> — <一句话>`（slug 取本次 fan-out 的主要 concept slug）

### `/distill <source-slug>`

**精确 2 个内容文件变动**（log.md append 是共享副作用，不计入）：

1. 新建 `notes/distill/<slug>.md`：frontmatter 含 `source: [[<source-slug>]]`，正文末尾固定章节 `## 来源\n- [[<source-slug>]]`
2. 修改 `sources/.../<slug>.md`：`status: distilled`、`distilled_at: YYYY-MM-DD`、末尾固定章节 `## 提炼\n- [[<distill-slug>]]`

**绝不动 entities/concepts/index**。写入前列两文件 diff 等用户 yes。

**必须输出概念候选建议**（防止 `concept_candidates` 长期空白、`promote` 沦为死代码）：

- distill 完成正文后，提议 ≤ 5 个概念候选 slug（来自正文反复出现的名词性短语）
- **碰撞处理**：每个候选先与 `concepts/` 现有页比对——
  - 已存在 → 标注"已有页面，建议直接 wikilink"，**不写入** `concept_candidates`
  - 不存在 → 用户 accept / skip / rename 后写入 `concept_candidates`

`log.md` append：`## [YYYY-MM-DD] distill <slug>`

### `/wiki-log <内容>`

在 `log.md` append 一条带日期前缀的记录。可选追问"是否要 ingest 成结构化页"。

格式按 §5.2 严格遵守。

### `/lint [--slow]`

跑 §7 的自检流程，写报告到 `notes/lint-reports/YYYY-MM-DD-lint.md`。

- 默认 fast（10 项纯文本检查，预期 < 2 秒，500 页规模）
- `--slow` 追加 contradictions 检查（需 LLM 调用）

`log.md` append：`## [YYYY-MM-DD] lint — N issues across M categories`

### `/promote <candidate-slug>`

把 distill 中的 `concept_candidates` 升格成 `concepts/<slug>.md`。原子操作：

1. 创建 `concepts/<slug>.md`（基础 frontmatter + 简短描述，详细内容留空待补）
2. 全库扫描所有 distill 页 frontmatter `concept_candidates`，移除 `<slug>`
3. 全库正文扫描，把"该 slug 在表述中**首次出现**的位置"替换为 wikilink `[[<slug>]]`（仅首次，不暴力 replace-all；遍历顺序按文件 mtime 升序）
4. 列出所有变动文件 diff，等用户 yes
5. `log.md` append：`## [YYYY-MM-DD] promote <slug>`

### `/rename <old-slug> <new-slug>`

原子改名。任何手动 mv 会破坏全库 backlink，必须经此命令：

1. 文件 mv：所有 `<old-slug>.md` → `<new-slug>.md`（含 distill ↔ source 配对页：默认提示用户"是否同时重命名配对页"）
2. 全库 wikilink 更新：`[[<old>]]` `[[<old>|...]]` `[[<old>#...]]` 全部改为 `<new>`
3. 更新 `index.md` 对应行
4. 列出变动 diff，等用户 yes
5. `log.md` append：`## [YYYY-MM-DD] rename <old>→<new>`

---

## 7. 自检（`/lint`）流程

所有项**输出候选报告，不自动改文件**。报告写到 `notes/lint-reports/YYYY-MM-DD-lint.md`，每项分组列出。

### Fast 检查（默认，预期 < 2 秒 / 500 页）

1. **stale raw sources**：`sources/**` 中 `status: raw` 且 `created` > 14 天 → 建议 `/distill`
2. **orphan pages**：用 wikilink 解析器（识别 §5.3 三种形式）找零入链页，排除 `index.md` / `log.md` / `notes/lint-reports/*` / `tags: [orphan-ok]` / `status: archived`
3. **broken wikilinks**：`[[...]]` 目标 slug 在磁盘找不到
4. **index drift**：磁盘上有但 `index.md` 未登记的页 / 反之
5. **tag hygiene**：只出现 1 次的 tag 列出（拼写 / 合并候选）
6. **log gaps**：近 7 天有改动但 `log.md` 未提及的页
7. *（slow，见下）*
8. **schema symlink 完整性**：`test -L AGENTS.md && [ "$(readlink AGENTS.md)" = "CLAUDE.md" ]`，断了报警
9. **frontmatter 必填字段缺失**：按 §4 校验
10. **升格候选**：聚合所有 distill 页 `concept_candidates`，对未在 `concepts/` 下存在的 slug 计数，≥ 3 次的列出建议升格
11. **slug 违规**：扫描所有页面文件名（去 `.md`），违反 §3 命名规范的列出

### Slow 检查（仅 `/lint --slow` 触发）

7. **contradiction candidates**（候选报告，不断言）：成本控制——
   - 仅在近 7 天改动的页中筛选
   - 用 tag overlap ≥ 2 + token-level Jaccard ≥ 0.3 缩成候选页对池
   - 池大小**硬上限 5 对/次**（按 tag overlap 降序），超出截断
   - 对每对让 LLM 输出"可能矛盾的句子片段 + 置信度"，**不做断言**

### lint 扫描范围排除

- `CLAUDE.md` / `AGENTS.md`：协议本身，不参与 frontmatter / wikilink / orphan 检查
- `.claude/**`：harness 配置目录
- `notes/lint-reports/**`：lint 自身产物
- `.git/**`、`.obsidian/**`
- 任何 `status: archived` 的页：跳过 orphan 检查

### 周期

默认手动 `/lint`。如要每周自动跑：在 Claude Code 里执行 `/schedule`，配置「每周日 22:00 在 /Users/alyx/note 跑 /lint --slow，把摘要发给我」。本仓库初始化时不替你启用 schedule。

---

## 8. 升格规则（entity / concept / distill 边界）

避免重复建页的硬规则：

- **`entities/<type>/<slug>.md`** 描述**具体实例**（"我对 Cursor 的看法" / "alyx 在 X 公司"）。属性 + 个人评价。
- **`concepts/<slug>.md`** 描述**抽象概念或方法论**（"AI-augmented refactoring" / "ADR 决策框架"）。可被多个 entity 引用。
- **`notes/distill/<slug>.md`** 是**对单一 source 的提炼**——只引用该 source，不独立存在。

### 升格触发（由 `/lint` 提示，由用户/`/promote` 执行）

1. ≥ 3 个 distill 页的 `concept_candidates` 含同一未存在 slug → `/lint` 建议 `/promote <slug>`
2. 某 entity 页内的论述被另外 ≥ 2 个页面引用同一段 → 建议把那段抽出成 `concepts/<...>.md`，原 entity 页改为引用
3. **distill 不升格**：distill 永远绑定单 source；要抽象的内容应升到 concept

### "概念词"的可执行定义

避免分词、大小写折叠、中英文混合等不可执行歧义，**采用显式标注机制**：

- distill 页 frontmatter 加 `concept_candidates: [<slug>, ...]` 字段
- 每个候选必须是合法 slug（按 §3 命名规范）
- `/lint` 第 10 项**只统计这些显式 slug**，不做自然语言分词
- `/distill` 命令**强制**在完成提炼时建议候选

---

## 9. Hooks（已写入 `.claude/settings.json`）

- **SessionStart**：在 `/Users/alyx/note` 启会话时，echo `index.md` + `log.md tail -30` 作为额外上下文，让 LLM 一上来就知道知识库现状
- **Stop**：会话结束时输出一句提示："💡 本次会话有值得沉淀的内容吗？可用 /wiki-log 或 /ingest"

**hooks 只读**——所有写入必须经显式 slash command，不在 hooks 里做自动写入，避免 LLM 静默改动 wiki。

---

## 10. Obsidian Web Clipper 集成

### 浏览器侧设置

| 字段 | 值 |
|---|---|
| Vault | `note` |
| Folder | `sources/articles` |
| Filename | `{{date}}-{{title\|slug}}` |

### Template

```yaml
---
title: {{title}}
type: source
tags: [clipped]
created: {{date}}
updated: {{date}}
source_url: {{url}}
author: {{author}}
media_type: article
status: raw
---

{{content}}
```

> **关于"不预置 template"原则**：本仓库不预置任何**页面 template**（`entities/<type>/<slug>.md` 类的样板页），但 Web Clipper Template 是 **Clipper 与通信协议的绑定层**——它的唯一职责是让外部进入 wiki 的 source 页满足 frontmatter 必填字段。两件事不冲突。

所有 Web Clipper 进来的页都视为待 `/distill`，`/lint` 第 1 项会催（14 天阈值）。

---

## 11. 同步与备份

- **本地**：`/Users/alyx/note` 是 git 仓库，所有写入立即可 commit
- **远程**：用户在 GitHub 建私有 repo 后，跑 `git remote add origin <url> && git push -u origin master`
- **跨机**：在另一台机器 `git clone <url>`，Obsidian "Open folder as vault" 即可
- **不用** Obsidian Sync / iCloud（避免与 git 冲突）

---

## 12. 不变量（写入新页时反复检查）

1. 文件名必须满足 §3 slug 命名规范
2. frontmatter 必含 §4 通用最小集
3. wikilink 必须按 §5.3 三种形式之一书写
4. 所有写入操作必须 append 一条对应 action 的 log（按 §5.2）
5. `/distill` 永远只动 2 个内容文件
6. `/promote` 永远不动 distill 之外的旧页正文（除"首次出现"那一处替换）
7. `archive` 不删文件，`delete` 才删
8. `AGENTS.md` 必须始终是 `CLAUDE.md` 的相对路径 symlink

违反上述任一条 = bug，必须修。
