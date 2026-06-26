# 14 -- Feedback Learning Architecture

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Version:** 1.0\
> **Architecture:** Offline-First\
> **Deployment:** Local Only (No Cloud Services)

------------------------------------------------------------------------

# 1. Purpose

The Feedback Learning Architecture enables ARIA to continuously improve
its decision-making by capturing analyst feedback after each
investigation.

This subsystem **does not automatically retrain models**. Instead, it
collects, validates, stores, and evaluates analyst feedback for future
supervised retraining.

The entire workflow operates offline.

------------------------------------------------------------------------

# 2. Objectives

-   Capture analyst decisions
-   Measure model performance
-   Identify false positives and false negatives
-   Build a verified feedback dataset
-   Support future offline retraining
-   Maintain complete auditability

------------------------------------------------------------------------

# 3. High-Level Architecture

``` mermaid
flowchart LR
InvestigationReport
-->Dashboard

Dashboard
-->FeedbackForm

FeedbackForm
-->FeedbackValidator

FeedbackValidator
-->FeedbackDatabase

FeedbackDatabase
-->EvaluationEngine

EvaluationEngine
-->DatasetBuilder

DatasetBuilder
-->FutureRetraining
```

------------------------------------------------------------------------

# 4. Feedback Lifecycle

``` mermaid
stateDiagram-v2

[*] --> InvestigationGenerated
InvestigationGenerated --> AnalystReview
AnalystReview --> FeedbackSubmitted
FeedbackSubmitted --> Validation
Validation --> Stored
Stored --> Evaluation
Evaluation --> DatasetUpdated
DatasetUpdated --> [*]
```

------------------------------------------------------------------------

# 5. Feedback Components

  Component           Responsibility
  ------------------- -------------------------------
  Dashboard           Display investigation results
  Feedback Form       Capture analyst review
  Validator           Validate submitted feedback
  Feedback Store      Persist feedback records
  Evaluation Engine   Analyze model performance
  Dataset Builder     Prepare retraining dataset

------------------------------------------------------------------------

# 6. Feedback Data Model

Each investigation stores:

``` json
{
  "investigationId":"uuid",
  "prediction":"DoS Hulk",
  "confidence":0.96,
  "analystVerdict":"True Positive",
  "falsePositive":false,
  "notes":"Confirmed brute-force activity",
  "reviewDuration":92,
  "timestamp":"ISO8601"
}
```

------------------------------------------------------------------------

# 7. Analyst Feedback Types

Supported outcomes:

-   True Positive
-   False Positive
-   True Negative
-   False Negative

Additional actions:

-   Escalated
-   Closed
-   Requires Investigation

------------------------------------------------------------------------

# 8. Feedback Validation

Validation rules:

-   Authenticated analyst only
-   Investigation must exist
-   Verdict required
-   Timestamp generated automatically
-   Notes optional

Invalid feedback is rejected and logged.

------------------------------------------------------------------------

# 9. Storage Architecture

``` mermaid
flowchart TD
Feedback
-->PostgreSQL

PostgreSQL
-->EvaluationEngine

EvaluationEngine
-->Reports

EvaluationEngine
-->RetrainingDataset
```

Feedback is never stored in ChromaDB.

------------------------------------------------------------------------

# 10. Evaluation Metrics

The evaluation engine calculates:

-   Accuracy
-   Precision
-   Recall
-   F1 Score
-   False Positive Rate
-   False Negative Rate
-   Average Review Time
-   Analyst Agreement Rate

Reports are generated locally.

------------------------------------------------------------------------

# 11. Dataset Builder

The Dataset Builder creates verified training samples.

Workflow:

``` mermaid
flowchart LR
Feedback
-->Validation
-->ApprovedRecords
-->TrainingDataset
```

Only analyst-approved records are exported.

------------------------------------------------------------------------

# 12. Model Improvement Workflow

``` mermaid
flowchart TD
Feedback
-->Evaluation

Evaluation
-->ApprovedDataset

ApprovedDataset
-->OfflineTraining

OfflineTraining
-->RandomForest

RandomForest
-->ModelRegistry
```

Retraining is a manual operation initiated by an administrator.

------------------------------------------------------------------------

# 13. Audit Logging

Every feedback action generates an immutable audit entry.

Tracked events:

-   Investigation viewed
-   Feedback submitted
-   Feedback updated
-   Evaluation executed
-   Dataset exported

------------------------------------------------------------------------

# 14. Security

-   Local authentication
-   RBAC enforced
-   Immutable audit logs
-   Offline storage
-   Encrypted PostgreSQL database
-   No internet connectivity required

------------------------------------------------------------------------

# 15. Performance Targets

  Operation                 Target
  --------------------- ----------
  Feedback Submission     \<100 ms
  Validation               \<30 ms
  Database Write           \<50 ms
  Evaluation Report       \<500 ms

------------------------------------------------------------------------

# 16. Hardware Optimization

Target System:

-   AMD Ryzen 7 7840HS
-   16 GB RAM
-   RTX 3050 Laptop GPU (6 GB VRAM)

CPU performs evaluation and report generation.

GPU remains dedicated to Ollama inference.

------------------------------------------------------------------------

# 17. Failure Handling

  Failure                Recovery
  ---------------------- -----------------------
  Invalid feedback       Reject submission
  Database unavailable   Retry and log
  Duplicate submission   Ignore duplicate
  Evaluation failure     Preserve raw feedback

------------------------------------------------------------------------

# 18. Future Enhancements

-   Confidence calibration analysis
-   Analyst performance dashboard
-   Model drift detection
-   Active learning support
-   Semi-supervised retraining
-   Automated quality reports

------------------------------------------------------------------------

# 19. End-to-End Feedback Flow

``` mermaid
flowchart TD
Investigation
-->Analyst

Analyst
-->Feedback

Feedback
-->Validation

Validation
-->Storage

Storage
-->Evaluation

Evaluation
-->Dataset

Dataset
-->OfflineRetraining
```

------------------------------------------------------------------------

# 20. Integration with ARIA

The feedback subsystem closes the loop between prediction and continuous
improvement.

ML Prediction → Investigation Report → Analyst Review → Feedback
Collection → Evaluation → Verified Dataset → Future Offline Retraining

------------------------------------------------------------------------

# Next Document

**15_Deployment_Architecture.md** -- Docker Compose topology, local
networking, service orchestration, startup sequence, resource
allocation, and offline deployment guide.
