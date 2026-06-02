---
layout: post
title: "Observability for Complex Agentic Systems"
date: 2026-07-04
categories: [ai, enterprise]
tags: [agentic-ai, enterprise, coding-agents, ai-in-sdlc]
description: "An agent that takes 47 steps, calls 12 tools, and costs $0.23 to run is a black box without proper observability. Tracing, metrics, and cost attribution for agentic systems — what to instrument and how to make the data useful."
author: akashtalole
---

Traditional application observability — request/response logs, error rates, p99 latency — is necessary but not sufficient for agentic systems. When something goes wrong in a 40-step agent workflow, "the request failed" is useless. You need to know which step failed, what the agent was trying to do, what it had retrieved so far, and why it made the decision it made.

Agentic observability is a different discipline.

---

## What You Need to Observe

**The agent trace:** Every step in the agent's execution — which node ran, what it received, what it output, how long it took, how many tokens it used.

**Tool calls:** Every external call the agent made — tool name, input parameters, result summary, latency, success/failure.

**LLM calls:** Every inference call — model, input token count, output token count, cost, latency.

**Decision points:** Where the agent chose between paths — what options were considered, what was chosen, why (the reasoning if captured).

**Session context:** What state was active when the agent was running — user, task description, retrieved memory.

---

## OpenTelemetry for Agent Tracing

OpenTelemetry is the standard for distributed tracing. Agent steps map naturally to spans:

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
import time

# Setup
provider = TracerProvider()
provider.add_span_processor(BatchSpanProcessor(OTLPSpanExporter()))
trace.set_tracer_provider(provider)
tracer = trace.get_tracer("agent.tracer")

async def traced_agent_step(step_name: str, state: dict, step_fn) -> dict:
    with tracer.start_as_current_span(step_name) as span:
        # Set context attributes
        span.set_attribute("session.id", state.get("session_id"))
        span.set_attribute("agent.step", step_name)
        span.set_attribute("agent.iteration", state.get("iteration_count", 0))
        
        start_time = time.time()
        try:
            result = await step_fn(state)
            span.set_attribute("step.success", True)
            return result
        except Exception as e:
            span.record_exception(e)
            span.set_attribute("step.success", False)
            span.set_attribute("step.error", str(e))
            raise
        finally:
            span.set_attribute("step.latency_ms", int((time.time() - start_time) * 1000))

async def traced_llm_call(prompt: str, model: str, session_id: str) -> tuple[str, dict]:
    with tracer.start_as_current_span("llm_call") as span:
        span.set_attribute("llm.model", model)
        span.set_attribute("session.id", session_id)
        
        start = time.time()
        response = await llm_client.generate(model=model, prompt=prompt)
        latency = time.time() - start
        
        # Track token usage
        span.set_attribute("llm.input_tokens", response.usage.input_tokens)
        span.set_attribute("llm.output_tokens", response.usage.output_tokens)
        span.set_attribute("llm.latency_ms", int(latency * 1000))
        
        # Cost calculation
        cost = calculate_cost(model, response.usage.input_tokens, response.usage.output_tokens)
        span.set_attribute("llm.cost_usd", cost)
        
        return response.content, {"tokens": response.usage, "cost": cost, "latency_ms": int(latency * 1000)}
```

Each agent step is a child span. Each LLM call within a step is a child span of that step. This gives you a complete trace tree: session → step → LLM call.

---

## LangSmith and Langfuse

For teams using LangChain/LangGraph, LangSmith provides agent-specific observability out of the box:

```python
from langsmith import Client
from langchain_core.tracers import LangChainTracer

# Automatic tracing for LangGraph
import os
os.environ["LANGCHAIN_TRACING_V2"] = "true"
os.environ["LANGCHAIN_API_KEY"] = "your-api-key"
os.environ["LANGCHAIN_PROJECT"] = "production-agent"

# Every agent run is automatically traced, including:
# - Graph node execution
# - LLM calls with token counts
# - Tool calls
# - State transitions
```

Langfuse is the open-source alternative — self-hosted, no data leaving your infrastructure:

```python
from langfuse.decorators import observe, langfuse_context

@observe(name="research_agent_step")
async def research_step(state: dict) -> dict:
    langfuse_context.update_current_observation(
        metadata={"session_id": state["session_id"]},
        user_id=state.get("user_id")
    )
    
    result = await perform_research(state)
    
    langfuse_context.update_current_observation(
        output=result,
        usage={"input": result["input_tokens"], "output": result["output_tokens"]}
    )
    return result
```

Both tools give you: trace visualisation (see the full agent path), step-by-step inspection, token/cost breakdown per trace, and session replay.

---

## Cost Attribution and Alerting

Token costs compound fast in multi-step agents. Without cost attribution, expensive agents hide in average cost figures:

```python
from dataclasses import dataclass, field
from collections import defaultdict

@dataclass
class CostTracker:
    session_costs: dict = field(default_factory=lambda: defaultdict(float))
    step_costs: dict = field(default_factory=lambda: defaultdict(float))
    
    MODEL_COSTS = {
        "claude-haiku-4-5-20251001": {"input": 0.000001, "output": 0.000005},
        "claude-sonnet-4-6": {"input": 0.000003, "output": 0.000015},
        "claude-opus-4-8-20261001": {"input": 0.000015, "output": 0.000075},
    }
    
    def record_llm_call(self, session_id: str, step_name: str, model: str, 
                         input_tokens: int, output_tokens: int):
        rates = self.MODEL_COSTS.get(model, {"input": 0, "output": 0})
        cost = (input_tokens * rates["input"]) + (output_tokens * rates["output"])
        
        self.session_costs[session_id] += cost
        self.step_costs[f"{session_id}:{step_name}"] += cost
        
        # Alert on high-cost sessions
        if self.session_costs[session_id] > 1.00:  # $1 threshold
            logger.warning(f"High-cost session {session_id}: ${self.session_costs[session_id]:.2f}")
    
    def session_summary(self, session_id: str) -> dict:
        steps = {k.split(":", 1)[1]: v 
                 for k, v in self.step_costs.items() 
                 if k.startswith(f"{session_id}:")}
        return {
            "total_cost": self.session_costs[session_id],
            "cost_by_step": dict(sorted(steps.items(), key=lambda x: x[1], reverse=True))
        }
```

Cost-by-step attribution tells you which steps are expensive. Usually it's not the reasoning steps — it's the steps that pass large documents or full conversation history into context.

---

## The Observability Dashboard

The metrics worth surfacing in a dashboard for production agents:

- **P50/P95/P99 task latency** — broken down by task type
- **Cost per completed task** — trends over time, broken down by model
- **Error rate by step** — which steps fail most often
- **Tool call volume** — calls per task, broken down by tool
- **Token consumption by step** — where context is being used
- **Session abandonment rate** — sessions that started but never completed

These seven metrics give you a complete picture of whether your agent is healthy, getting more expensive, and where failures concentrate.

---

*Day 24 of the [Production Agentic AI series](/posts/production-agentic-ai-30-day-plan/). Previous: [Evaluating Agent Quality](/posts/evaluating-agent-quality-at-production-scale/)*
