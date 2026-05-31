---
title: "用 Codex 构建自我改进的税务 Agent"
type: note
tags: [distill, engineering, ai-tools, learning]
created: 2026-05-30
updated: 2026-05-30
status: active
summary: "生产痕迹、专家反馈和 eval 循环可把 Codex 变成持续改进 agent 的工程系统。"
source: [[building-self-improving-tax-agents-with-codex]]
concept_candidates:
  - self-improving-agents
  - production-trace-eval-loop
  - practitioner-feedback-loop
  - codex-driven-improvement-loop
  - bounded-agent-task-environment
---

# 用 Codex 构建自我改进的税务 Agent

## 核心论点

这篇文章的核心不是“Codex 能写代码”，而是：当真实业务产品能把生产反馈转成结构化证据，并配套 targeted eval 与 regression eval，Codex 才能进入一个可验证、可复用、可持续改进的工程循环。

Tax AI 的案例说明，自我改进 agent 不是靠模型自行觉醒，而是靠三件事同时成立：领域专家持续产生高质量纠错信号，产品保留从输入、抽取、映射到最终提交的完整 trace，工程系统把重复失败模式包装成有成功标准的 Codex 任务。

## 关键洞察

- 生产环境的问题不能只看最终输出。真实失败可能来自抽取遗漏、字段映射、产品不支持、税务判断或正常流程噪音；没有 trace，就无法区分。
- 专家反馈的价值不只是“标答案”，而是帮助系统判断哪些差异真的需要修、哪些只是业务偏好或流程噪音。
- Product trace 是 eval 的原材料。系统需要记录 Tax AI 的预测、专家修改、最终申报结果、字段来源和中间映射，才能把纠错转成可复现评测。
- Codex 的任务边界必须清楚：它读取 trace、eval、repo、skills 和文档，调查根因，修改受控产品表面，再通过 targeted eval 和 regression eval 验证。
- 自动化只覆盖 bounded layer。架构、产品判断和最终 shipping 仍由工程师负责；证据不清或不适合自动化的 case 应回流给产品和工程团队。

## 与已有知识的连接

这篇文章和 [[ai-should-help-us-produce-better-code]] 形成一组更完整的 agentic engineering 图景：前者强调 agent 降低重构、实验和质量改进成本；这篇进一步说明，若要让 agent 持续改进业务系统，必须先把生产问题变成可读、可测、可验证的任务环境。

它也补充了 [[agent-toolchain-investment]]：投资工具链不只是给 agent 更多命令，而是建立能产生证据、压缩上下文、限定可写范围、运行 eval、保留人工 review 的闭环。

## 个人评价

这篇文章值得沉淀的地方，是它把“自我改进 agent”从宣传语落到工程约束：专家反馈、生产 trace、eval、任务封装、只读证据、可写 worktree、回归验证、人类 review，这些缺一不可。

对个人知识库来说，最有用的迁移不是税务场景本身，而是这个判断标准：如果一个 agent 系统无法把真实失败转成可复现 eval 和明确成功条件，那它就还不是可靠的自我改进系统，只是一个需要人不断救火的自动化工具。

## 来源

- [[building-self-improving-tax-agents-with-codex]]
