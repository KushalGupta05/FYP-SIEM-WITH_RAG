# 05 -- Domain Model

> **Project:** ARIA -- Autonomous SIEM Triage Assistant\
> **Document:** Domain Model Specification\
> **Version:** 1.0

------------------------------------------------------------------------

# 1. Purpose

This document defines the business domain of ARIA independent of
implementation details. It identifies the core entities, their
relationships, ownership, lifecycle, invariants, and business rules.

The domain model is the contract between the business problem and the
software architecture.

------------------------------------------------------------------------

# 2. Domain Overview

``` mermaid
classDiagram

class Alert
class Investigation
class Prediction
class Evidence
class ThreatIntel
class MITRETechnique
class Playbook
class Feedback
class Analyst
class Embedding
class AuditLog

Alert "1" --> "1" Investigation
Alert "1" --> "1" Prediction
Investigation "1" --> "*" Evidence
Evidence "*" --> "*" ThreatIntel
ThreatIntel "*" --> "*" MITRETechnique
Investigation "*" --> "*" Playbook
Analyst "1" --> "*" Feedback
Investigation "1" --> "*" Feedback
ThreatIntel "1" --> "*" Embedding
Investigation "1" --> "*" AuditLog
```

------------------------------------------------------------------------

# 3. Aggregate Roots

  Aggregate       Purpose
  --------------- ---------------------------------------------------
  Alert           Central business object representing a SIEM event
  Investigation   AI-generated triage package
  Feedback        Human validation and corrections
  ThreatIntel     External knowledge retrieved by RAG

------------------------------------------------------------------------

# 4. Entity Specifications

## Alert

### Description

Represents the immutable normalized security event received from a SIEM.

### Attributes

  Field        Type       Required   Notes
  ------------ ---------- ---------- ----------------------
  alertId      UUID       Yes        Primary identifier
  source       Enum       Yes        Wazuh, Elastic, etc.
  severity     Integer    Yes        0--10
  eventType    Enum       Yes        Canonical event
  hostname     String     Yes        Source host
  username     String     No         User involved
  timestamp    DateTime   Yes        UTC
  rawPayload   JSON       Yes        Original event

### Lifecycle

``` mermaid
stateDiagram-v2
[*] --> Created
Created --> Validated
Validated --> Normalized
Normalized --> Classified
Classified --> Archived
```

### Invariants

-   alertId never changes
-   rawPayload is immutable
-   timestamp cannot be modified

------------------------------------------------------------------------

## Investigation

### Purpose

Represents the complete AI investigation for one alert.

### Contains

-   ML prediction
-   Confidence
-   Evidence
-   MITRE mappings
-   Recommended actions
-   Generated summary

### States

Draft → Generated → Reviewed → Closed

------------------------------------------------------------------------

## Prediction

Business meaning:

Stores deterministic ML output.

Fields

-   predictionId
-   modelVersion
-   predictedClass
-   confidence
-   probabilities
-   inferenceTime

Rules

-   One prediction belongs to one alert.
-   Predictions are immutable.

------------------------------------------------------------------------

## Evidence

Represents facts supporting an investigation.

Types

-   IOC
-   CVE
-   MITRE
-   Log snippet
-   Threat report
-   Analyst note

------------------------------------------------------------------------

## Threat Intelligence

Represents external cybersecurity knowledge.

Sources

-   MITRE ATT&CK
-   CVE
-   CERT-In
-   Vendor advisories
-   Internal SOPs

Metadata

-   source
-   publicationDate
-   confidence
-   tags
-   url

------------------------------------------------------------------------

## MITRE Technique

Fields

-   techniqueId
-   name
-   tactic
-   description
-   detection
-   mitigation

Relationship

Many pieces of evidence may reference one technique.

------------------------------------------------------------------------

## Playbook

Contains standardized investigation procedures.

Attributes

-   playbookId
-   title
-   severity
-   checklist
-   owner
-   version

------------------------------------------------------------------------

## Feedback

Captures analyst interaction.

Fields

-   verdict
-   falsePositive
-   escalation
-   notes
-   reviewDuration
-   reviewer

Business Rule

Feedback never modifies the original prediction.

------------------------------------------------------------------------

## Analyst

Represents authenticated SOC personnel.

Fields

-   analystId
-   role
-   team
-   permissions

Roles

-   Tier-1
-   Tier-2
-   SOC Manager
-   Admin

------------------------------------------------------------------------

## AuditLog

Every important action generates an immutable audit record.

Example actions

-   Alert received
-   Prediction generated
-   Prompt executed
-   Feedback submitted

------------------------------------------------------------------------

# 5. Relationship Matrix

  Entity A        Entity B         Cardinality
  --------------- ---------------- -------------
  Alert           Investigation    1:1
  Alert           Prediction       1:1
  Investigation   Evidence         1:N
  Investigation   Feedback         1:N
  Evidence        ThreatIntel      N:N
  ThreatIntel     MITRETechnique   N:N
  Analyst         Feedback         1:N

------------------------------------------------------------------------

# 6. Business Rules

1.  Every Alert must have exactly one Prediction.
2.  An Investigation cannot exist without an Alert.
3.  Feedback requires authentication.
4.  Raw alerts are immutable.
5.  Audit records cannot be deleted.
6.  Investigation evidence must reference at least one source.

------------------------------------------------------------------------

# 7. Domain Events

``` mermaid
flowchart TD
AlertCreated-->PredictionGenerated
PredictionGenerated-->InvestigationGenerated
InvestigationGenerated-->EvidenceLinked
EvidenceLinked-->FeedbackSubmitted
FeedbackSubmitted-->InvestigationClosed
```

------------------------------------------------------------------------

# 8. Ubiquitous Language

  Term            Meaning
  --------------- -------------------------
  Alert           Normalized SIEM event
  Investigation   AI triage package
  Prediction      ML classification
  Evidence        Supporting facts
  Feedback        Human validation
  Playbook        Investigation checklist
  ThreatIntel     External knowledge

------------------------------------------------------------------------

# 9. Extension Points

Future entities

-   Incident
-   Case
-   Ticket
-   SOAR Action
-   Asset Inventory
-   Threat Actor
-   Campaign

------------------------------------------------------------------------

# 10. Next Document

**06_Database_Design.md**

Defines logical schema, ER diagrams, normalization, indexes, partitions,
constraints, and storage strategy.
