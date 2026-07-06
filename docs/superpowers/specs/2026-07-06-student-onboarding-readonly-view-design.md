# Student Read-Only Onboarding View — Design

**Date:** 2026-07-06
**Status:** Approved (design) — awaiting spec review
**Scope:** `wireframe/index.html` (single-file wireframe). Student portal only (+ one entry in the Fellow notifications list).
**Related memory:** `samavesh-feedback-batch2` (ID 11 context + credentials landmine), `samavesh-approval-review-cluster` (the reusable form-clone component), `samavesh-use-only-context-terms`, `samavesh-no-half-baked`, `samavesh-render-verify`, `samavesh-git-workflow`, `samavesh-ripple-check`.

## Problem

The student fills the onboarding form during self-onboarding (`#s-onboard`), submits it, the Fellow reviews/approves it (`#f-approval-detail`), and the portal unlocks. But **the student can never see what they submitted** — the form is editable before submit and hidden after. The client asked (last walkthrough) that the student's filled onboarding response be reflected back to the student **in view-only mode**, reusing the Fellow's onboarding-approvals full-form review interface.

This is the onboarding form — the only form a *student* fills. (The Scholarship Data Entry form is filled by the Fellow, holds government-portal credentials, and is explicitly out of scope.)

## Goals

- The student can view their **complete submitted onboarding form, read-only**, on their portal.
- Reuse the existing form-clone component (one source of truth for the 42-Q form) — no second copy of the markup.
- Improve the flow: let the student peek at their submission **while it is under review**, give a **"Request a change"** path (RBAC-safe), and stamp the record with **who approved it and when**.
- Never expose editing to the student (view-only per RBAC).

## Non-goals (YAGNI)

- No change to the editable self-fill (`#s-onboard`) or the Fellow approval screen (`#f-approval-detail`) — both only *reused*.
- No student edit rights.
- The Scholarship Data Entry form is untouched (separate surface, credentials).
- No new domain vocabulary.

## The form's third render mode

The onboarding form is already rendered in two modes; this adds a third:

| Mode | Where | Helpers |
|---|---|---|
| Student editable self-fill | `#s-onboard` (`buildStudentOnboard`) | `cloneOnbForm('s_')` + `wireOnbClone` |
| Fellow editable review | `#f-approval-detail` (`openApproval`) | `cloneOnbForm('rev_')` + `wireOnbClone` + `prefillOnbClone` |
| **Student read-only (NEW)** | `#s-my-onboarding` (`buildMyOnboarding`) | `cloneOnbForm('myonb_')` + `wireOnbClone` + `prefillOnbClone` + **`lockOnbClone` (new)** |

### `lockOnbClone(clone)` — new helper
Makes a cloned+prefilled form non-editable:
- Disable every `input`, `select`, `textarea` inside the clone (`el.disabled = true`).
- Hide the form's sticky action bar (`clone.querySelector('.dt-actions').style.display = 'none'`).
- Make the documents child-table static: disable the doc-status `select`s and hide/disable the `.attach-btn` buttons (view-only; a small "—" or the recorded status remains visible).
- Keep the anchor-tab bar and scroll navigation working (read-only navigation is fine).

Order in `buildMyOnboarding`: `cloneOnbForm('myonb_')` → append to host → `wireOnbClone(clone,'myonb_')` → `prefillOnbClone(clone,'myonb_',MY_ONBOARDING)` (this fires the change/blur events that compute the DOB→age and ✓Valid chips) → `lockOnbClone(clone)` (disable everything last).

### Data
`MY_ONBOARDING` — the logged-in student's submitted answers, same shape as the approval submissions (`{name, submitted, approvedBy, approvedDate, fields:{baseId:value}, docs:[statusValues], checks:{baseName:[optionIndices]}}`). Seeded with Aarti Pawar's onboarding answers so the read-only view shows real content. `buildMyOnboarding` guards with a `dataset.built` flag (build once).

## Surfaces

### 1. Home card (post-approval) — primary placement
On student Home (`#home`), a card: **"Your onboarding submission"** with a sub-line "Submitted <date> · Approved by <Fellow> on <date>" and a **"View my onboarding form →"** button → `go('s-my-onboarding')`. Home is only reachable once approved, so this card is inherently the post-approval record.

### 2. Read-only screen `#s-my-onboarding` (new)
Inserted after `#s-onboard`. Structure:
- **Back-link** — context-aware: returns to `home` when approved, or to `s-gate` when still under review.
- **Header card with the stamp** (context-aware by `STUDENT_STAGE`):
  - `approved` → "✓ Approved by <Fellow> on <date>" (green).
  - `submitted` → "⏳ Submitted <date> · under review by your Fellow" (amber).
- **Host `#sMyOnbHost`** — the locked, pre-filled 42-Q form (built by `buildMyOnboarding`).
- **"Request a change" action bar** (§3).
- `go()` hook: when `id==='s-my-onboarding'`, call `buildMyOnboarding()` (mirrors the `s-onboard`→`buildStudentOnboard` hook) and set the stamp/back-link per `STUDENT_STAGE`.
- `CRUMBS`: add `'s-my-onboarding'`.

### 3. "Request a change" (RBAC-safe loop)
On `#s-my-onboarding`, a **Request a change** button → reveals an inline form (textarea "What needs correcting? Your Fellow will update it.") + Send / Cancel. No `prompt()`. On **Send**:
- Replace the form with a green confirmation: "✓ Sent to your Fellow — they'll review and update your form."
- **Push a notification into the Fellow's `NOTIF` list** (reuse the batch-2 notifications system): "Aarti Pawar requested a change to her onboarding form." So the loop is visible on the Fellow side (not a dead-end). The Fellow edits via the already-built approval/edit screen.

### 4. Peek during "under review"
On the gate's pending card (`#sGatePending`), add a **"Review what you submitted →"** link → `go('s-my-onboarding')`. Reachable while gated (it's the student's own submission; `go()` shows the screen without unlocking the rail). The screen's stamp + back-link render the "under review" variant.

## Production note (Frappe)

The student role gets **read-only field permission** on their own Onboarding record (view-only per RBAC — students never edit their record directly). "Request a change" creates a change-request / comment routed to the assigned Fellow (who has edit permission). The onboarding form contains **no** government-portal credentials, so nothing is masked here (unlike the Scholarship Data Entry form).

## Edge cases

- **Reached while `submitted` (gated):** rail stays hidden; the read-only screen shows with the "under review" stamp; back-link → `s-gate`. Approving later flips the stamp to "Approved by…".
- **Build-once:** `buildMyOnboarding` no-ops if already built (`#sMyOnbHost` `dataset.built`), but the **stamp + back-link are re-set every `go()`** so switching stages shows the right variant.
- **Request-a-change re-open:** after sending, the button/form is replaced by the confirmation (no duplicate submissions in one view).
- **No inert UI:** every added control does something visible (opens the view / sends the request / pushes the notification).

## Verification (per `samavesh-render-verify`)

Headless-Chrome (`--headless=new`, isolated `--user-data-dir`), read each PNG:
1. **Home card → read-only form** (`?role=student&screen=s-my-onboarding`, approved): full 42-Q form pre-filled with Aarti's answers, **all fields disabled**, no Save/Submit bar, "Approved by Rahul More on <date>" stamp.
2. **Request a change**: inject the click → inline textarea → Send → green confirmation; confirm a Fellow `NOTIF` entry was added.
3. **Pending peek**: with `STUDENT_STAGE='submitted'`, the pending gate card shows "Review what you submitted →"; opening it shows the same form with the "under review" stamp + back-link to the gate.

## Decisions (record)

1. Target form = **onboarding** (the only form the student fills); Scholarship Data Entry out of scope.
2. Placement = **card on Home** (post-approval record).
3. Enhancements included: **peek during under-review**, **Request a change**, **Approved-by stamp** (all three).
4. Read-only via a new `lockOnbClone` on the reused clone (one form, third mode) — no second markup copy.
5. "Request a change" pushes a **Fellow notification** to close the loop; students remain view-only (RBAC).
