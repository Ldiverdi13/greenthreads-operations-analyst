# GreenThreads Operations — Project Context

Everything established across HW1, HW2, and HW3. Read this first; it tells you what
the other files are and what has already been verified.

## Who we are

Operations team (student consultant team) for GreenThreads, a sustainable apparel
company opening a Denver store. Course: AI.205 AI Integration in Business I,
Prof. Jeff Eyet. Function goal: get the four launch products onto the Denver sales
floor on time.

**Scope boundary:** We are Operations. Data from Finance, HR, and Marketing is an
input to our decisions only. We do not make recommendations to fix other
departments' problems.

## The four launch products

| Product | SKU | Supplier | Country | Units | Wholesale | MSRP | Unit margin | Lead time | Units/carton |
|---|---|---|---|---|---|---|---|---|---|
| EcoFleece Hoodie | GT-HOD-04 | Andes Knitworks | Colombia | 850 | $38 | $98 | $60 | 30 d | 12 |
| Classic Tee | GT-TEE-01 | Delta Organics | India | 1,600 | $14.25 | $38 | $23.75 | 45 d | 24 |
| Active Shorts | GT-SHT-02 | Mekong Textile Co. | Vietnam | 1,200 | $22 | $58 | $36 | 60 d | 20 |
| Bamboo Joggers | GT-JOG-03 | Song Hong Apparel | Vietnam | 950 | $30 | $78 | $48 | 65 d | 16 |

Totals: 4,600 units, $287,800 retail value, $177,800 gross margin.
Wholesale cost is merchandise only. It excludes freight and duty, so true landed
COGS is higher and margin figures are gross of both.

## Verified findings

**Supplier reliability** (from 96 POs, 80 received, PO-4400 to PO-4495):

| Supplier | Country | Received | % late | Avg days late | Max | In transit | Overdue now |
|---|---|---|---|---|---|---|---|
| Song Hong Apparel | Vietnam | 12 | 100% | 12.1 | 19 | 3 | 2 |
| Mekong Textile Co. | Vietnam | 17 | 100% | 6.9 | 12 | 9 | 9 |
| Delta Organics | India | 28 | 57.1% | 0.8 | 2 | 4 | 4 |
| Andes Knitworks | Colombia | 23 | 52.2% | 0.5 | 1 | 0 | 0 |

No shipment in the dataset has ever arrived early. "On-time" only ever means
landing exactly on the promised day, so on-time rate flatters every supplier.
Use average days late as the severity measure and report both.

**Exposure:** The two Vietnam suppliers make the Active Shorts and Bamboo Joggers,
which together carry $143,700 of retail value (49.9%) and $88,800 of gross margin
(half the launch).

**Margin economics** (Austin store, 134 days, Mar 2 – Jul 13 2026):
Realized gross margin 61.4%, closely tracking the 62% the catalog assumes.
$33.04 gross margin per unit sold, ~93 units/day, ~$3,064 gross margin per average
day, ~$4,334 on a Saturday. Austin is an established store, so opening days should
run above this. Treat it as a conservative floor, not a forecast.

## The central open issue: the opening date is not established

| Source | What it establishes | Implied date |
|---|---|---|
| SKU catalog (Ops) | Order-by + lead time for all four launch SKUs | Oct 12 — planning assumption |
| HR offers + handbook 3.3 | Associates start Sept 28; 2 weeks paid training | Oct 12 — zero slack |
| Lease §3.1 / §3.4 | Delivery "on or about" Sept 1; build-out customarily 6–8 weeks | **Oct 13–27 — earliest open is Oct 13** |
| Lease §4.2 | Rent starts on earlier of opening or 60 days after delivery | Oct 31 — hard financial deadline |
| Required in-store date | OPS-04 step 1 requires it before ordering | **NOT ESTABLISHED anywhere** |

The earliest date the lease's own language allows is one day after the date all
three departments are planning toward. October 12 is not tight; it is outside the
window. No document in any department states a required in-store date.

**Planning inference (not a forecast):** Song Hong's 12.1-day mean applied to
SH-2291's October 3 promise implies ~October 15 receipt.

## Governing documents

- **OPS-04 (receiving SOP):** Plan backward from the required in-store date.
  Suppliers with a documented late-delivery history require a minimum two-week
  buffer — but "late" is never defined, so the rule has never fired. Inspect every
  carton within 24 hours; log any variance the same day.
- **Master Supply Agreement (Song Hong):** §4 delivery dates are good-faith
  estimates, not guarantees. §5 excuses delays from capacity, inspection, port
  congestion, freight, customs. §6 no penalty; remedy limited to cancellation
  before dispatch. §7 dispatch gated on supplier's own pre-shipment inspection.
  §9 ±2% quantity tolerance on cut-and-sew apparel.
- **PO SH-2291:** 950 Bamboo Joggers, $28,500, ordered Jul 24, promised Oct 3.
  Letter promises no issues while disclosing peak capacity through late September
  and 18–22 day ocean transit from container load.
- **Denver lease:** 2,400 RSF, Suite 120 Cherry Creek North. Base rent $8,400/mo.
  §8 no penalty, offset, or abatement for delayed landlord delivery.

## Current recommendation (HW3 v2)

Choice C, "Gate": confirm the required in-store date first, put both Vietnam launch
orders on exception control (require ex-factory, container-load, booking, weekly
milestones), secure partial-expedite and backup-source quotes now but release spend
only on a verified trigger, and adopt a standing rule that any supplier averaging
more than 3 days late gets an automatic two-week buffer.

Decision rights: Operations owns escalation. Procurement owns spend and sourcing.
Legal owns contract terms. Store Manager confirms the required in-store date.
AI calculates and flags; a human reviews every trigger and makes the final call.

## Known limits — state these, do not paper over them

1. **No Denver in any dataset.** Shipments, Austin sales, Portland sales, Marketing
   customer data all come from other stores. Denver behavior is inferred, never
   observed.
2. **Thin sample on Song Hong.** The 12.1-day average rests on 12 completed orders.
   Real pattern, small sample. Directional only.
3. **Carton multiples unresolved.** 3 of 4 launch buys are not whole cartons
   (Hoodie 70.8, Tee 66.7, Joggers 59.4). Escalated for confirmation, not changed.
4. **Store name conflict.** HR applicant file says "Denver-LoHi"; lease and offer
   letter say Cherry Creek North. Must be settled before any PO destination is set.
5. **No freight or duty figures exist** in any file, so landed cost and expedite
   cost remain quote-based. Do not invent them.
6. **Door_Count (Portland data)** counts door entries, not shoppers, so conversion
   rate built on it is understated by an unknown amount.

## Data quality checks already run (all passed)

0 duplicate POs, 0 required-field gaps, 0 date/status conflicts, 0 line-cost
errors, 0 catalog-mapping errors, 0 Days_Late mismatches across 80 received rows.
16 in-transit rows have blank receipt dates — legitimately incomplete, preserved as
unknowns rather than scored as on-time.

## Other departments' data (context only — not our remit)

- **Finance:** Denver budget ~$322k; Opening-buy lines are Operations-owned.
  Austin daily sales give our margin rate. Lease terms above.
- **HR:** 148 applicants, 14 openings, 7 accepted, 39% offer-accept rate. Sales
  Associates start Sept 28. Three declines cite start date too far out.
- **Marketing A:** Austin channel test, 30 days. Instagram ROAS falls as daily
  spend rises (correlation −0.97).
- **Marketing B:** Customer LTV by acquisition channel. Returns run 0.27 per order
  — reverse logistics volume our SOP does not currently cover. AOV $92, ~1.4 units
  per order.

## Grading history

HW2: strong marks; feedback said tighten presentation and frame recommendations as
risk scenarios rather than guaranteed outcomes.
HW3: 94/100. Points lost on options not being priced in a common unit, sample size
not qualified, and a few claims stated more confidently than the evidence supports.
Both are addressed in the v2 materials.
