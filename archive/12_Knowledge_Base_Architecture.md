# 12 -- Knowledge Base Architecture

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Version:** 1.0\
> **Architecture:** Offline-First\
> **Vector Store:** ChromaDB\
> **LLM Runtime:** Ollama

------------------------------------------------------------------------

# 1. Purpose

The Knowledge Base (KB) is the foundation of the RAG subsystem. It
stores cybersecurity knowledge locally so ARIA can enrich ML predictions
without requiring internet access.

Design goals:

-   100% offline operation
-   Version-controlled knowledge
-   Fast semantic retrieval
-   Source traceability
-   Easy updates
-   Organization-specific customization

------------------------------------------------------------------------

# 2. High-Level Architecture

``` mermaid
flowchart LR
Docs[PDF/MD/TXT]
-->Loader
-->Cleaner
-->Chunker
-->Embedding
-->ChromaDB
ChromaDB-->Retriever
Retriever-->PromptBuilder
PromptBuilder-->Ollama
Ollama-->Investigation
```

------------------------------------------------------------------------

# 3. Knowledge Sources

  Source               Purpose
  -------------------- ----------------------------------
  MITRE ATT&CK         Techniques, tactics, mitigations
  CVE Database         Vulnerability descriptions
  CAPEC                Attack patterns
  CERT-In Advisories   Indian cybersecurity advisories
  CISA Advisories      Global threat advisories
  Organization SOPs    Internal procedures
  Incident Playbooks   Investigation checklists
  Security Policies    Organizational policies

All resources are downloaded manually and stored locally.

------------------------------------------------------------------------

# 4. Directory Structure

``` text
knowledge_base/
├── MITRE/
├── CVE/
├── CAPEC/
├── CERT-IN/
├── CISA/
├── Playbooks/
├── SOPs/
├── Policies/
├── Threat_Reports/
└── Organization/
```

------------------------------------------------------------------------

# 5. Document Ingestion Pipeline

``` mermaid
flowchart TD
A[Local Documents]
-->B[Loader]
-->C[Text Extraction]
-->D[Cleaning]
-->E[Chunking]
-->F[Embedding Generation]
-->G[Metadata Creation]
-->H[ChromaDB]
```

Steps:

1.  Load supported documents
2.  Extract plain text
3.  Remove formatting noise
4.  Split into semantic chunks
5.  Generate embeddings
6.  Store vectors and metadata

------------------------------------------------------------------------

# 6. Supported Formats

-   PDF
-   Markdown (.md)
-   Text (.txt)
-   JSON
-   CSV (reference data)

------------------------------------------------------------------------

# 7. Chunking Strategy

  Parameter           Value
  ------------------- ---------------------------
  Chunk Size          500--800 tokens
  Overlap             100--150 tokens
  Strategy            Recursive character split
  Preserve Headings   Yes

------------------------------------------------------------------------

# 8. Metadata Schema

Each chunk stores:

``` json
{
  "document_id":"uuid",
  "title":"MITRE T1110",
  "source":"MITRE",
  "category":"Credential Access",
  "version":"1.0",
  "tags":["Brute Force"],
  "created_at":"ISO8601"
}
```

------------------------------------------------------------------------

# 9. Embedding Architecture

Embedding Model:

-   sentence-transformers
-   all-MiniLM-L6-v2

Pipeline:

``` mermaid
flowchart LR
Chunk-->EmbeddingModel-->Vector-->ChromaDB
```

Embeddings are generated locally.

------------------------------------------------------------------------

# 10. ChromaDB Collections

  Collection     Description
  -------------- ------------------------
  mitre          ATT&CK techniques
  cve            Vulnerabilities
  capec          Attack patterns
  certin         CERT-In advisories
  playbooks      Response playbooks
  organization   Internal documentation

------------------------------------------------------------------------

# 11. Retrieval Strategy

1.  Receive ML prediction
2.  Build semantic query
3.  Generate query embedding
4.  Search ChromaDB
5.  Retrieve Top-K chunks
6.  Rank by similarity
7.  Pass context to Prompt Builder

Top-K = 5--10 documents.

------------------------------------------------------------------------

# 12. Knowledge Versioning

``` text
knowledge_base/
└── versions/
    ├── v1/
    ├── v2/
    └── archive/
```

Each update includes:

-   version number
-   update date
-   source
-   checksum
-   indexed status

------------------------------------------------------------------------

# 13. Offline Update Workflow

``` mermaid
flowchart TD
Download-->LocalFolder
LocalFolder-->Indexer
Indexer-->EmbeddingGeneration
EmbeddingGeneration-->ChromaDB
```

Updates are performed manually; runtime never depends on internet
connectivity.

------------------------------------------------------------------------

# 14. Security

-   Local storage only
-   Read-only KB during inference
-   Hash verification for imported documents
-   Audit logs for indexing operations
-   No external API calls

------------------------------------------------------------------------

# 15. Performance Targets

  Operation                        Target
  ---------------------- ----------------
  Document Loading               \<500 ms
  Chunk Retrieval                \<150 ms
  Embedding Generation     \<100 ms/chunk
  Top-K Search                   \<200 ms

------------------------------------------------------------------------

# 16. Hardware Optimization

Target System:

-   AMD Ryzen 7 7840HS
-   16 GB DDR5
-   RTX 3050 Laptop GPU (6 GB)
-   NVMe SSD

CPU handles ingestion and indexing; GPU remains available for Ollama
inference.

------------------------------------------------------------------------

# 17. Failure Handling

  Failure                Recovery
  ---------------------- -------------------------
  Missing document       Skip and log
  Corrupted file         Quarantine
  Embedding failure      Retry
  ChromaDB unavailable   Return ML-only response

------------------------------------------------------------------------

# 18. Future Improvements

-   Hybrid BM25 + Vector search
-   Cross-encoder re-ranking
-   Incremental indexing
-   Automatic duplicate detection
-   Knowledge freshness scoring
-   Multi-language knowledge support

------------------------------------------------------------------------

# Next Document

**13_Prompt_Engineering_Architecture.md** -- Prompt templates,
orchestration, response schema, hallucination mitigation, and structured
output validation.
