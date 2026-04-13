---
layout: post
title: "What Makes a Good Coding Agent — Design Principles"
date: 2026-04-29
categories: [ai, coding-agents]
tags: [coding-agents, agentic-ai, agent-skills, automation, claude-code]
description: "Arc 4 starts here. Coding agents that work in production aren't just LLMs with tool access — they're systems designed around specific principles: scoped action spaces, recovery from failure, human checkpoints, and evaluations that reflect real use."
author: akashtalole
---

Arc 4 starts today. For the next six days I'm covering coding agents — what they are, how to build them, and what I've learned from building them in real enterprise contexts.

I want to start with design principles rather than implementation, because the design mistakes are what cause coding agents to fail in production. Getting the implementation right on a poorly designed agent just means you've reliably built the wrong thing.

---

## The Design Problem With Coding Agents

A coding agent is an LLM with access to tools that operate on code: reading files, writing files, running tests, executing commands, calling APIs. Given a goal, it plans a series of actions, executes them, observes the results, and continues until the goal is met.

The design challenge: the agent operates in a real environment with real consequences. A file it writes incorrectly will be wrong until someone fixes it. A test it runs might have side effects. A command it executes might affect running systems. Unlike a chat interface where the output is text you can ignore, agents take actions in the world.

This means the design choices that feel like optimisations — how much access to give it, how much autonomy to allow, where to require human confirmation — are actually safety and reliability decisions.

---

## Principle 1: Scope the Action Space to the Actual Task

The most important design decision is what tools the agent has access to.

More tools is not better. More tools means more ways for the agent to go wrong, more attack surface for prompt injection, and more blast radius when something fails. The right action space is the minimum required to accomplish the task reliably.

A coding agent designed to write and run unit tests needs: read files, write test files, run the test suite. It does not need: write to production code files, access external APIs, execute arbitrary shell commands, push to git.

Map out the task. Identify the minimum set of operations required. Give the agent exactly those operations. Everything else is risk with no return.

This isn't just a security principle — it's a reliability principle. Agents with constrained action spaces make simpler plans. Simpler plans have fewer failure modes. Fewer failure modes means more predictable behaviour.

---

## Principle 2: Design for Failure, Not Just Success

Coding agents will fail. They'll misunderstand the task, hit unexpected states in the codebase, encounter errors they don't know how to handle, and sometimes make changes that create new problems.

Design for this from the start:

**Structured error handling in every tool.** When a tool call fails, the agent needs enough information to decide what to do next: retry, try a different approach, or stop and ask for help. "Error: something went wrong" is not actionable. "Error: file not found at path /src/auth/middleware.py — the file was recently moved to /src/middleware/auth.py" gives the agent something to work with.

**Recovery strategies, not just failure modes.** For each tool, define what the agent should do when it fails. Can it retry? Should it try an alternative approach? Should it stop and report? Build these decision points into the agent's system prompt and tool descriptions.

**Defined stopping conditions.** When should the agent give up rather than continuing? An agent without clear stopping conditions will sometimes loop indefinitely, making the same mistake repeatedly. Define: maximum number of retries, maximum number of steps, conditions under which the agent should stop and escalate to a human.

**State that survives interruption.** For long-running tasks, the agent should be able to describe its current state clearly if interrupted. You should be able to look at the agent's output at any point and understand what it's done and what it was planning to do next.

---

## Principle 3: Human-in-the-Loop Checkpoints at the Right Granularity

Fully autonomous and fully manual are both wrong for most coding agent use cases. The right answer is strategic human checkpoints.

Too much autonomy: the agent makes consequential decisions without review and you find out about problems after the fact.

Too much human involvement: the agent has to ask permission for every step, defeating the purpose of having an agent.

The principle: require human review before irreversible or high-blast-radius actions. Let the agent act autonomously for reversible, low-blast-radius operations.

In practice:
- Reading files: no checkpoint needed
- Writing to non-critical paths in a branch: no checkpoint needed
- Running tests: no checkpoint needed
- Writing to shared configuration: checkpoint before executing
- Any action outside the defined scope of the task: always checkpoint
- Pushing to any branch: checkpoint before executing

The checkpoint granularity should be calibrated to your risk tolerance and the maturity of the specific agent. New agents get more checkpoints. Well-tested agents with a track record in production get fewer.

---

## Principle 4: Observability Is Not Optional

A coding agent that you can't observe is an agent you can't debug, improve, or trust.

At minimum, log:
- Every tool call with inputs and outputs
- Every reasoning step (the agent's internal monologue, if your framework exposes it)
- Every decision point and which path was taken
- Every error and how it was handled
- The final state and how the task was resolved

This isn't just for debugging failures — it's for understanding successes. When an agent solves a problem in an unexpected way, you want to know how it did it. When it makes a decision you wouldn't have made, you want to understand why.

Build structured, searchable logging from day one. The logs you need when something goes wrong are the ones you decided to collect before anything went wrong.

---

## Principle 5: Evaluate on Real Tasks, Not Toy Examples

The most dangerous phase of coding agent development is when the agent works well on demo tasks and you start trusting it for real tasks without adequate evaluation.

Demo tasks have clean boundaries, well-understood solutions, and success criteria that are easy to verify. Real tasks don't. They have ambiguous requirements, edge cases specific to your codebase, and success criteria that require domain knowledge to evaluate.

Before deploying a coding agent for a class of real tasks, evaluate it on a set of representative examples — tasks drawn from your actual backlog, with real code, real constraints, and real success criteria. Not curated examples, but representative ones.

Track:
- Task completion rate (did it produce a working solution?)
- Correctness rate (did the solution do the right thing, not just a plausible thing?)
- Revision rate (how often does the human reviewer need to make significant changes?)
- Failure modes (when it fails, how does it fail?)

Set a bar. Deploy when the agent meets it consistently. Don't deploy because it looks good in demos.

---

## The Summary

A good coding agent is not the most capable agent. It's the most reliable agent for the specific tasks you've designed it for. That means:

- Action space scoped to the task
- Failure modes that are recoverable rather than silent
- Human checkpoints where the stakes are high
- Observability that lets you understand what happened
- Evaluation on real tasks before real deployment

Get these right and the implementation details — which framework, which model, which tools — become secondary. Get them wrong and no amount of implementation sophistication will make the agent trustworthy.

---

*Day 20 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [GitHub Copilot in Enterprise Governance Frameworks](/posts/github-copilot-in-enterprise-governance-frameworks/).*
