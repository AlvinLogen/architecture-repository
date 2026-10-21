# Architecture Principles

**Version:** 0.1 · **Status:** Draft · Template: Name · Statement · Rationale · Implications (TOGAF)
Each principle also lists its **Evidence** — where the principle is already visible in the build.

## P1 — Git is the source of truth

**Statement.** Everything needed to build, configure, test and understand the system — code, configuration,
contracts, fixtures and decisions — is under version control; nothing essential exists only on a server or in a GUI.

**Rationale.** Reproducibility, review and audit all depend on one authoritative history. A server-only change
is invisible, untested and lost on rebuild.

**Implications.**

- Environments are built from scripts (installer, compose), never by hand.
- Class definitions, including compiled Storage once data matters, are exported and committed.
- A change made in a management UI must be reproduced in Git or discarded.

**Evidence.** `Installer.cls`, `create-namespace.script`, `docker-compose.yml`, `src/`.

## P2 — Contract before code

**Statement.** An interface or transformation is specified (schema, mapping, OpenAPI) before, or together with,
its first implementation.

**Rationale.** A written contract can be reviewed by consumers, generates tests, and prevents N-to-N private
agreements between systems.

**Implications.**

- Every transformation has an explicit mapping specification.
- Contracts are semantically versioned; a breaking change means a new major version.
- No endpoint ships without its OpenAPI description (from S2).

**Evidence.** `docs/canonical-patient-mapping.md`; shared test vectors; `SCHEMAVERSION`.

## P3 — Identity is explicit and issuer-qualified

**Statement.** An identifier is meaningful only together with the authority that issued it; internal surrogate
keys are never exposed as business identity.

**Rationale.** The same number from two issuers can name two different people. Merges, links and exchanges
stay safe only if the issuer travels with the value.

**Implications.**

- Uniqueness is enforced on (authority, identifier), not on the identifier alone.
- Contracts and messages carry both parts; `%Id()` never appears in a contract.
- Deciding that two records are the same person is a governed human decision, not an automatic one.

**Evidence.** `IdentifierIdx`; `FindByIdentifier`; canonical fields `assigningAuthority` + `mrn`.

## P4 — Protect personal data by design

**Statement.** Collect and keep the minimum, treat all personal data as sensitive, never log identifiers, and use
synthetic data only until real-data governance exists.

**Rationale.** Privacy law (for example POPIA in South Africa), patient harm, public trust, and the fact that
these repositories are public.

**Implications.**

- Synthetic data is enforced by tests, not by good intentions.
- Errors, logs and traces exclude identifiers; scrubbing is built before real data is ever considered.
- Secrets live outside Git; application accounts hold least privilege.

**Evidence.** `SYNTH-` authorities asserted in generator tests; `Mrn` errors omit the value; `secrets/` ignored.

## P5 — Evidence over assertion

**Statement.** A quality claim is true only when an automated check in CI demonstrates it; manual success is not
evidence.

**Rationale.** Memory and confidence are unreliable; a repeatable check protects every future change.

**Implications.**

- Each driving characteristic has a fitness function (see `characteristics.md`).
- Required checks block merging; evidence is attached to releases.
- A bug fix starts with a failing test.

**Evidence.** pytest and `node:test` suites; `%UnitTest` and CI on Night 5.

## P6 — Environments are disposable and reproducible

**Statement.** Any environment can be destroyed and rebuilt from Git in minutes; no environment is precious.

**Rationale.** It removes "works on my machine", makes CI identical to development, and makes recovery a routine.

**Implications.**

- Durable storage is introduced only where data must outlive the environment, by ADR.
- Base images are pinned; setup scripts are idempotent.
- Docker Compose reproduces every environment locally.

**Evidence.** `docker-compose.yml`; idempotent `Installer.Run`; the data vanishing on `docker compose down`.

## P7 — Fail explicitly and safely

**Statement.** Errors are detected, reported with a typed result, and leave data consistent; the system never
fails silently and never half-completes a multi-step write.

**Rationale.** In an identity system a refused update is acceptable; a half-applied one is a data-integrity incident.

**Implications.**

- Explicit `%Status` handling; `TRY/CATCH` for the unexpected.
- Multi-step writes sit inside a transaction; lock waits have timeouts.
- Interfaces return standard problem details; asynchronous flows get idempotency, retry and a dead-letter path.

**Evidence.** `Installer.Run`; the merge-safe update (Night 4); rollback tests (Night 5).

## P8 — Choose the simplest sufficient mechanism, and record why

**Statement.** Prefer the simplest design that satisfies the actual requirement, and record each significant
decision — including the integration mechanism (API, queue, event, batch, file) — as an ADR with alternatives
and consequences.

**Rationale.** Complexity is a cost paid for ever; unrecorded decisions are either re-argued or copied without
understanding.

**Implications.**

- A significant decision has an ADR before it is built, with revisit triggers.
- Speculative layers (frameworks, abstractions) are rejected until a requirement demands them.
- "Out of scope" lists are maintained.

**Evidence.** `scope.md` out-of-scope list; ADR-0001.

## Priority when principles conflict

P3 (identity) and P4 (data protection) outrank convenience. Any other conflict is resolved in an ADR that names
both principles and the trade-off taken.

## Quality check (TOGAF five tests) — applied

| Principle | Understandable          | Robust                                 | Complete                                                                           | Consistent                              | Stable                                                     |
| --------- | ----------------------- | -------------------------------------- | ---------------------------------------------------------------------------------- | --------------------------------------- | ---------------------------------------------------------- |
| P1–P8     | One-sentence statements | Each rules out a concrete bad decision | Cover source, contract, identity, data, evidence, environment, failure, complexity | Conflicts resolved by the priority rule | Technology-neutral wording; no product names in statements |
