# Admin Student-Detail View — Design Spec

**Date:** 2026-06-05
**File touched:** `wireframe/index.html` (single-file clickable prototype)
**Scope:** Add a Program Admin student record view, closing the known rough edge where
Admin "All Students" rows route to the global Document Verification queue instead of the
student's own record.

## Problem

In the admin shell (`#adminApp`), every row of **All Students** (`#a-students`) currently
fires `onclick="go('a-verify')"`. Clicking any student dumps the Admin into the global
verification queue rather than opening that student's record. The Admin has no dedicated
student-detail screen (heartbeat "Still to do" item).

## Decision (user, 2026-06-05)

- **Build approach:** A new dedicated `a-student` screen adapted from the Fellow record
  view (`f-student`) structure ("Approach A"), with a light dynamic touch ("light C") so
  the header reflects the clicked row.
- **Action scope:** *Oversight + admin-only actions.* Read-only on everything the Fellow
  operates; surfaces only the two genuine Admin actions plus governance:
  - **Accept / Reject** each document (the Program Admin's RBAC action — `samavesh-fellow-journey`).
  - **Edit profile** (profile is editable by assigned Fellow and Program Admin, audit-logged).
  - **Reassign Fellow** (governance control in the sidebar).
  - **No** Fellow operator actions (Upload / Re-upload / Submit & Record / Update status /
    Record Disbursement) — those remain only in the Fellow view.

This honors the RBAC split in memory `samavesh-fellow-journey`: operator verbs live with the
Fellow; the Admin gets oversight + Accept/Reject + profile edit + reassignment.

## Architecture & routing

1. **New section:** `<section class="screen" id="a-student">` added inside `#adminApp`'s
   `<main class="content">`, after `#a-students`. Reuses the existing `.rec-layout` →
   `.rec-main` + `.rec-side` scaffolding and the `.fd` branded Desk theme (inherited, no new CSS).

2. **Back-link:** `‹ Back to All Students` via `data-go="a-students"` (matches the
   `.backlink` pattern used in `f-student`).

3. **Row routing:** In `#a-students` `<tbody>`, change each row from
   `onclick="go('a-verify')"` to `onclick="goStudent(this)"`.

4. **New helper `goStudent(row)`** (added to the existing `<script>` near `go()`):
   - Reads `row.dataset.stage`, `row.dataset.fellow`, the student name text from the first
     cell, and the avatar `<span class="sa">` (initial + inline gradient style).
   - Writes name, avatar, stage pill, and assigned-fellow name into the `a-student` header
     and sidebar via `textContent` / `innerHTML` on stable element IDs
     (`asName`, `asAvatar`, `asStagePill`, `asFellow`).
   - Drives the journey path's "current" step from the stage (maps stage → step index;
     stages: onboarded / documents / submitted / approved / disbursed).
   - Sets `CRUMBS['a-student'] = 'Students › ' + name` *before* calling `go('a-student')`,
     so the breadcrumb shows the correct student name. (`setCrumb` already reads `CRUMBS`.)
   - Resets the admin record to its **Profile** tab on open (adds `on` to `ap-profile` +
     its tab button, clears the others). `ftab()` toggles `.on` across *all* `.tabpane`s
     globally, so without this reset the view could land on a stale tab left active by a
     prior visit. Harmless to the (hidden) Fellow panes.

5. **Breadcrumb:** `a-student` resolves through the existing `crumbAdmin` element because its
   id starts with `a-` (existing `setCrumb` logic). No change to `setCrumb`.

6. **MIS dashboard tables:** unchanged. The Fellow-wise / Scholarship-wise tables are about
   fellows and schemes, not individual students, so they keep their current `go()` targets.

7. **Tab bodies** stay representative sample data (same convention as the Fellow `f-student`
   record, which is a single static record). Only the *header + sidebar identity* is dynamic.

## The four tabs (role lens)

Same tab bar and `ftab()` switcher as the Fellow view. Tab pane ids are admin-scoped to
avoid colliding with the Fellow panes (`ap-profile`, `ap-sch`, `ap-docs`, `ap-apps`).

| Tab | Admin treatment |
|---|---|
| **Profile** | Full read of profile fields (DOB, gender, category, religion, income, state/district, course/year, last exam %, mobile, masked Aadhaar, guardian, consent). Header note "Editable by the assigned Fellow and Program Admin · all edits are audit-logged" + an **Edit profile** ghost button. |
| **Scholarships** | Read-only eligibility view — matched schemes with per-scheme document %/status pills. **No** "Submit Application" / "Manage documents" buttons. |
| **Documents** | Each doc row shows a status pill + metadata. Rows in **Under Review** get the real `[View file] [Reject] [Accept]` buttons calling the existing `vAct(this,'accept'|'reject')`. Accepted / Rejected / Pending rows are status-only (Pending = "awaiting Fellow upload", not an Admin action). |
| **Applications** | Read-only tracking: per-application status pill + timeline (Submitted → Under Scrutiny → Application Approved → Benefits Received; or Re-apply / Rejected). A muted note states submission / status updates / disbursement are performed by the assigned Fellow. **No** Submit / Update-status / Record-disbursement controls. |

Status vocabulary follows the canonical model (`samavesh-status-model`): application statuses use
form/SOP vocabulary (Under Scrutiny / Application Approved / Benefits Received / Re-apply /
Rejected); document statuses use DOC-03 (Pending / Uploaded / Under Review / Accepted / Rejected).

## Sidebar, activity, journey

- **Journey path** (reused `.journey` / `.path`): 6-step funnel (Onboarded → Scholarship
  Identified → Documents Complete → Submitted → Approved → Disbursed), current step driven by
  the passed-in stage.
- **Right sidebar** (`.rec-side`):
  - **Assigned Fellow** card → fellow name (`asFellow`, from `data-fellow`) + a **Reassign**
    ghost button (prototype `alert()`/inline note, "audit-logged").
  - **Tags** card (category / district / scheme chips).
  - **Details** card (Created / Last modified / Student ID).
- **Activity timeline** (reused `.activity`): cross-role events (Fellow onboarded, Fellow
  uploaded, Program Admin accepted X, status changes). Comment-box avatar = admin "M".

## Reuse summary

- **CSS:** none new — reuses `.rec-layout`, `.rec-main`, `.rec-side`, `.journey`, `.path`,
  `.tabs`, `.tabpane`, `.doc-row`, `.pill`, `.activity`, `.side-card`, `.backlink`, `.btn*`.
- **JS:** reuses `go()`, `setCrumb()`, `CRUMBS`, `ftab()`, and `vAct()` (Accept/Reject).
  One new function: `goStudent(row)`. The `[data-go]` delegated click handler covers the
  back-link and Reassign-less links automatically.

## Out of scope

- No new dynamic data for tab bodies (representative sample data, matching Fellow view).
- No changes to the Fellow record view, the verification queue, or the MIS dashboard logic.
- Mentor / Super Admin role apps; Documentation form; LMS — all unaffected.

## Verification

Manual, in-browser (this is a clickable prototype, treated as final dev source of truth):
1. Sign in as Admin (`9800000001`). Open **All Students**.
2. Click each of the 6 rows → lands on `a-student` with that row's **name, avatar, stage
   pill, assigned fellow, breadcrumb, and journey current-step** all matching the row.
3. Tabs switch via `ftab()`; **Documents** Under-Review rows Accept/Reject via `vAct` (pill
   flips, buttons clear), exactly like the verification queue.
4. Profile shows **Edit profile**; Scholarships/Applications show **no** operator buttons.
5. Sidebar shows the correct assigned Fellow + **Reassign**; Activity timeline renders.
6. Back-link returns to **All Students**; breadcrumb reads `Home › Students › <name>`.
7. Fellow and Student apps still behave exactly as before (no regression from shared
   `ftab`/`vAct`/`CRUMBS`).
