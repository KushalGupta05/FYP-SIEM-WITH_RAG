# 06 -- Database Design (Offline-First Engineering Specification)

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Version:** 1.0\
> **Architecture Constraint:** **100% Offline Deployment -- No Paid
> Cloud Services or API Subscriptions**

------------------------------------------------------------------------

# 1. Design Goals

This database architecture is designed specifically for an offline
deployment.

## Constraints

-   No cloud database
-   No managed vector database
-   No paid APIs
-   No SaaS authentication
-   No internet dependency during runtime

Everything runs locally using Docker Compose.

------------------------------------------------------------------------

# 2. Database Stack

  Purpose           Technology           Reason
  ----------------- -------------------- ---------------------------
  Relational Data   PostgreSQL           Mature, ACID, open-source
  Vector Search     ChromaDB             Offline embeddings
  Development       SQLite               Lightweight testing only
  Model Files       Local Filesystem     Version-controlled
  Knowledge Base    Local Markdown/PDF   Indexed into ChromaDB

------------------------------------------------------------------------

# 3. Storage Architecture

``` mermaid
flowchart TD
A[FastAPI]
-->B[(PostgreSQL)]
A-->C[(ChromaDB)]
A-->D[/Models/]
A-->E[/Knowledge Base/]
E-->C
```

------------------------------------------------------------------------

# 4. Logical Databases

## PostgreSQL

Stores transactional data.

Tables:

-   users
-   alerts
-   predictions
-   investigations
-   evidence
-   feedback
-   audit_logs
-   playbooks
-   sessions

## ChromaDB

Collections:

-   mitre_embeddings
-   cve_embeddings
-   certin_embeddings
-   playbooks
-   organization_docs

------------------------------------------------------------------------

# 5. Entity Relationship Diagram

``` mermaid
erDiagram

USERS ||--o{ FEEDBACK : submits
ALERTS ||--|| PREDICTIONS : has
ALERTS ||--|| INVESTIGATIONS : creates
INVESTIGATIONS ||--o{ EVIDENCE : contains
INVESTIGATIONS ||--o{ FEEDBACK : receives
INVESTIGATIONS ||--o{ AUDIT_LOGS : generates
PLAYBOOKS ||--o{ INVESTIGATIONS : referenced_by
```

------------------------------------------------------------------------

# 6. Table Specifications

## alerts

Primary Key: alert_id (UUID)

Core fields:

-   source
-   severity
-   hostname
-   username
-   event_type
-   timestamp
-   raw_payload
-   status

Indexes:

-   timestamp
-   severity
-   source
-   event_type

Retention: configurable.

------------------------------------------------------------------------

## predictions

Stores immutable ML output.

Fields

-   prediction_id
-   alert_id (FK)
-   model_version
-   predicted_class
-   confidence
-   probabilities_json
-   inference_time_ms

Rule:

One alert → One prediction.

------------------------------------------------------------------------

## investigations

Stores generated investigation package.

Contains

-   summary
-   mitre_mapping
-   recommendations
-   prompt_hash
-   llm_version
-   generated_at

------------------------------------------------------------------------

## evidence

Each row references a single investigation.

Evidence Types

-   IOC
-   CVE
-   MITRE
-   Threat Report
-   Log Snippet

------------------------------------------------------------------------

## feedback

Stores analyst review.

Columns

-   verdict
-   false_positive
-   escalation
-   notes
-   duration_seconds

Feedback never overwrites predictions.

------------------------------------------------------------------------

## audit_logs

Immutable.

Tracks:

-   login
-   alert viewed
-   investigation generated
-   feedback submitted
-   prompt executed

------------------------------------------------------------------------

# 7. ChromaDB Collections

## mitre_embeddings

Documents

-   ATT&CK techniques
-   mitigations
-   detections

Metadata

-   tactic
-   technique
-   version

## certin_embeddings

Contains

-   CERT-In advisories
-   Indian banking alerts
-   RBI notifications

## playbooks

Internal investigation procedures.

------------------------------------------------------------------------

# 8. Index Strategy

  Table            Index
  ---------------- ---------------------
  alerts           timestamp, severity
  predictions      alert_id
  investigations   generated_at
  feedback         analyst_id
  audit_logs       timestamp

------------------------------------------------------------------------

# 9. Backup Strategy

Offline backups only.

-   PostgreSQL nightly dump
-   ChromaDB directory copy
-   Models versioned locally
-   Knowledge base under Git

------------------------------------------------------------------------

# 10. Security

-   PostgreSQL password authentication
-   Local Docker network isolation
-   Least-privilege database roles
-   Encrypted disks recommended
-   Audit trail enabled

------------------------------------------------------------------------

# 11. Offline Folder Layout

``` text
aria/
├── backend/
├── frontend/
├── database/
│   ├── postgres/
│   └── chromadb/
├── models/
├── knowledge_base/
│   ├── MITRE/
│   ├── CVE/
│   ├── CERT-In/
│   └── Playbooks/
├── docker-compose.yml
└── backups/
```

------------------------------------------------------------------------

# 12. Future Migration

The schema intentionally avoids vendor lock-in.

Possible future replacements:

-   ChromaDB → Weaviate
-   PostgreSQL → Managed PostgreSQL
-   Local storage → S3-compatible object store

No application-layer redesign should be required.

------------------------------------------------------------------------

# Design Decisions

1.  PostgreSQL chosen over MongoDB because the investigation workflow is
    highly relational.
2.  ChromaDB selected because it operates fully offline and integrates
    well with LangChain.
3.  SQLite remains a developer convenience only and is not the primary
    runtime database.
4.  All knowledge documents are stored locally and indexed; no online
    retrieval is required.

------------------------------------------------------------------------

# Next Document

**07_API_Specification.md** --- REST endpoints, request/response
schemas, authentication, versioning, and error contracts.
