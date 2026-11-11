# Architecture Characteristics — Patient Registry (P1)

**Version:** 0.1 · **Status:** Draft · **Inputs:** `scope.md` stakeholder concerns

## 1. Method

1. Start from stakeholder concerns in `scope.md`.
2. Translate each concern to a named characteristic (Fundamentals Ch 4; ISO/IEC 25010 names in brackets).
3. Select the few that drive structure, rank the top three, make each measurable.
4. Record what is deliberately **not** a driver, and the trade-offs between those chosen.

## 2. Driving characteristics (ranked)

| Rank | Characteristic                                          | Definition for this system                                                                                                             | Measure (fitness function)                                                                                  | Enforced by (S1)                                                                                                     |
| ---- | ------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- |
| 1    | **Integrity** [Security → Integrity; Reliability]       | No two patient records share an identity; invalid data is refused at the boundary; a failed multi-step update leaves no partial change | Duplicate (authority, MRN) saves rejected: 100 % in tests · rollback test leaves data unchanged             | `IdentifierIdx` (unique); `Required`/`PATTERN`/`VALUELIST`; transaction in `MergeUpdate`; `PatientTest`, merge tests |
| 2    | **Interoperability** [Compatibility → Interoperability] | Any consumer can exchange a patient using one canonical, versioned format without bilateral agreements                                 | Every `Patient` field appears in the mapping spec · JSON export → import round-trip returns an equal record | `docs/canonical-patient-mapping.md`; `SCHEMAVERSION`; round-trip test                                                |
| 3    | **Auditability** [Security → Accountability]            | Who changed what, and when, can be reconstructed — for data and for the code that handles it                                           | Every saved record has UTC `CreatedAt`/`UpdatedAt` · every change traces Issue → PR → test → release        | `%OnBeforeSave`; `TestAuditTimestamps`; PR workflow                                                                  |

## 3. Supporting characteristics

| Characteristic                                                     | Why it matters here                                                              | Measure (initial target — revisit after first measurement)                                                     |
| ------------------------------------------------------------------ | -------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------------- |
| **Security and privacy** (cross-cutting)                           | Identity data is the most sensitive data class; POPIA applies to real data (S4+) | 0 Gitleaks findings · no real data (generator test) · application role has no admin rights                     |
| **Testability** [Maintainability → Testability]                    | Evidence over assertion (P5)                                                     | Every public class method has at least one `%UnitTest`; full suite runs on a clean namespace in CI on every PR |
| **Deployability / reproducibility** [Portability → Installability] | Disposable environments (P6)                                                     | Clean checkout to green tests using only documented commands; CI run under 15 minutes                          |
| **Maintainability** [Maintainability → Modifiability]              | Four projects reuse these ideas; Week 3 is a rebuild from memory                 | Package-per-concern layout; each class has one reason to change                                                |

## 4. Deliberately NOT drivers in S1

| Characteristic                  | Why not (yet)                           | Revisit when                              |
| ------------------------------- | --------------------------------------- | ----------------------------------------- |
| Scalability / elasticity        | Synthetic data, one user, small volumes | A load requirement exists (S6 k6 tests)   |
| Availability / mirroring        | Single disposable container             | A service-level objective is stated (S6+) |
| Performance (latency)           | No interface yet                        | The REST API exists (S2)                  |
| Recoverability (backup/restore) | Data is regenerable from fixtures       | Real or non-regenerable data exists       |

## 5. Trade-offs between the chosen characteristics

| Pair                                 | Tension                                            | Position taken                                                        |
| ------------------------------------ | -------------------------------------------------- | --------------------------------------------------------------------- |
| Integrity vs throughput/availability | Locks and transactions serialise writers           | Integrity wins; lock timeouts bound the wait (P7)                     |
| Interoperability vs privacy          | Every extra exposed field widens data sharing      | Contract carries only fields with a stated consumer need              |
| Auditability vs privacy              | Audit trails are themselves personal data          | Audit by identifier-free metadata where possible; scrub logs (P4)     |
| Deployability vs realism             | A disposable environment hides durability problems | Accept now; add durable volumes through an ADR when data must survive |
