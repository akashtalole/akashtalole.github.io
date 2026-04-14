---
layout: post
title: "11 Years In, AI-Augmented — How My Workflow Actually Changed"
date: 2026-04-15
categories: [ai, meta]
tags: [claude-code, github-copilot, sdlc, ai-in-sdlc, agentic-ai]
description: "An honest before-and-after. Eleven years of engineering, eighteen months of AI tooling — here's what actually changed in how I plan, code, review, and ship. And what didn't."
author: akashtalole
---

I want to close Arc 1 of this series with something personal: an honest account of what actually changed in my day-to-day work when I started using AI tools seriously. Not the marketing version. The real version — including the parts that didn't change as much as I expected.

---

## Before I Had These Tools

Let me paint the picture accurately. Before I integrated AI tools into my workflow, I was already a reasonably efficient senior engineer. I had patterns that worked. I knew how to read a codebase, design a system, debug a hard problem. Eleven years of that builds real muscle memory.

The friction wasn't incompetence — it was the unavoidable tax on certain kinds of work:

- Understanding an unfamiliar codebase took days of reading, running, and dead ends
- Writing documentation felt like paying a tax with no immediate return
- The gap between "I have an idea" and "I have running code to validate it" involved a lot of typing I wasn't thinking about
- Code review caught what reviewers had time to check, not everything that should be checked

These weren't problems I was failing at. They were just the cost of doing the work.

---

## What Changed: The Honest Accounting

### Planning and Exploration — Significantly Faster

This is where I see the biggest delta. Before, when I picked up an unfamiliar piece of work — a new codebase, a new API, a problem domain I hadn't worked in — I'd spend a lot of time just orienting. Reading docs, running code, building a mental map.

Now I use Claude Code as an interactive guide through that orienting phase. I paste in code, ask questions, test hypotheses. The process is still the same — read, run, understand — but the iteration speed is dramatically higher.

What used to take a day often takes a morning. That compounds.

### Writing Code — Incrementally Better, Not Transformatively

This is where people expect the most change and often experience the most disappointment if they're already strong engineers.

Copilot makes the typing faster. It's good at boilerplate, repetitive patterns, filling out function signatures I already know the shape of. For a senior engineer who types fast and knows what they want, the acceleration is real but not enormous.

Where I feel it more is in unfamiliar territory — a language I use occasionally, a library I haven't worked with recently, a framework pattern I need to look up. The cost of those situations dropped significantly. I reach for documentation less and stay in the editor more.

### Code Review — Meaningfully Better

I now run AI pre-review on my own code before submitting PRs and on others' code before starting detailed review. It catches enough low-hanging fruit — inconsistent error handling, obvious edge cases, style issues, missing validations — that human review time is genuinely more focused.

My PRs are tighter before they go up. Other people's PRs take less of my attention on the mechanical stuff so I can focus on the architecture and logic questions that actually need a human.

### Documentation — The Biggest Relative Improvement

I wrote documentation before. Not as much as I should have. The effort-to-reward ratio felt wrong and I'd often defer it.

Now I write significantly more documentation than I did two years ago. Not because I suddenly care more — because the cost is low enough that I actually do it. Docstrings, ADRs, README sections, runbook entries — I draft them in seconds, edit them in minutes. The activation energy is gone.

That's a bigger change than it sounds. Documentation debt is one of the most consistently painful things in software teams and it's almost entirely an incentive problem, not a knowledge problem.

### Debugging — Mixed Results

This is the one that surprised me. I expected AI to be a debugging superpower. It's more nuanced than that.

For certain classes of bugs — logic errors with clear symptoms, type errors, missing null checks — Claude Code is excellent. Paste the error, paste the relevant code, get a likely cause.

For the hard bugs — the ones that involve subtle interactions between distributed systems, race conditions, environment-specific behaviour, business logic that's wrong in a way that's hard to describe — AI is helpful as a thinking partner but rarely diagnoses the root cause directly. You still need to do the detective work.

### Architecture and Design — Mostly Unchanged

My process for designing systems is largely the same as it was before. AI is useful for quickly generating options and poking holes in reasoning, but the hard work of understanding tradeoffs in context — your team, your constraints, your existing system — is still human work.

I use Claude Code to pressure-test ideas and generate alternatives I might not have considered. I don't use it to make the decisions.

---

## What Didn't Change

**Judgment.** Knowing what the right problem is, whether a proposed solution is actually right for the context, what matters and what doesn't — none of that came from AI.

**Understanding the business.** The context that lives in people's heads, in meetings, in three years of history — AI doesn't have it and can't substitute for building it.

**The hard conversations.** Technical leadership, navigating disagreements, deciding what to build and what to cut — still entirely human.

**Debugging the truly hard things.** Concurrency bugs, distributed system failures, environment mysteries — still detective work.

---

## The Honest Summary

The AI tools I use daily have made me faster at a meaningful subset of my work. Not at all of it. The parts that got faster are real, and the compounding effect over months is significant.

But the things that separate a good senior engineer from an average one — judgment, understanding, communication, system thinking — those are unchanged. If anything, they matter more now, because the baseline productivity has risen and the differentiator has shifted further toward the things AI doesn't do.

That's the honest picture after eighteen months. Starting tomorrow, Arc 2 gets specific: Claude Code for enterprise teams.

---

*Day 6 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [Choosing Your AI Toolchain](/posts/choosing-your-ai-toolchain-claude-code-copilot-or-copilot-studio/).*
