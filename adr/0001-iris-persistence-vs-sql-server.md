# ADR-0001 — IRIS native persistence vs external SQL Server for the patient registry

**Status:** Accepted · **Date:** 2026-10-21
**Deciders:** Architecture author · **Issue:** A-01
**Related:** Characteristics 1–3 (integrity, interoperability, auditability), testability, deployability · Principles P1, P3, P6, P8

## Context

The registry will be the system of record for patient identity (assigning authority + MRN, demographics, address)
and must be provable by automated tests from a clean state. Later projects (P2–P4) will consume it through an API
and messages. SQL Server appears elsewhere in the estate (P4 treats it as a _legacy source_), so the question is
real: should the registry's own data live in IRIS persistent classes, or in SQL Server with IRIS calling it?

Constraints: free and open-source tools only; one learner; CI must create the whole system from scratch;
IRIS Community Edition licence limits apply (verify the current limits; synthetic data is far below them).

## Decision drivers

Integrity of identity · interoperability · auditability · testability · deployability · operational simplicity

## Options considered

**A. IRIS native persistence.** Persistent classes with a unique composite index, ACID transactions and a SQL
projection; the Interoperability engine that later sprints need runs on the same platform.

- Pros: one engine for storage, business logic and integration; clean namespace in seconds; unique index and
  transactions are first-class; SQL access to the same data for tools and reporting.
- Cons: IRIS/ObjectScript specialisation and fewer practitioners; global storage is platform-specific;
  no ready-made temporal/audit features.

**B. External SQL Server as system of record, IRIS as integration layer.** Domain rules in ObjectScript calling
SQL Server through SQL access.

- Pros: familiar DBA tooling and skills; mature constraints, temporal tables and change tracking.
- Cons: a second runtime to build, migrate, secure and run in every CI job; extra network hop and failure mode
  between rules and data; two places for integrity rules to live.

**C. Hybrid: IRIS store plus SQL Server master, synchronised.**

- Pros: local speed with an enterprise master.
- Cons: dual-write or synchronisation logic, conflict resolution and two sources of truth — the worst case for
  an identity system.

## Trade-off analysis

Scores 1 (poor) to 5 (strong). Weights sum to 100. Weighted score = sum(weight x score) / 5, giving 0–100.

| Criterion                              | Weight | A: IRIS | B: SQL Server | C: Hybrid |
| -------------------------------------- | ------ | ------- | ------------- | --------- |
| Integrity and transactional guarantees | 25     | 4       | 4             | 2         |
| Interoperability / integration fit     | 20     | 5       | 3             | 3         |
| Testability and reproducible CI        | 20     | 5       | 3             | 2         |
| Operational simplicity                 | 15     | 4       | 2             | 1         |
| Auditability support                   | 10     | 3       | 4             | 3         |
| Learning-objective fit                 | 10     | 5       | 2             | 3         |
| **Weighted score (0–100)**             | 100    | **88**  | **62**        | **45**    |

Sensitivity checks (the weights are judgement, so test them):

- Remove the learning-objective criterion and re-normalise: A 87, B 64, C 43 — the ranking is unchanged.
- Raise B's integrity score to 5: B reaches 67 — A still leads.

Interpretation: the result is driven by testability, simplicity and platform fit, not by one generous score.

## Decision

We will store the patient registry in **IRIS native persistent classes** (Option A), with the SQL projection and a
canonical JSON export as the exit path. SQL Server remains a _source or target system_ integrated through IRIS
in later projects; it is not the registry's store.

## Consequences

**Positive**

- A single container builds, tests and tears down quickly, so the fitness functions run on every pull request.
- Integrity rules (unique identity, transactions, validation) live in one engine next to the domain model.
- The same platform hosts the Interoperability engine needed from S3.

**Negative**

- Platform and skills concentration: fewer people can maintain ObjectScript than SQL.
- Community Edition licence limits apply; production would need a licensed edition.
- Storage definitions are IRIS-specific and must be exported and committed once data matters (P1).
- Audit capability must be designed by us; it is not provided out of the box.

**Neutral / follow-ups**

- Mitigate lock-in with the canonical mapping spec, the JSON export/import round-trip and the SQL projection.
- Record a separate ADR when durable storage is introduced (P6).
- Design auditing properly with the REST layer (S2).

## Revisit triggers

Reopen this decision if any of these occurs: (1) an enterprise standard requires SQL Server as the system of
record; (2) data volume or licence limits are approached; (3) reporting needs frequent joins with SQL Server estate
data; (4) the operating team cannot support IRIS; (5) a temporal or built-in audit requirement is better met by
SQL Server.
