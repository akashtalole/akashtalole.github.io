---
layout: post
title: "Building Your First Coding Agent — A Practical Walkthrough"
date: 2026-04-30
categories: [ai, coding-agents]
tags: [coding-agents, agentic-ai, agent-skills, automation, claude-code]
description: "Stop reading about coding agents and build one. A practical walkthrough of a real, useful coding agent — the decisions made along the way, the things that broke, and what the finished version looks like."
author: akashtalole
---

Yesterday I covered the design principles for coding agents. Today I want to get concrete: let's actually build one.

Not a toy demo. A useful agent that solves a real problem — one I've built and use in my own workflow. I'll walk through the decisions, the things that broke during development, and what the finished version looks like.

---

## The Task: A Test Gap Finder Agent

The agent I'm going to walk through does one thing: given a Python module, it finds functions that have no corresponding tests and reports them with enough context to write those tests.

This is useful because test coverage metrics lie. 80% coverage tells you lines are executed, not that behaviour is tested. The gap finder finds the specific functions with no test coverage at all — the easiest wins.

The agent needs to:
1. Read the source module and identify all functions
2. Read the test files and identify which functions are tested
3. Compare the two and report the gaps
4. For each gap, provide enough context to write a test

Simple enough to build in a day. Real enough to use in production.

---

## Step 1: Define the Tools

Following the design principle from Day 20: define the minimum action space required. For this agent:

```python
tools = [
    {
        "name": "read_file",
        "description": "Read the contents of a file at the given path. Returns the file content as a string. Use this to read source files and test files.",
        "input_schema": {
            "type": "object",
            "properties": {
                "path": {
                    "type": "string",
                    "description": "Absolute or relative path to the file"
                }
            },
            "required": ["path"]
        }
    },
    {
        "name": "list_files",
        "description": "List all files in a directory matching a glob pattern. Returns a list of file paths.",
        "input_schema": {
            "type": "object",
            "properties": {
                "directory": {"type": "string"},
                "pattern": {
                    "type": "string",
                    "description": "Glob pattern, e.g. '**/*.py'"
                }
            },
            "required": ["directory", "pattern"]
        }
    }
]
```

Two tools. Read-only. No write access, no execution. The blast radius of a mistake is zero — the agent can look, not touch.

---

## Step 2: Implement the Tool Handlers

```python
import glob
import os

def handle_tool_call(tool_name, tool_input):
    if tool_name == "read_file":
        path = tool_input["path"]
        if not os.path.exists(path):
            return {"error": f"File not found: {path}"}
        with open(path, "r") as f:
            return {"content": f.read(), "path": path}

    elif tool_name == "list_files":
        directory = tool_input["directory"]
        pattern = tool_input.get("pattern", "**/*.py")
        full_pattern = os.path.join(directory, pattern)
        files = glob.glob(full_pattern, recursive=True)
        return {"files": files, "count": len(files)}

    return {"error": f"Unknown tool: {tool_name}"}
```

Notice the structured error returns. When a file isn't found, the agent gets a message it can reason about — not a Python exception that crashes the loop.

---

## Step 3: The Agent Loop

```python
import anthropic

client = anthropic.Anthropic()

def run_agent(source_path, test_directory):
    system_prompt = """You are a test coverage analyst. Your job is to identify 
functions in a Python source file that have no corresponding tests.

For each function with no tests, report:
- The function name and signature
- What it does (from the docstring or code)
- What test cases would be most valuable

Be specific. Do not report functions that are clearly tested."""

    messages = [
        {
            "role": "user",
            "content": f"Find untested functions in {source_path}. Test files are in {test_directory}."
        }
    ]

    while True:
        response = client.messages.create(
            model="claude-opus-4-6",
            max_tokens=4096,
            system=system_prompt,
            tools=tools,
            messages=messages
        )

        # Add assistant response to history
        messages.append({"role": "assistant", "content": response.content})

        # If no tool calls, we're done
        if response.stop_reason == "end_turn":
            # Extract final text response
            for block in response.content:
                if hasattr(block, "text"):
                    return block.text
            break

        # Process tool calls
        tool_results = []
        for block in response.content:
            if block.type == "tool_use":
                result = handle_tool_call(block.name, block.input)
                tool_results.append({
                    "type": "tool_result",
                    "tool_use_id": block.id,
                    "content": str(result)
                })

        # Add tool results and continue loop
        if tool_results:
            messages.append({"role": "user", "content": tool_results})
```

The loop is straightforward: send messages, check if done, process tool calls, add results, repeat. This is the core pattern for any tool-using agent with the Anthropic API.

---

## What Broke During Development

**First problem: the agent read too many files.** Without constraints, it tried to read every Python file in the project — which hit rate limits and took forever. Fix: narrow the task in the prompt. "Limit your search to the test directory provided — don't explore the whole project."

**Second problem: false positives.** The agent reported functions as untested when they were tested indirectly through integration tests. Fix: add a note in the system prompt acknowledging this limitation. "Note: you can only detect direct test references, not indirect coverage through integration tests."

**Third problem: vague output.** Early versions reported "function X has no tests" without enough context to act on. Fix: update the prompt to require the structured output format (function name, signature, purpose, suggested test cases).

Each of these was a prompt or tool design fix, not a code fix. That's the pattern: when an agent misbehaves, look at the system prompt and tool descriptions before touching the implementation.

---

## Using It

```python
report = run_agent(
    source_path="src/orders/processor.py",
    test_directory="tests/"
)
print(report)
```

Output looks like:

```
## Untested Functions Found

### `calculate_shipping_cost(order, destination, express=False)`
**What it does:** Calculates shipping cost based on order weight and destination zone.
**Suggested tests:**
- Standard domestic shipping at different weight thresholds
- Express vs standard rate comparison
- International destination handling
- Edge case: zero-weight order
```

Actionable. Takes ten seconds to run. I run it on every module before closing a sprint.

---

## The Broader Pattern

Every coding agent follows this same structure:

1. **Define the minimum tools** the task requires
2. **Write structured error returns** so the agent can recover
3. **Write a precise system prompt** that constrains scope and output format
4. **Build the observe-reason-act loop** with message history
5. **Test it on real cases, fix the misbehaviours in prompt and tool design**

The implementation is the easy part. The design — especially the system prompt and the tool definitions — is where the quality of the agent lives.

---

*Day 21 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [What Makes a Good Coding Agent](/posts/what-makes-a-good-coding-agent-design-principles/).*
