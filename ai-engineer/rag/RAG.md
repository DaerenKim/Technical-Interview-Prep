# Production-Grade Retrieval-Augmented Generation (RAG)

## Definition

Retrieval-Augmented Generation (RAG) is an architecture that combines information retrieval with Large Language Models (LLMs) to generate responses grounded in external knowledge sources.

The primary objectives of RAG are:

* Reduce hallucinations
* Incorporate up-to-date information
* Enable enterprise knowledge retrieval
* Improve answer traceability through citations
* Maintain lower costs compared to fine-tuning large models

---

# High-Level Architecture

```text
                    ┌─────────────────────┐
                    │   Raw Documents     │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Parsing & Cleaning  │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │ Metadata Extraction │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      Chunking       │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │      Embedding      │
                    └──────────┬──────────┘
                               │
                               ▼
                    ┌─────────────────────┐
                    │   Vector Database   │
                    └──────────┬──────────┘

────────────────────────────────────────────────────────────

User Query
        ▼
Query Understanding
        ▼
Query Rewriting
        ▼
Metadata Filter Generation
        ▼
Hybrid Retrieval
        ▼
Reranking
        ▼
Context Compression
        ▼
Guardrails
        ▼
LLM Generation
        ▼
Groundedness Validation
        ▼
Citation Generation
        ▼
Final Response
```

---

# Ingestion Layer

## Objective

Transform unstructured enterprise knowledge into retrievable documents.

## Common Sources

* PDF
* DOCX
* PPTX
* HTML
* Notion
* Confluence
* SharePoint
* Slack
* Databases
* APIs

## Challenges

### OCR

Scanned documents require optical character recognition before indexing.

### Tables

Financial and operational information often exists in structured tables.

### Images

Important information may be embedded in screenshots, diagrams, and charts.

### Duplicates

Multiple documents may contain identical information.

### Formatting Noise

Headers, footers, page numbers, and malformed HTML frequently introduce retrieval noise.

## Common Frameworks

* Unstructured
* LlamaParse
* Azure Document Intelligence
* Docling
* Tesseract OCR

---

# Metadata Extraction

## Definition

Metadata refers to structured information associated with a document or chunk.

Metadata enables:

* Filtering
* Access control
* Governance
* Search optimization
* Citation generation

---

## Metadata Categories

### System Metadata

Automatically available.

Examples:

```json
{
  "filename": "q1_sales_report.pdf",
  "source": "SharePoint",
  "author": "John Doe",
  "created_at": "2026-01-01"
}
```

---

### Extracted Metadata

Generated through rules or LLMs.

Examples:

```json
{
  "industry": "Finance",
  "country": "Singapore",
  "department": "Sales",
  "quarter": "Q1",
  "summary": "...",
  "keywords": [
    "revenue",
    "forecast"
  ]
}
```

---

## LLM-Based Metadata Extraction

Example extraction schema:

```json
{
  "industry": "",
  "department": "",
  "country": "",
  "document_type": "",
  "summary": "",
  "keywords": []
}
```

Typical extraction targets:

* Keywords
* Entities
* Topics
* Summaries
* Industries
* Departments
* Regions

---

# Chunking

## Definition

Chunking is the process of splitting documents into retrievable units while preserving semantic meaning.

---

## Fixed Chunking

Example:

```text
Chunk Size: 500 Tokens
Overlap: 50 Tokens
```

Advantages:

* Simple
* Fast

Disadvantages:

* Arbitrary boundaries
* Context fragmentation

---

## Recursive Chunking

Hierarchy:

```text
Document
 └── Paragraph
      └── Sentence
           └── Word
```

Advantages:

* Preserves structure
* Better contextual boundaries

Common Implementations:

* RecursiveCharacterTextSplitter
* Sentence-based splitters

---

## Semantic Chunking

Chunks are created based on meaning rather than token count.

Example:

```text
Section A: Product Overview

Section B: Revenue Analysis

Section C: Market Outlook
```

Advantages:

* Better retrieval precision
* Better context coherence

Disadvantages:

* Increased preprocessing cost

---

## Parent-Child Chunking

Storage:

```text
Parent Chunk = 2000 Tokens

Child Chunk = 300 Tokens
```

Retrieval:

```text
Retrieve Child
Return Parent
```

Advantages:

* High retrieval precision
* Larger contextual coverage

Common in enterprise RAG systems.

---

# Embeddings

## Definition

Embeddings are dense vector representations of text.

Example:

```text
"Apple released a new iPhone"

↓

[0.23, -0.88, 0.41, ...]
```

Embeddings enable semantic similarity search.

---

## Embedding Models

### Proprietary

* OpenAI text-embedding-3-small
* OpenAI text-embedding-3-large

### Open Source

* BGE
* E5
* GTE
* Jina Embeddings
* Nomic Embeddings

---

## Embedding Evaluation Criteria

### Retrieval Quality

Ability to retrieve relevant chunks.

### Latency

Embedding generation speed.

### Cost

Embedding cost per token.

### Dimensionality

Storage and retrieval tradeoffs.

---

# Vector Databases

## Definition

Vector databases store embeddings alongside metadata and original text.

---

## Storage Schema

```json
{
  "id": "chunk_123",
  "vector": [...],
  "text": "...",
  "metadata": {
    "country": "Singapore",
    "year": 2026
  }
}
```

---

## Popular Vector Databases

### Managed

* Pinecone

### Open Source

* Qdrant
* Weaviate
* Milvus
* Chroma
* FAISS

---

## Retrieval Metrics

### Cosine Similarity

Measures angular similarity between vectors.

### Dot Product

Measures vector alignment.

### Euclidean Distance

Measures geometric distance.

---

# Query Understanding

## Objective

Convert raw user input into a retrieval-optimized query.

---

## Example

User Query:

```text
Show Singapore Q1 revenue projections
```

Structured Output:

```json
{
  "country": "Singapore",
  "quarter": "Q1",
  "intent": "retrieve_financial_projection"
}
```

---

## Tasks

### Intent Detection

Determine user objective.

### Entity Extraction

Identify:

* Dates
* Regions
* Organizations
* Products

### Parameter Extraction

Generate structured retrieval filters.

---

# Query Rewriting

## Definition

Transform user queries into retrieval-friendly queries.

---

## Example

Original Query:

```text
What happened last year?
```

Rewritten Query:

```text
What was Singapore revenue in 2025?
```

---

## Benefits

### Improved Recall

Retrieves more relevant documents.

### Improved Precision

Reduces ambiguity.

### Multi-Turn Support

Maintains conversational context.

---

## Common Techniques

* LLM Rewriting
* History-Aware Retrieval
* Query Expansion
* HyDE (Hypothetical Document Embeddings)

---

# Metadata Filtering

## Definition

Metadata filtering constrains retrieval to specific subsets of documents before vector search.

---

## Example

Generated Filters:

```json
{
  "country": "Singapore",
  "year": 2026,
  "department": "Finance"
}
```

---

## Workflow

```text
User Query
      ↓
Metadata Extraction
      ↓
Metadata Filters
      ↓
Vector Retrieval
```

---

## Benefits

### Higher Precision

Reduces irrelevant retrievals.

### Lower Latency

Searches smaller subsets.

### Lower Cost

Reduces token consumption.

### Better Grounding

Improves answer reliability.

---

# Retrieval

## Dense Retrieval

Uses embeddings for semantic similarity search.

Advantages:

* Captures meaning
* Handles paraphrasing
* Handles synonyms

Disadvantages:

* Weak exact keyword matching

---

## Sparse Retrieval

Keyword-based retrieval.

Most common algorithm:

```text
BM25
```

Advantages:

* Exact matching
* Strong performance on IDs and codes

Disadvantages:

* No semantic understanding

---

# Hybrid Retrieval

## Definition

Combines dense retrieval and sparse retrieval.

Architecture:

```text
Dense Retrieval
+
Sparse Retrieval
=
Hybrid Retrieval
```

---

## Benefits

Dense Retrieval:

* Semantic matching

Sparse Retrieval:

* Exact keyword matching

Hybrid Retrieval:

* Higher recall
* Higher precision

---

## Common Architectures

### Weighted Fusion

```text
Final Score

=

0.7(Dense Score)

+

0.3(Sparse Score)
```

### Reciprocal Rank Fusion (RRF)

Frequently used in production search systems.

---

# Reranking

## Definition

Second-stage ranking applied after retrieval.

---

## Pipeline

```text
Query
    ↓
Top 50 Retrieved Chunks
    ↓
Cross Encoder Reranker
    ↓
Top 5 Chunks
```

---

## Cross Encoder Reranking

Evaluates:

```text
(Query, Chunk)
```

jointly.

Advantages:

* Better ranking quality
* Higher retrieval precision

Disadvantages:

* Increased latency

---

## Popular Models

* BGE Reranker
* Cohere Rerank
* Jina Reranker

---

# Context Compression

## Definition

Reduce retrieved context while preserving relevant information.

---

## Motivation

Retrieved Context:

```text
20 Chunks
```

Available Context Window:

```text
5 Chunks
```

---

## Compression Techniques

### Extractive Compression

Retain relevant sentences only.

### Abstractive Compression

Generate summaries.

### Contextual Compression

Remove irrelevant sections while preserving evidence.

---

## Benefits

* Lower token usage
* Faster inference
* Better context efficiency

---

# Guardrails

## Definition

Guardrails enforce reliability, safety, and grounding constraints before generation.

---

# Similarity Threshold Guardrail

Example:

```text
Similarity Score = 0.91
```

High confidence.

```text
Similarity Score = 0.42
```

Low confidence.

Action:

```text
Refuse Answer
```

or

```text
Request Clarification
```

---

# Minimum Evidence Requirement

Require:

```text
≥ 2 Supporting Chunks
```

before generating an answer.

---

# Metadata Validation

Validate retrieved documents satisfy query constraints.

Example:

Query:

```text
Singapore Revenue 2026
```

Retrieved:

```text
USA Revenue 2024
```

Action:

```text
Discard Context
```

---

# Source Diversity Validation

Require evidence from multiple sources.

Example:

```text
Minimum Distinct Sources = 2
```

---

# LLM-as-a-Judge

## Definition

A secondary LLM evaluates generated answers.

---

## Inputs

```text
Question

Retrieved Context

Generated Answer
```

---

## Evaluation Dimensions

### Groundedness

Supported by retrieved evidence.

### Faithfulness

No unsupported claims.

### Completeness

Addresses all aspects of the question.

### Relevance

Directly answers user intent.

---

## Example Judge Prompt

```text
Can every statement in the answer
be supported by the retrieved context?

Return:
- PASS
- FAIL
- Explanation
```

---

# Groundedness Verification

## Objective

Ensure:

```text
Answer ⊆ Retrieved Context
```

Any unsupported statement is flagged.

---

## Benefits

* Reduced hallucinations
* Higher trustworthiness
* Better compliance

---

# Generation

## Prompt Structure

```text
System:
Answer only using provided context.

Context:
...

Question:
...
```

---

## Best Practices

* Ground responses in retrieved context
* Refuse unsupported questions
* Require evidence-backed answers
* Preserve citations

---

# Citation Generation

## Objective

Provide transparent attribution.

---

## Example

```text
Revenue increased by 14%.

(Source: Q1_Report.pdf)
```

---

## Benefits

* Traceability
* Transparency
* Trust

---

# Post-Generation Validation

## Hallucination Detection

Detect unsupported claims.

---

## PII Detection

Examples:

* Email addresses
* Phone numbers
* Government IDs

---

## Compliance Validation

### Finance

Regulatory compliance checks.

### Healthcare

HIPAA compliance checks.

### Enterprise

Internal policy enforcement.

---

# Evaluation

## Offline Evaluation

Dataset:

```text
Question

Ground Truth Answer
```

---

## Retrieval Metrics

### Context Precision

Percentage of retrieved chunks that are relevant.

### Context Recall

Percentage of required information retrieved.

### Hit Rate

Percentage of successful retrievals.

### MRR (Mean Reciprocal Rank)

Measures ranking quality.

### NDCG

Measures ranking effectiveness.

---

## Generation Metrics

### Faithfulness

Answer supported by context.

### Relevancy

Answer addresses the question.

### Correctness

Matches expected answer.

### Groundedness

No unsupported claims.

---

## Evaluation Frameworks

### RAGAS

Metrics:

* Faithfulness
* Context Precision
* Context Recall
* Answer Relevancy

### DeepEval

LLM-based evaluation framework.

### TruLens

Evaluation and observability.

### LangSmith

Tracing, debugging, evaluation.

---

# Production Monitoring

## Retrieval Metrics

* Recall
* Precision
* Hit Rate
* Search Latency

---

## Generation Metrics

* Hallucination Rate
* Refusal Rate
* Groundedness Score
* Citation Coverage

---

## Operational Metrics

* Query Volume
* Error Rate
* Availability
* Throughput

---

## Cost Metrics

* Embedding Cost
* Retrieval Cost
* Reranking Cost
* Generation Cost

---

## Latency Metrics

* Query Rewriting Latency
* Retrieval Latency
* Reranking Latency
* Generation Latency

---

# Common Design Tradeoffs

## Dense Retrieval vs Sparse Retrieval

### Dense Retrieval

Advantages:

* Semantic understanding
* Synonym matching
* Natural language search

Disadvantages:

* Weak exact matching

---

### Sparse Retrieval

Advantages:

* Exact keyword matching
* Strong identifier retrieval

Disadvantages:

* No semantic understanding

---

### Hybrid Retrieval

Advantages:

* Higher recall
* Higher precision
* Better production performance

---

# Chunking Tradeoffs

## Fixed Chunking

Pros:

* Fast
* Simple

Cons:

* Context fragmentation

---

## Semantic Chunking

Pros:

* Better retrieval quality

Cons:

* More expensive

---

## Parent-Child Chunking

Pros:

* High retrieval precision
* Rich contextual coverage

Cons:

* Increased storage complexity

---

# Production RAG System Design

```text
Ingestion
    ↓
Metadata Extraction
    ↓
Chunking
    ↓
Embeddings
    ↓
Vector Database

User Query
    ↓
Query Understanding
    ↓
Query Rewriting
    ↓
Metadata Filtering
    ↓
Hybrid Retrieval
    ↓
Reranking
    ↓
Context Compression
    ↓
Guardrails
    ↓
LLM Generation
    ↓
Groundedness Verification
    ↓
Citation Generation
    ↓
Monitoring & Evaluation
```

## Core Design Goals

* High Recall
* High Precision
* Low Latency
* Low Cost
* Strong Grounding
* Robust Observability
* Production Reliability
