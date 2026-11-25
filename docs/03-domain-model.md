# Domain model

IBR’s domain model maps DCI data objects onto durable entities (Beneficiary, Programme, Enrolment, Benefit, …). Field lists below are the **product intent**.

How those entities are packaged depends on [Decision 0](08-open-decisions.md#0-product-shape-where-ibr-lives):

- **A / A′** — Registry Gen2 registers/tables (`G2PRegister`, …)
- **B / B+** — service-owned tables (may still mirror the same logical model)

Exact column names are finalised at implementation.

## Principles

1. **Minimise PII** — enough to identify the assistance unit and join to SR/ID systems; not a second social registry.
2. **Assistance unit aware** — individual, family, or household can be the beneficiary unit (`CD.COM.13`).
3. **Programme-centric facts** — enrolment status and benefits are always tied to a programme.
4. **Event-friendly benefits** — each benefit receipt is a discrete, idempotent fact.
5. **DCI-aligned naming** — outbound Partner API payloads follow DCI JSON-LD shapes.

## Register hierarchy (proposed)

```
Programme (catalogue)                    Group (household / family)
        │                                         │
        │                                         ├── Member
        │                                         │
        └────────── Enrolment ◀───────────────────┤
                        │                         │
                        │                    Beneficiary (root)
                        │                         │
                        └── Benefit ──────────────┘
```

| Kind | Mnemonic (proposed) | Parent | Role |
|---|---|---|---|
| Register | `Beneficiary` | — | Root subject who has received ≥1 benefit |
| Register | `Programme` | — | Catalogue of programmes known to IBR |
| Register / Table | `Group` | Beneficiary (optional) | Household or family assistance unit |
| Table | `Member` | Group | Members of a family/household unit |
| Table | `Enrolment` | Beneficiary | Enrolment of a subject in a programme |
| Table | `Benefit` | Enrolment | Benefit events received under that enrolment |

> Exact Register vs Table classification will follow Registry platform conventions (functional ID, change-request behaviour). Likely: `Beneficiary` and `Programme` as primary registers; others as supporting tables or child registers.

## Mapping to DCI data objects

| DCI object | IBR entity | Notes |
|---|---|---|
| DO.IBR.04 Beneficiary | `Beneficiary` (+ nested enrolment/benefits) | Wire format may nest programme and benefits |
| DO.IBR.02 Programme | `Programme` | Also referenced from Enrolment |
| DO.IBR.03 Benefit | `Benefit` | Per receipt / cycle line |
| DO.IBR.O4 Benefit Frequency | Attribute on Programme or Enrolment | Periodicity of delivery |
| DO.IBR.01 Person | Fields on Beneficiary / Member | Minimised person attributes |
| DO.IBR.02 Member | `Member` | Individual in a group |
| DO.IBR.03 Group | `Group` | Family / household |

## Entity details

### Beneficiary

Assistance unit that has received benefits from at least one programme.

| Attribute | Source / type | Cardinality | Description |
|---|---|---|---|
| assistance_unit | CD.COM.13 | 0..1 | `person` \| `family` \| `household` |
| identifiers | DO.COM.01 | 1..* | Foundational / functional IDs (e.g. UIN) |
| name | DO.COM.02 | 0..1 | Minimal name for display / matching |
| place / address | DO.COM.07 / .03 | 0..1 | Location for coverage analysis |
| group | Group | 0..1 | When unit is family/household |
| member | Member | 0..1 | Nominated / head individual when unit is group |
| additional_attributes | array | 0..* | Profile-safe extensions |

DCI example (family assistance unit) nests `programme_identifier`, `enrolment_*`, and `benefits` on the Beneficiary payload. Internally IBR normalises those into Enrolment and Benefit rows.

### Programme

| Attribute | Cardinality | Description |
|---|---|---|
| programme_name | 1..1 | Display name |
| programme_identifier / programme_code | 1..1 | Stable code shared with PBMS |
| programme_description | 0..1 | Objectives / summary |
| implementing_institution | 0..1 | Responsible institution |
| institution_type | 0..1 | CD.IBR.03 |
| legal_status | 0..1 | CD.IBR.04 |
| social_protection_functions | 0..* | CD.IBR.05 |
| contribution_type | 0..1 | CD.IBR.06 |
| programme_type | 0..1 | CD.IBR.07 |
| benefit_frequency | 0..* | Delivery periodicity |

### Enrolment

Links a Beneficiary to a Programme with status history.

| Attribute | Cardinality | Description |
|---|---|---|
| beneficiary_id | 1..1 | Link to Beneficiary |
| programme_id | 1..1 | Link to Programme |
| enrolment_date | 0..1 | When enrolled |
| enrolment_status | 0..1 | CD.IBR.01 |
| status_change_date | 0..* | Dates of status transitions |
| source_system | 0..1 | Publishing SP-MIS / PBMS instance id |
| external_enrolment_id | 0..1 | Idempotency / correlation key from publisher |

#### Enrolment status (CD.IBR.01)

| Code | Value | Meaning |
|---|---|---|
| 1 | Active | Currently receiving benefits |
| 2 | Deceased | Beneficiary deceased |
| 3 | Graduated | Met graduation criteria |
| 4 | Suspended | Suspended from the programme |
| 5 | Inactive | Temporarily not receiving; not formally suspended/exited |
| 6 | Reinstated | Returned after suspension/exit |
| 7 | Awaiting Renewal | Renewal / reassessment in progress |

### Benefit

| Attribute | Cardinality | Description |
|---|---|---|
| enrolment_id | 1..1 | Parent enrolment |
| benefit_type | 0..1 | CD.IBR.02 (cash, voucher, in-kind, …) |
| benefit_date / date_received | 0..1 | When received |
| benefit_value / benefit_amount | 0..1 | Monetary value |
| currency | 0..1 | CD.COM.06 / DF.COM.STRING.08 |
| benefit_description / benefit_name | 0..1 | Label or in-kind detail |
| benefit_quantity | 0..1 | Quantity for in-kind |
| external_benefit_id | 0..1 | Idempotency key from PBMS (e.g. disbursement line id) |

### Group and Member

Used when `assistance_unit` is family or household.

- **Group** — group identifiers, group_type, address
- **Member** — person identifiers/name, relationship to head, nominated beneficiary flag

## Code directories (DCI)

| Code directory | Purpose |
|---|---|
| CD.IBR.01 enrolment_status | Enrolment lifecycle |
| CD.IBR.02 benefit_type | Cash, voucher, in-kind, training, … |
| CD.IBR.03 institution_type | Implementing institution classification |
| CD.IBR.04 legal_status | Anchored in legislation or not |
| CD.IBR.05 social_protection_functions | Function tags |
| CD.IBR.06 contribution_type | Contributory / non-contributory |
| CD.IBR.07 programme_type | Support mechanism |
| CD.COM.13 assistance_unit | person / family / household |
| CD.COM.06 currency | Currency codes |

OpenG2P Master Data datasets should back coded fields where the platform already uses that pattern.

## Queries the model must support

1. All enrolments for a given identifier (UIN / household id)
2. Benefits received by programme and date range (sum and detail)
3. Is subject enrolled in programme X? (boolean)
4. Subjects enrolled in both programme A and B (overlap)
5. Coverage aggregates by geography, programme, benefit type, status

## What we deliberately do not store

- Full Proxy Means Test / poverty scores (live in SR)
- Detailed disability assessments (live in Disability Registry)
- Payment instrument / bank account details (live in PBMS / Bridge)
- Programme rule definitions and entitlement formulas (live in PBMS)

Those may be **queried** for analysis (PRS.IBR.04) but are not copied into IBR as system of record.
