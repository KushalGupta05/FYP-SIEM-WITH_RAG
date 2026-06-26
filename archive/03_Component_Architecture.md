# 03 -- Component Architecture

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Document:** Component Architecture Specification\
> **Version:** 1.0

------------------------------------------------------------------------

# 1. Purpose

This document decomposes ARIA into deployable components, internal
modules, interfaces, responsibilities, dependencies, and runtime
interactions.

The objective is to define each subsystem clearly enough that different
developers can implement them independently while preserving a
consistent architecture.

------------------------------------------------------------------------

# 2. Component Decomposition

``` text
                        +----------------------------+
                        |        Streamlit UI        |
                        +-------------+--------------+
                                      |
                               REST / HTTPS
                                      |
                        +-------------v--------------+
                        |        FastAPI API         |
                        +------+------+------+-------+
                               |      |      |
             ------------------       |      -----------------
             |                        |                        |
             v                        v                        v
 +--------------------+    +-------------------+   +-------------------+
 | Alert Pipeline     |    | ML Service        |   | RAG Service       |
 +---------+----------+    +---------+---------+   +---------+---------+
           |                         |                       |
           v                         |                       v
 +--------------------+              |             +--------------------+
 | Feature Extractor  |              |             | Retriever          |
 +---------+----------+              |             +---------+----------+
           |                         |                       |
           v                         |                       v
 +--------------------+              |             +--------------------+
 | Decision Engine    |--------------+-------------| Prompt Builder     |
 +---------+----------+                            +---------+----------+
           |                                                 |
           v                                                 v
 +--------------------+                            +--------------------+
 | Feedback Engine    |                            | Ollama LLM         |
 +---------+----------+                            +---------+----------+
           |                                                 |
           +--------------------------+----------------------+
                                      |
                                      v
                    PostgreSQL / SQLite / ChromaDB
```

------------------------------------------------------------------------

# 3. Component Catalog

  Component           Responsibility                        State
  ------------------- ------------------------------------- -----------
  API Gateway         Entry point for all requests          Stateless
  Alert Pipeline      Validate and normalize SIEM alerts    Stateless
  Feature Extractor   Produce ML features                   Stateless
  ML Service          Predict Benign/Suspicious/Malicious   Stateless
  Decision Engine     Coordinate workflow                   Stateless
  Retriever           Semantic search                       Stateless
  Prompt Builder      Build LLM prompt                      Stateless
  LLM Adapter         Call Ollama                           Stateless
  Feedback Engine     Persist analyst decisions             Stateful
  Audit Store         Investigation history                 Stateful

------------------------------------------------------------------------

# 4. Alert Pipeline

## Responsibilities

-   Receive SIEM webhook
-   Validate schema
-   Reject malformed alerts
-   Generate Alert ID
-   Normalize fields
-   Timestamp ingestion
-   Publish normalized object

### Input

``` json
{
  "source":"Wazuh",
  "severity":8,
  "event":"Brute Force",
  "ip":"192.168.1.10"
}
```

### Output

Canonical Alert Object

``` json
{
  "alertId":"UUID",
  "severity":8,
  "source":"Wazuh",
  "normalizedEvent":"AUTH_BRUTE_FORCE",
  "receivedAt":"ISO8601"
}
```

Failure handling: - Invalid schema → HTTP 400 - Missing fields →
quarantine queue - Duplicate IDs → ignore and audit

------------------------------------------------------------------------

# 5. Feature Extraction Component

Responsibilities

-   Numerical encoding
-   Feature scaling
-   Missing value handling
-   Feature ordering
-   Version tracking

Pipeline

``` text
Raw Alert
   ↓
Cleaning
   ↓
Encoding
   ↓
Scaling
   ↓
Feature Vector
```

Output is compatible with every supported ML model.

------------------------------------------------------------------------

# 6. ML Service

Responsibilities

-   Load serialized model
-   Execute inference
-   Produce confidence
-   Return probability vector

Input

Feature Vector

Output

``` json
{
 "prediction":"Malicious",
 "confidence":0.94,
 "probabilities":{
   "Benign":0.01,
   "Suspicious":0.05,
   "Malicious":0.94
 }
}
```

Internal modules

-   Model Loader
-   Inference Engine
-   Confidence Calculator
-   Metrics Collector

Future models

-   Random Forest
-   XGBoost
-   LightGBM
-   Neural Networks

------------------------------------------------------------------------

# 7. Decision Engine

Purpose

Acts as the orchestration layer.

Rules

IF confidence \< threshold → Manual Review

IF malicious → Execute RAG

IF benign → Generate short summary

IF suspicious → Deep retrieval

No business logic exists inside the UI.

------------------------------------------------------------------------

# 8. RAG Service

Subcomponents

-   Query Generator
-   Embedding Generator
-   Retriever
-   Context Ranker
-   Prompt Builder
-   Citation Builder

Processing Flow

``` text
Alert
 ↓
Query Expansion
 ↓
Embedding
 ↓
Vector Search
 ↓
Ranking
 ↓
Prompt Construction
 ↓
LLM
```

Knowledge Sources

-   MITRE ATT&CK
-   CVE
-   CAPEC
-   CERT-In advisories
-   Internal playbooks
-   Organization SOPs

------------------------------------------------------------------------

# 9. Prompt Builder

Sections

1.  System Prompt
2.  Alert Context
3.  ML Prediction
4.  Retrieved Documents
5.  MITRE Mapping
6.  Output Format

Design goals

-   Deterministic
-   Minimal hallucination
-   Structured JSON response

------------------------------------------------------------------------

# 10. Dashboard

Views

-   Live Alerts
-   Investigation
-   Evidence
-   MITRE Matrix
-   Feedback
-   Analytics

Dashboard never performs inference; it only consumes APIs.

------------------------------------------------------------------------

# 11. Feedback Engine

Stores

-   Analyst verdict
-   False positive
-   Escalated
-   Notes
-   Investigation duration

Future use

-   Dataset enrichment
-   Evaluation
-   Model retraining

------------------------------------------------------------------------

# 12. Persistence Layer

## PostgreSQL

-   Alerts
-   Investigations
-   Feedback
-   Users
-   Audit logs

## ChromaDB

-   Threat intelligence embeddings
-   MITRE embeddings
-   Playbooks
-   CERT advisories

------------------------------------------------------------------------

# 13. Component Dependencies

``` text
Dashboard
    ↓
FastAPI
    ↓
Decision Engine
 ├── ML
 ├── RAG
 ├── Feedback
 └── Database

RAG
 ├── Embeddings
 ├── ChromaDB
 └── Ollama
```

------------------------------------------------------------------------

# 14. Design Patterns

  Pattern                Usage
  ---------------------- --------------------
  Repository             Database access
  Strategy               ML model selection
  Factory                Model loading
  Adapter                SIEM connectors
  Facade                 RAG orchestration
  Dependency Injection   FastAPI services

------------------------------------------------------------------------

# 15. Error Handling

  Failure                Behaviour
  ---------------------- --------------------------
  ML unavailable         Manual review
  Vector DB offline      Return ML result only
  LLM timeout            Show retrieved evidence
  DB failure             Retry then persist audit
  Invalid SIEM payload   Reject request

------------------------------------------------------------------------

# 16. Deployment Boundaries

## FYP

Single Docker Compose stack

-   FastAPI
-   Streamlit
-   Ollama
-   PostgreSQL
-   ChromaDB

## Production

Independent services with Kubernetes, API Gateway, Redis, RabbitMQ/Kafka
and autoscaling.

------------------------------------------------------------------------

# 17. Component Sequence (Summary)

``` text
SIEM
 ↓
API
 ↓
Normalize
 ↓
Extract Features
 ↓
ML
 ↓
Decision Engine
 ↓
Retriever
 ↓
Prompt Builder
 ↓
LLM
 ↓
Dashboard
 ↓
Feedback
```

------------------------------------------------------------------------

# 18. Next Document

**04_Data_Flow_Architecture.md**

This document will describe every data object, event, state transition,
API payload, and end-to-end lifecycle from alert ingestion to
investigation closure.

------------------------------------------------------------------------

End of Document
