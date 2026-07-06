# Student Read-Only Onboarding View — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Let a student view their submitted onboarding form read-only (Home card + a peek while under review), reusing the existing form-clone component in a new "locked" mode, with a "Request a change" action that notifies the Fellow.

**Architecture:** Reuse `cloneOnbForm`/`wireOnbClone`/`prefillOnbClone` (built for the Fellow approval review) and add a `lockOnbClone` that disables the clone. A new `#s-my-onboarding` screen renders the student's seeded submission (`MY_ONBOARDING`) with a context-aware stamp; entry points on Home and the pending gate reach it; "Request a change" pushes an entry to the Fellow's `NOTIF` list. All in `wireframe/index.html`.

**Tech Stack:** Vanilla HTML/CSS/JS in one file. No build. Verification is **headless-Chrome render**, not unit tests.

## Global Constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** (user's chosen cadence); **ask before pushing** (memory `samavesh-git-workflow`). Commit only `wireframe/index.html` (+ `BUILD_STATUS.md` in the last task); never `git add -A`.
- **Reuse the ONE onboarding form** — no second copy of the 42-Q markup (memory `samavesh-approval-review-cluster`).
- **Student is view-only** (RBAC): the read-only form must be genuinely non-editable; "Request a change" is the only student affordance and it routes to the Fellow.
- **No inert UI** (memory `samavesh-no-half-baked`): no `prompt()`; every control does something visible.
- **Reuse existing vocabulary** (memory `samavesh-use-only-context-terms`).
- **Render-verify behavior** (memory `samavesh-render-verify`): kill chrome first; `--headless=new` + `--user-data-dir="C:\Temp\cprof"`; read each PNG.

### Render snippet (reuse)
```powershell
Copy-Item "C:\Users\Jitesh (DHW-L06)\Desktop\Samavesh\wireframe\index.html" "C:\Temp\swf.html" -Force
try { Get-Process chrome -ErrorAction Stop | Stop-Process -Force } catch {}
Start-Sleep -Milliseconds 500
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --disable-gpu --no-sandbox --hide-scrollbars --user-data-dir="C:\Temp\cprof" --window-size=1440,2400 --virtual-time-budget=4000 --screenshot="C:\Temp\shot.png" "file:///C:/Temp/swf.html?role=student&screen=s-my-onboarding" 2>&1 | Out-String
```

---

## Task 1: Read-only screen `#s-my-onboarding` + engine + Request-a-change

**Files:**
- Modify `wireframe/index.html`:
  - CSS before `</style>` (~line 1141-region — anchor on the `.doc-modal-note` rule added earlier, or just before `</style>`).
  - Insert `#s-my-onboarding` after the `#s-onboard` `</section>` (~line 1365).
  - Insert `lockOnbClone` after `prefillOnbClone` (ends line 4259).
  - Insert `MY_ONBOARDING` + `buildMyOnboarding` + `renderOnbChangeStart`/`startOnbChange`/`sendOnbChange` near `buildStudentOnboard` (~4260).
  - Add the `go()` hook after the `s-onboard` hook (line 4395).

**Interfaces:**
- Consumes: `cloneOnbForm(prefix)`, `wireOnbClone(clone,prefix)`, `prefillOnbClone(clone,prefix,sub)`, `STUDENT_STAGE`, `NOTIF`, `go`.
- Produces: `lockOnbClone(clone)`, `MY_ONBOARDING`, `buildMyOnboarding()`, `renderOnbChangeStart()`, `startOnbChange()`, `sendOnbChange()`.

- [ ] **Step 1: Add CSS.** Immediately before `</style>`, insert:
```css
.onb-readonly input:disabled,.onb-readonly select:disabled,.onb-readonly textarea:disabled{opacity:1;-webkit-text-fill-color:var(--ink);color:var(--ink);background:#f6f7f8;cursor:default}
```

- [ ] **Step 2: Add the read-only screen.** Immediately AFTER the `</section>` closing `#s-onboard` (line ~1365), insert:
```html
      <!-- ========== STUDENT — MY ONBOARDING (read-only view of their submitted form) ========== -->
      <section class="screen" id="s-my-onboarding">
        <span class="backlink" id="myOnbBack" data-go="home"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m15 18-6-6 6-6"/></svg>Back</span>
        <div class="page-h" style="margin-bottom:10px"><div class="eyebrow">Onboarding</div><h1>My Onboarding Form</h1><p>This is the form you submitted. It's view-only — need a correction? Use “Request a change” below and your Fellow will update it.</p></div>
        <div class="card" id="myOnbStamp" style="margin-bottom:14px"></div>
        <div id="sMyOnbHost"></div>
        <div class="card" id="myOnbChange" style="margin-top:14px"></div>
      </section>
```

- [ ] **Step 3: Add `lockOnbClone`.** Immediately AFTER `prefillOnbClone` closes (line 4259, the `}` before the `// Clone the EXACT Fellow…` comment), insert:
```js
// Make a cloned onboarding form read-only (student view of their own submission).
function lockOnbClone(clone){
  clone.querySelectorAll('input, select, textarea').forEach(function(el){ el.disabled = true; });
  var acts = clone.querySelector('.dt-actions'); if(acts) acts.style.display='none';
  clone.querySelectorAll('.doc-ctable .attach-btn').forEach(function(b){ b.style.display='none'; });
  clone.classList.add('onb-readonly');
}
```

- [ ] **Step 4: Add data + builder + Request-a-change.** Insert immediately BEFORE `function buildStudentOnboard(){` (line 4261) — i.e., after the `lockOnbClone` from Step 3 and before the existing `// Clone the EXACT Fellow…` comment. Paste:
```js
// The logged-in student's submitted onboarding answers (demo: Aarti). Same shape as the approval submissions.
var MY_ONBOARDING = {
  name:'Aarti Ramesh Pawar', submitted:'12 Apr 2026', approvedBy:'Rahul More', approvedDate:'14 Apr 2026',
  fields:{ onbEmail:'aarti.pawar@gmail.com', onbFullName:'Aarti Ramesh Pawar', onbDob:'2005-06-18', onbGender:'F',
           onbMobile:'9800000021', onbFormDate:'2026-04-12', onbState:'Maharashtra',
           onbPermAddr:'23 Shanti Nagar, Kothrud, Pune 411038', onbDistrict:'Pune',
           onbCurrAddr:'23 Shanti Nagar, Kothrud, Pune 411038', onbCurrDistrict:'Pune',
           onbCollege:'Fergusson College, Pune', onbHeard:'College',
           onbGrade:'Graduation - 2nd Year', onbStream:'Commerce & Management', onbMarks:'76%',
           onbPrevGrade:'Graduation - 1st Year', onbPrevMarks:'72%',
           onbSocial:'SC', onbIncome:'₹1 – 2 lakh', onbParentDis:'No', onbOccupation:'Daily Wages', onbDrug:'No',
           onbGoals:'Complete my B.Com and qualify for a government scholarship.', onbCareer:'Become a chartered accountant.',
           onbShare:'Mother is the sole earner; fee support needed this year.', onbConsent:'Aarti Ramesh Pawar' },
  docs:['have','have','have','have','have','dont','have','dont','na'],
  checks:{ onb_support:[0], onb_vuln:[1,2], onb_sp:[1], onb_house:[0], onb_net:[0], onb_comp:[0] }
};
// Build the student's read-only view of their submission; re-set the stamp/back-link/request box on every visit.
function buildMyOnboarding(){
  var host=document.getElementById('sMyOnbHost');
  if(host && !host.dataset.built){
    var clone=cloneOnbForm('myonb_');
    if(clone){ clone.id='myOnbForm'; host.appendChild(clone); host.dataset.built='1'; wireOnbClone(clone,'myonb_'); prefillOnbClone(clone,'myonb_',MY_ONBOARDING); lockOnbClone(clone); }
  }
  var approved=(STUDENT_STAGE==='approved');
  var stamp=document.getElementById('myOnbStamp');
  if(stamp){
    stamp.innerHTML = approved
      ? '<div style="display:flex;align-items:center;gap:10px;color:var(--green);font-weight:600"><svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M20 6 9 17l-5-5"/></svg>Approved by '+MY_ONBOARDING.approvedBy+' on '+MY_ONBOARDING.approvedDate+'</div><div style="font-size:12.5px;color:var(--muted);margin-top:4px">Submitted '+MY_ONBOARDING.submitted+'</div>'
      : '<div style="display:flex;align-items:center;gap:10px;color:#9a5b00;font-weight:600">⏳ Submitted '+MY_ONBOARDING.submitted+' · under review by your Fellow</div><div style="font-size:12.5px;color:var(--muted);margin-top:4px">Your full portal unlocks once approved — we\'ll notify you.</div>';
  }
  var back=document.getElementById('myOnbBack'); if(back) back.setAttribute('data-go', approved ? 'home' : 's-gate');
  renderOnbChangeStart();
}
// Request-a-change (student is view-only; the request routes to the Fellow).
function renderOnbChangeStart(){
  var box=document.getElementById('myOnbChange'); if(!box) return;
  box.innerHTML='<div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:10px"><div><b>Spotted something to fix?</b><div style="font-size:12.5px;color:var(--muted)">You can\'t edit directly — send the change to your Fellow.</div></div><button class="btn btn-ghost btn-sm" onclick="startOnbChange()">Request a change</button></div>';
}
function startOnbChange(){
  var box=document.getElementById('myOnbChange'); if(!box) return;
  box.innerHTML='<div><b>Request a change</b><div style="font-size:12.5px;color:var(--muted);margin:2px 0 8px">Describe what needs correcting — your Fellow will review and update your form.</div>'+
    '<textarea id="onbChangeText" placeholder="e.g. My current address has changed / a mark was entered wrong" style="width:100%;min-height:72px;padding:10px 12px;border:1px solid var(--line);border-radius:9px;font-family:inherit;font-size:13.5px;box-sizing:border-box"></textarea>'+
    '<div style="display:flex;gap:8px;justify-content:flex-end;margin-top:10px"><button class="btn btn-ghost btn-sm" onclick="renderOnbChangeStart()">Cancel</button><button class="btn btn-primary btn-sm" onclick="sendOnbChange()">Send to my Fellow</button></div></div>';
  var t=document.getElementById('onbChangeText'); if(t) t.focus();
}
function sendOnbChange(){
  var v=((document.getElementById('onbChangeText')||{}).value||'').trim();
  var first=MY_ONBOARDING.name.split(' ')[0];
  NOTIF.fellow.unshift({t:first+' requested a change to her onboarding form'+(v?(': “'+v+'”'):''), s:'today'});
  var box=document.getElementById('myOnbChange'); if(box) box.innerHTML='<div style="display:flex;align-items:center;gap:10px;color:var(--green);font-weight:600"><svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M20 6 9 17l-5-5"/></svg>Sent to your Fellow — they\'ll review and update your form.</div>';
}
```

- [ ] **Step 5: Wire the `go()` hook.** After the line `if(id==='s-onboard' && typeof buildStudentOnboard==='function') setTimeout(buildStudentOnboard,0);` (line 4395), add:
```js
  if(id==='s-my-onboarding' && typeof buildMyOnboarding==='function') setTimeout(buildMyOnboarding,0);
```

- [ ] **Step 6: Render — approved (read-only) view.** Render `?role=student&screen=s-my-onboarding` (per the snippet). Read the PNG. Expect: the full 42-Q form pre-filled with Aarti's answers, **every field disabled/greyed**, **no Save/Submit bar**, a green **"Approved by Rahul More on 14 Apr 2026"** stamp, and the "Spotted something to fix? · Request a change" card at the bottom.

- [ ] **Step 7: Render — Request-a-change flow.** Inject before `</body>` in a copy:
  ```
  <script>window.addEventListener("load",function(){setTimeout(function(){loginAs("student");go("s-my-onboarding");startOnbChange();},700);});</script>
  ```
  Render it → expect the inline textarea + Send/Cancel. Then in a second copy inject `...startOnbChange();document.getElementById("onbChangeText").value="My marks were entered wrong";sendOnbChange();` → expect the green "Sent to your Fellow…" confirmation; also dump the DOM and confirm `NOTIF.fellow[0].t` contains "requested a change".

- [ ] **Step 8: Render — under-review variant.** Inject `...loginAs("student");STUDENT_STAGE="submitted";go("s-my-onboarding");...` → expect the amber "⏳ Submitted 12 Apr 2026 · under review by your Fellow" stamp (form still locked).

- [ ] **Step 9 (GATED): commit.** `git add wireframe/index.html && git commit -m "Student read-only onboarding view: #s-my-onboarding + lockOnbClone + Request-a-change"` — ask first.

---

## Task 2: Entry points — Home card + pending-gate peek

**Files:**
- Modify `wireframe/index.html`: student Home `#home` (insert before the "My Applications" block, ~line 1409); pending gate card `#sGatePending` (~line 1345-1349).

**Interfaces:** Consumes the `#s-my-onboarding` screen + `go` (data-go delegation) from Task 1.

- [ ] **Step 1: Add the Home card.** Immediately BEFORE the line `<!-- My Applications — one row per scheme with separate status (matches Fellow/Admin pattern) -->` (line ~1409), insert:
```html
        <!-- Onboarding submission (read-only record) -->
        <div class="card" style="margin:14px 0;border-left:4px solid var(--teal-700);display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:10px">
          <div><h4 class="serif" style="font-size:17px;margin:0 0 2px">Your onboarding submission</h4><p style="color:var(--ink-soft);font-size:13px;margin:0">Submitted 12 Apr · Approved by Rahul More on 14 Apr — view the form you filled.</p></div>
          <button class="btn btn-ghost btn-sm" data-go="s-my-onboarding">View my onboarding form →</button>
        </div>
```

- [ ] **Step 2: Add the pending-gate peek.** In `#sGatePending`, immediately AFTER its `<p>…we'll notify you.</p>` line (line ~1348), before the closing `</div>`, insert:
```html
            <button class="btn btn-ghost btn-sm" data-go="s-my-onboarding" style="margin-top:16px">Review what you submitted →</button>
```

- [ ] **Step 3: Render — Home card.** Render `?role=student&screen=home` → expect the "Your onboarding submission" card with "View my onboarding form →"; clicking it (inject `document.querySelector('[data-go=\"s-my-onboarding\"]').click()`) lands on the read-only screen.

- [ ] **Step 4: Render — pending peek.** Inject `loginAs("student","new");STUDENT_STAGE="submitted";applyStudentGate();` → the pending gate card shows "Review what you submitted →"; clicking it opens the read-only screen with the "under review" stamp.

- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html && git commit -m "Student read-only onboarding: Home card + under-review peek entry points"` — ask first.

---

## Task 3: Full-sweep verify + heartbeat + memory

**Files:** Modify `wireframe/BUILD_STATUS.md`; after user OK, update memory.

- [ ] **Step 1: Consistency check.** Grep confirms: exactly one static onboarding form (`id="onbForm"` count = 1); `s-my-onboarding` reachable only via the two new entry points + deep-link; no `prompt()` added; `lockOnbClone` applied (the read-only clone has `onb-readonly`). Confirm the read-only clone's `name`/`id` prefix (`myonb_`) doesn't collide with the Fellow review clone (`rev_`) or the self-fill (`s_`).
- [ ] **Step 2: Behavior render sweep.** Re-render + read: approved read-only view, Request-a-change (form → confirmation + NOTIF), under-review variant, Home card, pending peek — all per `samavesh-render-verify`.
- [ ] **Step 3: Update heartbeat.** Add a dated section to `wireframe/BUILD_STATUS.md` summarising the student read-only onboarding view (Home card + under-review peek + Request-a-change + Approved-by stamp) as done + render-verified.
- [ ] **Step 4 (GATED): commit** `git add wireframe/index.html wireframe/BUILD_STATUS.md && git commit -m "Heartbeat: student read-only onboarding view complete"` — then ask whether to **push** (separate gate).
- [ ] **Step 5 (after user OK): update memory** — note in `samavesh-feedback-batch2` (or a short new memory) that this ID-11-adjacent "student views own onboarding read-only" is built; record the onboarding form's three render modes (editable self-fill `s_`, Fellow review `rev_`, student read-only `myonb_`).

---

## Self-Review (against the spec)

- **Spec coverage:** §"third render mode" + `lockOnbClone` → Task 1 Steps 1,3,4. §Home card → Task 2 Step 1. §read-only screen `#s-my-onboarding` + stamp + go hook → Task 1 Steps 2,4,5. §Request-a-change (+Fellow NOTIF) → Task 1 Step 4. §peek during under-review → Task 2 Step 2. §verification → Task 1 Steps 6-8 + Task 3 Step 2. All spec sections mapped.
- **Placeholder scan:** every step has complete code; demo dates/names are concrete (Aarti / Rahul More / 12 Apr). No TBD/TODO.
- **Type/name consistency:** `lockOnbClone`, `MY_ONBOARDING`, `buildMyOnboarding`, `renderOnbChangeStart`, `startOnbChange`, `sendOnbChange`, screen id `s-my-onboarding`, host `sMyOnbHost`, stamp `myOnbStamp`, back `myOnbBack`, change box `myOnbChange`, clone prefix `myonb_` — all defined in Task 1 and reused consistently; entry points in Task 2 use the same `s-my-onboarding` id.
- **Scope:** one coherent student-facing feature in one file — one plan is correct.
- **Constraint:** commits GATED; push is a separate gate.
