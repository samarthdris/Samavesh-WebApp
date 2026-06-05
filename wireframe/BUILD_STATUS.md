# Samavesh Wireframe — Build Status (heartbeat)

Single file: `wireframe/index.html`. Role-based login (Student / Fellow). Treat as the FINAL dev source of truth — everything must be clickable.

**Live (GitHub Pages, served from `dev` root):** https://samarthdris.github.io/Samavesh-WebApp/wireframe/
**Instant fallback (htmlpreview):** https://htmlpreview.github.io/?https://github.com/samarthdris/Samavesh-WebApp/blob/dev/wireframe/index.html

_Last updated: 2026-06-04 — working through the big Fellow build._

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

## Verification audit (2026-06-05) — done
- [x] Full audit vs workflow PDF + BRD. Confirmed aligned: roles, RBAC (student view-only), doc lifecycle, eligibility, masking, dup-check, MIS, forms (onboarding 8-step/9-doc/bilingual; scholarship branched).
- [x] FIXED: application-status vocabulary was half-migrated → now FORM vocab everywhere (Under Scrutiny / Application Approved / Benefits Received / Re-apply / Rejected). "Under Review" kept ONLY as document status (DOC-03). "Redirected for Correction" removed.
- Decisions: forms-vocab canonical · deviations (admin verify, student view-only docs) kept as-is, justified verbally via BRD refs · Frappe target = HYBRID.

## NEXT (greenlit): Frappe hybrid re-skin
- [ ] Student → Frappe Web Portal look
- [ ] Fellow → Frappe Desk (workspace sidebar, List View w/ filter bar + status dots, Form View w/ sectioned fields + Save + activity timeline)
- [ ] Admin → Frappe Desk + number-card dashboard
- Approach: build a Frappe-Desk CSS layer + list/form components, convert app-by-app (Fellow first), preserving all content/journeys/working filters.

## Still to do / next
- [ ] Confirm Documentation Application form design with user (derived, no MD spec)
- [ ] Documentation follow-up could become its own global queue (like Applications) if user wants
- [ ] Mentor + Super Admin role apps (not built yet)
- [ ] Admin: build a real student-detail view (currently admin rows route to Verify)
- [ ] Optional: add f-scholarship / f-docapp to fellow rail nav (currently reached contextually)

## Key decisions (see memory)
- Forms = source of truth (richer than BRD §10). Use form vocabulary over BRD §11 for statuses.
- Status model: see memory `samavesh-status-model`.
- Documentation Application form has NO MD spec — derived; confirm with user.

## Open questions for user
- Is there a dedicated spec/MD for the Documentation Application form, or proceed with derived version?
- Confirm status→tab mapping (Onboarded/Documents Pending/Submitted/Approved/Disbursed).
