# HRMS — replace the custom build with standard Frappe HR — Design

**Date:** 2026-09-01
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` — the Fellow attendance card (`#fAttCard`, line 2350), the Fellow "My Attendance" screen (`#f-attendance`, line 3289) and the Admin "Attendance" screen (`#a-attendance`, line 4346); plus three screenshots copied into `wireframe/img/`.
**Supersedes:** `docs/superpowers/specs/2026-08-21-hrms-attendance-leave-design.md` and its plan — the module they describe is what this one removes.
**Source:** Shweta, 2026-09-01 — *"the HRMS part currently built is custom, we need to give the client the default HRMS feature of the frappe."*

## Problem

`New Feedbacks/Point_09_HRMS_Attendance_Leave.md` §9 already said it:

> "the first step is installing it (Occam's Razor — don't rebuild what already exists as an app) rather than building parallel custom DocTypes."

The 2026-08-21 round built the custom UI anyway (commit `ff3b565`), on top of attendance surfaces that were already custom. The wireframe is the source of truth for the Frappe build — so a bespoke attendance/leave UI in it reads as a specification to *build* one. It has to stop saying that.

**Timing is favourable:** `ff3b565` sits on `feature/scholarship-application-pdf` and is **not merged to `dev`**, so the client has never seen the leave/holiday module live. What *is* on `dev`, and has been reviewed, is the older custom attendance layer: `#fAttCard`, `#f-attendance`, `#a-attendance`. Both layers are custom, so both go.

## Decision (Shweta, 2026-09-01)

1. **Stub, not a mock.** Delete the custom attendance/leave UI and its JS. Both attendance screens render one "Delivered by the standard Frappe HR app" page. No custom HRMS UI is drawn anywhere in the wireframe.
2. **Pure vanilla Frappe HR.** No customisation is designed in. Where Point 9's ask isn't literally what stock Frappe HR does, the difference is *documented on the page*, so the client meets it in review rather than in UAT.

## Revision — 2026-09-01, after Shweta reviewed the first build

The first build made both Attendance screens a **written explainer**. Shweta's response: *"why have you added all the written record and not the direct implementation of HRMS as per frappe?"* — a page of prose shows the client nothing. **Revised decision: build the standard Frappe HR screens themselves, clickable, under Attendance.** The mapping table survives, small, below the screens (Shweta's choice).

Note the earlier caution that produced the stub: CLAUDE.md permanently bans `wireframe/frappe-desk.html`. That ban is on **that file**, not on the approach — this build introduces no dependency on it and does not reference it.

## What the Attendance screens show

Both `#f-attendance` and `#a-attendance` keep their ids, nav entries and `CRUMBS`, and mount `renderHrDesk(role)` — a working replica of the standard Frappe HR desk, styled to match the three reference screenshots (maroon top bar, breadcrumb, list view with the Filter By sidebar, filter row, count, and the primary add button).

**Doctype tabs, per role.** Fellow: *Employee Checkin · Attendance · Leave Application · Holiday List*. Admin: *Attendance · Employee Checkin · Leave Application · Leave Allocation · Holiday List*. The Fellow's views are scoped to their own Employee record; the Admin sees every Fellow.

**Everything on screen works** (hard rule 2):

| Screen | What it does |
|---|---|
| **Employee Checkin** (list + form) | `+ Add Employee Checkin` opens the standard form — Employee, Log Type (IN / OUT), Time, Location / Device ID, Skip Auto Attendance. Save writes a row to the top of the list. Log Type filter narrows the list. |
| **Attendance** | The daily record — Status, Attendance Date, In Time, Out Time, Gross Hours, Early Exit. Status and Leave Type filters work. |
| **Leave Application** | `+ Add Leave Application` — Leave Type, From / To Date, Reason. Save creates it with status **Open**. The Admin's copy carries **Approve / Reject** on Open rows. |
| **Leave Allocation** (Admin) | Per Employee per Leave Type: New Leaves Allocated, used, balance. Editable. Approving a Leave Application increments used and writes **On Leave** rows into Attendance — the standard chain, not a custom one. |
| **Holiday List** | Pick a Holiday List and see its holidays. One per state, which is how per-state calendars are done. |

**Vocabulary is Frappe's own**, not invented: Log Type `IN` / `OUT`; Attendance statuses *Present · Absent · On Leave · Half Day · Work From Home*; Leave Application status *Open · Approved · Rejected*; Leave Types *Casual Leave · Sick Leave · Privilege Leave*. Employees are the existing Fellows & Users roster — **Rahul More · Sandip K · Dhanashree O · Vaibhav D** — which also retires the old Attendance-only names (Priya Kulkarni, Sunil Kadam) that appeared in no other roster, closing the watch-item raised on 2026-08-21.

**No explanatory copy on the screens** (Shweta, 2026-09-01 — *"why is all the text part still showing… this is the wireframe we are aiming to build like a prototype"*). The client clicks a prototype; they do not read a document. The mapping table, the delivery statement and the role lists are all removed from the page. Below the desk sit the three **reference screenshots** as images, each opening full size — no captions, no prose.

Where standard Frappe HR differs from Point 9 — Work From Home raised via an **Attendance Request** rather than chosen at check-in, and the old "working now" panel not being a screen the app ships — is recorded in `BUILD_STATUS.md` and in this document **for the team**. It must still be said to the client in conversation; it is simply not printed on the prototype.

## Where vanilla differs from Point 9 — for the team, not printed on the page

| Point 9 asked for | Stock Frappe HR | Consequence |
|---|---|---|
| Fellow picks Present / WFH / Absent **at check-in** | Employee Checkin records **IN / OUT only**. *Work From Home* is a standard **Attendance** status, raised through an **Attendance Request**, not chosen at punch. | The status still exists and is still fellow-raised — through a different screen. A status dropdown on the punch itself would be a customisation on top of the app, not part of it. |
| State-specific holiday calendars | **Holiday List** is a standard doctype, assigned per Employee (or per Company). | Met as-is: one Holiday List per state, assigned to the fellows in that state. Configuration, not development. |
| Admin "who's working now" live panel *(exists in the current custom build; not asked for in Point 9)* | Not a stock screen. Checkin data is there; a live panel over it is not. | **Removed.** If the client wants it back it is a custom report on Employee Checkin — flagged, not built. |
| Attendance export | Standard list-view export on Attendance / Employee Checkin. | Met as-is. |

Nothing on that list is being built. This is the team's record of it — the prototype itself carries no such commentary.

## What is deleted

**HTML** — the punch control on `#fAttCard`; the whole body of `#f-attendance` (date/status filters, 15-row history table, `#myLeave` host); the whole body of `#a-attendance` (Export button, "Working now" panel, the three stat cards, the fellow/date/status filters, `#attTable`, `#adminLeave` host).

**JS** — `fellowPunch` (5006), `attFilter` / `attClear` (5067, 5082), `fAttFilter` / `fAttClear` (5089, 5103), and the entire Point 9 block at 6117–6340: `ATT_STATUS_CHOICES`, `LEAVE_TYPES`, `LEAVE_BALANCE`, `LEAVE_REQUESTS`, `HOLIDAYS`, `FELLOW_STATE`, `HOLIDAY_STATE_SEL`, `renderMyLeave`, `applyLeave`, `renderAdminLeave`, `decideLeave`, `adjustBalance`, `renderHolidays`, `setHolidayState`, `addHoliday`, plus the two boot calls at 7216–7217.

**Kept:** `hrFellow()` (6154) — Camps uses it at 5950, so it is not part of this deletion. Any CSS class left with zero users after the deletion goes too (`.att-card`, `.att-btn`, `.att-status-sel`, `.wn-*`, `.hr-sec`, `.leave-*`, `.lb-tile`, `.lreq`, `.hol-*`, `.sample-tag`) — verified by grep before removal, not assumed.

**`#fAttCard` becomes** a pointer: "Attendance and leave run in Frappe HR" with a working link to the stub screen. It keeps a live control rather than becoming a dead card (hard rule 2).

## Non-goals

- No mock of the Frappe desk UI, and no reference to `wireframe/frappe-desk.html` (out of scope, CLAUDE.md).
- No customisation designed on top of the app — including the check-in status dropdown and the "who's working now" panel.
- No new nav item, no screen ids changed, no mobile work.
- Camps, Targets, MIS and every other module untouched.

## Estimate note — carried forward

Frappe HR is **not installed on the Samavesh bench** (Point 9 §7). It is running on other Dhwani benches — the screenshots on the page come from one. So this module's cost is **installation + configuration** (Leave Types, Leave Policy / Allocation, Holiday Lists per state, Shift Type for auto-attendance, employee records for fellows), with **zero custom development** — which is the point of the change.

## Verification (render-verify — `docs/CONTEXT.md` § Verifying, hard rule 7: kill only the captured PID)

1. `?role=fellow&screen=f-attendance` — the stub renders, the three screenshots load (no broken images), the link back to the fellow dashboard works.
2. `?role=admin&screen=a-attendance` — same page, Admin framing; the mapping table including the WFH and "who's working now" rows is visible.
3. `?role=fellow&screen=f-home` — `#fAttCard` shows the pointer, no punch button, no status dropdown; its link lands on `f-attendance`.
4. Console clean on both roles — no `ReferenceError` from a deleted function.
5. Greps, expecting **zero** hits: `fellowPunch|renderMyLeave|renderAdminLeave|applyLeave|decideLeave|adjustBalance|renderHolidays|LEAVE_REQUESTS|LEAVE_BALANCE|HOLIDAY_STATE_SEL|fAttTable|attTable`. And `hrFellow` still defined, Camps still rendering.
6. Nav sweep — the Attendance entry in both sidebars still navigates; `CRUMBS` labels still correct.
