# DCI processes

Source: [DCI IBR Process](https://standards.spdci.org/standards/dci-standards/wip-integrated-beneficiary-registry-v1.0.0/ibr/1.-crvs/1.2-process).

These use cases sit on the social protection delivery chain as follows:

| Stage | Use case |
|---|---|
| Enrolment | PRS.IBR.01 — verify enrolment in another scheme |
| Manage | PRS.IBR.02 — publish beneficiaries and benefits |
| Manage | PRS.IBR.03 — push enrolment status changes |
| Manage / Analyse | PRS.IBR.04 — IBR queries Social Registry for analysis |

---

## PRS.IBR.01 — Verify enrolment in a specific scheme

### Intent

SP-MIS needs to know whether a potential beneficiary (person or household) is currently enrolled in a **specific** programme — typically because that enrolment is an inclusion or exclusion criterion for a new scheme.

### Behaviour

- Initiator: SP-MIS (PBMS or other)
- Request: subject identifier(s) + programme code (individual or batch)
- Response: **yes or no only** — no benefit history, no PII dump
- One programme per request; additional programmes require additional calls

### OpenG2P mapping

| Actor | System |
|---|---|
| Requester | PBMS eligibility stage (or another SP-MIS Partner client) |
| Responder | IBR Partner API `POST /enrolled` (API.IBR.02) |

### Examples (from DCI)

- Disability Cash Transfer Scheme — check prior enrolment that would exclude or include
- Education Cash Transfer Scheme — verify enrolment conditions tied to another programme

### Acceptance criteria

- [ ] Boolean response only; no leakage of benefit amounts or other enrolments
- [ ] Supports individual and batch checks
- [ ] Works for person and household assistance units
- [ ] Partner auth + audit of who asked what

---

## PRS.IBR.02 — Publish beneficiaries and benefits received

### Intent

After benefits are provided (monthly, quarterly, ad hoc, …), SP-MIS publishes who received what so IBR can maintain the cross-programme ledger.

### Behaviour

- Trigger: benefit provision as part of the programme delivery cycle
- Unit: individual or household
- Benefit types: cash, vouchers, training, work opportunities, insurance, in-kind
- Transfer: **batch** for all beneficiaries receiving benefits at that time
- IBR confirms receipt

### OpenG2P mapping

| Actor | System |
|---|---|
| Publisher | PBMS after disbursement cycle completion (Celery / Bridge success) |
| Receiver | IBR `POST /enrollment/updates` (API.IBR.01) with benefit payloads |

Recommended PBMS hook: when a disbursement batch reaches a terminal “paid / delivered” state (or country-defined equivalent), emit an IBR publish job.

### Acceptance criteria

- [ ] Batch ingest of Beneficiary + Benefit facts
- [ ] Idempotent on publisher benefit / disbursement ids
- [ ] Acknowledgement of receipt to PBMS
- [ ] Creates Beneficiary if first-ever benefit; appends Benefit otherwise

---

## PRS.IBR.03 — Update enrolment status

### Intent

When a beneficiary’s enrolment status changes (suspend, graduate, reinstate, deceased, …), SP-MIS updates IBR so eligibility checks and analytics stay current.

### Behaviour

- Trigger: status change in SP-MIS
- Payload: subject + programme + new `enrolment_status` + `status_change_date`
- May travel on the same `/enrollment/updates` channel as PRS.IBR.02

### OpenG2P mapping

| Actor | System |
|---|---|
| Publisher | PBMS enrolment / exit / suspension workflows |
| Receiver | IBR `POST /enrollment/updates` (API.IBR.01) |

Status values follow CD.IBR.01 (Active, Deceased, Graduated, Suspended, Inactive, Reinstated, Awaiting Renewal).

### Acceptance criteria

- [ ] Status transitions recorded with timestamps
- [ ] `/enrolled` reflects current status (e.g. only Active — or Active+Reinstated — counts as “yes”; policy to confirm)
- [ ] History of status changes retained for audit / analysis

---

## PRS.IBR.04 — IBR queries Social Registry for analysis

### Intent

IBR (or analytics acting on its behalf) needs attributes that were deliberately **not** stored in IBR — income, education, employment, disability severity, etc. — for coverage and equity analysis.

### Behaviour

- Initiator: IBR / analytics
- Responder: Social Registry (NSR or domain registry)
- Sensitive attributes stay in SR; IBR does not become a second copy of SR
- Uses standard SR search (DCI COM search), not IBR-specific write APIs

### OpenG2P mapping

| Actor | System |
|---|---|
| Requester | IBR analytics job or staff analysis tool |
| Responder | NSR / Farmer / Disability Partner API search |

Related: Social Registry standards also describe SR updating IBR attributes in batch (PRS.SR.04 in SR↔SP-MIS docs). OpenG2P should treat **PBMS as the primary publisher** of enrolment/benefits; SR→IBR updates are secondary and must not invent benefit facts SR does not own.

### Acceptance criteria

- [ ] Analysis joins IBR enrolment/benefit facts with SR attributes without permanent duplication
- [ ] Consent / partner scopes enforced on SR queries
- [ ] Clear data-retention policy for any temporary analysis extracts

---

## Process ↔ API summary

| Process | Primary API | Direction |
|---|---|---|
| PRS.IBR.01 | API.IBR.02 `/enrolled` | SP-MIS → IBR |
| PRS.IBR.02 | API.IBR.01 `/enrollment/updates` | SP-MIS → IBR |
| PRS.IBR.03 | API.IBR.01 `/enrollment/updates` | SP-MIS → IBR |
| PRS.IBR.04 | API.COM.01 Search (on SR) | IBR → SR |
| Discovery / reporting | API.COM.01 Search (on IBR) | Client → IBR |
| Event-driven consumers | API.COM.02 / .03 Subscribe / Notify | Bidirectional patterns |
