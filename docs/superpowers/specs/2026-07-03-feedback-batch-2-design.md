# Feedback Batch 2 (IDs 7, 10, 12 + global English-only) — Design Spec

**Date:** 2026-07-03
**Source:** client tracker `Feedbacks.xlsx` (Fellow-profile items) + follow-up requirements from user.
**Status:** design approved by user; pending spec review → implementation plan.
**Scope:** `wireframe/index.html` only. Branch `dev`. **Ask before any git commit/push** (hard rule — see [[samavesh-git-workflow]]).

## Scope summary

Three independent features + one global cleanup, built in this order:
1. **ID 7 — Fellow attendance** (login/logout + Fellow's own list) **+ Admin report + "Working now" live view**.
2. **ID 12 — Fellow notes on student profile** (student sees read-only).
3. **ID 10 — Student self-onboarding** (gated portal → submit → Fellow approves → profile unlocks) **+ rebuilt Documents section** (universal upload + search + to-and-fro with re-upload + notifications).
4. **Global — English-only:** strip all bilingual (Marathi) content + the language toggle.

Dropped: ID 8 (merge My Students/My Cases). Parked: ID 11 (Scholarship Data Entry on student view), mobile polish for student self-serve, "additional/other document" catch-all upload.

---

## Global change — English-only

Remove all secondary-language content so the UI is single-language English.
- Onboarding form (`#onbForm`): drop every `.qmr` (Marathi) label; keep the `.qen` English text as the sole label.
- Remove the **language toggle/pill** and any `setLang`-style JS.
- Sweep the rest of the file for any other bilingual strings.
- Rationale: cleaner interface now; a proper i18n strategy comes later. Reverses the "bilingual onboarding" note in [[samavesh-forms-spec]] — this is intentional and temporary.

---

## ID 7 — Attendance

**Model (adapted from the client's Frappe HRMS, relevant fields only):** a punch layer (IN/OUT events with time + date + location) rolled up into a daily record (date, in, out, gross hours, status).

**Fellow home (`#f-home`)** — a Log In / Log Out card at the top:
- One primary button that toggles **Log In → Log Out**, stamping **time + date + browser geolocation** (wireframe simulates e.g. `Pune, MH · 18.51, 73.85` via `navigator.geolocation` with a static fallback).
- Live today state: "Logged in 10:06 AM · Pune" → after logout "Logged out 6:47 PM · 8.6 hrs today."
- **"My Attendance"** list (own days only): Date · In · Out · Gross Hours · Status · Location.

**Admin — new "Attendance" screen** in the rail (WORKSPACE group):
- **"Working now" live panel at the top** (THE feature — kills the "call each agent to check" pain): count + list of fellows currently logged IN, with punch-in time + location.
- Historical report below: one row per fellow per day — Fellow · Date · Status · In · Out · Gross Hours · Location — with filters (fellow / date range / status), summary tiles (Present today · On leave · Avg hours), and **Export**.
- No late/early flags (raw times only, per user). No-punch on a working day = **Absent / No-record** (leaves handled manually).

---

## ID 12 — Fellow notes (student-visible)

On the **Fellow's student-detail** view (`#f-student`), a **Notes** section (new tab or block on the Profile tab):
- Fellow adds a note: **interaction type** (In-person / Call / Virtual) + **date/time** + free-text note. Timestamped, author = Fellow.
- Renders as a reverse-chronological note log.

On the **student's** Profile (`#profile`): the same notes appear **read-only** under "Notes from your Fellow," with a line: "Something missing? Message your Fellow on WhatsApp." Student cannot add/edit.

---

## ID 10 — Student self-onboarding + gated portal + approval + document to-and-fro

### Student journey states (the new spine)
1. **Signed up, not onboarded** — after Sign Up + OTP (built in ID 1), the portal is **gated**: only a welcome + **"Fill Onboarding Form"** button. All other sections/rail hidden.
2. **Submitted, pending approval** — after submit, gated screen switches to "Your form is with your Fellow for review." Still gated.
3. **Approved** — Fellow approves → **full portal unlocks** (Home, Scholarships, Documents, Profile).

### Demo representation (how a static wireframe shows all 3 states)
- The **Sign Up path** (`verifySignupOtp`) lands on a **new gated student** (states 1 → 2) instead of the full portal.
- The existing **"Student" login (Aarti)** remains the **already-approved** student (state 3 — the full portal built so far).
- This makes all three states clickable with no hidden toggle.

### Student onboarding form
- Reuse the existing 42-question form (`#onbForm`, now English-only) rendered inside the student portal.
- Submittable only when mandatory fields are filled (enforces the no-reject rule).
- **Prominent Save-draft + Resume** for the student path (long form, likely multi-sitting).

### Fellow approval (Both surfaces, per user)
- New **"Onboarding Approvals"** rail item (Fellow) with a **count badge** — queue of submitted forms (student · submitted date · Review).
- Also a **"Pending Approval"** status + tab on **My Students** (`#f-students`).
- Opening a submission shows the form **in editable mode**: Fellow fixes discrepancies inline, then **Approve** (no reject on the form; coordinate over WhatsApp). Approve → student → approved; assigns the student to the approving Fellow.

### Documents — rebuilt (grouped + search + upload + to-and-fro)
- Student Documents (`#documents`) keeps **Must-Have / Good-to-Have grouping**; adds a **search box** (filter by document name) and an **Upload** action on **every** document row.
- **"Have it" vs "Don't have" are MUTUALLY EXCLUSIVE** (corrected 2026-07-03 per user). Each outstanding doc shows a **"Do you have this document?" [Have it | Don't have]** toggle:
  - **Have it** → the main-document **Attach is enabled**; the sub-documents panel is **hidden**.
  - **Don't have** → the main-document Attach **fades/disables** (you can't attach what you don't have); the **sub-documents panel appears** so the Fellow can procure it.
  - Only ONE path is active at a time — never both. Uploading the main doc (in "Have it") resolves the doc (→ Uploaded · pending review) and removes the toggle + sub-docs.
- This toggle only appears on **outstanding docs the student must provide** (e.g. Ration, Pending). Docs already on record (Accepted / Under Review / Uploaded) show no toggle and no sub-docs.
- **To-and-fro loop with a return path:**
  - Student uploads → doc flips to **"Uploaded · pending Fellow review"**; **Fellow notified**; in the Fellow view the doc **floats to top** with a **"New"** badge.
  - Fellow opens it → **Download/view** → **Accept** *or* **Request re-upload (with reason)** → bounces back to the student as **"Action needed."** (Reuses the existing Rejected → re-upload lifecycle.)
- **"Whose turn" label** on every document, both sides: `Action needed: upload` / `Uploaded · with your Fellow` / `Accepted` / `Re-upload requested: <reason>`.

### Notifications (both roles — closes the loop)
- **Fellow** (existing bell → real list + badge): new onboarding submission; student uploaded/re-uploaded a document.
- **Student** (bell/indicator): onboarding **form approved → profile unlocked**; document **Accepted**; document **re-upload requested**.

### Woven-in extras (user's additional points)
- **Fellow downloads** every document from the student's onboarding responses (to fill the govt portal) — Download action in the Fellow's doc view.
- Fellow document-activity notifications (above).

---

## Reconciliation with Batch 1 (already built)
- **ID 1 (signup+OTP):** `verifySignupOtp` currently routes to the full portal; ID 10 changes it to route to the **gated new-student** state.
- **ID 5 (Don't-have sub-docs):** unchanged; now nested inside the rebuilt Documents section as the "don't have it" path.
- **ID 3/4:** the Home + Scholarships surfaces are the state-3 (approved) portal — untouched except they now sit behind the approval gate for a new student.

## Constraints
- English-only (this batch).
- Keep our status vocab: Pending → Uploaded → Under Review → Accepted (+ N/A); document return path uses the existing **Rejected** state.
- Only BRD/SOP/tracker vocabulary; "IP" = Implementation Partner (Fellow).
- Every control functional (no inert UI / no `prompt()`), per [[samavesh-no-half-baked]]; ripple across student + Fellow + Admin surfaces per [[samavesh-ripple-check]].
- `wireframe/index.html` only. Branch `dev`. **Ask before every commit/push.**

## Out of scope / parked
- ID 8 (dropped). ID 11 (Scholarship Data Entry reflected on student view — later).
- Mobile polish for student self-serve (web-first standing rule; known follow-up).
- "Additional / other document" catch-all upload (add later if needed).
- Real language/i18n strategy (revisit after English-only cleanup).
