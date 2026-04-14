---
layout: post
title: "Agent Memory and Context Management — The Hard Part"
date: 2026-05-02
categories: [ai, coding-agents]
tags: [coding-agents, agentic-ai, agent-skills, automation, claude-code]
description: "Memory is where agentic systems get subtle and hard to debug. Context windows fill up. Conversation history grows. The agent starts forgetting, confusing, or contradicting itself. Here's how to design memory properly from the start."
author: akashtalole
---

If you've built a basic agent and run it on longer tasks, you've hit the memory problem. The agent starts well, but somewhere in the middle it seems to forget what it was doing. It re-reads files it already processed. It contradicts conclusions it reached earlier. It loses track of progress.

This isn't a model quality issue. It's a context management issue — and it's entirely solvable if you design for it from the start.

---

## Why Memory Is Hard

An LLM agent's "memory" is its context window: the messages, tool results, and reasoning that fit in the current conversation. When the context fills up, earlier content gets truncated or compressed. The agent can no longer reference things it processed earlier.

For short tasks this doesn't matter. For longer tasks — analysing a large codebase, executing a multi-hour workflow, accumulating results across many tool calls — it becomes the primary reliability bottleneck.

There are three distinct memory problems:

1. **Working memory** — keeping track of the current task state
2. **Short-term memory** — results from earlier in the current session
3. **Long-term memory** — knowledge that should persist across sessions

Each needs a different approach.

---

## Working Memory: Explicit Task State

The most common failure mode in coding agents is losing track of progress mid-task. The agent finishes analysing three files, then a few tool calls later can't remember which three it processed.

The fix: make task state explicit. At each step, maintain a structured state object that the agent can reference and update.

```python
# Initial task state
task_state = {
    "goal": "Find all functions missing docstrings in the src/ directory",
    "files_to_process": ["src/auth.py", "src/orders.py", "src/payments.py"],
    "files_processed": [],
    "findings": [],
    "status": "in_progress"
}
```

Include the current state in the system prompt or as a recurring context injection:

```python
def build_system_prompt(task_state):
    return f"""You are analysing a Python codebase for missing docstrings.

Current task state:
- Files remaining: {task_state['files_to_process']}
- Files completed: {task_state['files_processed']}
- Findings so far: {len(task_state['findings'])} functions flagged

Process one file at a time. After processing each file, the state will be updated."""
```

When the agent completes each step, update the state programmatically. The agent always knows where it is.

---

## Short-Term Memory: Summarise, Don't Accumulate

Raw tool results accumulate fast. A tool that reads a 500-line file produces 500 lines of context. Do that ten times and you've used most of your context window on file contents that aren't all relevant anymore.

The pattern: summarise tool results before adding them to the conversation history. Instead of keeping the full file content, keep a structured summary of what was found.

```python
def summarise_file_analysis(file_path, raw_content, findings):
    """Replace raw file content with a structured summary in conversation history."""
    return {
        "file": file_path,
        "lines": len(raw_content.split("\n")),
        "functions_found": [f["name"] for f in findings],
        "issues": findings,
        "processed": True
    }
```

After processing a file, replace the raw content in the message history with the summary. The agent retains the findings without accumulating the full content.

---

## Managing the Context Window Actively

Don't wait for the context to fill up. Manage it proactively as part of the agent loop.

```python
def estimate_tokens(messages):
    # Rough estimate: 1 token per 4 characters
    total_chars = sum(len(str(m)) for m in messages)
    return total_chars // 4

def trim_conversation_history(messages, max_tokens=50000):
    """Keep system message and recent messages, summarise the middle."""
    if estimate_tokens(messages) < max_tokens:
        return messages

    # Always keep: system message, last N messages
    system = [m for m in messages if m.get("role") == "system"]
    recent = messages[-10:]  # Keep last 10 exchanges
    middle = messages[len(system):-10]

    # Summarise the middle section
    summary = summarise_messages(middle)  # Your summarisation logic
    summary_message = {
        "role": "user",
        "content": f"[Earlier conversation summary: {summary}]"
    }

    return system + [summary_message] + recent
```

Called at the start of each loop iteration, this keeps the context window from silently overflowing.

---

## Long-Term Memory: External Storage

For knowledge that needs to persist across sessions — previous findings, learned patterns, accumulated context — the context window isn't the right place. Use external storage.

The simplest approach: write findings to a structured file as you go.

```python
import json
from datetime import datetime

class AgentMemory:
    def __init__(self, memory_file):
        self.memory_file = memory_file
        self.load()

    def load(self):
        try:
            with open(self.memory_file) as f:
                self.data = json.load(f)
        except FileNotFoundError:
            self.data = {"sessions": [], "findings": {}, "patterns": []}

    def save(self):
        with open(self.memory_file, "w") as f:
            json.dump(self.data, f, indent=2, default=str)

    def record_finding(self, category, finding):
        if category not in self.data["findings"]:
            self.data["findings"][category] = []
        self.data["findings"][category].append({
            "finding": finding,
            "timestamp": datetime.now().isoformat()
        })
        self.save()

    def get_relevant_context(self, task_description):
        """Return previously recorded findings relevant to current task."""
        # Simple keyword matching — can be replaced with embeddings
        relevant = []
        for category, findings in self.data["findings"].items():
            if any(word in task_description.lower()
                   for word in category.lower().split()):
                relevant.extend(findings[-5:])  # Last 5 from relevant categories
        return relevant
```

At the start of each session, inject relevant previous findings into the system prompt. The agent builds on previous work rather than starting from scratch.

---

## The Episodic Memory Pattern

For complex multi-session agents, organise memory episodically: each session has a summary, and the agent starts new sessions with relevant previous episode summaries rather than raw history.

```python
def start_new_session(memory, task):
    relevant_history = memory.get_relevant_context(task)

    if relevant_history:
        context = f"""Previous relevant work:
{json.dumps(relevant_history, indent=2)}

Build on this context for the current task."""
    else:
        context = "No relevant previous work found."

    return context
```

The agent gets the benefit of institutional memory without the cost of replaying every previous conversation.

---

## The Principle

Memory management is context window management. Agents that work for five minutes of a task and then lose coherence almost always have unmanaged context growth at the root.

Design your memory strategy before you build the agent:
- What state needs to be maintained per step? (working memory)
- What results need to be retained vs. summarised? (short-term)
- What knowledge needs to persist across sessions? (long-term)

Answer those three questions and your agent will be coherent across tasks of arbitrary length.

---

*Day 23 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [Tool Use in AI Agents — Patterns That Work in Production](/posts/tool-use-in-ai-agents-patterns-that-work-in-production/).*
