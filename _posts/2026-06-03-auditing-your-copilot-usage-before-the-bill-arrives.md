---
layout: post
title: "Auditing Your Copilot Usage Before the Bill Arrives"
date: 2026-06-03
categories: [ai, github-copilot]
tags: [github-copilot, enterprise, ai-in-sdlc]
description: "Most teams don't know how they're using Copilot until the bill surprises them. A practical guide to auditing current usage, categorising requests by cost tier, and building a usage baseline before optimising."
author: akashtalole
---

The first step to managing Copilot costs under the new pricing model is understanding what you're actually doing today.

Most teams don't. They know they're "using Copilot" but have no visibility into which features, how often, and at what cost tier. You can't optimise what you haven't measured, and the surprise bill is what happens when measurement comes after the fact.

Here's how to do the audit before the next billing cycle.

---

## Step 1: Get the Data

**GitHub admin dashboard.** If you're on Copilot Business or Enterprise, the organisation admin has access to usage analytics. Go to your organisation settings → Copilot → Usage. You'll see seat usage, active users, and — in the new model — request breakdowns by type.

What to look for:
- Active users vs. licensed seats (gap indicates unused licences you're paying for)
- Request volume by type: completions vs. chat requests vs. agent requests
- Top users by request volume — the distribution is usually heavy-tailed
- Time-of-day patterns (useful for understanding whether usage is deliberate or ambient)

**IDE-level telemetry.** GitHub provides VS Code extension telemetry you can surface through your analytics infrastructure. This gives per-engineer breakdowns that the org dashboard aggregates. If you need engineer-level cost attribution, this is where to look.

---

## Step 2: Categorise What You See

Once you have the raw data, categorise requests into the three tiers I described in yesterday's post:

| Request type | Tier | Budget impact |
|---|---|---|
| Inline completions, tab suggestions | Free / standard | None under base subscription |
| Copilot Chat conversations | Premium | Moderate per-request |
| `/explain`, `/fix`, `/test` commands | Premium | Moderate per-request |
| PR summaries | Premium | Moderate per-PR |
| Copilot Workspace multi-file edits | High | Significant per-task |
| Agent mode / autonomous tasks | High | Significant per-task |

Most teams find their usage skews heavily toward one or two categories. The engineers who drove most adoption are often the ones generating the most premium requests — which is fine if the value is there, but worth verifying.

---

## Step 3: Identify the Heavy Users

In almost every team, 20–30% of engineers generate 70–80% of premium request costs. This isn't a problem — it may mean your most productive AI users are getting significant value. But it's worth understanding.

Questions to ask about heavy users:
- Are they generating more premium requests because they're using Copilot for genuinely complex tasks? (Good — this is high-ROI usage.)
- Are they in a Copilot Chat loop of clarification because prompts aren't specific? (Bad — optimisable.)
- Are they using high-tier features for tasks that lower-tier tools handle? (Optimisable.)

A 15-minute conversation with your top-3 Copilot users is more informative than any dashboard.

---

## Step 4: Calculate Your Current Effective Cost per Engineer

Take your monthly bill, divide by the number of active users (not licensed seats — active users). That's your current cost per active engineer per month.

Compare that to the value per engineer: if an engineer produces X hours of productive output with AI assistance that would otherwise take Y hours, the cost per engineer-hour-saved is (monthly cost) ÷ (hours saved per month). If that number is less than your engineer's hourly rate, you're ROI-positive. If it's higher, you have a problem.

Most teams are ROI-positive but haven't done this calculation explicitly, which leaves them unable to defend the spend when challenged.

---

## Step 5: Set a Baseline Before Optimising

Don't start making changes before you have a baseline. The baseline is: what is current usage, at what cost, producing what measurable output (velocity, cycle time, defect rate)?

Without the baseline, any optimisation you make is flying blind. You'll reduce costs without knowing whether you also reduced value. You'll change engineer behaviour without knowing the starting point.

The baseline takes a week to capture properly. Set it now, before the next pricing pressure arrives.

---

## What the Audit Usually Reveals

From the audits I've done and heard about:

- **15–25% of licensed seats are inactive.** Engineers who got access, tried it briefly, and stopped. These seats can be reclaimed.
- **50%+ of premium requests come from Chat, not agent mode.** Chat is the biggest cost driver for most teams — and the most optimisable through prompt quality.
- **Agent mode is a small fraction of requests but a large fraction of cost.** A handful of engineers using Workspace extensively can represent disproportionate spend.
- **PR summary generation is being used more than expected.** This is often a good sign — it's moderate cost, high value — but it adds up.

The audit rarely produces bad news about whether Copilot is worth it. It usually produces actionable information about where to redirect usage for better cost-value ratio.

---

*Part of a week-long series on GitHub Copilot's new pricing model. Started with [the overview post](/posts/github-copilot-new-pricing-model-token-economy/) yesterday.*
