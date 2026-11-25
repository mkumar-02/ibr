# Overview

## The problem

OpenG2P’s product stack today covers the **front** and **middle** of the social protection delivery chain well:

| Product | What it holds | What it answers |
|---|---|---|
| Social Registries (NSR, Farmer, Disability, …) | People and households who are *potentially eligible* | Who might receive benefits? |
| PBMS | Programme rules, enrolment lists, entitlements, disbursements | How does *this* programme run? |

What is missing is a **cross-programme ledger** of actual beneficiaries and benefits:

- How much has this person (or household) received, across all programmes?
- How many programmes are they enrolled in, and in what status?
- Are there unintended overlaps that affect eligibility for a new scheme?
- What is national or regional coverage by programme, benefit type, and target group?

Without IBR, that picture is reconstructed manually from siloed PBMS / SP-MIS instances — or not at all.

## What IBR is

An **Integrated Beneficiary Registry** consolidates data on benefits received by individuals or households across several programmes. Per DCI:

> The goal of an IBR is to provide a comprehensive picture of who is receiving which benefit from which programme.

IBR is both:

- **Operational** — eligibility checks against other programmes, coordination, overlap detection
- **Analytical** — coverage monitoring, policy planning, accountability

## What IBR is not

| Not this | Why |
|---|---|
| A Social Registry | SR holds *potential* beneficiaries; IBR holds people/households who have *received* benefits from at least one programme |
| A replacement for PBMS | PBMS remains the system of record for programme rules, cycles, and disbursement execution |
| A dump of all SR attributes | IBR minimises personal data; sensitive analysis fields are fetched from SR on demand (PRS.IBR.04) |

## Objectives (from DCI, mapped to OpenG2P)

1. Monitor and evaluate coverage across programmes, groups, and regions
2. Prevent and identify unintended enrolment overlaps
3. Support eligibility verification when enrolment in another programme is a criterion (inclusion or exclusion)
4. Facilitate coordination and transitions across programmes
5. Inform policy, programme adjustments, and strategic planning
6. Improve transparency and accountability of social protection resources

## Product positioning in OpenG2P

```
 Social Registries          PBMS (SP-MIS)              IBR
 (potential eligible)       (programme ops)     (cross-programme ledger)
 ───────────────────        ───────────────     ────────────────────────
 NSR / Farmer / DR    ───▶  eligibility        ◀── yes/no enrolment
                            entitlement             check
                            disbursement       ───▶ publish beneficiaries
                                                    + benefits
 IBR may query SR ◀──────────────────────────────── analysis attributes
 for analysis only                                  (not stored in IBR)
```

**Product shape is open:** IBR may be a Registry Gen2 manifestation, a standalone service, and/or a service that FR/NSR/DR **call for read**. Compare options in [Architecture options](10-architecture-options.md) before implementation.

Operating model we assume in all serious options: **partners (PBMS) write; staff mostly read.** Benefits should not become a second socio-economic store inside each social registry.

## In scope (documentation phase)

- Domain model aligned to DCI IBR data objects
- DCI process mapping (PRS.IBR.01–04)
- API surface IBR exposes and consumes
- Integration contracts with PBMS and Social Registries
- Architecture options for decision
- Phased delivery plan and open decisions

## Out of scope (for now)

- Application code, schemas as code, Helm charts
- Payment execution (stays in G2P Bridge / payment rails)
- Replacing per-programme beneficiary lists inside PBMS
- Storing full socio-economic profiles (that remains in Social Registry)

## Spec authority

Functional and interoperability behaviour is governed by:

- [DCI Integrated Beneficiary Registry v1.0.0](https://standards.spdci.org/standards/dci-standards/wip-integrated-beneficiary-registry-v1.0.0)
- Machine-readable schemas under [spdci/schemas `extensions/ibr/v1`](https://github.com/spdci/schemas/tree/main/extensions/ibr/v1)

OpenG2P may profile the standard (identifier types, programme codes, currencies) but should not invent incompatible payloads for the DCI Partner API surface.
