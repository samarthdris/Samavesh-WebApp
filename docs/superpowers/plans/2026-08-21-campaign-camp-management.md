# Campaign / Camp Management & Student Assignment — Implementation Plan

**Goal:** Four new screens (Admin program overview, camp dashboard, camp creation; Fellow My Camps), camp pre-fill and an Assign Fellow control on the onboarding form, and a real reassignment control with an audit trail on the Admin student-detail.

**Architecture:** One `CAMPS` array plus `STUDENT_ASSIGN` / `ASSIGN_LOG` objects; renderers per surface (`renderCampList`, `renderCampDash`, `renderMyCamps`, `renderAssignPanel`); derived helpers (`campDuration` excluding Sundays, `campId`, `campTimeline`); one `?camp=` URL entry point that pre-fills the onboarding form.

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build, no external libraries (the QR is a drawn placeholder). Verification = render + click-through.

**Design doc:** `docs/superpowers/specs/2026-08-21-campaign-camp-management-design.md`

## Global constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** — ask before every commit, ask again before any push.
- **Desktop rails only** — do not touch `#fellowBot` / `#adminBot` mobile navs (hard rule 6).
- **Fellows never create camps or self-assign** — no such control may appear anywhere in the Fellow app.
- **Publish is blocked without a Primary Fellow**; **duration is never editable**; **the timeline is derived**.
- **No inert UI:** Copy / Download QR / Share, and every camp action, must respond.
- **Do not touch** the onboarding form's 42 questions, its three render modes, or its validation.
- **Render-verify with the PID-scoped headless recipe** in `docs/CONTEXT.md` — never kill Chrome by name (hard rule 7).

---

## Task 1: Data model, derived helpers, CSS

**Files:** `wireframe/index.html` — CSS before `</style>`; JS after the HRMS block.

- [ ] **Step 1: CSS.** `.camp-grid` / `.camp-card` (dashboard cards), `.camp-prog` (progress bar), `.camp-tl` / `.camp-tl-row` (timeline ticks), `.qr-box` (drawn placeholder + sample label), `.camp-link-row`, `.camp-sec` (creation form sections), `.assign-hist` (audit rows), `.pill.warn` reuse for Not Assigned.
- [ ] **Step 2: `CAMPS`** — three seeded camps (Active with real numbers, Scheduled, Completed), fields per the design doc, figures consistent with the existing demo roster.
- [ ] **Step 3: `STUDENT_ASSIGN` + `ASSIGN_LOG`** — seeded for the canonical demo students, including **one deliberately Not Assigned** so that state is visible without creating it.
- [ ] **Step 4: Derived helpers.** `campDuration(start,end)` counting calendar days **excluding Sundays**; `campIdFor(institution, date, n)` (first 3 letters + dd/mm/yy + number); `campTimeline(camp)` returning the 7 ticks computed from fields only; `campLink(camp)` returning the display URL.
- [ ] **Step 5: QR placeholder.** `qrBox(link)` returning an inline SVG that reads as a QR block, with a visible "sample QR — generated server-side in production" label.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Camps: data model, derived duration/ID/timeline helpers, QR placeholder"` — ask first.

---

## Task 2: Program overview + Admin nav

**Files:** `wireframe/index.html`.

- [ ] **Step 1:** New Admin nav item **Camps** (`data-go="a-camps"`) in the desktop rail only, after Attendance.
- [ ] **Step 2:** New `<section class="screen" id="a-camps">` with a `#campTotals` host and a `#campTable` host; `renderCampList()` renders the aggregate totals (camps, target, registered, applications identified/completed, documents identified/completed, completed students) and the camp table (dates, venue, fellow, onboarded, completed, status), each row opening `openCamp(id)`.
- [ ] **Step 3: Render — verify.** Totals and three camps render; a row opens the dashboard.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Camps: program-level overview + Admin nav"` — ask first.

---

## Task 3: Camp dashboard — 7 cards + student table

**Files:** `wireframe/index.html`.

- [ ] **Step 1:** New `<section class="screen" id="a-camp">` with a `#campDash` host; `renderCampDash(id)` renders the header (name, status pill, dates, assigned fellows) and the 7 cards per the design doc.
- [ ] **Step 2:** Card 5 (camp link) gets working Copy / Download QR / Share plus an **Open onboarding link** action that routes to the pre-filled form (Task 5).
- [ ] **Step 3:** Card 6 renders `campTimeline()` — ticks derived, never editable.
- [ ] **Step 4:** Below the cards, the camp-level student table: Student · Mobile · Onboarding Date · Documents · Scholarships Mapped · Application Status · Final Status · Assigned Fellow.
- [ ] **Step 5: Render — verify.** All 7 cards present; timeline ticks match the camp's status; changing a camp's status changes the ticks.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Camps: camp dashboard (7 cards + student table)"` — ask first.

---

## Task 4: Camp creation form + publish gate

**Files:** `wireframe/index.html`.

- [ ] **Step 1:** New `<section class="screen" id="a-camp-new">` — sections for basic info, venue & institution, partner (revealed by a Yes/No toggle), fellow assignment (Primary required, Supporting multi), then a summary block.
- [ ] **Step 2:** The fellow picker shows **workload context**, not just names — "Rahul More · 2 active camps · 24 students" — sourced from `CAMPS` and the existing roster.
- [ ] **Step 3:** `campRecalcDuration()` on either date change → writes the read-only duration (Sundays excluded).
- [ ] **Step 4:** `saveCamp('draft'|'publish')` — publish **refuses without a Primary Fellow**, naming the reason; on success generates the camp ID and link, appends to `CAMPS`, and opens its dashboard.
- [ ] **Step 5: Render — verify.** Mon–Sun span reads 6 days; duration not editable; publish refused then accepted; new camp appears in the overview with link and QR.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Camps: creation form, Sunday-excluded duration, publish gate"` — ask first.

---

## Task 5: Both assignment tracks on the onboarding form

**Files:** `wireframe/index.html`.

- [ ] **Step 1: Assign Fellow control** on the onboarding form — full org list, no pre-fill (Track 2).
- [ ] **Step 2: `?camp=<ID>` entry** — read it in the existing URL-param handler; open the onboarding form, hide the camp/venue/institution questions, set their values, and show a banner naming the camp, its date and venue.
- [ ] **Step 3: Fellow scoping** — one fellow on the camp → no dropdown, a line stating the auto-assignment; multiple → a dropdown of **exactly that camp's** fellows.
- [ ] **Step 4: Submit confirmation** — student, camp, camp date, assigned fellow, onboarding status; writes `STUDENT_ASSIGN` and bumps that camp's registration count so the dashboard moves.
- [ ] **Step 5: Render — verify.** Both tracks; a one-fellow camp auto-assigns; a two-fellow camp lists only those two; the regular form shows the full list with no pre-fill.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Camps: camp-link onboarding pre-fill + Assign Fellow on both tracks"` — ask first.

---

## Task 6: Fellow's My Camps

**Files:** `wireframe/index.html`.

- [ ] **Step 1:** New Fellow nav item **My Camps** (`data-go="f-camps"`, desktop rail only) and `<section class="screen" id="f-camps">`.
- [ ] **Step 2:** `renderMyCamps()` — camps where this fellow is Primary or Supporting only; each shows name/date/venue/target and Open Camp; opening shows venue & directions, timing, partner contact, link/QR and onboarded/completed/pending counts.
- [ ] **Step 3: Render — verify.** Only this fellow's camps appear; **no** create, self-assign or campaign-select control exists anywhere in the Fellow app.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Camps: Fellow My Camps view"` — ask first.

---

## Task 7: Reassignment + assignment status

**Files:** `wireframe/index.html`, Admin student-detail (line ~3924).

- [ ] **Step 1:** Replace the inert `alert(...)` on the existing **Reassign** button with `openReassign()` — a real inline control: new fellow, optional reason, Save.
- [ ] **Step 2:** `saveReassign()` — updates the assigned fellow, sets assignment status to **Reassigned**, appends `{from,to,by,on,reason}` to `ASSIGN_LOG`, and renders the history beneath the field.
- [ ] **Step 3:** Render assignment status as a pill wherever the student's fellow is shown on this screen; **Not Assigned** in a warning colour so unassigned students are identifiable.
- [ ] **Step 4: Render — verify.** Reassign twice → both entries in history, newest first, status Reassigned, current fellow correct.
- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html && git commit -m "Camps: real fellow reassignment with assignment status and audit trail"` — ask first.

---

## Task 8: Full-sweep verify + heartbeat + commit

**Files:** `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Behaviour sweep** — all 10 checks in the design doc's Verification section, with screenshots of the overview, the dashboard, the creation form and My Camps.
- [ ] **Step 2: Consistency greps** — zero camp-create or self-assign controls inside the Fellow app; duration inputs are read-only everywhere; the timeline has no editable control; the mobile bottom navs are unchanged.
- [ ] **Step 3: Heartbeat** — dated `BUILD_STATUS.md` section: the four new screens, the derived duration/ID/timeline, the publish gate, both assignment tracks, the reassignment stub becoming real, which dashboard numbers are mock and why (Point 7 / Module 1 not built), and that this module was built despite its out-of-scope header. Source: `New Feedbacks/Point_05_Campaign_Creation.md`.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html wireframe/BUILD_STATUS.md docs/superpowers/specs/2026-08-21-campaign-camp-management-design.md docs/superpowers/plans/2026-08-21-campaign-camp-management.md "New Feedbacks/Point_05_Campaign_Creation.md"` — ask first. Push is a separate gate.
- [ ] **Step 5:** Note for Shweta: the two source `.docx` files are excluded by `.gitignore` (`*.docx` with four explicit exceptions). If they should live in the repo like the BRD and SOP do, that needs two negation lines added — her call, not assumed.

---

## Self-Review (against the design doc)

- **Spec coverage:** data + helpers → Task 1. Overview → Task 2. Dashboard → Task 3. Creation + publish gate → Task 4. Both tracks → Task 5. Fellow view → Task 6. Reassignment → Task 7. Docs + verification → Task 8.
- **Rules made structural, not decorative:** publish refuses without a Primary Fellow; duration is read-only and recomputed; the timeline is derived from fields so it cannot disagree with the camp.
- **Absorbs rather than duplicates:** the existing inert Reassign button becomes the real control; the onboarding form is extended, never forked.
- **Mock declared:** dashboard counts that depend on Point 7 / Module 1 are mock, labelled in the heartbeat, with the dependency named.
- **Edge cases built, not just noted:** one seeded student is Not Assigned so that state is visible on first load.
- **Constraint:** every commit GATED; push separate; PID-scoped renders only.
