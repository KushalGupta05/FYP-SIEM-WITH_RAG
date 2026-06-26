## **Autonomous SIEM Triage Assistant (ARIA)**

### **Master Architecture Document**

**Version:** 1.0

**Status:** Draft

**Project Code Name:** ARIA (Autonomous Response & Investigation Assistant)

---

# **Table of Contents**

1\. Executive Summary

2\. Product Vision

3\. Business Problem

4\. Users & Stakeholders

5\. Functional Requirements

6\. Non Functional Requirements

7\. High Level Architecture

8\. Component Architecture

9\. Data Flow Architecture

10\. Machine Learning Architecture

11\. Retrieval-Augmented Generation Architecture

12\. Threat Intelligence Pipeline

13\. Knowledge Base Architecture

14\. Backend Architecture

15\. API Gateway

16\. Authentication & Authorization

17\. Database Design

18\. Event Processing Pipeline

19\. Alert Lifecycle

20\. Investigation Workflow

21\. Analyst Feedback Engine

22\. Continuous Learning Pipeline

23\. Deployment Architecture

24\. Container Architecture

25\. Monitoring Architecture

26\. Logging Architecture

27\. Security Architecture

28\. Scalability

29\. Disaster Recovery

30\. Future Enhancements

31\. Technology Stack

32\. Appendix  
---

# **Project Overview**

## **Product Name**

**ARIA — Autonomous SIEM Triage Assistant**

---

## **Elevator Pitch**

ARIA is an AI-assisted investigation platform that sits on top of open-source SIEM solutions and reduces the amount of manual work performed by Tier-1 Security Operations Center (SOC) analysts.

Instead of replacing analysts, ARIA automates the repetitive investigative tasks that occur after an alert is generated. It classifies incoming alerts using supervised machine learning, retrieves contextual threat intelligence through Retrieval-Augmented Generation (RAG), correlates relevant security knowledge, maps attacker behavior to the MITRE ATT\&CK framework, and generates a structured Investigation Briefing that accelerates analyst decision-making. This aligns with the refined problem framing in your project documents: the primary bottleneck is **context assembly**, not a lack of detections.

---

# **Vision**

Build an intelligent investigation assistant that reduces alert fatigue while remaining transparent, explainable, and compatible with open-source SIEM ecosystems.

---

# **Mission**

Reduce the average Tier-1 investigation time from minutes to seconds without sacrificing analyst control.

---

# **Core Philosophy**

Traditional SIEM

Generate Alert

↓

Human Investigation

↓

Search Threat Intel

↓

Search MITRE

↓

Write Notes

↓

Escalate

ARIA

Generate Alert

↓

AI Investigation

↓

Investigation Brief Generated

↓

Human Verification

↓

Escalate

Notice that the analyst remains the decision-maker; ARIA assembles evidence and recommendations.

---

# **Problem Statement**

Modern Security Operations Centers receive thousands of security alerts every day. Existing SIEM platforms excel at detection but require analysts to manually collect threat intelligence, identify MITRE ATT\&CK techniques, consult playbooks, and document findings before deciding whether an alert should be escalated. This repetitive context assembly creates alert fatigue, slows incident response, and contributes to analyst burnout. ARIA addresses this gap by combining deterministic machine learning with retrieval-augmented generation to automatically assemble investigation context and produce structured triage packages.

---

# **Design Principles**

## **1\. Human-in-the-loop**

The system never performs autonomous remediation.

Analyst approval is mandatory.

---

## **2\. Explainable AI**

Every recommendation must include:

* ML confidence score  
* supporting evidence  
* retrieved documents  
* MITRE mapping  
* reasoning chain (summarized)  
* recommended next steps

---

## **3\. Vendor Agnostic**

ARIA must integrate with:

* Wazuh  
* Elastic  
* Graylog  
* Splunk (future)  
* Microsoft Sentinel (future)

through normalized alert formats.

---

## **4\. Modular Architecture**

Every subsystem is independently replaceable.

For example:

Random Forest

↓

XGBoost

↓

Neural Network

can be swapped without changing the RAG subsystem.

Likewise, ChromaDB can later be replaced with Pinecone, Weaviate, or Milvus without redesigning the rest of the platform.

---

## **5\. Security by Design**

Security is embedded into every layer:

* encrypted communication  
* least privilege  
* API authentication  
* audit logging  
* immutable investigation history  
* role-based access control

---

# **Product Goals**

### **Primary Goals**

* Reduce analyst workload  
* Reduce investigation time  
* Reduce false-positive handling effort  
* Improve context availability  
* Standardize investigation quality  
* Improve SOC productivity

---

### **Secondary Goals**

* Learn analyst preferences within a deployment  
* Build organizational knowledge over time  
* Provide consistent investigation reports  
* Enable future SOAR integration  
* Support compliance reporting

---

# **Success Metrics (KPIs)**

| Metric | Target |
| ----- | ----- |
| ML Classification Accuracy | \>95% |
| Investigation Brief Generation Time | \<5 seconds |
| RAG Retrieval Latency | \<500 ms |
| API Response Time | \<300 ms (excluding LLM inference) |
| Concurrent Users | 50 (FYP), 500+ (future) |
| Alert Processing Throughput | 100 alerts/min (FYP target) |
| System Availability | 99.5% (future deployment) |
| Analyst Feedback Capture Rate | \>80% of reviewed alerts |

---

# **High-Level System Flow**

                   \+----------------------+  
                   |     SIEM Platform    |  
                   | Wazuh / Elastic etc. |  
                   \+----------+-----------+  
                              |  
                              v  
                   \+----------------------+  
                   |   Alert Ingestion    |  
                   \+----------+-----------+  
                              |  
                              v  
                   \+----------------------+  
                   | Alert Normalization  |  
                   \+----------+-----------+  
                              |  
                              \+-------------------+  
                              |                   |  
                              v                   v  
                 \+--------------------+   \+--------------------+  
                 |  ML Classification |   | Metadata Enrichment|  
                 \+---------+----------+   \+---------+----------+  
                           |                        |  
                           \+-----------+------------+  
                                       |  
                                       v  
                           \+------------------------+  
                           |   Decision Engine      |  
                           \+-----------+------------+  
                                       |  
                                       v  
                           \+------------------------+  
                           |   RAG Context Builder  |  
                           \+-----------+------------+  
                                       |  
                                       v  
                        \+-------------------------------+  
                        | Vector DB \+ Threat Knowledge  |  
                        \+---------------+---------------+  
                                        |  
                                        v  
                               \+----------------+  
                               | Local LLM      |  
                               | Investigation  |  
                               | Brief          |  
                               \+--------+-------+  
                                        |  
                                        v  
                             \+----------------------+  
                             | Analyst Dashboard    |  
                             \+--------+-------------+  
                                      |  
                                      v  
                             \+----------------------+  
                             | Feedback Engine      |  
                             \+--------+-------------+  
                                      |  
                                      v  
                             \+----------------------+  
                             | Knowledge Updates    |  
                             \+----------------------+  
