---
layout: post
title: "The AI-First Team Maturity Model — Where Is Your Team Today?"
date: 2026-05-12
categories: [ai, sdlc]
tags: [agentic-ai, sdlc, ai-in-sdlc, enterprise]
description: "A practical maturity model for engineering teams adopting AI. Not a framework for consultants — a diagnostic for leads and engineers to understand where they actually are and what the next step looks like."
author: akashtalole
---

Yesterday I described four levels of AI adoption, from AI-absent to AI-first. Today I want to make that more concrete — because "where is your team today?" is the question that determines everything else.

---

## The Diagnostic

For each dimension, rate your team 0–4. Be honest. The point is diagnosis, not benchmarking.

### Dimension 1: Tool Coverage

How consistently are AI tools used across the team?

- **0**: No AI tools in use
- **1**: Some engineers use AI tools, others don't. No policy.
- **2**: Most engineers have access and use tools for at least some tasks
- **3**: Consistent tool usage across the team; clear decisions about which tools for which tasks
- **4**: Tool usage is embedded in workflow; not using AI for applicable tasks is the exception, not the norm

### Dimension 2: Shared Practices

Does the team have shared norms for AI usage?

- **0**: No norms at all
- **1**: Implicit norms from observation but nothing explicit
- **2**: Some written guidance exists (which tools, acceptable use), but it's not comprehensive
- **3**: Documented practices for the main AI touchpoints: code review, documentation, test generation
- **4**: Living practice documentation that engineers contribute to and update; norms evolve through shared experience

### Dimension 3: Workflow Integration

Is AI integrated into team processes, not just individual work?

- **0**: AI is only individual/personal, no team process involvement
- **1**: AI occasionally appears in team artefacts (PRs with AI-generated descriptions) but not by design
- **2**: AI is intentionally used in at least one team process (e.g., PR descriptions, sprint estimation)
- **3**: AI touchpoints exist across multiple team processes; the workflow was redesigned to include them
- **4**: Team processes were designed from the ground up with AI as a participant; AI absence from a process is a deliberate choice

### Dimension 4: Output Trust and Verification

Does the team have calibrated trust in AI output?

- **0**: Either complete distrust (rarely use output) or naive trust (rarely verify)
- **1**: Individual engineers have developed personal calibration; no team-level shared understanding
- **2**: Team has discussed AI reliability; some shared intuitions about what to trust and when
- **3**: Documented understanding of where AI is reliable and where it needs verification; review practices reflect this
- **4**: Team actively evaluates AI output quality, tracks failure modes, and updates practices based on what they learn

### Dimension 5: Culture and Mindset

Does the team culture support AI adoption?

- **0**: Active resistance or significant anxiety; AI is not discussed openly
- **1**: Mixed sentiment; some enthusiasts, some resisters; not a safe topic in team discussions
- **2**: Generally positive but not proactive; people use AI when convenient but don't push for adoption
- **3**: Psychologically safe to discuss AI failure modes and limitations; learning happens in the open
- **4**: Team actively experiments with AI applications, shares what works, and iterates on practices together

---

## Interpreting Your Score

**0–5 total: AI-absent or very early adoption**
The foundations aren't in place. Focus on getting to consistent tool coverage before anything else. Pick one or two tools, get the team access, and establish basic acceptable-use norms.

**6–10 total: AI-available**
You have tools but not practices. The highest-leverage move is building Dimension 2 — shared practices. Even a short document that codifies what you're already doing informally creates a foundation to build on.

**11–15 total: AI-adopted**
The basics are working. Now the work is workflow redesign (Dimension 3) — deliberately integrating AI into team processes rather than just individual work. Pick one process and redesign it.

**16–18 total: AI-integrated**
You're doing this well. The remaining work is cultural (Dimension 5) and evaluation (Dimension 4). Are failure modes discussed openly? Is the team learning and updating practices based on real evidence?

**19–20 total: AI-first**
You're in rare company. The work now is maintaining it as the team grows and the tools evolve.

---

## The Most Common Imbalance

The pattern I see most often: high scores on Dimensions 1 and 4 (engineers use tools and have personal calibration), low scores on Dimensions 2 and 3 (no shared practices, no workflow integration).

This is the "individual productivity" trap. AI is making individual engineers more productive, but the team isn't capturing compounding benefits because there's no shared practice layer.

Fixing this doesn't require a big programme. It requires a lead who's willing to make practices explicit, write them down, and treat them as living documents. That's the work of an AI-first team lead, and it's mostly a writing and facilitation job, not a technical one.

---

*Day 2 of the [AI-First Engineering Team series](/posts/ai-first-engineering-team-30-day-plan/). Previous: [What Does "AI-First Engineering Team" Actually Mean?](/posts/what-does-ai-first-engineering-team-actually-mean/)*
