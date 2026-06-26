# 09 -- Machine Learning Architecture

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Version:** 1.0\
> **Architecture:** Offline-First\
> **Dataset:** CICIDS2017

------------------------------------------------------------------------

# 1. Purpose

This document describes the complete Machine Learning architecture used
by ARIA, including training, preprocessing, model management, inference,
evaluation, deployment, and integration with the RAG pipeline.

------------------------------------------------------------------------

# 2. Design Goals

-   100% offline execution
-   Deterministic predictions
-   Explainable inference
-   Fast CPU inference
-   Easy model replacement
-   Version-controlled models

------------------------------------------------------------------------

# 3. High-Level Architecture

``` mermaid
flowchart TD
A[CICIDS2017 Dataset]
-->B[Data Cleaning]
-->C[Feature Engineering]
-->D[Feature Selection]
-->E[Train/Test Split]
-->F[Random Forest Training]
-->G[Model Evaluation]
-->H[Model Registry]
-->I[random_forest.joblib]
-->J[FastAPI ML Service]
-->K[Decision Engine]
-->L[RAG Pipeline]
```

------------------------------------------------------------------------

# 4. System Components

  Component          Responsibility
  ------------------ ---------------------------------
  Dataset Loader     Load CICIDS2017 CSV files
  Data Cleaner       Remove invalid rows, duplicates
  Feature Engineer   Generate ML features
  Label Encoder      Encode attack classes
  Model Trainer      Train Random Forest
  Model Registry     Store model versions
  Inference Engine   Predict attack class
  Policy Engine      Map attack → SOC severity

------------------------------------------------------------------------

# 5. Dataset

Dataset: **CICIDS2017**

Characteristics:

-   CSV format
-   Multi-class labels
-   80+ network flow features
-   Benign and malicious traffic
-   Suitable for supervised learning

Attack classes include:

-   BENIGN
-   FTP-Patator
-   SSH-Patator
-   DoS Hulk
-   DDoS
-   Bot
-   PortScan
-   Heartbleed
-   Infiltration
-   Web Attack (SQLi, XSS, Brute Force)

------------------------------------------------------------------------

# 6. Preprocessing Pipeline

``` mermaid
flowchart LR
Load-->Duplicates
Duplicates-->Missing
Missing-->Infinite
Infinite-->Encoding
Encoding-->Scaling
Scaling-->FeatureSelection
FeatureSelection-->Training
```

Steps:

1.  Load CSV files
2.  Remove duplicate records
3.  Handle missing values
4.  Replace infinite values
5.  Encode labels
6.  Scale numerical features (if required)
7.  Select features
8.  Split into training and testing sets

------------------------------------------------------------------------

# 7. Feature Engineering

Feature groups:

-   Flow features
-   Packet statistics
-   TCP flag counts
-   Timing metrics
-   Statistical aggregates

Feature artifacts:

``` text
ml/
├── feature_columns.json
├── scaler.pkl
├── label_encoder.pkl
└── preprocessing.py
```

------------------------------------------------------------------------

# 8. Model Training

Primary Algorithm:

-   Random Forest Classifier

Training Flow:

``` mermaid
flowchart TD
Dataset-->TrainSplit
TrainSplit-->RandomForest
RandomForest-->Validation
Validation-->Metrics
Metrics-->ExportModel
```

Saved artifacts:

-   random_forest.joblib
-   metrics.json
-   metadata.json

------------------------------------------------------------------------

# 9. Model Evaluation

Metrics recorded:

-   Accuracy
-   Precision
-   Recall
-   F1 Score
-   Confusion Matrix
-   ROC-AUC (where applicable)

Evaluation reports are stored with the model version.

------------------------------------------------------------------------

# 10. Model Registry

``` text
models/
├── random_forest_v1.joblib
├── random_forest_v2.joblib
├── metadata.json
├── metrics.json
├── scaler.pkl
└── feature_columns.json
```

Each model stores:

-   version
-   training date
-   dataset version
-   hyperparameters
-   evaluation metrics

------------------------------------------------------------------------

# 11. Inference Architecture

``` mermaid
sequenceDiagram
participant API
participant PRE as Preprocessor
participant ML as Random Forest
participant POL as Policy Engine
participant RAG

API->>PRE: Alert
PRE->>ML: Feature Vector
ML-->>POL: Attack Class
POL-->>RAG: Severity + Attack Type
```

Inference outputs:

``` json
{
  "prediction":"DoS Hulk",
  "confidence":0.97,
  "severity":"Critical",
  "modelVersion":"rf_v1"
}
```

------------------------------------------------------------------------

# 12. Policy Engine

Maps attack labels to SOC priorities.

  Attack        Severity
  ------------- ----------
  BENIGN        Low
  FTP-Patator   High
  SSH-Patator   High
  Bot           High
  PortScan      Medium
  DoS Hulk      Critical
  DDoS          Critical
  Heartbleed    Critical

The original attack class is preserved and passed to the RAG pipeline.

------------------------------------------------------------------------

# 13. Integration with RAG

``` mermaid
flowchart LR
Prediction-->Policy
Policy-->Retriever
Retriever-->ChromaDB
ChromaDB-->Ollama
Ollama-->Investigation
```

The ML model identifies **what** happened. The RAG pipeline explains
**why it happened** and **how to respond**.

------------------------------------------------------------------------

# 14. Offline Deployment

Hardware Target:

-   AMD Ryzen 7 7840HS
-   16 GB DDR5 RAM
-   NVIDIA RTX 3050 Laptop (6 GB VRAM)

Runtime Allocation:

-   CPU: Data preprocessing + Random Forest inference
-   GPU: Ollama Local LLM
-   PostgreSQL: Transactional storage
-   ChromaDB: Vector search

No cloud services or subscriptions are required.

------------------------------------------------------------------------

# 15. Failure Handling

  Failure                     Action
  --------------------------- ------------------------------
  Model missing               Return service unavailable
  Invalid feature vector      Reject request
  Prediction confidence low   Flag for manual review
  Registry mismatch           Load previous stable version

------------------------------------------------------------------------

# 16. Future Enhancements

-   Hyperparameter optimization
-   Cross-validation automation
-   Feature importance dashboard
-   Model drift detection
-   Incremental retraining
-   Additional datasets (CSE-CIC-IDS2018, CIC-DDoS2019)

------------------------------------------------------------------------

# 17. Next Document

**10_RAG_Architecture.md**

Defines document ingestion, embedding generation, vector database
design, retrieval pipeline, prompt engineering, and local LLM
orchestration.
