> **CONCEPT — what an ADR is.** An **Architecture Decision Record** is a short document capturing _one_ significant decision: the context that forced it, the options considered, what was chosen, and the consequences — including the bad ones. It is **immutable once accepted**: if the decision changes, you write a _new_ ADR that supersedes the old one, so the history of reasoning is preserved. In TOGAF terms the ADR log is your **Governance Log** for architecture decisions.

### File: `adr/0000-template.md`

```markdown
# ADR-NNNN — Title (a decision, stated as a noun phrase)

**Status:** Proposed | Accepted | Superseded by ADR-XXXX · **Date:** YYYY-MM-DD
**Deciders:** · **Issue:** · **Related principles / characteristics:**

## Context

The forces at play: requirement, constraints, what makes this decision necessary now.

## Decision drivers

The characteristics and principles that matter most for this choice.

## Options considered

For each option: a short description, and the most important pros and cons.

## Trade-off analysis

How the options compare against the drivers (matrix, scenarios, or prose).

## Decision

"We will …" — one clear statement, in your own words.

## Consequences

Positive, negative and neutral results. Be honest about the costs.

## Revisit triggers

The concrete events that would make us reopen this decision.
```
