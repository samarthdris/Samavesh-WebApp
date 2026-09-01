# HRMS — Attendance & Leave Management — Implementation Plan

> **SUPERSEDED 2026-09-01.** The module this plan describes was removed. Shweta: the HRMS built here
> was custom; the client gets the **standard Frappe HR app** instead. Kept as the record of what was
> built and why it was withdrawn. Current: `docs/superpowers/specs/2026-09-01-hrms-frappe-standard-design.md`
> and `docs/superpowers/plans/2026-09-01-hrms-frappe-standard.md`.


**Goal:** Add the five genuine gaps to the attendance system that already exists — Work From Home as a status, status chosen at check-in, a Fellow leave flow (balance / apply / history / own-state holidays), Admin leave approval and balance adjustment, and a state-specific holiday calendar — and rewrite the two "leaves are managed manually" sentences that this module makes false.

**Architecture:** One state block (`LEAVE_TYPES`, `LEAVE_BALANCE`, `LEAVE_REQUESTS`, `HOLIDAYS`, `FELLOW_STATE`, `ATT_STATUS_CHOICES`), renderers per surface (`renderMyLeave`, `renderAdminLeave`, `renderHolidays`), and approval as the only writer to a balance. Check-in extends the existing `fellowPunch()` rather than replacing it.

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build. Verification = render + click-through.

**Design doc:** `docs/superpowers/specs/2026-08-21-hrms-attendance-leave-design.md`

## Global constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** — ask before every commit, ask again before any push.
- **On Leave is never a check-in choice** — it is written only by an approved leave request.
- **Only approval moves a balance.** Applying and rejecting must not.
- **Keep what exists:** the punch flow's location capture, the "who's working now" panel, the three summary cards and the existing Export button all stay as they are.
- **No inert UI, no `prompt()`.** Every new select, button and field does something visible.
- **Sample state must be labelled as a sample** in the UI (hard rule 1).

---

## Task 1: State + CSS

**Files:** `wireframe/index.html` — CSS before `</style>`; JS after the fellow-targets block.

- [ ] **Step 1: CSS.** `.hr-sec` (section wrapper), `.leave-bal` / `.lb-tile` (balance tiles), `.leave-form`, `.lreq` (request row), `.lreq .lst` (status pill reuse), `.hol-list`, `.hol-row`, `.sample-tag` (the sample-state label).
- [ ] **Step 2: State.** `ATT_STATUS_CHOICES` (Present / Work From Home / Absent), `LEAVE_TYPES` (Casual / Sick / Earned), `LEAVE_BALANCE` seeded per fellow, `LEAVE_REQUESTS` seeded with one Pending and one Approved so both states are visible on first load, `HOLIDAYS` for Maharashtra plus one sample state, `FELLOW_STATE` mapping all four demo fellows to Maharashtra.
- [ ] **Step 3 (GATED): commit.** `git add wireframe/index.html && git commit -m "HRMS: leave/holiday state model + styles"` — ask first.

---

## Task 2: Check-in with status

**Files:** `wireframe/index.html`, `#fAttCard` (line 2291) and `fellowPunch()` (line 4891).

- [ ] **Step 1:** Add a status select beside the Log In button, options from `ATT_STATUS_CHOICES`, defaulting to Present.
- [ ] **Step 2:** Extend `fellowPunch()` — read the chosen status, keep the existing `getGeo` location capture and the state-line behaviour, and additionally write today's row into `#fAttTable` on check-in (Date · In · — · 0.0 · chosen status · location), then fill Out and gross hours on check-out. Read the function first; nothing it already does may be lost.
- [ ] **Step 3:** Add `Work From Home` to both status filter selects (`#f-attendance` line 3219, `#a-attendance` line 4284) so the new status is filterable on both surfaces.
- [ ] **Step 4: Render — verify.** Pick Work From Home → Log In → state line names it and the row appears with that status; Log Out fills Out and hours; the filter finds the row.
- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html && git commit -m "HRMS: status at check-in + Work From Home across both filters"` — ask first.

---

## Task 3: Fellow leave section

**Files:** `wireframe/index.html`, `#f-attendance`.

- [ ] **Step 1:** Add a `#myLeave` host below the attendance table; `renderMyLeave()` renders three balance tiles (total / used / remaining per type), the apply form (type · from · to · reason · Apply), the fellow's own request history, and their state's holiday list read-only with the sample tag where applicable.
- [ ] **Step 2:** `applyLeave(btn)` validates type and both dates (and that from ≤ to), computes the day count, pushes a **Pending** request, re-renders, and **does not touch the balance**.
- [ ] **Step 3:** Rewrite the Fellow screen's "Leaves & holidays are managed manually" sentence to describe what the screen now does.
- [ ] **Step 4: Render — verify.** Apply → Pending row appears, balances unchanged; submitting with a missing date is refused with a message naming the field; from-after-to is refused.
- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html && git commit -m "HRMS: Fellow leave balance, application and own-state holidays"` — ask first.

---

## Task 4: Admin leave approval, balances and holiday calendar

**Files:** `wireframe/index.html`, `#a-attendance`.

- [ ] **Step 1:** Add a `#adminLeave` host after the attendance table; `renderAdminLeave()` renders the request queue (fellow · type · dates · days · reason · Approve / Reject with a note field) and the per-fellow balance adjustment grid.
- [ ] **Step 2:** `decideLeave(id, 'approve'|'reject', note)` — approval deducts the day count from that fellow's balance for that type **and** writes `On Leave` rows into the attendance table for those dates; rejection changes status only. Re-render both surfaces.
- [ ] **Step 3:** `adjustBalance(fellow, type, value)` — Admin-only direct edit, re-rendering the Fellow view too.
- [ ] **Step 4:** `renderHolidays()` — state selector (Maharashtra + the labelled sample), the list, and an add-holiday row (date + name → appears in the list).
- [ ] **Step 5:** Rewrite the Admin screen's "Leaves & holidays are managed by Admin" sentence to match what it now does.
- [ ] **Step 6: Render — verify.** Approve → balance drops by exactly the day count, request reads Approved, `On Leave` rows appear in the attendance table. Reject → status only, balance untouched. Adjust a balance → the Fellow screen shows the new figure. Switch state → list changes; add a holiday → it appears.
- [ ] **Step 7 (GATED): commit.** `git add wireframe/index.html && git commit -m "HRMS: Admin leave approval, balance adjustment, state holiday calendar"` — ask first.

---

## Task 5: Full-sweep verify + heartbeat + commit

**Files:** `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Behaviour sweep** — all 8 checks in the design doc's Verification section, with screenshots of the Fellow and Admin screens.
- [ ] **Step 2: Consistency greps** — `Work From Home` present in the check-in selector and both filters; **zero** remaining occurrences of "managed manually" / "managed by Admin"; no leave control on any student, mentor or Admin-student surface; the "Present today" / "On leave" summary cards don't contradict the attendance table.
- [ ] **Step 3: Heartbeat** — dated `BUILD_STATUS.md` section: what already existed versus what this round added, the four-status decision and why On Leave stays system-set, the sample-state placeholder, the two sentences rewritten, and — importantly for the estimate — that **Frappe HR is not installed**, so this module's real cost includes installing and configuring it. Source: `New Feedbacks/Point_09_HRMS_Attendance_Leave.md`.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html wireframe/BUILD_STATUS.md docs/superpowers/specs/2026-08-21-hrms-attendance-leave-design.md docs/superpowers/plans/2026-08-21-hrms-attendance-leave.md "New Feedbacks/Point_09_HRMS_Attendance_Leave.md"` — ask first. Push is a separate gate.

---

## Self-Review (against the design doc)

- **Spec coverage:** state → Task 1. Check-in status → Task 2. Fellow leave → Task 3. Admin leave + holidays → Task 4. Docs + verification → Task 5.
- **Placeholder scan:** every field has a named source; the one invented value (the sample state) is labelled as a sample in the UI, per hard rule 1.
- **Invariant made structural, not just stated:** only `decideLeave(...,'approve')` writes to a balance, so applying or rejecting cannot move it.
- **Ripple check:** the new status reaches the check-in selector *and* both status filters in the same round; the two now-false sentences are rewritten in the same commit as the behaviour that falsifies them.
- **Nothing removed:** location capture, the working-now panel, the summary cards and the existing Export all survive untouched.
- **Constraint:** every commit GATED; push separate.
