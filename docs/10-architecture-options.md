# Architecture options (decision brief)

**Status:** Open — choose one product shape before Phase 1.

This document captures every option discussed so far. Common facts (DCI processes, domain model, PBMS publish path) stay the same; **where IBR lives and who operates it** differ.

Related: [Architecture](02-architecture.md) (shared context) · [Open decisions](08-open-decisions.md) (Decision 0).

---

## Shared operating-model insight

IBR is **not** operated like NSR / Farmer / Disability Registry:

| Actor | Typical IBR action |
|---|---|
| **Partners (PBMS / SP-MIS)** | Write — publish enrolments and benefits; call `/enrolled` |
| **Registry / programme staff** | Read — see programmes & benefits for a person/household; investigate overlap |
| **Field intake staff** | Do **not** register people *into* IBR via forms |

Any option that assumes full intake + change-request culture for every benefit line is probably the wrong UX fit — even if Registry Gen2 is reused under the hood.

---

## Options at a glance

| ID | Name | IBR SoR location | Staff experience | Cross-programme |
|---|---|---|---|---|
| **A** | Full Registry Gen2 manifestation | Own registry product | Full staff portal (like NSR) | Strong |
| **A′** | Lean Registry manifestation | Own registry product | Read-heavy portal; partner auto-ingest | Strong |
| **B** | Standalone IBR service | Own service + DB | Minimal / separate UI | Strong |
| **B+** | Standalone IBR + registry clients | Own service + DB | Programmes panel **inside** FR/NSR/DR (live read) | Strong |
| **C** | Programmes tag inside FR/NSR/DR | Inside each social registry | Edit/view on SR record | Weak / fragmented |

```
                    ┌─────────────────────────────────────┐
                    │     Who is system of record?        │
                    └─────────────────────────────────────┘
                                      │
              ┌───────────────────────┼───────────────────────┐
              │                       │                       │
              ▼                       ▼                       ▼
     Separate IBR product      Separate IBR service    Inside SR (FR/NSR/DR)
         A / A′                      B / B+                    C
              │                       │
              │              ┌────────┴────────┐
              │              │                 │
              │              ▼                 ▼
              │         Standalone UI    Used by extensions
              │              B                 B+
```

---

## Option A — Full Registry Gen2 manifestation

### Idea

IBR is packaged like Farmer / NSR: `ibr-extension` + platform runtimes + Helm. Full registry capabilities (intake, change requests, staff portal, Partner API, search).

### Diagram

```
PBMS ──▶ IBR (Registry manifestation) ◀── Staff Portal (full)
                │
                └── DCI Partner APIs
```

### Pros

- Maximum reuse of Registry Gen2 (DCI search, consent, encryption, packaging)
- Same ops model as other OpenG2P registries
- Natural home for full DCI COM search / subscribe

### Cons

- Heavier than the operating model needs (intake/CR may go unused)
- Staff may think they “register” beneficiaries into IBR
- Longer time-to-first-value

### Best if

You want IBR to be a first-class registry product with the same governance UX as NSR, and expect staff corrections/workflows on IBR records.

---

## Option A′ — Lean Registry manifestation

### Idea

Same deployable as A, but **product behaviour** is partner-fed:

- Trusted partner ingest auto-approves (no CR per benefit line)
- No field intake product
- Staff portal = search / history / overlap only

### Pros

- Reuses Partner API + DCI stack without pretending IBR is a social registry
- Still one national ledger
- Fits OpenG2P packaging conventions

### Cons

- Still carries Registry platform weight (many components)
- Risk of drifting back into “full registry” UX unless discipline holds
- Change-request / metadata model may feel awkward for pure ledger facts

### Best if

You want OpenG2P consistency and DCI reuse, but agree staff do not operate IBR like NSR.

---

## Option B — Standalone IBR service

### Idea

Dedicated service (FastAPI + Postgres ± workers). Implements DCI IBR APIs. Not a Registry Gen2 manifestation. Own Helm chart. Optional thin admin UI.

### Diagram

```
PBMS ──▶ IBR service ◀── partners / analytics
              │
         (optional thin UI)
```

### Pros

- Matches partner-write / staff-read cleanly
- Smaller conceptual surface; faster to reason about
- No unused intake/CR baggage

### Cons

- May reimplement partner auth, signing, DCI envelopes, consent
- Diverges from registry-platform patterns unless libraries are shared
- Staff have no natural place to see benefits unless a new UI is built

### Variants

| | |
|---|---|
| **B1** | Fully standalone stack |
| **B2** | Standalone data plane; reuse registry-platform Partner/DCI libraries |

### Best if

Phase 1 scope is mainly `/enrollment/updates` + `/enrolled` + internal queries, and you accept a new service type in the platform.

---

## Option B+ — Standalone IBR service + used by registry extensions _(strong candidate)_

### Idea

**IBR service is the only system of record** for enrolments/benefits.  
**FR / NSR / DR extensions do not store benefit facts** — they **call IBR** to show a “Programmes & benefits” panel (and similar) in staff UI.

```
                    ┌──────────────┐
   PBMS / SP-MIS ──▶│  IBR service │◀── other partners (/enrolled, search)
                    │  (SoR)       │
                    └──────▲───────┘
                           │ read by id (UIN / household)
           ┌───────────────┼───────────────┐
           │               │               │
     ┌─────┴─────┐   ┌─────┴─────┐   ┌─────┴─────┐
     │ NSR staff │   │ FR staff  │   │ DR staff  │
     │  UI panel │   │  UI panel │   │  UI panel │
     └───────────┘   └───────────┘   └───────────┘
```

### What “use in extensions” means

| Do | Don’t |
|---|---|
| Live (or short-TTL cache) read from IBR by shared identifier | Persist authoritative benefit amounts in NSR/FR/DR tables |
| Shared client library e.g. `openg2p-ibr-client` | Embed IBR DB inside each registry |
| Degrade gracefully if IBR is down | Make registries publishers of benefit facts |

### Pros

- Correct SoR (one ledger) + familiar staff UX (inside registries they already use)
- Fits partner-write / staff-read
- Cross-programme works across FR + NSR + DR
- Aligns with DCI “IBR ≠ Social Registry”
- Lighter than A; more usable for staff than bare B

### Cons

- Extra deployable to run and monitor
- Identity join contract must be strict (same UIN / household id)
- Staff UX depends on IBR availability
- Extension teams must not “accidentally” invent local programmes tables as SoR
- Still need a plan for DCI library reuse (prefer B2 under the hood)

### Best if

You want a dedicated ledger service **and** staff should see benefits in context of the person/farmer/PWD they already manage.

---

## Option C — Programmes / benefits tag inside FR, NSR, DR

### Idea

Add a programmes (or benefits) section on each social registry record; PBMS updates that section.

### Diagram

```
PBMS ──▶ NSR programmes tag
PBMS ──▶ FR programmes tag
PBMS ──▶ DR programmes tag
```

### Pros

- No new deployable
- Staff see programmes on the record they already open
- Fast to prototype for a **single** registry

### Cons

- **Wrong SoR** vs DCI (SR = potential eligible; IBR = received benefits)
- **Fragmented** across FR / NSR / DR — where does a multi-programme person live?
- Cross-programme overlap queries are hard or incomplete
- People can receive benefits without existing in every SR
- Pulls payment/benefit history into registries that should stay domain-focused

### Softened form (acceptable later, not as SoR)

A “Programmes” **tab that only displays IBR data** (that is Option **B+**, not C).  
C means the **data lives in** the social registry.

### Best if

Never as the long-term design — only as a short demo that must be replaced. Prefer B+ for the same UX with correct SoR.

---

## Comparison matrix

| Criterion | A | A′ | B | B+ | C |
|---|---|---|---|---|---|
| Partner-write / staff-read fit | Medium | High | High | **High** | Medium |
| Cross-programme single ledger | High | High | High | **High** | Low |
| DCI IBR API clarity | High | High | High | High | Low |
| Reuse Registry Gen2 | **High** | High | Low–Med | Low–Med | N/A (wrong place) |
| Staff UX in FR/NSR/DR context | Low* | Low* | Low | **High** | High (but wrong SoR) |
| Build / ops complexity | High | Med–High | Med | Med–High | Low short-term |
| Risk of becoming second SR | Med | Low | Low | Low | **High** |
| Time to first PBMS→ledger slice | Slower | Medium | **Faster** | Medium | Fast but debt |

\*Unless you also add registry clients (which turns A/A′ toward a hybrid with B+).

---

## Decision criteria (use these in review)

Score 1–5 each; weight what matters for OpenG2P now:

1. **Operating model** — partners write; staff mostly read  
2. **Cross-registry coverage** — one person across NSR + FR + DR  
3. **DCI conformance** — `/enrollment/updates`, `/enrolled`, search  
4. **Reuse vs weight** — registry-platform libraries without unused intake  
5. **Staff discoverability** — see benefits without a separate “IBR app”  
6. **Time to Phase 1** — PBMS cycle visible in ledger  
7. **Governance clarity** — who owns benefit facts?

---

## Straw-man recommendation (not decided)

For debate only:

> Prefer **B+** (standalone IBR service as SoR, FR/NSR/DR as read clients), implemented with **B2** (reuse Partner/DCI libraries where practical).  
> Reject **C** as system of record.  
> Keep **A′** as fallback if the org insists IBR must ship as a Registry manifestation.

**Final choice:** record in [Open decisions — Decision 0](08-open-decisions.md).

---

## What does *not* change across options

Regardless of A / A′ / B / B+ (and rejecting C as SoR):

- PBMS remains programme ops SoR and the primary **publisher** into IBR  
- Domain facts: Beneficiary, Programme, Enrolment, Benefit ([domain model](03-domain-model.md))  
- DCI processes PRS.IBR.01–04 ([processes](04-dci-processes.md))  
- APIs `/enrollment/updates`, `/enrolled`, search ([apis](05-apis.md))  
- IBR does not execute payments; does not replace PBMS lists  
- IBR does not permanently mirror full SR socio-economic profiles  

---

## Review checklist

- [ ] Product / architecture review of this brief  
- [ ] PBMS team agrees publish hooks  
- [ ] Registry (NSR/FR/DR) team agrees client-only integration if B+  
- [ ] Decision 0 recorded with date and owner  
- [ ] [02-architecture.md](02-architecture.md) updated to describe the **chosen** option only  
- [ ] Roadmap Phase 1 deliverables adjusted to match choice  
