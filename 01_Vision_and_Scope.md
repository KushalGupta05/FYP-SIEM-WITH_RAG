# 01 -- Vision & Scope

> **Project:** ARIA --- Autonomous SIEM Triage Assistant\
> **Document Version:** 1.0\
> **Status:** Draft

------------------------------------------------------------------------

# 1. Purpose

This document defines the product vision, project scope, stakeholders,
objectives, assumptions, constraints, functional requirements,
non-functional requirements, and success criteria for the ARIA platform.

It serves as the foundation for all subsequent architecture documents.

------------------------------------------------------------------------

# 2. Vision Statement

Build an AI-assisted investigation platform that augments Tier-1 SOC
analysts by automating security alert context assembly, investigation
preparation, MITRE ATT&CK mapping, and threat intelligence retrieval
while keeping the analyst in control of all security decisions.

------------------------------------------------------------------------

# 3. Mission

Reduce the average time required to triage a SIEM alert from several
minutes to a few seconds by combining deterministic Machine Learning
with Retrieval-Augmented Generation (RAG).

------------------------------------------------------------------------

# 4. Problem Statement

Modern Security Operations Centers generate thousands of alerts every
day. While SIEMs detect suspicious events effectively, analysts still
spend significant time collecting context from multiple tools before
making an escalation decision.

ARIA addresses this operational bottleneck by producing a structured
Investigation Briefing that includes:

-   ML classification
-   Confidence score
-   Threat intelligence
-   MITRE ATT&CK mapping
-   Evidence summary
-   Recommended investigation checklist

------------------------------------------------------------------------

# 5. Target Users

## Primary User

**Tier-1 SOC Analyst**

Responsibilities:

-   Monitor incoming alerts
-   Perform initial investigation
-   Escalate confirmed incidents
-   Close false positives

Pain Points:

-   Alert fatigue
-   Context switching
-   Manual documentation
-   High false-positive volume

## Secondary Users

-   SOC Manager
-   Incident Response Team
-   Security Engineers
-   Threat Hunters

------------------------------------------------------------------------

# 6. Project Scope

## Included

-   SIEM alert ingestion
-   Alert normalization
-   ML classification
-   RAG-based retrieval
-   Investigation briefing generation
-   Analyst dashboard
-   Feedback collection
-   Knowledge-base updates

## Excluded

-   Autonomous remediation
-   Endpoint response
-   Live malware analysis
-   Network intrusion prevention
-   SOAR orchestration (future)

------------------------------------------------------------------------

# 7. Functional Requirements

  ID      Requirement
  ------- -----------------------------------------------------
  FR-01   Ingest alerts from supported SIEM platforms
  FR-02   Normalize alerts into a common schema
  FR-03   Classify alerts as Benign, Suspicious, or Malicious
  FR-04   Retrieve relevant threat intelligence
  FR-05   Generate an investigation briefing
  FR-06   Map evidence to MITRE ATT&CK
  FR-07   Capture analyst feedback
  FR-08   Persist investigation history
  FR-09   Provide REST APIs for integration

------------------------------------------------------------------------

# 8. Non-Functional Requirements

  Category         Target
  ---------------- ----------------------------------------------
  Availability     99.5% (future)
  Response Time    \<5 seconds for full investigation
  Scalability      Modular architecture
  Security         RBAC, audit logging, encrypted communication
  Reliability      Graceful degradation if LLM unavailable
  Explainability   Every recommendation must include evidence

------------------------------------------------------------------------

# 9. Assumptions

-   SIEM produces structured alerts.
-   Threat intelligence knowledge base is periodically updated.
-   Local LLM is available for inference.
-   Analyst validates all recommendations.

------------------------------------------------------------------------

# 10. Constraints

-   Final-year project implementation budget
-   Open-source technology stack
-   Offline-capable deployment
-   Limited GPU availability

------------------------------------------------------------------------

# 11. Success Metrics

-   ML Accuracy \>95%
-   Investigation generation \<5 seconds
-   RAG retrieval \<500 ms
-   Positive analyst feedback \>80%
-   Reduced manual investigation effort

------------------------------------------------------------------------

# 12. Risks

-   Dataset bias
-   Hallucinations from LLM
-   Knowledge-base staleness
-   False positives
-   SIEM schema variability

------------------------------------------------------------------------

# 13. Future Scope

-   Multi-tenant deployment
-   SOAR integration
-   Graph-based threat correlation
-   Federated learning
-   Autonomous playbook recommendation

------------------------------------------------------------------------

**End of Document**
