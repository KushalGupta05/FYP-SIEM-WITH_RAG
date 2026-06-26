# 11 -- RAG Architecture

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Version:** 1.0\
> **Architecture:** Offline-First\
> **LLM Runtime:** Ollama\
> **Vector Database:** ChromaDB

------------------------------------------------------------------------

# 1. Purpose

This document defines the Retrieval-Augmented Generation (RAG)
architecture used by ARIA. The RAG subsystem enriches ML predictions
with cybersecurity knowledge and generates structured investigation
reports using a local LLM.

The entire pipeline executes offline.

------------------------------------------------------------------------

# 2. Objectives

-   Operate without cloud services
-   Improve investigation quality
-   Reduce LLM hallucinations
-   Provide evidence-backed responses
-   Retrieve organization-specific knowledge
-   Support explainable investigations

------------------------------------------------------------------------

# 3. High-Level Architecture

``` mermaid
flowchart LR
Alert-->MLPrediction
MLPrediction-->QueryBuilder
QueryBuilder-->EmbeddingModel
EmbeddingModel-->ChromaDB
ChromaDB-->Retriever
Retriever-->ContextRanker
ContextRanker-->PromptBuilder
PromptBuilder-->Ollama
Ollama-->ResponseValidator
ResponseValidator-->InvestigationBrief
```

------------------------------------------------------------------------

# 4. Core Components

  Component            Responsibility
  -------------------- ------------------------------
  Query Builder        Builds semantic search query
  Embedding Model      Generates vector embeddings
  ChromaDB             Stores embeddings
  Retriever            Performs similarity search
  Context Ranker       Ranks retrieved chunks
  Prompt Builder       Creates structured prompt
  Ollama Runtime       Runs local LLM
  Response Validator   Validates JSON structure
  Citation Engine      Links evidence to sources

------------------------------------------------------------------------

# 5. Knowledge Base

Local knowledge sources:

``` text
knowledge_base/
├── MITRE/
├── CVE/
├── CERT-In/
├── CAPEC/
├── Playbooks/
├── SOPs/
└── Organization/
```

Each document is indexed into ChromaDB.

------------------------------------------------------------------------

# 6. Knowledge Ingestion Pipeline

``` mermaid
flowchart TD
PDF/Markdown-->Loader
Loader-->Cleaner
Cleaner-->Chunker
Chunker-->EmbeddingModel
EmbeddingModel-->ChromaDB
```

Steps:

1.  Load document
2.  Extract text
3.  Remove noise
4.  Chunk document
5.  Generate embeddings
6.  Store vectors with metadata

------------------------------------------------------------------------

# 7. Chunking Strategy

-   Chunk Size: 500--800 tokens
-   Chunk Overlap: 100--150 tokens
-   Preserve headings
-   Preserve document source
-   Preserve section metadata

Metadata stored:

-   document_id
-   title
-   source
-   section
-   version
-   tags

------------------------------------------------------------------------

# 8. Embedding Model

Offline embedding model:

-   sentence-transformers
-   all-MiniLM-L6-v2 (default)

Embeddings are generated locally and never leave the machine.

------------------------------------------------------------------------

# 9. ChromaDB Collections

  Collection     Contents
  -------------- -------------------------
  mitre          ATT&CK techniques
  cve            CVE descriptions
  certin         CERT-In advisories
  playbooks      Investigation playbooks
  organization   Internal documentation

------------------------------------------------------------------------

# 10. Retrieval Pipeline

``` mermaid
sequenceDiagram
participant API
participant QB as Query Builder
participant EMB as Embedding Model
participant DB as ChromaDB
participant RET as Retriever

API->>QB: Alert + Prediction
QB->>EMB: Query
EMB->>DB: Vector Search
DB-->>RET: Top-K Results
RET-->>API: Ranked Context
```

Top-K: 5--10 documents

Similarity search uses cosine distance.

------------------------------------------------------------------------

# 11. Prompt Engineering

Prompt Sections:

1.  System Instructions
2.  Alert Details
3.  ML Prediction
4.  Retrieved Context
5.  MITRE Mapping
6.  Output Schema

Expected Output:

-   Executive Summary
-   Threat Analysis
-   MITRE ATT&CK
-   Evidence
-   Recommended Actions
-   Analyst Checklist

------------------------------------------------------------------------

# 12. Ollama Integration

Recommended Models:

  Model           Purpose
  --------------- ----------------
  Mistral 7B Q4   Default
  Qwen2.5 3B      Lightweight
  Phi-3 Mini      Fast inference

``` mermaid
flowchart LR
Prompt-->Ollama
Ollama-->LocalLLM
LocalLLM-->JSONResponse
```

No external API calls are made.

------------------------------------------------------------------------

# 13. Response Validation

The validator checks:

-   Valid JSON
-   Required fields present
-   Confidence values
-   Citation availability
-   Recommendation formatting

Invalid responses are regenerated or flagged.

------------------------------------------------------------------------

# 14. Citation Engine

Each recommendation references retrieved evidence.

Example:

``` json
{
  "recommendation":"Block source IP",
  "source":"MITRE ATT&CK T1110",
  "confidence":0.95
}
```

------------------------------------------------------------------------

# 15. Failure Handling

  Failure                Recovery
  ---------------------- ----------------------------
  ChromaDB unavailable   Return ML result only
  Ollama offline         Display retrieved evidence
  Embedding failure      Retry generation
  Invalid LLM output     Re-prompt validator

------------------------------------------------------------------------

# 16. Performance Targets

  Stage                Target
  --------------- -----------
  Embedding          \<100 ms
  Retrieval          \<150 ms
  Prompt Build        \<20 ms
  LLM Inference     \<3000 ms
  Validation          \<50 ms

------------------------------------------------------------------------

# 17. Hardware Optimization

Target Hardware:

-   AMD Ryzen 7 7840HS
-   16 GB RAM
-   RTX 3050 Laptop GPU (6 GB VRAM)

Allocation:

-   CPU: Retrieval, embeddings, API
-   GPU: Ollama inference
-   SSD: ChromaDB + Knowledge Base

------------------------------------------------------------------------

# 18. Security

-   Local-only execution
-   No cloud APIs
-   Encrypted PostgreSQL storage
-   Local ChromaDB
-   Prompt logging with hashes
-   Audit trail for investigations

------------------------------------------------------------------------

# 19. Future Improvements

-   Hybrid BM25 + Vector Retrieval
-   Cross-Encoder Re-ranking
-   Knowledge versioning
-   Incremental indexing
-   Multi-model routing
-   Automatic document freshness checks

------------------------------------------------------------------------

# 20. End-to-End RAG Flow

``` mermaid
flowchart TD
Alert-->MLPrediction
MLPrediction-->Query
Query-->Embedding
Embedding-->ChromaDB
ChromaDB-->Retriever
Retriever-->Prompt
Prompt-->Ollama
Ollama-->Validator
Validator-->InvestigationBrief
InvestigationBrief-->Dashboard
```

------------------------------------------------------------------------

# Next Document

**12_Deployment_Architecture.md** -- Docker Compose, service topology,
local networking, startup sequence, resource allocation, and offline
deployment guide.
