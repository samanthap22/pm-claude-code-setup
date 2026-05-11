# PRD: InventoryInsight Autocorrect

**Status:** Discovery  
**Author:** Samantha Pinches  
**Date:** 2026-05-11  
**Stakeholders:** Store/Warehouse Ops, Merchandising, Data/Analytics, Engineering

---

## Problem

Inventory count discrepancies are a persistent operational cost. When physical counts diverge from system-of-record values, teams currently resolve them manually — a slow, inconsistent process that scales poorly across locations and SKUs. Left unresolved, discrepancies distort demand signals, trigger incorrect reorders, and erode trust in the inventory system.

**Why now:** As InventoryInsight expands to more locations, manual reconciliation is becoming a bottleneck and a source of avoidable shrink.

---

## Goal

Reduce the time and manual effort required to resolve inventory count discrepancies, while maintaining auditability and user trust in system-generated corrections.

---

## Users

| User | Job to be done | Pain today |
|---|---|---|
| Store / warehouse ops staff | Keep counts accurate without constant manual correction | Logging discrepancies is tedious; corrections take multiple steps |
| Retail buyers / merchandisers | Trust inventory data when making replenishment decisions | Stale or wrong counts lead to over/under-ordering |
| Internal analysts / data team | Audit inventory health and spot systemic issues | Hard to distinguish legitimate variance from data errors |

---

## Proposed Direction

InventoryInsight Autocorrect detects count discrepancies that fall within a defined confidence threshold and either:
1. **Auto-applies** the correction with a logged rationale (high-confidence cases), or  
2. **Surfaces a suggested correction** for a human to approve (lower-confidence or high-value SKUs)

The system uses existing variance signals (cycle count history, vendor tolerance bands, loss/shrink benchmarks) to classify discrepancy severity and confidence.

This is **not** a black-box fix — every correction is logged, reversible, and explainable.

---

## Success Metrics

| Metric | Baseline (today) | Target |
|---|---|---|
| Avg time to resolve a discrepancy | TBD | 50% reduction |
| % of discrepancies resolved without manual intervention | ~0% | 60%+ (auto-applied) |
| Data team audit escalations | TBD | No increase (trust maintained) |
| Buyer confidence in inventory data | Qualitative / survey | Measurable improvement at 90-day mark |

*Note: Baselines need instrumentation before we can finalize targets.*

---

## Scope (Discovery — not committed)

**In scope for this initiative:**
- Discrepancy detection logic (threshold-based)
- Auto-correction for high-confidence cases with audit log
- Suggested-correction UI for lower-confidence cases
- Rollback / manual override for all auto-corrections

**Out of scope (explicitly):**
- Predictive inventory adjustments based on demand signals (separate initiative)
- Integration with external WMS or ERP systems in v1
- Real-time RFID or IoT count feeds

---

## Open Questions

1. What confidence threshold separates auto-apply from suggest-and-review? Who owns this parameter — ops, data, or product?
2. Are there regulatory or audit requirements (e.g., SOX, loss prevention) that constrain what can be auto-corrected without a human in the loop?
3. What is the source of truth for "expected" count — last cycle count, system perpetual inventory, or vendor invoice?
4. How do we handle high-value or high-risk SKUs where even a small discrepancy warrants human review?
5. Do ops staff need to act in the UI, or does this run as a background job they see in a report?

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Auto-corrections erode ops trust if they get it wrong | Medium | High | Start with suggest-only mode; build trust before enabling auto-apply |
| Audit/compliance concerns block auto-write to inventory | Medium | High | Validate with legal/finance early; design for full auditability from day one |
| Discrepancy signals are too noisy for meaningful thresholds | Medium | Medium | Data team analysis of historical variance before engineering starts |
| Ops staff bypass the feature and keep doing it manually | Low | Medium | Co-design review experience with ops to ensure it's faster than status quo |

---

## Next Steps

- [ ] Align stakeholders on problem framing and scope (this doc)
- [ ] Instrument discrepancy resolution flow to establish baselines
- [ ] Data team: analyze historical variance patterns to validate threshold feasibility
- [ ] Legal/compliance review: confirm auto-write permissions
- [ ] Schedule co-design session with ops staff to validate UX direction
- [ ] Return to PRD with scoping stage once baselines and constraints are known
