---
title: /ingest 支持 log 心得作为输入
type: note
tags: [meta, schema, decision]
created: 2026-05-02
updated: 2026-05-02
status: active
summary: /ingest 输入除 URL/source 路径外，新增支持 log:<date-or-slug> 引用，跳过 source 页直接 fan-out
---

# /ingest 支持 log 心得作为输入

## 背景

2026-05-02 第一次实战使用本仓库时，触发场景：

1. 通过 `/wiki-log` 速记了一条心得（"如何衡量好的 agent 工程师..."）
2. 这条心得包含 3 个独立可升格的 concept
3. 想用 `/ingest` 做 fan-out 创建 3 个 concept 骨架
4. 但原 `/ingest` 规范要求输入是 URL 或 source 路径——心得既不是 URL 也没有外部 source

当时的 workaround 是手动跳过"创建 source 页"步骤，但这违反了"slash command 必须有清晰契约"的设计初衷。

## 决定

`/ingest` 输入语法扩展为支持 `log:<date>` 或 `log:<slug>` 形式，从 `log.md` 中找对应条目作为 fan-out 起点。这种情况**不创建 source 页**，直接进入"分析输入 → 列 plan → 写入 entities/concepts/index"流程。

## 后果

- ✅ 心得 → concept 路径有了正式入口，不再依赖手动 workaround
- ✅ `sources/` 目录的语义保持纯净（仅外部材料）
- ⚠️ `/ingest` 的输入分支多了一种，spec 复杂度+1（已用表格组织缓解）
- ⚠️ "log:<slug>" 形式依赖 log.md 中条目的 slug 字段——目前 log 行格式允许 slug 省略（action=log 时），所以"按 slug 查 log"在很多场景失败，只有"按日期查 log"可靠。这是一个待解决的子问题。

## 实施

- 改动文件：`CLAUDE.md` §6 `/ingest` 段（输入表 + 第三类型说明）；`.claude/commands/ingest.md`（输入表 + 步骤 1 第三分支）
- 不需要其他命令改动（`/wiki-log`、`/distill`、`/lint`、`/promote`、`/rename` 不受影响）

## 相关

- 触发心得：`log.md` 2026-05-02 那条 `log` action
- 由此 fan-out 创建的 concept：[[agent-engineer-quality]]、[[agent-toolchain-investment]]、[[agent-adversarial-quality-test]]
