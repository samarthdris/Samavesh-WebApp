# Fellow Interface Dashboard Cards — Design

**Date:** 2026-08-20
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` only — the Fellow home screen (`#f-home`, the KPI grid at line 2252).
**Source:** `New Feedbacks/Point_08_Fellow_Interface_Cards.md` ("Point 8 — Fellow Interface Cards Changes", Module 7 in the master FRD — Fellow portion only). In Scope, **Medium**. Definitions confirmed with Shweta on 2026-08-20.

## Problem

A Fellow has no single view of their own workload. The Fellow home currently shows **4** stat cards — *My students 24 · Documents Pending 7 · Applications Submitted 12 · Approved 5* (line 2253) — plus a 4-tile caseload triage summary and an attendance card. The client has now confirmed a **12-card** list, so 8 cards are genuinely new and the existing 4 need to fold into the new set rather than sit beside it.

**Correction to the client note.** The document says the card set was "discussed but not finalized" and describes the current state as a draft; in fact 4 cards are already built and live. This is an extend-and-absorb, not a fresh build.

## Two claims in the document that the repo already settles

1. **"Cards 4, 6 and 10 are blocked on Module 3's unresolved status values."** They are not. The application status set is locked in this wireframe — `CASES`/`CASE_STATUS` at line 4843: *Eligibility Identified · Documents Pending · Ready to Submit · Under Scrutiny · Application Approved · funds awaited · Benefits Received · Re-apply · Rejected*. All three cards are buildable now.
2. **"Cards 5 and 7 can use Module 1's confirmed 5-value document status set, where completed = Submitted to Scholarship."** That set is not what this build has. The wireframe's document statuses are **Accepted · Pending · Rejected · Under Review** — neither "Submitted to Scholarship" nor "Follow-up in Progress" appears anywhere in the file. This spec therefore defines document-case completion as **Accepted**, the wireframe's terminal document status, and flags the mismatch for the client.

## Confirmed definitions (Shweta, 2026-08-20)

| Term | Definition |
|---|---|
| Application **completed** | status = **Benefits Received** — the money actually reached the student |
| Application **in process** | Ready to Submit · Under Scrutiny · Application Approved · funds awaited |
| Application **rejected** | status = Rejected (its own card; Re-apply counts as neither completed nor in process) |
| Document case **completed** | document status = Accepted |
| Document case **in process** | Pending · Under Review |
| **Funds unlocked** | SUM of Financial Tracking amounts whose status is **Funds Disbursed** — note the real status value, not the feedback doc's "Disbursed" |

## The 12 cards, and where each number comes from

Eight of the twelve are **computed live** from `CASES` (filtered to `CASE_FELLOW`, i.e. the logged-in Fellow) and from Module 5's `FIN` object — so they move when the Fellow actually changes something, rather than being hardcoded:

| # | Card | Source |
|---|---|---|
| 1 | Students assigned | live — distinct students in `CASES` for this Fellow (keeps the existing "My students 24" figure as the demo baseline) |
| 2 | Applications assigned | **live** — count of this Fellow's cases |
| 3 | Document support cases assigned | mock (`FDASH` object — no document-case array exists in the wireframe) |
| 4 | Applications completed | **live** — status = Benefits Received |
| 5 | Document cases completed | mock |
| 6 | Applications in process | **live** — the three in-process statuses |
| 7 | Document cases in process | mock |
| 8 | Total funds unlocked | **live** — sums `FIN` entries with status "Funds Disbursed", plus a mock baseline for the Fellow's other students |
| 9 | Total documents supported | mock |
| 10 | Rejected applications | **live** — status = Rejected |
| 11 | Pending beyond N days | mock ages + a **configurable threshold** select (7 / 15 / 30 days) that re-renders the card — the document's acceptance criterion says this must not be hardcoded |
| 12 | Month-over-month completions | mock — 6 compact bars, this Fellow's own completions per month |

Card 8 deliberately reads from `FIN`: recording a disbursement in the Financial Tracking tab visibly raises this card, which demonstrates that the two modules are one system rather than two screens.

## Non-goals

- No Admin dashboard cards — the document says those remain a separate pending item.
- No change to the caseload triage tiles or the attendance card; they answer a different question and stay.
- No real status-change history. Cards 11 and 12 need timestamps the wireframe doesn't record; they use mock ages, and the production prerequisite (Frappe `track_changes: 1`) is documented, not faked as solved.
- No Frappe build.

## UI

- The existing 4-across KPI grid becomes a responsive **12-card grid** (`repeat(auto-fit,minmax(150px,1fr))`, wrapping to 6×2 or 4×3 by width), reusing the existing `.card.stat` component so it looks native and inherits the stagger animation.
- Cards read as plain counts with the existing icon treatment — no colour-coded status chips, per the document's "minimal visual style" note.
- Card 11 carries the threshold select in its own card body; changing it re-renders that card only.
- Card 12 renders 6 slim bars with month initials, reusing the funnel bar colour so it reads as one visual family.
- Grid sits where the current KPI grid sits (line 2252), above "My caseload" — the Fellow sees workload totals first, then triage.

## Production note (Frappe)

12 aggregate queries scoped by `frappe.session.user` → assigned-Fellow lookup, over Student, Scholarship Application, Application Document and Disbursement Entry. Cards 11–12 need `track_changes: 1` on Scholarship Application and Application Document so Frappe's Version doctype holds the status-change history — the **same prerequisite** the MIS dashboard's turnaround-time indicators need, so it's one enablement serving both. Consider a short cache only if 12 concurrent aggregates become noticeable; not needed at expected volumes.

## Verification (render-verify)

1. `?role=fellow&screen=f-home` — 12 cards render, none blank, no "0" where the caseload plainly has rows.
2. Cross-check the live cards against the caseload: `#f-applications` counts for this Fellow must equal cards 2, 4, 6 and 10 exactly.
3. Change a case's status in the caseload (Record disbursement / Reopen) → return to home → the affected live cards have moved.
4. Record a First Installment in Financial Tracking → card 8's total rises by that amount.
5. Card 11: switch the threshold 7 → 15 → 30 and confirm the count changes each time.
6. Confirm no card shows another Fellow's data — `CASES` entries for Sandip K / Dhanashree O must be excluded from every count.
