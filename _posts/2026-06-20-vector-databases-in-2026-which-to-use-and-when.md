---
layout: post
title: "Vector Databases in 2026 — Which to Use and When"
date: 2026-06-20
categories: [ai, coding-agents]
tags: [agentic-ai, coding-agents, agent-skills, rag]
description: "The vector database landscape has matured. Qdrant, pgvector, Weaviate, and Pinecone each have a clear profile. Here's the decision framework based on scale, existing infrastructure, and query patterns — not benchmarks."
author: akashtalole
---

The vector database market has matured significantly since 2023. There are now clear leaders with distinct profiles, and the decision between them is more about your context than about which one has the best benchmark numbers.

This post is the practical decision guide.

---

## The Candidates

**pgvector (PostgreSQL extension)**
Vector search as a PostgreSQL extension. If you're already running Postgres, it's the zero-infrastructure-overhead option.

**Qdrant**
Purpose-built vector database, Rust-based, strong performance and filtering capabilities. The default choice when you need a dedicated vector store.

**Weaviate**
Purpose-built with a strong GraphQL-based query interface and built-in vectorisation. Good for complex data with rich metadata and cross-object queries.

**Pinecone**
Fully managed, serverless, minimal ops overhead. The right choice when you want to not think about the infrastructure.

**Redis with vector search**
If you're already running Redis for caching, the vector search module adds semantic search without adding an infrastructure component.

---

## The Decision Framework

**Start with: do you already have the infrastructure?**

- Already on Postgres, small-to-medium scale, want minimal ops? → **pgvector**
- Already on Redis? → **Redis vector search**
- Already on Weaviate for another use case? → **Weaviate**
- No existing vector infrastructure? → choose based on scale and requirements below

**For new deployments:**

| Scenario | Recommendation | Reason |
|----------|---------------|--------|
| < 10M vectors, team prefers managed | Pinecone | Zero ops, predictable cost |
| < 10M vectors, team prefers open source | Qdrant (self-hosted or cloud) | Best performance/features balance |
| > 10M vectors, complex filtering | Qdrant | Strongest filtering and performance at scale |
| Rich metadata, complex cross-object queries | Weaviate | GraphQL interface handles complex queries well |
| Existing Postgres, don't want new infrastructure | pgvector | Good enough for most use cases |
| Existing Redis, primarily caching + search | Redis vector | Unified infrastructure |

---

## What Actually Matters in the Decision

**Filtering performance.** Most production RAG queries are not pure semantic search. They include metadata filters: "find documents from this department, created in the last 30 days, tagged as policy." Qdrant's filtering is the strongest of the purpose-built options. pgvector combines vector search with native SQL WHERE clauses — very flexible, good performance.

**Operational complexity.** Pinecone is fully managed — no infrastructure to run, no upgrades to manage. Qdrant, Weaviate require you to run and maintain the infrastructure (or use their cloud options). For teams without dedicated infra engineers, managed options reduce risk.

**Hybrid search support.** Purpose-built databases now all support hybrid search (dense + sparse) natively. Qdrant and Weaviate have strong hybrid search. pgvector supports it with PostreSQL full-text search integration.

**Embedding update frequency.** When your documents change frequently, you need efficient upsert and deletion. All the major options handle this, but the ease of integrating with your document update pipeline varies by stack.

---

## The pgvector Case in Depth

pgvector deserves a longer look because it's often dismissed in favour of purpose-built solutions, sometimes wrongly.

At under 1–2 million vectors, pgvector performs competitively with purpose-built databases. The query patterns for most RAG applications — find N similar vectors, filter by metadata — are well-handled by pgvector with the right indexing (HNSW index, which pgvector now supports natively).

The advantages are real:
- No additional infrastructure component
- Transactional consistency with the rest of your data (vector and structured data in the same database)
- Standard PostgreSQL tooling for backups, monitoring, scaling
- SQL for complex filtering (joins, subqueries, date ranges) that would require proprietary query languages elsewhere

For enterprise teams that already operate Postgres and have moderate vector requirements, pgvector is often the right choice. The purpose-built databases win at very large scale and for teams that need the advanced features, but those cases are the minority.

---

## What Doesn't Matter As Much

**Raw similarity search speed.** All major vector databases are fast enough for most applications. The difference between 10ms and 15ms for a similarity search is rarely the production bottleneck.

**Maximum vector dimensionality.** Unless you're using unusual embedding models, standard dimensions (768, 1536) are supported everywhere.

**Exact vs approximate nearest neighbour.** Approximate nearest neighbour (HNSW, IVF) is the right choice for production — exact search is too slow at scale. All major options support HNSW.

---

*Day 10 of the [Production Agentic AI series](/posts/production-agentic-ai-30-day-plan/). Previous: [Context Window Management in Production Agents](/posts/context-window-management-in-production-agents/)*
