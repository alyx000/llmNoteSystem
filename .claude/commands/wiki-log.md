---
description: 在 log.md 追加一条带日期前缀的记录（闪念、archive、delete、milestone、decision 等）
argument-hint: [<action>] <内容>
---

# /wiki-log

在 `/Users/alyx/note/log.md` append 一条记录。这是闪念记录、手动 archive/delete、记录 milestone/decision/pitfall 等无结构化页面操作的统一入口。

## 输入

`$ARGUMENTS` 可以是：
- 单纯一句话 → 默认 action = `log`，slug 省略
- `<action> <slug> <一句话>` → 显式指定
- action 必须 ∈ `{log, milestone, decision, pitfall, archive, delete, rename}`（其他 action 由对应专属命令写入：init/ingest/distill/lint/promote/rename）

> 注：rename 也有专属 `/rename` 命令——这里只支持"已经手动 mv 完后补登记 log"的场景，但**强烈不推荐**，因为不会更新全库 wikilink。永远优先用 `/rename`。

## 步骤

1. 读今天日期
2. 解析 `$ARGUMENTS`：
   - 第一个 token 在 action 枚举中？→ 当 action 用，剩余按 `[<slug>] <内容>` 解析
   - 否则 → action = `log`，全部当 `<内容>`
3. 按 §5.2 格式构造行：`## [YYYY-MM-DD] <action> [<slug>] — <一句话>`
   - 一句话**不换行、不含 markdown 链接**（用 slug 自己即可）
4. 显示给用户确认：
   ```
   将 append 到 log.md：
   ## [2026-05-01] log — 今天发现 Cursor 1.5 的 background agent 模式速度提升明显
   ```
5. 用户 yes 后 append（不覆盖、不重排序、严格在文件末尾追加）
6. 如果 action ∈ `{archive, delete}`：
   - **archive**：还要修改对应页 frontmatter `status: archived`、`updated: YYYY-MM-DD`
   - **delete**：还要 `rm` 对应文件（git 历史保留），并问用户"是否同时删除配对页（如 distill/source 配对）"
7. 如果 action = `log` 且内容含明显的"工具/项目/概念"信号 → 追问"是否要 ingest 成结构化页"

## 不变量

- 永远 append，绝不重排已有 log 条目
- 一行一条，严格按 §5.2 格式
- archive 不删文件，delete 才删
