---
title: "AI should help us produce better code - Agentic Engineering Patterns"
source_url: "https://simonwillison.net/guides/agentic-engineering-patterns/better-code/"
author: "Simon Willison"
updated: "2026-05-03"
created: 2026-05-03
type: "source"
tags:
  - "clipped"
media_type: "article"
status: "distilled"
distilled_at: 2026-05-03
---
## AI should help us produce better code

Many developers worry that outsourcing their code to AI tools will result in a drop in quality, producing bad code that's churned out fast enough that decision makers are willing to overlook its flaws.

If adopting coding agents demonstrably reduces the quality of the code and features you are producing, you should address that problem directly: figure out which aspects of your process are hurting the quality of your output and fix them.

Shipping worse code with agents is a *choice*. We can choose to ship code [that is better](https://simonwillison.net/guides/agentic-engineering-patterns/code-is-cheap/#good-code) instead.

## Avoiding taking on technical debt

I like to think about shipping better code in terms of technical debt. We take on technical debt as the result of trade-offs: doing things "the right way" would take too long, so we work within the time constraints we are under and cross our fingers that our project will survive long enough to pay down the debt later on.

The best mitigation for technical debt is to avoid taking it on in the first place.

In my experience, a common category of technical debt fixes is changes that are simple but time-consuming.

- Our original API design doesn't cover an important case that emerged later on. Fixing that API would require changing code in dozens of different places, making it quicker to add a very slightly different new API and live with the duplication.
- We made a poor choice naming a concept early on - teams rather than groups for example - but cleaning up that nomenclature everywhere in the code is too much work so we only fix it in the UI.
- Our system has grown duplicate but slightly different functionality over time which needs combining and refactoring.
- One of our files has grown to several thousand lines of code which we would ideally split into separate modules.

All of these changes are conceptually simple but still need time dedicated to them, which can be hard to justify given more pressing issues.

## Coding agents can handle these for us

Refactoring tasks like this are an *ideal* application of coding agents.

Fire up an agent, tell it what to change and leave it to churn away in a branch or worktree somewhere in the background.

I usually use asynchronous coding agents for this such as [Gemini Jules](https://jules.google.com/), [OpenAI Codex web](https://developers.openai.com/codex/cloud/), or [Claude Code on the web](https://code.claude.com/docs/en/claude-code-on-the-web). That way I can run those refactoring jobs without interrupting my flow on my laptop.

Evaluate the result in a Pull Request. If it's good, land it. If it's almost there, prompt it and tell it what to do differently. If it's bad, throw it away.

The cost of these code improvements has dropped so low that we can afford a zero tolerance attitude to minor code smells and inconveniences.

## AI tools let us consider more options

Any software development task comes with a wealth of options for approaching the problem. Some of the most significant technical debt comes from making poor choices at the planning step - missing out on an obvious simple solution, or picking a technology that later turns out not to be exactly the right fit.

LLMs can help ensure we don't miss any obvious solutions that may not have crossed our radar before. They'll only suggest solutions that are common in their training data but those tend to be the [Boring Technology](https://boringtechnology.club/) that's most likely to work.

More importantly, coding agents can help with **exploratory prototyping**.

The best way to make confident technology choices is to prove that they are fit for purpose with a prototype.

Is Redis a good choice for the activity feed on a site which expects thousands of concurrent users?

The best way to know for sure is to wire up a simulation of that system and run a load test against it to see what breaks.

Coding agents can build this kind of simulation from a single well crafted prompt, which drops the cost of this kind of experiment to almost nothing. And since they're so cheap we can run multiple experiments at once, testing several solutions to pick the one that is the best fit for our problem.

## Embrace the compound engineering loop

Agents follow instructions. We can evolve these instructions over time to get better results from future runs, based on what we've learned previously.

Dan Shipper and Kieran Klaassen at Every describe their company's approach to working with coding agents as [Compound Engineering](https://every.to/chain-of-thought/compound-engineering-how-every-codes-with-agents). Every coding project they complete ends with a retrospective, which they call the **compound step** where they take what worked and document that for future agent runs.

If we want the best results from our agents, we should aim to continually increase the quality of our codebase over time. Small improvements compound. Quality enhancements that used to be time-consuming have now dropped in cost to the point that there's no excuse not to invest in quality at the same time as shipping new features. Coding agents mean we can finally have both.

## 提炼

- [[ai-should-help-us-produce-better-code]]
