# Point 9 — HRMS / Attendance and Leave Management System
*(Module 13 in the master FRD)*

**Scope Status:** Out of Scope for the current phase | **Priority:** Low *(if scoped in)*
**Note:** Fully resolved — zero open items. Still Out of Scope, but documented ready for CLI whenever this gets picked up.

---

**1. Feature/Module Name:** HRMS — Attendance & Leave Management System

**2. Current State (Existing Wireframe Behavior):** Not built. No attendance or leave tracking currently exists in the portal.

**3. Requested Change:** Fellow check-in/check-out and work status (Present / Work From Home / Absent) tracking, plus leave and holiday management — including state-specific holiday calendars — with Admin able to schedule holidays and adjust leave balances. **Confirmed Out of Scope for the current phase.**

**4. Business Objective:** Program management currently has no system of record for fellow attendance/leave — likely tracked manually or not at all — relevant to payroll/compliance and to contextualizing Point 6's performance data against days actually worked.

**5. Detailed Functional Requirements (Deferred — drafted for future scoping only):**
- Check-in/check-out: fellow records a check-in and check-out action per day. Confirmed: work status (Present / Work From Home / Absent) is fellow-selected at check-in — no location inference or other signal involved.
- Data export of attendance records (explicitly requested; export format unspecified).
- Leave management: fellows apply for leave; Admin approves; leave balances adjustable by Admin.
- Holiday calendars are state-specific — a fellow's applicable holiday list depends on their assigned state/location.

**6. UI/UX Changes Required (Deferred):** A simple daily check-in/check-out action for fellows, with a status selector (Present / WFH / Absent) at the point of check-in; a leave-application form; an Admin screen for holiday calendar setup and leave-balance adjustment.

**7. Data Requirements (Deferred):** Confirmed: the Frappe HR app is **not** currently installed on the Samavesh bench. Recommend installing it rather than building custom equivalents — it maps closely to every requirement here: Employee Checkin (check-in/out + fellow-selected status), Leave Application, Leave Type, Holiday List (assignable per location/state).

**8. Acceptance Criteria (Deferred — draft):** A fellow can check in/out and select their own status for the day; a fellow can apply for leave against their state's holiday calendar and balance; Admin can adjust balances and export attendance data.

**9. Technical Considerations (Deferred):** Since Frappe HR isn't installed yet, the first step is installing it (Occam's Razor — don't rebuild what already exists as an app) rather than building parallel custom DocTypes. Custom build should only be a fallback if installation turns out to be infeasible for infrastructure/licensing reasons.

**10. Priority Level:** Not prioritized — Out of Scope for the current phase. If scoped in: **Low** relative to the other deferred items, since it's tangential to the core scholarship-tracking mission — factor in Frappe HR installation/setup effort when this does get estimated, since it isn't pre-installed.

**11. Implementation Recommendation:** Install the Frappe HR app first — this turns most of this module into a configuration task (holiday lists per state, leave types, Employee Checkin setup) rather than new DocType design. Fully resolved — ready for CLI whenever this gets scoped in.
