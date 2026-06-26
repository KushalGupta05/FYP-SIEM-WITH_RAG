# 07 -- API Specifications (Offline-First Engineering Specification)

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Version:** 1.0\
> **Architecture:** 100% Offline • No Cloud • No Paid APIs • Local LLM
> via Ollama

------------------------------------------------------------------------

# 1. Purpose

This document defines the REST API exposed by ARIA. All APIs are served
locally by FastAPI and communicate only with on-device services (ML,
ChromaDB, PostgreSQL, Ollama).

Base URL:

    http://localhost:8000/api/v1

------------------------------------------------------------------------

# 2. Architecture

``` mermaid
flowchart LR
Client[Streamlit UI]
-->API[FastAPI]

API-->ML[ML Service]

API-->RAG[RAG Service]

RAG-->CH[(ChromaDB)]

API-->DB[(PostgreSQL)]

RAG-->OLL[Ollama]

OLL-->LLM[Local LLM]
```

No endpoint communicates with external cloud providers.

------------------------------------------------------------------------

# 3. API Design Principles

-   RESTful resources
-   JSON request/response
-   Versioned endpoints (`/api/v1`)
-   Idempotent GET operations
-   Consistent error format
-   OpenAPI 3.1 compatible
-   Offline operation only

------------------------------------------------------------------------

# 4. Authentication

For the FYP:

-   Local username/password
-   JWT access tokens
-   Refresh tokens
-   Role Based Access Control (RBAC)

Roles:

-   Analyst
-   Senior Analyst
-   SOC Manager
-   Administrator

------------------------------------------------------------------------

# 5. Standard Response Format

Success

``` json
{
  "success": true,
  "data": {},
  "requestId": "uuid",
  "timestamp": "2026-06-26T12:00:00Z"
}
```

Error

``` json
{
  "success": false,
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Severity out of range"
  },
  "requestId": "uuid"
}
```

------------------------------------------------------------------------

# 6. Endpoint Catalogue

  Method   Endpoint       Purpose
  -------- -------------- -------------------------
  POST     /auth/login    Authenticate user
  POST     /alerts        Ingest SIEM alert
  GET      /alerts/{id}   Fetch alert
  POST     /predict       Run ML inference
  POST     /retrieve      Execute RAG retrieval
  POST     /investigate   Generate investigation
  POST     /feedback      Submit analyst feedback
  GET      /models        List local models
  POST     /models/load   Load Ollama model
  GET      /system        Hardware status
  GET      /health        Service health

------------------------------------------------------------------------

# 7. Endpoint Details

## POST /alerts

Creates a normalized alert.

Request

``` json
{
  "source":"Wazuh",
  "severity":8,
  "eventType":"AUTH_BRUTE_FORCE",
  "hostname":"server01",
  "rawPayload":{}
}
```

Response

``` json
{
  "alertId":"uuid",
  "status":"RECEIVED"
}
```

Validation

-   Severity 0--10
-   Valid hostname
-   Non-empty payload

------------------------------------------------------------------------

## POST /predict

Runs the local Scikit-learn model.

Input

``` json
{
  "alertId":"uuid"
}
```

Output

``` json
{
  "prediction":"Malicious",
  "confidence":0.94,
  "modelVersion":"rf_v3"
}
```

------------------------------------------------------------------------

## POST /retrieve

Runs semantic retrieval using ChromaDB.

Returns:

-   topK documents
-   similarity score
-   metadata
-   citations

------------------------------------------------------------------------

## POST /investigate

Pipeline:

Alert → ML → RAG → Ollama → Investigation

Response

``` json
{
  "investigationId":"uuid",
  "summary":"...",
  "mitre":["T1110"],
  "recommendations":[]
}
```

------------------------------------------------------------------------

## POST /feedback

Stores analyst review.

``` json
{
  "investigationId":"uuid",
  "verdict":"True Positive",
  "notes":"Confirmed brute force",
  "durationSeconds":85
}
```

------------------------------------------------------------------------

## GET /models

Returns locally installed Ollama models.

Example

``` json
{
  "models":[
    "mistral:7b-instruct-q4",
    "qwen2.5:3b",
    "phi3:mini"
  ]
}
```

------------------------------------------------------------------------

## POST /models/load

Loads a model into memory.

``` json
{
  "model":"mistral:7b-instruct-q4"
}
```

The orchestrator checks RAM and GPU availability before loading.

------------------------------------------------------------------------

## GET /system

Returns local hardware information.

``` json
{
  "cpu":"AMD Ryzen 7 7840HS",
  "ramGB":16,
  "gpu":"RTX 3050 Laptop 6GB",
  "diskFreeGB":250,
  "llmRuntime":"Ollama"
}
```

Used by the dashboard to recommend appropriate models.

------------------------------------------------------------------------

## GET /health

Checks:

-   PostgreSQL
-   ChromaDB
-   Ollama
-   ML Model
-   Disk Space

------------------------------------------------------------------------

# 8. Error Codes

  Code               Meaning
  ------------------ ------------------------
  VALIDATION_ERROR   Invalid request
  MODEL_NOT_LOADED   No LLM loaded
  GPU_UNAVAILABLE    GPU not detected
  VECTOR_DB_ERROR    ChromaDB failure
  DATABASE_ERROR     PostgreSQL unavailable
  LLM_TIMEOUT        Ollama timed out

------------------------------------------------------------------------

# 9. LLM Orchestrator API Flow

``` mermaid
sequenceDiagram
participant API
participant ORCH as LLM Orchestrator
participant O as Ollama
participant L as Local Model

API->>ORCH: Investigation Request
ORCH->>ORCH: Check RAM
ORCH->>ORCH: Check GPU
ORCH->>O: Generate
O->>L: Inference
L-->>O: Response
O-->>ORCH: JSON
ORCH-->>API: Validated Result
```

------------------------------------------------------------------------

# 10. Recommended Local Models

  Model           Purpose
  --------------- --------------------------
  Phi-3 Mini      Fast inference
  Qwen2.5 3B      Lightweight reasoning
  Mistral 7B Q4   Default production model

------------------------------------------------------------------------

# 11. OpenAPI & Versioning

-   Semantic versioning
-   `/api/v1`
-   Backward compatible changes only within major version
-   Swagger UI generated automatically by FastAPI

------------------------------------------------------------------------

# 12. Future APIs

-   /knowledge/upload
-   /knowledge/reindex
-   /cases
-   /playbooks
-   /metrics
-   /benchmark
