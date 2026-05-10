---
layout: post
title: "Documentation Culture in an AI-First Team — Better or Worse?"
date: 2026-05-23
categories: [ai, sdlc]
tags: [agentic-ai, sdlc, ai-in-sdlc, claude-code]
description: "The honest answer: AI makes documentation happen more often, but it can make documentation less useful if you're not careful. The documentation culture patterns that work in AI-first teams, and the failure modes to watch."
author: akashtalole
---

One of the consistent findings from my April series: AI dramatically lowers the activation energy for writing documentation. Engineers who would never have written a README are willing to prompt Claude Code to generate one from their code.

This is unambiguously good for teams that had a documentation deficit — and most teams did.

But it surfaces a different problem. When documentation is easy to generate, teams can end up with a lot of documentation that's technically correct but doesn't contain the information that actually matters. Volume without value.

---

## What AI Does Well for Documentation

**First drafts from code.** Point Claude Code at a module and ask for a README. The output is accurate about what the code does, structured coherently, and covers the main cases. It takes 10 minutes of review and editing to turn into something good. It would have taken 45 minutes to write from scratch.

**API documentation.** Given a function signature and some examples, AI generates well-structured API documentation that covers parameters, return values, and common cases. The patterns are correct; the descriptions are accurate.

**Keeping structure consistent.** When you have 20 services that each need a README with the same sections, AI generates all 20 with consistent structure. Human-written READMEs drift; AI-generated ones follow the template.

**Summarising change impact.** "I've changed the authentication flow — what in the existing documentation needs updating?" is a question Claude Code can answer usefully. Not perfectly, but usefully.

---

## What AI Documentation Gets Wrong

**The "why" is missing.** AI documents what code does. It doesn't know why the approach was chosen over alternatives, what was tried first and failed, or what the architectural context was. This is the documentation that's most valuable to future engineers and most absent from AI output.

**Decision context.** "We use eventually consistent writes here because..." — AI can't write this unless you tell it the reasoning. If you tell it, it will reproduce it accurately. But someone has to write the reasoning first.

**Tacit knowledge.** "This service has an unusual deployment requirement because of the legacy integration with..." AI has no access to the institutional knowledge that lives in engineers' heads. It can only document what's visible in the code.

**"Don't do X" warnings.** The caveats that come from having been bitten by something. "Don't call this method more than once per request because the underlying SDK creates a new connection each time." AI doesn't know your war stories.

---

## The Documentation Practice That Works

The pattern that produces high-quality documentation with AI:

1. **AI generates the structure and the what.** Complete coverage of what the code does, parameters, examples.

2. **Human writes the why, the context, and the caveats.** Architectural decisions, tradeoffs, failure modes, "don't do this."

3. **Document is reviewed like code.** Is the why present? Is the context sufficient for a new engineer? Is there anything misleading?

This takes more time than "AI generates, we ship it." It's much less time than writing documentation from scratch. The quality is higher than either AI-only or human-only, because each covers what the other misses.

---

## The Living Documentation Problem

Documentation written today is wrong in six months because code changes. This was a problem before AI; AI doesn't solve it.

What AI does help with: detecting when documentation is likely outdated. When code that affects a documented behaviour changes, a well-configured system can flag "this change may affect the documentation in X." Not automatic — still requires human review — but it surfaces the issue faster than waiting for someone to notice.

The practice I've found most useful: add documentation review to the definition of done. If a PR changes documented behaviour, updating the documentation is part of completing the story, not optional. AI makes this faster; the human process makes it happen.

---

## The Volume Problem

One failure mode I've seen in teams that start using AI for documentation: documentation sprawl. AI makes it so easy to document things that engineers document everything, creating a large body of documentation that nobody reads because it's not curated.

Good documentation is as much about what you don't document as what you do. A lean, accurate, well-curated set of docs is more valuable than comprehensive documentation that nobody trusts because they've been burned by outdated content.

AI lowers the cost of creation. The judgment about what to create, maintain, and delete is still entirely human.

---

*Day 13 of the [AI-First Engineering Team series](/posts/ai-first-engineering-team-30-day-plan/). Previous: [AI in the CI/CD Pipeline — Automated Quality Gates](/posts/ai-in-the-ci-cd-pipeline/)*
