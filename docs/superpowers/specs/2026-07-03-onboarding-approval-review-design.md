# Onboarding Approval Review + Action-Gate Cluster — Design

**Date:** 2026-07-03
**File touched:** `wireframe/index.html` (single-file wireframe)
**Status:** Awaiting user approval (brainstorming output; no code written yet)

## Problem

The Fellow's **Onboarding Approvals** screen (`#f-approvals`) lets a Fellow approve a
self-registered student's onboarding while showing only **6 of the 42 submitted fields**
(Full Name, DOB, Course/Class, Category, Mobile, District), hardcoded per card. Approving
what you cannot fully see is a broken review gate — a rubber stamp, not a review.

The full 42-question form already exists and is already cloned for the student portal via
`buildStudentOnboard()` (clones `#onbForm` → `s_`-prefixed). The approval side simply never
reused it.

### Root-cause / process note
This is not a missing skill — `samavesh-no-half-baked`, `samavesh-ripple-check`, and
`frappe-doctype-skill` (list-view → form-view) all already cover it. The miss was **altitude**:
the approvals screen was built like a notifications inbox (a scannable summary) instead of a
**review surface**. New durable principle to record in memory after this batch:

> **Review-gate rule:** any approve / verify / reject gate MUST expose the *complete* artifact
> being decided on — reusing the same component the data was captured in — before the decision
> button. A summary teaser behind an approve button is a defect.

## Scope (user-confirmed 2026-07-03)

Four items, one coherent pass:

- **A1** — Onboarding approval → full filled-form review (the flagged issue).
- **B1** — Approval completes the flow (student appears in My Students; stage advances).
- **A2** — Real document preview replacing the `alert()` "View file" stubs.
- **C1** — "Message student" → real WhatsApp `wa.me` deep-link.

Out of scope (flagged, not built): real PDF rendering; Admin `a-student` sub-document parity;
the Frappe amendment / re-open-after-approve workflow (Bucket B, blocked on client input).

## A1 — Full-submission review (list → form view)

**Chosen approach:** List view → dedicated review screen (Frappe list→form). Rejected: inline
accordion (42-field forms make the list unusable) and modal overlay (cramped; fights scroll-spy).

### `#f-approvals` becomes a real list view
Each pending submission is a compact link row — avatar · name · `Submitted {date} · self-registered · {email}` ·
*Pending Approval* pill · chevron. No inline fields, no per-card buttons. Row `onclick="openApproval(id)"`.
Keep the existing empty state (`#approvalEmpty`) and nav badge (`#fApprovalBadge`).

### New screen `#f-approval-detail` (the form view)
- Back link → `#f-approvals`.
- Header card: avatar, name, `Submitted {date} · self-registered · {email}`, *Pending Approval* pill,
  lead text: "Review the student's full onboarding submission. Edit any field to correct a
  discrepancy (audit-logged), then approve."
- Body: the **full onboarding form**, produced by cloning `#onbForm` (all 8 sections:
  support / personal / location / college / edu / docs / socio / consent) with the anchor-tab bar
  intact, IDs prefixed `rev_` to avoid colliding with the Fellow `#onbForm` and the student `s_` clone.
  Fields are **pre-filled** from the submission's answer map and left **editable**.
- Sticky bottom action bar: **Message student** (WhatsApp, see C1) + **Approve onboarding**.

### Data model
```js
// Keyed by submission id; each holds the student's full answer set.
const PENDING_ONBOARDINGS = {
  'onb-ravi':  { name:'Ravi Deshmukh',  email:'ravi.deshmukh@gmail.com', mobile:'9812345678',
                 submitted:'3 Jul 2026', avatarBg:'...', avatarChar:'R',
                 fields: { onbFullName:'Ravi Deshmukh', onbDob:'2006-03-12', onbGender:'M',
                           onbMobile:'9812345678', onbCategory:'OBC', onbCourse:'B.A Year 1',
                           onbDistrict:'Pune', /* ...all remaining §1–§8 answers... */ } },
  'onb-sneha': { /* Sneha Kale, full answer set */ },
};
```
The exact field-id ↔ value mapping is enumerated in the implementation plan (read `#onbForm`
field IDs during planning). Wireframe simplification: seed the fields that differ per student
richly; any not seeded inherit the form's sensible demo defaults, so the review always shows a
complete, populated form. This is noted as a wireframe convenience, not production behaviour.

### Functions
- `openApproval(id)` — clone+prefill `#onbForm` into `#fApprovalHost` (rebuild per id), wire
  DOB/mobile/gender/doc-status listeners + anchor smooth-scroll (same wiring `buildStudentOnboard`
  uses; factor a shared `wireOnbClone(clone, prefix)` helper), populate header, `go('f-approval-detail')`.
- `approveOnboarding(id)` — replaces the old `approveOnboarding(btn)`. See B1.

## B1 — Approval completes the flow

On **Approve onboarding** (from `#f-approval-detail`):
1. Remove the submission from `PENDING_ONBOARDINGS` and its list row; decrement `#fApprovalBadge`
   and the f-students "Pending Approval (n)" chip (`#fPendingChip`); show `#approvalEmpty` at zero.
2. **Prepend a new `.stu-row` to `#stuList`** built from the submission, `data-status="onboarded"`,
   `data-date` = today, with a brief highlight class. Bump the "All (n)" chip, "Onboarded (n)" chip,
   and the listbar count ("10 Students" → "11"). New row `onclick="go('f-student')"` (existing
   wireframe convention — all rows route to the shared detail).
3. On the detail screen, swap the action bar for a success state:
   "✓ Approved — added to your caseload as **Onboarded**" + a **View in My Students** button
   (`go('f-students')`, scroll to + flash the new row).

## A2 — Real document preview (shared modal)

Replace every `alert('View file …')` with a reusable preview modal.

- `openDocPreview(title, meta)` opens overlay `#docPreviewModal` rendering a **faux document viewer**:
  a page canvas (title, placeholder text lines, a seal/signature mock) + caption
  (`{type} · uploaded by {who} · {version}`), plainly labeled
  *"Sample preview (wireframe) — production renders the actual uploaded PDF (1–2 MB)."*
  Close button + backdrop-click to dismiss; `Esc` to close.
- Entry points updated (ripple check — ALL of them, not just the obvious four):
  - `a-verify` `#verifyList` — 4 "View file" buttons.
  - `a-student` inline verify row — 1 "View file" button (line ~3998).
  - Fellow document view/download path (`downloadDoc`) — student's docs the Fellow reviews.
- Accept/Reject remain on the row (`vAct`) — the modal is **view-only**, which keeps wiring safe.
  Optional add-on (not required): Accept/Reject buttons inside the modal that call `vAct` on the
  origin row via a stored `_previewRow` reference, then close.

## C1 — Real WhatsApp on "Message student"

"Message student" (on `#f-approval-detail`) becomes an `<a>` to
`https://wa.me/91{mobile}?text={encoded message}` — same pattern as the Student Home `.wa-cta`
buttons. Message prefilled, e.g. "Hi {name}, this is your Samavesh Fellow about your onboarding
form — ". Mobile comes from `PENDING_ONBOARDINGS[id].mobile`.

## Consistency / ripple checklist
- [ ] Doc-preview modal used by **every** view-file entry point (a-verify ×4, a-student ×1, Fellow docs).
- [ ] On approve: nav badge, f-students "Pending Approval" chip, and all list counts stay in sync.
- [ ] Review form reuses the single `#onbForm` source — no fourth copy of the form markup.
- [ ] English-only (no Marathi) — the cloned form already is.
- [ ] Every new control does something visible (no inert UI) per `samavesh-no-half-baked`.

## Verification (per `samavesh-render-verify`)
Headless-Chrome render each touched screen and exercise behaviour, not just structure:
- `?role=fellow&screen=f-approvals` — list rows; click → `f-approval-detail` shows full form pre-filled.
- Edit a field, click Approve → success state; then `f-students` shows the new highlighted Onboarded row + bumped counts.
- `?role=admin&screen=a-verify` — "View file" opens the preview modal; Accept/Reject still work.
- WhatsApp link has a valid `wa.me` href.

## Follow-up (after build, on request)
- Save the **review-gate rule** as a `feedback`-type memory.
