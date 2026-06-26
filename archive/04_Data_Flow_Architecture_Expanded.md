# 04 -- Data Flow Architecture (Expanded Engineering Specification)

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Revision:** 2.0

------------------------------------------------------------------------

# 1. Document Purpose

The purpose of this document is to define **how data is created,
transformed, enriched, validated, stored, secured, and retired**
throughout the lifetime of an investigation inside ARIA.

Unlike a conventional Data Flow Diagram (DFD), this specification
defines:

-   Business data flow
-   Technical data flow
-   Control flow
-   Event flow
-   Error flow
-   Security flow
-   Data ownership
-   Trust boundaries
-   Storage lifecycle
-   Observability lifecycle

------------------------------------------------------------------------

# 2. Guiding Principles

## 2.1 Immutability

The original SIEM alert is **never modified**. Every downstream
transformation produces a new derived object while preserving the raw
event for forensic purposes.

## 2.2 Canonical Schema

Vendor-specific alert formats are converted into a single canonical
representation before any ML or RAG processing occurs.

## 2.3 Explainability

Every generated recommendation must be traceable to:

-   ML prediction
-   Retrieved evidence
-   Prompt version
-   Knowledge sources
-   Analyst feedback

------------------------------------------------------------------------

# 3. Trust Boundaries

``` mermaid
flowchart LR
SIEM -->|External| Gateway
Gateway -->|Internal| Core
Core --> ML
Core --> RAG
RAG --> LLM
Core --> Database
Dashboard --> Analyst
```

External systems are never granted direct access to internal storage.

------------------------------------------------------------------------

# 4. Complete End-to-End Lifecycle

``` mermaid
flowchart TD
A[SIEM Alert]
-->B[Gateway]
-->C[Validation]
-->D[Normalization]
-->E[Feature Extraction]
-->F[ML Inference]
-->G[Decision Engine]
-->H[RAG Context Builder]
-->I[Embedding]
-->J[Vector Search]
-->K[Prompt Builder]
-->L[LLM]
-->M[Investigation Package]
-->N[Dashboard]
-->O[Analyst Review]
-->P[Feedback]
-->Q[Audit]
-->R[Evaluation]
```

Each stage has a clearly defined input, output, owner, timeout, and
failure policy.

------------------------------------------------------------------------

# 5. Stage Specifications

## Stage 1 -- Alert Gateway

### Responsibilities

-   Authenticate SIEM
-   Validate API key
-   Generate Request ID
-   Timestamp request
-   Rate limiting
-   Duplicate detection

### Inputs

-   JSON webhook
-   REST request
-   Future message queue

### Outputs

Canonical ingress envelope.

### Failure Handling

  Failure              Response
  -------------------- ----------------
  Invalid API key      401
  Payload too large    413
  Unsupported schema   422
  Duplicate request    Ignore + Audit

------------------------------------------------------------------------

## Stage 2 -- Validation

Validation categories:

-   Structural
-   Semantic
-   Security
-   Business

Examples:

-   Severity must be 0--10
-   Timestamp must be ISO-8601
-   IP must be valid IPv4/IPv6
-   Hostname length \<255

Rejected alerts are quarantined rather than discarded.

------------------------------------------------------------------------

## Stage 3 -- Normalization

Maps vendor-specific fields to a canonical schema.

Example:

  Vendor    Raw Field      Canonical
  --------- -------------- ---------------
  Wazuh     rule.id        detectionRule
  Elastic   event.action   eventType
  Graylog   source         hostname

Benefits:

-   Vendor independence
-   Stable downstream APIs
-   Easier testing

------------------------------------------------------------------------

## Stage 4 -- Feature Engineering

``` mermaid
flowchart LR
Raw-->Cleaning
Cleaning-->Encoding
Encoding-->Scaling
Scaling-->FeatureVector
```

Responsibilities:

-   Missing-value handling
-   Encoding
-   Scaling
-   Feature ordering
-   Feature versioning

------------------------------------------------------------------------

## Stage 5 -- ML Inference

Outputs:

-   Prediction
-   Confidence
-   Probability distribution
-   Model version
-   Feature checksum

Decision thresholds are configurable and version-controlled.

------------------------------------------------------------------------

## Stage 6 -- RAG Retrieval

``` mermaid
sequenceDiagram
participant DE as Decision Engine
participant RT as Retriever
participant DB as ChromaDB
participant PB as Prompt Builder
participant L as LLM

DE->>RT: Search(query)
RT->>DB: Similarity Search
DB-->>RT: Ranked Chunks
RT-->>PB: Context
PB->>L: Prompt
L-->>PB: Investigation JSON
```

Knowledge sources include MITRE ATT&CK, CVEs, CERT-In advisories,
playbooks, and internal SOPs.

------------------------------------------------------------------------

# 6. Canonical Data Objects

## Alert

Lifecycle:

Received → Validated → Normalized → Classified → Archived

Key fields:

-   alertId
-   source
-   severity
-   eventType
-   hostname
-   username
-   timestamp
-   IOC list

## Investigation

Contains:

-   prediction
-   confidence
-   evidence
-   retrieved documents
-   analyst notes
-   final verdict

## Feedback

Stores:

-   analyst decision
-   false-positive flag
-   escalation
-   review duration

------------------------------------------------------------------------

# 7. Event Catalogue

  Event                    Publisher        Subscribers
  ------------------------ ---------------- -----------------
  AlertReceived            Gateway          Validator
  AlertValidated           Validator        Normalizer
  FeaturesGenerated        Feature Engine   ML
  PredictionCreated        ML               Decision Engine
  RetrievalCompleted       Retriever        Prompt Builder
  InvestigationGenerated   LLM              Dashboard
  FeedbackSubmitted        Dashboard        Feedback Engine

Events are idempotent and uniquely identified by Request ID.

------------------------------------------------------------------------

# 8. Storage Lifecycle

``` mermaid
flowchart LR
Alert-->PostgreSQL
Evidence-->ChromaDB
Feedback-->PostgreSQL
Audit-->PostgreSQL
Embedding-->ChromaDB
```

Retention policy:

-   Alerts: configurable
-   Audit logs: long-term
-   Embeddings: regenerated when knowledge changes

------------------------------------------------------------------------

# 9. Security Data Flow

Security controls applied at every transition:

-   TLS in transit
-   AES encryption at rest
-   RBAC
-   Immutable audit logs
-   Prompt hashing
-   Input sanitization

Sensitive fields may be masked before being included in LLM prompts.

------------------------------------------------------------------------

# 10. Failure & Recovery

``` mermaid
flowchart TD
MLFail-->ManualReview
RetrieverFail-->MLOnly
LLMFail-->EvidenceOnly
DBFail-->Retry
Retry-->Audit
```

Failures never delete alerts. Partial results remain visible to
analysts.

------------------------------------------------------------------------

# 11. Observability

Metrics:

-   alert_processing_latency
-   ml_latency
-   retrieval_latency
-   llm_latency
-   dashboard_latency

Logs include:

-   Request ID
-   Alert ID
-   Model Version
-   Prompt Hash
-   Retrieval IDs
-   Final Verdict

Tracing follows the Request ID across every service.

------------------------------------------------------------------------

# 12. Performance Budget

  Stage                    Target
  --------------------- ---------
  Gateway                   10 ms
  Validation                20 ms
  Feature Engineering       40 ms
  ML                       100 ms
  Retrieval                150 ms
  Prompt Builder            20 ms
  LLM                     2500 ms
  Dashboard                100 ms

Target end-to-end latency: **\<3 seconds** (excluding unusually large
prompts).

------------------------------------------------------------------------

# 13. Future Evolution

-   Kafka-based event streaming
-   Redis distributed cache
-   Multi-region deployment
-   Multi-tenant isolation
-   Online evaluation
-   Real-time analytics
-   SOAR integration

------------------------------------------------------------------------

# Appendix A -- Recommended Follow-up Documents

1.  05_Domain_Model.md
2.  06_Database_Design.md
3.  07_API_Specification.md
4.  08_ML_Architecture.md
5.  09_RAG_Architecture.md
