---
layout: post
title: "AI in the SDLC — The Honest State of Things in 2026"
date: 2026-04-11 08:00:00 +0530
categories: [ai, sdlc]
tags: [sdlc, ai-in-sdlc, agentic-ai, coding-agents, github-copilot, claude-code]
description: "Where AI genuinely fits in the software development lifecycle today, where it's oversold, and what 11 years of engineering experience tells me about separating signal from noise."
author: akashtalole
---

Every few months someone publishes a diagram showing AI slotted neatly into every phase of the SDLC. Requirements, design, development, testing, deployment, monitoring — little AI icons everywhere. It looks very tidy.

Reality is messier. Some of those slots are real. Some are demos that don't survive contact with an actual enterprise codebase. And a few are genuinely transformative in ways the diagrams understate.

After spending real time with these tools across real projects, here's my honest map of where things stand.

---

## Where AI Is Genuinely Useful Right Now

### Development — This Is the Real Win

Let's start with the obvious one because it's obvious for good reason: AI is most useful where the feedback loop is tightest, and that's code writing.

Not because it writes perfect code — it doesn't. But because it dramatically compresses the time between "I have a vague idea" and "I have something I can test and iterate on." For code I understand well, it's a fast first-drafter. For code in a domain I'm less familiar with, it's an interactive tutor that also writes the examples.

Where it works best:
- Boilerplate and scaffolding that you know exactly what you want but don't want to type
- Exploring unfamiliar libraries or APIs
- First-pass implementations where correctness isn't the point yet — direction is
- Translating logic from one language or framework to another

Where it still trips up: complex business logic with lots of implicit constraints, anything requiring deep knowledge of your specific data model or domain, and anything where the right answer depends on decisions made in three other files it hasn't seen.

### Code Review — As a Pre-Reviewer, Not a Reviewer

This is an underused one. Before a PR goes to a human reviewer, running it through an AI pre-review catches a surprising amount of low-hanging fruit: inconsistent error handling, missing null checks, functions that do too many things, security patterns you forgot about.

The value isn't catching everything. It's reducing the cognitive load on human reviewers so they can focus on what actually needs human judgment — architecture decisions, business logic correctness, whether this approach is even right for this problem.

I've seen AI pre-review meaningfully shorten review cycles. It won't replace human review. But it's a useful first pass.

### Documentation — Quietly One of the Best Use Cases

Documentation is tedious. It's important. Engineers consistently underprioritize it because the reward is diffuse and distant.

AI makes the cost low enough that you actually do it. Docstrings, PR descriptions, README sections, architecture decision records, runbooks — all of these are dramatically faster to produce when you're mostly editing AI output rather than starting from a blank page.

The output quality is high enough that I rarely throw it away. I edit it, I correct it, I add context it couldn't know. But the structure is usually solid and it saves the worst part: starting.

### Legacy Code Comprehension — Underrated

I mentioned this in Day 1 but it deserves more space. One of the most painful parts of senior engineering work is inheriting code you didn't write, with no tests, no documentation, and an author who left two years ago.

AI has meaningfully reduced the time it takes me to build a working mental model of unfamiliar code. Not because it's always right about what the code does — it sometimes isn't — but because having something to react to is much faster than building understanding from scratch. It's the difference between interrogating a witness and staring at an empty room.

### Test Generation — Useful With Caveats

AI can generate unit tests quickly. For well-scoped, pure functions with clear inputs and outputs, the generated tests are often genuinely good and catch real edge cases.

The problem is coverage theater: you can end up with a test suite that has high coverage metrics and low actual confidence. AI-generated tests tend to test the happy path well and miss the failure modes that matter — the ones that depend on your specific data, your specific environment, your specific business rules.

Useful as a starting point. Not a substitute for thinking about what actually needs to be tested.

---

## Where AI Is Oversold

### Requirements and Planning

The pitch: AI reads stakeholder documents, generates user stories, spots gaps in requirements, produces acceptance criteria.

The reality: it can do all of those things at a surface level. The output looks like requirements. It has the right structure. But requirements work is fundamentally about surfacing hidden assumptions, resolving conflicts between what different stakeholders want, and making explicit the things everyone "knows" but nobody has written down.

AI doesn't have access to the three years of context your BA has about why the system works the way it does. It doesn't know that what the product manager said and what they meant are two different things. It can help you structure and draft, but it can't do the actual work of requirements elicitation.

Use it to format and generate options. Don't use it to think.

### Architecture and System Design

Similar story. AI can generate architecture diagrams, suggest patterns, compare approaches. For greenfield work with standard patterns, this can be a useful starting point.

But architectural decisions in real systems are constrained by things that never make it into a prompt: the team's actual skill set, the performance characteristics of the existing infrastructure, the organizational structure that maps onto the codebase, the technical debt you're not allowed to touch right now.

AI doesn't know about the six-year-old batch job that everything depends on, or the database that can't handle more than X connections, or the fact that the team has zero experience with the event-driven pattern it just recommended. These constraints are what make architecture hard, and they live entirely outside the context window.

### End-to-End Feature Delivery

This one gets demo'd convincingly. "Give AI a spec and get a working feature." It works, sometimes, for small, well-defined features in greenfield contexts.

For anything complex — real business rules, multiple system integrations, non-trivial state management — it falls apart well before the finish line. The last 20% of making something actually work in production is still very much human work.

The demos show the first 80%. Pay attention to what they skip.

---

## The Shift That Actually Matters

The most important change AI has made to how I work isn't in any specific SDLC phase. It's in the cost of exploration.

Before: if I needed to understand an unfamiliar library, or try a new architectural approach, or investigate whether a refactor was feasible, the upfront cost was high enough that I'd often not bother unless I was confident it was worth it.

Now: the cost of trying something is low enough that I explore more. I generate a prototype faster. I understand a tradeoff quicker. I document what I tried even when it didn't work.

That change — lower exploration cost — compounds over time. You make better decisions because you've seen more options. You understand more of the codebase because the cost of diving into an unfamiliar module went down. You write more documentation because it's not the chore it used to be.

That's the real SDLC impact. Not AI-generated architecture diagrams. A lower cost of doing the things that were always worth doing but often got skipped.

---

## What This Means for the Rest of the Series

The next 28 posts are going to go deep on specific tools — Claude Code, GitHub Copilot, Copilot Studio. But this is the lens I'm applying throughout: not "what can this tool do?" but "where in the actual development process does this tool meaningfully lower the cost of doing the right thing?"

When I find a genuine answer to that question, I'll say so. When the answer is "mostly in demos," I'll say that too.

---

*Day 2 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Yesterday: [Why I'm writing this](/posts/why-im-writing-30-days-of-ai-engineering/).*
