---
layout: post
title: "Evaluating RAG Pipelines — The Metrics That Matter"
date: 2026-06-22
categories: [ai, coding-agents]
tags: [agentic-ai, coding-agents, agent-skills, rag]
description: "Most RAG systems are deployed without a proper evaluation framework. Teams discover quality problems from user complaints rather than metrics. The evaluation metrics, frameworks, and test set design that make RAG quality measurable."
author: akashtalole
---

The most common RAG failure mode in production is not a technical failure — it's a quality failure that nobody noticed during development because there was no evaluation framework.

The pipeline worked. It retrieved something. It generated a response. Nobody checked whether the retrieved content was relevant or whether the generated response was accurate.

---

## The Three Quality Dimensions

RAG quality has three components that can fail independently:

**Retrieval quality**: did we find the right documents? A RAG system that retrieves irrelevant documents will produce a bad answer regardless of the LLM quality.

**Generation quality**: given correct documents, did the LLM generate an accurate, complete answer? A RAG system that retrieves perfect documents but hallucinates in the generation step still fails.

**End-to-end quality**: did the user get a correct, useful response? This combines retrieval and generation, plus factors like response format and completeness.

Evaluating all three tells you where failures are occurring and guides improvements to the right layer.

---

## RAGAS: The Evaluation Framework

RAGAS (Retrieval-Augmented Generation Assessment) is the most widely used framework for RAG evaluation. It provides five metrics that cover the quality dimensions:

**Context precision**: of the retrieved chunks, what fraction are actually relevant to the query? High precision means retrieval is focused.

**Context recall**: of the relevant information that exists, what fraction was retrieved? High recall means retrieval is comprehensive.

**Faithfulness**: does the generated answer only use information from the retrieved context? Low faithfulness indicates hallucination.

**Answer relevancy**: does the generated answer address the actual question? A technically faithful but off-topic response fails this metric.

**Context entity recall**: does the retrieved context contain the key entities mentioned in the ground truth answer?

```python
from ragas import evaluate
from ragas.metrics import (
    faithfulness,
    answer_relevancy,
    context_precision,
    context_recall,
)
from datasets import Dataset

# Your test cases: question, context retrieved, answer generated, ground truth
test_data = Dataset.from_dict({
    "question": ["What is the refund policy for digital products?"],
    "contexts": [["Our refund policy... (retrieved chunks)"]],
    "answer": ["The refund policy for digital products is..."],
    "ground_truth": ["Digital products are eligible for refund within 30 days if..."]
})

results = evaluate(
    test_data,
    metrics=[faithfulness, answer_relevancy, context_precision, context_recall]
)
print(results)
```

---

## Building a Test Set

The evaluation framework is only as good as the test set. Building a good test set is where most teams underinvest.

**Coverage requirements:**
- Common queries (the 80% case)
- Edge cases (queries near the boundary of your knowledge base)
- Queries that should return "I don't know" (not in the knowledge base)
- Multi-hop queries that require synthesising across documents
- Queries that test for known failure modes from production

**Sources:**
- Production logs (real user queries, anonymised)
- Subject matter experts writing queries they'd expect the system to handle
- Adversarial queries designed to probe failure modes

**Ground truth:**
- For retrieval evaluation: label which documents in your corpus are relevant for each test query
- For generation evaluation: write the correct answer for each test query

The minimum useful test set is 100–200 examples. Smaller sets have too much variance. Larger sets are better but require proportional labelling effort.

---

## Automating Evaluation with LLM Judges

Manual evaluation of 200 test cases is feasible for initial setup, but continuous evaluation (running evals on every code change) requires automation.

LLM-as-judge: use a capable model to assess quality dimensions automatically.

```python
FAITHFULNESS_JUDGE_PROMPT = """
Question: {question}
Context: {context}
Answer: {answer}

Is the answer faithful to the context? Assess whether the answer contains 
only information present in the context, or whether it introduces facts not 
supported by the context.

Respond with:
- "FAITHFUL" if all claims in the answer are supported by the context
- "HALLUCINATION" if any claims are not supported by the context
- "PARTIAL" if some claims are supported and some are not

Explain your reasoning briefly.
"""
```

LLM judges are not perfect — they have their own failure modes — but they're consistent enough to detect regressions between versions. The key insight: you don't need perfect evaluation. You need evaluation that's consistent enough to tell you when quality has changed.

---

## The Evaluation Cadence

**Before deployment:** run the full eval suite. Establish baseline scores.

**On every change to the RAG pipeline:** run a subset of the eval suite (the most sensitive tests). A 10% drop in faithfulness from a chunking change should be caught before it reaches production.

**Weekly:** run the full eval suite. Track trends. Rising context precision with declining context recall may indicate over-aggressive filtering.

**When users report problems:** add the reported case to the test set. This makes the test set grow naturally to cover the actual failure modes of your system.

---

*Day 12 of the [Production Agentic AI series](/posts/production-agentic-ai-30-day-plan/). Previous: [Chunking Strategies That Actually Work](/posts/chunking-strategies-that-actually-work-in-production/)*
