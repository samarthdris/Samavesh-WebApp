# Onboarding Approval Review + Action-Gate Cluster — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Make the Fellow's onboarding approval a real review gate — click a submission to see the student's *complete* 42-question form (editable), approve it into the caseload — plus give document verification a real preview and a working WhatsApp reach-out.

**Architecture:** Single-file wireframe (`wireframe/index.html`). Reuse the ONE onboarding form (`#onbForm`) as the review body by cloning it per submission (same mechanism already used for student self-onboarding), pre-filled from a JS data map. Approval completes end-to-end (student lands in My Students). A shared modal replaces every "View file" alert.

**Tech Stack:** Vanilla HTML/CSS/JS in one file. No build, no framework, no Node. Verification is by **headless Chrome render** (per memory `samavesh-render-verify`), not unit tests.

## Global Constraints

- **Single file:** all edits are in `wireframe/index.html`. Copy relevant screenshots-of-truth from memory only; do not read the ignore-list files (memory `samavesh-ignored-files`).
- **COMMIT GATE:** NEVER run `git commit`/`git push` without explicit user approval (memory `samavesh-git-workflow`). Every "commit" below is gated — STOP and ask first.
- **English-only:** no Marathi anywhere (the cloned form already conforms).
- **No inert UI:** every new control must do something visible (memory `samavesh-no-half-baked`).
- **One source of truth:** the review form MUST be a clone of `#onbForm` — do NOT author a second copy of the form markup.
- **Doc-status vocab unchanged:** Pending / Uploaded / Under Review / Accepted / Rejected / N/A.
- **Verify behavior, not structure:** headless-render every touched screen and exercise the control (memory `samavesh-render-verify`); kill lingering chrome first.

---

## Task 1: Address onboarding fields + shared clone/prefill helpers

**Files:**
- Modify: `wireframe/index.html` — `#onbForm` fields (lines ~2429–2662); `buildStudentOnboard` (lines ~4492–4506).

**Interfaces:**
- Produces: `cloneOnbForm(prefix) -> HTMLElement|null`; `wireOnbClone(clone, prefix) -> void`; `prefillOnbClone(clone, prefix, sub) -> void`. `sub` shape: `{ fields:{<baseId>:value}, docs:[statusValue,...], checks:{<baseName>:[optionIndex,...]} }`.
- Consumes: existing globals `GENDER_MASTER`, `onbDocStatus`.

- [ ] **Step 1: Add stable IDs to the scalar `#onbForm` fields.** Apply each edit (add only the `id=`; leave everything else):

  | Field (current line) | Add id |
  |---|---|
  | §1 Email input (2429) | `id="onbEmail"` |
  | §2 Full Name input (2444) | `id="onbFullName"` |
  | §2 Date of Form Submission input (2463) | `id="onbFormDate"` |
  | §3 State select (2473) | `id="onbState"` |
  | §3 Permanent District select (2476) | `id="onbDistrict"` |
  | §3 Current District select (2482) | `id="onbCurrDistrict"` |
  | §4 College select (2492) | `id="onbCollege"` |
  | §4 How-heard select (2495) | `id="onbHeard"` |
  | §5 Current Grade select (2516) | `id="onbGrade"` |
  | §5 Stream select (2518) | `id="onbStream"` |
  | §5 Current Marks input (2519) | `id="onbMarks"` |
  | §5 Previous Grade select (2521) | `id="onbPrevGrade"` |
  | §5 Previous Marks input (2522) | `id="onbPrevMarks"` |
  | §7 Social Background select (2587) | `id="onbSocial"` |
  | §7 Annual Family Income select (2594) | `id="onbIncome"` |
  | §7 Parent disability select (2616) | `id="onbParentDis"` |
  | §7 Family Occupation select (2624) | `id="onbOccupation"` |
  | §7 Drug-addiction select (2637) | `id="onbDrug"` |
  | §8 Immediate Goals textarea (2655) | `id="onbGoals"` |
  | §8 Long-term textarea (2656) | `id="onbCareer"` |
  | §8 Anything-else textarea (2657) | `id="onbShare"` |

  Example edit (State select):
  ```html
  <!-- before -->
  <select onchange="onState(this)"><option value="">Select state</option>
  <!-- after -->
  <select id="onbState" onchange="onState(this)"><option value="">Select state</option>
  ```
  (`onbDob`, `onbGender`, `onbMobile`, `onbMobileVerify`, `onbPermAddr`, `onbCurrAddr`, `onbConsent` already have IDs — leave them.)

- [ ] **Step 2: Add the three shared helpers.** Insert immediately ABOVE `function buildStudentOnboard(){` (line ~4492):

  ```js
  // Clone the ONE onboarding form (#onbForm) with an id/name prefix so multiple copies coexist.
  function cloneOnbForm(prefix){
    var src=document.getElementById('onbForm'); if(!src) return null;
    var clone=src.cloneNode(true); clone.id=prefix+'onbForm';
    clone.querySelectorAll('[id]').forEach(function(el){ el.id=prefix+el.id; });
    clone.querySelectorAll('[name]').forEach(function(el){ el.name=prefix+el.name; });   // isolate radio/checkbox groups from other copies
    clone.querySelectorAll('.dt-anchors a[href^="#"]').forEach(function(a){ a.setAttribute('href','#'+prefix+a.getAttribute('href').slice(1)); });
    return clone;
  }
  // Wire the interactive bits of a cloned onboarding form (gender master, DOB age, mobile chip, doc-status, anchor scroll).
  function wireOnbClone(clone, prefix){
    var g=clone.querySelector('#'+prefix+'onbGender'); if(g) g.innerHTML='<option value="">Select</option>'+GENDER_MASTER.map(function(x){return '<option value="'+x.code+'">'+x.en+'</option>';}).join('');
    var dob=clone.querySelector('#'+prefix+'onbDob'), age=clone.querySelector('#'+prefix+'onbAge');
    if(dob&&age) dob.addEventListener('change',function(){ var v=dob.value; if(!v){age.textContent='Age: —';age.classList.add('empty');return;} var y=(new Date().getTime()-new Date(v).getTime())/(365.25*864e5); if(y<0||y>120){age.textContent='Age: invalid date';age.classList.add('empty');return;} age.textContent='Age: '+y.toFixed(1)+' years'; age.classList.remove('empty'); });
    var mob=clone.querySelector('#'+prefix+'onbMobile'), chip=clone.querySelector('#'+prefix+'onbMobileVerify');
    if(mob&&chip){ var chk=function(){ var v=(mob.value||'').replace(/\D/g,''); chip.classList.remove('ok','bad'); if(!v)return; if(/^[6-9]\d{9}$/.test(v)){chip.innerHTML='<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>Valid';chip.classList.add('ok');}else{chip.innerHTML='<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M6 6l12 12M18 6 6 18"/></svg>Invalid';chip.classList.add('bad');} }; mob.addEventListener('blur',chk); mob.addEventListener('input',function(){ if(chip.classList.contains('ok')||chip.classList.contains('bad'))chk(); }); }
    clone.querySelectorAll('.doc-ctable td.doc-stat select').forEach(function(s){ s.addEventListener('change',onbDocStatus); });
    clone.querySelectorAll('.dt-anchors a[href^="#"]').forEach(function(a){ a.addEventListener('click',function(e){ e.preventDefault(); var sec=document.getElementById(a.getAttribute('href').slice(1)); if(sec) sec.scrollIntoView({behavior:'smooth',block:'start'}); }); });
  }
  // Pre-fill a cloned onboarding form from a submission's answer map.
  function prefillOnbClone(clone, prefix, sub){
    sub=sub||{};
    Object.keys(sub.fields||{}).forEach(function(id){ var el=clone.querySelector('#'+prefix+id); if(el) el.value=sub.fields[id]; });
    var stats=clone.querySelectorAll('.doc-ctable td.doc-stat select');
    (sub.docs||[]).forEach(function(v,i){ if(stats[i]){ stats[i].value=v; stats[i].dispatchEvent(new Event('change')); } });
    Object.keys(sub.checks||{}).forEach(function(nm){ var opts=clone.querySelectorAll('[name="'+prefix+nm+'"]'); (sub.checks[nm]||[]).forEach(function(i){ if(opts[i]) opts[i].checked=true; }); });
    var dob=clone.querySelector('#'+prefix+'onbDob'); if(dob) dob.dispatchEvent(new Event('change'));
    var mob=clone.querySelector('#'+prefix+'onbMobile'); if(mob) mob.dispatchEvent(new Event('blur'));
  }
  ```

- [ ] **Step 3: Refactor `buildStudentOnboard` to use the helpers** (keeps student self-onboard identical, DRY). Replace the whole function body (lines ~4492–4506) with:

  ```js
  function buildStudentOnboard(){
    var host=document.getElementById('sOnbHost'); if(!host || host.dataset.built) return;
    var clone=cloneOnbForm('s_'); if(!clone) return;
    clone.id='sOnbForm';                       // preserve the original container id for backward-compat
    host.appendChild(clone); host.dataset.built='1';
    wireOnbClone(clone,'s_');
  }
  ```

- [ ] **Step 4: Regression render — student self-onboard still works.** Per memory `samavesh-render-verify`:
  ```powershell
  Copy-Item "wireframe/index.html" C:\Temp\swf.html -Force
  Get-Process chrome -ErrorAction SilentlyContinue | Stop-Process -Force; Start-Sleep -Milliseconds 600
  & "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless --disable-gpu --no-sandbox --hide-scrollbars --window-size=1440,2200 --virtual-time-budget=3000 --screenshot="C:\Temp\t1.png" "file:///C:/Temp/swf.html?role=student&screen=s-onboard"
  ```
  Read `C:\Temp\t1.png`. Expected: the full 8-section onboarding form renders with gender dropdown populated, DOB/age and mobile-verify present (i.e. the refactor didn't break the student clone).

- [ ] **Step 5 (GATED): Ask the user to commit.** If yes:
  ```bash
  git add wireframe/index.html
  git commit -m "Onboarding form: stable field IDs + shared clone/prefill helpers"
  ```

---

## Task 2: Approval list view + full-form review screen (A1 + C1)

**Files:**
- Modify: `wireframe/index.html` — `#f-approvals` markup (2351–2394); add `#f-approval-detail` screen after it; add data + functions near `approveOnboarding` (4508); `go()` hook (4545); `CRUMBS` map (4554); DOMContentLoaded (4669-area).

**Interfaces:**
- Consumes: `cloneOnbForm`, `wireOnbClone`, `prefillOnbClone` (Task 1); `go`.
- Produces: `PENDING_ONBOARDINGS` (object); `renderApprovalList()`; `openApproval(id)`. `approveOnboarding` signature CHANGES to `approveOnboarding(id)` (implemented in Task 3).

- [ ] **Step 1: Replace the hardcoded approval cards with a JS-rendered list.** Replace lines 2353–2392 (the `<div id="approvalList">…</div>` block containing the two `.approval-card`s) with just:
  ```html
  <div id="approvalList"></div>
  ```
  (Keep the `#approvalEmpty` div at 2393 and the `<section>` wrapper.)

- [ ] **Step 2: Add the review detail screen.** Immediately AFTER the `</section>` that closes `#f-approvals` (line 2394), insert:
  ```html
  <!-- ===== ONBOARDING APPROVAL — full-submission review (form view) ===== -->
  <section class="screen" id="f-approval-detail">
    <span class="backlink" data-go="f-approvals"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m15 18-6-6 6-6"/></svg>Back to Onboarding Approvals</span>
    <div class="card" style="margin-bottom:14px">
      <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:10px">
        <div class="who"><span class="sa" id="faAvatar">R</span><div><b id="faName">Student</b><div style="font-size:12px;color:var(--muted)" id="faMeta"></div></div></div>
        <span class="pill amber"><span class="pdot"></span>Pending Approval</span>
      </div>
      <p style="font-size:12.5px;color:var(--ink-soft);margin:10px 0 0">Review the student's <b>full onboarding submission</b> below. Edit any field to correct a discrepancy (audit-logged), then approve — approval unlocks the student's portal and adds them to your caseload.</p>
    </div>
    <div id="fApprovalHost"></div>
    <div class="dt-actions" id="faActionBar" style="position:sticky;bottom:0;z-index:5">
      <div class="dt-act-meta">Reviewing self-registered onboarding submission</div>
      <div class="dt-act-btns">
        <a class="btn btn-ghost btn-sm" id="faWhatsapp" href="#" target="_blank" rel="noopener"><svg viewBox="0 0 24 24" width="14" height="14" fill="currentColor"><path d="M12 2a10 10 0 0 0-8.5 15.3L2 22l4.8-1.5A10 10 0 1 0 12 2Z"/></svg>Message student</a>
        <button class="btn btn-primary" onclick="approveOnboarding(document.getElementById('faActionBar').dataset.aid)">Approve onboarding</button>
      </div>
    </div>
  </section>
  ```

- [ ] **Step 3: Add the data + list renderer + opener.** Replace the OLD `approveOnboarding(btn)` function (lines 4508–4519) with the data map, `renderApprovalList`, and `openApproval` (the new `approveOnboarding(id)` is added in Task 3):
  ```js
  // Self-registered students awaiting Fellow onboarding approval. Each holds the FULL submitted answer set.
  var PENDING_ONBOARDINGS = {
    'onb-ravi': {
      name:'Ravi Deshmukh', email:'ravi.deshmukh@gmail.com', mobile:'9812345678',
      submitted:'3 Jul 2026', dateIso:'2026-07-03',
      avatarChar:'R', avatarStyle:'background:linear-gradient(140deg,#e8821e,#f6b24a);color:#3a2400',
      fields:{ onbEmail:'ravi.deshmukh@gmail.com', onbFullName:'Ravi Deshmukh', onbDob:'2006-03-12', onbGender:'M',
               onbMobile:'9812345678', onbFormDate:'2026-07-03', onbState:'Maharashtra',
               onbPermAddr:'12 Shivaji Nagar, Near Zilla Parishad, Pune 411005', onbDistrict:'Pune',
               onbCurrAddr:'12 Shivaji Nagar, Near Zilla Parishad, Pune 411005', onbCurrDistrict:'Pune',
               onbCollege:'Fergusson College, Pune', onbHeard:'College',
               onbGrade:'Graduation - 1st Year', onbStream:'Arts / Humanities', onbMarks:'74%',
               onbPrevGrade:'12th', onbPrevMarks:'78%',
               onbSocial:'OBC', onbIncome:'₹1 – 2 lakh', onbParentDis:'No', onbOccupation:'Daily Wages', onbDrug:'No',
               onbGoals:'Complete my B.A and secure a government scholarship.', onbCareer:'Become a school teacher.',
               onbShare:'Father is the sole earner; needs fee support this semester.', onbConsent:'Ravi Deshmukh' },
      docs:['have','dont','have','have','dont','na','have','have','na'],
      checks:{ onb_support:[0], onb_vuln:[1,2], onb_sp:[1], onb_house:[0], onb_net:[1], onb_comp:[1] }
    },
    'onb-sneha': {
      name:'Sneha Kale', email:'sneha.kale@gmail.com', mobile:'9898989898',
      submitted:'2 Jul 2026', dateIso:'2026-07-02',
      avatarChar:'S', avatarStyle:'background:linear-gradient(140deg,#2f6fd6,#1c4ea8);color:#fff',
      fields:{ onbEmail:'sneha.kale@gmail.com', onbFullName:'Sneha Kale', onbDob:'2005-11-27', onbGender:'F',
               onbMobile:'9898989898', onbFormDate:'2026-07-02', onbState:'Maharashtra',
               onbPermAddr:'44 Gangapur Road, Nashik 422013', onbDistrict:'Nashik',
               onbCurrAddr:'Hostel Block C, Nashik 422005', onbCurrDistrict:'Nashik',
               onbCollege:'Other', onbHeard:'Mudita Alliance',
               onbGrade:'Graduation - 2nd Year', onbStream:'Science', onbMarks:'CGPA 8.1',
               onbPrevGrade:'Graduation - 1st Year', onbPrevMarks:'CGPA 7.9',
               onbSocial:'SC', onbIncome:'₹50,000 – ₹1 lakh', onbParentDis:'No', onbOccupation:'Farming', onbDrug:'No',
               onbGoals:'Clear my B.Sc with distinction.', onbCareer:'Pursue a Masters in Microbiology.',
               onbShare:'First-generation learner in the family.', onbConsent:'Sneha Kale' },
      docs:['have','have','have','have','dont','dont','dont','dont','na'],
      checks:{ onb_support:[0,1], onb_vuln:[2], onb_sp:[1], onb_house:[2], onb_net:[0], onb_comp:[0] }
    }
  };
  function renderApprovalList(){
    var host=document.getElementById('approvalList'); if(!host) return;
    var ids=Object.keys(PENDING_ONBOARDINGS);
    host.innerHTML = ids.map(function(id){ var s=PENDING_ONBOARDINGS[id];
      return '<div class="stu-row" style="cursor:pointer" onclick="openApproval(\''+id+'\')">'+
        '<div class="sa" style="'+s.avatarStyle+'">'+s.avatarChar+'</div>'+
        '<div class="si"><b>'+s.name+'</b><small>Submitted '+s.submitted+' · self-registered · '+s.email+'</small></div>'+
        '<div class="smeta"><span class="pill amber"><span class="pdot"></span>Pending Approval</span></div>'+
        '<svg class="chev2" viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2"><path d="m9 18 6-6-6-6"/></svg>'+
      '</div>'; }).join('');
    var empty=document.getElementById('approvalEmpty'); if(empty) empty.style.display = ids.length ? 'none' : '';
    var badge=document.getElementById('fApprovalBadge'); if(badge){ badge.textContent=ids.length; badge.style.display = ids.length ? '' : 'none'; }
    var chip=document.getElementById('fPendingChip'); if(chip) chip.textContent='Pending Approval ('+ids.length+')';
  }
  function openApproval(id){
    var s=PENDING_ONBOARDINGS[id]; if(!s) return;
    var host=document.getElementById('fApprovalHost'); if(!host) return;
    host.innerHTML='';
    var clone=cloneOnbForm('rev_'); if(!clone) return;
    var acts=clone.querySelector('.dt-actions'); if(acts) acts.style.display='none';   // hide the form's own submit bar; we use faActionBar
    host.appendChild(clone);
    wireOnbClone(clone,'rev_'); prefillOnbClone(clone,'rev_',s);
    document.getElementById('faName').textContent=s.name;
    document.getElementById('faMeta').textContent='Submitted '+s.submitted+' · self-registered · '+s.email;
    var av=document.getElementById('faAvatar'); av.textContent=s.avatarChar; av.style.cssText=s.avatarStyle;
    var wa=document.getElementById('faWhatsapp'); if(wa) wa.href='https://wa.me/91'+(s.mobile||'').replace(/\D/g,'')+'?text='+encodeURIComponent('Hi '+s.name+', this is your Samavesh Fellow about your onboarding form — ');
    document.getElementById('faActionBar').dataset.aid=id;
    go('f-approval-detail');
  }
  ```

- [ ] **Step 4: Wire startup + breadcrumb.** (a) In `CRUMBS` (line 4554) add after the `'f-approvals':...` entry: `'f-approval-detail':'Onboarding Approvals › Review',`. (b) Find the `DOMContentLoaded` block that calls `onbInit` (line ~5669) and append `if(document.getElementById('approvalList')) renderApprovalList();` inside a DOMContentLoaded handler (reuse the existing one at 5669 by adding the call after the onbInit line).

- [ ] **Step 5: Render — list → full pre-filled form.**
  ```powershell
  Copy-Item "wireframe/index.html" C:\Temp\swf.html -Force
  Get-Process chrome -ErrorAction SilentlyContinue | Stop-Process -Force; Start-Sleep -Milliseconds 600
  & "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless --disable-gpu --no-sandbox --hide-scrollbars --window-size=1440,2400 --virtual-time-budget=3000 --screenshot="C:\Temp\t2a.png" "file:///C:/Temp/swf.html?role=fellow&screen=f-approvals"
  ```
  Read `t2a.png` — expect two clickable submission rows (Ravi, Sneha), no inline fields.
  Then verify the detail opens pre-filled by injecting an auto-open before `</body>` in the copy:
  ```powershell
  (Get-Content C:\Temp\swf.html -Raw) -replace '</body>', '<script>window.addEventListener("load",function(){setTimeout(function(){loginAs("fellow");openApproval("onb-ravi");},700);});</script></body>' | Set-Content C:\Temp\swf2.html
  Get-Process chrome -ErrorAction SilentlyContinue | Stop-Process -Force; Start-Sleep -Milliseconds 600
  & "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless --disable-gpu --no-sandbox --hide-scrollbars --window-size=1440,3200 --virtual-time-budget=4000 --screenshot="C:\Temp\t2b.png" "file:///C:/Temp/swf2.html"
  ```
  Read `t2b.png` — expect the FULL onboarding form (all 8 sections) pre-filled with Ravi's answers (Full Name "Ravi Deshmukh", Gender selected, DOB with age chip, category OBC, marks, docs, socio-economic), the anchor-tab bar, and the sticky **Message student / Approve onboarding** bar. Confirm the "Message student" link's `href` starts with `https://wa.me/91981...` (grep the rendered DOM or inspect the source you injected).

- [ ] **Step 6 (GATED): Ask the user to commit.** If yes: `git add wireframe/index.html && git commit -m "Onboarding approval: list view + full-form review (A1) + WhatsApp reach-out (C1)"`

---

## Task 3: Approval completes the flow (B1)

**Files:**
- Modify: `wireframe/index.html` — add `approveOnboarding(id)`, `bumpStudentCounts`, `incChipCount`, `viewNewStudent` (near the Task 2 functions); add `.just-approved` CSS (in the `<style>` block).

**Interfaces:**
- Consumes: `PENDING_ONBOARDINGS`, `renderApprovalList` (Task 2); `go`.
- Produces: `approveOnboarding(id)` (called by the Approve button added in Task 2).

- [ ] **Step 1: Add the approval-completion functions.** Insert after `openApproval` (Task 2):
  ```js
  function incChipCount(el){ if(el) el.innerHTML=el.innerHTML.replace(/\((\d+)\)/,function(m,n){return '('+(parseInt(n)+1)+')';}); }
  function bumpStudentCounts(){
    document.querySelectorAll('#f-students .chip').forEach(function(c){ var t=c.textContent; if(/^All /.test(t)||/^Onboarded/.test(t)) incChipCount(c); });
    var cnt=document.querySelector('#f-students .listbar .cnt b'); if(cnt) cnt.textContent=(parseInt(cnt.textContent)||0)+1;
    var p=document.querySelector('#f-students .page-h p'); if(p) p.innerHTML=p.innerHTML.replace(/^\d+/,function(n){return (parseInt(n)+1);});
  }
  function viewNewStudent(){ go('f-students'); var r=document.querySelector('#stuList .just-approved'); if(r){ r.scrollIntoView({behavior:'smooth',block:'center'}); } }
  function approveOnboarding(id){
    var s=PENDING_ONBOARDINGS[id]; if(!s) return;
    var list=document.getElementById('stuList');
    if(list){
      var row=document.createElement('div');
      row.className='stu-row just-approved'; row.setAttribute('data-status','onboarded'); row.setAttribute('data-date', s.dateIso||'2026-07-03');
      row.setAttribute('onclick',"go('f-student')");
      row.innerHTML='<div class="sa" style="'+s.avatarStyle+'">'+s.avatarChar+'</div>'+
        '<div class="si"><b>'+s.name+'</b><small>New · '+(s.fields.onbGrade||'')+' · '+(s.fields.onbDistrict||'')+' · Onboarded '+s.submitted+'</small></div>'+
        '<div class="smeta"><span class="pill amber"><span class="pdot"></span>Onboarded</span><span class="sc">Eligibility pending · docs pending</span></div>'+
        '<svg class="chev2" viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2"><path d="m9 18 6-6-6-6"/></svg>';
      list.insertBefore(row, list.firstChild);
    }
    bumpStudentCounts();
    delete PENDING_ONBOARDINGS[id];
    renderApprovalList();
    var bar=document.getElementById('faActionBar');
    bar.innerHTML='<div style="display:flex;align-items:center;justify-content:space-between;gap:12px;width:100%;flex-wrap:wrap">'+
      '<span style="display:flex;align-items:center;gap:8px;color:var(--green);font-weight:600"><svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="2.5"><path d="M20 6 9 17l-5-5"/></svg>Approved — '+s.name+' added to your caseload as Onboarded.</span>'+
      '<button class="btn btn-primary btn-sm" onclick="viewNewStudent()">View in My Students</button></div>';
  }
  ```

- [ ] **Step 2: Add the highlight CSS.** In the `<style>` block, add:
  ```css
  .stu-row.just-approved{ background:var(--green-50); box-shadow:inset 3px 0 0 var(--green); animation:flashRow 2.4s ease-out 1; }
  @keyframes flashRow{ 0%{background:#d6f5e3;} 100%{background:var(--green-50);} }
  ```
  (If `--green-50` is not defined, use `#eafaf0`.)

- [ ] **Step 3: Render — approve lands the student in My Students.** Inject an approve sequence:
  ```powershell
  (Get-Content C:\Temp\swf.html -Raw) -replace '</body>', '<script>window.addEventListener("load",function(){setTimeout(function(){loginAs("fellow");openApproval("onb-ravi");setTimeout(function(){approveOnboarding("onb-ravi");},400);},700);});</script></body>' | Set-Content C:\Temp\swf3.html
  Get-Process chrome -ErrorAction SilentlyContinue | Stop-Process -Force; Start-Sleep -Milliseconds 600
  & "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless --disable-gpu --no-sandbox --hide-scrollbars --window-size=1440,2600 --virtual-time-budget=4000 --screenshot="C:\Temp\t3.png" "file:///C:/Temp/swf3.html"
  ```
  Read `t3.png` — expect the success state ("Approved — Ravi Deshmukh added to your caseload…" + View in My Students). Then screenshot `?role=fellow&screen=f-students` from the same injected file (add a second nav) and confirm a highlighted "Ravi Deshmukh · Onboarded" row is at the top and the "All (11)" / "Onboarded (3)" chips + "11 Students" count bumped.

- [ ] **Step 4 (GATED): Ask the user to commit.** If yes: `git add wireframe/index.html && git commit -m "Onboarding approval completes flow: student enters My Students caseload (B1)"`

---

## Task 4: Real document preview modal (A2)

**Files:**
- Modify: `wireframe/index.html` — add `#docPreviewModal` markup (before `</body>`); add modal CSS (`<style>`); add `openDocPreview`/`closeDocPreview` + Esc handler; replace 5 "View file" alerts (3744, 3749, 3754, 3759, ~3998); add a "View" button to the Fellow review strip (2845).

**Interfaces:**
- Produces: `openDocPreview(title, meta)`, `closeDocPreview()`.

- [ ] **Step 1: Add the modal markup.** Immediately before `</body>`, insert:
  ```html
  <div class="doc-modal" id="docPreviewModal" onclick="if(event.target===this)closeDocPreview()">
    <div class="doc-modal-card">
      <div class="doc-modal-head"><div><b id="dpTitle">Document</b><div id="dpMeta" style="font-size:12px;color:var(--muted)"></div></div><button class="doc-modal-x" onclick="closeDocPreview()" aria-label="Close">&#10005;</button></div>
      <div class="doc-modal-page">
        <div class="dmp-title">CERTIFICATE</div>
        <div class="dmp-line" style="width:70%"></div><div class="dmp-line" style="width:88%"></div><div class="dmp-line" style="width:60%"></div>
        <div class="dmp-line" style="width:82%"></div><div class="dmp-line" style="width:48%"></div>
        <div class="dmp-seal">SEAL</div>
        <div class="dmp-sign">Authorised Signatory</div>
      </div>
      <div class="doc-modal-note">Sample preview (wireframe) — production renders the actual uploaded PDF (1&ndash;2 MB) from secure storage.</div>
    </div>
  </div>
  ```

- [ ] **Step 2: Add modal CSS** (in the `<style>` block):
  ```css
  .doc-modal{position:fixed;inset:0;background:rgba(15,23,32,.55);display:none;align-items:center;justify-content:center;z-index:2000;padding:20px}
  .doc-modal.on{display:flex}
  .doc-modal-card{background:#fff;border-radius:14px;max-width:620px;width:100%;max-height:90vh;overflow:auto;box-shadow:0 24px 60px rgba(0,0,0,.3)}
  .doc-modal-head{display:flex;justify-content:space-between;align-items:flex-start;gap:12px;padding:16px 18px;border-bottom:1px solid var(--line)}
  .doc-modal-x{border:none;background:none;font-size:18px;cursor:pointer;color:var(--muted);line-height:1}
  .doc-modal-page{margin:18px;padding:28px;border:1px solid var(--line);border-radius:8px;background:#fcfcfa;position:relative;min-height:340px}
  .dmp-title{font-weight:700;letter-spacing:.14em;text-align:center;color:#334;margin-bottom:22px}
  .dmp-line{height:11px;border-radius:4px;background:#e7e9ee;margin:12px 0}
  .dmp-seal{position:absolute;right:36px;bottom:70px;width:74px;height:74px;border:2px dashed #b9bfca;border-radius:50%;display:flex;align-items:center;justify-content:center;color:#9aa1ad;font-size:11px;letter-spacing:.1em}
  .dmp-sign{position:absolute;left:36px;bottom:44px;border-top:1px solid #99a;padding-top:4px;font-size:11px;color:#889}
  .doc-modal-note{padding:0 18px 18px;font-size:12px;color:var(--muted)}
  ```

- [ ] **Step 3: Add the JS.** Near the other doc functions (e.g. after `downloadDoc`, line 4825):
  ```js
  function openDocPreview(title, meta){ document.getElementById('dpTitle').textContent=title||'Document'; document.getElementById('dpMeta').textContent=meta||''; document.getElementById('docPreviewModal').classList.add('on'); }
  function closeDocPreview(){ document.getElementById('docPreviewModal').classList.remove('on'); }
  document.addEventListener('keydown',function(e){ if(e.key==='Escape') closeDocPreview(); });
  ```

- [ ] **Step 4: Replace the 5 "View file" alerts.** For each of lines 3744, 3749, 3754, 3759 (a-verify) and ~3998 (a-student), replace the button's `onclick="alert('View file …')"` with a call carrying that row's real title/meta. The four a-verify rows in order:
  ```html
  onclick="openDocPreview('Domicile Certificate — Aarti Pawar','Type: Domicile · uploaded by Fellow Rahul · 2 Jun · Re-upload Version 1')"
  onclick="openDocPreview('Income Certificate — Kavya Deshmukh','Type: Income · uploaded by Fellow Rahul · 30 May')"
  onclick="openDocPreview('Caste Validity Certificate — Imran Shaikh','Type: Caste · uploaded by Fellow Sandip K · 1 Jun')"
  onclick="openDocPreview('Bonafide Certificate — Ganesh Pawar','Type: Academic · uploaded by Fellow Dhanashree O · 3 Jun')"
  ```
  For the a-student row (~3998), read its `.vb b`/`small` text and pass the matching title/meta (e.g. `openDocPreview('<that doc> — <that student>','<that meta>')`).

- [ ] **Step 5: Give the Fellow a View before Accept.** In the "Recently uploaded by student" strip (line 2845), insert a View button as the FIRST child of `.review-acts` (before the Download link):
  ```html
  <button class="btn btn-ghost btn-sm" onclick="openDocPreview('Ration Card','Uploaded by Aarti just now — pending your review')">View</button>
  ```

- [ ] **Step 6: Render — preview modal works.**
  ```powershell
  Copy-Item "wireframe/index.html" C:\Temp\swf.html -Force
  (Get-Content C:\Temp\swf.html -Raw) -replace '</body>', '<script>window.addEventListener("load",function(){setTimeout(function(){loginAs("admin");go("a-verify");openDocPreview("Domicile Certificate — Aarti Pawar","Type: Domicile · uploaded by Fellow Rahul · 2 Jun");},700);});</script></body>' | Set-Content C:\Temp\swf4.html
  Get-Process chrome -ErrorAction SilentlyContinue | Stop-Process -Force; Start-Sleep -Milliseconds 600
  & "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless --disable-gpu --no-sandbox --hide-scrollbars --window-size=1440,2000 --virtual-time-budget=3000 --screenshot="C:\Temp\t4.png" "file:///C:/Temp/swf4.html"
  ```
  Read `t4.png` — expect the preview modal open over the verification queue, showing the faux certificate page + title/meta + the "Sample preview" note. Also verify Accept still works (row → Accepted pill) by a separate injected `vAct` call if desired.

- [ ] **Step 7 (GATED): Ask the user to commit.** If yes: `git add wireframe/index.html && git commit -m "Document verification: real preview modal replaces View-file alerts + Fellow View-before-accept (A2)"`

---

## Task 5: Full-sweep verification + heartbeat + memory

**Files:**
- Modify: `wireframe/BUILD_STATUS.md` (heartbeat). After user OK: add memory `samavesh-review-gate` + MEMORY.md pointer.

- [ ] **Step 1: Ripple/consistency sweep.** Confirm: (a) the doc-preview modal is used by ALL view-file entry points (5 alerts replaced + Fellow strip View added); (b) approving keeps `#fApprovalBadge`, `#fPendingChip`, and the f-students counts in sync; (c) no fourth copy of the onboarding form exists (only `#onbForm` + runtime clones); (d) English-only. Grep to confirm zero remaining `alert('View file`:
  ```powershell
  Select-String -Path wireframe/index.html -Pattern "View file — opens"
  ```
  Expected: no matches.

- [ ] **Step 2: Behavior render sweep.** Re-render and Read each: `?role=fellow&screen=f-approvals` (list), injected `openApproval`→`approveOnboarding` (full form → success → new My Students row), `?role=admin&screen=a-verify` with injected `openDocPreview` (modal). All per memory `samavesh-render-verify` (kill chrome between shots; one chrome call at a time).

- [ ] **Step 3: Update the heartbeat.** Add a dated section to `wireframe/BUILD_STATUS.md` summarising A1/B1/A2/C1 as done + render-verified.

- [ ] **Step 4 (GATED): Ask the user to commit** the heartbeat (and code if not already): `git add -A && git commit -m "Heartbeat: onboarding-approval review + doc preview cluster (A1/B1/A2/C1)"` — then ask whether to **push** (separate gate).

- [ ] **Step 5 (after user OK): Save the review-gate principle to memory** as a `feedback`-type memory `samavesh-review-gate` ("any approve/verify/reject gate must expose the complete artifact, reusing the capture component") + add the MEMORY.md pointer line.

---

## Self-Review (against the spec)

- **Spec coverage:** A1 → Tasks 1+2; B1 → Task 3; A2 → Task 4; C1 → Task 2 Step 3 (WhatsApp href). Root-cause memory rule → Task 5 Step 5. All spec sections mapped.
- **Placeholder scan:** no TBD/TODO; every code step has complete code; the a-student view-file (Task 4 Step 4) requires reading one row's text at implementation time — flagged explicitly, not a silent gap.
- **Type/name consistency:** `cloneOnbForm`/`wireOnbClone`/`prefillOnbClone` defined in Task 1 and consumed with the same signatures in Task 2; `PENDING_ONBOARDINGS`, `renderApprovalList`, `openApproval`, `approveOnboarding(id)` consistent across Tasks 2–3; `openDocPreview(title,meta)` consistent in Task 4.
- **Scope:** single coherent feature cluster in one file — one plan is correct.
- **Constraint:** commit steps are all GATED per `samavesh-git-workflow`.
