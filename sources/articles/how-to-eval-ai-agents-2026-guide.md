---
title: "How to Eval AI Agents — The 2026 Guide"
source_url: "https://www.howtoeval.com/"
author: "Ben Hylak/Published May 2026/10,014 views"
updated: "2026-05-31"
created: 2026-05-31
type: "source"
tags:
  - "clipped"
media_type: "article"
status: "distilled"
distilled_at: 2026-05-31
---
## Foreword

A year ago, agents barely existed. Now they are everywhere: in banking, engineering, medicine and more.

For a while, I hated the word agent. Why add a new word for an LLM call? But, it soon became obvious to me and everyone else that a new word was indeed necessary.

Agents are an entity, almost self-aware, navigating their environment. Using tools at their disposal, and finding creative solutions to problems their creators could never have imagined.

Sometimes those creative solutions are helpful. Sometimes they cause real harm. But what's for sure: we've come a long way from next token prediction.

It is interesting, then, that the method for evaluating reliability and performance of agents mostly hasn't changed since the previous "chatbot" and RAG era.

Companies in the business of evaluating agents ("evals"), are often suggesting increasingly absurd and over-complicated strategies. The tools that are being sold have never been more divergent from what's really working for good companies.

I'm writing this to say: it is not that complicated. I know because I work at a company that builds agents. And we, in turn, work with the best AI companies in the world: Framer, Clay, Vercel, [GC.AI](https://gc.ai/), and more.

This guide is designed to be neutral. Yes, I will talk about the eval company I founded, [Raindrop](https://raindrop.ai/), because I think it is the best manifestation of the practices described therein. If it was not already, we would make it so. I will also mention [Raindrop Workshop](https://raindrop.ai/workshop), which is a free/completely open source tool for local testing.

Reader beware: this guide is about agents. The way you evaluate classifiers, RAG pipelines, and many other AI systems is differentSampling results and evaluating with Opus, for example, can be much more useful there.. That is not what I am covering here.

## Benchmark-maxxer or floor-raiser?

Before you think about evals, you have to choose what kind of progress you are trying to make.

For most products, this boils down to two choices: you can **benchmark maxx**, or you can **raise the floor**. Most teams never make this choice explicitly. They borrow the language of evals from labs, copy the shapes of benchmarks, and end up chasing a goal that was never made for them.

Take the quiz

QUESTION 1 OF 7

Could your agent’s worst mistake end up in the news?

Imagine a finance agent. It would be nice if it could cross-reference all of your accounts, predict your spending, and cancel subscriptions automatically. But if it sometimes cannot answer the basic question, "how much money is in my bank account?", you would stop trusting it.

That is why most product teams should think in terms of **floor raising**: making the agent reliable where reliability matters, in the workflows users actually run, in the moments where mistakes are expensive, in the cases that would make you afraid to ship.

### Floor raising is error analysis

This is why floor raising looks less like benchmark design and more like error analysis, as Hamel Husain would say. You are not starting with an abstract test suite. You are doing detective work. You are finding the places where the system bends, classifying them, and deciding which ones deserve engineering effort.

The process is simple, but not easy. Review user messages, agent responses, and the trajectory the agent took to get to its answer. Use AI as much as you can to scale the number of traces you look at: filter, cluster, summarize. The questions stay the same: what was the last successful step? What was the first real failure? Did retrieval miss? Did the agent ignore context? Did a tool call go wrong? Did the final answer overstate what the system knew?

Then you fix the pattern, not just the incident. Sometimes that means better retrieval. Sometimes it means constraining the agent. Sometimes it means changing a tool, adding a guardrail, or teaching the system to stop and say "I don't know."

Only then do you add evals. And when you do, they are targeted. You are not testing imaginary coverage. You are locking in lessons from real failures. A floor-raising eval suite is a memory of bugs you refuse to reintroduce.

This is why massive eval sets are often a distraction. The thing that hurts you is usually not the thousandth synthetic variant of a prompt. It is the refund policy hallucination, the infinite loop, the subtle context loss after three tool calls. Floor raising is how you find those failures, understand them, and make the next version of the agent harder to embarrass.

Autocomplete

Cursor

Claude Code

Support chatbot

Banking agent

Devin

AI Doctor

← Benchmark maxx

Floor raise →

If you could ship with a 90% pass rate or a 99% pass rate, which would you choose? If your instinct is "99%, obviously," you are still thinking like a benchmarker. If your first question is "which 1% fails?", you are thinking like someone raising the floor.

One of the lowest-hanging floor-raising techniques is also one of the most frustrating for users: teaching your agent to refuse. If confidence is below a threshold, say "I don't know." If the query is outside your domain, say "I don't know." If the retrieved context is insufficient, say "I don't know."

Benchmark maxxers hate this. It lowers their pass rate. But floor raisers understand that a confident wrong answer is worse than an honest "I don't know." For products replacing humans, trust is more important than capability demonstration.

## Know your agent before you ship

Before you trust an agent in production, you need to get acquainted with it: what it handles cleanly, where it gets confused, and which failures would actually matter.

### Golden cases

Pick 5 to 10 cases that represent your critical paths and write them down somewhere. They don't need a file format or a harness. Start with the simplest version: a single, common user question your agent should always handle correctly. Try it. You can make this more complex over time, but begin there. If your agent starts failing your golden cases, you do not ship.

### Understand your agent locally

For your golden cases, do not just check the final output. Inspect the full trajectory: the user message, the tool calls, the retrieved context, the reasoning chain. The path matters as much as the answer. [Raindrop Workshop](https://raindrop.ai/workshop) is useful here: it lets you capture real agent traces locally, inspect every tool call, and replay scenarios before they hit production.

### Pro tip: asking your agent

One of the best ways to deeply inspect your agent is asking it questions. Weirdly, I haven't heard anyone talk about this.

To be clear: I do not mean asking *an* agent to look at the traces and guess why. I mean talking to *your* agent. Reconstruct the run exactly as it was passed to the agent, along with the latest response, and ask it directly what happened.

This matters because reasoning traces are often encrypted. You cannot always read the chain of thought yourself. Passing the trace back to the same model is the closest you can get to asking the agent what actually happened, because it can use the available trace, context, and prior messages as evidence for its answer.

This often helps you learn where your agent is misinterpreting or overindexing on a specific part of your prompt. Ask something like: "You were wrong. The answer was X. What would I need to have changed for you to get this right?" Obviously, what your agent will answer is not always the truth. Use it as a clue.

## Offline evals

You need code-aware evals. Testing prompts by themselves makes no sense once the agent is entangled with code, tools, retrieval, permissions, and product state. The behavior lives in the whole system, not in the prompt string.

This is why offline evals look less like prompt scoring and more like ordinary software testing. Think Vitest, pytest, or jest. A good offline eval takes an input, runs the real agent path, and asserts on the result: output, tool calls, files changed, structured data, or final state.

Sentry calls this harness-backed, and their [vitest-evals](https://github.com/getsentry/vitest-evals) examples use \`describeEval(...)\`, an app-local harness, an explicit \`run(...)\`, normal \`expect(...)\` assertions, and tool-call checks.

OpenAI calls the same idea “macro evals” in their [agentic systems cookbook](https://github.com/openai/openai-cookbook/blob/main/examples/partners/macro_evals_for_agentic_systems/macro_evals_for_agentic_systems.ipynb): drive the real agent loop on representative inputs and grade the full trajectory, including tool calls, intermediate state, and the final answer.

I do not care what you call it. The important thing is that you're evaluating the agent, not an LLM call.

// evals/refund\_agent.eval.ts

import { expect } from 'vitest'

import { describeEval, toolCalls } from 'vitest-evals'

import { refundAgentHarness } from '../harness'

describeEval('refund agent', { harness: refundAgentHarness() }, (it) => {

it('approves refundable invoice', async ({ run }) => {

const result = await run('Refund invoice inv\_123')

expect(result.output.status).toBe('approved')

expect(toolCalls(result.session).map((c) => c.name)).toEqual(\['lookupInvoice', 'createRefund'\])

})

})

[Raindrop Workshop](https://raindrop.ai/workshop) makes it really easy to create code-aware evals: you can save trajectories, and Codex/Claude can annotate runs to let you know if they pass or fail (along with more specific commentary).

You can literally store your evals in a markdown file and just ask Codex to run them.

I *really don't recommend* trying to use a hosted eval dashboard to run agent evals. There's a reason Raindrop doesn't offer one, and it's not because it's hard to build (it's actually very easy).

It's because the report UI is the easy part to build, and the part that should be the most tightly coupled with your product. I'd start with spinning up a quick, local HTML viewer for your evals to see pass/fail.

## Learning from production

Once an agent is in the world, the work changes. Before ship, you are guessing where it might break. After ship, users show you where the product is actually confusing, brittle, or underspecified.

### Start with raw logs

Start by reading the raw interaction: user messages, agent responses, tool calls, and the moments where the user expected one thing and the agent did another. Do this until you hit saturation, until the same patterns start appearing again and again.

> “Error analysis \[is\] the single most valuable activity in AI development and consistently the highest-ROI activity.”

This is tedious work. It is also the only way to build a real mental model of the system. Skip it and you will automate the wrong metrics.

### What you need depends on volume

The workflow should scale with your traffic. At ten agent runs a day, you can read everything. At ten thousand, you need the system to tell you what deserves attention. The mistake is reaching for the high-volume machinery before you have enough raw texture to know what you are looking for.

1-100 runs/day

#### Stumbles

Use raw logs as the firehose. Look for confusion, frustration, near-misses, repeated prompts, and "you're holding it wrong" moments. At this stage, the goal is taste and taxonomy.

100-1,000 runs/day

#### Issues

When the same stumble keeps appearing, turn it into an [Issue](https://raindrop.ai/docs): an emerging problem the team can discuss, reproduce, and decide whether to fix.

1,000+ runs/day

#### Signals

Use [Signals](https://raindrop.ai/docs) for behaviors you want to monitor over long horizons: aesthetics complaints, refusal quality, ignored tool errors, context loss, user frustration.

5,000+ runs/day

#### Experiments

Once you understand the Issue, ship the fix behind a feature flag and compare the affected Issues and Signals. Production should tell you whether the change helped.

### Self diagnostics

Raindrop can also let the agent report its own problems. Under the hood, this injects a hidden tool into the agent run. When the agent thinks it is missing context, hitting a capability gap, dealing with a broken tool, or failing the task outright, it can call that tool and Raindrop records the report as a signal on the same event. In the AI SDK integration, it is basically one line on the wrapper:

const { generateText } = raindrop.wrap(ai, { selfDiagnostics: { enabled: true } })

Conceptually, the hidden tool looks something like this:

\_\_raindrop\_report({

category: 'missing\_context',

summary: 'I could not find the refund policy for enterprise plans.',

severity: 'medium',

})

In practice, we have seen this be less useful than it initially seemed on the surface. Treat self diagnostics like Stumbles: a firehose of random things where you might pick out something interesting. It takes real work to dial in sensitivity and make it a useful generalized signal. Specific wording can bias whether the agent reports at all. At the end of the day, it is a binary-classification problem.

## Making fixes and changes

You have found a problem. Now you need to fix it. The approach depends on what kind of problem it is.

### Sometimes direct

Some fixes are obvious and immediate. A broken tool call. A missing instruction in the system prompt. A retrieval query that is returning stale data. Fix it, ship it, move on.

The key is knowing when a fix is truly direct versus when it is a bandaid on a deeper issue. A quick fix that addresses the symptom but not the cause will just create a weirder bug later. If you find yourself adding special-case handling for specific user phrases, you are probably treating symptoms.

### Repro, then fix

For non-trivial issues, the first step is always reproduction. Can you make this failure happen again? If not, you do not understand it yet. Go back to the trace. Read it again. Find another example.

Once reproducible, add it as a golden case before fixing. This guarantees you will not regress. The pattern is: find in production, reproduce locally, add to evals, fix, verify eval passes, ship.

### Adding as an eval case often has diminishing returns

Here is a trap: every time you find a bug, you add it to your eval suite. Six months later, you have 500 cases, 400 of which are weird edge cases that never happen in production. Your CI takes 20 minutes. Your team starts ignoring eval failures because "it is always something."

Not every bug deserves an eval case. Ask: is this a critical path? Could it regress? Is it representative of a class of failures, or truly one-off? Be ruthless about pruning your eval suite. 20 high-signal cases beats 200 low-signal ones.

A good heuristic: if an eval case has not failed in 3 months, it is either not testing something important or your agent has genuinely improved. Either way, question whether it needs to be there.

### Often: experiment in production

For many changes, the only way to know if they work is to test with real users. Compare models, compare prompts, compare tool configurations. Run A/B tests on real traffic and measure actual outcomes.

This is where production monitoring becomes essential. You need to know: did the new model reduce hallucinations? Did the new prompt increase user satisfaction? You cannot answer these offline. The eval suite will tell you it did not break golden cases. Production tells you if it actually helped.

// A/B test: GPT-5.3 vs Claude 4.5 Sonnet

Treatment: 88% task completion rate (up 12%)

Control: 76% task completion rate

Statistical significance: p<0.001

Promote treatment to 100% traffic

## Rinse, wash, repeat

This is the whole loop: ship, watch what happens, understand the failures, fix the important ones, and keep the useful lessons as regression tests.

Each iteration should make the agent a little less embarrassing and the eval suite a little more grounded in reality. You are not trying to front-load all possible test cases. You are building a system that learns from production without forgetting what it learned.

The work of evaluation is ongoing, not something you check off once. If the loop stops, the suite goes stale and confidence becomes theater.

Plan to spend 10 to 20% of your agent development time on evaluation and monitoring. Not just writing eval cases, but reading traces, tuning signals, and investigating issues. This is the cost of building reliable AI systems. Teams that try to skip this pay the price in production incidents.

## Looking forward

The evaluation landscape is changing fast. Here is what we are watching.

### The collapse of harnesses

The current model of "agent SDKs that call models" is already showing cracks. As models become more capable and agentic, the distinction between "the model" and "the agent" blurs. We are seeing this with Claude Code, the Cursor CLI, and the emerging Cursor Agent SDK.

What does evaluation look like when the agent is just a prompt and a model? When there is no framework code to instrument, no explicit tool loop to trace? The harness, the scaffolding around the agent, collapses into the model itself.

This changes monitoring. Today we trace tool calls because we need to. Tomorrow we might just record input/output pairs and ask the model to explain what it did. The intermediate steps become opaque, like asking a human what they were thinking. You get an answer, but verification becomes harder.

The implication: end-to-end evaluation becomes even more important. If you cannot inspect the internals, you have to trust (and verify) the outputs. Golden cases and production monitoring, already our recommendations, become the only game in town.

### From app to MCP

Coming soon.

### What stays constant

Through all these changes, some things remain true:

- You need contact with real user behavior. Synthetic evals diverge from reality.
- Trajectory matters. Whether explicit (tool calls) or implicit (reasoning), how an agent got there is diagnostic.
- Golden cases work. A small set of high-signal tests beats a large set of low-signal ones.
- Production monitoring is essential. You cannot anticipate all failure modes in advance.
- Evaluation is ongoing work, not a one-time setup.

The tools will change. The frameworks will change. The models will change. The fundamental challenge, verifying that an autonomous system does what you want, stays the same.

## The short version

If you remember nothing else:

- **Pick the right frame.** Benchmark maxxing is for augmenting experts. Floor raising is for agents used in-place of humans.
- **Floor raising is error analysis.** Read real interactions, find the patterns that matter, and fix the system behind them.
- **Use code-aware offline evals.** Test the running agent path: tools, state, files, structured output, and side effects.
- **Scale production review with volume.** Start with Stumbles, let recurring patterns become Issues, track long-term Signals, and validate fixes with Experiments.
- **Keep the loop tight.** Real failures become targeted regressions, not a giant speculative eval suite.

### Ready to evaluate your agents properly?

Raindrop Workshop gives you local trace capture, trajectory inspection, and replay for agent evaluation. Start there. When you are ready for production monitoring, Raindrop scales with you.

## 提炼
- [[how-to-eval-ai-agents-2026-guide]]
