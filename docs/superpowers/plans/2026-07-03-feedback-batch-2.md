# Feedback Batch 2 (English-only · ID 7 · ID 12 · ID 10) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply Batch 2 of the client tracker to `wireframe/index.html`: make the UI English-only, add Fellow attendance (+ Admin report with a "Working now" live view), add Fellow→student notes, and re-architect the student journey into self-onboarding (gated portal → Fellow approval → unlocked profile) with a two-way document exchange.

**Architecture:** Single-file wireframe (`wireframe/index.html`), inline `<style>`/`<script>`. No build, no test runner — **verification is manual browser click-through** per task + the standing rules (`samavesh-no-half-baked`, `samavesh-ripple-check`). Design spec: `docs/superpowers/specs/2026-07-03-feedback-batch-2-design.md`.

**Tech Stack:** Hand-written HTML/CSS/vanilla JS. Reuse existing tokens/patterns (`.card`, `.pill`, `.btn`, `.dtable`, `.tabpane`/`ftab`, `.stu-row`, `go`, `loginAs`, `showApp`, the batch-1 `SUBDOCS`/`renderSubdocList`, and the `.subdoc-*` styles).

## Global Constraints

- **English-only:** no secondary-language (Marathi) text anywhere. Remove all `.qmr` blocks + any language toggle. (This batch; i18n revisited later.)
- **Status vocab unchanged:** Pending → Uploaded → Under Review → Accepted (+ N/A). The document return path uses the existing **Rejected** state ("Request re-upload").
- **Only BRD/SOP/tracker vocabulary;** "IP" = Implementation Partner (the Fellow).
- **Every control must work** (no inert UI, no `prompt()`); ripple across Student + Fellow + Admin surfaces.
- **`wireframe/index.html` only.** Branch `dev`.
- **🚦 ASK THE USER BEFORE EVERY `git commit`/`git push`.** The commit step in each task is a *checkpoint to request approval*, NOT an autonomous commit. (Hard rule — see `samavesh-git-workflow`.)
- Line numbers drift as tasks land — **anchor every edit on a unique string**, re-reading the region first.

---

### Task 1: Global — English-only cleanup

**Files:** Modify `wireframe/index.html` — the onboarding form `#onbForm` (35 `.qmr` blocks), any language toggle/pill, and a sweep for stray bilingual strings.

**Interfaces:**
- Consumes: nothing.
- Produces: an English-only `#onbForm` (labels are the former `.qen` text only). No `.qmr` nodes remain; no `setLang`/language-pill code remains.

- [ ] **Step 1: Inventory the bilingual markup.** Grep `qmr`, `qen`, `lang`, `setLang`, `language` to list every occurrence and the toggle (if any). Confirm the count (~35 `.qmr`).
- [ ] **Step 2: Remove Marathi label nodes.** For each question, the pattern is `<div class="qen">English</div><div class="qmr">मराठी</div>`. Delete the `<div class="qmr">…</div>` sibling, keeping `.qen`. Do this for all occurrences (the `.qen`/`.qmr` pairing is uniform — safe as a scoped `replace_all` on the wrapper if the inner text differs, else per-block).
- [ ] **Step 3: Remove the language toggle** (pill/button + its `onclick`/JS `setLang`), if present. If `.qen`/`.qmr` CSS rules exist, leave `.qen` styling, drop `.qmr` rules.
- [ ] **Step 4: Sweep** for any other Devanagari/bilingual strings outside the form (hero quotes are English already; verify). Remove/EN-only them.
- [ ] **Step 5: Verify in browser.** Open the onboarding form (Fellow → + Onboard New Student): every question shows a single English label, no Marathi, no toggle, layout intact (no empty gaps where `.qmr` was). No console errors.
- [ ] **Step 6: Checkpoint — ASK USER before committing.** Proposed message: `Batch 2 Task 1: English-only cleanup (removed bilingual labels + language toggle)`.

---

### Task 2: ID 7 — Fellow attendance (Log In/Out + My Attendance)

**Files:** Modify `#f-home` (top of the section, anchor: `<section class="screen active" id="f-home">` → the `.hero` block) + add CSS + JS.

**Interfaces:**
- Consumes: existing `.card`, `.pill`, `.dtable`, `.btn`.
- Produces (used by Task 3): `ATT_LOG` array of `{fellow, date, inTime, outTime, gross, status, location}`; `fellowPunch(btn)` toggling IN/OUT; `FELLOW_LOGGED_IN` bool; location helper `getGeo(cb)`.

- [ ] **Step 1: Add attendance CSS** (near other portal styles):

```css
.att-card{display:flex;align-items:center;justify-content:space-between;gap:16px;flex-wrap:wrap;border-left:4px solid var(--teal-500)}
.att-state{font-size:13.5px;color:var(--ink-soft)}
.att-state b{color:var(--ink)}
.att-btn{font-family:inherit;font-weight:700;font-size:14px;border:none;border-radius:9px;padding:12px 22px;cursor:pointer;transition:.15s}
.att-btn.in{background:var(--green);color:#fff}
.att-btn.out{background:var(--red);color:#fff}
.att-dot{display:inline-block;width:8px;height:8px;border-radius:50%;margin-right:6px}
```

- [ ] **Step 2: Add the Log In/Out card** as the first block inside `#f-home` (before the KPI grid):

```html
<div class="card att-card" id="fAttCard">
  <div>
    <div class="eyebrow">Attendance</div>
    <div class="att-state" id="fAttState">You're <b>logged out</b>. Log in to start your day.</div>
  </div>
  <button class="att-btn in" id="fAttBtn" onclick="fellowPunch(this)">Log In</button>
</div>
```

- [ ] **Step 3: Add a "My Attendance" list** lower in `#f-home` (after the existing "Recent students" block) — a `.dtable` (Date · In · Out · Gross Hours · Status · Location) seeded with ~6 demo rows for Rahul, newest first, plus one `On Leave` and one `Absent` row for realism.
- [ ] **Step 4: Add attendance JS:**

```javascript
// ===== ID 7 attendance =====
var FELLOW_LOGGED_IN = false;
function getGeo(cb){
  // Wireframe: try the browser, fall back to a static Pune location.
  if(navigator.geolocation){
    navigator.geolocation.getCurrentPosition(
      function(p){ cb('Pune, MH · '+p.coords.latitude.toFixed(2)+', '+p.coords.longitude.toFixed(2)); },
      function(){ cb('Pune, MH · 18.51, 73.85'); }, {timeout:2000});
  } else cb('Pune, MH · 18.51, 73.85');
}
function fellowPunch(btn){
  getGeo(function(loc){
    var now = new Date(); // Date() is fine at runtime in the browser (not in workflow scripts)
    var t = now.toLocaleTimeString('en-IN',{hour:'2-digit',minute:'2-digit'});
    var st = document.getElementById('fAttState');
    if(!FELLOW_LOGGED_IN){
      FELLOW_LOGGED_IN = true;
      btn.className='att-btn out'; btn.textContent='Log Out';
      st.innerHTML='<span class="att-dot" style="background:var(--green)"></span>Logged in <b>'+t+'</b> · '+loc;
    } else {
      FELLOW_LOGGED_IN = false;
      btn.className='att-btn in'; btn.textContent='Log In';
      st.innerHTML='Logged out <b>'+t+'</b> · thanks for today.';
    }
  });
}
```

- [ ] **Step 5: Verify in browser.** Fellow → Home: "Log In" → button turns red "Log Out", state shows time + location (allow/deny geolocation both work via fallback). Click again → logged out. "My Attendance" list renders with statuses. No console errors.
- [ ] **Step 6: Checkpoint — ASK USER before committing.** `Batch 2 Task 2: ID 7 Fellow attendance — Log In/Out + My Attendance`.

---

### Task 3: ID 7 — Admin Attendance report + "Working now" live view

**Files:** Add a new `#a-attendance` screen inside `#adminApp`; add a rail nav item (anchor: Admin rail, after `data-go="a-tasks"`); add to `CRUMBS`.

**Interfaces:**
- Consumes: `.dtable`, `.card stat`, filter patterns from `a-students`.
- Produces: `#a-attendance` screen; nav item routes via existing `[data-go]` handler.

- [ ] **Step 1: Add the rail nav item** in the Admin WORKSPACE group (after Cases & Tasks): `<a href="#" class="nav-link" data-go="a-attendance">…Attendance</a>` (reuse an appropriate SVG). Add matching entry to `botnav` if the admin bottom-nav lists it.
- [ ] **Step 2: Add `#a-attendance` screen** with:
  - `.page-h` "Attendance".
  - **"Working now" panel** (top): a `.card` with a heading "Working now · <n> logged in" and a list of currently-IN fellows (avatar · name · "in since 10:06 AM" · location). Seed 3 demo fellows IN.
  - Summary tiles (`.grid g-3`): Present today · On leave · Avg hours.
  - Filters row (fellow / date / status selects — wire with a live "Filtered by…" note like `misFilter`).
  - Historical `.dtable`: Fellow · Date · Status · In · Out · Gross Hours · Location (seed ~12 rows across fellows/days incl. On Leave + Absent).
  - An **Export** button (reuse the MIS export stub pattern).
- [ ] **Step 3: Add to `CRUMBS`**: `'a-attendance':'Attendance'`.
- [ ] **Step 4: Verify in browser.** Admin → Attendance: "Working now" shows the 3 IN fellows; filters update the note; Export fires its stub; table renders. No console errors.
- [ ] **Step 5: Checkpoint — ASK USER before committing.** `Batch 2 Task 3: ID 7 Admin attendance report + Working-now`.

---

### Task 4: ID 12 — Fellow notes on student profile (student read-only)

**Files:** Modify `#f-student` (add a Notes block/tab) + the student `#profile` (read-only notes) + CSS + JS.

**Interfaces:**
- Consumes: `.card`, `.tl` timeline styling, `.tabpane`/`ftab`.
- Produces: `addFellowNote(btn)` + inline note form; `FELLOW_NOTES` seed array `{type, when, text}`; a renderer used on both the Fellow and student sides.

- [ ] **Step 1: Add notes CSS** (`.note-item`, `.note-meta`, `.note-type` pill for In-person/Call/Virtual, reusing token colors).
- [ ] **Step 2: Fellow side** — add a **"Notes"** area on `#f-student` (either a new `ftab` pane `fp-notes` alongside Profile/Eligible/Documents/Applications, or a block on the Profile tab). Include:
  - An "Add note" inline form: interaction-type `<select>` (In-person / Call / Virtual), a date-time `<input type="datetime-local">`, a `<textarea>`, Save/Cancel. `addFellowNote` prepends a `.note-item` (newest first) and clears the form. No `prompt()`.
  - Seed 2 demo notes.
- [ ] **Step 3: Student side** — on `#profile`, add a read-only **"Notes from your Fellow"** block rendering the same note items (no add/edit controls) + helper line: "Something missing? Message your Fellow on WhatsApp."
- [ ] **Step 4: Verify in browser.** Fellow → open student → Notes: add a Call note dated now → appears on top with type pill + timestamp. Student → Profile: same notes appear read-only, no edit controls. No console errors.
- [ ] **Step 5: Checkpoint — ASK USER before committing.** `Batch 2 Task 4: ID 12 Fellow notes (student read-only)`.

---

### Task 5: ID 10 — Gated student portal + signup routing

**Files:** Modify the student app shell (`#studentApp`, rail `1210`, botnav `1821`), add a gated "Welcome / Fill Onboarding" screen, and re-point `verifySignupOtp` (from batch-1 ID 1).

**Interfaces:**
- Consumes: `loginAs`, `showApp`, `go`, `verifySignupOtp` (batch 1).
- Produces: `STUDENT_STAGE` (`'new'` | `'submitted'` | `'approved'`); `applyStudentGate()` that shows/hides rail links + screens per stage; `startAsNewStudent()` used by signup.

- [ ] **Step 1: Add a gated screen** `#s-gate` inside `#studentApp` (a `.screen`): welcome heading, one-paragraph explainer (fill form → Fellow reviews → profile unlocks), and a primary **"Fill Onboarding Form"** button (`data-go` to the student onboarding screen from Task 6). A second visual state (submitted) shows "Your form is with your Fellow for review" — toggled by `STUDENT_STAGE`.
- [ ] **Step 2: Add gating JS:**

```javascript
// ===== ID 10 student journey gate =====
var STUDENT_STAGE = 'approved'; // default demo student (Aarti) is already approved
function applyStudentGate(){
  var gated = (STUDENT_STAGE !== 'approved');
  // hide/show rail + botnav links for the full portal when gated
  document.querySelectorAll('#studentApp .rail .nav-link[data-go], #studentBot .nav-link[data-go]').forEach(function(a){
    var keep = a.dataset.go==='s-gate' || a.getAttribute('onclick'); // keep logout
    a.style.display = (gated && !keep) ? 'none' : '';
  });
  if(gated) go('s-gate');
  var sub = (STUDENT_STAGE==='submitted');
  var g1=document.getElementById('sGateFill'), g2=document.getElementById('sGatePending');
  if(g1&&g2){ g1.style.display=sub?'none':''; g2.style.display=sub?'':'none'; }
}
function startAsNewStudent(){ STUDENT_STAGE='new'; loginAs('student'); applyStudentGate(); }
function submitOnboarding(){ STUDENT_STAGE='submitted'; applyStudentGate(); window.scrollTo(0,0); }
```

- [ ] **Step 3: Re-point signup** — change batch-1 `verifySignupOtp` so its success path calls `startAsNewStudent()` instead of `loginAs('student')`. The existing **"Student" demo shortcut/login stays `loginAs('student')` with `STUDENT_STAGE='approved'`** (reset it in `loginAs` for the student role, or in the demo button).
- [ ] **Step 4: Ensure `loginAs('student')` from the demo path resets `STUDENT_STAGE='approved'` and calls `applyStudentGate()`** so the two entry points don't leak state between logins.
- [ ] **Step 5: Verify in browser.** From login → **Sign up** → email → OTP → Verify → lands on **gated** portal (only "Fill Onboarding Form", rail hidden). The **Student demo login** → full approved portal (unchanged). No console errors.
- [ ] **Step 6: Checkpoint — ASK USER before committing.** `Batch 2 Task 5: ID 10 gated student portal + signup routing`.

---

### Task 6: ID 10 — Student onboarding form in the portal (self-fill + save-draft/resume)

**Files:** Add a student onboarding screen reusing the `#onbForm` markup (English-only after Task 1) inside `#studentApp`; wire submit → `submitOnboarding()`.

**Interfaces:**
- Consumes: `#onbForm` structure, `applyStudentGate`, `submitOnboarding` (Task 5), existing `saveDraft` (batch-1 audit added it).
- Produces: `#s-onboard` screen with a submit that gates the portal to `submitted`.

- [ ] **Step 1: Add `#s-onboard`** — the same single-page onboarding form (reuse the `.dt-form` markup pattern from `#onbForm`; a trimmed clone is fine for the wireframe as long as sections + mandatory markers match). Sticky action bar: **Save draft** (prominent, calls `saveDraft`) + **Submit for review** (calls `submitOnboarding`, only after a mandatory-field check).
- [ ] **Step 2: Mandatory gate** — a `validateOnboarding()` that checks required fields and blocks submit with an inline message if incomplete (no `prompt()`); on success → `submitOnboarding()`.
- [ ] **Step 3: Resume affordance** — the gated `#s-gate` "Fill Onboarding Form" button reads "Resume Onboarding Form" if a draft exists (a `localStorage`-free demo flag `ONB_DRAFT=true` set by `saveDraft`).
- [ ] **Step 4: Verify in browser.** Signup → gated → Fill Onboarding → try Submit empty (blocked with message) → fill mandatory → Save draft (confirmation; gate button now says Resume) → Submit → gate switches to "with your Fellow for review". No console errors.
- [ ] **Step 5: Checkpoint — ASK USER before committing.** `Batch 2 Task 6: ID 10 student onboarding self-fill + save-draft/resume`.

---

### Task 7: ID 10 — Fellow approval (inbox + My Students status + editable review → approve)

**Files:** Add `#f-approvals` screen + Fellow rail item (after `data-go="f-tasks"`); add "Pending Approval" to `#f-students` status tabs + a demo row; add `CRUMBS` entry.

**Interfaces:**
- Consumes: `.dtable`/`.stu-row`, `ftab`, the `#onbForm` review markup.
- Produces: `#f-approvals` screen; `approveOnboarding(btn)` that marks a submission approved and updates the badge; `PENDING_APPROVALS` count for the rail badge.

- [ ] **Step 1: Rail item** "Onboarding Approvals" with a count badge (reuse the `.soon`-style badge used on Cases & Tasks) after My Cases. Add `botnav` entry if applicable.
- [ ] **Step 2: `#f-approvals` screen** — a list of submitted onboarding forms (student · submitted date · **Review**). Review opens the submitted form **in editable mode** (reuse `#onbForm` layout, values pre-filled) with a sticky **Approve** action (`approveOnboarding`). On approve: remove from the queue, decrement the badge, show a confirmation, and (demo) note the student is now assigned to this Fellow.
- [ ] **Step 3: My Students** — add a **"Pending Approval"** chip to the `filterStu` tabs + a `data-status="pending"` demo row that routes to the review. Ensure `filterStu('pending',…)` works (it filters by `data-status`).
- [ ] **Step 4: `CRUMBS`** += `'f-approvals':'Onboarding Approvals'`.
- [ ] **Step 5: Verify in browser.** Fellow → Onboarding Approvals (badge shows count) → Review a submission → edit a field → Approve → row leaves queue, badge decrements, confirmation shown. My Students → "Pending Approval" tab filters correctly. No console errors.
- [ ] **Step 6: Checkpoint — ASK USER before committing.** `Batch 2 Task 7: ID 10 Fellow onboarding approval (inbox + My Students + review/approve)`.

---

### Task 8: ID 10 — Student Documents rebuild (search + upload on every row + "whose turn")

**Files:** Modify student `#documents` (grouped list from batch 1) — add a search box, an Upload action per main-doc row, and a "whose-turn" label; keep the batch-1 `.subdoc-*` "Don't have" nesting.

**Interfaces:**
- Consumes: batch-1 `SUBDOCS`/`renderSubdocList`/`toggleDontHave`, `.pill`, `.doc-row`/`.arr-card`.
- Produces: `docSearch(q)` filter; `uploadMainDoc(btn, key)` that flips a row to "Uploaded · pending review" + sets a `data-turn="fellow"` + enqueues a Fellow notification (Task 9); `DOC_TURN` label helper.

- [ ] **Step 1: Add doc-search + whose-turn CSS** (`.doc-search` input; `.turn-pill` variants: `need` amber "Action needed: upload", `fellow` blue "With your Fellow", `done` green "Accepted", `reupload` red "Re-upload requested").
- [ ] **Step 2: Search box** at the top of `#documents`: `<input class="doc-search" oninput="docSearch(this.value)" placeholder="Search a document by name…">`; `docSearch` shows/hides `.card`/`.arr-card` doc rows by matching the `<b>` doc name; empty-state note when nothing matches.
- [ ] **Step 3: Upload on every main-doc row (ALWAYS available)** — add an **Upload** button (reuse `.attach-btn` / `.btn-gold btn-sm`) to every document row that isn't already Accepted. **This Upload is present regardless of the "Don't have" toggle** — a student may have obtained the doc after onboarding, so direct upload must never be gated behind "Don't have". `uploadMainDoc` flips the row: pill → "Uploaded", small text → "Uploaded just now — sent to your Fellow for review", `.turn-pill` → `fellow`, calls `notify('fellow', …)` (Task 9), and — if "Don't have" was toggled on for that doc — clears it and collapses the sub-doc panel (the doc is now provided, so procurement help is moot). Reuse/extend the existing `fileAct` pattern. Apply the Upload to the 4 sub-doc cards (Ration/Caste/Domicile/Income) too, so the Ration example in the screenshot gets a direct Upload alongside its "Don't have" panel.
- [ ] **Step 4: Whose-turn labels** — add a `.turn-pill` to each of the target doc rows reflecting current state (e.g. Ration = "Action needed: upload"; Domicile = "With your Fellow"; Income = "Accepted"). Keep the batch-1 "Don't have" sub-doc panel nested and functional.
- [ ] **Step 5: Verify in browser.** Student → Documents: search "income" → only Income shows; clear → all return. Upload on a Pending doc → flips to Uploaded + "With your Fellow" + (Task 9) a Fellow notification appears. "Don't have" sub-doc panel still works. No console errors.
- [ ] **Step 6: Checkpoint — ASK USER before committing.** `Batch 2 Task 8: ID 10 student Documents — search + upload + whose-turn`.

---

### Task 9: ID 10 — To-and-fro loop: Fellow review (float-to-top + Accept/Request re-upload), notifications both sides, onboarding-doc download

**Files:** Modify Fellow `#fp-docs` (float-to-top + New badge + Accept/Request re-upload + Download) and the notification bell for both roles.

**Interfaces:**
- Consumes: `uploadMainDoc`/`DOC_TURN` (Task 8), batch-1 fellow sub-doc panels, `showNotifications` (existing bell stub).
- Produces: `notify(role, msg)` + badge counters (`NOTIF={fellow:[],student:[]}`); `fellowDocAction(btn, 'accept'|'reupload')` with an inline reason form for re-upload; a "New" badge + top-sort for freshly-uploaded docs.

- [ ] **Step 1: Notifications model** — replace the `showNotifications` stub with a real dropdown/list per role: `notify(role,msg)` unshifts into `NOTIF[role]`, bumps a bell badge; opening the bell lists items. Wire the existing bell(s) to render `NOTIF[currentRole]`.
- [ ] **Step 2: Fellow doc review actions** — in `#fp-docs`, each student-uploaded/newly-uploaded doc shows **Download**, **Accept**, and **Request re-upload**. `fellowDocAction(btn,'accept')` → pill Accepted + `notify('student','<doc> accepted')`. `fellowDocAction(btn,'reupload')` → inline reason textarea → on save, pill Rejected + turn-pill "Re-upload requested: <reason>" + `notify('student', …)`. No `prompt()`.
- [ ] **Step 3: Float-to-top + New badge** — when a doc is (demo) freshly uploaded by the student, render it at the top of the Fellow's `#fp-docs` list with a **"New"** badge; provide one seeded "New" doc so the behaviour is visible without a round-trip.
- [x] **Step 4: Fellow can Download every document on record (DONE early, 2026-07-03)** — RULE: any doc with a file (status Accepted / Under Review / Uploaded) shows a **Download** action at ALL times in `#fp-docs`; docs with no file yet (Pending / Rejected-awaiting-reupload / N/A) show their action or nothing. Implemented `downloadDoc(name)` + Download on Aadhaar/Income/10th/12th/Caste (Accepted) + Domicile (Under Review); sub-docs download from their panel. When Ration/NCL flip to Uploaded via the to-and-fro (Steps 2-3), their Download must appear too — wire it in `uploadMainDoc`/`fellowDocAction`.
- [ ] **Step 5: Student-side notification wiring** — student bell shows: onboarding **approved**, doc **Accepted**, doc **re-upload requested**; the matching document's `.turn-pill` updates to `need`/`done` accordingly.
- [ ] **Step 6: Verify in browser.** Student uploads a doc → Fellow bell badge increments → Fellow opens student → doc is at top with "New" → **Request re-upload** with a reason → student bell shows it, that doc shows "Re-upload requested: <reason>" (Action needed). Re-run with **Accept** → student sees "Accepted". Fellow **Download** works on onboarding docs. No console errors.
- [ ] **Step 7: Checkpoint — ASK USER before committing.** `Batch 2 Task 9: ID 10 to-and-fro loop + notifications + onboarding-doc download`.

---

## Self-Review

**Spec coverage:**
- English-only → Task 1 ✓
- ID 7 (Fellow attendance + Admin report + **Working now**) → Tasks 2, 3 ✓
- ID 12 (Fellow notes, student read-only) → Task 4 ✓
- ID 10 gated portal + signup routing → Task 5 ✓; student self-fill form + save-draft/resume → Task 6 ✓; Fellow approval (Both surfaces, editable review, no reject) → Task 7 ✓; Documents rebuild (grouped + search + upload + whose-turn, ID 5 nested) → Task 8 ✓; to-and-fro (float-to-top, Accept/Request re-upload, notifications both sides, onboarding-doc download) → Task 9 ✓
- Improvements: Working-now (T3), Request re-upload (T9), student notifications + whose-turn (T8/T9), prominent save-draft/resume (T6) — all mapped ✓
- Parked (ID 11, mobile, "other doc" catch-all) — intentionally absent ✓

**Placeholder scan:** New JS/CSS shown; HTML blocks specified with anchors + seed-data instructions (execution reads current state first, per the batch-1 model). No "TBD/add validation".

**Type consistency:** `STUDENT_STAGE` values (`new`/`submitted`/`approved`) consistent across Tasks 5–7. `notify(role,msg)` + `NOTIF` defined in Task 9, produced there and consumed by Task 8's `uploadMainDoc` (Task 8 calls `notify('fellow',…)`; Task 9 defines `notify` — **build order note:** Task 8's upload will no-op its notification until Task 9 lands; acceptable since tasks commit sequentially and Task 9 completes the loop. Alternatively define the `notify` stub in Task 8 and enrich in Task 9). `fellowDocAction(btn, mode)` and `.turn-pill` variants (`need`/`fellow`/`done`/`reupload`) consistent between Tasks 8 and 9.

**Build-order dependency fix:** Move the minimal `notify`/`NOTIF` definition into **Task 8 Step 1** (stub: unshift + badge), and have Task 9 Step 1 *enrich* the bell rendering. This removes the forward reference. (Apply at execution.)
