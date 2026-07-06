# Unified Applications Caseload — Design (Fellow + Admin)

**Date:** 2026-07-06
**Status:** Approved (design) — awaiting spec review
**Scope:** `wireframe/index.html` (single-file wireframe). Fellow section **and** Admin section.
**Related memory:** `samavesh-cases-tasks`, `samavesh-status-model`, `samavesh-use-only-context-terms`, `samavesh-ripple-check`, `samavesh-no-half-baked`, `samavesh-render-verify`, `samavesh-git-workflow`.

## Problem

A student × scholarship is tracked on **two independent status axes** that the Fellow must maintain separately:

- **Application status** (FORM vocabulary): Ready to Submit / Under Scrutiny / Application Approved / Benefits Received / Re-apply / Rejected — shown in the **Applications** screen (`#f-applications`).
- **Work-state** (Cases module): To Do / In-progress / On Hold / Completed / Discarded — shown in the **My Cases** screen (`#f-tasks`).

Both are the *same entity* (a "Case" is defined as "one Student × one Scholarship — a single application bundle"). The Cases table already prints the application status inside its Journey Stage column (e.g. `Submitted · Under Scrutiny`). This creates two problems for the Fellow:

1. **Two rail tabs that read as duplicates** (Applications and My Cases are both lists of student×scholarship rows with status chips).
2. **Double status maintenance** — the Fellow updates the application status *and* separately advances the work-state for the same reality (e.g. a disbursed application must also be hand-marked "Completed").

Analysis: of the 5 work-states, **three (To Do / In-progress / Completed) are derivable** from the application's own progression. Only **On Hold** and **Discarded** carry information the application status cannot infer.

## Goals

- One rail item, one screen, per role (remove the duplicate).
- **One status field** the Fellow advances — ideally as a byproduct of the real action they already take.
- Preserve the genuinely-useful bits of the Cases module: the "what needs my action today" triage, and the On Hold / Discarded exception tracking (with reasons).
- Reuse existing vocabulary; invent no new domain terms.
- Keep Fellow and Admin consistent (apply the same merge to both).

## Non-goals (YAGNI)

- No kanban / drag-and-drop board.
- No changes to the **student** portal (students never see cases).
- No renaming of the FORM application-status terms beyond merging the two axes.
- No change to the MIS Dashboard funnel logic (it already reports journey stages); only the Admin rail label for the merged screen changes.

## The model

### 1. One status ladder (single source of truth)

A student × scholarship carries **one** ordered status, assembled entirely from vocabulary already in the wireframe:

| # | Status | Origin | Terminal |
|---|---|---|---|
| 1 | Eligibility Identified | journey stage | no |
| 2 | Documents Pending | student-list term | no |
| 3 | Ready to Submit | FORM app-status | no |
| 4 | Under Scrutiny | FORM app-status | no |
| 5 | Application Approved · funds awaited | FORM app-status | no |
| 6 | Benefits Received | FORM app-status | **yes (success)** |
| 7 | Re-apply | FORM app-status | no (loops back to action) |
| 8 | Rejected | FORM app-status | **yes** |

The status advances as a byproduct of the real action wherever one exists (Submit & record → Under Scrutiny; Record disbursement → Benefits Received). Where no natural action exists in the wireframe, a single status control advances it. There is **no second field** to keep in sync.

### 2. Two exception flags (the only manual, non-derivable states)

Layered **on top** of the status as a badge; both reuse existing terms:

- **On Hold** (+ reason) — work paused (family unreachable, document delay, student travel). Toggle on and off; when cleared, the row returns to its status-derived triage.
- **Discarded** (+ reason) — no longer pursued (withdrew, missed deadline, switched scheme). Terminal, reopenable.

To Do / In-progress / Completed are **removed as manual states**.

### 3. Derived triage (the former work-state, now a computed filter — never a stored field)

Echoes the document "whose-turn" chips already shipped (Action needed / With reviewer / Complete):

| Bucket | Rule |
|---|---|
| **Action needed** | status ∈ {Eligibility Identified, Documents Pending, Ready to Submit, Re-apply} and not On Hold |
| **Awaiting** | status ∈ {Under Scrutiny, Application Approved · funds awaited} and not On Hold |
| **Done** | status ∈ {Benefits Received, Rejected} or Discarded |
| **On Hold** | the On Hold flag is set (takes precedence over the status-derived bucket) |

## Fellow experience

### Screen: `#f-applications` (merged; `#f-tasks` retired)

- **Page header:** eyebrow "Caseload", title **"Applications"**, subtitle clarifying it is the Fellow's full set of student×scholarship cases across the whole journey. Keep the existing **"+ New Scholarship Entry"** primary button.
- **Filter chips = triage buckets:** `All (N)` · `Action needed (N)` · `Awaiting (N)` · `On Hold (N)` · `Done (N)`. (These replace both the old application-status chips and the old work-state chips.) A secondary **status** filter (dropdown) is available for granularity but is optional to use.
- **List — one row per student × scholarship:**
  - Student avatar + name
  - Scholarship (logo + name) + App ID
  - **Status pill** (the single status)
  - **On Hold / Discarded badge** when the flag is set (amber "On Hold — <reason>" / grey-red "Discarded — <reason>")
  - Last-updated date
  - **One context-aware primary action** (the single status-advancing action):

    | Status | Primary row action |
    |---|---|
    | Eligibility Identified | Start → Collect documents (→ Documents Pending) |
    | Documents Pending | Open documents (deep-links to the student's Documents tab) |
    | Ready to Submit | Submit & record (→ Under Scrutiny) |
    | Under Scrutiny | Update outcome → choose Approved · funds awaited / Re-apply / Rejected |
    | Application Approved · funds awaited | Record disbursement (→ Benefits Received) |
    | Re-apply | Re-apply (→ Ready to Submit) |
    | Benefits Received / Rejected | — (Reopen available in ⋯) |

  - **Overflow ⋯ menu on every row:** *Put on hold / Resume* (with reason prompt shown inline, not `prompt()`), *Discard case* (with reason), *Reopen* (for terminal/discarded). These are the only manual exception actions and never clutter the primary flow.

### Ripple — Fellow

- **Dashboard `#f-home`:** the "My Cases inbox" summary cards re-label to the triage buckets (**Action needed / Awaiting / On Hold / Done**) and deep-link into `#f-applications` pre-filtered.
- **`#f-student` → Applications tab:** the per-student application cards keep their status control + Record-disbursement + correction-note, aligned to the one status ladder; add the On Hold / Discard actions here too. Any "Mark Complete" work-state control is removed.
- **Rail:** the **My Cases** entry (and its count badge) is removed; **Applications** is the single entry. Its count badge reflects "Action needed" (the actionable subset), matching the triage-first framing.

## Admin experience

The Admin has the same dual model: `#a-tasks` (Cases & Tasks) plus application status seen in `#a-student` detail and MIS aggregates. Apply the same merge, oversight-flavored.

### Screen: `#a-tasks` → relabelled **"Applications"** (cross-Fellow oversight)

- **Rail:** the WORKSPACE **"Cases & Tasks"** entry becomes **"Applications"** (same slot, same count badge semantics = actionable subset across all Fellows).
- **Page header:** eyebrow "Oversight", title **"Applications"**, subtitle "All student × scholarship cases across your Fellows."
- **Filters:** the same **triage-bucket chips** (All / Action needed / Awaiting / On Hold / Done) **plus** the existing **Fellow** dropdown and a **status** dropdown. (The old 5 work-state summary cards are replaced by the triage buckets.)
- **List/table — one row per student × scholarship across Fellows:** student · scholarship · assigned Fellow · **status pill** · On Hold/Discarded badge · last-updated · **Reassign** action.
  - Admin is **oversight**: it shows the status the Fellows drive (read-only status) and the On Hold/Discarded badges (read-only). Admin's only row action is **Reassign Fellow** (existing capability). No operator actions (Submit / Record disbursement) on the Admin oversight list — those remain the Fellow's, available per-student in `#a-student` if needed.

### Ripple — Admin

- **`#a-student` → Applications tab:** align vocabulary to the one status ladder; show On Hold / Discarded badges (read-only). Read-only oversight is unchanged otherwise.
- **MIS Dashboard `#a-home`:** unchanged (stage funnel already maps to the status ladder; Decision sub-status breakdown already aligns). Only the rail label of the merged screen changes.
- **Dashboard/rail badges:** the "Cases (10)" style badge becomes the actionable-subset count for Applications.

## Vocabulary reconciliation (old → new)

Used to migrate the wireframe's demo data so each student × scholarship appears **once** with a single status + optional flag.

| Old work-state | Old app-status (if any) | New single status | New flag |
|---|---|---|---|
| To Do | (pre-submission) | Eligibility Identified **or** Documents Pending **or** Ready to Submit (by journey stage shown) | — |
| In-progress | Under Scrutiny | Under Scrutiny | — |
| In-progress | (docs) | Documents Pending | — |
| Completed | Benefits Received | Benefits Received | — |
| On Hold | any | (its underlying status) | **On Hold** + reason |
| Discarded | any | (its underlying status) | **Discarded** + reason |
| — | Re-apply / Rejected | Re-apply / Rejected | — |

**Demo dataset (Fellow "Rahul", representative — exact rows finalized in the plan):** ~8 cases spanning every bucket, including at least one **On Hold** (with reason) and one **Discarded** (with reason), so all filters and badges are demonstrable. Admin demo: the same reconciled cases across ~4 Fellows. No student×scholarship pair appears twice.

## Production data model (Frappe, for reference)

One `Case` DocType (the "Application" record):

```
DocType: Case (Submittable: No)
  - case_id: Data (auto-name: CASE-{YYYY}-{####})
  - student: Link → Student
  - scholarship: Link → Scholarship
  - assigned_fellow: Link → User (Fellow role)
  - status: Select (Eligibility Identified / Documents Pending / Ready to Submit /
            Under Scrutiny / Application Approved · funds awaited /
            Benefits Received / Re-apply / Rejected)   # single source of truth
  - on_hold: Check
  - on_hold_reason: Small Text (mandatory_depends_on: on_hold)
  - discarded: Check
  - discarded_reason: Small Text (mandatory_depends_on: discarded)
  - created, modified, owner — standard
  # attention bucket is COMPUTED from status + on_hold, never stored
  # work_state field REMOVED
```

## Code cleanup (wireframe)

- Retire `#f-tasks` markup and the standalone work-state machinery no longer needed: `filterMyCases`, `transitionCase`, `applyCaseState`, the 5 work-state summary cards, `.case-state` pills for To Do/In-progress/Completed (On Hold/Discarded styling is retained for the flags).
- Repurpose Admin `#a-tasks` into the merged Applications oversight; retire `filterCases`/`filterCasesByStage`/`caseChangeState` work-state logic (keep `caseReassign`).
- Retain and generalize: `filterApp` (extend to triage buckets), `updateAppStatus` (aligned to the ladder), disbursement + correction-note handlers.
- No orphaned functions left referenced by dead markup (verify by grep).

## Edge cases

- **On Hold at any status:** badge shows; triage bucket = On Hold (overrides). Resume returns to the status-derived bucket.
- **Re-apply:** triage = Action needed (Fellow must redo); primary action re-submits.
- **Discarded:** terminal; triage = Done bucket (or filtered via the On Hold/Done set); Reopen returns it to its prior status.
- **Two scholarships, one student:** two separate rows/cases (unchanged).
- **Empty filters:** each triage filter shows a proper empty state (reuse existing `.empty` pattern) — no inert screens.

## Verification (per `samavesh-render-verify`)

Headless-Chrome render (`--headless=new`, isolated `--user-data-dir`), reading each screenshot to confirm **behavior**:

1. Fellow `#f-applications` — default list; each triage filter actually filters; a row's primary action advances the status pill; ⋯ → Put on hold shows the badge + reason; Discard shows the badge.
2. Fellow `#f-home` — re-labelled triage cards deep-link pre-filtered.
3. Fellow `#f-student` Applications tab — aligned status control + On Hold/Discard.
4. Admin `#a-tasks` (Applications) — cross-Fellow list; triage + Fellow + status filters compose; Reassign works; badges read-only.
5. Rail — one entry per role, no dead nav, count badge = actionable subset.
6. Grep: no `#f-tasks`/`My Cases` references remain; no orphaned work-state functions referenced by markup.

## Decisions (record)

1. Merged rail item name = **"Applications"** (both roles).
2. Apply to **Admin too**, in the same build.
3. Keep **"Discarded"** wording (not "Dropped").
4. Fellow list is operator (row actions); Admin list is oversight (Reassign + read-only status/flags).
5. Rail count badge = the **actionable subset** ("Action needed") count.
