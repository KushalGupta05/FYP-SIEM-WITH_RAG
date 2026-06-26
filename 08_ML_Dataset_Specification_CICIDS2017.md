# 08 -- Machine Learning Dataset Specification

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Version:** 1.0\
> **Dataset:** CICIDS2017\
> **Deployment:** Fully Offline

------------------------------------------------------------------------

# 1. Dataset Selection

The machine learning model used in ARIA is trained using the
**CICIDS2017 (Canadian Institute for Cybersecurity Intrusion Detection
System 2017)** dataset.

The dataset was selected because:

-   It is publicly available.
-   It contains modern network attacks.
-   It provides labeled network flow data.
-   It is widely used in intrusion detection research.
-   It is suitable for supervised machine learning.

------------------------------------------------------------------------

# 2. Dataset Characteristics

  Property   Value
  ---------- -------------------------------------
  Dataset    CICIDS2017
  Type       Network Intrusion Detection Dataset
  Format     CSV
  Labels     Multi-class
  Features   80+ Network Flow Features
  Target     Traffic Classification

------------------------------------------------------------------------

# 3. Attack Classes

The dataset contains benign traffic and multiple attack categories
including:

-   BENIGN
-   FTP-Patator
-   SSH-Patator
-   DoS Hulk
-   DoS GoldenEye
-   DoS Slowloris
-   DoS SlowHTTPTest
-   DDoS
-   Heartbleed
-   Bot
-   PortScan
-   Infiltration
-   Web Attack -- Brute Force
-   Web Attack -- SQL Injection
-   Web Attack -- XSS

------------------------------------------------------------------------

# 4. Data Preprocessing Pipeline

``` mermaid
flowchart LR
A[Load CSV]
-->B[Remove Duplicates]
-->C[Handle Missing Values]
-->D[Replace Infinite Values]
-->E[Encode Labels]
-->F[Feature Selection]
-->G[Train/Test Split]
-->H[Feature Scaling]
-->I[Model Training]
```

------------------------------------------------------------------------

# 5. Training Pipeline

``` mermaid
flowchart TD
A[CICIDS2017 CSV]
-->B[Preprocessing]
-->C[Random Forest Training]
-->D[Model Evaluation]
-->E[Export Model]
-->F[random_forest.joblib]
-->G[FastAPI Inference]
```

------------------------------------------------------------------------

# 6. Model Selection

Primary Model:

-   Random Forest Classifier

Future Models:

-   XGBoost
-   LightGBM
-   Extra Trees
-   Gradient Boosting

------------------------------------------------------------------------

# 7. Model Output

The classifier predicts the original attack class from CICIDS2017.

Example:

``` json
{
  "prediction":"FTP-Patator",
  "confidence":0.96
}
```

A policy engine then converts the prediction into SOC severity.

  Prediction      Severity
  --------------- ----------
  BENIGN          Low
  FTP-Patator     High
  SSH-Patator     High
  DoS Hulk        Critical
  DDoS            Critical
  Heartbleed      Critical
  Bot             High
  SQL Injection   Critical

------------------------------------------------------------------------

# 8. Model Artifacts

``` text
models/
├── random_forest.joblib
├── scaler.pkl
├── label_encoder.pkl
├── feature_columns.json
├── metrics.json
└── metadata.json
```

------------------------------------------------------------------------

# 9. Offline Deployment

Training is performed locally.

Inference is performed using the serialized model through FastAPI.

No cloud services, online APIs, or paid subscriptions are required.

------------------------------------------------------------------------

# 10. Hardware Target

Optimized for:

-   CPU: AMD Ryzen 7 7840HS
-   RAM: 16 GB DDR5
-   GPU: NVIDIA RTX 3050 Laptop (6 GB VRAM)

The classical ML model runs primarily on the CPU, while the GPU is
reserved for the local LLM (Ollama).

------------------------------------------------------------------------

# 11. Integration with ARIA

``` mermaid
flowchart LR
Alert-->FeatureExtraction
FeatureExtraction-->RandomForest
RandomForest-->AttackPrediction
AttackPrediction-->PolicyEngine
PolicyEngine-->RAG
RAG-->LocalLLM
LocalLLM-->InvestigationBrief
```

------------------------------------------------------------------------

# 12. Future Improvements

-   Hyperparameter tuning
-   Feature importance analysis
-   Cross-validation
-   Model versioning
-   Incremental retraining
-   Support for newer IDS datasets
