# Wireframe Consistency + Polish Batch — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Fix the audit's cross-surface data discrepancies (A1–A4) and add the approved UX polish (A5/B2 inline Edit-profile, B5 Fellow profile, B6 discoverability) — value-alignment only, no refactor.

**Architecture:** Hand-align Aarti's demo values across the existing surfaces; add a generic inline `editProfile`/`saveProfile`/`cancelProfile` (works on any `.card` with `.field/.k/.v` grid, skipping `.mask` + `[data-noedit]`), reused by the student profile (Fellow + Admin) and the Fellow's own profile. All in `wireframe/index.html`.

**Tech Stack:** Vanilla HTML/CSS/JS, one file. Verification = headless-Chrome render.

## Global Constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit**; **ask before pushing** (memory `samavesh-git-workflow`). Commit only `wireframe/index.html` (+ `BUILD_STATUS.md` last task); never `git add -A`.
- **Value-align only** — no structural refactor (spec decision 1).
- **Canonical Aarti values:** DOB **14 Aug 2005**; marks **78% (prev 74%)**; apps = PM *Under Scrutiny*, MS *Ready to Submit*, AB **Documents Pending**, SM *Re-apply*; docs = 5 Accepted / 1 Under Review / 1 Pending / 2 N/A of 9.
- **No inert UI / no `prompt()`** (memory `samavesh-no-half-baked`); reuse existing inline-form style.
- **Render-verify behavior** (memory `samavesh-render-verify`): `--headless=new` + `--user-data-dir="C:\Temp\cprof"`; read each PNG.

### Render snippet
```powershell
Copy-Item "C:\Users\Jitesh (DHW-L06)\Desktop\Samavesh\wireframe\index.html" "C:\Temp\swf.html" -Force
try { Get-Process chrome -ErrorAction Stop | Stop-Process -Force } catch {}; Start-Sleep -Milliseconds 500
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --disable-gpu --no-sandbox --hide-scrollbars --user-data-dir="C:\Temp\cprof" --window-size=1440,1600 --virtual-time-budget=4000 --screenshot="C:\Temp\shot.png" "<url>" 2>&1 | Out-String
```

---

## Task 1: Data value-alignment (A1 + A2 + A3)

**Files:** `wireframe/index.html` — `CASES` CASE-2026-008 (line ~4759); `fp-apps` AB accordion (lines ~2996–3022); `MY_ONBOARDING` fields (lines ~4287+); Home doc tile (line ~1395).

- [ ] **Step 1: Retarget the Shahu-Maharaj case to Aarti (A1).** Replace the CASE-2026-008 object:
```js
  {id:'CASE-2026-008', student:'Suresh Jadhav', av:'S', avStyle:'background:linear-gradient(140deg,#0f857a,#0a3f3a);color:#fff', scheme:'Shahu Maharaj Merit', appId:'MH-SM-2026-77120', fellow:'Rahul More', status:'Re-apply', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'4 Jun'},
```
with (student + avatar → Aarti):
```js
  {id:'CASE-2026-008', student:'Aarti Pawar', av:'A', avStyle:'background:linear-gradient(140deg,#e8821e,#f6b24a);color:#3a2400', scheme:'Shahu Maharaj Merit', appId:'MH-SM-2026-77120', fellow:'Rahul More', status:'Re-apply', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'4 Jun'},
```

- [ ] **Step 2: Fix the `fp-apps` AB accordion → Documents Pending (A1).** Replace the whole AB accordion block (from `<!-- AB · Approved → record disbursement -->` through the two closing `</div>`s at ~3022):
```html
            <!-- AB · Documents Pending -->
            <div class="app-acc-item" data-app="ab">
              <div class="app-acc-head" onclick="toggleAppAcc(this)">
                <div class="sr-head">
                  <div class="sr-logo" style="background:linear-gradient(140deg,#2f6fd6,#1c4ea8)">AB</div>
                  <div class="sr-name"><b>Dr. Babasaheb Ambedkar Scholarship</b><small>Eligibility matched · Docs 6/8 Accepted · 2 pending</small></div>
                </div>
                <span class="pill amber"><span class="pdot"></span>Documents Pending</span>
                <div class="sr-mini" title="Step 2 of 5">
                  <span class="ms done"></span><span class="ml done"></span>
                  <span class="ms cur"></span><span class="ml"></span>
                  <span class="ms"></span><span class="ml"></span>
                  <span class="ms"></span><span class="ml"></span>
                  <span class="ms"></span>
                </div>
                <span class="sr-step">2 of 5</span>
                <span class="app-acc-chev"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m9 6 6 6-6 6"/></svg></span>
              </div>
              <div class="app-acc-body">
                <p style="font-size:13px;color:var(--ink-soft);margin:0 0 12px">6 of 8 documents Accepted · 2 still being arranged (Caste Validity, Income proof). Submit on the portal once all mandatory documents are Accepted.</p>
                <div class="btnrow" style="display:flex;gap:10px;flex-wrap:wrap">
                  <button class="btn btn-primary btn-sm" type="button" data-go="f-docapp">Open Documentation Application</button>
                  <button class="btn btn-ghost btn-sm" type="button" onclick="event.stopPropagation();go('f-student');setTimeout(function(){var t=document.querySelectorAll('#f-student .tabs button');if(t[2])ftab(t[2],'fp-docs');},100)">View documents</button>
                </div>
              </div>
            </div>
```

- [ ] **Step 3: Align the onboarding read-only DOB + marks (A2).** In `MY_ONBOARDING.fields`: `onbDob:'2005-06-18'` → `onbDob:'2005-08-14'`; `onbMarks:'76%'` → `onbMarks:'78%'`; `onbPrevMarks:'72%'` → `onbPrevMarks:'74%'`.

- [ ] **Step 4: Fix the Home document tile (A3).** Replace `<div class="num">5/8</div><div class="cap">Documents Accepted</div>` with `<div class="num">5 of 9</div><div class="cap">Documents Accepted</div>`.

- [ ] **Step 5: Render-verify Aarti consistency.** Render and read: `?role=fellow&screen=f-applications` (Aarti now has 4 rows incl. Shahu-Maharaj Re-apply); inject-open the `f-student` Applications tab (AB reads **Documents Pending**, not Approved); `?role=student&screen=scholarships` (AB Documents Pending); `?role=student&screen=s-my-onboarding` (DOB **14 Aug 2005**, marks 78%); `?role=student&screen=home` (tile "5 of 9"). Confirm AB is Documents Pending on all and Aarti shows 4 apps in the caseload.

- [ ] **Step 6 (GATED): commit** `-m "Consistency: align Aarti's apps/DOB/marks/doc-counts across all surfaces (A1–A3)"` — ask first.

---

## Task 2: Student count label (A4) + Request-a-change discoverability (B6)

**Files:** `wireframe/index.html` — `#f-students` page-h (line ~2268); `bumpStudentCounts` (JS); Home onboarding card (line ~1422).

- [ ] **Step 1: Relabel the My Students count (A4).** Replace the page-h paragraph:
```html
          <div><div class="eyebrow">Caseload</div><h1>My Students</h1><p>10 students assigned to you. You can only see students assigned to you (RBAC).</p></div>
```
with:
```html
          <div><div class="eyebrow">Caseload</div><h1>My Students</h1><p>Showing <span id="fStuShown">10</span> of 24 students assigned to you (rest paginated) · you only see your own students (RBAC).</p></div>
```

- [ ] **Step 2: Point `bumpStudentCounts` at the new span.** Replace the page-h line inside `bumpStudentCounts`:
```js
  var p=document.querySelector('#f-students .page-h p'); if(p) p.innerHTML=p.innerHTML.replace(/^\d+/,function(n){return (parseInt(n)+1);});
```
with:
```js
  var sh=document.getElementById('fStuShown'); if(sh) sh.textContent=(parseInt(sh.textContent)||0)+1;
```

- [ ] **Step 3: Add Request-a-change hint to the Home card (B6).** In the Home onboarding-submission card, change the sub-text:
```html
<p style="color:var(--ink-soft);font-size:13px;margin:0">Submitted 12 Apr · Approved by Rahul More on 14 Apr — view the form you filled.</p>
```
to:
```html
<p style="color:var(--ink-soft);font-size:13px;margin:0">Submitted 12 Apr · Approved by Rahul More on 14 Apr — view the form you filled or request a change.</p>
```

- [ ] **Step 4: Render-verify.** `?role=fellow&screen=f-students` → header reads "Showing 10 of 24"; then inject an approval (`loginAs('fellow');openApproval('onb-ravi');approveOnboarding('onb-ravi')`) and re-render f-students → "Showing 11 of 24". `?role=student&screen=home` → card sub-text includes "or request a change".

- [ ] **Step 5 (GATED): commit** `-m "A4 'Showing 10 of 24' student count + B6 request-a-change hint on Home card"` — ask first.

---

## Task 3: Inline "Edit profile" (A5 / B2) — Fellow + Admin student-detail

**Files:** `wireframe/index.html` — CSS (before `</style>`); JS (near other student functions); `f-student` fp-profile Edit button (line ~2711) + Consent `.v` (line ~2725); `a-student` ap-profile Edit button (line ~3730) + Consent `.v` (line ~3744).

**Interfaces:** Produces `editProfile(btn)`, `saveProfile(btn)`, `cancelProfile(btn)`, `restoreProfileBtn(btn,card,saved)` — generic over any `.card` with a `.field/.k/.v` grid.

- [ ] **Step 1: Add CSS** (before `</style>`):
```css
.v-in{width:100%;padding:6px 9px;border:1px solid var(--line);border-radius:7px;font-family:inherit;font-size:14px;box-sizing:border-box}
.prof-edit-acts{display:inline-flex;gap:8px}
.prof-saved{margin-left:10px;color:var(--green);font-size:12.5px;font-weight:600}
```

- [ ] **Step 2: Add the inline-edit JS** (insert near the other student/profile functions, e.g. after `saveNote`):
```js
// Generic inline profile edit — flips a card's .field values to inputs (skips .mask + [data-noedit]).
function editProfile(btn){
  var card=btn.closest('.card'); if(!card) return;
  card.querySelectorAll('.field .v:not(.mask):not([data-noedit])').forEach(function(v){
    var txt=v.textContent.trim(); v.dataset.orig=txt;
    var inp=document.createElement('input'); inp.className='v-in'; inp.value=txt;
    v.textContent=''; v.appendChild(inp);
  });
  btn.outerHTML='<span class="prof-edit-acts"><button class="btn btn-ghost btn-sm" onclick="cancelProfile(this)">Cancel</button><button class="btn btn-primary btn-sm" onclick="saveProfile(this)">Save</button></span>';
}
function saveProfile(btn){
  var card=btn.closest('.card'); if(!card) return;
  card.querySelectorAll('.field .v').forEach(function(v){ var i=v.querySelector('input.v-in'); if(i){ v.textContent=i.value; delete v.dataset.orig; } });
  restoreProfileBtn(btn, card, true);
}
function cancelProfile(btn){
  var card=btn.closest('.card'); if(!card) return;
  card.querySelectorAll('.field .v').forEach(function(v){ if(v.dataset.orig!==undefined){ v.textContent=v.dataset.orig; delete v.dataset.orig; } });
  restoreProfileBtn(btn, card, false);
}
function restoreProfileBtn(btn, card, saved){
  var acts=btn.closest('.prof-edit-acts'); if(!acts) return;
  acts.outerHTML='<button class="btn btn-ghost btn-sm" onclick="editProfile(this)">Edit profile</button>'+(saved?'<span class="prof-saved">✓ Saved · audit-logged</span>':'');
  if(saved){ var s=card.querySelector('.prof-saved'); if(s) setTimeout(function(){ if(s&&s.parentNode) s.parentNode.removeChild(s); }, 2600); }
}
```

- [ ] **Step 3: Wire the Fellow student-detail Edit button.** In `fp-profile` (line ~2711), change `onclick="alert('Edit profile — opens the profiling form. Editable by the assigned Fellow and Program Admin; every edit is audit-logged (RBAC).')"` → `onclick="editProfile(this)"`.

- [ ] **Step 4: Mark the Fellow Consent field non-editable.** In `fp-profile` (line ~2725), change `<div class="v" style="color:var(--green)">Captured · 12 Apr 2026</div>` → `<div class="v" data-noedit style="color:var(--green)">Captured · 12 Apr 2026</div>`. (Aadhaar already carries `.mask`, so it's skipped automatically.)

- [ ] **Step 5: Ripple to the Admin student-detail.** In `ap-profile` (line ~3730), change the same `onclick="alert('Edit profile …')"` → `onclick="editProfile(this)"`; and (line ~3744) add `data-noedit` to the Consent `.v` the same way.

- [ ] **Step 6: Render-verify.** `?role=fellow&screen=f-student` → click Edit profile (inject `document.querySelector('#fp-profile .card button').click()`) → fields become inputs + Save/Cancel; change a value, inject Save → value updated + "✓ Saved · audit-logged" flash; Aadhaar + Consent stay static. Repeat for `?role=admin&screen=a-student`.

- [ ] **Step 7 (GATED): commit** `-m "A5/B2: real inline Edit profile (Fellow + Admin student-detail), replaces alert stub"` — ask first.

---

## Task 4: Fellow My Profile hub (B5)

**Files:** `wireframe/index.html` — Fellow `#f-profile` card (lines ~3129–3140). Reuses `editProfile` from Task 3.

- [ ] **Step 1: Expand the Fellow profile card + add an Edit button + mark read-only fields.** Replace the card body:
```html
        <div class="card">
          <div class="prof-head">
            <div class="pa">R</div>
            <div><h3 style="font-size:23px">Rahul More</h3><div style="color:var(--ink-soft);font-size:14px">Samavesh Fellow · Pune region</div><div style="margin-top:8px"><span class="pill teal"><span class="pdot"></span>Active</span></div></div>
          </div>
          <div class="grid g-2" style="margin-top:18px">
            <div class="field"><div class="k">Role</div><div class="v" data-noedit>Fellow</div></div>
            <div class="field"><div class="k">Region</div><div class="v">Pune</div></div>
            <div class="field"><div class="k">Students assigned</div><div class="v" data-noedit>24</div></div>
            <div class="field"><div class="k">Mobile</div><div class="v">+91 98xxx xxx11</div></div>
            <div class="field"><div class="k">Email</div><div class="v" data-noedit>rahul.more@samavesh.org</div></div>
            <div class="field"><div class="k">Joined</div><div class="v" data-noedit>Jan 2025</div></div>
            <div class="field"><div class="k">Focus areas</div><div class="v" data-noedit>Post-Matric · OBC · Minority · Pune &amp; PCMC</div></div>
          </div>
          <div style="display:flex;justify-content:flex-end;margin-top:14px"><button class="btn btn-ghost btn-sm" onclick="editProfile(this)">Edit profile</button></div>
        </div>
```
(Only **Region** + **Mobile** lack `data-noedit`, so only they become editable; `editProfile` restores an "Edit profile" button on save/cancel.)

- [ ] **Step 2: Render-verify.** `?role=fellow&screen=f-profile` → shows Email/Joined/Focus + Edit button; inject the Edit click → only Region + Mobile become inputs (Role/Students/Email/Joined/Focus stay text); inject Save → "✓ Saved · audit-logged".

- [ ] **Step 3 (GATED): commit** `-m "B5: Fellow My Profile hub — email/joined/focus + editable contact"` — ask first.

---

## Task 5: Full sweep + heartbeat + memory

**Files:** `wireframe/BUILD_STATUS.md`; after user OK, memory.

- [ ] **Step 1: Consistency grep.** Confirm: no remaining `alert('Edit profile` (both replaced); `fp-apps` AB no longer says "Application Approved"; `CASE-2026-008` student is Aarti; the caseload shows Aarti with 4 (grep `student:'Aarti Pawar'` in `CASES` = 4). Fix anything surfaced.
- [ ] **Step 2: Behavior render sweep.** Re-render + read: Aarti consistent across caseload / fp-apps / fp-sch / student Scholarships / s-my-onboarding / My Profile; My Students "Showing 10 of 24"; inline Edit profile (Fellow, Admin, Fellow-own); Home card hint. All per `samavesh-render-verify`.
- [ ] **Step 3: Heartbeat.** Add a dated section to `wireframe/BUILD_STATUS.md` summarising the consistency+polish batch (A1–A5, B2–B6) done + render-verified; note the **value-align-only** decision as a "data lives in multiple places — watch for future drift" caveat.
- [ ] **Step 4 (GATED): commit** `-m "Heartbeat: consistency + polish batch (A1–A5, B2–B6)"` — then ask whether to **push** (separate gate).
- [ ] **Step 5 (after user OK): update memory** — note the audit + this batch in a short memory (or extend `samavesh-cases-tasks`/`samavesh-feedback-batch2`): Aarti is now the aligned canonical demo student; generic `editProfile` inline-edit pattern exists; value-align-only (future-drift watch-item).

---

## Self-Review (against the spec)

- **Spec coverage:** A1 → Task 1 Steps 1–2; A2 → Task 1 Step 3; A3/B4 → Task 1 Step 4; A4 → Task 2 Steps 1–2; B6 → Task 2 Step 3; A5/B2 (+Admin ripple) → Task 3; B5 → Task 4. All mapped.
- **Placeholder scan:** every step has complete code / exact edits; demo values concrete (Aarti / Rahul / dates). No TBD.
- **Type/name consistency:** `editProfile`/`saveProfile`/`cancelProfile`/`restoreProfileBtn` defined in Task 3, reused in Task 4; `#fStuShown` created in Task 2 Step 1 and targeted in Step 2; CASE-2026-008 + AB block + MY_ONBOARDING keys consistent with the live code.
- **Scope:** one coherent batch in one file — one plan is correct.
- **Constraint:** commits GATED; push is a separate gate; value-align only.
