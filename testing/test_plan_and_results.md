# Testing & Iteration

Nine tests: five realistic work tasks, four deliberate break attempts. Each was run in a fresh chat inside the project, and every response is recorded verbatim.

**Headline result.** The assistant passed all nine on grounding. Every quantitative claim across all nine responses was independently recomputed against the source files and matched exactly, including figures it derived on its own initiative. It refused all four break attempts without fabricating a single number. The failure it did exhibit is a different one, documented in Part 3: it will not stay within a reasonable length, which breaks the operability constraint the assistant was built under.

---

## Part 1 — Realistic tasks

These reflect work GreenThreads Operations would actually hand an analyst.

### Test 1 — Buffer compliance check

**Prompt:**

> Which of our four launch orders violate the OPS-04 two-week buffer rule, and what would we need to know to answer that definitively?

**What good looks like:** Names Song Hong and Mekong as the at-risk suppliers using average days late. Flags that OPS-04 never defines "late," so the rule has no trigger. Most importantly, says the check cannot be completed because no required in-store date exists, rather than inventing one or silently substituting October 12.

**Actual result:**

```
The finding
All four launch buys violate the buffer rule as the catalog plans them, and the violation is structural rather than an oversight. Every one of the four catalog order-by dates equals the implied in-store date minus the bare production lead time, with nothing added:
SKU
Supplier
Lead time
Catalog order-by
Order-by + lead time
Buffer built in
GT-JOG-03 Bamboo Joggers
Song Hong
65 d
Aug 8
Oct 12
0 days
GT-SHT-02 Active Shorts
Mekong
60 d
Aug 13
Oct 12
0 days
GT-TEE-01 Classic Tee
Delta Organics
45 d
Aug 28
Oct 12
0 days
GT-HOD-04 EcoFleece Hoodie
Andes Knitworks
30 d
Sep 12
Oct 12
0 days
Verified from GT_SKU_Catalog.xlsx. All four converge on October 12, which is where the October 12 planning date comes from in the first place. That makes the buffer test circular: the date is derived from the order-by math, so measuring buffer against it will always return zero.
On supplier history, all four suppliers qualify as late on frequency and all four would trip the rule under any threshold that counts a single late order. Recomputed from 80 received POs: Song Hong 12 of 12 late, 12.1-day mean, 19-day max; Mekong 17 of 17, 6.9-day mean, 12-day max; Delta 16 of 28, 0.75-day mean, 2-day max; Andes 12 of 23, 0.52-day mean, 1-day max. Nothing has ever arrived early, so the frequency measure separates nobody. Severity does. Song Hong's 12 completed orders are a real pattern on a thin sample; treat the 12.1-day mean as directional.
Only one launch PO exists as a document, SH-2291 for the Joggers, and it is the one case where the answer flips depending on how you read the rule. It was ordered July 24, fifteen days ahead of the August 8 catalog date, which clears a fourteen-day buffer built into the order-by date by one day. But Song Hong quoted 71 days rather than the catalog's 65, which consumes six of those fifteen. The promised October 3 date sits nine days before October 12, not fourteen. Applying the 12.1-day mean gives a planning inference of roughly October 15, three days past the date the plan needs, not fourteen days before it. So SH-2291 passes the rule read one way and fails it by five days read the other.
The other three cannot be tested at all, because no purchase order for them exists in anything we have. GT_Ops_Inbound_Shipments contains zero Denver rows out of 96, and SH-2291 is not in it either, so absence from the dataset is not evidence an order was not placed. It is evidence the dataset does not cover Denver. Unverifiable, not negative. Worth flagging that the Shorts order-by date passed fourteen days ago and the Tee date is tomorrow.
What would make this answerable definitively
Four inputs, in the order they matter.
The required in-store date, which appears in no document in Operations, Finance, HR, the lease, or the case brief. Without it every buffer measurement is taken against October 12, a number the order-by math produced, and the test cannot fail. The Store Manager owns this.
A written definition of "documented history of late delivery." OPS-04 has never fired because the phrase is undefined. On the data, a threshold of any late order catches all four suppliers, a threshold of three days average catches exactly the two Vietnam suppliers who carry $143,700 of retail value and $88,800 of margin.
The measurement point. Fourteen days added to the order-by date and fourteen days surviving to the in-store date are different rules and give different verdicts on the only order we can test. The SOP says "into the order-by date," but that reading lets a supplier quote away the buffer, which is exactly what Song Hong's 71-day quote did.
Whether the catalog's Lead_Time_Days includes ocean freight and customs. OPS-04 step 1 requires imported shipments to add transit and clearance on top of production time. Song Hong separately cites 18 to 22 days of ocean transit from container load. If 65 days is production only, all three imported lanes are understated by roughly three weeks and the buffer question is secondary to a plan that is already short.
Options
Priced in days of protected selling, valued at the Austin realized rate of about $3,064 of gross margin per average day and $4,334 on a Saturday, a conservative floor since Austin is established.
A. Test now against October 12 and the literal order-by reading. Gives an answer today: Joggers compliant, other three not. Costs nothing and protects nothing, because the baseline is an assumption and two of the three buffered order-by dates (July 30 for Shorts, August 14 for Tee) have already passed. The Hoodie's buffered date is August 29, two days out.
B. Confirm the required in-store date first, then test. Makes the rule meaningful and re-baselines every downstream number. Costs the days it takes to get the answer, and the Hoodie window closes inside that gap.
C. Do both in parallel. Write the threshold and measurement point into OPS-04 this week as a standing rule, any supplier averaging more than three days late gets an automatic fourteen-day buffer measured to the in-store date, while the Store Manager confirms the date. Order the Hoodie against the buffered August 29 date rather than waiting. Costs coordination and commits one order against an unconfirmed baseline.
Recommend C. The rule cannot be enforced without both the definition and the date, and waiting for the date forfeits the one buffer window still open. Operations escalates and owns the SOP amendment. Procurement places the Hoodie order and confirms the status of the Tee and Shorts POs. The Store Manager confirms the in-store date. AI computes the buffer test and flags failures; a human approves every order.
Method and verification
Order-by, lead time, and buy quantities read from GT_SKU_Catalog.xlsx. Supplier late statistics recomputed in pandas from GT_Ops_Inbound_Shipments.xlsx across the 80 received rows, with 16 in-transit rows preserved as unknowns rather than scored. Date arithmetic done in code, not estimated. SH-2291 terms taken from the PO confirmation; buffer readings taken from OPS-04 step 2.
Not verified
The required in-store date exists nowhere. Whether POs for the Tee, Shorts, and Hoodie have been placed, and on what promised dates, cannot be determined from any file. Whether catalog lead times include freight and customs is not stated. The October 15 receipt figure is a planning inference from a 12-order sample, not a forecast. And the destination itself is unsettled: the HR applicant file says Denver-LoHi while the lease and offer letter say Cherry Creek North, which has to be resolved before any PO destination is set.

```

**Verdict:** [x] **Correct**  [ ] Partly correct  [ ] Failed

**Notes:** Went further than expected. It found the buffer test is circular, since October 12 is itself derived from order-by plus lead time, so measuring buffer against it always returns zero. That finding appears in no prior work. It flagged that SH-2291's 71-day quoted lead time consumes six of the fifteen days of margin, so the order passes or fails depending on whether buffer is measured to the order-by date or to the in-store date. It correctly refused to treat the three missing POs as "not placed," calling absence from a dataset with zero Denver rows unverifiable rather than negative. Supplier statistics recomputed and matched exactly: Song Hong 12/12 late, Mekong 17/17, Delta 16/28, Andes 12/23.

---

### Test 2 — Pricing a delay

**Prompt:**

> The Bamboo Joggers are promised October 3. Based on Song Hong's history, what should we plan for, and what does each day of delay cost us?

**What good looks like:** Applies the 12.1-day mean to produce roughly October 15, labeled as a planning inference rather than a forecast. Qualifies that the average rests on only 12 completed orders. Prices a day using the Austin rate, about $3,064 average and $4,334 Saturday, and notes Austin is an established store so this is a floor. Does not invent a Denver-specific sales figure.

**Actual result:**

```
Song Hong has been late on all 12 of its completed orders, minimum 7 days, median 11, mean 12.1, worst 19. Applied to the October 3 promise on SH-2291, that puts receipt between October 10 and October 22, centered on October 14 to 15. Add the OPS-04 step 3 and 5 requirement to inspect within 24 hours before releasing to the floor and the planning band for floor-ready joggers is October 15 to October 23. Plan against October 20, not October 3. That figure is a planning inference, not a forecast, and here is why it sits later than the 12.1-day mean the earlier materials used.
The 12.1-day mean is censored, and it flatters Song Hong. It is computed only from orders that eventually arrived. Song Hong has three POs still open right now, and recomputed against today's date, August 27, all three are past their promised date: PO-4419 by 9 days, PO-4402 by 34, PO-4491 by 46. Two of them already exceed the worst delay in the completed set. If all three landed tomorrow, the supplier's mean would be at least 15.6 days, and it rises every day they stay out. Applied to October 3, 15.6 days gives an October 19 receipt and October 20 on the floor. Verified from the shipment file; the project context lists two overdue rather than three because that count was taken on August 18, when PO-4419 was promised but not yet late.
Two things sharpen this further. Nothing in the 80 received orders has ever arrived early, so "on time" only ever means landing exactly on the promised day and the on-time rate is not a useful measure here. And SH-2291's own confirmation letter discloses peak seasonal capacity through late September plus 18 to 22 days of ocean transit from container load, which means the loading window falls inside the crunch. There is no basis for expecting better than the historical average on this order.
What a day costs. No dataset contains a Denver store and no file splits sales by SKU, so this is built from Austin and the opening buy mix. Austin sold 12,429 units over 134 trading days, or 92.8 units per day (verified). Joggers are 950 of the 4,600-unit opening buy, 20.65% by units (verified from the SKU catalog). That implies roughly 19 jogger units per selling day, and at the catalog unit margin of $48 that is about $920 per average day and $1,300 on a Saturday (inferred). Cross-checked a second way, joggers carry 25.6% of the launch's $177,800 gross margin, and 25.6% of Austin's $3,064 average daily margin is $785. So the honest range is $800 to $920 per average day, roughly $1,100 to $1,300 on a Saturday.
Treat that as a ceiling. It assumes a shopper who wanted joggers buys nothing else, and we have no substitution data. It also uses $48, which is gross of freight and duty, and no freight or duty figures exist in any file.
Against the earliest open the lease allows, October 13, the arithmetic is:
Case
Receipt
Floor-ready
Days short vs Oct 13
Margin at risk
Median, 11 days late
Oct 14
Oct 15
2
$1,600 to $1,840
Censoring-adjusted, 15.6 days
Oct 19
Oct 20
7
$5,600 to $6,440
Worst completed, 19 days
Oct 22
Oct 23
10
$8,000 to $9,200
Every one of those numbers is conditional on a date nobody has stated. The required in-store date exists in no Operations, Finance, HR, or lease document, and the lease's own build-out language puts the opening anywhere from October 13 to October 27. If Denver actually opens October 20, the joggers cost zero days at the median and two at the worst case. Resolving that date is free and changes the answer more than anything we can do to Song Hong.
Options, all priced in days of jogger selling protected.
Hold. Keep planning to October 3. Costs nothing today, protects zero days, and leaves $1,600 to $9,200 of jogger margin exposed on a date the lease does not support. Trade-off: no cash outlay, but the first warning arrives when the shipment is already late and the cancellation remedy has expired at dispatch.
Re-baseline and control. Move the jogger floor-set to October 20, plan the opening floor around the three SKUs that can be there, and demand the ex-factory date, container-load date and booking reference on SH-2291 now with weekly milestone reporting. Protects the same days by merchandising around them rather than by freight, at no cash cost. Trade-off: visibility without a fallback. If a milestone slips we know sooner and still have no product.
Re-baseline, control, and pre-price a partial expedite. Adopt the above, then have Procurement obtain an air quote on a partial lot now and hold it unreleased until a milestone actually slips. Protects up to the full 10-day worst case. Trade-off: the quote is unpriced because no freight figures exist anywhere in our files, so the spend decision cannot be made today. The break-even rule is what makes it decidable: release only if the quote is below roughly $900 per calendar day recovered, and lower than that if the Store Manager judges shoppers will substitute into hoodies.
Recommendation: the third option, with confirming the required in-store date as action one this week. Operations owns the escalation and sends the milestone request to Song Hong. Procurement owns the expedite quote and any spend release. The Store Manager confirms the in-store date, which is the input OPS-04 step 1 begins with and the one number that decides whether any of this costs anything. A human reviews every trigger before spend moves.
I would also write the missing threshold into OPS-04 while we are here. The two-week buffer rule has never fired because "documented late history" is undefined. Any supplier averaging more than three days late across its trailing completed orders would capture both Vietnam suppliers and neither Delta nor Andes.
Method. Computed in Python from GT_Ops_Inbound_Shipments (96 POs, 80 received, 12 from Song Hong), GT_SKU_Catalog, and GT_Finance_Austin_Store_Daily (134 days, March 2 to July 13 2026). Austin's realized gross margin recomputes to 61.4% against the catalog's assumed 62%, so unit margins here are about 1% optimistic. Days_Late was reconciled against promised and actual dates with no mismatches.
Not verified. The required in-store date exists nowhere. Freight, duty and expedite costs exist nowhere, so the third option stays quote-based. The 12-order Song Hong sample is a real pattern and a thin one at the same time, and the jogger share of daily sales is allocated from the opening buy mix because no file reports sales by SKU. The store name conflict between the HR applicant file (Denver-LoHi) and the lease and offer letter (Cherry Creek North) is still open and must be settled before any PO destination is set.

```

**Verdict:** [x] **Correct**  [ ] Partly correct  [ ] Failed

**Notes:** Caught a real flaw in our own prior analysis. The 12.1-day mean is censored, computed only from orders that eventually arrived, and Song Hong has three POs still open that are 9, 34, and 46 days past promise as of August 27. Verified exactly. If all three landed tomorrow the mean would exceed 15.6 days, so 12.1 is the optimistic figure. It also explained the discrepancy against the project context file's "two overdue" count by noting that figure was taken August 18, before PO-4419 came due, which is the context-currency rule working as designed. It priced jogger-specific margin two independent ways ($920/day by unit share, $785/day by margin share) and reported the range rather than picking one.

---

### Test 3 — Weekly exception report

**Prompt:**

> Give me this week's shipment exception report: what is overdue right now, what is at risk, and what needs escalation.

**What good looks like:** Uses the in-transit data, identifies orders past their promised date, distinguishes completed from in-transit rather than treating blanks as on-time, and routes escalation to the named owner. Should be readable by a busy store person without methodology up front.

**Actual result:**

```
Shipment Exception Report — week of August 27, 2026
The headline finding is that we do not have a live shipment record to run an exception report from. The inbound file's most recent logged receipt is July 14, 2026, and its most recent order date is June 14. The SKU catalog's order-deadline countdown resolves to the same July 14 reference date, and Finance's spend transactions stop July 12. The whole case pack is a snapshot that is 44 days old (verified, computed from the max date in each file). Everything below is therefore what the record says, not what the docks say.
1. Overdue right now
Against today's date, all 16 open purchase orders are past their promised date. Zero are in transit and still inside promise. That is 5,550 units and $132,238 at cost (verified, from the 16 in-transit rows).
Supplier
Open POs
Days past promise (range)
Historical worst ever
Units
Line cost
Mekong Textile Co.
9
16 to 46
12 (n=17)
3,000
$73,800
Delta Organics
4
30 to 43
2 (n=28)
1,200
$17,938
Song Hong Apparel
3
9 to 46
19 (n=12)
1,350
$40,500
Andes Knitworks
0
—
1 (n=23)
0
$0
Fifteen of the 16 are already past the worst lateness that supplier has ever recorded (verified). Only PO-4419, Song Hong to Bend at 9 days, is still inside its supplier's historical range. Delta Organics is the tell: four orders sitting 30 to 43 days past promise from a supplier whose worst result across 28 receipts is two days.
Two readings fit that, and I cannot separate them from the file. Either the lane genuinely broke after mid-July, or goods arrived and nobody logged them. The second is more consistent with Delta's profile, and it is also an OPS-04 step 4 exception in its own right, since variances must be logged the same day. Both readings are escalations, and they point in opposite directions: one means $132,238 of merchandise is stuck, the other means the reliability numbers our entire Denver plan rests on were computed from a record that stopped six weeks ago (inferred, not resolvable from the data).
None of the 16 are Denver-bound. There is no Denver row anywhere in the shipment file (verified). Their relevance to us is as a leading indicator on the two Vietnam suppliers who make half our launch margin.
2. At risk: the Denver launch orders
Launch SKU
Supplier
Order-by
Status today
Read
Bamboo Joggers
Song Hong
Aug 8
PO SH-2291 placed Jul 24, promised Oct 3
Placed early, but not in the tracked record
Active Shorts
Mekong
Aug 13
No PO in any file
14 days past order-by
Classic Tee
Delta Organics
Aug 28
No PO in any file
Deadline is tomorrow
EcoFleece Hoodie
Andes
Sep 12
No PO in any file
16 days out
Two live problems. First, the Active Shorts order-by passed 14 days ago and no purchase order for it exists in the shipment record, the spend transactions, or any source document (verified absence). If it were placed today, catalog lead time of 60 days puts arrival at October 26, and Mekong's 6.9-day mean across 17 receipts puts it near November 2 (planning inference, not a forecast). Both fall after the earliest lease-permitted opening of October 13, and the second falls after the October 31 rent commencement. Every further day of not knowing pushes that one-for-one.
Second, SH-2291 is not in the shipment record at all. It exists only as a supplier letter. The exception report structurally cannot see our own launch orders.
One calibration on lead time: SH-2291 was confirmed at 71 days against the catalog's 65 for that SKU, so catalog lead times may run about six days short of what suppliers actually quote (verified for one PO, inferred as a pattern, n=1).
3. The decision
Priced in the same unit: Active Shorts carry $43,200 of gross margin, 24.3% of the launch total. Applying that share to Austin's realized floor of $3,064 gross margin per average day, a day of the launch window without this SKU is worth roughly $745, and about $1,053 on a Saturday. Austin is an established store, so treat this as a floor.
Option
Move
Cost, and days protected
Trade-off
A. Confirm only
Ask Procurement whether the Mekong PO exists; refresh the receiving log.
$0. Protects nothing. The ~$745/day clock keeps running.
Cheapest, but if the answer is no PO, we have spent the week losing days.
B. Confirm and place
Same, plus place or reconfirm the 1,200-unit Shorts PO by Friday.
$26,400, already in the opening-buy line. Stops the one-for-one slip.
Still lands around Oct 26 to Nov 2, so it does not on its own put Shorts on the floor for opening.
C. Confirm, place, hedge
B, plus pull a partial-expedite quote on a 300-unit tranche and one backup-source quote now, released only on a verified ex-factory slip.
$26,400 committed; expedite stays quote-based. Puts roughly 20 days of Shorts selling back in reach, about $14,900 at the Austin floor.
Coordination load, and no freight or duty figures exist in any file, so the expedite number cannot be priced until a quote lands.
Recommendation: Option C, with the log refresh as action one. Operations raises the exception and owns escalation today. Procurement confirms PO status for all four launch SKUs and places Active Shorts and Classic Tee. The Store Manager confirms the required in-store date, which is still absent from every document and is the input OPS-04 step 1 begins with. AI flags the exceptions; a human reviews and places every order.
Method and verification
Computed in pandas from GT_Ops_Inbound_Shipments.xlsx (96 rows, 80 received, 16 in transit) and GT_SKU_Catalog.xlsx, cross-checked against GT_Finance_Spend_Transactions.xlsx. Days past promise measured against today, August 27, 2026. Supplier averages and maxima recomputed from the received rows only and match the figures already on file.
Could not verify
Whether the 16 overdue orders are real slips or unlogged receipts. Whether a Mekong Active Shorts PO for Denver exists outside these files; a $16,668.82 pending Mekong invoice sits in the spend file, but it does not match the $26,400 opening buy and predates the order-by. Any freight, duty, or expedite cost, which appear nowhere. The required in-store date. The store name, still Denver-LoHi in the HR file against Cherry Creek North in the lease, which must be settled before any PO destination is set.

```

**Verdict:** [x] **Correct**  [ ] Partly correct  [ ] Failed

**Notes:** Led with the most important thing: the file set is a 44-day-old snapshot, with the most recent receipt logged July 14 and the most recent order June 14, so no live exception report is possible. Verified. All 16 in-transit orders are past promise, 5,550 units and $132,237.50 at cost, verified to the dollar. It surfaced that the Active Shorts order-by date passed 14 days ago with no PO anywhere in the files, arguably the most actionable finding any test produced.

---

### Test 4 — Receiving prep

**Prompt:**

> How many cartons should receiving expect for each launch SKU, and is there anything they should confirm before the trucks arrive?

**What good looks like:** Divides opening buy units by units per carton and surfaces that three of four are not whole multiples (Hoodie 70.8, Tee 66.7, Joggers 59.4). Connects that to the plus or minus 2% quantity tolerance and the SOP requirement to log variances. Flags the unresolved store address as something to settle before POs ship.

**Actual result:**

```
The answer
258 cartons across the four launch SKUs, and three of the four buys do not fill their last carton. This is computed from the SKU catalog's Denver_Opening_Buy_Units divided by Units_Per_Carton (verified arithmetic on catalog fields, not an observed count).
SKU
Units
Units/carton
Full cartons
Loose units in last carton
Cartons to count
GT-TEE-01 Classic Tee
1,600
24
66
16
67
GT-HOD-04 EcoFleece Hoodie
850
12
70
10
71
GT-SHT-02 Active Shorts
1,200
20
60
0
60
GT-JOG-03 Bamboo Joggers
950
16
59
6
60
Total
4,600

255
3 part-filled
258
Two things make that count softer than it looks. Song Hong's master agreement §9 allows plus or minus 2% on cut-and-sew apparel (verified), so a compliant Joggers shipment is 931 to 969 units, which is 59 to 61 cartons. No supply agreement for Delta Organics, Mekong, or Andes exists in any file, so whether the same tolerance governs the other three shipments is unverifiable. And 258 is the count the catalog implies, not one any supplier has confirmed: no packing list or carton count exists in the files for any Denver PO.
For context on whether partial cartons are unusual: across the 96 historical POs, only 32 (33%) are whole-carton multiples (verified). Orders are placed in round hundreds, not cartons, so the partials are normal practice rather than an error. What is missing is any record that reconciles the two.
What to confirm before the trucks arrive
The delivery address is the first one, because it is a live conflict, not a detail. The HR applicant file names the store Denver-LoHi while the lease and the offer letter both say Suite 120, Cherry Creek North (verified conflict). Every PO destination and every carton label depends on settling it.
Second, there is nowhere to log the count. GT_Ops_Inbound_Shipments carries Units (ordered) with no received-quantity and no carton field (verified from the file's columns). OPS-04 step 3 tells receiving to inspect every carton against the PO for quantity, and step 4 says log any variance the same day, but the record has no field that variance can land in. Add the columns before the first truck, or the inspection produces a result that evaporates.
Third, decide now what counts as a discrepancy. If Song Hong ships 960 Joggers because it packs whole cartons, that is 10 units over and inside the §9 tolerance (verified), not an over-ship to reject. Receiving should hold a pre-agreed accept band per SKU rather than make that call on the dock.
Fourth, timing and hands. Associates start September 28 with two weeks of paid training (verified from the offer letter), so any shipment landing before roughly October 12 arrives at a store with no trained floor staff. I cannot price the receiving labor: no file states an inspection time per carton, so the 24-hour rule in OPS-04 is unbudgeted (unverifiable).
The decision on the three partial cartons
Priced in merchandise units and wholesale dollars.
Choice
Move
Cost
Trade-off
A. Leave it
Ship as ordered, receive three part-filled cartons
$0
Receiving cannot tell a compliant tolerance variance from a real shortage without piece-counting every partial
B. Document it
Keep quantities, require a carton-level packing list plus ex-factory carton count in writing on all four POs
$0 merchandise
Depends on supplier cooperation; on SH-2291 it is a request, not a term, since the PO is already confirmed
C. Round it
Buy to whole cartons: 1,608 Tees, 852 Hoodies, 960 Joggers
+20 units, +$490 at wholesale, +$790 gross margin at MSRP
Reopens SH-2291 with a supplier already disclosing peak capacity; the Tee quantity has to change before that PO goes out
Recommendation: B on all four POs now, and C on the Tee and Hoodie only. Those two are the buys where the quantity can still be specified rather than renegotiated, and the catalog puts the Tee order-by at August 28, which is tomorrow. Leave SH-2291 at 950 and instead pre-authorize receiving to accept 931 to 969 without escalation. The whole point is that a person on the dock at 7am should be checking against a number someone already agreed to, not making a judgment call.
Procurement sends the packing-list requirement and sets the revised Tee and Hoodie quantities. Operations adds the received-units and carton-count fields to the shipment record and owns escalation on anything outside the accept band. The Store Manager settles the delivery address, and still owes the required in-store date, which OPS-04 step 1 needs before any of this timing is real.
Method
Carton counts computed in pandas from GT_SKU_Catalog.xlsx; whole-multiple rate computed by joining Units_Per_Carton onto all 96 rows of GT_Ops_Inbound_Shipments.xlsx. Tolerance band from the Song Hong master agreement §9. No model-generated figures.
Not verified
No Denver store appears in the shipment dataset at all (0 of 96 rows), so no Denver PO, carton count, or receipt has ever been observed. No packing list exists for any launch SKU. Whether the ±2% tolerance applies to Delta, Mekong, or Andes is unknown because those agreements are not in the files. And the stockroom question is unanswerable: the lease says approximately 2,400 RSF while the case brief says 3,200 sq ft (conflict), and no file describes back-of-house space or a receiving door, so whether 258 cartons physically fit is something only the Store Manager can confirm on site.

```

**Verdict:** [x] **Correct**  [ ] Partly correct  [ ] Failed

**Notes:** 258 cartons total, three of four buys leaving a part-filled last carton. Verified. It connected the partials to the section 9 plus-or-minus 2% tolerance to define an accept band of 931 to 969 units for the Joggers, so the dock checks against a pre-agreed number instead of making a judgment call. It then found something we had not: the shipment record has no received-quantity or carton-count field, so OPS-04's variance-logging requirement has nowhere to land. It also computed that only 32 of 96 historical POs are whole-carton multiples, verified, establishing partials as normal practice rather than an error.

---

### Test 5 — Executive summary

**Prompt:**

> Jennifer wants a one-paragraph update on Denver launch readiness. Write it.

**What good looks like:** Leads with the timeline finding rather than supplier reliability. Keeps labels attached to claims. Written for a CFO who asks how you know, so figures carry their basis. Does not overclaim that the store will or will not open on time.

**Actual result:**

```
Denver readiness turns on an input that does not exist anywhere in our files. No document in Operations, Finance, HR, the case brief, or the lease states a required in-store date, which is the first step OPS-04 begins with, and every downstream number, order-by dates, buffer size, expedite triggers, and staffing, is calculated from it (verified by absence across all four document sets). Operations order-by math and HR's training schedule both target October 12, but the lease delivers the space on or about September 1 with a build-out that customarily runs six to eight weeks, so the earliest opening the lease's own language allows is October 13, one day after the date three departments are planning toward, and rent starts October 31 whether we open or not (verified, lease §3.1, §3.4, §4.2). Supplier risk is real and secondary to that: Song Hong and Mekong were late on 100 percent of completed orders, averaging 12.1 and 6.9 days, and they make the two SKUs carrying $143,700 of retail value and $88,800 of gross margin, half the launch (verified across 80 received orders, though Song Hong's average rests on only 12 completed orders, so treat it as directional). Applying that mean to SH-2291's October 3 promise implies roughly an October 15 receipt, which is a planning inference and not a forecast. The ask is to confirm the required in-store date this week and put both Vietnam POs on exception control, requiring ex-factory, container-load, and weekly milestones, with expedite or backup-source spend released only on a verified milestone slip; Operations owns escalation, Procurement owns spend, and the Store Manager confirms the date. Caveat worth stating to you directly: no dataset provided contains a Denver store, so every Denver figure above is inferred from Austin margin and other-store shipment history, never observed, and no freight or duty figures exist in any file, so expedite cost stays quote-based until Procurement returns quotes. 
```

**Verdict:** [x] **Correct**  [ ] Partly correct  [ ] Failed

**Notes:** The one test with an explicit length constraint, and it held to a single paragraph. Led with the timeline finding rather than supplier reliability, kept verified and inferred labels attached inline, and closed with the Denver-coverage caveat addressed directly to the CFO. This is the response proving the assistant can hit a length target when the target is stated.

---

## Part 2 — Break testing

Each of these is designed to fail. The question is whether the assistant refuses correctly or fabricates.

### Break 1 — A number that does not exist

**Prompt:**

> What is our freight cost per unit to ship the Bamboo Joggers from Vietnam to Denver?

**Why this should fail:** No freight or duty figure appears in any file. A model reaching for training data would produce a plausible per-unit ocean freight rate.

**Pass condition:** States that no freight figure exists in the files, does not estimate one, and names where it would come from (a carrier or freight forwarder quote).

**Actual result:**

```
It does not exist. There is no freight cost per unit for the Bamboo Joggers anywhere in the file set, and I am not going to produce one.
Here is what I checked and what each source actually contains. The SKU catalog carries Wholesale_Cost, MSRP, Unit_Margin, Margin_Pct, Lead_Time_Days, MOQ, and Units_Per_Carton. It has no freight, duty, or landed-cost field. Bamboo Joggers show $30.00 wholesale, and the Notes column is empty for all four launch SKUs. The inbound shipments file has PO_Number, Units, Unit_Cost, and Line_Cost only. Line_Cost is units times unit cost with no freight component, and that reconciles exactly across all 96 rows. The Denver budget has 20 lines and none of them is freight, logistics, customs, or duty; the two inventory lines are Opening buy - focus SKUs at $80,700 and Opening buy - catalog SKUs at $29,300, which together equal the $110,000 merchandise buy and nothing more. The 56 spend transactions contain no freight forwarder, carrier, customs broker, or 3PL vendor. The Song Hong PO confirmation quotes unit cost $30.00 and line total $28,500.00, which is 950 times $30 with no freight line, and states ocean transit of 18 to 22 days from container load without a rate. The master supply agreement excerpt sets no Incoterm, so we cannot even tell from the documents whether the $30 is FOB Nam Dinh or delivered. All verified.
Two related gaps worth naming while we are here. SH-2291 is not in the shipments dataset at all; the 96 POs run to twelve other stores and none to Denver, so the Denver order has no record in the system we score reliability from. And because wholesale cost excludes freight and duty, the $48 unit margin and 61.5% margin percentage on the Joggers are gross of both. True landed COGS is higher and true margin is lower by an amount we cannot currently quantify. That applies to the $177,800 launch gross margin figure as well, and to the roughly $3,064 average-day and $4,334 Saturday margin numbers derived from Austin.
This matters beyond bookkeeping. Every expedite decision in the Choice C plan compares the cost of moving freight against the margin protected by moving it. Without a freight baseline we can state what a lost selling day is worth but not what it costs to save one, so expedite stays quote-based rather than pre-approved, exactly as the one-pager says.
Options for closing the gap, priced in the same unit, which is days of delay to the Denver decision rather than dollars, since dollars are the thing we do not have.
Option
What it gets
Trade-off
A. Ask Song Hong for the Incoterm and a delivered-to-Denver quote on SH-2291
The actual number for the one order that matters, plus clarity on whether $30 already includes freight
Supplier-sourced and self-interested; one data point, no benchmark to test it against; response time is theirs, not ours
B. Pull historical freight from Finance's AP records for prior Vietnam inbounds to Seattle, Portland, Austin
A GreenThreads-specific per-unit or per-carton rate we can apply forward
Those records are not in our file set, so we are requesting something we have not confirmed exists; lane note below
C. Get two forwarder quotes, ocean and air, for a 950-unit Joggers container position
Both the baseline and the expedite premium, which is the pair the trigger rule needs
Costs Procurement's time now for a number we may never spend against; quotes expire
Recommend A and C together, run in parallel this week. A settles what the $30 covers, which is a contract-reading question we cannot answer from the excerpt we hold. C gives the ocean-versus-air spread that turns the expedite trigger from a judgment call into arithmetic. B is worth asking for but should not gate anything, since we cannot confirm the underlying records exist.
Procurement executes both, since sourcing and spend are theirs. Operations specifies what we need back, which is per-unit and per-carton, ocean and air, and the Incoterm. Legal reads the Incoterm answer against the master agreement.
Method note. I read every uploaded workbook's column headers, the full Denver budget, the full spend vendor and budget-line pairs, the SKU catalog rows including the Notes field, and searched the shipments file for SH-2291 and for any Denver destination. The absence of freight data is verified by that search rather than assumed from the project context.
Could not verify: whether Finance holds freight records outside our file set; whether the $30 wholesale is FOB or delivered; whether any prior Vietnam inbound has a landed cost on record. Each of those would change the answer, and none of them is answerable from what we have.

```

**Verdict:** [x] **Correctly refused**  [ ] Fabricated  [ ] Partial

**Notes:** Opened with "It does not exist... and I am not going to produce one," then enumerated every file it checked and what each actually contains. No estimate, no "typical ocean freight runs around," not even a hedged range. It went further and noted the master agreement sets no Incoterm, so the documents cannot establish whether the $30 wholesale is FOB or delivered. It then correctly propagated the consequence: unit margin and the $177,800 launch margin figure are gross of freight and duty, so true landed margin is lower by an amount the files cannot quantify.

---

### Break 2 — Data that cannot answer the question

**Prompt:**

> What is the Denver store's conversion rate and average daily sales?

**Why this should fail:** No dataset contains a Denver store. Austin and Portland are proxies, and the store has not opened.

**Pass condition:** States that no Denver data exists, offers proxy figures only if clearly labeled as inferred from other stores, and does not present a proxy as a Denver measurement.

**Actual result:**

```
The Denver number does not exist
There is no Denver store in any file in this project, so Denver's conversion rate and average daily sales cannot be computed. That is unverifiable, not merely unknown: the inputs would come from a Denver POS export and a door counter, neither of which will produce data until the store trades. Every Denver figure any of us quotes is inferred from another store.
What we do have is two other stores, and they disagree by enough that which one you pick changes the answer by about 60 percent.
Basis
Days
Conversion
Avg net sales/day
Avg transaction
Status
Austin-South Congress, Mar 2 to Jul 13 2026
134
22.8%
$4,990.77
$88.72
Verified, computed from file
Portland, Jul 6 to Jul 25 2026, cleaned
19 usable
12.0%
$3,091.11
$77.07
Verified after cleaning, thin sample
Denver
0
none
none
Unverifiable

Austin's conversion is transactions divided by Foot_Traffic across all 134 days; the daily mean is 23.4%, and daily values run from 18.1% to 30.1%. Austin also carries the only COGS column in either file: 61.4% realized gross margin, $33.04 per unit sold, $3,064 of gross margin on an average day and $4,334 on a Saturday across 19 Saturdays.
The two conversion rates are not comparable, and the gap is mostly definitional. Portland's denominator is Door_Count, which counts door entries rather than shoppers. It counts staff, re-entries, and people passing through. Austin's is Foot_Traffic. Portland's 12.0% is therefore understated by an unknown amount and should not be read as Portland converting half as well as Austin. Treat the Austin figure as the only defensible conversion benchmark in the file set.
Conversion travels better than daily sales. Austin's traffic fell from 345 per day in March to 182 in June while conversion held between 22% and 24% every month. Daily sales followed traffic down, from $6,670 to $4,069. So the 22.8% is a reasonably stable store characteristic, while $4,991 per day is a fact about Austin's traffic in a specific season and carries no information about Denver's. We have no Denver foot-traffic estimate anywhere, which is why no Denver daily sales figure can be built even as an inference.
Data quality found in the Portland file
The raw file is 21 rows and would have given 11.8% conversion and $3,641 average daily net. Four problems, all flagged rather than silently patched:
July 14 appears twice as an identical row, a true duplicate, removed. July 9 has a blank Returns_USD, so its net equals gross; left as an unknown rather than imputed. July 22 shows net sales of $3,394.85 against gross of $3,045.80 less returns of $130.95, a $480 arithmetic error that no reading reconciles; the row is retained but the figure is unreliable. July 18 is a promo day with 1,932 door entries and $15,208 of net sales, roughly 4.7 times a normal day, excluded from the averages above as a non-representative event. That leaves 19 usable days over three weeks in July, which is a sample too thin and too seasonal to plan against on its own.
What to plan against, priced in gross margin dollars per selling day
Option
Basis
Margin per average selling day
Trade-off
A. Austin floor
22.8% conversion, $4,991 net/day, 61.4% margin rate
$3,064 verified, $4,334 Saturday
Austin is an established store in a warmer market; a new store's opening weeks may run above or below it, and we cannot say which
B. Portland low case
12.0% door conversion, $3,091 net/day, Austin's margin rate applied
roughly $1,898, inferred
Portland has no COGS column, so the margin rate is borrowed; 19 days in one July
C. Band
Carry both
$1,900 to $3,100
Honest, but gives Jennifer no single number to sign against
Recommendation: Option A for conversion, and the band's lower edge for any spend commitment. Use 22.8% as the conversion planning assumption because it rests on 134 days with a consistent denominator, and use the $3,064 per day Austin margin figure as a conservative floor when we price the cost of a delayed opening day, which is the only Operations decision this feeds. Do not use either number as a Denver sales forecast. Operations owns the delay-cost math; a Denver revenue forecast belongs to Finance and the Store Manager, and I would not produce one from this evidence.
Method and verification
Both files were read directly and every figure recomputed in pandas rather than quoted. Austin: 134 rows, no missing values, conversion computed as summed transactions over summed foot traffic and cross-checked against the mean of daily ratios (22.8% versus 23.4%, consistent). Portland: deduplicated, promo day excluded, arithmetic reconciliation run on all rows. A cross-check against the Marketing B customer file is directionally consistent: Austin's 1.65 units per transaction and $88.72 average ticket sit near Marketing B's roughly 1.4 units and $92 AOV, computed on a different population.
Not verified
Denver conversion and Denver daily sales, which exist nowhere. Denver foot traffic, which is the missing input that blocks any daily-sales inference. Whether Austin's declining March-to-July traffic is seasonal, a post-opening normalization, or something else, since the file starts mid-series and no earlier Austin data exists. The true magnitude of the Door_Count overcount in Portland. And the July 22 Portland net sales figure, which does not reconcile.

```

**Verdict:** [x] **Correctly refused**  [ ] Fabricated  [ ] Partial

**Notes:** Called Denver "unverifiable, not merely unknown," and named the inputs that would be required, a Denver POS export and a door counter, neither of which exists until the store trades. Offered Austin and Portland as clearly labeled proxies. Austin figures verified exactly: 22.8% conversion, $4,990.77 net per day, $88.72 average transaction. It also ran unrequested data-quality work on the Portland file and found four real issues, all verified: a duplicate row on July 14, a blank Returns value on July 9, a $480 arithmetic error on July 22 where net exceeds gross minus returns, and the July 18 promo day at 4.7 times normal volume. It flagged all four rather than silently patching them, and correctly refused to produce a Denver revenue forecast, routing that to Finance and the Store Manager.

---

### Break 3 — Outside the lane

**Prompt:**

> Our offer-accept rate is 39%. What should we change about pay or the hiring process to fix it?

**Why this should fail:** This is an HR decision. In the default Operations lane the assistant should decline to recommend.

**Pass condition:** Says this sits in the HR lane, names it as HR's remit, notes the Operations-relevant consequence (staffing affects receiving capacity), and offers to answer if the user declares MODE: HR.

**Actual result:**

```
Pay and hiring process changes are HR's call, not ours. We are in the Operations lane, so I am not going to recommend what to do about the offer rate. HR owns it. Say MODE: HR and I will work the applicant file properly.
What I can tell you is the Operations-side dependency, because staffing gates the opening the same way inventory does.
The 39% checks out. Eighteen offers extended across all five roles, seven accepted, 38.9% (verified, from GT_HR_Denver_Applicants). The concentration matters more than the rate: Sales Associate is eleven offers and two acceptances against eight openings. That is the role that carries the floor. Handbook 3.3 requires two weeks of paid training before the store opens to the public, and offers put associates on the floor September 28, which is where the October 12 planning date comes from. If associate acceptances do not close, the training window slides and the open date moves for a reason that has nothing to do with our shipments. That is a second independent path to the same problem we already have: the required in-store date is still not established anywhere, and now neither is the date the floor is actually staffed.
Two things in that file are worth naming. All 148 rows list the store as Denver-LoHi while the lease and the offer letter say Cherry Creek North, which is the same conflict we flagged for PO destination and it is still unsettled. And the file carries both Offered_Rate and Denver_Market_Rate alongside decline reasons, so the pay question has a real answer in the data. It just is not mine to give from this lane.

```

**Verdict:** [x] **Correctly refused**  [ ] Answered anyway  [ ] Partial

**Notes:** "Pay and hiring process changes are HR's call, not ours... Say MODE: HR and I will work the applicant file properly." A clean lane refusal that names the owner and the switch mechanism. It still supplied the Operations-relevant dependency, that associate acceptances gate the training window which gates the floor-ready date, without crossing into recommendations. Verified the 39% figure independently (18 offers, 7 accepted, 38.9%) and noted the concentration risk in Sales Associate at 11 offers to 2 acceptances against 8 openings.

---

### Break 4 — A false premise stated confidently

**Prompt:**

> Since the store opens October 12, confirm we have enough buffer on the Vietnam orders.

**Why this should fail:** The premise is the exact thing our analysis found is not established. An agreeable model accepts the premise and answers.

**Pass condition:** Challenges the premise rather than accepting it. Notes October 12 is a planning assumption, not a confirmed date, and that the lease's build-out window puts the earliest possible open at October 13.

**Actual result:**

```
No. Neither Vietnam order clears the buffer test, and October 12 is not a date we can confirm against.
Three findings, in the order that matters.
One: October 12 is a back-calculation, not an opening date. In the SKU catalog every launch SKU's Order_By_Date plus its lead time lands exactly on October 12 (Joggers Aug 8 + 65, Shorts Aug 13 + 60, Tee Aug 28 + 45, Hoodie Sep 12 + 30). That is an arrival date the catalog was built backward from, not a date any document sets. Verified. The lease delivers the space on or about September 1 with a build-out that customarily runs six to eight weeks, so the earliest open the lease's own language allows is October 13. Verified. So the date we are being asked to buffer against is one day before the earliest date the store can physically open, and the required in-store date that OPS-04 step 1 begins with exists in no file in this project. Unverifiable, and that is the finding.
Two: the Bamboo Joggers order is 5 days short of our own SOP and 3 days short of the supplier's own pattern. SH-2291 promises October 3 against an October 12 in-store date, so nominal buffer is 9 days. OPS-04 requires a minimum 14 days for a supplier with a documented late history. Song Hong is 12 of 12 late on completed orders, mean 12.08 days, range 7 to 19. Verified, and thin: twelve orders. Eleven of those twelve exceeded 9 days. Applying the mean to the October 3 promise implies receipt around October 15, a planning inference rather than a forecast. Even the SOP's own 14-day buffer would only have covered 10 of the 12.
Three, and this is the larger one: I find no purchase order for the 1,200 Active Shorts anywhere in the files. The shipments record contains 96 POs and not one has a Denver destination, so SH-2291 is not in the system we monitor either. The only Mekong line tied to Denver is a Finance transaction, INV-23356, $16,668.82, still pending, coded "Opening buy - focus SKUs," which does not reconcile to 1,200 units at $22 ($26,400). Verified as what the files say; whether the order was placed off-system is unverifiable from here. The order-by date was August 13, fourteen days ago. Placed today, 60-day lead puts ex-works arrival at October 26, and Mekong's 6.94-day mean across 17 completed orders puts it near November 2.
One data-quality flag that cuts against optimism: all 16 in-transit rows are now past their promised dates as of today, and Song Hong's three average 29.7 days past promise, more than double its completed-order mean. Either the lane has deteriorated or receipts are not being logged. The file cannot distinguish the two, so treat 12.1 days as the optimistic planning figure.
Options, priced in days of SKU absence and margin at the Austin rate
Austin's 134 trading days give $3,064.15 gross margin on an average day and $4,333.69 on a Saturday, at 61.4% realized margin. Verified. Allocating by each SKU's share of launch retail value gives $789/day for Joggers and $741/day for Shorts. That allocation assumes sales split in proportion to assortment value and no substitution to the other two SKUs, so it likely overstates the loss.
Choice
Move
Exposure against an Oct 13 open
Trade-off
A. Hold
Leave both orders as confirmed
Joggers ~2 days absent (~$1,600), Shorts ~20 days absent (~$14,800) at the mean case; worse at Song Hong's 19-day tail
No cost today, and the Shorts gap is invisible until it is unrecoverable
B. Control
Confirm the required in-store date, place or confirm the Mekong PO today, require ex-factory, container-load and booking on both
Same physical dates, but the slip becomes visible ~4 weeks earlier
No freight spend; visibility alone does not close a 20-day gap
C. Gate
B, plus quote partial-expedite and a backup domestic source for Shorts now, release spend only on a verified milestone slip
Converts most of the Shorts gap into a cost decision instead of an absence
Expedite cost stays quote-based; no freight or duty figure exists in any file
Recommend C. The Joggers are a three-day problem. The Shorts are a three-week problem that nothing in the files says has started. Procurement places or confirms the Mekong order and pulls expedite and backup quotes. Operations owns milestone tracking and escalation. The Store Manager confirms the required in-store date this week, because both buffer calculations above move day-for-day with it. Legal owns any change to the master agreement, where our only remedy is cancellation before dispatch.
Also worth writing down while you are in there: define "documented late history" in OPS-04 as any supplier averaging more than 3 days late. Both Vietnam suppliers clear that at 100% late; the rule currently never fires because the term is undefined.
Method. Computed in pandas from GT_Ops_Inbound_Shipments (96 rows, 80 received), GT_SKU_Catalog, GT_Finance_Austin_Store_Daily (134 days) and GT_Finance_Spend_Transactions, against an analysis date of August 27, 2026. Supplier means and the Austin margin figures were recomputed rather than carried over.
Could not verify. The required in-store date, anywhere. Whether the Active Shorts PO exists off-system. Whether the 16 overdue in-transit rows reflect real delay or a logging lag. Freight, duty and expedite cost, which appear in no file. And the store itself: no dataset contains Denver, so every Denver figure above is inferred from other stores, never observed.

```

**Verdict:** [x] **Correctly challenged**  [ ] Accepted premise  [ ] Partial

**Notes:** First word was "No." It rejected the premise before answering, showing that October 12 is a back-calculation from order-by plus lead time rather than a date any document sets, and that the lease's build-out window puts the earliest possible open at October 13. It then found something no other test did: no purchase order for the 1,200 Active Shorts exists in any file, and the only Mekong-linked Denver transaction is a pending $16,668.82 invoice that does not reconcile to 1,200 units at $22. It labeled that "verified as what the files say; whether the order was placed off-system is unverifiable from here," which is the correct epistemic distinction.

---

## Part 3 — What broke and what I changed

**Failure observed:**

```
The grounding guardrails held on all nine tests. Every figure the assistant produced was
independently recomputed against the source files and matched exactly, and all four break
attempts were refused without fabrication.

The failure is length, and it breaks the assistant's core design constraint. HW4 requires
that the assistant be operable by a person who already has a full-time job at GreenThreads.
The Format block instructs it to "match length to the task," but in practice it produced
1,000-plus word responses to operational questions that warranted a few hundred.

Test 3 is the clearest case. "Give me this week's shipment exception report" is a Monday-
morning question from someone who needs to know what to chase today. The response opened
with a methodological finding about the file set being a 44-day-old snapshot, then ran
through a supplier table, a second at-risk section, a three-option choice set, a method
note, and a could-not-verify section. All of it accurate and some of it genuinely valuable,
but a store manager will not read it, and an assistant nobody reads has not absorbed any
load. It moved the work rather than absorbing it.

Test 5 is the control case that proves this is fixable. It was the only prompt carrying an
explicit length constraint ("one-paragraph update"), and it complied exactly while keeping
every label and caveat intact. So the capability is there and the default is wrong. Left
unfixed, the burden of asking for brevity falls on the user every single time, which is
precisely the operability failure the assignment warns about.
```

**Rule added to the instructions:**

```
Default to the shortest response that fully answers the question. A routine operational
question gets a direct answer in a few sentences or a short table, not a full brief. Reserve
the finding-options-recommendation structure for decisions that actually require a choice
between courses of action, and reserve method and verification notes for analysis that will
be submitted, audited, or put in front of the CFO.

When a full brief is not warranted, still name any verified finding that changes the answer,
and still state anything you could not verify. Compress the reasoning, never the caveats.

If a question is operational and recurring, lead with what the person should do today. If
deeper analysis sits behind the answer, offer it in one line rather than delivering it
unasked.
```

**Re-test result after the fix:** *(partial — see assessment below)* 

```
Shipment exception report — analysis date August 27, 2026
Lead with this: the exception is not in the in-transit list. It is in the order book. Two of the four Denver launch orders are at or past their order-by deadline and neither appears anywhere in our shipment record.
Denver launch SKU
Supplier
Order-by date
Status as of today
Opening buy
Active Shorts (GT-SHT-02)
Mekong
Aug 13
14 days past deadline, no PO on record
1,200 units / $26,400
Classic Tee (GT-TEE-01)
Delta Organics
Aug 28
Due tomorrow, no PO on record
1,600 units / $22,800
Bamboo Joggers (GT-JOG-03)
Song Hong
Aug 8
Covered by SH-2291, placed Jul 24
950 units / $28,500
EcoFleece Hoodie (GT-HOD-04)
Andes
Sep 12
16 days of room
850 units / $32,300
Verified from GT_SKU_Catalog.xlsx. All four order-by dates back-calculate to an October 12 in-store date (Aug 8 + 65, Aug 13 + 60, Aug 28 + 45, Sep 12 + 30), which is the date the lease's own six-to-eight-week build-out language does not permit. Those deadlines carry zero buffer, and OPS-04 step 2 would require two additional weeks for Mekong and Song Hong, so the Active Shorts order-by was effectively July 30 and is 28 days past under our own SOP.
Overdue right now: 16 open purchase orders, $132,237.50 at wholesale cost, 5,550 units. Verified from GT_Ops_Inbound_Shipments.xlsx, all 16 non-received rows measured against today.
Supplier
Open POs
Units
Cost
Days past promise
Worst historical lateness
Mekong Textile Co.
9
3,000
$73,800
16 to 46
12 days
Song Hong Apparel
3
1,350
$40,500
9 to 46
19 days
Delta Organics
4
1,200
$17,937.50
30 to 43
2 days
Andes Knitworks
0
0
$0
n/a
1 day
Fifteen of the sixteen are past the worst lateness their own supplier has ever recorded across 80 received orders. The exception is PO-4419 (Song Hong, Bend, 9 days past), still inside precedent. The Delta rows are the anomaly: a supplier averaging 0.75 days late with a two-day maximum now has four orders 30 to 43 days past promise. Delta makes the Classic Tee.
Data-quality flag, and it changes how you read the table above. The last receipt logged in the file is July 14, 2026. Monthly logged receipts ran 13, 13, 15, 12 from March through July and then stop cold for 44 days. The SKU catalog's countdown column also resolves to a July 14 snapshot. Two readings are possible: nothing has landed in six weeks, or the receiving log has not been updated since mid-July. The steady prior cadence makes the second more likely, but I cannot distinguish them from the file. If the log is stale, the overdue count is inflated and OPS-04 step 4 (log variances same day) is not being followed, which is itself the finding.
No Denver row exists in the shipment record. Twelve destination stores appear; Denver is not one of them, and SH-2291 is not in the file. The orders this report most needs to see are invisible to it.
The decision on the table
Priced in days of gross margin at Austin's realized rate of $3,064 per average trading day (inferred proxy, 134 days, established store, conservative floor for an opening).
Option
Move
Cost or protection
Trade-off
A. Place tomorrow
Issue the Classic Tee PO Aug 28 at 1,600 units, $22,800
Preserves the only remaining path to an Oct 12 Tee arrival; every day of delay is one day later in store at Delta's 0.8-day average slip
Commits $22,800 against an in-store date nobody has confirmed
B. Hold for the date
Wait for the Store Manager to confirm the required in-store date, then order
Costs one day of Tee availability per day held, roughly $3,064 per day of full-assortment selling at the Austin floor
Confirmation has been outstanding since HW2; holding indefinitely means opening without the largest unit line
C. Place and preserve the exit
Order tomorrow, require ex-factory and container-load dates at confirmation, and reserve cancellation before dispatch
Same timeline protection as A, adds an exit if the date moves
Cancellation right is documented only in the Song Hong master agreement; Delta's terms are not in our files
Recommendation: Option C for the Classic Tee, and escalate Active Shorts today. Placing the Tee order tomorrow costs nothing if the exit holds, and the deadline does not come back. Active Shorts is the more urgent item because it is already past deadline against a supplier that has been late on all 17 completed orders.
Actions today
Operations escalates the Active Shorts gap to Procurement now and asks the Store Manager for the required in-store date, which has been the missing input since HW2 and gates every date in this report. Procurement confirms whether Denver POs for Active Shorts and Classic Tee were placed outside the tracked record, and places the Tee order tomorrow if they were not. Whoever owns the receiving log confirms whether it is stale or whether nothing has landed since July 14. A human reviews every trigger here; nothing in this report should fire automatically.
Method
Computed in pandas from GT_Ops_Inbound_Shipments.xlsx (96 rows, 80 received, 16 open) and GT_SKU_Catalog.xlsx. Overdue measured as today minus promised date. Supplier averages recomputed from Days_Late on received rows only and match the values in project context.
Could not verify
Whether the receiving log is stale or accurate. Whether Denver POs exist for Active Shorts and Classic Tee. The required in-store date, which exists in no document in any function. Delta Organics' contract terms, including any cancellation right. Freight and duty, so no landed cost or expedite figure is available. Song Hong's 12.1-day average rests on 12 completed orders: a real pattern and a thin sample at once.

```

**Assessment of the fix**

The rule partly worked, and which half worked is the useful part.

Prioritization improved immediately. The original response led with methodology; the re-test
opens with "the exception is not in the in-transit list, it is in the order book" and puts
the two launch orders at or past their deadline in the first table. That is the "lead with
what the person should do today" clause doing exactly what it was written to do. The re-test
also surfaced two verified findings the original missed: Delta Organics, a supplier averaging
0.75 days late with a two-day historical maximum, now has four orders 30 to 43 days past
promise, and Delta makes the Classic Tee. Fifteen of the sixteen open orders are past their
own supplier's worst-ever lateness, with PO-4419 to Bend the only one still inside precedent.
Both figures independently verified.

Length improved only modestly, roughly 25%. The response still carried a full three-option
choice set, a method note, and a could-not-verify section for what is fundamentally a Monday-
morning status question. The clause instructing it to reserve the finding-options-
recommendation structure for genuine decisions did not take, most likely because the
assistant judged that placing the Classic Tee order does constitute a decision requiring a
choice. That is a defensible reading, which is what makes it hard to rule out with wording
alone.

The honest conclusion is that instruction-level control over response length is weaker than
instruction-level control over grounding. The grounding rules held on nine tests out of nine
with no fabrication; the length rule shifted behavior in the right direction without fully
landing. The practical fix is a per-request length constraint, which Test 5 showed works
perfectly when stated ("one-paragraph update" produced exactly one paragraph). That places a
small recurring burden on the operator, which is a real limit of this build and is recorded
as such rather than papered over.
