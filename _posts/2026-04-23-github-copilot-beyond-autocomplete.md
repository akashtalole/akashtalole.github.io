---
layout: post
title: "GitHub Copilot Beyond Autocomplete — The Features Most Engineers Miss"
date: 2026-04-23
categories: [ai, github-copilot]
tags: [github-copilot, copilot-workspace, sdlc, ai-in-sdlc, coding-agents]
description: "Most engineers use about 20% of GitHub Copilot. Copilot Chat, slash commands, context variables, terminal integration, code explanation — the features that change how you work are the ones people skip past."
author: akashtalole
---

Arc 3 starts today. Seven days on GitHub Copilot — not the basics, but the parts that most engineers haven't fully explored.

Let me start with an observation: every team I've worked with that uses GitHub Copilot mostly uses it for autocomplete. Inline completions as you type. Which is fine — that's genuinely useful. But it's also roughly 20% of what Copilot can do.

The other 80% is where the real leverage is, and most of it sits there unused because nobody took fifteen minutes to explore past the first feature they found.

---

## Copilot Chat — More Than a Sidebar

Copilot Chat is the conversational interface, available in the sidebar or inline in the editor. Most engineers open it once, ask a question, get an answer, and use it intermittently for one-off questions.

What they miss: Copilot Chat is context-aware in ways that make it significantly more powerful than a generic AI chat interface. When you use it in VS Code, it knows which file you have open, which code you have selected, and what's in your other open tabs. You don't need to paste code into the chat — it's already there.

Practical patterns that change how you work:

**Select a function, open chat, ask what it does.** No pasting. No explaining what you're looking at. It reads your selection.

**Select a confusing block, ask for a plain-English explanation.** Faster than reading the code yourself when you're oriented and just need a quick translation.

**Select a failing test, ask what's wrong.** It sees the test, it sees the error (if you paste it), and it gives you a diagnosis without context switching to a browser.

---

## Slash Commands — The Shortcuts Nobody Uses

Slash commands are the fastest way to invoke common operations. Type `/` in Copilot Chat to see the full list. The ones I use most:

**`/explain`** — Explains the selected code or the current file. Takes the context you have selected and gives a structured explanation of what it does, how it works, and what edge cases it handles.

**`/fix`** — Attempts to fix the selected code based on an error or description. Give it the error message, it suggests a fix. Not perfect, but hits the common cases reliably.

**`/tests`** — Generates unit tests for the selected function or class. The output needs review (as covered in an upcoming post), but it's a much faster starting point than writing from scratch.

**`/doc`** — Generates documentation for the selected code. Docstrings, JSDoc, XML docs depending on your language. Edits well.

**`/new`** — Scaffolds a new file, class, or component based on a description. Useful for boilerplate that follows patterns you're familiar with.

**`/optimize`** — Analyses selected code for performance improvements and suggests changes. Quality varies — treat as input, not instructions.

The value isn't that these are magic — it's that they're fast. A two-second slash command is the difference between doing the thing and deciding to do it later.

---

## Context Variables — Telling Copilot Exactly What to Look At

This is the most underused feature in my observation. Context variables let you explicitly tell Copilot Chat what context to use when generating a response.

**`@workspace`** — Tells Copilot to search across your entire workspace when answering. Instead of only knowing about the file you have open, it can reference your broader codebase. Dramatically better results for questions like "how is authentication handled in this project?" or "show me examples of how we use this pattern."

**`@file`** — References a specific file by path. "Looking at `@src/auth/middleware.ts`, what does the token validation logic do?"

**`@selection`** — Explicitly uses your current selection. Usually implicit, but useful when you want to combine it with other context.

**`#file`** — Drags in a specific file as context. Useful when the relevant code isn't in your current tab.

The practical difference: questions about your codebase without `@workspace` get generic answers. With `@workspace`, they get answers grounded in your actual code.

---

## Terminal Integration

Copilot in the integrated terminal is one of the most practically useful features for engineers who live on the command line, and it's the one that gets the least attention in demos.

In the VS Code terminal, you can get command suggestions inline. Start typing, or use the Copilot trigger, and it suggests shell commands based on your current context and what you seem to be trying to do.

Where this pays off:

- Complicated git commands you know exist but can't remember the flags for
- Docker commands with the right flags for your use case
- Grep patterns that would take you two minutes to construct manually
- AWS CLI commands where the syntax is always slightly different than you remember

The suggestions aren't always right. But for "I know there's a command for this and I can't remember the exact syntax," they're right often enough to save meaningful time.

---

## Copilot for Pull Requests

GitHub.com (not just VS Code) has Copilot integrated into the PR workflow.

**PR description generation.** In the PR creation flow, Copilot can generate a description from the diff. Same value proposition as the CI automation I described on Day 12, but without any setup.

**Code review suggestions.** Copilot can flag potential issues directly in the PR review interface. Not as thorough as a dedicated AI review step, but catches common issues inline.

**Copilot review.** You can explicitly request a Copilot review on a PR. It produces structured feedback similar to a human reviewer's comments.

---

## Inline Chat — The Flow-State Feature

Inline chat is triggered directly in the editor — Ctrl+I by default — and opens a small chat interface inline with your code without breaking your editor layout.

The use case: you're in flow, you need to ask something or make a quick change, and you don't want to switch to a sidebar. The inline chat keeps you in the context of the code you're working on.

Particularly useful for: "rewrite this in a more readable way," "add error handling to this function," "explain this regex," "make this function handle the case where the list is empty."

Small interactions that would break flow if they required a context switch.

---

## The Pattern Across All of These

What all these features have in common: they're designed to reduce the cost of doing things you should do but often skip because the friction is too high.

Documenting a function takes thirty seconds with `/doc`. So you do it. Explaining a confusing block takes ten seconds with `/explain`. So you do it before assuming you understand it. Generating a test takes a minute with `/tests`. So you do it instead of shipping without coverage.

The aggregate effect isn't each individual feature — it's that the activation energy for good engineering practices drops low enough that you actually do them.

---

*Day 14 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [Claude Code in Regulated Environments](/posts/claude-code-in-regulated-environments/).*
