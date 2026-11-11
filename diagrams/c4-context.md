# C4 Level 1 — System Context: Patient Registry (P1)

**Version:** 0.1 · **Status:** Draft · Solid boxes exist in S1; dashed boxes are planned future systems.

```mermaid
flowchart LR
  dev["Registry operator / developer<br/>[Person]"]
  reg["Patient Registry (P1)<br/>[Software System]<br/>System of record for patient identity.<br/>Synthetic data only."]
  gh["GitHub and GitHub Actions<br/>[External System]<br/>Source, review, CI, releases"]
  gw["API gateway<br/>[Planned: S2]"]
  idp["Identity provider<br/>[Planned: S2]"]
  lab["Lab results hub (P2)<br/>[Planned: S3-S4]"]
  fhir["FHIR interop bridge (P3)<br/>[Planned: S5-S6]"]
  leg["Legacy estate (P4)<br/>[Planned: S7-S8]"]

  dev -->|"develops, tests, operates"| reg
  dev -->|"commits, opens pull requests"| gh
  gh -->|"builds, tests, releases"| reg
  gw -.->|"will call (REST)"| reg
  idp -.->|"will authenticate callers for"| gw
  lab -.->|"will resolve patient identity via"| gw
  fhir -.->|"will exchange patients via"| gw
  leg -.->|"will be reconciled with"| gw

  classDef planned stroke-dasharray: 5 5,fill:#f2f2f2,color:#222;
  class gw,idp,lab,fhir,leg planned;
```

## Reading the diagram

- The registry is the **only** system built in S1; GitHub supplies the CI that proves it.
- All planned systems reach the registry **through the gateway** (principle P2: contract first — the gateway
  enforces the published interface).
- Technology (IRIS, Docker, KrakenD) is intentionally absent at this level; it appears at Level 2.
