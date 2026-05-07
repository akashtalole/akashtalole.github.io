---
layout: post
title: "Measuring ROI and Making the Business Case for AI"
date: 2026-06-07
categories: [ai, enterprise]
tags: [agentic-ai, enterprise, ai-in-sdlc, sdlc]
description: "At some point every AI investment has to justify itself. How to build a credible business case for AI tooling, what metrics to use, what to avoid, and the honest conversation about what AI adoption is actually worth."
author: akashtalole
---

At some point you will have to justify the AI investment to someone who doesn't use the tools and wants to see numbers.

The mistake most engineers make in this conversation is leading with technology. "Claude Code is amazing, Copilot saves me so much time." This lands as evangelism and doesn't move decision-makers.

The conversation that works is business outcomes: what changed, what it cost, what the organisation got.

---

## The ROI Calculation Problem

AI tooling ROI is genuinely hard to calculate with precision, and anyone who claims otherwise is either in a rare situation or selling something.

The difficulty: isolating the AI contribution. Cycle time improved, but so did the team's experience level. Defect rate dropped, but so did the test coverage requirement. Features shipped faster, but the scope was also more clearly specified. Separating AI's contribution from everything else that changed is hard.

The honest answer to "what's the ROI?" for most teams is: we have strong directional evidence it's positive, and the magnitude is probably in the range of X, but the precision is limited.

Decision-makers who need exact numbers before approving AI investment are setting a standard they wouldn't apply to other tools. A $50k/year Copilot licence doesn't require the ROI precision of a $5M infrastructure investment.

---

## Building the Business Case

**Start with cost baseline.** Engineer time costs money. If you know average fully-loaded cost per engineer-hour and you have realistic estimates of time saved on specific tasks, you have the foundation of a financial case. Don't inflate the time savings estimates — you'll be held to them.

**Use measurable, specific tasks.** "AI saves time on development" is not a business case. "AI reduces the time to write unit tests from 45 minutes to 15 minutes per feature, based on our measurement over the last quarter" is. Specific, measured, defensible.

**Include the risk-reduction case.** Some of the value of AI is in reduction of defects, faster incident resolution, and more consistent documentation. These reduce costs and risk but are harder to quantify. Include them directionally, not with false precision.

**Account for adoption costs.** Licence costs are the visible cost. Training time, workflow changes, productivity dip during transition — these are real costs that business cases often omit. Include them to maintain credibility.

**Show the alternative.** What does the same outcome cost without AI? If AI-assisted development of a feature takes 4 days and human-only takes 7, the difference is 3 engineer-days. At whatever engineer-day cost, that's the savings per feature. Project across the quarter's work.

---

## The Numbers That Work in Presentations

I've found these metrics land well with non-technical stakeholders:

**Cycle time reduction for specific task types.** Clear before/after on tasks that are easily understood. "Writing API integration tests: before AI, 2 days. After AI tools, 4 hours."

**Code review cycle shortening.** "Average PR from submission to merge reduced from 3.2 days to 1.8 days." Easy to understand, directly linked to delivery velocity.

**Time-to-first-contribution for new engineers.** "New engineers are making meaningful contributions in week 2 rather than week 4." This is valuable and intuitive to non-engineers.

**Feature delivery acceleration.** "We shipped 40% more features in Q1 2026 vs. Q1 2025, with the same team size." Attribution is imprecise, but the direction is clear.

---

## The Honest Conversation

The ROI case is usually positive. But there's a version of AI adoption that looks positive on metrics and is actually creating hidden costs — technical debt, skill atrophy, over-reliance.

The complete ROI conversation includes: "Here's what we've gained. Here's what we're investing to prevent over-reliance and maintain quality. Here's how we're monitoring for the risks."

This is a more credible business case than a one-sided productivity story. Decision-makers trust it more because it acknowledges that adoption has costs and that you're managing them.

---

*Day 28 of the [AI-First Engineering Team series](/posts/ai-first-engineering-team-30-day-plan/). Previous: [Security, IP, and Compliance in an AI-First Team](/posts/security-ip-and-compliance-in-an-ai-first-team/)*
