# APIs

IBR’s external API surface follows DCI common methods plus IBR-specific endpoints. Normative definitions live in:

- [DCI IBR API section](https://standards.spdci.org/standards/dci-standards/wip-integrated-beneficiary-registry-v1.0.0/ibr/1.-crvs/api)
- [spdci/standards](https://github.com/spdci/standards) (`ibr_api_v1.0.0.yaml` when published in release/src)
- [spdci/schemas `extensions/ibr/v1`](https://github.com/spdci/schemas/tree/main/extensions/ibr/v1)

OpenG2P implements these on an IBR-facing Partner/API surface. Packaging depends on [Decision 0](08-open-decisions.md#0-product-shape-where-ibr-lives): Registry Partner API runtime (A/A′) or standalone IBR service routes (B/B+), optionally reusing registry-platform DCI libraries.

## Endpoints IBR implements

### API.IBR.01 — SYN Enrolment / updates

| | |
|---|---|
| Path | `POST /enrollment/updates` |
| Implemented by | IBR |
| Called by | SP-MIS / PBMS |
| Purpose | Receive enrolment updates and benefit publications |

**Supports:** PRS.IBR.02 (benefits), PRS.IBR.03 (status changes).

**Payload (conceptual):** DCI `Beneficiary` objects including:

- assistance unit / group / member
- `programme_identifier`
- `enrolment_date`, `enrolment_status`, `status_change_date`
- `benefits[]` (type, amount, currency, date, quantity, description)

**Response:** acknowledgement of receipt (success / partial / error per DCI common patterns).

**OpenG2P notes:**

- Prefer batch requests aligned to PBMS disbursement cycles
- Require idempotency keys (`external_benefit_id`, `external_enrolment_id`)
- Validate programme codes against Programme catalogue (or auto-register with provenance)
- Persist raw envelope reference in object storage for replay/audit when useful

---

### API.IBR.02 — SYN Enrolled

| | |
|---|---|
| Path | `POST /enrolled` |
| Implemented by | IBR |
| Called by | SP-MIS / PBMS |
| Purpose | Answer whether a subject is enrolled in a specific programme |

**Supports:** PRS.IBR.01.

**Request (conceptual):** subject identifier(s) + programme code (+ optional as-of date).

**Response:** yes / no only.

**OpenG2P notes:**

- Define which statuses count as “enrolled” (default proposal: `Active` and `Reinstated`; confirm in [open decisions](08-open-decisions.md))
- Support batch checks for eligibility list generation
- Log requester partner id for audit without storing unnecessary PII in logs

---

### API.COM.01 — SYN / ASY Search

| | |
|---|---|
| Paths (OpenG2P pattern) | `POST /dci/registry/sync/search`, `POST /dci/registry/async/search` |
| Implemented by | IBR Partner API |
| Called by | Partners, analytics, other SP systems |
| Purpose | Search beneficiary / enrolment / benefit records |

Same envelope, signing, and consent patterns as existing Registry DCI search (see OpenG2P Partner Register Search docs).

**Sync:** return signed response envelope.  
**Async:** acknowledge, then publish `on-search` via WebSub.

**Searchable entities (planned):** Beneficiary, Enrolment, Benefit, Programme (via allowlisted fields and outbound templates).

---

### API.COM.02 — Event Subscription

Subscribe to IBR change topics (e.g. enrolment status changes, new benefits) via WebSub / DCI subscription methods already used by Registry.

### API.COM.03 — Event Notification

Notify subscribers when subscribed events occur (outgestion pipeline).

### API.COM.04 — SYN / ASY Status

Poll or async-complete status of long-running search / ingest operations.

---

## Endpoints IBR consumes

| Target | API | Process |
|---|---|---|
| Social Registry Partner API | API.COM.01 Search | PRS.IBR.04 |
| (Optional) Disability / Farmer registries | Domain search | Analysis enrichment |

IBR does **not** call PBMS to pull benefits in the baseline design; PBMS **pushes** to IBR. A pull-based reconciler may be added later as an operational tool, not as the primary DCI path.

---

## Staff and Beneficiary portals

Not part of the DCI standard, but expected product surfaces:

| API | Audience | Initial scope |
|---|---|---|
| Staff Portal API | Programme / registry staff | Search beneficiary, list enrolments, benefit history, overlap views |
| Beneficiary Portal API | Residents | Own enrolments and benefits (later phase; consent-aware) |

These reuse Registry staff / bene portal patterns; they are not substitutes for the Partner DCI APIs.

---

## Error handling

Follow DCI common exception handling and API error code directories:

- CD.COM.API.01 common errors
- CD.COM.API.02 search / on-search
- CD.COM.API.03 subscribe / unsubscribe
- CD.COM.API.04 status / on-status

IBR-specific cases to document at implementation:

| Case | Expected behaviour |
|---|---|
| Unknown programme code | Reject or quarantine with clear error; do not silently invent programme metadata |
| Duplicate benefit id | Idempotent success (no double-count) |
| Subject not found on `/enrolled` | `no` (not an error), unless identifier format invalid |
| Partial batch failure | Per-item errors + overall acknowledgement semantics per DCI |

---

## Versioning

- Align Partner routes with DCI versioning practices used by Registry
- Pin consumed JSON-LD schemas to a tagged `extensions/ibr/v1` revision
- Document OpenG2P profile deviations (if any) in this repo under `docs/` — prefer zero deviations
