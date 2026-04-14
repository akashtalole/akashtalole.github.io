---
layout: post
title: "Tool Use in AI Agents — Patterns That Work in Production"
date: 2026-05-01
categories: [ai, coding-agents]
tags: [coding-agents, agentic-ai, agent-skills, tool-use, automation]
description: "Function calling works in demos. In production it breaks in ways demos don't show. The tool use patterns that hold up under real load — retry logic, result validation, tool chaining, and error handling the agent can actually reason about."
author: akashtalole
---

Yesterday I showed you how to build a basic coding agent with tool use. Today I want to go deeper on the tool use patterns themselves — specifically the ones that distinguish agents that work reliably in production from the ones that work in demos and then silently fail.

The gap between demo and production tool use is larger than most people expect. Here's where it lives.

---

## Pattern 1: Structured Results Over Raw Strings

The single highest-leverage improvement you can make to tool reliability is returning structured results instead of raw strings.

When a tool returns unstructured text, the agent has to parse meaning from it. Parsing introduces failure modes: the agent misinterprets the result, acts on an incorrect understanding, and the error is invisible because nothing raised an exception.

**Raw string (fragile):**
```python
def search_code(query):
    results = run_search(query)
    return f"Found {len(results)} results: {', '.join(r['file'] for r in results)}"
```

**Structured result (robust):**
```python
def search_code(query):
    results = run_search(query)
    return {
        "query": query,
        "count": len(results),
        "results": [
            {"file": r["file"], "line": r["line"], "snippet": r["snippet"]}
            for r in results
        ]
    }
```

The structured version gives the agent discrete, unambiguous fields it can reference in subsequent reasoning. The agent knows `result["count"]` is the number of results. It doesn't have to parse "Found 3 results" and hope the format doesn't change.

---

## Pattern 2: Always Return a Result — Even on Failure

Tools that raise exceptions or return `None` on failure create dead ends in the agent loop. The agent calls the tool, gets an unhelpful result, and has no basis for deciding what to do next.

The pattern: every tool returns a result. On success, a meaningful result. On failure, a structured error with enough information for the agent to recover.

```python
def read_file(path):
    try:
        if not os.path.exists(path):
            return {
                "success": False,
                "error": "file_not_found",
                "path": path,
                "suggestion": "Check the path is correct. Use list_files to find available files."
            }
        with open(path) as f:
            content = f.read()
        return {
            "success": True,
            "path": path,
            "content": content,
            "size_bytes": len(content.encode())
        }
    except PermissionError:
        return {
            "success": False,
            "error": "permission_denied",
            "path": path
        }
    except Exception as e:
        return {
            "success": False,
            "error": "unexpected_error",
            "message": str(e),
            "path": path
        }
```

The `suggestion` field in the `file_not_found` case is worth noting — it tells the agent what to try next. You're pre-emptively answering the agent's next question.

---

## Pattern 3: Idempotent Tools Where Possible

An idempotent tool produces the same result whether you call it once or ten times. Read operations are naturally idempotent. Write operations often aren't.

Design write tools to be idempotent where you can:

```python
def write_file(path, content):
    # Check if content already matches — idempotent
    if os.path.exists(path):
        with open(path) as f:
            existing = f.read()
        if existing == content:
            return {"success": True, "action": "no_change", "path": path}

    with open(path, "w") as f:
        f.write(content)
    return {"success": True, "action": "written", "path": path}
```

When an agent retries a write (due to uncertainty about whether the first attempt succeeded), an idempotent tool handles it cleanly. A non-idempotent tool creates duplicates, overwrites, or appends incorrectly.

---

## Pattern 4: Tool Chaining With Explicit Dependencies

Complex tasks require sequences of tool calls where each depends on the previous. The agent needs to understand these dependencies explicitly — not infer them.

Document dependencies in tool descriptions:

```
"description": "Run the test suite for a specific test file. 
IMPORTANT: The file must exist before running tests — use 
read_file to verify the file exists first. Returns test 
results including pass/fail counts and failure details."
```

Making dependencies explicit in descriptions reduces the rate at which agents try to run tools out of order, which is a common failure mode in complex multi-step tasks.

---

## Pattern 5: Rate Limiting and Retry Logic

Production environments have rate limits. External APIs throttle. File systems have I/O constraints. An agent that hits a rate limit and crashes the loop is an agent that doesn't work reliably.

Build retry logic at the tool level, not the agent level:

```python
import time
from functools import wraps

def with_retry(max_attempts=3, delay=1.0):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_attempts):
                result = func(*args, **kwargs)
                if result.get("success") or result.get("error") != "rate_limited":
                    return result
                if attempt < max_attempts - 1:
                    time.sleep(delay * (2 ** attempt))  # exponential backoff
            return result
        return wrapper
    return decorator

@with_retry(max_attempts=3)
def call_external_api(endpoint, params):
    # ... API call implementation
    pass
```

The agent doesn't need to know about rate limits and retries — that's an implementation detail. It calls the tool and gets a result. The retry logic lives transparently in the tool.

---

## Pattern 6: Tool Result Validation

Before passing a tool result to the agent, validate it. Malformed results — truncated files, partial API responses, encoding errors — can cause the agent to reason incorrectly and produce confident-looking wrong outputs.

```python
def read_file(path):
    try:
        with open(path) as f:
            content = f.read()

        # Validate result before returning
        if not isinstance(content, str):
            return {"success": False, "error": "invalid_content_type"}
        if len(content) > 100_000:
            # Truncate with warning rather than silently
            content = content[:100_000]
            truncated = True
        else:
            truncated = False

        return {
            "success": True,
            "content": content,
            "truncated": truncated,
            "size_chars": len(content)
        }
    except Exception as e:
        return {"success": False, "error": str(e)}
```

When a file is truncated, the agent knows — it can reason about whether the missing content matters or decide to read the file in chunks.

---

## The Anti-Patterns to Avoid

**Tools that do too much.** A `manage_project` tool that creates, updates, and deletes based on a parameter is harder for the agent to use correctly than three separate tools. Separate concerns.

**Silent failures.** Tools that return empty results on failure with no indication of what went wrong force the agent to guess. Always return failure information.

**Mutable global state in tools.** Tools that maintain internal state between calls make the agent's behaviour dependent on call order in ways that are hard to reason about. Keep tools stateless.

**Overly long tool descriptions.** Descriptions longer than 2-3 sentences reduce the signal-to-noise ratio. The agent has to parse more text to find the relevant information. Be concise.

---

## The Principle

Tool use reliability comes from the tools, not the model. The model is fixed — you can't make it smarter at using poorly designed tools. But you can design tools that are easy to use correctly: structured results, explicit errors, idempotent writes, clear descriptions, sensible retry logic.

Invest in the tool layer. That's where production reliability lives.

---

*Day 22 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [Building Your First Coding Agent](/posts/building-your-first-coding-agent-a-practical-walkthrough/).*
