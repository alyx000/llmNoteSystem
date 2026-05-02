---
title: 深度投资个人 agent 工具链
type: concept
tags: [ai-tools, engineering]
created: 2026-05-02
updated: 2026-05-02
status: active
summary: 把自己的 agent 工具链当作长期资产持续打磨，而非临时拼凑
---

# 深度投资个人 agent 工具链

> 由 `/ingest` 从 log 心得（2026-05-02）升格而来。

## 定义

「深度投资」指：把 hooks、slash commands、自定义 agent、prompt 模式、知识库当成**长期资产**持续打磨；而非每次任务临时拼凑。

类比：和 dotfiles / 个人 vim 配置 / 个人脚本库的关系——好工程师会持续迭代这些，而不是每次新机器从零开始。

## 为什么重要

是 [[agent-engineer-quality]] 的关键能力维度之一。原因：

- agent 的输出质量很大程度上取决于上下文（prompt、工具、记忆）质量
- 工具链投资的回报随使用次数线性放大；临时拼凑则每次重付成本
- 个人化的工具链能编码"你独有的判断"，让 agent 反映你的偏好而非通用平均

## 实践

*（待补，可能形态：）*

- 配置 SessionStart hooks 自动加载相关上下文
- 写专属 slash commands 把常用工作流原子化
- 维护一份 agent toolchain 的个人 wiki（**像本仓库就是**）
- 给 agent 配持久 memory，让它记住你的偏好不用每次重申
- 给 codex / Claude Code 等不同 agent 设不同擅长领域，按任务路由

## 在 wiki 中的连接

- [[agent-engineer-quality]]
- 相关 entity：`entities/tools/` 下各 agent 工具的评测页（待积累）
