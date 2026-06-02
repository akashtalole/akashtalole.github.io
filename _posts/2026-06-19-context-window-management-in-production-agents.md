---
layout: post
title: "Context Window Management in Production Agents"
date: 2026-06-19
categories: [ai, coding-agents]
tags: [agentic-ai, coding-agents, agent-skills, rag]
description: "Context windows are finite even at 100K+ tokens. Long-running agents accumulate state, conversation history, and tool outputs that eventually overflow the window. The strategies that keep production agents working correctly over time."
author: akashtalole
---

Context windows have grown dramatically — 100K, 200K, even 1M tokens for some models. Engineers sometimes assume this makes context management a solved problem.

It doesn't. Long-running agents accumulate state. A customer service agent handling a complex inquiry may go through dozens of tool calls and multi-turn conversation before resolution. An autonomous coding agent working on a large codebase exploration fills its context quickly. And as context fills, model performance degrades — the "lost in the middle" problem, where information in the middle of a long context is less reliably attended to.

Context management is an active concern in any non-trivial agent deployment.

---

## How Agents Fill Their Context

Understanding the accumulation pattern helps design the right management strategy.

**Messages**: user turns and assistant turns accumulate linearly. A 10-turn conversation is 10 message pairs, potentially thousands of tokens.

**Tool call results**: each tool call adds both the call itself and the result to the context. A database query returning 50 rows, a file read of a 1000-line file, a web search returning multiple results — each adds significant tokens.

**Retrieved documents**: RAG results added to context can be large. If you're adding 3 documents × 500 tokens each per query turn, a 20-turn conversation has added 30,000 tokens of retrieved context.

**Reasoning traces**: models that use extended thinking or chain-of-thought produce reasoning tokens that accumulate.

---

## Strategy 1: Conversation Summarisation

As conversation history grows, replace older messages with a summary. The summary preserves the key information without the token cost of the full history.

```python
SUMMARISE_PROMPT = """
Summarise the following conversation history, preserving:
- Key facts and decisions made
- The user's original goal and any sub-goals
- Outstanding action items or questions
- Any context that will be needed for future turns

Conversation:
{history}

Return a concise summary (max 500 tokens).
"""

async def maybe_summarise_history(messages: list, threshold: int = 8000) -> list:
    history_tokens = count_tokens(messages)
    if history_tokens < threshold:
        return messages
    
    # Keep last 3 turns verbatim; summarise everything before
    recent = messages[-6:]  # last 3 turns (user + assistant each)
    older = messages[:-6]
    
    summary = await llm.generate(SUMMARISE_PROMPT.format(
        history=format_messages(older)
    ))
    
    summary_message = {"role": "system", "content": f"[Conversation summary]: {summary}"}
    return [summary_message] + recent
```

**Tradeoff**: the summary loses detail. For most conversational agents, this is acceptable. For agents where precise recall of earlier information matters (complex debugging sessions, multi-document analysis), summarisation may lose critical context.

---

## Strategy 2: Working Memory and External Storage

Separate what needs to be in the context window from what needs to be available on demand.

**In context** (working memory): the current task, recent turns, immediately relevant facts.

**External storage** (retrievable on demand): earlier conversation details, reference documents, intermediate results that may be needed again.

```python
class AgentMemory:
    def __init__(self):
        self.working_memory = []   # Always in context
        self.episodic_store = []   # Retrieved when needed
    
    def add_to_working_memory(self, item: dict):
        self.working_memory.append(item)
        if self.working_memory_tokens() > 4000:
            self._move_oldest_to_episodic()
    
    def _move_oldest_to_episodic(self):
        oldest = self.working_memory.pop(0)
        self.episodic_store.append(oldest)
    
    def retrieve_relevant(self, query: str) -> list:
        # Embed query, search episodic store
        return semantic_search(query, self.episodic_store, top_k=3)
```

This architecture mirrors how humans work: we keep recent context in working memory and retrieve older information when it becomes relevant.

---

## Strategy 3: Selective Tool Result Storage

Tool results are often verbose. A database query returns a full result set; only one or two rows are relevant. A file read returns thousands of lines; the agent needed one function.

Pattern: extract and compress before storing in context.

```python
async def call_tool_with_compression(tool_name: str, args: dict, max_tokens: int = 500) -> str:
    raw_result = await execute_tool(tool_name, args)
    
    if count_tokens(raw_result) <= max_tokens:
        return raw_result
    
    # Compress: extract the relevant portion
    compressed = await llm.generate(f"""
    The agent called {tool_name} with {args} and received this result:
    
    {raw_result}
    
    Extract only the information relevant to the agent's current task. Max {max_tokens} tokens.
    """)
    
    return compressed
```

---

## The Token Budget Pattern

For production agents with variable-length tasks, implement an explicit token budget that distributes tokens across context components:

| Component | Budget (% of window) |
|-----------|---------------------|
| System prompt | 5% |
| Conversation summary | 10% |
| Recent turns (last 4) | 20% |
| Retrieved documents | 30% |
| Tool results | 25% |
| Response buffer | 10% |

When any component threatens to exceed its budget, apply the appropriate compression strategy. This prevents any single component from crowding out others.

---

*Day 9 of the [Production Agentic AI series](/posts/production-agentic-ai-30-day-plan/). Previous: [RAG vs Fine-Tuning — The Hybrid Answer](/posts/rag-vs-fine-tuning-the-hybrid-answer-in-2026/)*
