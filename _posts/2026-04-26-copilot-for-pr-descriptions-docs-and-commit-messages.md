---
layout: post
title: "Copilot for PR Descriptions, Docs, and Commit Messages"
date: 2026-04-26
categories: [ai, github-copilot]
tags: [github-copilot, sdlc, ai-in-sdlc]
description: "The non-code uses of GitHub Copilot that save the most time aren't the ones in the demos. PR descriptions, inline documentation, commit messages, and changelogs — the writing that accompanies code is where Copilot quietly earns its keep."
author: akashtalole
---

Every codebase has a writing problem. Not the code — the prose that's supposed to explain the code. PR descriptions that say "fixes bug." Commit messages that say "changes." Functions with no docstrings or docstrings that describe the implementation instead of the contract. Changelogs that are wrong or missing.

These problems exist not because engineers can't write, but because writing about code after you've written the code is a second task that competes with the next thing. It consistently loses.

Copilot reduces that competition by making the writing faster than the decision to defer it.

---

## PR Descriptions

A good PR description answers three questions: what changed, why it changed, and how to verify it works. Most engineers know this. Most PR descriptions answer one of the three, badly.

**Using Copilot in the GitHub PR creation flow.** When you open a new PR on GitHub.com and the Copilot icon is available (requires Copilot for Business or Enterprise), it can generate a draft description from the diff. The output follows a structured format: summary, changes, testing notes.

The draft is usually 70-80% correct. The part it gets right reliably: what changed and how. The part it gets wrong: why. It doesn't know whether this is a hotfix, a planned feature, a performance improvement, or a security patch unless you tell it in the commit messages.

The workflow that works: write descriptive commit messages as you go (covered below), then let Copilot generate the PR description from those commits. The description it generates from good commit messages is much better than what it generates from diff alone.

**Using `/doc` in VS Code before opening the PR.** If you work through the changes in VS Code before creating the PR, you can use inline Copilot Chat to help draft the description as a markdown block, which you then paste into GitHub. More setup, but useful when you want a specific format or structure.

---

## Commit Messages

Commit messages have the highest leverage for everything that comes after them: PR descriptions, changelogs, blame history, debugging. Engineers know this and still write bad commit messages, because each commit is a small decision and the value feels distant.

**Copilot in the terminal.** With Copilot CLI or Copilot in the VS Code terminal, you can get commit message suggestions based on your staged changes. Stage your changes, start typing `git commit -m "`, and get a suggestion.

The suggestions are better than the average commit message engineers write under time pressure. They're not better than a thoughtful commit message from an engineer who has time. The use case is the common case: you've made a clear change, you know what it is, you don't want to spend thirty seconds writing it out.

**What to provide for better suggestions.** If the staged diff is large or covers multiple concerns, a better approach is to describe the intent yourself and let Copilot expand it. Type the first clause: `"Refactor auth middleware to"` and let it complete based on the diff. The combination of your intent plus its diff analysis produces better messages than either alone.

---

## Inline Documentation

Function docstrings, class documentation, module-level READMEs — Copilot generates all of these quickly via the `/doc` slash command or by accepting completions after opening a docstring.

The output quality follows the same pattern as everything else: better for well-named, well-scoped functions with clear behaviour, weaker for complex functions with implicit side effects and domain-specific logic.

**The workflow I use:** write the function, then trigger `/doc` before moving on. Two seconds to trigger, thirty seconds to review and edit. The result is a docstring that captures the mechanical description — parameters, return value, what it does — and I add any business context or gotchas that Copilot couldn't know.

This matters more than it sounds. Docstrings written immediately after the function is written, while the context is fresh, are dramatically more accurate than docstrings written later. Copilot makes "immediately" frictionless enough that it actually happens.

**READMEs for new modules.** When creating a new module or package, use Copilot to generate a README skeleton: what this module does, what it exports, how to use it, known limitations. The skeleton takes thirty seconds to generate. Filling it in with real content takes another five minutes. The result is documentation that exists instead of documentation you planned to write.

---

## Changelogs

Changelogs are the most consistently neglected documentation artifact in engineering. They're important for users, partners, and internal teams tracking what changed between releases. They're updated retrospectively, often under time pressure, from imperfect memory.

**Using Copilot to generate changelog entries from commits.** If you have descriptive commit messages (and if you've built the habit above, you will), Copilot can aggregate them into changelog format. Give it the commit log for the release period and a description of your changelog format, and it produces a structured draft.

The output needs editing: grouping related changes, identifying the right version-bump type (breaking change, new feature, bugfix), ensuring consistency with your changelog format. But the raw material is there.

---

## The Compounding Effect

Each of these individually saves a few minutes. The aggregate effect is larger than the sum.

When PR descriptions are consistently good, reviewers spend less time on context and more on substance. When commit messages are descriptive, debugging and archaeology get faster. When documentation is written as code ships rather than deferred, it exists. When changelogs are maintained, releases are communicated properly.

These are all things engineering teams know they should do. The reason they often don't is the activation energy is too high when you're trying to ship. Copilot doesn't eliminate the work — it eliminates enough friction that the work actually gets done.

That's the quiet category of Copilot value that doesn't make it into the demos.

---

*Day 17 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [GitHub Copilot for Test Generation](/posts/github-copilot-for-test-generation-does-it-actually-work/).*
