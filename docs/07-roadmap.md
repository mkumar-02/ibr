# Roadmap

Documentation-first. Implementation follows once open decisions are locked.

Aligned with OpenG2P platform roadmap: **finish DCI IBR work in 2026**.

## Phase 0 — Spec & product docs _(current)_

**Goal:** Shared understanding before code; **choose product shape**.

- [x] Problem framing vs Social Registry and PBMS
- [x] Shared architecture context + [architecture options](10-architecture-options.md)
- [x] Domain model mapped to DCI objects
- [x] Process mapping PRS.IBR.01–04
- [x] API and integration contracts
- [ ] Lock [Decision 0](08-open-decisions.md#0-product-shape-where-ibr-lives) and remaining open decisions
- [ ] Pin DCI schema / OpenAPI commit SHAs used as normative input
- [ ] Stakeholder review (product, registry, PBMS, country implementers)

**Exit:** Decision 0 closed; Phase 1 scaffold matches the chosen option (manifestation **or** standalone service ± registry client).

---

## Phase 1 — Write path (PRS.IBR.02 + PRS.IBR.03)

**Goal:** IBR can receive and store enrolments and benefits from PBMS.

Deliverables (adjust packaging to Decision 0):

- Service or manifestation skeleton + deployables
- Domain tables: Beneficiary, Programme, Enrolment, Benefit (+ Group/Member as needed)
- `POST /enrollment/updates`
- Idempotent ingest + acknowledgement
- PBMS publisher adapter (minimum: one programme cycle → IBR)
- Basic read of ingested records (IBR UI and/or API)
- If **B+:** stub `ibr-client` + one registry panel (e.g. NSR) reading by UIN

**Exit:** A completed PBMS disbursement cycle appears correctly in IBR; re-publish does not duplicate.

---

## Phase 2 — Enrolment check (PRS.IBR.01)

**Goal:** Eligibility can ask “enrolled in programme X?” safely.

Deliverables:

- `POST /enrolled` (individual + batch)
- Status policy for what counts as “yes”
- PBMS eligibility rule integration (at least one reference rule)
- Partner audit of checks

**Exit:** Eligibility list generation can exclude/include based on IBR without receiving extra PII.

---

## Phase 3 — Search, events, multi-publisher

**Goal:** Full DCI read/interop surface.

Deliverables:

- DCI sync/async search with outbound templates
- Subscribe / notify for enrolment and benefit events
- Second publisher path (second PBMS or external SP-MIS stub)
- Overlap / multi-programme staff views

**Exit:** Partners can search and subscribe; two publishers coexist with `source_system` provenance.

---

## Phase 4 — Analysis (PRS.IBR.04) and product polish

**Goal:** Coverage and equity analysis without turning IBR into SR.

Deliverables:

- Analysis job pattern: IBR subjects × SR attributes
- Dashboards or export contracts for M&E
- Beneficiary portal (optional): own enrolments/benefits
- Hardening: retention, consent edge cases, load tests on batch ingest

**Exit:** Country can answer coverage / overlap questions with documented pipelines.

---

## Dependency order

```
Phase 0 docs/decisions
        │
        ▼
Phase 1 write path  ───▶  Phase 2 /enrolled
        │                        │
        └──────────┬─────────────┘
                   ▼
            Phase 3 search/events
                   │
                   ▼
            Phase 4 analysis/polish
```

## Non-goals for early phases

- Replacing PBMS beneficiary lists
- Full BI warehouse product (IBR feeds analysis; it is not the warehouse)
- Payment initiation from IBR
- Automatic SR→IBR socio-economic mirroring

## Tracking

Prefer GitLab issues / milestones (OpenG2P convention) with labels such as `ibr`, `dci`, `phase-1`. Reference issues as `#<n>` in commits and MRs when implementation starts.
