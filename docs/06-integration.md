# Integration

How IBR connects to PBMS, Social Registries, and partner systems.

## Integration map

```
┌────────────────────┐         ┌────────────────────┐
│  Social Registries │         │   Partner systems  │
│  NSR / Farmer / DR │         │  other SP-MIS, BI  │
└─────────┬──────────┘         └─────────▲──────────┘
          │ analysis (IBR.04)            │ search / subscribe
          │ + staff read (if B+)         │
          ▼                              │
┌───────────────────────────────────────────────────┐
│                        IBR                        │
│              (enrolment / benefit SoR)            │
└───────────────────────▲───────────────────────────┘
                        │ /enrollment/updates
                        │ /enrolled
┌───────────────────────┴───────────────────────────┐
│  PBMS (primary SP-MIS)  ·  other SP-MIS publishers│
└───────────────────────────────────────────────────┘
```

How registries attach depends on [Decision 0](08-open-decisions.md#0-product-shape-where-ibr-lives) — see [Architecture options](10-architecture-options.md).

## PBMS → IBR (primary write path)

PBMS is the main OpenG2P **publisher**. IBR never invents benefit facts; it records what SP-MIS reports.

### When to publish

| Event in PBMS | DCI process | Payload focus |
|---|---|---|
| Disbursement / delivery cycle completed | PRS.IBR.02 | Beneficiary + benefits |
| Enrolment activated | PRS.IBR.03 | Status `Active` |
| Suspension / exit / graduation / death | PRS.IBR.03 | New status + date |
| Reinstatement / renewal | PRS.IBR.03 | New status + date |

### Suggested PBMS-side components (future implementation)

1. **IBR publisher adapter** — maps PBMS enrolment + disbursement rows → DCI `Beneficiary` JSON-LD
2. **Outbox / bg-task** — reliable delivery with retries
3. **Correlation ids** — `external_enrolment_id`, `external_benefit_id` (e.g. disbursement line id)
4. **Programme code registry** — shared codes between PBMS programme config and IBR Programme catalogue

### Configuration PBMS needs

- IBR Partner API base URL
- Partner credentials / signing keys
- Programme code mapping
- Which terminal disbursement states trigger publish
- Which enrolment statuses map to CD.IBR.01

### Failure and replay

- Retries with idempotent keys
- Dead-letter visibility for operators
- Ability to re-publish a cycle without duplicating benefits

## PBMS → IBR (enrolment check)

During eligibility list generation (or interactive enrolment), PBMS may call `/enrolled` when rules depend on other-programme membership.

| Concern | Guidance |
|---|---|
| Latency | Prefer batch `/enrolled` for list generation, not per-row sync in tight loops |
| Caching | Short-lived cache optional; status can change — document TTL |
| Rule DSL | Eligibility rules reference programme codes IBR understands |

## IBR → Social Registry (analysis)

For PRS.IBR.04:

1. Analytics or IBR job identifies subjects of interest from IBR (e.g. all Active in programme X).
2. Job searches SR Partner API for allowed attributes (income, disability, education, …).
3. Join happens in the analysis layer; durable IBR tables stay enrolment/benefit focused.

Do **not** bulk-copy SR socio-economic fields into Beneficiary as a standing store.

## Social Registries → IBR (staff read — Option B+)

If Decision 0 = **B+**, FR / NSR / DR **consume** IBR; they do not own benefit facts.

| Piece | Responsibility |
|---|---|
| `openg2p-ibr-client` (proposed) | Shared HTTP client used by registry staff APIs / UI backends |
| Staff UI panel | “Programmes & benefits” on Individual / Farmer / PWD / Household |
| Join key | Same UIN / household functional id as IBR |
| Failure mode | Panel shows unavailable / empty; registry core flows still work |

**Rules:**

- Registries must **not** add authoritative `programmes` / `benefits` tables as SoR
- PBMS publishes to **IBR only**, not into each registry’s domain tables
- A display-only cache in the registry API is optional (Decision 9); it must stay non-authoritative

If Decision 0 = **A / A′ / B**, staff read IBR via IBR’s own UI/API instead (or as well).

## Other SP-MIS → IBR

IBR must accept publishers beyond OpenG2P PBMS (departmental systems, legacy SP-MIS):

- Onboard each publisher as a Partner
- Assign partner-scoped write permissions to `/enrollment/updates`
- Tag every Enrolment/Benefit with `source_system`
- Resolve identity via agreed identifier types (UIN, etc.)

## Partners reading IBR

| Consumer | Typical APIs |
|---|---|
| Another programme’s SP-MIS | `/enrolled`, optional search |
| National M&E / BI | Search, subscriptions, bulk extract under policy |
| Policy / planning tools | Aggregates (may be via analytics warehouse fed from IBR) |

Access controlled by Partner Management, scopes, and consent where person-level data is returned.

## Identity and matching

| Topic | Proposal |
|---|---|
| Primary person key | Foundational ID / UIN as used across OpenG2P registries |
| Household key | Household functional id from NSR (or country equivalent) |
| Assistance unit | Explicit on each Beneficiary / Enrolment |
| Conflicts | Same UIN, conflicting demographics → quarantine for staff resolution; do not merge silently |

## Security checklist

- [ ] mTLS or equivalent for Partner API in production profiles
- [ ] Signed DCI envelopes
- [ ] Least-privilege partner scopes (write updates ≠ full search)
- [ ] Audit: who published what batch; who queried `/enrolled`
- [ ] PII minimisation in logs and analytics extracts
- [ ] Retention policy for raw inbound payloads

## What not to integrate (baseline)

| Anti-pattern | Why |
|---|---|
| IBR pulls live from Bridge payment APIs | Bridge is payment execution; PBMS already owns disbursement state |
| Bidirectional sync of full beneficiary lists PBMS ↔ IBR | Diverges systems of record; use push of facts only |
| Embedding IBR DB views inside PBMS Odoo | Couples deployables; use APIs |
| Storing benefit SoR inside FR/NSR/DR | Fragments the ledger; breaks DCI IBR vs SR split (Option C) |
