# Open decisions

Resolve these before Phase 1 implementation. Record the chosen option and date in the decision log when decided.

**Start here:** [Decision 0 — Product shape](#0-product-shape-where-ibr-lives) — full comparison in [Architecture options](10-architecture-options.md).

---

## 0. Product shape (where IBR lives)

| Option | Summary |
|---|---|
| **A** | Full Registry Gen2 manifestation (like NSR/Farmer, full staff UX) |
| **A′** | Lean Registry manifestation (partner-fed; staff read-only portal) |
| **B** | Standalone IBR service (own API + DB; optional thin UI) |
| **B+** | Standalone IBR service **+** FR/NSR/DR extensions **read** it (panels/clients) |
| **C** | Programmes/benefits stored **inside** FR/NSR/DR (not recommended as SoR) |

| Criterion | Favours |
|---|---|
| Partner-write / staff-read | A′, B, B+ |
| Staff see benefits inside existing registries | B+, C (C wrong SoR) |
| Max Registry Gen2 reuse | A, A′ |
| Fastest thin ledger | B |
| Cross-programme single ledger | A, A′, B, B+ (not C) |
| DCI clarity | A, A′, B, B+ (not C) |

**Straw-man for debate:** B+ (with B2 library reuse). **Not decided.**

**Status:** Open — see [10-architecture-options.md](10-architecture-options.md).

---

## 1. Deployment topology

| Option | Pros | Cons |
|---|---|---|
| **A. One national IBR** | Matches DCI; true cross-programme view | Multi-ministry governance |
| B. Per-ministry IBR instances | Local ownership | Weak overlap unless federated |

**Status:** Open — default lean **A** (one national ledger). Applies once Decision 0 ≠ C.

---

## 2. Assistance unit primary keys

| Option | Pros | Cons |
|---|---|---|
| **A. Individual UIN + optional household id** | Fits person programmes and NSR links | HH-only programmes need care |
| B. Household-only root | Simple for HH cash | Awkward for individual schemes |
| C. Dual primary registers (Person + Household) | Flexible | More complex Enrolment linking |

**Status:** Open — default lean **A**. Critical for B+ identity join across registries.

---

## 3. Who may publish in Phase 1?

| Option | Pros | Cons |
|---|---|---|
| **A. OpenG2P PBMS only** | Fastest; controlled mapping | Delays multi-SP-MIS proof |
| B. PBMS + one external stub publisher | Proves Partner write early | Slightly more work |

**Status:** Open — default lean **A**, design auth for **B**.

---

## 4. Benefit grain

| Option | Pros | Cons |
|---|---|---|
| **A. Every disbursement line** | Full history; accurate sums | Higher volume |
| B. Per-cycle summary per beneficiary | Smaller | Less audit detail |
| C. Configurable per programme | Flexible | Harder to reason about |

**Status:** Open — default lean **A** for cash.

---

## 5. What counts as “enrolled” for `/enrolled`?

| Option | Meaning of “yes” |
|---|---|
| **A. Active + Reinstated** | Currently in-benefit |
| B. Active + Reinstated + Awaiting Renewal | Includes renewal pipeline |
| C. Any status except Deceased/Graduated | Broader; may block wrongly |

**Status:** Open — default lean **A**; optionally configurable later.

---

## 6. Programme catalogue ownership

| Option | Pros | Cons |
|---|---|---|
| **A. IBR catalogues programmes; PBMS syncs codes** | Validate on ingest | Sync job |
| B. Auto-create Programme on first ingest | Zero friction | Thin metadata |
| C. Shared Master Data service | Single source | Extra dependency |

**Status:** Open — default lean **B** for Phase 1, tighten to **A** later.

---

## 7. Write-path governance (ingest)

Depends partly on Decision 0.

| Option | Pros | Cons |
|---|---|---|
| **A. Trusted Partner ingest auto-applies** | Fits M2M DCI | Weaker human gate on write |
| B. All updates as change requests | Full registry governance | Throughput risk (mainly A/A′) |
| C. Auto-apply writes; staff CR only for corrections | Balance | Two paths |

**Status:** Open — default lean **A** or **C**; **B** only if Decision 0 = A.

---

## 8. Repository / packaging home

| Option | Notes |
|---|---|
| **A. This repo = Registry manifestation** (`ibr-extension` + Helm) | Matches Decision 0 = A / A′ |
| **B. This repo = standalone IBR service** (+ optional `ibr-client`) | Matches Decision 0 = B / B+ |
| C. GitLab under `openg2p/…` when productised | Likely eventual org home |

**Status:** Open — blocked on Decision 0.

---

## 9. How registries consume IBR (only if Decision 0 = B+)

| Option | Pros | Cons |
|---|---|---|
| **A. Live API read** on staff UI open | Always fresh | Latency / availability coupling |
| B. Short-TTL cache in registry API | Smoother UX | Stale window |
| C. Async projection table in SR (non-authoritative) | Fast UI | Sync complexity; must never become SoR |

**Status:** Open — default lean **A** or **B**; avoid **C** unless clearly marked non-authoritative.

---

## Decision log

| ID | Decision | Choice | Date | Decided by |
|---|---|---|---|---|
| 0 | Product shape | — | — | — |
| 1 | Deployment topology | — | — | — |
| 2 | Assistance unit keys | — | — | — |
| 3 | Phase 1 publishers | — | — | — |
| 4 | Benefit grain | — | — | — |
| 5 | `/enrolled` yes policy | — | — | — |
| 6 | Programme catalogue | — | — | — |
| 7 | Write-path governance | — | — | — |
| 8 | Repo / packaging | — | — | — |
| 9 | Registry consume mode | — | — | — |

When decided, fill the row and mark the section **Status: Decided**.
