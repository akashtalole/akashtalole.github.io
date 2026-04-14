---
layout: post
title: "Multi-Agent Debugging and Observability — Staying Sane"
date: 2026-05-08
categories: [ai, copilot-studio]
tags: [copilot-studio, multi-agent, microsoft, agentic-ai, enterprise]
description: "When a single agent misbehaves you have one thing to debug. When a multi-agent system misbehaves, the problem could be anywhere across orchestrator, specialists, connectors, and flows. How to build observability that makes the invisible visible."
author: akashtalole
---

Multi-agent systems fail in interesting ways.

A user asks about their order. The orchestrator routes to the order agent. The order agent calls a Power Automate flow. The flow calls the order management API. The API returns data that the flow parses incorrectly. The agent presents wrong information confidently. The user escalates.

At which point did it go wrong? Without observability, you're reconstructing the failure from user reports and guesswork. With observability, you have a trace.

---

## The Visibility Problem

Single agents are relatively easy to debug. You can watch the conversation, see what the agent said, compare it to what it should have said.

Multi-agent systems have multiple layers where things can go wrong:
- The orchestrator's intent classification
- The routing decision
- The handoff context passed to the specialist
- The specialist's handling of the query
- The connector/flow call
- The external system's response
- The agent's interpretation of the response
- The final response generation

Without deliberate observability instrumentation, most of these layers are invisible. You see the input (user query) and the output (agent response) but not what happened in between.

---

## Layer 1: Conversation Tracing

Every conversation should produce a trace that shows the full execution path.

In Copilot Studio, use conversation variables to build a running trace:

```
At each significant step, append to a ConversationTrace variable:
{
  "timestamp": "2026-05-08T10:23:45Z",
  "agent": "Orchestrator",
  "action": "IntentClassified",
  "intent": "OrderQuery",
  "confidence": 0.94,
  "entities": {"orderId": "ORD-12345"}
}
```

At conversation end (or on error), write the full trace to your logging infrastructure. Application Insights is the natural destination for Microsoft-stack deployments — it's already integrated with Power Platform.

---

## Layer 2: Flow Execution Logging

Power Automate flows have built-in run history, but the default retention is limited and the signal-to-noise ratio is low when you're debugging a specific issue.

Instrument your flows explicitly:

```json
// At the start of every significant flow
{
  "action": "FlowStarted",
  "flowName": "GetOrderDetails",
  "sessionId": "@{triggerBody()?['sessionId']}",
  "input": "@{triggerBody()}",
  "timestamp": "@{utcNow()}"
}

// At each significant step
{
  "action": "APICallCompleted",
  "endpoint": "OrderManagementSystem/orders/{id}",
  "statusCode": 200,
  "responseTimeMs": 342,
  "sessionId": "@{triggerBody()?['sessionId']}"
}

// At flow completion
{
  "action": "FlowCompleted",
  "success": true,
  "outputSummary": "Order ORD-12345 found, status: Shipped",
  "sessionId": "@{triggerBody()?['sessionId']}"
}
```

The `sessionId` is the thread that connects conversation traces to flow execution traces. Without it, correlating a conversation failure to a specific flow run requires matching timestamps — painful and imprecise.

---

## Layer 3: Agent Decision Logging

For generative AI responses, log the inputs to the generation step: what knowledge sources were used, what context was passed, what the generation produced.

This is the layer most teams skip and most regret skipping. When an agent produces a wrong answer, you need to know whether the knowledge source had wrong information or the agent generated incorrectly from correct information. These have different fixes.

In Copilot Studio, this means logging:
- Which knowledge sources were queried
- Whether a relevant result was found
- Whether the response was generative or topic-based
- The generative response before any post-processing

---

## Building a Correlation ID

The single most useful observability investment in a multi-agent system is a correlation ID — a unique identifier that follows a request through every layer and appears in every log entry related to that request.

```
User opens conversation → Generate SessionId (e.g., "sess-a3f92b")
Orchestrator routes → log includes SessionId
Handoff to specialist → pass SessionId in context
Specialist calls flow → pass SessionId as parameter
Flow logs include SessionId
External API call log includes SessionId
```

When a user reports a problem with "my conversation at 10:23am," you search for their SessionId and get the complete execution history across every layer. Without the correlation ID, you're searching through all logs for the right ones.

---

## Alerting on What Matters

Logs are passive — you have to look at them to learn from them. Alerts are active — they tell you when something needs attention.

The alerts worth setting up:

**Routing failure rate.** When the orchestrator can't classify intent and falls back to clarification more than X% of the time, something has changed — new query patterns the agent wasn't designed for, or a degraded intent model.

**Specialist failure rate.** When a specialist produces errors or escalations above baseline, it signals either a system integration issue or a scope mismatch.

**Response time percentiles.** P95 response time across the full conversation flow. When it spikes, find the slow layer. Usually a connector timeout or slow external API.

**Escalation rate.** How often does the system route to a human? Rising escalation rate means the automated agents are handling less successfully than before.

All of these should feed into a dashboard your team looks at regularly, not just when someone reports a problem.

---

## The Debugging Workflow

When something goes wrong in production:

1. **Get the SessionId** from the user report or conversation history
2. **Pull the correlation trace** — all log entries for that SessionId
3. **Find the divergence point** — where did the execution differ from expected?
4. **Isolate the layer** — was it routing, specialist handling, flow execution, or external system?
5. **Reproduce in test environment** with the same inputs
6. **Fix and verify** the fix before deploying

This workflow only works if you built the observability in step 1. Retrofitting logging after a production incident is painful and leaves you blind for the time it takes to deploy the fix.

---

## Copilot Studio Specific Tools

The Copilot Studio analytics dashboard gives you aggregate metrics: conversation volume, topic trigger rates, escalation rates, CSAT scores if configured. Useful for trend monitoring.

The conversation transcript viewer lets you replay individual conversations. Useful for debugging specific reported issues.

Application Insights (when configured) gives you custom event logging, funnel analysis, and alerting. This is where the instrumentation from earlier sections lands.

Power Automate run history gives you flow-level execution details including step-by-step timing and error information. Essential for debugging integration failures.

No single tool gives you the full picture. The correlation ID is what connects them.

---

## The Principle

Observability is not something you add to a system — it's something you design into a system. The cost of adding it after the fact is high. The cost of building it from the start is low.

Multi-agent systems that go to production without observability will fail in ways that take days to diagnose. Multi-agent systems built with correlation IDs, structured logging, and meaningful alerts will fail in ways that take hours to diagnose — and fail less often because the observability reveals problems before they become user-visible.

Build it first. Thank yourself later.

---

*Day 29 of the [30-Day AI Engineering series](/posts/30-day-ai-engineering-blog-plan/). Previous: [Connecting Copilot Studio Agents to Enterprise Systems](/posts/connecting-copilot-studio-agents-to-enterprise-systems/).*
