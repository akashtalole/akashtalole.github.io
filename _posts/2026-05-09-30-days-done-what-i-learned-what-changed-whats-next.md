---
layout: post
title: "30 Days Done — What I Learned, What Changed, What's Next"
date: 2026-05-09
categories: [ai, meta]
tags: [meta, ai, claude-code, github-copilot, copilot-studio, agentic-ai]
description: "Thirty posts across five arcs. Here's the honest synthesis: the biggest surprises, the things I was wrong about, what actually changed in my thinking, and where I think all of this is heading."
author: akashtalole
---

Thirty posts. Thirty consecutive days of writing about AI engineering from the inside — not reviewing tools from the outside, but trying to document what's actually happening in the work.

I want to close the series with something more than a summary. A synthesis. What I actually learned, where my thinking shifted, and the open questions I'm carrying forward.

---

## What I Was Wrong About at the Start

**I underestimated documentation as a use case.**

Before I started this series, I thought of AI tooling primarily as a code generation and code review accelerator. I knew documentation was a use case but I'd mentally filed it as secondary.

Writing about it for Day 17 made me realise how much the documentation impact compounds. It's not that AI writes better documentation — it's that AI makes documentation happen at all. The activation energy drop is big enough to change behaviour. And documentation debt, when it accumulates, has costs that are easy to underestimate until you're inside a legacy codebase trying to understand a system with no documentation.

**I was too optimistic about multi-step agents' current state.**

Going into this series, I was more bullish on autonomous coding agents than I am now. Not because they're not impressive — they are — but because the gap between "impressive in controlled conditions" and "reliable in production" is larger than I initially thought.

The evaluation dimension (Day 25) is the part most teams skip, and it's the part that would reveal the gap. When I've evaluated agents rigorously on representative tasks rather than cherry-picked demos, the pass rates are humbling.

**The mental model work (Days 3–4) mattered more than I expected.**

I almost cut those posts for being too theoretical. I kept them because the series needed foundation, not because I thought they'd be the most-read posts.

But the feedback I got was that Days 3 and 4 — the agentic AI mental model and agent skills anatomy — changed how people thought about the whole space. Getting the mental model right changes which questions you ask, which tradeoffs matter, and which failure modes you design for.

---

## What I'm More Confident About

**The layered toolchain principle (Day 5) is right.**

Claude Code, GitHub Copilot, and Copilot Studio are not competing. They operate at different layers — reasoning, in-editor flow, and product deployment — and each is genuinely best for its layer.

Engineers who try to use one tool for everything get worse results than engineers who use three tools intentionally. The pattern held across everything I wrote.

**Prompt and tool design is more important than model selection.**

This came up in different forms across the series. For Copilot, the context you provide (Day 15) matters more than the model version. For coding agents, the tool definitions (Days 20–22) matter more than the underlying model. For multi-agent systems, the handoff design (Day 27) matters more than which agent framework you use.

The model is a given. The interface you build around it is a choice, and it's the choice that determines reliability.

**Enterprise governance is not the obstacle — it's the enabler.**

I went into the enterprise sections (Days 7, 13, 19, 28) half-expecting to write about bureaucratic friction. I ended up writing about risk management that, when done properly, enables faster and more confident deployment.

Organisations that do the governance work — data classification, acceptable use policies, audit trails — end up with AI tools they can trust and expand. Organisations that skip it end up with tools they can't defend and eventually lose.

---

## The Open Questions I'm Carrying Forward

**Where is the reliability ceiling for coding agents on complex, real-world tasks?**

I still don't have a confident answer. The agents I build work well for bounded, well-defined tasks. For complex, open-ended engineering work — the kind that takes a senior engineer weeks — I see unreliable behaviour that I don't think is a prompt engineering problem. It might be a fundamental limit of current architectures.

**How do multi-agent systems age?**

Building a multi-agent Copilot Studio solution is one problem. Maintaining it as requirements change, as the enterprise systems it connects to evolve, as the team that built it changes — that's a different problem, and I haven't had enough time with deployed systems to have good answers.

**What does AI-augmented engineering look like at the team level?**

Most of what I've written is from the perspective of one engineer using AI tools. The team-level dynamics — how AI changes code review culture, pair programming, knowledge transfer, onboarding — are a whole separate set of questions I've only started to think about.

---

## What Didn't Make the Series

I planned a post on AI-assisted incident response that got cut for time. A post on using Claude Code for architectural decision record generation. A deep dive on Copilot for Azure that I didn't feel ready to write yet.

These are the next series.

---

## The Honest State of Things

We are in a period where the tools are genuinely useful, the failure modes are real and not fully understood, and the gap between "works in demos" and "works in production at scale" is significant.

The engineers who navigate this period well are the ones who:
- Use the tools seriously enough to find the real edges
- Are honest about limitations rather than evangelising capabilities
- Build governance and observability into their deployments, not as afterthoughts
- Evaluate their agents on real tasks rather than curated examples
- Think about the full system — tools, prompts, design, evaluation — not just the model

I started this series to document what I was learning. I'm ending it having learned more than I expected to, with more open questions than I started with, and more confident that the work of figuring this out carefully is worth doing.

Thirty days is a start. The real work continues.

---

*Day 30 — the final post of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/).*

*If you found this series useful, the full archive starts at [Day 1](/posts/why-im-writing-30-days-of-ai-engineering/).*
