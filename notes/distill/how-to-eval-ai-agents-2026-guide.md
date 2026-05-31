---
title: "如何评估 AI Agent：2026 指南"
type: note
tags: [distill, engineering, ai-tools, learning]
created: 2026-05-31
updated: 2026-05-31
status: active
summary: "Agent eval 应从真实失败出发，用小而高信号的代码感知评测持续抬高下限。"
source: [[how-to-eval-ai-agents-2026-guide]]
concept_candidates:
  - floor-raising-evals
  - code-aware-agent-evals
  - production-error-analysis
  - golden-agent-cases
  - agent-eval-loop
---

# 如何评估 AI Agent：2026 指南

## 核心论点

这篇文章反对把 agent eval 做成抽象 benchmark 竞赛。对多数产品团队来说，更重要的不是“刷高平均分”，而是 floor raising：找出真实用户流程里最危险、最丢信任、最不该回归的失败模式，并把它们转成高信号 eval。

作者的 eval 方法本质是一条持续工程循环：先读真实 trace 做 error analysis，再提炼 recurring issue，补小而代表性的 golden case 或 code-aware offline eval，修复后继续用生产监控验证，而不是预先堆一个巨大的合成测试集。

## 关键洞察

- Benchmark maxxing 和 floor raising 是两种不同目标。前者适合能力展示，后者适合替代人工的产品可靠性。
- Floor raising 本质是 error analysis：读用户消息、agent 响应、工具调用和轨迹，找到最后一个成功步骤与第一个真实失败点。
- Golden cases 不需要一开始复杂。先选 5-10 个关键路径 case，只要这些 case 失败，就不应发布。
- Agent eval 必须 code-aware。只测 prompt 没意义，应该跑真实 agent 路径，断言输出、工具调用、文件变更、结构化数据或最终状态。
- 生产后的 eval 工作要随流量扩展：低流量读 raw logs，中等流量沉淀 issues，高流量监控 signals，再用 experiments 验证改动。
- 不是每个 bug 都应该进 eval suite。关键标准是：是否关键路径、是否可能回归、是否代表一类失败。20 个高信号 case 胜过 200 个低信号 case。
- 自诊断信号可以辅助，但不能替代人工 error analysis；它本质上仍是需要调阈值和防偏置的分类问题。

## 与已有知识的连接

这篇文章和 [[building-self-improving-tax-agents-with-codex]] 高度互补：后者展示了生产 trace、专家反馈和 Codex eval loop 如何让 Tax AI 持续改进；这篇则把同一思想抽象成更通用的 agent eval 方法论。

它也补充了 [[agent-toolchain-investment]]：工具链投资不只是接入更多模型或 dashboard，而是让团队能稳定捕获 trace、复现失败、运行本地 eval、观察生产 issue，并把经验变成回归测试。

## 个人评价

这篇文章最值得吸收的是“先 error analysis，再 eval”的顺序。很多团队的问题不是 eval 不够多，而是没有先读足够多真实失败，因此测试集里充满看起来专业但无法保护产品信任的 case。

对代码开发实践来说，可以直接迁移成 FIND case 工作流：真实失败先落成静态 case 和最小复现，再决定是否进入 golden cases 或 code-aware eval。不要为了显得系统化而过早搭大平台；先用小而强的 case 抬高下限。

## 来源

- [[how-to-eval-ai-agents-2026-guide]]
