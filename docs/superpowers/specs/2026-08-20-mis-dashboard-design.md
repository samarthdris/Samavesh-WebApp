# MIS / M&E Dashboard — Design

**Date:** 2026-08-20
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` only. Extends the Admin Program Dashboard (`#a-home`, lines 3594–3664) and adds one new demographic field, which ripples into the onboarding form and three profile views.
**Source:** `New Feedbacks/Point_5_MIS_Dashboard.md` (headed "Point 4 — MIS Dashboard", Module 10 in the master FRD). The document marks itself **Out of Scope for the current phase** and says not to begin until Dhwani's effort estimate is reviewed; **Shweta directed on 2026-08-20 to build it now anyway.** Recorded here so the sequencing decision is traceable, not silent.

## Problem

Programme leadership and funders need aggregate visibility — reach, equity, fellow performance — that individual records don't surface. The client has confirmed ~27–28 indicators across five themes.

**Correction to the client note.** The document says "Not built. No program-monitoring dashboard currently exists." A Program Dashboard has existed since the Phase-4 build:

| Already built | Where |
|---|---|
| Filter strip — 6 working selects (date range, state, district, fellow, scholarship, caste category) wired to `misFilter()` | line 3601 |
| 5 number cards — Students onboarded 480 · Scholarships identified 1,240 · Applications submitted 264 · Applications approved 182 · Total disbursed ₹74.2L | line 3611 |
| Stage-wise funnel, 5 stages + decision breakdown | line 3620 |
| Fellow-wise performance table, incl. "₹ Unlocked" | line 4060 |
| Scholarship-wise breakdown | line 3651 |
| Export button (respects active filters) | line 3598 |

So the overview metrics, fellow-wise and scheme-wise cuts and the filter strip exist. This is an extend.

## Decisions confirmed with Shweta (2026-08-20)

1. **One funnel, not two.** The document asks for Eligible → Applied → Approved → Disbursed; a 5-stage funnel already exists. The existing funnel **stays** (its "Documents Complete" stage matters to the Fellow SOP) and gains a **drop-off rate at each stage**, which is what the requested conversion funnel adds. A second overlapping funnel would breach hard rule 2.
2. **Migrant status becomes a real field.** The migrant / non-migrant breakdown needs data that exists nowhere — zero hits in the wireframe, BRD v4.0, or the onboarding form spec. Shweta chose to **add the field** rather than skip the cut. See the ripple list below.
3. **All five sub-sections this round**, with mock figures, so the client reviews the whole dashboard. Real prerequisites are documented as Frappe notes, never faked as solved.
4. **"Funds unlocked" = SUM of amounts whose status is "Funds Disbursed"** — the actual value from Module 5, not the feedback doc's "Disbursed".

## What gets added

Organised into the five sub-sections the document asks for, each a card with a heading, rather than one long scroll:

| Sub-section | Indicators added |
|---|---|
| **Overview** | 6th number card — *Schemes tapped*; month-wise timeline (12 bars) |
| **Demographics** | Gender-wise · Caste-category-wise · Annual-income-wise · Education-stream-wise · Grade-wise + Total colleges |
| **Geography** | State/district-wise · **Migrant / non-migrant** (the new field) |
| **Scheme Performance** | Scheme utilisation (share of a scheme's capacity used) · Documents-support dashboard |
| **Process Efficiency** | Funnel drop-off (folded into the existing funnel) · Average time-to-approval · Average time-to-disbursement · Repeat/renewal rate · Document friction rate (which document types sit longest in follow-up) |

The existing Fellow-wise performance table and Scholarship-wise card are relabelled into the **Fellow Performance** and **Scheme Performance** groups where they already belong — moved under headings, not rebuilt.

## Architecture

One mock-data object `MIS` holding every indicator's rows, and one generic renderer `renderMisBars(rows)` producing a labelled horizontal bar list, reusing the existing `.funnel` bar styling so the new charts read as the same visual family. Single figures reuse the existing `.card.stat` component. No charting library — CSS bars only, consistent with the rest of the file.

## The filter strip, and a stated simplification

The document asks for year / fellow / student / college filters. The strip has year (date range), fellow, scholarship, state, district and category; **student** and **college** selects are added.

Every dropdown must visibly do something (hard rule 2). Re-querying 28 mock indicators per filter combination isn't meaningful in a wireframe, so: changing any filter sets a `MIS_FILTER` state, shows a **"Filtered view: …"** chip naming the active selections, and scales the mock figures by a deterministic factor so the numbers visibly respond. This is a **stated simplification** — the same kind already used for the Data Entry form's PDF field — not a claim that filtering is implemented. In the Frappe build each widget re-queries with the filter values.

## Migrant field — the full ripple

Adding one demographic field touches seven places; missing any of them is the drift the July audit flagged:

1. **Onboarding form markup** — new `<div class="q">` "Migrant Family?" (`onbMigrant`, values *Migrant* / *Non-migrant* / *Prefer not to answer*) beside Social Background and Annual Family Income (line 2630). One insert covers **all three render modes**, since the form is cloned via `cloneOnbForm(prefix)` for student self-fill (`s_`), Fellow review (`rev_`) and the student's read-only view (`myonb_`).
2. **`MY_ONBOARDING.fields`** (line 4387) — Aarti's own answer.
3. **`PENDING_ONBOARDINGS`** (lines 4449, 4466) — both pending submissions, so the Fellow's approval review shows it.
4. **Student's own profile** (line 1972, beside Annual Family Income).
5. **Fellow's student-detail profile** (line 2765).
6. **Admin's student-detail profile** (line 3808).
7. **MIS Geography sub-section** — the breakdown itself, plus a Migrant select in the filter strip.

Vocabulary: the client's own words are "migrant/non-migrant", so those are the values. Aarti is the canonical demo student (hard rule 7) — her value is set consistently in every one of the places above.

## Non-goals

- The Admin "₹ Unlocked" column and the student-list "₹71,000" stay hardcoded — same watch-item as Module 5, deliberately unchanged.
- No cohort funnel. A snapshot funnel is buildable from current statuses; a cohort funnel needs status-change history that doesn't exist.
- No scheme `capacity` field on the Scholarship master — utilisation uses mock capacity, with the missing field documented.
- No cross-year student identity, so repeat/renewal rate is mock.
- No Frappe build.

## Production note (Frappe) — carried forward

Evaluate Frappe's native Query Report / Dashboard Chart / Number Card before any custom reporting layer; the shared filter strip across many independent widgets is the non-trivial piece. Enable `track_changes: 1` on Scholarship Application and Application Document so the Version doctype supplies status-change history — this is what makes time-to-approval, time-to-disbursement and document friction rate real, and it is the **same prerequisite** as the Fellow cards 11–12, so enable once for both. Scheme utilisation additionally needs a `capacity`/`budget` field on the Scholarship master; repeat/renewal rate needs a stable cross-year student identifier that survives the Module 9 migration. "Funds unlocked" = `SUM(Disbursement Entry.amount)` where `status = 'Funds Disbursed'`.

## Open items for the client (not blockers for this build)

- The exact field-level list of the ~22–23 original indicators was never enumerated; this builds the categories as described. The client-shared indicator screenshot is still pending.
- Whether any indicator was meant to be cross-tabulated (e.g. "% rural female") rather than reported independently.
- Migrant status is now collected — the client should confirm the wording and whether it is mandatory in onboarding.

## Verification (render-verify)

1. `?role=admin&screen=a-home` — all five sub-sections render under headings; no empty card, no chart with zero bars.
2. Funnel shows a drop-off percentage on every stage after the first, and the percentages are consistent with the stage counts.
3. Change each filter, including the two new selects — the "Filtered view" chip names the selection and figures visibly change.
4. Migrant field: appears in the onboarding form (`?role=student&screen=onboarding`), in the student's read-only view of their own form, in the Fellow's approval review, and as a labelled field on all three profile views — with the same value for Aarti in every one.
5. Geography sub-section renders the migrant/non-migrant split from that field's demo values.
6. Grep: the new field's id `onbMigrant` appears once in markup and in all three prefill datasets.
