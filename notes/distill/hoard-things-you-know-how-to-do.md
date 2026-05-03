---
title: "Hoard things you know how to do 提炼"
type: note
tags: [distill, engineering, ai-tools, learning]
created: 2026-05-03
updated: 2026-05-03
status: active
summary: "把可运行方案和 POC 长期囤积成 coding agent 可复用的高质量输入。"
source: [[hoard-things-you-know-how-to-do]]
concept_candidates:
  - working-example-hoard
  - proof-of-concept-library
  - recombination-prompting
  - agent-input-assets
  - agentic-engineering
---

# Hoard things you know how to do 提炼

## 核心论点

Simon Willison 这篇文章的核心不是一般意义上的“收藏资料”，而是长期囤积自己实际跑通过的解决方案、代码片段、实验仓库和小型 POC。工程师真正有复利价值的资产，是一组“我知道这件事可以做到，并且我看过它如何用代码做到”的证明集合。

在 coding agent 出现之后，这种资产的价值进一步放大。过去这些例子主要帮助人类回忆方案；现在它们也可以直接变成 agent 的输入，让 agent 基于已有的可运行例子组合出新的解决方案。

## 关键洞察

- “知道理论上可行”和“亲眼见过可运行代码”不是一回事。后者能让工程师更敏锐地发现技术机会，也能减少面对新问题时的空泛猜测。
- 可运行例子比抽象建议更适合喂给 coding agent。agent 可以搜索、读取、改写和组合代码，因此一份小型但完整的 POC 往往比一段概念描述更有操作价值。
- 囤积方案的重点是保留 proof，而不是保留灵感。Simon 用 blog、TIL、GitHub repos、HTML tools、research repo 等形式保存已经验证过的做法。
- 组合已有例子是一种高杠杆 prompting pattern。文章中的 OCR 工具来自两个既有片段：PDF.js 把 PDF 页面渲染成图片，Tesseract.js 对图片做 OCR；模型把两者拼成了一个可用的浏览器工具。
- agent 让“只解决一次”的收益变大。只要一个技巧被记录成可检索、可运行、可理解的例子，未来相似任务就可以让 agent 复用它，而不是每次从零开始探索。

## 与当前 wiki 的连接

这篇文章和前一篇 Karpathy 访谈形成互补：Karpathy 更强调 agentic engineering 的质量边界和可验证性，Simon 这里强调的是 agent 的输入资产建设。两者合起来看，agentic engineering 不只是“更会提示模型”，还包括持续建设一套可被 agent 调用的个人工程资产库。

这也直接对应当前这个 Obsidian wiki 的设计目标：把一次性阅读、剪藏、聊天和代码经验整理成长期可复用的 markdown 知识库。对你来说，真正值得沉淀的不是“这篇文章讲了什么”，而是哪些例子、工具链模式和验证方法以后可以被 Codex、Claude Code、Cursor 或 Gemini 重新调用。

## 个人判断

这篇文章最值得落地的一点是：以后遇到“我证明过某件事可以做到”的任务，不要只把结论写进聊天记录。更好的做法是保留最小可运行例子、验证命令、适用边界和失败条件。这样它就能从个人记忆变成 agent 可操作的输入资产。

对当前 wiki 来说，可以把这类材料沉淀成两层：source/distill 保存原始来源和观点提炼；真正多次复用的模式再升格成 concept，例如 working-example-hoard 或 recombination-prompting。这样既不会把 concepts 目录变成文章摘要堆，也能让重复出现的方法论逐步浮现。

## 来源

- [[hoard-things-you-know-how-to-do]]
