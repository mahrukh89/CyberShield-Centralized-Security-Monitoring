# Solution Architecture — CyberShield Centralized Security Monitoring

## Purpose

This document describes the target technical architecture the project delivered. It exists so that a reader (recruiter, hiring manager, or engineer) can see that the Project Manager understood the solution being coordinated, not only the paperwork around it.

> **Portfolio note:** Component names are generic/vendor-neutral by design, since this is a fictional simulation. The architecture pattern is representative of real centralized-monitoring (SIEM-style) implementations.

---

## 1. Architecture Overview

```text
 ┌─────────────────────────────────────────────────────────────────────┐
 │                         LOG SOURCE LAYER                            │
 │                                                                       │
 │   Endpoints      Identity/IAM      Cloud Services      Network       │
 │  (EDR/AV logs)   (auth/SSO logs)   (IaaS/SaaS logs)   (FW/IDS logs)  │
 └────────┬──────────────┬───────────────────┬──────────────┬──────────┘
          │              │                   │              │
          ▼              ▼                   ▼              ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │                      COLLECTION & INGESTION LAYER                    │
 │        Log forwarders / agents → normalization → enrichment          │
 └───────────────────────────────┬───────────────────────────────────────┘
                                  ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │                 CENTRALIZED MONITORING PLATFORM (SIEM)               │
 │   Correlation engine · Detection rules · Priority use cases          │
 └────────┬───────────────────────────────────────────────┬─────────────┘
          ▼                                                ▼
 ┌───────────────────────────┐                 ┌────────────────────────┐
 │     ALERT TRIAGE LAYER     │                 │   REPORTING LAYER       │
 │  Severity scoring           │                 │  Executive dashboard    │
 │  Deduplication               │                 │  KPI/trend reporting    │
 │  Escalation workflow         │                 │  Compliance reporting   │
 └────────┬────────────────────┘                 └────────────────────────┘
          ▼
 ┌─────────────────────────────────────────────────────────────────────┐
 │                    RESPONSE & OPERATIONS LAYER                       │
 │   SOC Analyst triage → Incident ticket → IT Ops/Security Eng action  │
 └─────────────────────────────────────────────────────────────────────┘
```

---

## 2. Component Description

| Layer | Component | Responsibility | Primary Owner |
|---|---|---|---|
| Log Source | Endpoints, Identity, Cloud, Network | Generate raw security telemetry | IT Operations / Cloud Team |
| Collection | Forwarders/agents, normalization | Ship logs reliably, standardize format | IT Operations |
| Platform | SIEM / correlation engine | Store, correlate and evaluate events against detection rules | Security Engineering |
| Detection | Priority use cases | Encode what "bad" looks like (e.g. brute-force, impossible travel, privilege escalation) | Security Lead |
| Triage | Alert workflow | Score, deduplicate and route alerts | SOC |
| Reporting | Dashboard | Present KPIs and security posture to management | Security Lead / PM |
| Response | Incident/ticketing | Track investigation and remediation to closure | SOC / IT Operations |

---

## 3. Data Flow (Alert Lifecycle)

```text
Event generated at source
        │
        ▼
Collected & normalized
        │
        ▼
Evaluated against detection use case
        │
        ▼
Alert raised → severity scored → deduplicated
        │
        ▼
Routed per escalation matrix
        │
        ▼
SOC Analyst triage
        │
        ├── False positive → tune detection rule → close
        │
        └── True positive → incident ticket → investigation → remediation → closure
                                                                        │
                                                                        ▼
                                                            Logged in reporting/KPIs
```

---

## 4. Priority Log Sources (In-Scope for Onboarding)

| Priority | Source Category | Rationale |
|---|---|---|
| P1 | Identity/IAM (authentication, SSO, MFA) | Highest signal for account compromise |
| P1 | Endpoint (EDR/AV) | Direct indicator of host-level compromise |
| P2 | Cloud services (IaaS/SaaS admin activity) | Growing attack surface, privilege misuse |
| P2 | Network (firewall, IDS/IPS) | Perimeter and lateral-movement visibility |
| P3 | Application logs | Contextual enrichment for investigations |

Prioritization follows the **90% priority-source onboarding** success criterion defined in the Project Charter — P1 and P2 sources are mandatory for the ≥90% target; P3 sources are best-effort within the 12-week window.

---

## 5. Non-Functional / Architectural Constraints

| Constraint | Detail |
|---|---|
| Availability | Monitoring platform must maintain high availability; ingestion gaps are treated as a monitoring risk |
| Data retention | Logs retained per compliance requirement; referenced in UAT/documentation |
| Access control | Role-based access to the SIEM and dashboards (SOC vs. management views) |
| Auditability | All configuration and detection-rule changes go through change control |
| Scalability | Architecture assumes incremental source onboarding beyond the initial 12-week scope |
| Integration | No replacement of existing endpoint tooling — the platform ingests from, but does not replace, existing security products (see Scope: Out of Scope) |

---

## 6. Why This Belongs in a PM Portfolio

A Project Coordinator/Junior PM does not design the architecture, but a strong one:

- Understands the system well enough to sequence a realistic WBS and schedule (log source onboarding **before** detection tuning, detection **before** dashboarding, etc.)
- Can translate technical risk (e.g. "SIEM ingestion pipeline is a single point of failure") into a project risk with an owner and a response plan
- Can hold a technical conversation with a Security Lead without needing every requirement re-explained
- Can scope change requests (e.g. CR-001 adding an MTTA KPI) with an accurate sense of what's technically trivial vs. structural

This document is the artifact that demonstrates that understanding.

---

## 7. Traceability to Other Artifacts

| Architecture Element | Related Artifact |
|---|---|
| Priority log sources | `01-project-initiation/requirements.md` (FR-01), `02-planning/wbs.md` (4.2) |
| Detection use cases | `01-project-initiation/requirements.md` (FR-02), `02-planning/wbs.md` (3.3, 4.3) |
| Alert triage / escalation | `01-project-initiation/requirements.md` (FR-03, FR-05) |
| Dashboard / reporting | `01-project-initiation/requirements.md` (FR-04), `05-monitoring/` |
| Ingestion pipeline risk | `03-risk-management/risk-register.xlsx` (R-01) |
| MTTA KPI addition | `06-change-management/change-request.md` (CR-001) |

> **Portfolio note:** This is a fictional architecture created for demonstration. No real vendor, product, or organizational infrastructure is represented.
