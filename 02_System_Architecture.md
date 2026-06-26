# 02 -- System Architecture

> **Project:** ARIA --- Autonomous SIEM Triage Assistant\
> **Version:** 1.0

------------------------------------------------------------------------

# 1. Architectural Overview

ARIA is an event-driven, modular AI platform that augments Security
Operations Center (SOC) analysts by automating alert triage. The
platform separates responsibilities into independent services so that
the ML classifier, RAG engine, dashboard, and storage can evolve
independently.

## Architectural Goals

-   Modular design
-   Explainable AI
-   Vendor-agnostic SIEM integration
-   Human-in-the-loop decisions
-   Extensible microservice architecture
-   Production-inspired while remaining FYP implementable

------------------------------------------------------------------------

# 2. High-Level Architecture

``` text
              +----------------------+
              | SIEM (Wazuh/Elastic) |
              +----------+-----------+
                         |
                         v
                Alert Ingestion Service
                         |
                         v
                Alert Normalization
                         |
        +----------------+----------------+
        |                                 |
        v                                 v
 ML Classification Service         Metadata Enrichment
        |                                 |
        +---------------+-----------------+
                        |
                        v
                 Decision Engine
                        |
                        v
               RAG Context Builder
                        |
                        v
      +-----------------+------------------+
      | Vector Database | Threat Knowledge |
      +-----------------+------------------+
                        |
                        v
                  Prompt Builder
                        |
                        v
                    Local LLM
                        |
                        v
           Investigation Brief Generator
                        |
                        v
               Analyst Dashboard/API
                        |
                        v
                 Feedback Engine
                        |
                        v
           Knowledge & Audit Storage
```

------------------------------------------------------------------------

# 3. C4 Level 1 -- System Context

## External Actors

-   Tier-1 SOC Analyst
-   SOC Manager
-   Open-source SIEM
-   Threat Intelligence Sources

## System Boundary

ARIA accepts alerts from SIEM platforms, enriches them with ML and RAG,
and produces an Investigation Briefing for analysts.

------------------------------------------------------------------------

# 4. Major Components

  Component           Responsibility
  ------------------- ------------------------------------------
  Alert Ingestion     Receives alerts from SIEM
  Normalizer          Converts alerts into a canonical schema
  Feature Extractor   Builds ML-ready features
  ML Service          Predicts Benign / Suspicious / Malicious
  Decision Engine     Routes alerts through workflow
  Retriever           Searches vector database
  Prompt Builder      Constructs LLM prompt
  LLM Service         Generates investigation briefing
  Dashboard           Presents findings
  Feedback Engine     Stores analyst feedback
  Audit Store         Persists history and evidence

------------------------------------------------------------------------

# 5. Data Flow

1.  SIEM emits an alert.
2.  Alert Ingestion validates the payload.
3.  Alert is normalized.
4.  Feature extraction prepares ML input.
5.  ML predicts severity and confidence.
6.  Decision Engine enriches context.
7.  Retriever fetches relevant knowledge.
8.  Prompt Builder combines alert + evidence.
9.  LLM generates Investigation Brief.
10. Dashboard displays results.
11. Analyst provides feedback.
12. Feedback is stored for future evaluation.

------------------------------------------------------------------------

# 6. Communication

  Source            Destination       Protocol
  ----------------- ----------------- -------------------
  SIEM              Ingestion         REST/Webhook
  Ingestion         ML                Internal REST
  ML                Decision Engine   JSON
  Decision Engine   Retriever         Internal API
  Retriever         Vector DB         Similarity Search
  Prompt Builder    LLM               HTTP
  Dashboard         Backend           REST
  Backend           Database          SQL

------------------------------------------------------------------------

# 7. Design Decisions

### Human-in-the-loop

The system recommends actions but never performs autonomous remediation.

### Loose Coupling

Each service can be upgraded independently.

### Explainability

Every verdict includes confidence, evidence, retrieved documents, and
MITRE mapping.

### Vendor Agnostic

The canonical alert schema isolates ARIA from SIEM-specific formats.

------------------------------------------------------------------------

# 8. Failure Handling

-   If ML fails, mark alert as "Needs Manual Review".
-   If RAG retrieval fails, return ML verdict with warning.
-   If LLM is unavailable, show retrieved evidence only.
-   Persist all failures in audit logs.

------------------------------------------------------------------------

# 9. Scalability Roadmap

## FYP

-   Single FastAPI backend
-   ChromaDB
-   SQLite/PostgreSQL
-   Streamlit UI

## Production

-   API Gateway
-   RabbitMQ/Kafka
-   Redis cache
-   Kubernetes
-   Horizontal autoscaling
-   Dedicated ML and RAG services

------------------------------------------------------------------------

# 10. Technology Stack

  Layer        Technology
  ------------ --------------
  Backend      FastAPI
  ML           Scikit-learn
  RAG          LangChain
  Vector DB    ChromaDB
  LLM          Ollama
  Database     PostgreSQL
  Dashboard    Streamlit
  Containers   Docker

------------------------------------------------------------------------

# 11. Next Document

03_Component_Architecture.md

This document will decompose every subsystem into internal components,
interfaces, dependencies, lifecycle, and deployment boundaries.

------------------------------------------------------------------------

End of Document
