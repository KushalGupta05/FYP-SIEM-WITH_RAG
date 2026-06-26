# 10 -- Feature Engineering Pipeline

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Document:** Feature Engineering Pipeline\
> **Version:** 1.0\
> **Dataset:** CICIDS2017\
> **Deployment:** 100% Offline

------------------------------------------------------------------------

# 1. Purpose

This document describes the feature engineering pipeline used to
transform raw CICIDS2017 network flow records into machine
learning-ready feature vectors for the ARIA prediction engine.

The pipeline is deterministic, version-controlled, and executed
completely offline.

------------------------------------------------------------------------

# 2. Objectives

-   Produce consistent feature vectors
-   Remove noisy and invalid data
-   Preserve attack-related information
-   Reduce preprocessing time during inference
-   Maintain compatibility between training and production

------------------------------------------------------------------------

# 3. Pipeline Overview

``` mermaid
flowchart LR
A[CICIDS2017 CSV]
-->B[Load Dataset]
-->C[Data Validation]
-->D[Duplicate Removal]
-->E[Handle Missing Values]
-->F[Replace Infinite Values]
-->G[Feature Selection]
-->H[Label Encoding]
-->I[Feature Scaling]
-->J[Train/Test Split]
-->K[Feature Vector]
-->L[Random Forest]
```

------------------------------------------------------------------------

# 4. Dataset Input

-   Format: CSV
-   Source: CICIDS2017
-   Records: Network flow entries
-   Labels: Multi-class attack categories

Input files are stored locally under:

``` text
dataset/
└── CICIDS2017/
```

------------------------------------------------------------------------

# 5. Data Validation

Validation checks:

-   Required columns exist
-   Correct data types
-   No corrupted rows
-   Timestamp integrity
-   Numeric feature validation

Invalid rows are logged and excluded.

------------------------------------------------------------------------

# 6. Duplicate Removal

Purpose:

-   Eliminate repeated network flows
-   Prevent model bias
-   Reduce storage

Method:

``` python
df = df.drop_duplicates()
```

------------------------------------------------------------------------

# 7. Missing Value Handling

Strategy:

  Data Type     Action
  ------------- ---------------------
  Numeric       Median imputation
  Categorical   Most frequent value
  Empty label   Remove row

------------------------------------------------------------------------

# 8. Infinite Value Handling

CICIDS2017 may contain `inf` or `-inf` values due to division-based flow
metrics.

Processing:

1.  Detect infinite values
2.  Replace with NaN
3.  Apply imputation
4.  Verify no invalid values remain

------------------------------------------------------------------------

# 9. Feature Selection

Representative feature groups:

### Flow Features

-   Flow Duration
-   Total Fwd Packets
-   Total Backward Packets

### Packet Statistics

-   Packet Length Mean
-   Packet Length Std
-   Max Packet Length

### TCP Flags

-   SYN Flag Count
-   ACK Flag Count
-   PSH Flag Count
-   FIN Flag Count

### Timing Features

-   Flow IAT Mean
-   Flow IAT Std
-   Active Mean
-   Idle Mean

### Statistical Features

-   Average Packet Size
-   Flow Bytes/s
-   Flow Packets/s

Selected feature names are stored in:

``` text
ml/
└── feature_columns.json
```

------------------------------------------------------------------------

# 10. Label Encoding

Original attack labels remain unchanged internally.

Example mapping:

  Dataset Label     Encoded Value
  --------------- ---------------
  BENIGN                        0
  FTP-Patator                   1
  SSH-Patator                   2
  DoS Hulk                      3
  DDoS                          4
  Bot                           5
  PortScan                      6

Encoder artifact:

``` text
label_encoder.pkl
```

------------------------------------------------------------------------

# 11. Feature Scaling

Random Forest does not strictly require scaling.

Scaling is included as an optional preprocessing step to support future
algorithms.

Supported scalers:

-   StandardScaler
-   MinMaxScaler

Scaler artifact:

``` text
scaler.pkl
```

------------------------------------------------------------------------

# 12. Train/Test Split

Configuration:

-   Training: 80%
-   Testing: 20%
-   Random State: Fixed
-   Stratified Sampling: Enabled

This preserves attack-class distribution.

------------------------------------------------------------------------

# 13. Feature Vector

Output structure:

``` json
{
  "featureVector":[
    152.0,
    23.4,
    0.82,
    ...
  ]
}
```

The feature order must exactly match `feature_columns.json`.

------------------------------------------------------------------------

# 14. Offline Artifacts

``` text
ml/
├── preprocessing.py
├── feature_columns.json
├── scaler.pkl
├── label_encoder.pkl
├── random_forest.joblib
└── metrics.json
```

------------------------------------------------------------------------

# 15. Runtime Pipeline

``` mermaid
sequenceDiagram
participant API
participant PRE as Preprocessor
participant MODEL as Random Forest

API->>PRE: Raw Alert
PRE->>PRE: Validate
PRE->>PRE: Clean
PRE->>PRE: Encode
PRE->>PRE: Scale (Optional)
PRE->>MODEL: Feature Vector
MODEL-->>API: Prediction
```

------------------------------------------------------------------------

# 16. Error Handling

  Condition          Action
  ------------------ ----------------------
  Missing feature    Reject request
  Invalid datatype   Validation error
  Unknown label      Log and skip
  Feature mismatch   Abort inference
  Corrupted scaler   Load backup artifact

------------------------------------------------------------------------

# 17. Performance Goals

  Stage                     Target
  --------------------- ----------
  Validation               \<20 ms
  Cleaning                 \<30 ms
  Feature Engineering      \<50 ms
  Vector Generation        \<20 ms
  Total Preprocessing     \<100 ms

------------------------------------------------------------------------

# 18. Integration with ARIA

``` mermaid
flowchart LR
Alert-->Preprocessing
Preprocessing-->FeatureVector
FeatureVector-->RandomForest
RandomForest-->PolicyEngine
PolicyEngine-->RAG
RAG-->LocalLLM
LocalLLM-->InvestigationBrief
```

------------------------------------------------------------------------

# 19. Future Improvements

-   Automated feature importance ranking
-   Recursive Feature Elimination (RFE)
-   PCA evaluation
-   SHAP feature explanations
-   Feature drift monitoring
-   Automated preprocessing validation

------------------------------------------------------------------------

# Next Document

**11_RAG_Architecture.md** --- Document ingestion, embedding generation,
ChromaDB design, retrieval workflow, prompt engineering, and Ollama
integration.
