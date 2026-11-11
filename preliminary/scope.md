# Preliminary Phase — Scope and Tailoring

**Version:** 0.1 · **Status:** Draft · **Owner:** Architecture author (learner)

## 1. Purpose

Establish the architecture capability for a **synthetic patient registry** that becomes the system of record
for patient identity, and for the integration projects (P2–P4) that depend on it.

## 2. Organisation scope

A fictional organisation ("the Trust") with one registry service and several consuming systems.
The "organisation" is a learning construct; no real organisation is described.

## 3. In scope (S1)

- Patient identity domain: identity = **(assigning authority + MRN)**; demographics; address; audit metadata.
- Synthetic data only; reproducible environment; automated tests as evidence.
- Principles, characteristics, decisions and diagrams for project P1 (Weeks 1–6) and as groundwork for P2–P4.

## 4. Out of scope (S1)

- Real patient data (PHI) of any kind; any real identifier format.
- Patient matching / record linkage algorithms and clinical or financial mappings (human-governed decisions).
- Production deployment; high-availability design; mirroring.\
- Paid tools or services (Rule 11: free and open source only).

## 5. Stakeholders and concerns (fictional roles)

| Stakeholder                                   | Concern                                          | How the architecture responds                                          |
| --------------------------------------------- | ------------------------------------------------ | ---------------------------------------------------------------------- |
| Registry operator                             | Can I rebuild, deploy and diagnose it?           | Disposable environment, scripted setup, health and logs                |
| Consuming system owners (lab, clinical, FHIR) | Stable identifiers and a stable contract         | Issuer-qualified identity; contract-first interfaces; versioned schema |
| Information officer / privacy                 | Is personal data minimised and protected?        | Synthetic data only; classification and scrubbing from S4              |
| Auditor                                       | Can changes be traced?                           | Issue → PR → test → release; audit timestamps; ADRs                    |
| Architecture author (me)                      | Do I understand and can I defend every decision? | Level-5 scaffold now; full rebuild in Week 3; own-words ADRs           |

## 6. Tailoring — how TOGAF is used

| TOGAF element               | Used?                | How                                                                     |
| --------------------------- | -------------------- | ----------------------------------------------------------------------- |
| Preliminary Phase           | Yes                  | This repository; S1                                                     |
| Architecture Principles     | Yes                  | `principles/principles.md`                                              |
| Architecture Repository     | Yes                  | Folder layout in the README                                             |
| Requirements Management     | Yes                  | GitHub Issues linked to branches, PRs, tests, releases                  |
| ADM Phases A–H (full cycle) | Selectively, from S2 | Vision/Business/Data/Application/Technology views as each project needs |
| Formal ArchiMate models     | Later                | Introduced when a viewpoint needs it (S3+); C4 now                      |

## 7. Assumptions and constraints

- Constraint: free/open-source tooling; public GitHub repositories; ~10–12 hours per week.
- Assumption: all data is synthetic; any resemblance to a real person is coincidental and a defect.
