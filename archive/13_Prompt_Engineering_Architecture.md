# 13 -- Prompt Engineering Architecture

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Version:** 1.0\
> **Architecture:** Offline-First\
> **LLM Runtime:** Ollama (Local Models)

------------------------------------------------------------------------

# 1. Purpose

This document defines the prompt engineering architecture used by ARIA
to generate structured, evidence-based investigation reports using a
local Large Language Model (LLM).

The prompt engineering layer bridges the Machine Learning prediction,
the retrieved knowledge base context, and the analyst-facing
investigation report.

------------------------------------------------------------------------

# 2. Design Goals

-   100% offline execution
-   Deterministic prompt structure
-   Minimize hallucinations
-   Evidence-backed responses
-   Structured JSON output
-   Explainable recommendations
-   Reusable prompt templates

------------------------------------------------------------------------

# 3. Prompt Architecture

``` mermaid
flowchart LR
Alert-->MLPrediction
MLPrediction-->ContextRetriever
ContextRetriever-->PromptBuilder
PromptBuilder-->PromptValidator
PromptValidator-->Ollama
Ollama-->ResponseValidator
ResponseValidator-->InvestigationReport
```

------------------------------------------------------------------------

# 4. Prompt Components

  Component        Purpose
  ---------------- ------------------------------------------
  System Prompt    Defines ARIA's role and response rules
  User Prompt      Contains alert and investigation request
  Context Block    Retrieved RAG documents
  Evidence Block   ML prediction and confidence
  Output Schema    Required JSON structure

------------------------------------------------------------------------

# 5. Prompt Template

``` text
SYSTEM:
You are ARIA, an AI SOC investigation assistant.
Use only the supplied context.
If evidence is insufficient, state that explicitly.

USER:
Alert Details:
{alert}

ML Prediction:
{prediction}

Confidence:
{confidence}

Retrieved Knowledge:
{context}

Generate:
1. Executive Summary
2. Threat Analysis
3. MITRE ATT&CK Mapping
4. Evidence
5. Recommended Actions
6. Analyst Checklist
```

------------------------------------------------------------------------

# 6. Prompt Construction Pipeline

``` mermaid
flowchart TD
Alert-->Normalize
Normalize-->ML
ML-->Retriever
Retriever-->ContextAssembly
ContextAssembly-->TemplateSelection
TemplateSelection-->PromptGeneration
PromptGeneration-->Ollama
```

------------------------------------------------------------------------

# 7. Context Assembly

The Prompt Builder combines:

-   Alert metadata
-   ML prediction
-   Confidence score
-   Retrieved MITRE techniques
-   CVE references
-   CERT-In advisories
-   Organization SOPs
-   Playbooks

Maximum context is trimmed to fit the selected model's context window.

------------------------------------------------------------------------

# 8. Prompt Templates

## Investigation Template

Used for incident investigation.

## Summary Template

Used for benign or informational alerts.

## Recommendation Template

Generates mitigation and containment steps.

## Explanation Template

Explains why the model reached its conclusion.

------------------------------------------------------------------------

# 9. Output Schema

``` json
{
  "executive_summary":"",
  "attack_type":"",
  "severity":"",
  "confidence":0.95,
  "mitre_mapping":[],
  "evidence":[],
  "recommendations":[],
  "analyst_checklist":[]
}
```

The LLM is instructed to return valid JSON only.

------------------------------------------------------------------------

# 10. Response Validation

Validation checks:

-   Valid JSON
-   Required fields present
-   Confidence value exists
-   MITRE references included
-   Recommendations formatted
-   No empty sections

Invalid responses trigger one retry before fallback.

------------------------------------------------------------------------

# 11. Hallucination Mitigation

Strategies:

-   Use retrieved context only
-   Low temperature inference
-   Fixed output schema
-   Evidence-first prompting
-   Citation requirements
-   Response validation

------------------------------------------------------------------------

# 12. Model Configuration

Recommended local models:

  Model           Use
  --------------- -----------------------
  Mistral 7B Q4   Default investigation
  Qwen2.5 3B      Lightweight systems
  Phi-3 Mini      Fast responses

Temperature: 0.2

Top-p: 0.9

Max Tokens: 1024

------------------------------------------------------------------------

# 13. Prompt Versioning

``` text
prompts/
├── investigation_v1.md
├── summary_v1.md
├── recommendation_v1.md
├── explanation_v1.md
└── metadata.json
```

Each prompt records:

-   version
-   author
-   creation date
-   supported model
-   checksum

------------------------------------------------------------------------

# 14. Error Handling

  Failure             Action
  ------------------- ---------------------------
  Missing context     Continue with ML evidence
  Invalid prompt      Reject generation
  Ollama timeout      Retry once
  Invalid JSON        Re-prompt validator
  Model unavailable   Return ML-only report

------------------------------------------------------------------------

# 15. Security

-   No secrets embedded in prompts
-   Prompt hash stored in audit logs
-   Sensitive fields masked before generation
-   Offline execution only

------------------------------------------------------------------------

# 16. Performance Targets

  Stage                      Target
  --------------------- -----------
  Prompt Assembly           \<30 ms
  Validation                \<20 ms
  Ollama Inference        \<3000 ms
  Response Validation       \<50 ms

------------------------------------------------------------------------

# 17. Hardware Optimization

Target Hardware:

-   AMD Ryzen 7 7840HS
-   16 GB RAM
-   RTX 3050 Laptop GPU (6 GB VRAM)

The GPU is dedicated to Ollama inference while the CPU handles prompt
assembly and validation.

------------------------------------------------------------------------

# 18. Future Enhancements

-   Dynamic prompt selection
-   Multi-model orchestration
-   Automatic prompt evaluation
-   Prompt A/B testing
-   Self-refinement prompts
-   Context compression

------------------------------------------------------------------------

# 19. End-to-End Flow

``` mermaid
flowchart TD
Alert-->MLPrediction
MLPrediction-->KnowledgeRetrieval
KnowledgeRetrieval-->PromptBuilder
PromptBuilder-->Ollama
Ollama-->ResponseValidator
ResponseValidator-->Dashboard
```

------------------------------------------------------------------------

# Next Document

**14_Deployment_Architecture.md** -- Docker Compose topology, local
networking, service orchestration, startup sequence, and offline
deployment guide.
