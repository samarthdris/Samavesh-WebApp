# Samavesh Wireframe — Build Status (heartbeat)

Single file: `wireframe/index.html`. Role-based login (Student / Fellow / Program Admin / Mentor). Treat as the FINAL dev source of truth — everything must be clickable.

**Live (GitHub Pages, served from `dev` root):** https://samarthdris.github.io/Samavesh-WebApp/wireframe/
**Instant fallback (htmlpreview):** https://htmlpreview.github.io/?https://github.com/samarthdris/Samavesh-WebApp/blob/dev/wireframe/index.html

_Last updated: 2026-06-09 — Student journey re-wired end-to-end (multi-app status, Also Eligible, doc grouping, functional filters)._

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

## Workflow diagram (NEW) — client-facing
- [x] `wireframe/workflow.html` — standalone branded algorithmic flowchart in the style of `Samavesh workflow.pdf` (green action / amber screen / blue ◆ decision / pink note / teal phase; legend)
- [x] Program Overview (4 phases + mentoring parallel; Admin verification gate added + status vocab reconciled — both pink-annotated)
- [x] 4 role flows (Student/Fellow/Program Admin/Mentor) mapped to built screens, with per-role RBAC notes; 27 amber screen-nodes deep-link into the live wireframe
- [x] Deep-link handler in index.html: `index.html?role=&screen=` auto-logs-in + navigates (6-line on-load reader; uses existing loginAs/go)
- [x] Cross-links: login screen → "View the role workflows" link to workflow.html; workflow footer → back to wireframe
- Spec+plan: docs/superpowers/*2026-06-05-workflow-diagram*.

## Student journey re-wire (2026-06-09) — done

All the patterns proven in the Fellow iteration applied to the Student section. No new UX inventions — every pattern is now exercised on at least three surfaces (Fellow / Admin / Student).

- [x] **Home** — single "My Scholarship Journey" 6-step bar replaced with **multi-application status block** (same `.scholar-apps` pattern as f-student/a-student). Each scholar-row is clickable → navigates to Scholarships screen + scrolls + outlines the matching card (`studentSelectApp`). Hero copy updated from singular ("Your Post-Matric scholarship is Under Scrutiny") to plural ("You have 3 scholarship applications").
- [x] **Scholarships** — restructured into two clearly-distinct sections (no redundancy with Home):
  - **My Active Applications** (3 detailed cards with timelines + status, the 3 `data-app` deep-link targets)
  - **Also Eligible — Your Fellow Could Apply** (3 info cards — GoI Post-Matric / RC Shahu Maharaj Merit / EBC Tuition Fees — read-only with eligibility criteria, no Apply CTA since students don't apply per BR-05)
- [x] **Documents** — **Must Have / Good to Have grouping** rippled from the Fellow's Documents tab. Same `.doc-group-banner` + `.dgb-pill` styling. 4 Must Have (Aadhaar/Income/10th SLC/12th SLC) + 5 Good to Have (Caste/Non-Creamy Layer/Domicile/Ration/Orphan). Rich expandable cards retained for Pending + Under Review + a rich rejected-then-accepted Income Cert history. Stats grid updated to match the canonical 9-doc list (1/1/5).
- [x] **Mentorship** — filter chips (Career / Higher Studies / Engineering / Scholarships / Commerce / Marathi / All) are now **functional**. Each mentor card has `data-tags`; `filterMentors()` toggles visibility with a chip-on highlight and empty state. "Join (link soon)" button no longer inert — explains video-link delivery timing.
- [x] **Login** — unified login (Google SSO + magic link) already serves all roles cleanly; no student-specific changes needed (students have compulsory Gmail at onboarding per [[samavesh-brd-discrepancies]] so Google SSO is the natural primary CTA).
- All patterns now consistent across Fellow / Admin / Student surfaces. No new memory needed — `samavesh-form-pattern`, `samavesh-ripple-check`, `samavesh-no-half-baked` already cover the rules applied this round.

## Consistency fixes II — Eligible Schemes + functional Applications (2026-06-09) — done

Three substantial fixes flagged by user; saved `samavesh-no-half-baked` memory for next time.

- [x] **Redundancy removed**: The "Scholarships" tab on student-detail no longer duplicates the top "Scholarship Applications" block. Renamed to **"Eligible Schemes"** — shows catalogue-matched schemes grouped into:
  - **Already Applied** (3 schemes with "View application details →" deep-link)
  - **Eligible — Not Yet Applied** (3 schemes with "+ Apply for this" CTA → opens Scholarship Data Entry form)
  Each row shows eligibility criteria matched.
- [x] **Top "Scholarship Applications" rows are now clickable** — clicking PM/MS/AB navigates to the Applications tab, scrolls to that app's card, briefly outlines it. Works on both `f-student` (Fellow) and `a-student` (Admin) views.
- [x] **Applications tab buttons are now fully functional** (Fellow's view):
  - Status dropdown (`updateAppStatus`) updates the pill in real time using the canonical form vocabulary.
  - "Add correction note" (`addCorrectionNote`) opens an inline textarea with Save/Cancel; Save replaces the form with a styled note view.
  - "Record Disbursement & Close" (`recordDisbursement`) validates inputs, updates the pill to "Benefits Received", replaces the form with a confirmation banner.
  - Each card has `data-app` attribute for deep-linking.
- [x] **Rippled to a-student** (Admin view): same Eligible Schemes structure (read-only — no Fellow CTAs), clickable scholar-rows, data-app on Applications cards. Added the previously-missing 3rd application card (Dr. Babasaheb Ambedkar) for consistency.

## Consistency fixes (2026-06-09) — done
User flagged 4 issues that should have been caught earlier (saved as `samavesh-ripple-check` memory for next time).

- [x] **Scroll-spy bug fixed** — `IntersectionObserver`-based generic `makeScrollSpy()` replaces the brittle scroll-listener-on-guessed-container approach. Container-agnostic. Filters `.hidden` sections (path A/B). Works on both Form 1 and Form 2.
- [x] **Radio rule sharpened** — "ONLY 2 options stay as radios". 3+ → dropdown. Applied to Onboarding §7 ("Do parents have any disability?", "Any drug-addiction problem at home?"). Form 2 already conformed.
- [x] **Multi-scholarship per-app status** — single "Scholarship Journey" replaced with "Scholarship Applications" block on both `f-student` (Fellow view) and `a-student` (Admin oversight). One row per scheme: logo + name + App ID + status pill + 6-dot mini-funnel + "N of 6" step. Demo shows Aarti's 3 applications at different stages.
- [x] **Documents Must Have / Good to Have grouping** rippled to `f-student` Documents tab and `a-student` Documents tab — same 4+5 categorisation as the Onboarding form §6 child-table. New `.doc-group-banner` styling reused.

## Form anchor nav → horizontal top tabs (2026-06-09) — done
- [x] `.dt-anchors` reskinned from vertical 220px left sidebar → horizontal pill tab bar at the top of the form body
- [x] `.dt-form` grid collapsed (no more 2-column layout for the anchors); form gets full width (~30% breathing room gain)
- [x] Active-tab styling: teal-50 pill bg + teal-700 text + filled teal-700 numbered chip
- [x] Sticky positioning preserved (tabs follow you as you scroll); scroll-spy + click-to-smooth-scroll unchanged
- [x] Scroll-spy threshold bumped 140→170, section `scroll-margin-top` bumped 80→150 to account for the sticky tab bar
- [x] Applied automatically to both Form 1 (`#onbForm`) and Form 2 (`#schForm`) — no HTML changes needed, CSS-only restyle
- [x] Form 3 (`#docForm`) untouched — it uses `.dt-form.simple` which has no anchors
- Skill: `frappe-dashboard-design` ("zero whitespace waste") + `frappe-doctype-skill` (real Frappe Tab Break is horizontal at top, not a side nav)

## All Fellow forms redesigned (2026-06-09) — done

The same Frappe DocType pattern is now applied to ALL three Fellow forms (Forms 1, 2, 3). No multi-step "Continue" wizards left in the Fellow section.

### Form 2 — Scholarship Data Entry (#schForm)
- [x] 7-step wizard → single-page with left anchor sidebar (7 sections); branch sections (Path A/B) toggle by `pickBranch`
- [x] **DOB picker + auto-age** (replaces old Age dropdown)
- [x] **Mobile validate chip** (Indian 10-digit regex)
- [x] **Gender dropdown** populated from the **same `GENDER_MASTER`** as the onboarding form — proves the master-DocType pattern (one source of truth, multiple consumers)
- [x] **26-scheme grouped radio list** → Frappe **`<select>` with `<optgroup>`** (categorised, single dropdown — huge vertical-space win)
- [x] All other radio lists ≥4 options → dropdowns (Category, Annual Income, Marks, Reference, College Area, etc.)
- [x] Branch selection (Scholarship Support vs Technical Support) hides/shows the relevant sections + anchor links
- [x] Sticky bottom Save bar with branch label

### Form 3 — Documentation Application (#docForm)
- [x] Reskinned to `.dt-form.simple` (single-column variant) for visual consistency with Forms 1 & 2
- [x] `.form-card` → `.dt-section` with numbered anum chips
- [x] Save/Cancel → sticky `.dt-actions` footer
- [x] Already single-page (no stepper); just visual consistency this round
- [x] Activity timeline (`.tl`) preserved — already Frappe-credible

### Form 1 — Student Onboarding (#onbForm) — already done earlier this session
- [x] Single-page Frappe DocType form replaces the 8-step "Continue" wizard
- [x] Left sticky **anchor sidebar with scroll-spy** (8 sections preserved as anchors, not steps)
- [x] **DOB picker** → auto-calculated **age in years (1 decimal)** displayed inline (replaces old Age dropdown)
- [x] **Mobile validated** as Indian 10-digit (regex `/^[6-9]\d{9}$/`); shows ✓ Valid / ✗ Invalid chip on blur
- [x] **Gender as master** — dropdown driven by `GENDER_MASTER` array (5 entries, codes M/F/NB/O/PNS)
- [x] **Documents child-table** — grouped Must Have / Mandatory (4 docs) and Good to Have / Scholarship-specific (5 docs); status dropdown + attach button (enabled only when status="Have it")
- [x] All radio lists with ≥4 options converted to **dropdowns** (Social Background, Annual Income, Family Occupation, How heard)
- [x] **Duplicate-check changed**: Name+DOB-or-Mobile → **Email OR full Name**
- [x] Sticky bottom Save/Submit bar
- Spec+plan: `docs/superpowers/specs/2026-06-09-onboarding-form-redesign.md`, `docs/superpowers/plans/2026-06-09-onboarding-form-redesign.md`

### Shared infrastructure
- New CSS scope: `.dt-form` / `.dt-anchors` / `.dt-section` / `.dt-grid` / `.dt-actions` / `.dob-inline` / `.dob-age` / `.mobile-input` / `.mobile-verify` / `.doc-ctable` — all introduced this session, reused across forms.
- Old `.formwrap` / `.fstep` / `.stepcount` / `.fprogress` / `.fnav` / `.matrix` CSS still present but no longer used by any Fellow form. Safe to clean up later when no form uses them.
- `formGo` engine still exists but is now orphaned (Forms 1/2 don't use it; Form 3 never did). Can be removed in a future cleanup.

### Skills applied
- `frappe-doctype-skill` — Date / Phone-validated Data / Link-to-master / Select / Table child-table / Select-with-optgroup fieldtypes; Section Break + Column Break form layout; branch as `depends_on` (sections show/hide by data attribute).
- `frappe-dashboard-design` — Inter type scale, light theme, 4px spacing, button hierarchy, status-chip palette.

## Onboarding form redesign (2026-06-09) — superseded by "All Fellow forms" above
- [x] Single-page Frappe DocType form replaces the 8-step "Continue" wizard
- [x] Left sticky **anchor sidebar with scroll-spy** (8 sections preserved as anchors, not steps)
- [x] **DOB picker** → auto-calculated **age in years (1 decimal)** displayed inline (replaces old Age dropdown)
- [x] **Mobile validated** as Indian 10-digit (regex `/^[6-9]\d{9}$/`); shows ✓ Valid / ✗ Invalid chip on blur
- [x] **Gender as master** — dropdown driven by `GENDER_MASTER` array (5 entries, codes M/F/NB/O/PNS); same array will become the future Frappe Gender DocType, reusable across all forms
- [x] **Documents child-table** — grouped Must Have / Mandatory (4 docs) and Good to Have / Scholarship-specific (5 docs); each row has Status dropdown + Attach button (enabled only when status="Have it")
- [x] All radio lists with ≥4 options converted to **dropdowns** (Social Background, Annual Income, Family Occupation, How heard, etc.)
- [x] **Duplicate-check changed**: Name+DOB-or-Mobile → **Email OR full Name** (matches SSO email-as-primary-identifier)
- [x] **Sticky bottom action bar**: Save draft (secondary) + Submit Onboarding (primary)
- [x] No "Continue" buttons anywhere; user scrolls the whole form on a single page
- [x] Form section 2-column grid layout (Frappe Column Break equivalent) — Personal section uses (Full Name full-width) → (DOB+age, Gender) → (Mobile, Form Date) etc.
- [x] **Forms 2 (Scholarship Data Entry) and 3 (Documentation Application) untouched** — they keep the `.fstep`/`formGo` wizard for now
- Spec+plan: `docs/superpowers/specs/2026-06-09-onboarding-form-redesign.md`, `docs/superpowers/plans/2026-06-09-onboarding-form-redesign.md`
- Skills applied: `frappe-doctype-skill` (Date / Phone-validated Data / Link-to-master / Select / Table child-table fieldtypes; Section Break + Column Break form layout), `frappe-dashboard-design` (Inter type scale, light theme, 4px spacing, button hierarchy, status-chip palette). `frappe-custom-html-block` not applicable here (form is not a CHB).

## SSO login (2026-06-09) — done
- [x] `wireframe/index.html` login redesigned: mobile-number/OTP → unified email-based sign-in
- [x] **Continue with Google** (primary) + **magic-link** for non-Google emails (secondary)
- [x] All 4 roles use the same login surface; role derived from the User's Role Profile post-auth (production model documented in spec)
- [x] Fake Google OAuth picker overlay (wireframe-only) for client demo realism
- [x] `ACCOUNTS` re-keyed to lowercase email (4 Google + 1 magic-link demo for the non-Google path)
- [x] Demo shortcuts kept, restyled as a tertiary section labelled "Demo shortcuts (this wireframe)"
- [x] Pitch panel (left teal/marigold) kept — brand surface, not a dashboard surface
- [x] Deep-link handler `?role=&screen=` unchanged — workflow.html links continue to work
- Spec+plan: `docs/superpowers/specs/2026-06-09-sso-login-design.md`, `docs/superpowers/plans/2026-06-09-sso-login.md`
- Skills applied: `frappe-doctype-skill` (data-model doc), `frappe-dashboard-design` (tokens/light theme/button hierarchy). `frappe-custom-html-block` not applicable here (login isn't a CHB).

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
