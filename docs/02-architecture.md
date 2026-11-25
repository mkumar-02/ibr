# Architecture

> **Product shape is not decided yet.**  
> Compare options in [Architecture options](10-architecture-options.md) and record the choice as [Decision 0](08-open-decisions.md).  
> This page holds **shared** context that applies to every serious option (A / A′ / B / B+). Option C (benefits SoR inside FR/NSR/DR) is documented in the options brief and is **not** assumed here.

OpenG2P’s 2026 roadmap calls out finishing DCI IBR work alongside Disability Registry interoperability.

## Operating model (shared)

| Actor | Role toward IBR |
|---|---|
| PBMS / other SP-MIS (partners) | **Write** enrolments & benefits; call `/enrolled` |
| Registry / programme staff | **Read** programmes & benefits; investigate overlap |
| Field intake | Does **not** register people into IBR |

IBR is a **cross-programme ledger**, not a social registry and not PBMS.

## System context (shared)

```
┌──────────────────┐   read eligibility    ┌─────────────────┐
│ Social Registry  │◀──────────────────────│      PBMS       │
│ NSR / Farmer /   │                       │    (SP-MIS)     │
│ Disability / …   │                       └────────┬────────┘
└────────▲─────────┘                                │
         │                                          │ PRS.IBR.02 / .03
         │ PRS.IBR.04                               │ enrolment + benefits
         │ analysis query                           │
         │                                          ▼
         │                                 ┌─────────────────┐
         └─────────────────────────────────│       IBR       │
                                           │  (ledger SoR)   │
                                           └────────┬────────┘
                                                    │
                          PRS.IBR.01 / search       │
                          + staff read (path TBD)   │
                                                    ▼
                                           ┌─────────────────┐
                                           │ Other SP-MIS /  │
                                           │ analytics /     │
                                           │ policy tools    │
                                           └─────────────────┘
```

How staff **reach** IBR data depends on the option:

| Option | Staff reach IBR via |
|---|---|
| A / A′ | IBR’s own staff portal |
| B | Thin IBR admin UI or APIs only |
| B+ | Panels inside NSR / FR / DR that **call** IBR (read) |
| C | Data stored on SR records (not recommended as SoR) |

## Primary data flows (shared)

### 1. Publish benefits and enrolments (PBMS → IBR)

1. PBMS builds DCI `Beneficiary` records after a cycle or status change.
2. PBMS calls `POST /enrollment/updates` (API.IBR.01).
3. IBR upserts Beneficiary / Enrolment / Benefit; acknowledges receipt.
4. Ingest is **idempotent**.

### 2. Enrolment check (PBMS → IBR)

1. PBMS calls `POST /enrolled` with subject + programme code.
2. IBR returns **yes / no only**.

### 3. Search (partners → IBR)

DCI sync/async search over beneficiary / enrolment / benefit facts (depth depends on phase).

### 4. Analysis enrichment (IBR → Social Registry)

IBR (or analytics) queries SR for attributes not stored in IBR (PRS.IBR.04). Do not copy SR wholesale into IBR.

### 5. Staff read (option-dependent)

- **A / A′ / B:** query IBR APIs or IBR UI directly  
- **B+:** registry staff API/UI calls IBR by UIN / household id; registries do **not** own benefit facts  

## Trust and access (shared)

| Concern | Approach |
|---|---|
| Partner authentication | Partner credentials (Keycloak / Partner Management patterns) |
| Envelope integrity | Signed DCI envelopes where applicable |
| Data minimisation | `/enrolled` is boolean; IBR stores minimal PII |
| Audit | Who published which batch; who queried `/enrolled` |

## Non-goals (shared)

- Embedding IBR tables inside the PBMS database
- Making IBR the disbursement executor
- Mirroring full Social Registry records into IBR
- Requiring a single PBMS instance (IBR must accept multiple publishers)
- Using FR/NSR/DR as the **system of record** for benefits (see options brief — reject C as SoR)

## After Decision 0

When a product shape is chosen, update this page to describe **only** that option’s runtime components, layering, and deployment layout (and keep the options brief as historical decision record).
