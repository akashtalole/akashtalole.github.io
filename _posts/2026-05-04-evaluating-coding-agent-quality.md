---
layout: post
title: "Evaluating Coding Agent Quality — Beyond 'Did It Run?'"
date: 2026-05-04
categories: [ai, coding-agents]
tags: [coding-agents, agentic-ai, agent-skills, automation, sdlc]
description: "An agent that runs without crashing is not the same as an agent that works. Evaluating coding agent quality properly — what to measure, how to build an eval suite, and what the numbers actually tell you."
author: akashtalole
---

The hardest question in coding agent development isn't how to build one — it's how to know if it's good enough to trust.

"Did it run?" is not sufficient. "Did the tests pass?" is not sufficient. I've seen agents that run cleanly, pass all tests, and produce code that's wrong in ways that only become visible in production. The evaluation problem in agentic systems is genuinely hard, and most teams solve it poorly.

Here's how I approach it.

---

## Why Standard Testing Doesn't Work

Unit tests verify deterministic behaviour. You call a function with specific inputs and assert on specific outputs. For coding agents, neither condition holds.

**Non-deterministic.** The same prompt with the same inputs can produce different outputs across runs. A single test case that passes is not a reliable signal — the agent might produce a correct result one in three times.

**Context-dependent.** Agent quality depends heavily on the task, the codebase, the specifics of the problem. An agent that works well on authentication code might work poorly on data processing code. Task-specific evaluation is necessary.

**Multi-dimensional.** Did the agent produce code that runs? Code that's correct? Code that follows conventions? Code that handles edge cases? These are distinct quality dimensions, and an agent can score well on some while failing others.

The right framework for coding agent evaluation is adapted from machine learning: evaluate over a distribution of representative tasks, across multiple quality dimensions, with multiple samples per task.

---

## The Evaluation Dimensions

I evaluate coding agents across five dimensions:

**1. Task completion rate** — Did the agent produce a result that addresses the stated goal? Binary, but requires human judgement to score accurately. The output doesn't have to be perfect; it has to be a reasonable attempt at the task.

**2. Correctness rate** — Of completed tasks, what fraction produced results that are actually correct? This requires either automated verification (run tests, check against a ground truth) or human review.

**3. Convention adherence** — Does the output follow the codebase's existing patterns, naming conventions, and style? An agent that produces correct code in the wrong style imposes revision cost.

**4. Scope discipline** — Did the agent stay within the defined scope of the task, or did it make unrequested changes? Scope creep in an agent is a reliability problem.

**5. Revision rate** — How much human editing does the output require before it's usable? The goal isn't zero revision — it's revision time less than time to write from scratch.

---

## Building an Eval Suite

An eval suite for a coding agent is a set of representative tasks with known correct answers (or at least known quality criteria).

**Collect tasks from real work.** The most useful eval tasks come from your actual backlog — issues, PRs, bug reports. Curated "easy" tasks don't reveal real failure modes. Representative tasks do.

**Include a range of difficulty.** Easy tasks that should always pass, medium tasks that test core capabilities, hard tasks that reveal the agent's limits. The distribution should reflect the actual distribution of work you'd use the agent for.

**Define acceptance criteria, not just correct answers.** For many coding tasks, there's no single correct answer — there are acceptable and unacceptable answers. Define what makes an answer acceptable (runs tests, follows conventions, addresses the stated requirement) rather than trying to specify the exact correct output.

```python
eval_suite = [
    {
        "id": "add_null_check_001",
        "task": "Add null/None validation to the `process_order` function in src/orders.py",
        "difficulty": "easy",
        "acceptance_criteria": [
            "Function raises ValueError or similar for None input",
            "Existing tests still pass",
            "No changes to function signature"
        ],
        "tags": ["validation", "error-handling"]
    },
    {
        "id": "refactor_auth_middleware",
        "task": "Refactor the auth middleware to support both JWT and API key authentication",
        "difficulty": "medium",
        "acceptance_criteria": [
            "Both auth methods work in integration tests",
            "Backwards compatible with existing JWT usage",
            "Follows existing middleware pattern"
        ],
        "tags": ["refactor", "auth"]
    }
]
```

---

## Running Evaluations Properly

Single-run evaluations are unreliable. Run each task three to five times and aggregate the results.

```python
def evaluate_agent(agent, eval_suite, runs_per_task=3):
    results = {}

    for task in eval_suite:
        task_results = []

        for run in range(runs_per_task):
            result = agent.execute(task["task"])
            score = score_result(result, task["acceptance_criteria"])
            task_results.append(score)

        results[task["id"]] = {
            "pass_rate": sum(task_results) / len(task_results),
            "runs": task_results,
            "difficulty": task["difficulty"],
            "tags": task["tags"]
        }

    return aggregate_results(results)
```

The pass rate across runs gives you a more reliable signal than a single pass/fail. An agent with a 100% pass rate on single runs might have a 60% pass rate over multiple runs — which tells you something very different about reliability.

---

## What the Numbers Tell You

**Overall pass rate by difficulty.** Easy tasks should have near-100% pass rates. If they don't, the agent has fundamental capability gaps. Medium tasks at 70-80% are reasonable for a production agent. Hard tasks below 50% may still be useful as "assisted" tasks rather than autonomous ones.

**Performance by tag.** Breaking down results by task type reveals where the agent is reliable and where it isn't. An agent might be excellent at adding validation (90% pass rate) and poor at refactoring (40% pass rate). That tells you where to deploy it autonomously and where to use it as an assistant.

**Revision rate trend.** Track revision rate over time. As you improve the agent, revision rate should decrease. If it isn't decreasing, your improvements aren't translating to real-world quality.

**Failure mode clustering.** When the agent fails, why? Cluster failure modes: wrong approach, scope creep, convention violations, incomplete implementation, incorrect logic. Clustered failures point to specific things to fix in the system prompt or tool design.

---

## Deploying Based on Eval Results

The eval results tell you where to trust the agent and where not to.

For tasks where pass rate is consistently above 85% with low revision rate: safe to deploy with light-touch human review.

For tasks where pass rate is 60-85%: deploy with structured human review — the agent produces a first draft, a human verifies and completes.

Below 60%: use as a thinking partner or starting point, not an autonomous executor. The agent might still be useful, just not trustworthy enough to run unattended.

Don't deploy uniformly across all task types. Deploy selectively based on where the eval data shows the agent is reliable.

---

## The Honest Reality

Building a rigorous eval suite is work. Running evals takes time. Most teams skip it or do it superficially. This is why so many coding agents that work in demos fail in production — the demo tasks are curated for the agent's strengths, not representative of real work.

The teams that get the most value from coding agents are the ones who invest in evaluation. They know where their agent is reliable. They deploy it in those areas with confidence. They improve it based on real failure data rather than intuition.

That's the difference between a demo and a production system.

---

*Day 25 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [Multi-Step Coding Agents — Patterns and Failure Modes](/posts/multi-step-coding-agents-patterns-and-failure-modes/).*
