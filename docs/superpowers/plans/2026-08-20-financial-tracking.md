# Financial Tracking — Implementation Plan

**Goal:** A separate "Financial Tracking" tab on the Fellow and Admin student-detail screens, plus a read-only summary on the Student's Scholarships cards, backed by one `FIN` object keyed by the existing 4 `data-app` values. Fellow enters, Admin and Student read. Locked vocabulary (First/Second Installment · Student/Institute · the three existing statuses). Gated on `Application Approved · funds awaited` / `Benefits Received`. First Installment mandatory, Second optional.

**Architecture:** One JS object `FIN` (4 applications × 4 entries: `s1`/`i1`/`s2`/`i2`), one render function per mode (editable grid / read-only grid / compact student line), `.fin-host[data-app]` containers, and one `initFinAll()` on load — the same engine-plus-hosts pattern as Modules 2 (`DOC_NOTES`) and 3 (`APP_PDF`), so cross-surface propagation is free.

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build. Verification = render + click-through.

**Design doc:** `docs/superpowers/specs/2026-08-20-financial-tracking-design.md`

## Global constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** — ask before every commit, ask again before any push.
- **Fellow is the only editor.** No Save button, no file input, no dropdown on Admin or Student.
- **Reuse, don't rewrite:** `recordDisbursement()` (line 5296, currently dead code) for validation; `.instgrid`/`.amt`/`.rs` styles from §7A; `ftab()` for the new tabs.
- **No inert UI, no `prompt()`.** Every control does something when used.
- **Do not touch** application statuses, the Path A/B branch logic, the "₹ Unlocked" column, or the student-list "₹71,000".

---

## Task 1: `FIN` engine + CSS

**Files:** `wireframe/index.html` — CSS immediately before `</style>` (after Module 3's `.app-pdf-row` block); JS immediately after Module 3's `saveSchFormPdf()` (~line 5052) and before `var NOTIF = {`.

- [ ] **Step 1: CSS** — `.fin-block`, `.fin-locked`, `.fin-cell`, `.fin-req`, `.fin-line` (student compact line), `.fin-saved` flash. Reuse `.instgrid`/`.amt`/`.rs` for the grid itself; add nothing that overrides them.
- [ ] **Step 2: `FIN` state** — 4 application keys × 4 entry keys, each `{amount:'', date:'', status:'', proof:''}`, all empty at start (per the design doc: nothing is seeded, because none of Aarti's applications is Approved yet).
- [ ] **Step 3: Status source** — `finAppStatus(key)` reads the application's current status from its existing status pill in `#fp-apps` (single source; the Fellow's dropdown already updates that pill via `updateAppStatus`), so the gate can never disagree with what's on screen.
- [ ] **Step 4: Renderers** — `renderFinBlock(key, mode)` where mode is `edit` (Fellow) / `view` (Admin) / `line` (Student compact). Locked state returns the unlock message for all three modes. `refreshFin(key)` re-renders every host for that application; `initFinAll()` renders all hosts on load.
- [ ] **Step 5: Save** — `saveFinInstallment(btn, key, inst)` validates First Installment (amount + date + proof) by delegating to the existing `recordDisbursement()` validation shape, writes into `FIN`, calls `refreshFin(key)`, shows the saved flash. Second Installment skips validation entirely.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Financial Tracking: FIN engine + renderers + gate"` — ask first.

---

## Task 2: Fellow — new "Financial Tracking" tab

**Files:** `wireframe/index.html`.

- [ ] **Step 1: Tab button.** After line 2730 (`ftab(this,'fp-apps')` / "Applications"), add `<button onclick="ftab(this,'fp-fin')">Financial Tracking</button>`.
- [ ] **Step 2: Tab pane.** Insert a new `<div class="tabpane" id="fp-fin">` immediately after `#fp-apps` closes (line 3095, before `<div class="activity">` at 3096). Inside: a lead line, then 4 `<div class="fin-host" data-app="pm|ms|ab|smm" data-mode="edit"></div>` in the same order as the Applications accordion.
- [ ] **Step 3: Render — verify.** `?role=fellow&screen=f-student` → Financial Tracking tab: 4 blocks, all showing "Financial Tracking unlocks once this application is Approved", no editable control visible.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Financial Tracking: Fellow tab (4/4 applications)"` — ask first.

---

## Task 3: Fellow — unlock, fill, save

**Files:** `wireframe/index.html`.

- [ ] **Step 1: Wire the gate to the existing status change.** In `updateAppStatus()`, after it updates the pill, call `refreshFin(key)` for that application so the tab unlocks the moment the Fellow moves PM to "Application Approved · funds awaited". Read the function first — it must keep doing everything it already does.
- [ ] **Step 2: Render — verify the live gate.** Applications tab → PM "Update status…" → "Application Approved · funds awaited" → Financial Tracking tab: PM's block editable, MS/AB/SMM still locked.
- [ ] **Step 3: Render — verify save + validation.** Fill First Installment (Student) amount + date + proof → Save → value appears in the grid. Clear the date → Save → refuses, naming the date. Leave Second Installment empty → Save → accepted, no complaint.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Financial Tracking: live gate + First-Installment validation"` — ask first.

---

## Task 4: Admin — read-only tab

**Files:** `wireframe/index.html`.

- [ ] **Step 1: Tab button.** After line 3763 (`ftab(this,'ap-apps')`), add `<button onclick="ftab(this,'ap-fin')">Financial Tracking</button>`.
- [ ] **Step 2: Tab pane.** New `<div class="tabpane" id="ap-fin">` immediately after `#ap-apps` closes (~line 4040, before `<div class="activity">` at 4041), with 4 `.fin-host[data-app][data-mode="view"]` hosts.
- [ ] **Step 3: Render — verify.** `?role=admin&screen=a-student` → Financial Tracking tab: same numbers the Fellow entered, **zero** Save buttons, zero file inputs, zero dropdowns. Locked blocks show the same unlock message without the Fellow-facing wording.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Financial Tracking: Admin tab (4/4, read-only)"` — ask first.

---

## Task 5: Student — read-only summary on the Scholarships cards

**Files:** `wireframe/index.html`, `#scholarships` (lines ~1555–1620).

- [ ] **Step 1:** Add `<div class="fin-host" data-app="pm" data-mode="line"></div>` to PM's `.sch2-side`, directly under the Module 3 `.app-pdf-host`. Repeat for `ms`, `ab`, `smm` (4 inserts).
- [ ] **Step 2: Render — verify.** `?role=student&screen=scholarships` — a recorded PM entry shows as one compact line (amount · status · date · View proof); un-recorded entries show nothing at all; no upload control, no dropdown, no Save anywhere on this screen. The GP/EBC "eligible, not applied" cards carry no `data-app` and must stay untouched.
- [ ] **Step 3 (GATED): commit.** `git add wireframe/index.html && git commit -m "Financial Tracking: Student Scholarships summary (4/4)"` — ask first.

---

## Task 6: Contain the duplicate grid (§7A) — **only if Shweta approves it**

**Files:** `wireframe/index.html`, `#sch-fin` (lines 3411–3426).

Per the design doc's "duplication problem": leave §7A visually identical, but bind its 8 inputs to the same `FIN` object so the two grids can never show different numbers. If Shweta declines, skip this task entirely and record in BUILD_STATUS that the form's grid stays inert and intentionally disconnected.

- [ ] **Step 1:** Add `data-fin-app` / `data-fin-entry` attributes to the 8 existing inputs — no layout, label or style change.
- [ ] **Step 2:** `onchange` on each writes into `FIN` and calls `refreshFin()`.
- [ ] **Step 3: Render — verify.** Type into §7A → the Fellow tab, Admin tab and Student card all show it. Then verify the reverse direction.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Financial Tracking: bind Data Entry §7A grid to the shared FIN data"` — ask first.

---

## Task 7: Full-sweep verify + heartbeat + commit

**Files:** `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Consistency grep.** `.fin-host` count = 12 (4 applications × 3 surfaces); `data-mode="edit"` appears exactly 4 times and only inside `#fp-fin`; zero Save buttons / file inputs inside `#ap-fin` and `#scholarships`.
- [ ] **Step 2: Vocabulary grep.** Zero occurrences of "Tranche", "Institution" (as a level label) or "Pending" as a disbursement status anywhere in the new markup — the three locked deviations from the feedback wording.
- [ ] **Step 3: Behaviour sweep.** Full click-through of all 7 checks in the design doc's Verification section, with screenshots of the three surfaces.
- [ ] **Step 4: Update heartbeat.** Dated section in `wireframe/BUILD_STATUS.md`: what shipped, the three vocabulary deviations and why, the Approved-gate demo path, the §7A containment decision (whichever way it went), and the still-hardcoded "₹ Unlocked" / "₹71,000" watch-item. Source: `New Feedbacks/Point 4_Financial_Tracking_Section.md`.
- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html wireframe/BUILD_STATUS.md "New Feedbacks/Point 4_Financial_Tracking_Section.md" && git commit -m "Heartbeat: Financial Tracking complete"` — this is also where the feedback `.md` finally enters the repo, per Shweta's instruction that it lands once the module is built. Then ask separately about pushing.

---

## Self-Review (against the design doc)

- **Spec coverage:** engine + gate → Task 1. Fellow entry → Tasks 2–3. Admin read-only → Task 4. Student read-only → Task 5. Duplication containment → Task 6 (conditional). Verification + docs → Task 7.
- **Placeholder scan:** every task names real line anchors and real ids; no TBD.
- **Open decision carried, not buried:** Task 6 is explicitly conditional on Shweta's answer, and the design doc states both outcomes.
- **Vocabulary:** locked wording enforced by an explicit grep in Task 7, not just by intent.
- **Ripple check:** all 4 applications on all 3 surfaces in the same round; the two hardcoded ₹ figures are deliberately out of scope and recorded as a watch-item rather than silently left.
- **Constraint:** every commit GATED; push is a separate, later gate.
