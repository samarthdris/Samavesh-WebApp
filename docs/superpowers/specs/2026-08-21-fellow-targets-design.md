# Fellow Target Assignment — Design

**Date:** 2026-08-21
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` only — the Admin "Fellows & Users" screen (`#a-fellows`, line 4186) and the Fellow home (`#f-home`, under the 12-card grid).
**Source:** `New Feedbacks/Point_06_Target_Assign_to_Fellow_Flow.md` ("Point 6 — Target Assign to Fellow Flow", Module 12 in the master FRD). ⚠️ The document marks itself **Out of Scope for the current phase / Low priority** and says "do not begin until scoped in"; **Shweta's standing instruction is that every document in `New Feedbacks/` gets implemented**, so it is built now. Recorded so the sequencing stays traceable. Decisions below confirmed with Shweta on 2026-08-21.

## Problem

Fellow performance can only be reviewed qualitatively today. The Admin sees each fellow's *actuals* — the "Fellow-wise performance" table on the MIS dashboard and the Students / Approved / ₹ Unlocked columns on Fellows & Users — but there is nothing to compare them against, so no objective per-fellow signal.

**Current state check:** the document says "Not built. No performance-target concept currently exists" — **this one is accurate.** Grep finds no target concept anywhere in the wireframe (the existing "target" hits are unrelated CSS and code comments).

## Confirmed decisions (Shweta, 2026-08-21)

1. **Admin sets targets on the Fellows & Users screen** — a per-fellow panel, not a new nav item. Targets are fellow reference data, and that screen already holds the fellow records.
2. **The fellow sees their own target-vs-actual** — a compact progress strip on Fellow home, under the Point 8 workload cards. A fellow who can't see the target can't work toward it, and the source document asks for targets to feed into those dashboard cards.
3. **Actuals are live where the data exists, mock where it doesn't** — same declared-mock discipline as Point 8.

## The three metrics, and where each actual comes from

The document confirms three metric types, and that actual performance is **computed, never manually entered**:

| Metric | Actual source in the wireframe |
|---|---|
| **Applications Submitted** | **Live** — this fellow's rows in `CASES` that have a real application ID (`appId !== '—'`). An application ID exists precisely once it has been submitted on the government portal, so this is a data-driven definition rather than an invented one. Moves when a case advances. |
| **Documents Collected** | Mock, per fellow (`TARGETS_ACTUAL`) — the wireframe holds no per-fellow document-case array. |
| **Students Onboarded** | Mock, per fellow — seeded with the values already shown in the Fellows & Users table (Rahul 24 · Sandip 19 · Dhanashree 22 · Vaibhav 17) so no figure contradicts what's on screen. |

## Data shape

One record per **fellow × period × metric**, exactly as the document specifies — not one record holding every metric:

```
FELLOW_TARGETS = { 'Aug 2026': { 'Rahul More': {apps:40, docs:120, students:8}, … }, 'Jul 2026': {…} }
```

A metric absent from the object means **no target set** — which the UI must render differently from a target of 0. That's the document's stated edge case, and the two states are genuinely different: "management hasn't set one" versus "management set it to zero".

## UI

**Admin — Fellows & Users.** A sixth column with a "Targets" button per fellow. Clicking it opens an inline panel row beneath that fellow (the same inline-expansion pattern the caseload's correction notes already use), containing:

- A **period selector** (Jul / Aug / Sep 2026) — switching it loads that period's targets, so targets demonstrably vary by period.
- One row per metric: target input · actual · a progress bar · the shortfall or surplus.
- **Save** writes the three targets for that fellow and period; **Clear** removes a target so the row returns to "No target set".
- Empty state: "No target set — actual N" in muted text, never "0 / 0".

**Fellow home.** A "My targets — <period>" strip under the workload cards: three progress bars, actual against target, each showing the percentage and the remaining gap. Metrics with no target show "No target set" rather than an empty bar.

## Non-goals

- No new Admin nav item, and no target entry on the MIS dashboard — the comparison lives in the Admin panel and on Fellow home; adding target columns to the MIS Fellow-wise table as well would repeat the same numbers on a third surface (hard rule 2).
- No approval workflow — the document is explicit that this is Admin-only data entry with a computed comparison.
- No change to the existing Fellows & Users figures, the MIS tables, or the Point 8 cards.
- No Frappe build.

## Production note (Frappe) — carried forward

New Master-type DocType **"Fellow Target"**: `fellow` (Link → User), `period` (Month or a Date-range pair), `metric_type` (Select: Applications Submitted / Documents Collected / Students Onboarded), `target_value` (Int). `actual_value` is **not stored** — computed on read, branching on `metric_type` to a different source table each time: Scholarship Application, Application Document, Student, each filtered by fellow + period. Write permission restricted to the Admin role; no submit/workflow. Depends on Modules 1/3 application data existing, but blocks nothing else.

## Verification (render-verify)

1. `?role=admin&screen=a-fellows` — a Targets button on all four fellows; opening one shows three metric rows with actual values.
2. Rahul More's "Applications Submitted" actual matches the count of his `CASES` rows carrying a real application ID.
3. Set a target, Save → the progress bar and the gap reflect it. Clear → the row reads "No target set", not 0.
4. Switch period → targets change; switch back → the saved value is still there.
5. `?role=fellow&screen=f-home` — the strip shows the same three metrics with the same numbers the Admin panel shows for Rahul More.
6. Advance one of Rahul's cases to a submitted state in the caseload → both the Admin panel and the Fellow strip show the higher actual, with no page reload.
7. A metric with no target set shows "No target set" on both surfaces.
