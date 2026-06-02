---
layout: post
title: "RAG in Production — Beyond the Basics"
date: 2026-06-17
categories: [ai, coding-agents]
tags: [agentic-ai, coding-agents, agent-skills, rag]
description: "The basic RAG tutorial gets you to a working prototype. Production RAG requires hybrid search, re-ranking, query transformation, and failure handling that tutorials don't cover. What separates a demo from a system that actually works."
author: akashtalole
---

Retrieval-Augmented Generation is the most widely implemented architectural pattern in enterprise AI. It's also the pattern where the gap between tutorial quality and production quality is widest.

The basic RAG pipeline is three steps: embed the query, retrieve similar documents, stuff them into the context window. This works well in demos. In production, it breaks in predictable ways that require specific solutions.

This post covers what those solutions are.

---

## Where Basic RAG Breaks

**Problem 1: Semantic search misses lexically distinct but conceptually related queries.**

If your documents contain "myocardial infarction" and the user asks about "heart attacks," cosine similarity on dense embeddings may not retrieve the right content. The embedding models understand the relationship, but the retrieval quality depends on how the documents were written.

**Problem 2: Retrieved chunks lack context.**

You embed 512-token chunks. Chunk 47 contains the relevant answer. But chunk 47 starts mid-sentence and references "the approach described above" — which is in chunk 46. The answer in the context is incomprehensible without surrounding context.

**Problem 3: The most similar documents aren't the most relevant.**

"Most similar" and "most relevant" are different. A document that's highly similar to your query might be about a closely related topic but not actually answer the question. A document that's less similar might contain the exact answer.

**Problem 4: Single-step retrieval fails for complex queries.**

"What were the revenue implications of the product decision made at the Q3 board meeting?" requires understanding the product decision (document A), mapping it to business context (document B), and then finding the revenue data (document C). Single-step retrieval rarely surfaces all three.

---

## The Production RAG Stack

**Hybrid search.** Combine dense (embedding) retrieval with sparse (BM25/keyword) retrieval, then fuse the results. The combination handles both semantic and lexical queries better than either alone.

```python
from langchain.retrievers import EnsembleRetriever
from langchain_community.retrievers import BM25Retriever
from langchain_community.vectorstores import Qdrant

# Dense retriever
vector_store = Qdrant(...)
dense_retriever = vector_store.as_retriever(search_kwargs={"k": 10})

# Sparse retriever
bm25_retriever = BM25Retriever.from_documents(docs, k=10)

# Ensemble with reciprocal rank fusion
ensemble_retriever = EnsembleRetriever(
    retrievers=[bm25_retriever, dense_retriever],
    weights=[0.4, 0.6]
)
```

**Re-ranking.** After retrieval, re-rank the candidates using a cross-encoder model that scores each document against the full query (not just the embedding distance). Cross-encoders are more accurate than bi-encoders for relevance scoring, though slower.

```python
from sentence_transformers import CrossEncoder

reranker = CrossEncoder("cross-encoder/ms-marco-MiniLM-L-6-v2")

def rerank(query: str, documents: list[str], top_k: int = 5) -> list[str]:
    pairs = [(query, doc) for doc in documents]
    scores = reranker.predict(pairs)
    ranked = sorted(zip(scores, documents), reverse=True)
    return [doc for _, doc in ranked[:top_k]]
```

**Contextual chunking.** When splitting documents, preserve context by including surrounding content. Two patterns:

- **Sentence window**: retrieve the target chunk but expand to N sentences on either side before adding to context
- **Parent document retrieval**: index small chunks for retrieval precision, but return the larger parent chunk to preserve context

**Query transformation.** For complex queries, transform the query before retrieval:

- **HyDE (Hypothetical Document Embeddings)**: generate a hypothetical answer to the query, then retrieve documents similar to that hypothetical answer rather than the query itself
- **Step-back prompting**: abstract the specific query to a more general form, retrieve against both
- **Multi-query retrieval**: generate multiple variants of the query, retrieve for each, deduplicate results

---

## The Retrieval Pipeline

A production retrieval pipeline looks like this:

```
User Query
    ↓
Query analysis (intent classification, entity extraction)
    ↓
Query transformation (HyDE, step-back, expansion)
    ↓
Hybrid retrieval (dense + sparse, multiple queries)
    ↓
De-duplication and initial filtering
    ↓
Cross-encoder re-ranking
    ↓
Context assembly (with surrounding context)
    ↓
LLM generation
```

Each step is testable independently. When retrieval quality degrades, you can diagnose which step is the source.

---

## The Failure Mode You Must Handle

The most common production RAG failure: the context window contains retrieved documents, but none of them are actually relevant to the query. The LLM generates a response anyway, hallucinating based on the partially relevant context.

Detection: include a relevance check before generation. Ask the model to assess whether the retrieved context is sufficient to answer the query. If not, either escalate to a human, tell the user you don't have the information, or trigger a broader search.

```python
RELEVANCE_CHECK_PROMPT = """
Given this query: {query}

And these retrieved documents: {context}

On a scale of 1-5, how relevant are these documents to the query?
Respond with just the number. If 3 or below, respond with "INSUFFICIENT".
"""
```

An agent that says "I don't have enough information to answer that" is better than an agent that confidently answers incorrectly.

---

*Day 7 of the [Production Agentic AI series](/posts/production-agentic-ai-30-day-plan/). Previous: [The LLM Pricing Collapse](/posts/llm-pricing-collapse-how-cheap-tokens-change-architecture/)*
