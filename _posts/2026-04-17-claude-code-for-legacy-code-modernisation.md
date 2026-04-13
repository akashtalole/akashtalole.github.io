---
layout: post
title: "Claude Code for Legacy Code Modernisation"
date: 2026-04-17
categories: [ai, claude-code]
tags: [claude-code, enterprise, sdlc, ai-in-sdlc, coding-agents]
description: "Legacy codebases are where senior engineers spend a disproportionate amount of their time — and where AI tooling pays off most. Here's the approach I use with Claude Code to understand, document, and safely modernise code nobody fully understands."
author: akashtalole
---

If you asked me where Claude Code has made the biggest practical difference in my work, I'd say legacy code. Not greenfield development. Not writing new services from scratch. The gnarly, underdocumented, no-tests-anywhere codebases that every engineering team has at least one of.

This is the work that's most painful to hand to junior engineers, most cognitively expensive for senior ones, and most likely to be deferred until it becomes a crisis. AI tooling doesn't eliminate that work, but it meaningfully reduces the cost of doing it well.

---

## The Problem With Legacy Code

Let me be specific about what makes legacy code hard, because the solutions follow from the problems.

**No documentation.** Not outdated documentation — no documentation. The code was written by someone who understood the context at the time and saw no need to explain it.

**No tests.** Which means you can't change anything with confidence. Every refactor is a calculated risk.

**Implicit dependencies.** The function that looks simple does three other things that only matter in production, on Tuesdays, when a specific flag is set.

**Author unavailable.** The person who wrote this left two years ago. The person who maintained it left last year. The business logic is encoded in the code and nowhere else.

**Fear of touching it.** Which means it's grown through addition rather than refactoring. New behaviour gets bolted on. Technical debt accumulates around the edges because changing the core feels too risky.

---

## The Approach: Understand Before You Touch

The worst thing you can do with legacy code is start refactoring before you understand it. I've seen this destroy projects. You change something that looked safe, something else breaks in a way that takes weeks to diagnose, confidence in the codebase craters.

The right order is always: understand → document → test → refactor. Claude Code is most useful in the first two phases.

### Phase 1: Building a Mental Model

The first thing I do with an unfamiliar legacy codebase is use Claude Code as an interactive explainer.

I start broad: paste the main entry points, the core data models, the primary flow. Ask Claude Code to describe what the system does, what the key abstractions are, and what it doesn't understand. That last part is important — asking it to identify what's unclear surfaces the places where the code is genuinely ambiguous, which are exactly the places I need to focus my attention.

Then I go narrow. Pick the most critical, least understood module. Paste it in full. Walk through it line by line with Claude Code. Ask it to explain what each section does, what assumptions it's making, what could go wrong.

Two things happen in this process. First, Claude Code's explanations are usually 80% correct and 20% wrong-but-interesting. The wrong parts are often wrong in revealing ways — they surface the implicit assumptions in the code that aren't visible to an outside reader. Second, the act of explaining the code to Claude Code (describing what I think it does) forces me to be precise about my own understanding and surfaces where I'm handwaving.

This phase typically takes a day for a module I'd otherwise spend a week on.

### Phase 2: Creating Documentation While You Understand

The moment you understand something about legacy code, document it. Don't plan to come back and document it later — you won't, and even if you do, the understanding will have faded.

Claude Code makes this frictionless. As I work through the understanding phase, I'm generating documentation in parallel:

- **Inline comments** for code blocks where the intent isn't obvious
- **Function docstrings** describing what each function does, what it assumes, and what it returns
- **Module-level README** describing the purpose of the module, its dependencies, and any known gotchas
- **Decision records** for any business logic that's opaque — "this handles the case where..." with enough context that the next engineer doesn't have to reverse-engineer the decision

I paste code sections to Claude Code, ask it to draft the documentation, then edit to add the context it couldn't know — business reasons, historical decisions, system-specific constraints. The resulting documentation is better than what I'd write from scratch because Claude Code captures the structural understanding while I add the contextual knowledge.

### Phase 3: Adding Tests Before You Refactor

This is non-negotiable. You don't refactor untested code. You add tests that characterise the current behaviour first — even the behaviour you think is wrong — and then you refactor.

Claude Code is useful here for generating test scaffolding. Give it the function signature and the docstring you just wrote, ask for unit tests that cover the main paths and obvious edge cases. The generated tests are a starting point, not a finish line — you need to verify that they actually run, that they test the right things, and that they fail for the right reasons when you break the code.

The goal of this test phase is confidence, not coverage. You want enough tests that when you change the refactored code, you'll know if you broke something.

### Phase 4: Refactoring With AI Support

Only now do you start changing things.

For the refactoring itself, Claude Code is useful as a thinking partner: "I want to extract this logic into its own function, here are the dependencies, here's what I'm worried about." It will often surface concerns I hadn't considered and suggest patterns that apply to the specific situation.

For the mechanical parts of refactoring — renaming, restructuring, updating call sites — Copilot's inline assistance is better than Claude Code. Use both: Claude Code for the design of the refactor, Copilot for the execution.

---

## What Claude Code Can't Do Here

It doesn't know your business domain. The comments it generates for business logic are often technically accurate but contextually wrong. A function that calculates pricing for a specific customer tier looks like generic pricing logic to Claude Code. The documentation it generates will describe the mechanics without understanding the why. You have to supply that.

It can't tell you if the existing behaviour is correct. If the legacy code has a bug that's been there for years and the system has adapted around it, Claude Code has no way to know that. It will faithfully describe the buggy behaviour as if it's intentional.

It can't validate that its understanding is complete. For truly complex systems with non-obvious interactions between components, Claude Code's understanding will have gaps. The gaps aren't always visible. Treat its explanations as hypotheses to verify, not facts to trust.

---

## The Payoff

The legacy codebase I've been working through for the past month had no documentation, no tests, and one engineer left who understood parts of it. Before AI tooling, this would have been a six-month project just to reach a point of confidence.

With the approach above — Claude Code for understanding and documentation, tests to lock in behaviour, then careful refactoring — we're moving significantly faster and with more confidence than I expected.

The code is still legacy. The problems are still real. But the cost of engaging with it has dropped enough that we're actually doing the work instead of working around it.

---

*Day 8 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [Setting Up Claude Code for Enterprise Teams](/posts/setting-up-claude-code-for-enterprise-teams/).*
