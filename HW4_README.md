GreenThreads Operations AI Analyst

A custom AI assistant built for the Operations function of GreenThreads, a sustainable apparel company opening its thirteenth store in Denver. Built as a Claude project (platform approved by the instructor in place of ChatGPT).

Course: AI.205 AI Integration in Business I · Prof. Jeff Eyet Author: Lucas Diverdi Assignment: Homework #4 — Custom AI Assistant Build

What this assistant does

GreenThreads hired a five-person consulting team to find where AI can absorb work the company has no headcount to do. This assistant is that analyst, built.

It handles the recurring Operations work behind the Denver launch: checking shipments against promised dates, testing purchase orders against the receiving SOP, pricing what a delay costs in margin terms, preparing receiving for what will actually arrive on the dock, and writing decision-ready updates for the CFO.

The design constraint that shaped everything: GreenThreads is not hiring anyone to run this. It has to be usable by a person who already has a full-time job there. So answers lead with the finding and the action rather than the methodology, and the user is never required to know the file structure or phrase a question a particular way.

The multi-lane design

The project holds data from four functions — Operations, Finance, HR, and Marketing. Project instructions apply to every chat equally, so the assistant cannot detect which "department chat" it is in. Instead the lane is declared by the user in the first message:

No declaration → Operations lane (the default).
MODE: Finance, MODE: HR, MODE: Marketing A, MODE: Marketing B → that lane.

In the Operations lane, other functions' data is an input to Operations decisions only. The assistant will not recommend how HR should fix hiring or how Finance should code spend. If asked, it names which lane owns the question and offers to switch. This keeps cross-functional data useful without letting the assistant wander outside its remit.

Instructions (Persona, Task, Context, Format)

Full text: instructions/PTCF_instructions.md

Summary of each block:

Block	What it establishes
Persona	An outside consultant, not an employee. Home function is Operations. Writes for Jennifer the CFO, who asks how you know, rather than Marcus the CEO, who wants good news.
Task	The lane rule, the grounding rules (compute from files, never invent a number, label every claim, say when unsure, treat a missing input as a finding), the decision format, and the operability constraint.
Context	The verified state of the engagement: the unestablished opening date, supplier reliability figures with their sample sizes, launch economics, governing contract terms, unresolved conflicts, and decision rights.
Format	Finding first with its inputs, then options, then recommendation and owner. Forward-looking figures labeled as planning inferences. Closes with what could not be verified.
Knowledge files

Twenty-five files uploaded to the project, all published in knowledge/ so the build can be inspected or rebuilt. 00_PROJECT_CONTEXT.md is read first and acts as the source of truth; the raw datasets are what the assistant computes from.

GreenThreads is a simulated company and all data is coursework material, so the full set is published. In a real engagement the applicant file and the executed supply agreement would stay in an access-controlled environment, for the reasons set out in the governance section below.

Context and case

00_PROJECT_CONTEXT.md — every verified figure, the timeline conflict, contract terms, known limits
GreenThreads_Case_Brief.md — the engagement brief, mandate, budget, and fixed facts

Operations (home function)

GT_Ops_Inbound_Shipments.xlsx — 96 purchase orders, promised vs. actual
GT_SKU_Catalog.xlsx — 8 SKUs, suppliers, lead times, costs, order deadlines
Sources_Operations.md — PO confirmation SH-2291, SOP OPS-04, master supply agreement

Cross-functional inputs

GT_Denver_Lease_Week03.md, GT_Finance_Austin_Store_Daily.xlsx, GT_Finance_Denver_Budget.xlsx, GT_Finance_Spend_Transactions.xlsx, Sources_Finance.md
GT_HR_Denver_Applicants.xlsx, GT_HR_Sales_Associate_Offer_Letter_Week03.md, Sources_HR.md
GT_MarketingA_Channel_Performance.xlsx, Sources_MarketingA.md
GT_MarketingB_Customers.xlsx, Sources_MarketingB.md
GT_Portland_Store_Daily.xlsx

Prior work (HW1–HW3)

HW1_Functional_Brief.docx — function map and ranked AI opportunities
HW2_Synthesis_Brief.docx — multi-source document analysis and verification
HW3_OnePager_v2.docx, HW3_Analysis_Workbook_v2.xlsx — data analysis and executive recommendation

Assignment materials

AI205_HW1_Instructions.pdf, AI205_HW2_Instructions.pdf, AI205_HW3_Instructions.md
Guardrails

The guardrails come directly from what went wrong in earlier assignments.

From HW#2 (documents): the model produces the next plausible sentence rather than retrieving a fact. Asked to summarize contract terms in general language, it supplied the penalty and on-time clauses a supply agreement usually contains instead of the ones §6 actually lacks. The absence was the finding. Rule added: answer from the files, and write that something is not in the document rather than filling the gap.

From HW#3 (data): the risk is a confident wrong number. Rules added: never produce a figure that is not in the files or derived by a calculation you can show; label every claim verified, inferred, or unverifiable; qualify figures to their sample size.

From HW#3 grading feedback: points were lost for options not priced in a common unit and for claims stated more confidently than the evidence supported. Both are now standing rules — every option set is priced in one unit, and confidence is calibrated to evidence (twelve out of twelve late is a real pattern and a thin sample at once, and saying both is the standard).

Testing

Full prompts, expected outcomes, and verbatim results: testing/test_plan_and_results.md

Nine tests were run: five realistic Operations tasks and four deliberate attempts to make the assistant fail.

Realistic tasks
Buffer compliance check against OPS-04
Pricing a Bamboo Joggers delay in margin terms
Weekly shipment exception report
Receiving carton prep
One-paragraph readiness update for the CFO
Break tests
A number that does not exist — freight cost per unit (absent from every file)
Data that cannot answer the question — Denver conversion rate (no Denver in any dataset)
Outside the lane — how to fix HR's 39% offer-accept rate
A false premise stated confidently — "since the store opens October 12, confirm we have buffer"
Results

The assistant passed all nine tests on grounding. Every quantitative claim across all nine responses was independently recomputed against the source files and matched exactly, including figures it derived on its own initiative. It refused all four break attempts without fabricating a single number.

Several responses went beyond the pass condition. Test 2 caught a flaw in my own prior analysis: the 12.1-day supplier average is censored, computed only from orders that eventually arrived, while three Song Hong orders sit 9, 34, and 46 days past promise and are excluded from it. Test 4 found that the shipment record has no received-quantity or carton-count field, so the SOP's variance-logging requirement has nowhere to land. Break 2 ran unrequested data-quality work on the Portland file and surfaced a duplicate row, a blank returns value, a $480 arithmetic error, and a promo-day outlier at 4.7x normal volume.

What broke and the rule I added

The failure was length, not accuracy. The assignment's design constraint is that the assistant be operable by someone who already has a full-time job at GreenThreads. The Format block instructs it to match length to the task, but in practice it produced 1,000-plus word responses to operational questions that warranted a few hundred. Test 3 is the clearest case: "give me this week's shipment exception report" is a Monday-morning question, and the response opened with methodology and ran through a supplier table, an at-risk section, a three-option choice set, a method note, and a could-not-verify section. Accurate throughout, but a store manager will not read it, and an assistant nobody reads has moved the work rather than absorbed it.

Test 5 was the control case proving this was fixable: it carried the only explicit length constraint ("one-paragraph update") and complied exactly while keeping every label and caveat intact.

Rule added to the grounding rules in the Task block:

Default to the shortest response that fully answers the question. A routine operational question gets a direct answer in a few sentences or a short table, not a full brief. Reserve the finding-options-recommendation structure for decisions that actually require a choice between courses of action, and reserve method and verification notes for analysis that will be submitted, audited, or put in front of the CFO. When a full brief is not warranted, still name any verified finding that changes the answer, and still state anything you could not verify. Compress the reasoning, never the caveats. If a question is operational and recurring, lead with what the person should do today.

Re-test result: partial fix. Prioritization improved immediately. The re-test opened with "the exception is not in the in-transit list, it is in the order book" and led with the two launch orders at or past their deadline, which is the new rule's final clause working as written. It also surfaced two verified findings the original had missed. Length improved only about 25%, and the choice set and method notes remained, most likely because the assistant judged that placing the Classic Tee order genuinely constitutes a decision requiring a choice.

The honest conclusion: instruction-level control over response length is weaker than instruction-level control over grounding. The grounding rules held nine times out of nine with zero fabrication; the length rule moved behavior in the right direction without fully landing. The reliable fix is a per-request length constraint, which places a small recurring burden on the operator and is recorded here as a real limit of this build rather than papered over.

Governance

Who may see the output. This assistant holds an executed supply agreement, unit-level supplier pricing, a signed lease, and an applicant file with individual candidate records. Contract terms and supplier pricing belong to Procurement and Legal; applicant data belongs to HR. The Operations user should not redistribute either. GreenThreads has no AI policy today and no analytics function, which is the gap this design closes rather than assumes.

Who is accountable when it is wrong. The person who acts on the output, not the assistant and not the person who built it. The assistant calculates and flags; the Operations Manager owns escalation, Procurement owns spend and sourcing, Legal owns contract terms, and the Store Manager confirms the required in-store date. No purchase order is placed, cancelled, or expedited on an AI flag alone.

Where a human must check. Any trigger that releases money, any change to a supplier relationship, and any figure going in front of the CFO. The assistant's numbers are reproducible from the workbook, so verification is a spot-check against the source rather than an act of faith.
