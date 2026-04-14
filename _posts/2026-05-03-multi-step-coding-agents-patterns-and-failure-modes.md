---
layout: post
title: "Multi-Step Coding Agents — Patterns and Failure Modes"
date: 2026-05-03
categories: [ai, coding-agents]
tags: [coding-agents, agentic-ai, agent-skills, automation, claude-code]
description: "Single-step agents are relatively predictable. Multi-step agents are where things get interesting — and where they fall apart in ways that are hard to diagnose. The orchestration patterns that hold up and the failure modes that will find you eventually."
author: akashtalole
---

A single-step agent calls one tool, gets a result, produces an output. Predictable, easy to test, easy to debug.

Multi-step agents chain multiple operations — read, analyse, write, test, repeat — toward a goal that requires several sequential decisions. These are where the real engineering value lives, and where the real engineering problems live.

Today I want to cover the orchestration patterns that work and the failure modes I've run into building multi-step agents in production.

---

## The Basic Multi-Step Pattern

A multi-step agent is just the basic agent loop running longer. But the longer it runs, the more important it becomes to structure the execution deliberately.

The pattern I've converged on: **plan, then execute in phases**.

```
Phase 1: Understand — gather context, read relevant files, identify scope
Phase 2: Plan — produce an explicit plan of what changes to make
Phase 3: Execute — make changes one at a time, verifying after each
Phase 4: Validate — run tests, check correctness, report results
```

Making these phases explicit in the system prompt prevents the most common multi-step failure: the agent jumping to execution before it has enough context, making changes based on incomplete understanding, and then struggling to recover.

```python
system_prompt = """You work in four phases:

UNDERSTAND: Read relevant files before making any changes.
PLAN: Write out what changes you'll make and why before making them.
EXECUTE: Make one change at a time. Verify each change before the next.
VALIDATE: Run tests after all changes. Report what passed and what failed.

Always complete UNDERSTAND and PLAN before beginning EXECUTE.
Do not skip phases."""
```

Explicit phase structure is more reliable than hoping the agent naturally sequences operations correctly.

---

## Pattern: Checkpoint Before Irreversible Steps

Some steps in a multi-step agent are reversible — reading files, running tests, writing to a branch. Others aren't — committing, pushing, deploying, sending messages.

Before any irreversible step, insert a checkpoint: a point where the agent summarises what it's about to do and your code decides whether to proceed.

```python
def should_proceed(action_description, auto_approve_threshold=0.9):
    """Return True to proceed, False to pause for human review."""

    # For fully automated pipelines with high-confidence tasks
    if confidence_score(action_description) >= auto_approve_threshold:
        log_action(action_description, approved_by="auto")
        return True

    # For human-in-the-loop workflows
    print(f"\nAgent wants to: {action_description}")
    response = input("Proceed? (y/n): ").strip().lower()
    return response == "y"
```

In practice, most multi-step coding agents I build have three tiers:
- **Auto-approve**: read operations, test runs, writing to designated output paths
- **Log-and-proceed**: write operations within defined scope
- **Require confirmation**: anything outside defined scope, commits, API calls with side effects

---

## Pattern: Atomic Steps With Verification

Each execution step should be atomic — one discrete change — followed by a verification that the change had the expected effect.

```python
def execute_with_verification(agent, step):
    """Execute a step and verify it succeeded before continuing."""

    # Execute the step
    result = agent.execute_step(step)

    if not result["success"]:
        return {
            "success": False,
            "step": step,
            "error": result["error"],
            "action": "retry_or_skip"
        }

    # Verify the change had the expected effect
    verification = agent.verify_step(step, result)

    if not verification["expected_state_reached"]:
        return {
            "success": False,
            "step": step,
            "actual_state": verification["actual_state"],
            "expected_state": step["expected_state"],
            "action": "investigate"
        }

    return {"success": True, "step": step, "result": result}
```

Without verification, a failed step looks like a succeeded step until much later when something downstream breaks. Atomic steps with immediate verification surfaces failures at the point where they're cheapest to fix.

---

## Failure Mode: The Cascade

The cascade failure: step 3 fails silently (returns a plausible-looking but wrong result), step 4 succeeds based on the wrong output of step 3, step 5 succeeds based on the wrong output of step 4, and by the time the failure becomes visible it's deeply entangled.

Prevention: the verification pattern above. Detection: structured logging of every step with enough detail to trace backward.

```python
execution_log = []

def log_step(step_number, step_description, result, verification):
    execution_log.append({
        "step": step_number,
        "description": step_description,
        "result_summary": summarise_result(result),
        "verification": verification,
        "timestamp": datetime.now().isoformat()
    })
```

When a cascade failure occurs, the log lets you identify which step first produced wrong output.

---

## Failure Mode: Goal Drift

Goal drift happens when the agent's behaviour across many steps gradually diverges from the original goal. It starts solving the problem as stated, makes some reasonable-looking subgoals, and ends up solving a subtly different problem.

Signs of goal drift: the agent produces a working solution that doesn't match the original requirement, or makes changes you didn't ask for because they seemed related.

Prevention: restate the goal periodically. After every N steps or every phase boundary, inject a reminder:

```python
def build_messages_with_goal_reminder(original_goal, messages, reminder_interval=5):
    """Insert goal reminders every N messages."""
    result = []
    for i, message in enumerate(messages):
        if i > 0 and i % reminder_interval == 0:
            result.append({
                "role": "user",
                "content": f"[Reminder: The original goal is: {original_goal}. Ensure your current work serves this goal directly.]"
            })
        result.append(message)
    return result
```

---

## Failure Mode: Infinite Retry Loops

An agent that can't complete a step will sometimes retry indefinitely. Each retry generates more context. The context window fills. Performance degrades. The agent loops.

Fix: explicit retry limits and escalation paths.

```python
MAX_RETRIES_PER_STEP = 3

def execute_step_with_limits(agent, step, step_number):
    for attempt in range(MAX_RETRIES_PER_STEP):
        result = execute_with_verification(agent, step)
        if result["success"]:
            return result

        if attempt < MAX_RETRIES_PER_STEP - 1:
            # Brief recovery prompt before retry
            agent.inject_message(
                f"Step {step_number} attempt {attempt + 1} failed: {result['error']}. "
                f"Try a different approach."
            )

    # All retries exhausted — escalate
    return {
        "success": False,
        "step": step_number,
        "escalated": True,
        "reason": f"Failed after {MAX_RETRIES_PER_STEP} attempts",
        "last_error": result["error"]
    }
```

When a step exceeds its retry limit, it escalates to a human rather than continuing to loop.

---

## The Orchestration Principle

Multi-step agents don't fail because of weak models. They fail because of weak orchestration — no phase structure, no verification, no retry limits, no goal reminders, no checkpoints before irreversible actions.

The model is capable. The engineering challenge is building the scaffolding that keeps it on track across many steps. That scaffolding is your job.

---

*Day 24 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [Agent Memory and Context Management](/posts/agent-memory-and-context-management/).*
