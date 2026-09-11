# GreenThreads Operations AI Analyst

**Lucas Diverdi** · AI.205: AI Integration in Business I · Prof. Jeff Eyet · Summer 2026

I spent this quarter as the Operations lead on a consulting team hired by GreenThreads, a sustainable apparel company opening its thirteenth store in Denver with 90 days on the clock and no new corporate headcount. My job was to find where AI could absorb Operations work nobody had time to do, then build something that actually does it.

This repository holds every deliverable from that engagement, from the first map of the function to a working AI assistant and the executive brief that recommends a decision to the CEO and CFO.

> **Start here:** [HW5 Executive Brief](HW5_Executive_Brief.pdf) · [HW4 assistant overview](HW4_README.md) · [The assistant's test results](testing/)

---

## The short version

The Operations goal all quarter was getting four launch products onto the Denver sales floor on time. Across four assignments the same lesson kept surfacing: **a promised date is not a plan.**

The supplier's confirmation letter said it expected no issues. The contract behind it called that date a good-faith estimate with no penalty for missing it. The shipment data showed both Vietnam suppliers late on every completed order. Four departments were planning to an October 12 opening that no document ever set and the lease made impossible.

The assistant I built in HW4 surfaced the finding that shaped the final recommendation: no purchase order for one of the four launch products existed anywhere, two weeks after its order deadline had passed. The client later confirmed the order had been missed.

## What's in here

| Deliverable | What it is | Built by |
|---|---|---|
| [HW1 · Functional Brief](PASTE_HW1_LINK) | Map of the Operations function, workflow cadences, and seven AI opportunities ranked by the MIT Sloan levels | Team: Isra Denaguir, Ivette Bruce, Chris Sudyka, Lucas Diverdi |
| [HW2 · Document Intelligence Brief](PASTE_HW2_LINK) | Cross-source synthesis of the supplier PO, receiving SOP, and master supply agreement, with every claim traced to a quoted passage | Team (same as above) |
| [HW3 · Data Intelligence One-Pager](PASTE_HW3_LINK) | One-page executive recommendation built on 96 purchase orders and 134 days of store sales, plus the [working analysis workbook](PASTE_HW3_WORKBOOK_LINK) | Lucas Diverdi |
| **HW4 · Custom AI Assistant** | The GreenThreads Operations AI Analyst: [overview](HW4_README.md), [instructions](instructions/), [knowledge files](knowledge/), and [nine documented tests](testing/) | Lucas Diverdi |
| [HW5 · Executive Brief](HW5_Executive_Brief.pdf) | The quarter told as one story, ending in a decision for the CEO and CFO ([live Google Doc](PASTE_HW5_LINK)) | Lucas Diverdi |

## How the assistant works

The assistant is a Claude project holding 25 source files: the case brief, Operations shipment and catalog data, the supplier documents, the earlier deliverables, and the Finance, HR, and Marketing datasets that feed Operations decisions. Its instructions follow the Persona, Task, Context, Format structure, and its rules come straight from what went wrong earlier in the quarter.

- **It labels every claim** as verified, inferred, or unverifiable, and keeps the label attached every time the claim appears.
- **It computes instead of estimating.** If a number is not in the files or derivable from them, it says the number does not exist.
- **It stays in its lane.** Operations is the default. Another function's data is an input, never a recommendation, unless the chat opens with a declared mode such as `MODE: HR`.
- **It challenges false premises.** Asked to confirm buffer against an October 12 opening, its first word was "No."

## What testing showed

I ran nine tests in fresh chats: five realistic Operations tasks and four deliberate attempts to break it.

**What held.** Every number across all nine responses was recomputed against the source files and matched. It refused all four break attempts without inventing anything, including a request for a freight cost that appears in no file. It also caught flaws in our own earlier work, like a supplier average that only counts orders that eventually arrived.

**What broke.** It would not stay short. A Monday-morning shipment question got over a thousand words that opened with methodology. I added a rule to lead with the action and keep answers brief; prioritization improved right away, but length only dropped about 25%. My conclusion: instruction-level control over grounding is much stronger than instruction-level control over length. The reliable fix is stating a length in the request, which worked every time.

**What a person still owns.** Anything that releases money, changes a supplier relationship, or goes in front of the CFO. The assistant calculates and flags. People decide. It also only knows what its context file tells it, so someone has to keep that file current. The clearest example came in HW5, when its draft recommended paying to air-freight a late product and a simple review question showed the math didn't support it.

## Tools

Claude (Opus, reasoning tier) in a Claude project for document analysis and the assistant; ChatGPT (reasoning tier) for parts of HW1 and HW3; Python with pandas to recompute every figure independently; Google Sheets for the reproducible workbook.

## A note on data

The `knowledge` folder holds the GreenThreads case pack exactly as the assistant uses it. It is fictional course material. In a real engagement, files like the applicant records and the signed supplier agreement would never sit in a public repository, and the governance section of my HW4 write-up explains who should be allowed to see them.

## About me

I run the support team and train customers at BuildingPoint Midwest & Gulf Coast, a Trimble construction technology dealer, and I'm finishing an Associate's degree in Marketing at Campus. Outside of class I've taught myself the Claude API, retrieval-augmented generation, and MCP servers, mostly by building tools for the support work I do every day.

What I'd bring to a team adopting AI is the part this repository tries to show: the tools do the reading and the arithmetic, and the job is knowing which date was never real, which number is a floor, and which question belongs to someone else.

