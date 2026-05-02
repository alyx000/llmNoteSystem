---
title: 用 agent 攻击系统作为质量度量
type: concept
tags: [engineering, ai-tools, testing]
created: 2026-05-02
updated: 2026-05-02
status: active
summary: 通过组织 agent 对系统发起对抗性测试，评估系统能否承受作为质量指标
---

# 用 agent 攻击系统作为质量度量

> 由 `/ingest` 从 log 心得（2026-05-02）升格而来。

## 命题

传统质量度量（test coverage、code review 评分、静态分析）在 agent 时代不够。提议：

> **组织一组 agent 对系统发起对抗性测试**——功能 fuzz、API 滥用、并发竞争、错误注入、边缘输入构造——看系统能否承受。系统的"agent 攻击韧性"作为新的质量指标。

为什么这是个有趣的度量：

- agent 比人能用更多种方式滥用一个接口
- 度量是**结果导向**的（系统挂了/没挂），不是过程导向的（行覆盖率）
- 可以对**已上线系统**持续跑，不只是开发期

## 与传统方法的关系

*（待补）*

- vs **chaos engineering**：chaos 模拟基础设施故障；agent 攻击模拟功能层滥用
- vs **fuzzing**：fuzzing 通常无目标随机；agent 可有目标地构造攻击路径
- vs **red team**：red team 是人；agent 攻击成本更低、覆盖更广，但创造性可能不如人

## 实施考量

*（待补，未来值得展开的问题：）*

- agent 编排：单 agent vs 多 agent 协作
- 测试场景生成：prompt 工程 vs 让 agent 自主探索
- 判定标准：什么算"系统没承受住"
- 与 CI/CD 集成的方式
- 成本控制：每次跑一组 agent 攻击的 token 成本

## 在 wiki 中的连接

- [[agent-engineer-quality]]（这是它的"质量度量"维度）
- 相关 source：*（待加 — Karpathy / Anthropic 关于 agent 评估的文章；chaos engineering 经典文献）*
