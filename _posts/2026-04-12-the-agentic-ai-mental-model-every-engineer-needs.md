---
layout: post
title: "The Agentic AI Mental Model Every Engineer Needs"
date: 2026-04-12
categories: [ai, agentic-ai]
tags: [agentic-ai, agents, coding-agents, agent-skills, claude-code, copilot-studio]
description: "Agentic AI is one of the most overused phrases in tech right now. Here's what it actually means for engineers building real systems — and why the mental model matters more than the buzzword."
author: akashtalole
---

"Agentic AI" is everywhere right now. Every product announcement uses it. Half the LinkedIn posts about AI use it. Most of them mean something slightly different by it, and a few mean nothing at all.

That's a problem, because the concept underneath the buzzword is actually important — not as a marketing term, but as an engineering paradigm. If you're building systems that use AI, or using AI tools to build systems, understanding what "agentic" really means will change how you think about design, failure, and trust.

So let's be precise about it.

---

## The Non-Agentic Baseline

Start with what most people actually use AI for today: you give it an input, it gives you an output, you do something with that output. A completion, a code suggestion, a chat response. One round trip.

This is AI as a function. `f(input) → output`. You're in control the whole time. You decide what to ask. You decide what to do with the answer. The AI doesn't take any actions in the world — it just produces text.

This is still genuinely useful. Most of GitHub Copilot works this way. Most of the Claude Code interactions I do in a day are this. Input → output → I decide what to do next.

Agentic AI is different in one fundamental way: **the AI decides what to do next.**

---

## What "Agentic" Actually Means

An agent has a goal, a set of tools it can use, and a loop: observe → reason → act → observe again.

Instead of one round trip, you get a process. You give the agent a task — "fix the failing tests in this module" or "find all the places we're not handling null in this service" — and it figures out the steps: what to look at, what to run, what to change, what to check. It doesn't ask for permission between each step. It decides.

The three things that distinguish an agent from a plain LLM call:

**1. Tools / Actions**
The agent can actually *do things* — read files, run code, call APIs, write to databases, trigger workflows. It's not just producing text for a human to act on. It's acting directly.

**2. A Loop**
The agent observes the result of each action and decides what to do next based on what it saw. If the test still fails after the first fix, it doesn't stop — it looks at the new error and tries again.

**3. Goal-Directed Behaviour**
You give it an objective, not a single question. It plans toward that objective, adapts when things don't go as expected, and stops when the goal is met (or when it gives up).

That's it. Everything else — memory, multi-agent orchestration, specialized tools — is layered on top of this core loop.

---

## Why This Mental Model Changes Everything

The shift from "AI as function" to "AI as process" sounds incremental. It isn't.

### Failure Propagates Differently

When an LLM call returns bad output, the damage is contained. You see the output, you reject it, nothing happened. When an agent acts on bad reasoning, it may have already written to a file, called an API, or made ten downstream decisions before you see anything wrong.

Agents fail in the middle of doing things. That's a different failure mode from tools you're used to. It requires thinking about what "undo" looks like, what the blast radius of a wrong action is, and where you need checkpoints.

### Observability Is Harder

With a function call, the trace is simple: input, output, done. With an agent, you have a sequence of reasoning steps, tool calls, intermediate observations, and decisions — many of which may not surface unless you explicitly log them.

I've spent more time building observability into agentic systems than almost anything else. Not because the agents are opaque by nature, but because the default level of visibility isn't enough to debug them when something goes wrong. And something always goes wrong eventually.

### Testing Is Fundamentally Different

You can unit test a function. What do you unit test in an agent? The individual tool calls? The planning step? The end-to-end behaviour given a particular goal?

In practice, agentic systems need evaluation frameworks, not just test suites. You're testing probabilistic behaviour over a distribution of inputs, not deterministic outputs for specific inputs. This is one of the things that's genuinely hard about agentic AI right now, and anyone who tells you they've fully solved it is probably selling something.

### The Action Space Is a Design Decision

When you're building an agent, one of the most important things you decide is what tools it has access to. This isn't just a capability question — it's a risk question.

An agent that can read files is less risky than one that can write them. One that can write files is less risky than one that can execute code. One that can execute code is less risky than one that can call external APIs with side effects.

Every tool you give an agent is a decision about what it can get wrong, and how badly. Scope the action space to what the task actually requires. Don't give it access to systems it doesn't need for this task. This sounds obvious and is frequently ignored.

---

## What This Looks Like in Practice

I'll give you two examples from work I'm doing right now.

**Coding Agent**
A coding agent that can read the codebase, run tests, write code changes, and re-run tests. The goal: fix a failing test suite. The loop: read the error → reason about the cause → make a change → run tests → observe new output → repeat.

The action space is intentionally limited: read files, write files, run tests. It can't commit, can't push, can't touch production. The blast radius of a mistake is bounded. I can inspect every step it took before I decide whether to accept the changes.

**Copilot Studio Multi-Agent**
A customer-facing agent that can look up account information, check order status, and escalate to a human. Behind it, specialist agents handle different domains. The orchestrator decides which agent to route to based on intent.

Here the human-in-the-loop is at the end: every action that has side effects (like modifying an order) goes through an approval step before it's committed. The agents can gather information and reason about it freely. They can't take irreversible actions autonomously.

Same principle in both cases: the loop is autonomous, but the *consequences* of the loop are bounded and visible.

---

## The Mental Model in One Sentence

An agent is an LLM with a goal, tools to act on the world, and a loop that keeps going until the goal is met or it gives up — and your job as the engineer is to decide what it can do, what it can see, and where a human needs to stay in the loop.

Get that right and agentic AI is genuinely powerful. Get it wrong and you have a system that confidently does the wrong thing with no easy way to stop it.

---

*Day 3 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [AI in the SDLC — The Honest State of Things in 2026](/posts/ai-in-the-sdlc-the-honest-state-of-things-in-2026/).*
