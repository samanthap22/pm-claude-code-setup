# PRD: InventoryInsight Autocorrect — EDEKA Phase 2

**Status:** Scoping  
**Author:** Samantha Pinches  
**Date:** 2026-05-12  
**Go-Live:** Monday, 13 April 2026  
**Stakeholders:** RI Product, RI Engineering, RI Security, EDEKA IT, EDEKA Accounting, Frank Ziegler (in-store validation)

---

## Problem

EDEKA stores carry persistent phantom and shadow stock — inflated or understated inventory counts — particularly in beverages, dairy alternatives, yoghurts, and chocolate. Today, corrections are manual, inconsistent across franchisees, and unscalable at 600-store volume. Phase 2 introduces automated nightly corrections that write directly into EDEKA's ERP, reducing store workload while meeting German financial audit standards.

---

## Goals

| Goal | Metric | Target |
|---|---|---|
| Correct phantom/shadow stock automatically | Perfect corrections | ≥50% |
| Directionally improve inventory | Directional accuracy | ≥30% |
| Limit bad corrections | False positives | ≤20% |
| Reduce manual store workload | Gap scan frequency | Declining trend post go-live |
| Audit compliance | Correction reversibility | 100% of corrections reversible |

**50-30-20 KPI framework** is the primary accuracy measure. False negatives are reviewed but ≤10% is not a Phase 2 target.

---

## Users

| User | What they need |
|---|---|
| Store staff (100 active stores) | Review overnight corrections each morning; reverse incorrect ones |
| EDEKA Accounting | New BWA movement type to categorise automated corrections in ERP and downstream systems |
| RI Data/ML team | Daily feeds from all 600 stores to build and validate correction models |
| Auditors | Full traceability of each correction — automatic vs. manual, reversed or applied |

---

## Store Scope

**100 Active Stores** — receive and can reverse automated nightly corrections. Final store list confirmed by EDEKA ~4 weeks before go-live.  
**500 Inactive Stores** — data supply only; no corrections issued.

Phased rollout: not all 100 active stores go live on Day 1. Go-live volume is capped to prevent store overload in week 1.

---

## Functional Requirements

### Automated Corrections

- RI generates nightly corrections for phantom and shadow stock.
- Corrections post autonomously into EDEKA's ERP via the existing interface — no Accept/Decline step.
- Store staff review corrections the following morning and may **reverse** any booking.
- RI receives reversal visibility via Internal Audit files.

**Correction logic by type:**

| Scenario | Action |
|---|---|
| Negative stock | Auto-correct to zero |
| Phantom stock — high confidence | Partial correction (e.g., 50% adjustment) |
| Phantom stock — medium confidence | Micro-correction (e.g., ±1 unit) |
| High-value discrepancy | Flagged for manager review; not automated |

> High-value item threshold: **definition still open** (see Open Questions).

### Store Workflow

Active stores must:
1. Review automated corrections each morning.
2. Reverse/cancel corrections as needed.
3. Continue separate stock corrections for routine operational reasons.
4. Perform **weekly control counts** against RI Inventory Insight lists.

No system-generated stock check lists (e.g., negative stock lists) will be provided.

### Oscillating Corrections

Repeated corrections on the same item are permitted. Accuracy validation uses the **last correction on the weekly control count day** as the reference point.

---

## Data Requirements

### Mandatory for Phase 2 (all 600 stores, daily)

- Store master data (RT1 & RT2)
- Item master data (RT1 & RT2)
- Sales data
- Inventory snapshot / on-hand quantity
- Stock correction history
- Internal Audit files (must include: automatic vs. manual flag, reversal indicator)
- Delivery quantity *(new feed for Phase 2 — required to reduce false positives in beverages)*

### Required for Accuracy (not blocking go-live)

- Promotions
- Store event calendars *(requested once store list is confirmed)*
- Store trait information *(SW region: full data; Northern Bavaria: limited — EDEKA will attempt manual enrichment)*

### Future Enhancements (not Phase 2)

Ordered quantity, in-transit quantity, case-pack size, shelf capacity — each requires a new API and would delay go-live if added now.

---

## Integration & IT Requirements

### Interface
Nightly corrections delivered through existing RI → EDEKA interface. No new channel required.

### Internal Audit File
Must be updated to include:
- Flag: automatic vs. manual correction
- Flag: reversal indicator

### New BWA Movement Type
EDEKA Accounting must create a new movement type for automated corrections and forward it to:
- Relex
- BI systems

### Data Latency for New Active Stores
Stores added late must have minimum 2–4 weeks of sales, inventory, and audit history before model can run. Late additions are possible but must meet this lead time.

### Mixed-Item Handling
Beverage crates and mixed packs require enhanced correction logic. Delivery quantity feed is the key input. This is a known complexity; additional engineering effort should be scoped separately.

---

## Security & Compliance

| Item | Current State | Required Action |
|---|---|---|
| SOC 2 coverage | Snowflake / Inventory Insight not currently in scope | Security review + updated questionnaire; meeting between RI Security and EDEKA Security |
| DR processes | Manual processes exist | Improvement planned; not blocking Phase 2 |
| SOC 2 roadmap | Planned for 2026 | Snowflake integration to SOC 2 underway |

**External messaging to EDEKA on SOC 2:** *"Snowflake is being integrated into our SOC 2 framework, and evidence generation is underway."*

**German audit compliance:** Corrections must be able to move stock up or down. Monitoring must prevent inventory undervaluation. Full reversibility is required.

---

## Rollout & Operational Requirements

| Milestone | Detail |
|---|---|
| Go-live | Monday, 13 April 2026 |
| Easter change freeze | EDEKA two-week freeze must be respected; no deploys during this window |
| Phased rollout | Not all 100 stores on Day 1; volume cap in go-live week |
| In-store validation | RI staff may conduct pre-go-live in-store counts; coordinate with Frank Ziegler (~3 weeks out) |
| Data room hygiene | EDEKA requests cleanup of inbound storage containers throughout project; EDEKA IT responsible for file visibility |

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Limited store trait data in Northern Bavaria | High | Medium | EDEKA manually enriching; model impact to be assessed at store list confirmation |
| Beverage/mixed crate logic underestimated | Medium | High | Scope separately; delivery quantity feed is key dependency |
| Correction volume spikes overwhelm stores at go-live | Medium | High | Volume cap in week 1; phased rollout |
| API additions (orders, transit, case-pack) delay timeline | Low (if held) | High | Explicitly deferred to post-Phase 2; do not scope now |
| Accuracy targets missed | Medium | High | 50-30-20 is ambitious; pre-go-live model validation against historical data required |
| No unanimous success criteria possible | Certain | Medium | EDEKA franchisees set their own standards; define RI-owned KPIs that don't depend on franchisee consensus |

---

## Open Questions

| # | Question | Owner | Priority |
|---|---|---|---|
| 1 | What is the threshold definition for "high-value items" requiring manager review? | RI + EDEKA | High — needed before correction logic is built |
| 2 | What granularity is required for correction reversal reporting in the Internal Audit file? | EDEKA IT | High — impacts interface spec |
| 3 | What is the exact time of nightly correction posting? | RI Engineering | Medium — determines store morning workflow |

---

## Next Steps

- [ ] EDEKA confirms 100-store list (~4 weeks before go-live)
- [ ] Resolve Open Questions 1–3 above
- [ ] EDEKA Accounting creates new BWA movement type and confirms Relex/BI forwarding
- [ ] RI Security + EDEKA Security meeting scheduled
- [ ] Delivery quantity API confirmed and ingestion scoped
- [ ] Internal Audit file spec updated with new flags
- [ ] Pre-go-live in-store counts scheduled with Frank Ziegler
- [ ] Go-live volume cap agreed with EDEKA ops
