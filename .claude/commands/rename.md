---
description: 原子改名页面 slug（mv 文件 + 全库 wikilink 更新 + index.md 更新）
argument-hint: <old-slug> <new-slug>
---

# /rename

把一个页面的 slug 从 `<old-slug>` 改为 `<new-slug>`。**任何手动 mv 都会破坏全库 backlink，必须经此命令**。

## 输入

`$ARGUMENTS` 形如：`old-slug new-slug`（用空格分隔）。

## 前置检查

1. **校验两个 slug**：都按 §3 命名规范
2. **找到旧文件**：在所有合法路径下查找 `<old-slug>.md`（entities/*/, concepts/, notes/*/, sources/*/）
   - 找到 0 个 → 报错"找不到 <old-slug>"
   - 找到 1 个 → 继续
   - 找到 ≥ 2 个 → 列出所有路径让用户选择（可能存在 distill ↔ source 同名配对）
3. **检查新名是否冲突**：`<new-slug>.md` 在任何合法路径下已存在 → 报错"<new-slug> 已存在，请换名或先处理冲突"
4. **配对检查**：如果旧文件是 source 或 distill，**默认提示用户**：
   ```
   <old-slug>.md 是 source/distill。检测到配对页 notes/distill/<old-slug>.md（或 sources/.../<old-slug>.md）。
   是否同时重命名配对页？(Y/n)
   ```
   - Y → 一并 rename（生成两组变动）
   - n → 仅 rename 当前页，配对页保持原 slug（用户需自行处理两边的 wikilink 一致性）

## 步骤

### 1. 文件 mv

按用户确认的范围（单页或配对）：
- `mv <old-path>/<old-slug>.md <old-path>/<new-slug>.md`

### 2. 全库 wikilink 更新

扫所有 `.md` 文件（含被 mv 后的新文件，因为可能引用自己），按以下规则替换：
- `[[<old-slug>]]` → `[[<new-slug>]]`
- `[[<old-slug>|<alias>]]` → `[[<new-slug>|<alias>]]`（alias 保留）
- `[[<old-slug>#<heading>]]` → `[[<new-slug>#<heading>]]`
- `[[<path>/<old-slug>]]` → `[[<path>/<new-slug>]]`（带路径前缀的形式）

**Parser 必须遵守 §5.3 扫描范围**：跳过 fenced code block / inline code / frontmatter / HTML comment。

每次替换后更新该页 frontmatter `updated: YYYY-MM-DD`。

### 3. 更新 index.md

在 `index.md` 中找到对应行，把 slug 部分替换。

### 4. 列出全部变动 diff

```
将 mv：
  - entities/tools/<old>.md → entities/tools/<new>.md
  - notes/distill/<old>.md → notes/distill/<new>.md (配对)

将更新 wikilink (M 个文件，N 处)：
  - notes/distill/<a>.md: 2 处
  - concepts/<b>.md: 1 处
  - ...

将更新 index.md：1 行
```

### 5. 等用户 yes 后写入

失败任一步 → 回滚。

### 6. log

`log.md` append：`## [YYYY-MM-DD] rename <old-slug>→<new-slug>`

如果配对页也 rename，**只写一条 log**（按主页 slug，并在一句话里说明"含配对页"）。

## 不变量

- 文件 mv 必须 atomic（要么全 mv 成功，要么不动）
- wikilink 替换不能误中代码块、inline code、frontmatter
- 替换后磁盘上**不能存在任何 `[[<old-slug>...]]` 形式的 wikilink**（除非在被排除的代码块内——这种情况是字面文档示例，应保留）
- 永不修改 `CLAUDE.md` / `AGENTS.md` / `.claude/**`
