# Campaign / Camp Management & Student Assignment — Design

**Date:** 2026-08-21
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` only. Adds four screens (Admin: camp creation, camp dashboard, program overview · Fellow: My Camps), extends the onboarding form with camp pre-fill and an Assign Fellow control, and turns the Admin student-detail's inert "Reassign" button into a real control with an audit trail.
**Source:** `New Feedbacks/Point_05_Campaign_Creation.md` ("Point 5 — Campaign Creation", Module 11 in the master FRD), itself drawn from two client documents — *Campaign Creation & Camp Management* and *Student Assign Flow*. ⚠️ Marked **Out of Scope / Medium**, "do not begin until formally scoped in and an effort estimate is reviewed"; built now under Shweta's standing instruction. Decisions below confirmed with Shweta on 2026-08-21.

## Problem

On-ground onboarding drives are coordinated manually. There is no camp concept in the product, so a student onboarded at a college drive is not tied to that drive or automatically to the fellow who ran it, and the Admin has no view of camp-level progress. "Campaign" and "Camp" are confirmed to be the same entity in the source material — modelled once, called **Camp**.

**Current state check:** the document says "Not built. No concept of a camp currently exists" — **accurate.** Nothing in the wireframe references camps.

Two things do exist and get absorbed rather than duplicated:

| Exists today | What happens to it |
|---|---|
| Admin student-detail "Reassign" button (line 3924) — an **alert stub**, does nothing | Becomes a real reassignment control with assignment status and history |
| The 42-question onboarding form, cloned into three render modes | Gains camp pre-fill (Track 1) and an Assign Fellow control (both tracks) — no redesign |

## Confirmed decisions (Shweta, 2026-08-21)

1. **Build all four surfaces this round**, as separately gated tasks — the pieces interlock, so shipping creation without a dashboard would leave a camp link pointing at nothing.
2. **QR code is a drawn placeholder**, visually a QR block, explicitly labelled as a sample generated server-side in production. Copy / Download / Share all do something.
3. **The camp link works as a link:** `?camp=<CAMP-ID>` opens the onboarding form pre-filled, and the dashboard's link card opens the same thing, so it is clickable without editing the address bar.
4. **Reassignment lives on the Admin student-detail**, where the student record already is, with the history beneath it.

## Screens

| Screen | id | Contents |
|---|---|---|
| Program overview | `a-camps` | Aggregate totals across all camps + a camp table; a row opens that camp. New Admin nav item **Camps**. |
| Camp dashboard | `a-camp` | Header (name, status, dates, assigned fellows) + the 7 cards + camp-level student table |
| Camp creation | `a-camp-new` | Multi-section form: basic info · venue & institution · partner (conditional) · fellow assignment · summary + Save as Draft / Publish |
| Fellow's My Camps | `f-camps` | Upcoming camps (name, date, venue, target, Open Camp), then venue & directions, timing, partner contact, link/QR, and onboarded/completed/pending counts. New Fellow nav item **My Camps**. |

Desktop rails only — the mobile bottom navs are left alone per hard rule 6.

## The 7 dashboard cards

Per the source document, all read from existing data rather than duplicating tracking:

1. **Student registrations** — count vs target, % and progress bar
2. **Onboarding completed** — completed + pending/draft
3. **Today's registrations** — day-wise count
4. **Fellow** — the primary fellow's onboarded / applications / documents-support counts, with View Fellow and View Students
5. **Camp link** — the link, registrations through it, Copy / Download QR / Share (the document calls this the most prominent card)
6. **Camp timeline** — a **derived** checklist, never independently editable: Camp Created · Fellow Assigned (`primary_fellow` set) · Link Generated (`unique_link` set) · Camp Published (status ≥ Scheduled) · Camp Live (Active) · Camp Completed · Follow-up Closed
7. **Pending actions** — e.g. "9 forms saved as draft", "4 students need document support"

## Rules the build must enforce structurally

- **A camp cannot be published without a Primary Fellow.** The Publish button refuses and says why.
- **Duration is computed from the dates, excluding Sundays** — never entered. Read-only, recomputed whenever either date changes.
- **Camp ID format:** first 3 letters of the institution + `dd/mm/yy` + camp number, generated on save.
- **The timeline is derived** from field values, so it cannot disagree with the camp's actual state.
- **One fellow on a camp → auto-assign** the student; **multiple → a dropdown scoped to that camp's fellows only** (Primary and Supporting both), never the full org list.
- **The regular onboarding form always shows Assign Fellow**, defaulting to no pre-fill, scoped to the full org list.

## Student assignment

**Track 1 — via the camp link.** `?camp=<CAMP-ID>` opens the onboarding form with camp, venue and institution pre-set and **hidden** (a summary banner names the camp instead), and the Assign Fellow control scoped to that camp. On submit: student → camp mapping plus fellow assignment, and a confirmation showing student, camp, camp date, assigned fellow, onboarding status.

**Track 2 — the regular form.** No camp mapping. Assign Fellow is visible with the full fellow list and no pre-fill; a fellow onboarding a student picks their own name.

**Assignment status** on every student: *Not Assigned / Assigned / Reassigned / Inactive-Closed*. Reassignment on the Admin student-detail keeps the current assignment active and appends to a history (previous fellow → new fellow, who changed it, when, optional reason). Unassigned students must be clearly identifiable, so the status renders as a pill with **Not Assigned** in a warning colour.

## Data shape

```
CAMPS = [ {id, name, program, start, end, owner, status, target, description,
           venueType, institution, address, city, district, state, venueContact, venuePhone,
           hasPartner, partnerName, partnerContact, partnerPhone, partnershipType, partnerNotes,
           primaryFellow, supportingFellows[], link, registered, completed, draft, docSupport, today} ]
STUDENT_ASSIGN = { '<student>': {fellow, camp, status} }
ASSIGN_LOG     = { '<student>': [ {from, to, by, on, reason} ] }
```

Three camps are seeded so the client sees each state without creating one: an **Active** camp with real registration numbers, a **Scheduled** one, and a **Completed** one. Figures are chosen to sit consistently alongside the existing demo data (24 students for Rahul More, the four-fellow roster on Fellows & Users).

## Non-goals

- No separate Campaign parent entity — the document confirms Camp and Campaign are one thing.
- No real QR encoding, and no `samavesh.org` requests — the displayed link is the production format, the click opens the local pre-filled form.
- No Institution master data model — the institution field reuses the onboarding form's existing college list with a free-text fallback.
- No change to the onboarding form's 42 questions, its three render modes, or its validation.
- No eligibility-matching or document-status engine work: dashboard cards that depend on Point 7 or Module 1 use declared mock counts, as the source document anticipates.
- No Frappe build.

## Production note (Frappe) — carried forward

New DocType **Camp** with the fields listed in the source document (`camp_name`, auto ID, `program`, dates, read-only `duration`, `camp_owner` as a role-filtered Link → User, `camp_status`, target, venue block, conditional partner block, `primary_fellow`, `supporting_fellows` as Table MultiSelect, read-only `unique_link`, `qr_code` Attach). **Student** gains `assigned_camp`, `assigned_fellow`, `assignment_status`, plus a **Fellow Assignment Log** child table. Institution/partner matching should use a shared Institution master with quick-create rather than free text, to avoid near-duplicate records. QR generation server-side off `unique_link` with any standard library. The program overview is a filtered aggregate over Camp records, not a join to a parent entity. The dashboard cards depend on Point 7 (eligibility matching) and Module 1 (documents) existing before they show real numbers.

## Verification (render-verify)

1. `?role=admin&screen=a-camps` — totals and the camp table render; a row opens that camp's dashboard.
2. `?role=admin&screen=a-camp-new` — fill dates → duration appears, computed, Sundays excluded (a Mon–Sun span reads 6, not 7); it is not editable.
3. Attempt Publish with no Primary Fellow → refused, with the reason named. Assign one → Publish succeeds, the camp appears in the overview, and the link and QR appear.
4. Camp dashboard — all 7 cards render; the timeline ticks match the camp's own status and fellow fields; changing the camp's status changes the ticks.
5. Camp link card — Copy, Download QR and Share all respond; opening the link lands on the onboarding form with camp/venue/institution pre-set and hidden, and a banner naming the camp.
6. A camp with one fellow → no fellow dropdown on that form (auto-assigned, stated on screen). A camp with two → dropdown listing exactly those two, not the org list.
7. `?role=fellow&screen=f-camps` — My Camps lists the fellow's camps only; Open Camp shows venue, timing, partner contact, link/QR and counts, with **no** create or self-assign control anywhere.
8. Regular onboarding (`?role=fellow&screen=f-onboard`) — Assign Fellow visible, full list, no pre-fill.
9. `?role=admin&screen=a-student` — Reassign is now a real control: reassigning updates the assigned fellow, sets status **Reassigned**, and appends to the visible history with who and when.
10. Grep: no camp create/self-assign control inside the Fellow app; `Not Assigned` renders as a warning pill.
