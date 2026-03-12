---
name: rag-pipeline-architect
description: >
  Design, build, and optimize Retrieval-Augmented Generation (RAG) pipelines
  from scratch. Use this skill when building a RAG system, diagnosing retrieval
  quality issues, choosing embedding models, designing chunking strategies,
  or optimizing the end-to-end pipeline for accuracy and latency. Covers
  naive RAG, advanced RAG, and modular RAG architectures.
license: MIT
metadata:
  author: ai-engineering-toolkit
  version: "1.0.0"
  category: ai-engineering
---

# RAG Pipeline Architect

Design production-grade RAG pipelines from document ingestion to
retrieval-augmented response generation, with systematic evaluation
and optimization at every stage.

## When to Use

- Designing a new RAG system from scratch
- Diagnosing "the LLM doesn't use my documents" problems
- Choosing between chunking strategies, embedding models, or vector databases
- Optimizing retrieval quality (precision, recall, relevance)
- Migrating from naive RAG to advanced RAG patterns
- Building evaluation harnesses for retrieval quality

## RAG Architecture Tiers

### Tier 1: Naive RAG

The simplest end-to-end pipeline. Good for prototyping.

```
Documents → Chunk → Embed → Store → Query → Retrieve → Generate
```

**Pros**: Fast to build, easy to understand
**Cons**: Poor handling of complex queries, no re-ranking, chunk boundary issues

### Tier 2: Advanced RAG

Adds pre-retrieval and post-retrieval optimization stages.

```
Documents → Clean → Chunk (smart) → Embed → Store
                                                ↓
Query → Rewrite → Embed → Retrieve → Re-rank → Filter → Generate
  ↑                                                        ↓
  └──────────── Self-reflection / Retry ──────────────────┘
```

**Key additions**:
- Query rewriting and expansion
- Hybrid search (dense + sparse)
- Re-ranking with cross-encoders
- Self-reflection loop for answer validation

### Tier 3: Modular RAG

Fully composable pipeline with pluggable components.

```
┌─── Ingestion Pipeline ────────────────────────────┐
│  Source Connectors → Parsing → Cleaning →          │
│  Chunking → Enrichment → Embedding → Indexing      │
└────────────────────────────────────────────────────┘

┌─── Query Pipeline ────────────────────────────────┐
│  Query Analysis → Intent Classification →          │
│  Query Routing → Retrieval Strategy Selection →    │
│  Multi-source Retrieval → Fusion → Re-ranking →   │
│  Context Assembly → Generation → Validation        │
└────────────────────────────────────────────────────┘
```

## Workflow

### Phase 1: Document Analysis & Ingestion Design

1. **Audit the corpus**:
   - Document types (PDF, HTML, Markdown, code, tables, images)
   - Total size and document count
   - Update frequency (static vs. streaming)
   - Language distribution
   - Structural complexity (headings, tables, nested lists, code blocks)

2. **Choose a parsing strategy**:

   | Document Type | Recommended Parser | Notes |
   |--------------|-------------------|-------|
   | PDF (text) | PyMuPDF / pdfplumber | Preserves layout and tables |
   | PDF (scanned) | Tesseract + layout detection | Add OCR preprocessing |
   | HTML | BeautifulSoup + Readability | Strip boilerplate first |
   | Markdown | Native parsing | Preserve heading hierarchy |
   | Code files | Tree-sitter | Parse by function/class boundaries |
   | Spreadsheets | pandas | Convert to structured text with headers |
   | Images/diagrams | Vision LLM (GPT-4V, Claude) | Generate text descriptions |

3. **Design the cleaning pipeline**:
   - Remove boilerplate (headers, footers, navigation)
   - Normalize whitespace and encoding
   - Extract and preserve metadata (title, author, date, source URL)
   - Handle special characters and formatting artifacts

### Phase 2: Chunking Strategy

Choose the chunking approach based on document structure:

| Strategy | Best For | Chunk Size | Overlap |
|----------|----------|-----------|---------|
| **Fixed-size** | Homogeneous text (articles, books) | 512-1024 tokens | 10-20% |
| **Recursive character** | General purpose | 500-1000 tokens | 50-200 tokens |
| **Semantic** | Mixed-content documents | Variable | Natural boundaries |
| **Document-structure** | Technical docs with headings | Section-based | Include parent heading |
| **Code-aware** | Source code | Function/class-level | Include imports/context |
| **Sentence-window** | High-precision retrieval | 1-3 sentences | ±2 sentences as context |
| **Parent-child** | Hierarchical documents | Small child, large parent | Child retrieves, parent provides context |

**Chunking decision tree**:

```
Is the document well-structured (headings, sections)?
├── Yes → Document-structure chunking
│         + Parent-child indexing for hierarchy
├── No → Is it source code?
│        ├── Yes → Code-aware chunking (tree-sitter)
│        └── No → Is high precision critical?
│                 ├── Yes → Sentence-window chunking
│                 └── No → Recursive character chunking
```

**Critical rules**:
- Always include metadata in chunks (source, page, section title)
- Never break mid-sentence or mid-code-block
- Test chunk quality by reading 20 random chunks — each should be self-contained and meaningful

### Phase 3: Embedding & Indexing

1. **Choose embedding model**:

   | Model | Dimensions | Max Tokens | Strengths |
   |-------|-----------|-----------|-----------|
   | OpenAI text-embedding-3-large | 3072 | 8191 | Best general quality |
   | OpenAI text-embedding-3-small | 1536 | 8191 | Cost-effective |
   | Cohere embed-v4 | 1024 | 512 | Multilingual excellence |
   | Voyage AI voyage-3-large | 1024 | 32000 | Long-context, code |
   | BGE-M3 (open source) | 1024 | 8192 | Multilingual, free |
   | Jina embeddings-v3 | 1024 | 8192 | Multilingual, open |
   | GTE-Qwen2 (open source) | 1024 | 8192 | Chinese + English, free |

2. **Choose vector database**:

   | Database | Type | Best For | Scale |
   |----------|------|---------|-------|
   | Pinecone | Managed | Production, zero-ops | Billions |
   | Weaviate | Managed/Self | Hybrid search, multi-modal | Millions-Billions |
   | Qdrant | Self-hosted | Performance, filtering | Millions |
   | ChromaDB | Embedded | Prototyping, local dev | Thousands-Millions |
   | pgvector | Postgres extension | Existing Postgres stack | Millions |
   | FAISS | In-memory library | Research, benchmarking | Millions |
   | Milvus | Distributed | Large-scale production | Billions |

3. **Indexing best practices**:
   - Store both the embedding vector AND the original text
   - Include rich metadata for filtering (source, date, category, language)
   - Build separate indexes for different document types if needed
   - Implement incremental indexing for updates (don't re-index everything)

### Phase 4: Retrieval Strategy

Design the query-time retrieval pipeline:

1. **Query preprocessing**:
   - Detect query intent (factual, analytical, procedural, conversational)
   - Expand query with synonyms or related terms (query expansion)
   - Rewrite ambiguous queries using the LLM (HyDE: Hypothetical Document Embeddings)
   - Decompose complex queries into sub-queries

2. **Retrieval methods** (choose one or combine):

   | Method | How It Works | When to Use |
   |--------|-------------|-------------|
   | **Dense retrieval** | Embedding similarity (cosine/dot product) | Semantic meaning matches |
   | **Sparse retrieval** | BM25 / TF-IDF keyword matching | Exact term matches, names, codes |
   | **Hybrid** | Dense + Sparse with score fusion | Best overall quality |
   | **Multi-query** | Generate N query variants, retrieve for each, merge | Complex or ambiguous queries |
   | **Parent-child** | Retrieve small chunks, return parent chunks | Need both precision and context |

3. **Re-ranking**:
   - Apply a cross-encoder re-ranker (e.g., Cohere Rerank, BGE-Reranker)
   - Re-ranking is the single highest-impact improvement for most RAG systems
   - Retrieve more candidates than needed (e.g., top-20), re-rank to top-5

4. **Post-retrieval filtering**:
   - Remove chunks below a relevance score threshold
   - Deduplicate near-identical chunks
   - Ensure diversity (don't return 5 chunks from the same page)
   - Respect recency requirements (filter by date if needed)

### Phase 5: Context Assembly & Generation

1. **Assemble the context**:
   - Order chunks by relevance (most relevant first AND last — avoid "lost in the middle")
   - Include source attribution metadata
   - Add a preamble: "Answer based ONLY on the following context. If the context
     doesn't contain the answer, say so."

2. **Generation prompt template**:

   ```
   You are a [domain] expert. Answer the user's question based on the
   provided context.

   ## Rules
   - Only use information from the provided context
   - Cite sources using [Source: document_name, page X]
   - If the context is insufficient, explicitly state what's missing
   - Never fabricate information

   ## Context
   {retrieved_chunks_with_metadata}

   ## Question
   {user_query}

   ## Answer
   ```

3. **Response validation**:
   - Check for hallucination: Does every claim have a supporting chunk?
   - Check for completeness: Did the response address all parts of the query?
   - Check for citation accuracy: Do citations point to the right chunks?

### Phase 6: Evaluation

Evaluate the RAG pipeline at two levels:

**Retrieval evaluation**:

| Metric | What It Measures | Target |
|--------|-----------------|--------|
| **Recall@K** | % of relevant docs in top-K results | > 0.85 |
| **Precision@K** | % of top-K results that are relevant | > 0.70 |
| **MRR** (Mean Reciprocal Rank) | How high the first relevant result ranks | > 0.80 |
| **NDCG** | Quality of the ranking order | > 0.75 |

**End-to-end evaluation**:

| Metric | What It Measures | Method |
|--------|-----------------|--------|
| **Faithfulness** | Does the answer stick to the context? | LLM-as-judge |
| **Relevance** | Does the answer address the question? | LLM-as-judge |
| **Completeness** | Are all aspects of the question covered? | Human eval + LLM |
| **Citation accuracy** | Do citations match the claims? | Automated check |

**Build an eval dataset**:
- Minimum 50 question-answer pairs with annotated relevant documents
- Include: easy factual, multi-hop reasoning, unanswerable, and adversarial queries
- Version control the eval set and never train/tune on it

## Common Failure Modes & Fixes

| Symptom | Root Cause | Fix |
|---------|-----------|-----|
| Answers are generic, don't use context | Weak retrieval, chunks not relevant | Improve chunking, add re-ranker |
| Answers hallucinate facts | Context insufficient, no guardrails | Add "only use context" constraint, validate |
| Answers are correct but miss details | Chunks too small, key info split | Increase chunk size or use parent-child |
| Wrong documents retrieved | Query-document vocabulary mismatch | Add hybrid search, query expansion |
| Good retrieval but bad answers | Poor prompt engineering | Improve generation prompt, add examples |
| Slow response times | Too many chunks, large context | Reduce top-K, use faster embedding model |
| Inconsistent quality | No evaluation framework | Build eval harness, monitor metrics |

## Guidelines

- **Start simple, iterate**: Begin with Tier 1 (naive RAG), measure, then add
  complexity only where metrics show gaps.
- **Re-ranking is the best single upgrade**: If you do one thing to improve quality,
  add a cross-encoder re-ranker.
- **Chunk quality > chunk quantity**: 5 excellent chunks beat 20 mediocre ones.
- **Always evaluate**: Without metrics, you're guessing. Build the eval harness
  before optimizing.
- **Version everything**: Embedding models, chunk strategies, and prompts should
  all be versioned and traceable.
