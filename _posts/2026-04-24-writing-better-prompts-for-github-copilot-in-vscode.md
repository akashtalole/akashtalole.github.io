---
layout: post
title: "Writing Better Prompts for GitHub Copilot in VS Code"
date: 2026-04-24
categories: [ai, github-copilot]
tags: [github-copilot, sdlc, ai-in-sdlc, coding-agents]
description: "Copilot reads your code and your comments before it generates anything. Understanding what it pays attention to — and shaping that context deliberately — is the difference between useful suggestions and noise."
author: akashtalole
---

Yesterday I covered the Copilot features most engineers miss. Today I want to go one level deeper on something that applies to all of them: how to give Copilot the context it needs to produce output worth using.

This is specifically about VS Code, where most of the interaction is implicit — Copilot reads your code as you write it and generates suggestions without you having to ask explicitly. That makes it different from prompting a chat interface, and it requires a slightly different set of habits.

---

## How Copilot Reads Your Code

Before you can prompt Copilot effectively, it helps to understand what it pays attention to.

When generating a completion, Copilot's context includes:
- The current file, with more weight on the code immediately surrounding your cursor
- Other open tabs in VS Code
- The file path and name
- Comments immediately above the insertion point

It does **not** automatically include:
- Files that aren't open
- Your entire repository
- Anything you haven't shown it

This is the most important thing to internalise. Copilot is working with a limited, specific context. Shaping that context deliberately produces dramatically better suggestions.

---

## Technique 1: Comment-Driven Development

The most reliable technique for getting good Copilot completions is writing a clear comment describing what you want before writing the code.

Not just a vague label — a precise description of what the function should do, what it accepts, what it returns, and any relevant constraints.

**Vague comment → vague completion:**
```python
# Process orders
def process_orders(orders):
```

**Precise comment → precise completion:**
```python
# Filter orders to only include those placed in the last 30 days,
# sort by total value descending, and return the top N.
# orders: list of dicts with 'placed_at' (datetime) and 'total' (float)
# n: max number of orders to return, defaults to 10
def get_recent_top_orders(orders, n=10):
```

The second comment gives Copilot the input types, output behaviour, sorting requirement, and default value. The completion it generates will use all of that.

---

## Technique 2: Show the Pattern Before You Use It

When you want Copilot to follow a specific pattern — an error handling convention, a data structure layout, a test format — write one example of the pattern first, then start writing the next occurrence.

Copilot learns from what it sees in the current file. If you've written three functions with a consistent error handling pattern, the fourth will usually follow the same pattern without you having to specify it. If you haven't, it will guess.

This is particularly valuable for test files. Write the first test in the format and with the naming conventions you want. The next test Copilot generates will usually match.

---

## Technique 3: Use Descriptive Names

This sounds trivial. It isn't.

Copilot uses function names, variable names, and class names as strong signals for what code should do. A function named `process` tells Copilot almost nothing. A function named `validate_and_normalise_shipping_address` tells it the domain, the operation, and the expected output type.

The completion quality difference between well-named and poorly-named functions is significant. This is true for your own code comprehension too — it's just that Copilot makes the benefit immediately visible.

Rename things before asking Copilot to extend them. The better the names, the better the completions.

---

## Technique 4: Keep Related Context in Open Tabs

Copilot reads your open tabs. If you're implementing something that needs to match an existing interface, keep the interface file open. If you're writing code that integrates with a service, keep the service definition open. If you're adding tests for a class, keep the class file open alongside the test file.

This is deliberate context management, not an accident of your workflow. Before starting a coding session on a particular task, open the three or four files that are most relevant to that task. Copilot's completions will reflect that context.

---

## Technique 5: Be Specific in Inline Chat

When using inline chat (Ctrl+I), specificity matters as much as it does in any other prompt context.

**Too vague:**
> "Improve this function"

**Specific:**
> "Refactor this function to handle the case where `user_id` is None, and raise a ValueError with a descriptive message instead of failing silently"

The vague version produces a generic improvement. The specific version produces the exact change you need.

For code modifications specifically, saying what you want changed and what you want to preserve is useful: "Add input validation but keep the existing error handling pattern."

---

## Technique 6: Use File Path as Context Signal

Copilot pays attention to the file path. A file at `src/auth/token_validator.py` will get more auth-relevant suggestions than the same code in `utils/helper.py`. A file named `test_order_service.py` will get test-shaped completions from the start.

This means: put files in directories that reflect what they are, and name them accurately. This is good practice anyway. The Copilot benefit is a bonus signal that it matters.

---

## When Completions Are Consistently Wrong

If completions for a particular section of code are consistently unhelpful — too generic, wrong language patterns, inconsistent with your conventions — it's usually one of three things:

1. **Not enough context.** Open more relevant files. Add a more descriptive comment above what you're writing. The model doesn't have enough signal.

2. **Conflicting context.** If your open tabs include code from multiple frameworks or languages, Copilot may be confused about which conventions apply. Close irrelevant files.

3. **Unfamiliar domain.** For very specialised domains — a niche internal framework, highly specific business logic, proprietary patterns — Copilot has less training data to draw on. You'll need to give more explicit guidance via comments and examples.

The fix is almost always more context, more specific context, or both.

---

## The Underlying Principle

Copilot is a context-driven autocomplete system. Every technique in this post is a way to shape the context it has access to. Comments, names, open tabs, file paths — all of these are signals that feed into what it generates.

Engineers who get good results from Copilot have developed an intuition for managing that context deliberately. Engineers who find it unreliable often haven't changed their habits to account for the fact that the quality of the output depends directly on the quality of the context they provide.

The habit shift is small. The output difference is large.

---

*Day 15 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [GitHub Copilot Beyond Autocomplete](/posts/github-copilot-beyond-autocomplete/).*
