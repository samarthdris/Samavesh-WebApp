# Samavesh Wireframe — Build Status (heartbeat)

Single file: `wireframe/index.html`. Role-based login (Student / Fellow / Program Admin / Mentor). Treat as the FINAL dev source of truth — everything must be clickable.

**Live (GitHub Pages, served from `dev` root):** https://samarthdris.github.io/Samavesh-WebApp/wireframe/
**Instant fallback (htmlpreview):** https://htmlpreview.github.io/?https://github.com/samarthdris/Samavesh-WebApp/blob/dev/wireframe/index.html

_Last updated: 2026-06-05 — added Mentor portal app (m-home/availability/sessions/profile)._

## Legend
- [x] done & in file
- [~] in progress
- [ ] not started

## Student section
- [x] Login (role-based, demo creds 9800000021=student / 9800000011=fellow + quick buttons)
- [x] Home (journey path MIS-06 terms), Scholarships (status tracking), Documents (status+history only), Mentorship (self-service booking), Learning (LMS placeholder), Profile (read-only)

## Fellow section
- [x] Dashboard (KPIs + workflow task list; tasks link to forms)
- [x] My Students — WORKING status tabs (All/Onboarded/Documents Pending/Submitted/Approved/Disbursed) + WORKING date filter + 10 entries across statuses
- [x] Applications — WORKING status tabs (Ready/Under Scrutiny/Approved/Benefits Received/Re-apply/Rejected) + 9 entries
- [x] Student Detail (tabs: Profile / Scholarships / Documents / Applications)
- [x] **Onboarding Form (Form 1)** — full 42Q, bilingual EN+MR, 8 steps, matrix doc checklist, conditional district, same-as-permanent, consent gate, FILLABLE (id onbForm)
- [x] **Scholarship Data Entry (Form 2)** — full 46Q, English, branch at Q25 (A/B), masked creds + eye toggle, schemes grouped, status→upload/proof conditional, 2×2 installment grid, FILLABLE (id schForm)
- [x] **Documentation Application + follow-up (Form 3, DERIVED)** — per-document apply + follow-up timeline (id docForm) — CONFIRM spec with user
- [x] Wire student detail (Docs→f-docapp, Apps→f-scholarship) + dashboard tasks to forms
- [x] Filters/steppers/branch/password-toggle JS all working

## Program Admin section (NEW)
- [x] MIS Dashboard (a-home) — 5 KPIs, MIS filters, stage funnel, fellow-wise + scholarship-wise tables, export button
- [x] Document Verification (a-verify) — queue with working Accept/Reject (reason) — the Admin-only action that closes the doc loop
- [x] Scholarship Catalogue (a-catalogue) — scheme list + working "Add Scholarship" inline form (CRUD)
- [x] All Students (a-students) — cross-fellow table with working stage + fellow filters
- [x] Fellows & Users (a-fellows) — performance table + Add User
- [x] Admin profile + sign out; admin demo login (9800000001 / "Program Admin · Meera Nair")
- [x] **Student Detail (a-student)** — dynamic per-row header (name/avatar/stage pill/assigned fellow/journey, via goStudent(this)); oversight + admin-only actions: Accept/Reject docs inline (reuses vAct), Edit profile + Reassign Fellow (audit-logged); Scholarships & Applications read-only (no operator buttons). All-Students rows now route here (were going to Verify).

## Mentor section (NEW)
- [x] Mentor portal app (.fp) — demo login 9800000031 (Dr. Meera Joshi); rail + bottom-nav; wired via ACCOUNTS/showApp/loginAs
- [x] Home (m-home) — next-session banner, monthly stats (incl. hours-used/cap), my-rating number, confidentiality note
- [x] My Availability (m-availability) — recurring weekly grid (toggle slots via mSlot; amber=booked) + one-off slots (toggleOneOff/saveOneOff)
- [x] Sessions (m-sessions) — Upcoming/Past tabs (reuse ftab); Join / Mark complete / Cancel / + Mentor note (mSession/mNote); privacy boundary = mentee first name + topic only, NO feedback shown (RBAC-03 / FR-MENT-011)
- [x] My Profile (m-profile) — editable bio/skill·stream·geo tags/languages/mode/monthly-hour-cap; Approved pill
- Only new CSS = the availability grid (.availgrid/.slot-cell). Spec+plan: docs/superpowers/*2026-06-05-mentor-interface*.

## Verification audit (2026-06-05) — done
- [x] Full audit vs workflow PDF + BRD. Confirmed aligned: roles, RBAC (student view-only), doc lifecycle, eligibility, masking, dup-check, MIS, forms (onboarding 8-step/9-doc/bilingual; scholarship branched).
- [x] FIXED: application-status vocabulary was half-migrated → now FORM vocab everywhere (Under Scrutiny / Application Approved / Benefits Received / Re-apply / Rejected). "Under Review" kept ONLY as document status (DOC-03). "Redirected for Correction" removed.
- Decisions: forms-vocab canonical · deviations (admin verify, student view-only docs) kept as-is, justified verbally via BRD refs · Frappe target = HYBRID.

## Frappe re-skin — DONE (branded "custom Frappe UI", not stock grey)
- [x] Fellow → branded Frappe Desk (.fd theme: Inter, teal sidebar + marigold accents, dot indicators, list/form styling)
- [x] Admin → same branded Frappe Desk + number-card MIS workspace
- [x] Student → branded Frappe Web Portal (.fp: Inter UI, warm brand kept, portal footer)
- [x] Higher-fidelity Desk extras (Fellow+Admin): breadcrumbs (JS via go()), list-view toolbars (count + Filter/Sort/refresh), Student Detail record view = 2-col form + right sidebar (Assigned To/Tags/Details) + Activity comment timeline
- Theme is scoped via classes: `.fd` (Desk) on #fellowApp/#adminApp, `.fp` (Portal) on #studentApp. See memory `samavesh-frappe-ui`.

## Still to do / next
- [ ] Confirm Documentation Application form design with user (derived, no MD spec)
- [ ] Documentation follow-up could become its own global queue (like Applications) if user wants
- [x] ~~Mentor role app~~ — DONE 2026-06-05 (.fp portal) · [ ] Super Admin role app (not built yet)
- [x] ~~Admin: build a real student-detail view~~ — DONE 2026-06-05 (a-student)
- [ ] Optional: add f-scholarship / f-docapp to fellow rail nav (currently reached contextually)

## Key decisions (see memory)
- Forms = source of truth (richer than BRD §10). Use form vocabulary over BRD §11 for statuses.
- Status model: see memory `samavesh-status-model`.
- Documentation Application form has NO MD spec — derived; confirm with user.

## Open questions for user
- Is there a dedicated spec/MD for the Documentation Application form, or proceed with derived version?
- Confirm status→tab mapping (Onboarded/Documents Pending/Submitted/Approved/Disbursed).
