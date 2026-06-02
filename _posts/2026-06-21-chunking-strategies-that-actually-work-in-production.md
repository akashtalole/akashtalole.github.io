---
layout: post
title: "Chunking Strategies That Actually Work in Production RAG"
date: 2026-06-21
categories: [ai, coding-agents]
tags: [agentic-ai, coding-agents, agent-skills, rag]
description: "How you split documents determines what your RAG system can find. Fixed-size chunking is the beginner approach. Semantic chunking, parent-document retrieval, and document-type-aware splitting are what production systems actually use."
author: akashtalole
---

Chunking is the unglamorous part of RAG that has an outsized impact on retrieval quality. Teams that spend all their effort on embedding models and vector databases but default to naive fixed-size chunking are leaving significant quality on the table.

---

## Why Fixed-Size Chunking Fails

The standard tutorial approach: split every document into 512-token chunks with 50-token overlap. Index them all.

The failures:

**Semantic fragmentation.** A section explaining a concept gets split in the middle. The chunk that gets retrieved is incomplete and confusing without the rest of the explanation.

**Context loss.** A chunk that says "as described in section 3.2" or "following the approach above" is meaningless without the surrounding context.

**Sub-optimal embedding quality.** A 512-token chunk that mixes two different topics produces an embedding that's vaguely about both, making it less likely to be the top result for either topic specifically.

**Metadata loss.** Fixed-size chunking often discards document structure: headers, section titles, table relationships, list membership. These are information-rich signals that help the model interpret the content.

---

## Strategy 1: Semantic Chunking

Split at semantic boundaries rather than token counts. A semantic boundary is where the topic shifts — a new paragraph after a section break, a new concept introduced, a new argument started.

```python
from langchain_experimental.text_splitter import SemanticChunker
from langchain_openai.embeddings import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
chunker = SemanticChunker(
    embeddings,
    breakpoint_threshold_type="percentile",  # or "standard_deviation"
    breakpoint_threshold_amount=95
)

chunks = chunker.split_text(document)
```

Under the hood: the chunker embeds sentences, measures semantic distance between consecutive sentences, and splits where the distance exceeds a threshold. Topics that transition abruptly get their own chunks.

**When to use:** narrative documents, policy documents, technical explanations — anything where semantic coherence within a section matters.

---

## Strategy 2: Document-Type-Aware Splitting

Different document types have natural structure that should govern chunking.

**Markdown / HTML:** split on headers. An H2 section is a coherent semantic unit. Include the header text in each chunk for context.

```python
from langchain.text_splitter import MarkdownHeaderTextSplitter

headers_to_split_on = [
    ("#", "H1"), ("##", "H2"), ("###", "H3")
]
splitter = MarkdownHeaderTextSplitter(headers_to_split_on=headers_to_split_on)
chunks = splitter.split_text(markdown_document)
# Each chunk includes the header hierarchy as metadata
```

**Code files:** split on function or class boundaries. A function is the natural unit of code. Splitting mid-function produces chunks that are incomplete and hard to embed meaningfully.

**Tables:** keep table rows together, not split across chunks. Include the table header in every chunk containing rows.

**Q&A documents:** keep question and answer together as a single chunk.

---

## Strategy 3: Parent-Document Retrieval

The most impactful pattern for retrieval quality improvement: index small chunks for precision, but return larger parent sections when the chunk matches.

```python
from langchain.retrievers import ParentDocumentRetriever
from langchain.storage import InMemoryStore
from langchain.text_splitter import RecursiveCharacterTextSplitter

# Small chunks for precise indexing
child_splitter = RecursiveCharacterTextSplitter(chunk_size=200)
# Larger chunks to return as context
parent_splitter = RecursiveCharacterTextSplitter(chunk_size=2000)

store = InMemoryStore()
retriever = ParentDocumentRetriever(
    vectorstore=vectorstore,
    docstore=store,
    child_splitter=child_splitter,
    parent_splitter=parent_splitter,
)

retriever.add_documents(documents)
# Retrieval: finds precise small chunks, returns parent context
results = retriever.invoke("your query here")
```

Why it works: small chunks produce more precise embeddings (less semantic noise). But returning only the small chunk to the LLM loses surrounding context. The parent-document pattern gets the best of both.

---

## Strategy 4: Contextual Enrichment

Before indexing, enrich each chunk with context that makes it self-contained.

Anthropic's contextual retrieval approach: for each chunk, generate a short context statement using the full document as reference, then prepend it to the chunk before embedding.

```python
CONTEXT_PROMPT = """
Here is the full document:
<document>
{full_document}
</document>

Here is a specific chunk from the document:
<chunk>
{chunk}
</chunk>

Write a 1-2 sentence context statement that explains what this chunk is about
and how it relates to the document as a whole. This will be prepended to the
chunk to improve retrieval.
"""

async def enrich_chunk(chunk: str, full_document: str) -> str:
    context = await llm.generate(CONTEXT_PROMPT.format(
        full_document=full_document,
        chunk=chunk
    ))
    return f"{context}\n\n{chunk}"
```

This is more expensive (one LLM call per chunk) but significantly improves retrieval for documents where individual chunks lack context about where they fit in the whole.

---

## Chunk Size: The Practical Guidance

There's no universally correct chunk size. The guidance:

- **Short, precise chunks (100–300 tokens)**: factual Q&A, specific data points, entity information. High retrieval precision, low context completeness.
- **Medium chunks (300–600 tokens)**: general-purpose RAG for narrative documents. Decent precision and context.
- **Larger chunks (600–1200 tokens)**: technical documentation, policy explanations, code with comments. Richer context, slightly noisier embeddings.

Test your specific use case. Run a retrieval evaluation with different chunk sizes on your actual documents and queries. The right answer is empirical, not theoretical.

---

*Day 11 of the [Production Agentic AI series](/posts/production-agentic-ai-30-day-plan/). Previous: [Vector Databases in 2026](/posts/vector-databases-in-2026-which-to-use-and-when/)*
