# HRMS — Attendance & Leave Management — Design

**Date:** 2026-08-21
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` only — the Fellow attendance card (`#fAttCard`, line 2291), the Fellow "My Attendance" screen (`#f-attendance`, line 3214) and the Admin "Attendance" screen (`#a-attendance`, line 4259).
**Source:** `New Feedbacks/Point_09_HRMS_Attendance_Leave.md` ("Point 9", Module 13 in the master FRD). ⚠️ The document marks itself **Out of Scope / Low, "do not begin until scoped in"**; built now under Shweta's standing instruction that every `New Feedbacks/` document gets implemented. Decisions below confirmed with Shweta on 2026-08-21.

## Problem

Programme management has no system of record for fellow attendance and leave — relevant to payroll and compliance, and to reading Point 6's performance data against days actually worked.

**Correction to the client note.** The document says "Not built. No attendance or leave tracking currently exists in the portal." That is substantially wrong. What already exists:

| Already built | Where |
|---|---|
| Fellow check-in / check-out, with time and browser location | `#fAttCard` + `fellowPunch()` (line 4891) |
| Fellow's own attendance history — Date · In · Out · Gross Hrs · Status · Location, with date and status filters | `#f-attendance` (line 3214) |
| Admin "who's working now" live panel, 3 summary cards (Present today · On leave · Avg hrs) | `#a-attendance` (line 4266) |
| Admin per-fellow daily record with fellow / date / status filters | `#attTable` (line 4289) |
| **Attendance export** — which the document lists as an explicit new requirement | Admin Attendance, line 4262 |
| Statuses | **Present · On Leave · Absent** |

The genuine gaps are therefore narrower than the document implies: **Work From Home** as a status, **status chosen at check-in**, the **leave application flow**, **Admin leave-balance adjustment**, and **state-specific holiday calendars**.

**And two sentences in the wireframe become false the moment this is built** — "Leaves & holidays are managed **manually**" on the Fellow screen and "Leaves & holidays are managed by Admin" on the Admin screen. Both are rewritten as part of this module; leaving them would be exactly the docs-versus-code drift the working agreement treats as a defect.

## Confirmed decisions (Shweta, 2026-08-21)

1. **Four statuses, not three.** The fellow picks **Present / Work From Home / Absent** at check-in, per the document. **On Leave** stays, but is **system-set** by an approved leave request rather than chosen — which is already what it means in the existing history tables. Nothing the client has already reviewed is removed.
2. **The fellow applies for leave on My Attendance** — balance, apply form and request history added to the screen that already holds their attendance history. No new nav item.
3. **The Admin manages leave and holidays on the existing Attendance screen** — leave requests, balance adjustment and the holiday calendar as sections. This is also what makes that screen's own "managed by Admin" line true.
4. **Holiday calendar: Maharashtra plus one clearly-marked sample state**, so the per-state mechanism is visible and clickable. Hard rule 1 permits a placeholder when it is explicitly marked as a sample, and it is labelled as such in the UI.

## What gets built

**Check-in with status.** A status selector sits beside the Log In button (Present / Work From Home / Absent). On check-in the chosen status is recorded, echoed in the card's state line, and **today's row is written into the Fellow's own attendance table** — so punching in visibly produces a record rather than only changing a label. Checking out fills the Out time and gross hours.

**Fellow leave (on My Attendance).**
- **Balance** per leave type — Casual, Sick, Earned — showing used and remaining.
- **Apply**: leave type, from and to dates, reason → creates a request in *Pending* state and deducts nothing until approved.
- **My requests**: the fellow's own history with status (Pending / Approved / Rejected) and the Admin's note where present.
- **Holidays**: the fellow sees *their own state's* holiday list, read-only — the list follows their assigned region.

**Admin leave and holidays (on Attendance).**
- **Leave requests** queue: fellow, type, dates, days, reason → Approve / Reject, with a note. Approving deducts from that fellow's balance and marks those dates **On Leave** in the attendance record, which is what makes On Leave system-set rather than a check-in choice.
- **Balance adjustment**: per fellow per type, editable by the Admin, as the document requires.
- **Holiday calendar**: pick a state, see its holidays, add one. State-specific, with the sample state labelled.

## Data shape

```
ATT_STATUS_CHOICES = ['Present','Work From Home','Absent']     // check-in choices
LEAVE_TYPES        = ['Casual','Sick','Earned']
LEAVE_BALANCE      = { '<fellow>': { Casual:{total,used}, Sick:{…}, Earned:{…} } }
LEAVE_REQUESTS     = [ {id, fellow, type, from, to, days, reason, status, note} ]
HOLIDAYS           = { 'Maharashtra':[{date,name}], '<sample state>':[…] }
FELLOW_STATE       = { '<fellow>': 'Maharashtra' }
```

Approving a request is the only thing that writes to a balance, and a rejection writes nothing — so the numbers can only move through the flow the client described.

## Non-goals

- No payroll, no salary, no attendance regularisation workflow — none of it is in the document.
- No location inference for status: the document is explicit that status is fellow-selected. The existing browser-location capture on punch is left exactly as it is.
- No second export button — the Admin screen already has one, and it covers the document's export requirement.
- No change to the "who's working now" panel or the three summary cards beyond keeping their figures consistent.
- No Frappe build.

## Production note (Frappe) — carried forward from the client spec

The document confirms **Frappe HR is not installed** on the Samavesh bench and recommends installing it rather than rebuilding equivalents — which maps this module almost entirely onto configuration: **Employee Checkin** (check-in/out plus the fellow-selected status), **Leave Application**, **Leave Type**, **Leave Allocation** for balances, and **Holiday List** assignable per state. Effort for this module must therefore include Frappe HR installation and setup, not just configuration of an app already present. A custom build is the fallback only if installation is infeasible for infrastructure or licensing reasons.

## Verification (render-verify)

1. `?role=fellow&screen=f-home` — choose Work From Home, Log In: the state line names the status and today's row appears in My Attendance with that status; Log Out fills Out time and hours.
2. `?role=fellow&screen=f-attendance` — balance shows three leave types; apply for leave → it appears under My requests as Pending, and **no balance moves**.
3. `?role=admin&screen=a-attendance` — the new request is in the Admin queue; Approve → the fellow's balance for that type drops by the day count, the request reads Approved, and those dates read **On Leave** in the attendance table.
4. Reject a second request → status Rejected, balance unchanged.
5. Admin adjusts a balance directly → the fellow's screen shows the new figure.
6. Holiday calendar: switching state changes the list; adding a holiday appears in it; the fellow sees their own state's list, read-only, with the sample state visibly labelled.
7. Both "Leaves & holidays are managed manually / by Admin" sentences are gone, replaced by wording that matches what the screens now do.
8. Grep: `Work From Home` present in the check-in selector and in both status filters; no leave control anywhere on the student, mentor or Admin-student surfaces.
