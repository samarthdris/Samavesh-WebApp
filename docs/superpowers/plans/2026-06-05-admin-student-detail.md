# Admin Student-Detail View Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a Program Admin student record view (`a-student`) so clicking a row in Admin → All Students opens that student's record (oversight + Accept/Reject + Edit profile + Reassign Fellow), instead of dumping into the global verification queue.

**Architecture:** A new `<section class="screen" id="a-student">` inside the existing `#adminApp` shell, reusing the `f-student` record scaffolding (`.rec-layout`, `.journey`, `.tabs`, `.rec-side`, `.activity`) and the branded `.fd` Desk theme (no new CSS). A new `goStudent(row)` JS helper drives a dynamic identity header + journey from the clicked row's `data-*` attributes. The Documents tab reuses the existing `vAct()` Accept/Reject function unchanged by adopting the `.vrow` row structure.

**Tech Stack:** Single-file static HTML/CSS/vanilla-JS prototype (`wireframe/index.html`). No build, no test framework — verification is structural (Grep) + manual in-browser. Frequent commits to `dev`.

**Reference spec:** `docs/superpowers/specs/2026-06-05-admin-student-detail-design.md`

**Verification convention (no test runner exists):** each task's "failing test" is a Grep that confirms the target state is *absent* beforehand; the "passing test" is a Grep confirming it's *present* after. A final manual browser checklist (Task 4) is the real acceptance gate. Commit after each task.

---

### Task 1: Add the `a-student` section markup

**Files:**
- Modify: `wireframe/index.html` — insert a new `<section>` between the end of `#a-students` (closes at line ~2257) and the start of `#a-fellows` (`<!-- ===== FELLOWS & USERS ===== -->`, line ~2259).

- [ ] **Step 1: Confirm the screen does not yet exist (failing check)**

Run (Grep tool): pattern `id="a-student"` in `wireframe/index.html`, output_mode `count`.
Expected: **0 matches** (only `id="a-students"` exists, which won't match `"a-student"` exactly).

- [ ] **Step 2: Insert the new section**

Locate the closing of the All Students section:

```html
        <div class="empty" id="aStuEmpty" style="display:none;margin-top:12px"><div>No students match this filter.</div></div>
      </section>

      <!-- ===== FELLOWS & USERS ===== -->
```

Insert the following block **between** `</section>` and the `<!-- ===== FELLOWS & USERS ===== -->` comment:

```html

      <!-- ===== ADMIN STUDENT DETAIL (oversight + admin-only actions) ===== -->
      <section class="screen" id="a-student">
        <span class="backlink" data-go="a-students"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m15 18-6-6 6-6"/></svg>Back to All Students</span>

        <div class="rec-layout"><div class="rec-main">
        <div class="card" style="margin-bottom:20px">
          <div class="prof-head">
            <div class="pa" id="asAvatar">A</div>
            <div style="flex:1">
              <h3 style="font-size:23px" id="asName">Aarti Ramesh Pawar</h3>
              <div style="color:var(--ink-soft);font-size:14px" id="asSub">SMV-2026-01182 · B.Com Year 2 · Fergusson College, Pune</div>
              <div style="margin-top:8px;display:flex;gap:8px;flex-wrap:wrap" id="asStagePill"><span class="pill blue"><span class="pdot"></span>Submitted</span></div>
            </div>
          </div>
        </div>

        <!-- per-student journey (rebuilt by goStudent) -->
        <div class="journey" style="margin-bottom:20px">
          <div class="jh"><h3>Scholarship Journey</h3><span>Program oversight view</span></div>
          <div class="path" id="asPath"></div>
        </div>

        <div class="tabs">
          <button class="on" onclick="ftab(this,'ap-profile')">Profile</button>
          <button onclick="ftab(this,'ap-sch')">Scholarships</button>
          <button onclick="ftab(this,'ap-docs')">Documents</button>
          <button onclick="ftab(this,'ap-apps')">Applications</button>
        </div>

        <!-- Profile tab (admin can edit; audit-logged) -->
        <div class="tabpane on" id="ap-profile">
          <div class="card">
            <div style="display:flex;justify-content:space-between;align-items:center;margin-bottom:10px"><h3 style="font-size:18px">Student Profile</h3><button class="btn btn-ghost btn-sm" onclick="alert('Edit profile — opens the profiling form. Editable by the assigned Fellow and Program Admin; every edit is audit-logged (RBAC).')">Edit profile</button></div>
            <p style="font-size:12.5px;color:var(--muted);margin-bottom:6px">Editable by the assigned Fellow and Program Admin · all edits are audit-logged.</p>
            <div class="grid g-2">
              <div class="field"><div class="k">Date of Birth</div><div class="v">14 August 2005</div></div>
              <div class="field"><div class="k">Gender</div><div class="v">Female</div></div>
              <div class="field"><div class="k">Caste / Category</div><div class="v">SC</div></div>
              <div class="field"><div class="k">Religion</div><div class="v">Buddhist</div></div>
              <div class="field"><div class="k">Annual Family Income</div><div class="v">₹1,80,000</div></div>
              <div class="field"><div class="k">State / District</div><div class="v">Maharashtra · Pune</div></div>
              <div class="field"><div class="k">Course / Year</div><div class="v">B.Com · Year 2</div></div>
              <div class="field"><div class="k">Last Exam %</div><div class="v">78%</div></div>
              <div class="field"><div class="k">Mobile</div><div class="v">+91 98xxx xxx21</div></div>
              <div class="field"><div class="k">Aadhaar</div><div class="v mask">xxxx xxxx 4821</div></div>
              <div class="field"><div class="k">Guardian</div><div class="v">Ramesh Pawar (Father)</div></div>
              <div class="field"><div class="k">Consent</div><div class="v" style="color:var(--green)">Captured · 12 Apr 2026</div></div>
            </div>
          </div>
        </div>

        <!-- Scholarships tab (read-only eligibility view) -->
        <div class="tabpane" id="ap-sch">
          <p style="font-size:13px;color:var(--ink-soft);margin-bottom:14px">Auto-computed from the student's profile (read-only oversight). Submission and document handling are performed by the assigned Fellow.</p>
          <div class="card" style="margin-bottom:12px"><div class="sch-card">
            <div class="logo" style="background:linear-gradient(140deg,#0f857a,#0a3f3a)">PM</div>
            <div class="body"><div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px"><h4>Post-Matric Scholarship for SC Students</h4><span class="pill blue"><span class="pdot"></span>Submitted</span></div>
            <div class="meta"><span><b class="amount">₹50,000</b>/year</span><span>Documents: 100% Accepted</span></div></div>
          </div></div>
          <div class="card" style="margin-bottom:12px"><div class="sch-card">
            <div class="logo" style="background:linear-gradient(140deg,#e8821e,#f6b24a);color:#3a2400">MS</div>
            <div class="body"><div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px"><h4>Maharashtra State Minority Scholarship</h4><span class="pill green"><span class="pdot"></span>Ready to Submit</span></div>
            <div class="meta"><span><b class="amount">₹25,000</b>/year</span><span>Documents: 100% Accepted</span></div></div>
          </div></div>
          <div class="card"><div class="sch-card">
            <div class="logo" style="background:linear-gradient(140deg,#2f6fd6,#1c4ea8)">AB</div>
            <div class="body"><div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px"><h4>Dr. Babasaheb Ambedkar Scholarship</h4><span class="pill amber"><span class="pdot"></span>Documents Pending</span></div>
            <div class="meta"><span><b class="amount">₹38,000</b>/year</span><span>Documents: 6/8 Accepted</span></div></div>
          </div></div>
        </div>

        <!-- Documents tab (ADMIN ACTION: Accept/Reject via vAct; .vrow structure) -->
        <div class="tabpane" id="ap-docs">
          <p style="font-size:13px;color:var(--ink-soft);margin-bottom:14px">Review documents uploaded by the Fellow. Accept or Reject each (rejection requires a reason). Procurement and upload are the Fellow's actions — the Admin verifies.</p>
          <div class="card" style="padding:0">
            <div class="vrow" data-vstatus="all">
              <div class="vi"><svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.7"><path d="M14 3H7a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V8Z"/><path d="M14 3v5h5"/></svg></div>
              <div class="vb"><b>Domicile Certificate</b><small>Type: Domicile · uploaded by Fellow · 2 Jun · Re-upload Version 1</small></div>
              <div class="vacts"><button class="btn btn-ghost btn-sm">View file</button><button class="btn btn-danger btn-sm" onclick="vAct(this,'reject')">Reject</button><button class="btn btn-accept btn-sm" onclick="vAct(this,'accept')">Accept</button></div>
            </div>
            <div class="vrow" data-vstatus="done">
              <div class="vi" style="background:var(--green-50);color:var(--green)"><svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.7"><path d="M14 3H7a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V8Z"/><path d="M14 3v5h5"/></svg></div>
              <div class="vb"><b>Income Certificate</b><small>Type: Income · Accepted 20 May</small></div>
              <div class="vacts"><span class="pill green"><span class="pdot"></span>Accepted</span></div>
            </div>
            <div class="vrow" data-vstatus="done">
              <div class="vi" style="background:var(--red-50);color:var(--red)"><svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.7"><path d="M12 9v4m0 4h.01M10.3 3.9 1.8 18a2 2 0 0 0 1.7 3h17a2 2 0 0 0 1.7-3L13.7 3.9a2 2 0 0 0-3.4 0Z"/></svg></div>
              <div class="vb"><b>Caste Validity Certificate</b><small style="color:var(--red)">Rejected: "Not clearly legible" — awaiting Fellow re-upload</small></div>
              <div class="vacts"><span class="pill red"><span class="pdot"></span>Rejected</span></div>
            </div>
            <div class="vrow" data-vstatus="done">
              <div class="vi" style="background:var(--amber-50);color:var(--amber)"><svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.7"><path d="M14 3H7a2 2 0 0 0-2 2v14a2 2 0 0 0 2 2h10a2 2 0 0 0 2-2V8Z"/><path d="M14 3v5h5"/></svg></div>
              <div class="vb"><b>Non-Creamy Layer Certificate</b><small>Type: Caste · awaiting Fellow upload (procurement in progress)</small></div>
              <div class="vacts"><span class="pill amber"><span class="pdot"></span>Pending</span></div>
            </div>
          </div>
        </div>

        <!-- Applications tab (READ-ONLY tracking; no operator actions) -->
        <div class="tabpane" id="ap-apps">
          <p style="font-size:13px;color:var(--ink-soft);margin-bottom:14px">Read-only tracking. Submission on the government portal, status updates and disbursement recording are performed by the assigned Fellow.</p>
          <div class="card" style="margin-bottom:16px">
            <div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px;align-items:center"><h4 class="serif" style="font-size:17px">Post-Matric Scholarship for SC Students</h4><span class="pill blue"><span class="pdot"></span>Under Scrutiny</span></div>
            <div class="meta" style="display:flex;gap:16px;flex-wrap:wrap;color:var(--ink-soft);font-size:13px;margin:8px 0 12px"><span>Application ID: MH-PM-2026-44871</span><span>Submitted 28 May</span><span>Last status check: 2 Jun</span></div>
            <div class="tl">
              <div class="tl-item done"><div class="knob"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg></div><div class="t">Submitted</div><div class="d">28 May</div></div>
              <div class="tl-item active"><div class="knob"></div><div class="t">Under Scrutiny</div><div class="d">Department processing</div></div>
              <div class="tl-item future"><div class="t">Application Approved → Benefits Received</div><div class="d">—</div></div>
            </div>
          </div>
          <div class="card" style="border-left:4px solid var(--green)">
            <div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px;align-items:center"><h4 class="serif" style="font-size:17px">Maharashtra State Minority Scholarship</h4><span class="pill green"><span class="pdot"></span>Ready to Submit</span></div>
            <p style="font-size:13px;color:var(--ink-soft);margin-top:8px">All documents Accepted. Awaiting Fellow submission on the portal.</p>
          </div>
        </div>
            <div class="activity">
              <h3>Activity</h3>
              <div class="comment-box"><span class="av2">M</span><input placeholder="Add a comment…" /></div>
              <div class="act-item"><div class="ai-dot"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 6 9 17l-5-5"/></svg></div><div><div class="ai-t"><b>Program Admin</b> accepted Income Certificate</div><div class="ai-d">20 May, 11:03 AM</div></div></div>
              <div class="act-item"><div class="ai-dot"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="m22 2-7 20-4-9-9-4Z"/></svg></div><div><div class="ai-t"><b>Fellow</b> submitted the Post-Matric application on the portal</div><div class="ai-d">2 Jun, 4:12 PM</div></div></div>
              <div class="act-item"><div class="ai-dot"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><circle cx="12" cy="8" r="3.6"/><path d="M4.5 20a7.5 7.5 0 0 1 15 0"/></svg></div><div><div class="ai-t"><b>Fellow</b> onboarded this student</div><div class="ai-d">12 Apr, 10:20 AM</div></div></div>
            </div>
          </div>
          <aside class="rec-side">
            <div class="side-card"><h5>Assigned Fellow</h5><div class="assignee"><span class="av2" id="asFellowAv">R</span><small id="asFellow">Rahul More</small></div><button class="btn btn-ghost btn-sm" style="margin-top:10px;width:100%;justify-content:center" onclick="alert('Reassign Fellow — pick another Fellow to take over this caseload. The change is audit-logged (RBAC).')">Reassign Fellow</button></div>
            <div class="side-card"><h5>Tags</h5><div class="tagchips"><span class="tagchip">SC</span><span class="tagchip">Pune</span><span class="tagchip">Post-Matric</span></div></div>
            <div class="side-card"><h5>Details</h5><div class="meta-line">Created <b>12 Apr 2026</b></div><div class="meta-line">Last modified <b>2 Jun 2026</b></div><div class="meta-line">Student ID <b>SMV-2026-01182</b></div></div>
          </aside>
        </div>
      </section>
```

- [ ] **Step 3: Verify the section now exists (passing check)**

Run (Grep tool): pattern `id="a-student"` (exact), output_mode `count`.
Expected: **1 match**. Also Grep `id="ap-profile"` → 1 match, `id="asPath"` → 1 match, `id="asFellow"` → 1 match.

- [ ] **Step 4: Commit**

```bash
git add wireframe/index.html
git commit -m "Add Admin student-detail screen markup (a-student)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 2: Add the `goStudent(row)` helper + stage→step map

**Files:**
- Modify: `wireframe/index.html` — add JS immediately after the `setCrumb` function block (ends with line `el.innerHTML=home+...join('');` then `}` at line ~2367), before the `document.querySelectorAll('[data-go]')...` line (~2368).

- [ ] **Step 1: Confirm the helper does not yet exist (failing check)**

Run (Grep tool): pattern `function goStudent` in `wireframe/index.html`, output_mode `count`.
Expected: **0 matches**.

- [ ] **Step 2: Add the helper after `setCrumb`**

Find this exact block:

```javascript
function setCrumb(id){
  const txt=CRUMBS[id]; if(!txt)return;
  const el=id.indexOf('a-')===0?document.getElementById('crumbAdmin'):document.getElementById('crumbFellow');
  if(!el)return;
  const home='<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 10.5 12 3l9 7.5"/><path d="M5 9.5V21h14V9.5"/></svg>';
  const parts=txt.split(' › ');
  el.innerHTML=home+parts.map((p,i)=> i<parts.length-1 ? '<span>'+p+'</span><span class="sep">›</span>' : '<b>'+p+'</b>').join('');
}
```

Insert the following **immediately after** the closing `}` of `setCrumb`:

```javascript
// ---- Admin: open a student's record from an All-Students row (dynamic header) ----
const A_JOURNEY=['Onboarded','Scholarship Identified','Documents Complete','Submitted','Approved','Disbursed'];
const STAGE_TO_STEP={onboarded:0,documents:2,submitted:3,approved:4,disbursed:5};
function goStudent(row){
  const name=row.querySelector('.who').textContent.trim();
  const av=row.querySelector('.sa');
  const stage=row.dataset.stage, fellow=row.dataset.fellow||'Unassigned';
  // identity header
  const avEl=document.getElementById('asAvatar');
  avEl.textContent=av.textContent.trim();
  avEl.setAttribute('style', av.getAttribute('style')||'');
  document.getElementById('asName').textContent=name;
  document.getElementById('asStagePill').innerHTML=row.querySelector('.pill').outerHTML;
  // assigned fellow (sidebar)
  document.getElementById('asFellow').textContent=fellow;
  document.getElementById('asFellowAv').textContent=fellow.trim().charAt(0);
  // journey: rebuild 6 steps, current driven by stage
  const cur=(stage in STAGE_TO_STEP)?STAGE_TO_STEP[stage]:0;
  const check='<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2.4"><path d="M20 6 9 17l-5-5"/></svg>';
  document.getElementById('asPath').innerHTML=A_JOURNEY.map(function(lbl,i){
    const cls=i<cur?'done':(i===cur?'current':'upcoming');
    const bub=i<cur?check:(i+1);
    return '<div class="step '+cls+'"><div class="bub">'+bub+'</div><div class="lbl">'+lbl+'</div><div class="sub"></div></div>';
  }).join('');
  // reset to Profile tab (ftab toggles .on globally, so normalize on open)
  const tabBtns=document.querySelectorAll('#a-student .tabs button');
  tabBtns.forEach(function(b,i){b.classList.toggle('on',i===0);});
  ['ap-profile','ap-sch','ap-docs','ap-apps'].forEach(function(id,i){document.getElementById(id).classList.toggle('on',i===0);});
  // breadcrumb + navigate
  CRUMBS['a-student']='Students › '+name;
  go('a-student');
}
```

- [ ] **Step 3: Verify the helper exists (passing check)**

Run (Grep tool): pattern `function goStudent` → expect **1 match**. Grep `STAGE_TO_STEP` → expect **2 matches** (declaration + usage).

- [ ] **Step 4: Commit**

```bash
git add wireframe/index.html
git commit -m "Add goStudent(row) helper: dynamic admin student-detail header

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 3: Wire the All-Students rows + add CRUMBS fallback

**Files:**
- Modify: `wireframe/index.html` — the 6 `<tr>` rows in `#aStuTable` `<tbody>` (lines ~2248-2253), and the `CRUMBS` object literal (line ~2359).

- [ ] **Step 1: Confirm rows still route to the verify queue (failing check)**

Run (Grep tool): pattern `onclick="go\('a-verify'\)"` in `wireframe/index.html`, output_mode `count`.
Expected: **6 matches** (the All-Students rows). After this task, expect **0**.

- [ ] **Step 2: Repoint each All-Students row to `goStudent(this)`**

In the `#aStuTable` `<tbody>`, replace the 6 occurrences of `onclick="go('a-verify')"` on the `<tr>` elements with `onclick="goStudent(this)"`. Use a replace-all on this exact attribute string scoped to those rows. The rows are:

```html
            <tr data-stage="submitted" data-fellow="Rahul More" onclick="go('a-verify')">
            <tr data-stage="documents" data-fellow="Rahul More" onclick="go('a-verify')">
            <tr data-stage="approved" data-fellow="Sandip K" onclick="go('a-verify')">
            <tr data-stage="disbursed" data-fellow="Dhanashree O" onclick="go('a-verify')">
            <tr data-stage="documents" data-fellow="Sandip K" onclick="go('a-verify')">
            <tr data-stage="onboarded" data-fellow="Dhanashree O" onclick="go('a-verify')">
```

become (only the `onclick` changes):

```html
            <tr data-stage="submitted" data-fellow="Rahul More" onclick="goStudent(this)">
            <tr data-stage="documents" data-fellow="Rahul More" onclick="goStudent(this)">
            <tr data-stage="approved" data-fellow="Sandip K" onclick="goStudent(this)">
            <tr data-stage="disbursed" data-fellow="Dhanashree O" onclick="goStudent(this)">
            <tr data-stage="documents" data-fellow="Sandip K" onclick="goStudent(this)">
            <tr data-stage="onboarded" data-fellow="Dhanashree O" onclick="goStudent(this)">
```

Note: only `#aStuTable` rows use `go('a-verify')` as a row `onclick`. The verification queue's nav link (`data-go="a-verify"`) and the MIS funnel are unaffected.

- [ ] **Step 3: Add a CRUMBS fallback entry for `a-student`**

Find the `CRUMBS` object literal:

```javascript
const CRUMBS={'f-home':'Home','f-students':'Students','f-onboard':'Students › New Student','f-student':'Students › Aarti Pawar','f-applications':'Applications','f-scholarship':'Applications › New Entry','f-docapp':'Students › Aarti Pawar › Documentation','f-profile':'My Settings','a-home':'Home','a-verify':'Document Verification','a-catalogue':'Scholarship Catalogue','a-students':'Students','a-fellows':'Users','a-profile':'My Settings'};
```

Add `'a-student':'Students › Student'` after the `'a-students':'Students'` entry (this is only a fallback; `goStudent` overwrites it with the real name before navigating):

```javascript
const CRUMBS={'f-home':'Home','f-students':'Students','f-onboard':'Students › New Student','f-student':'Students › Aarti Pawar','f-applications':'Applications','f-scholarship':'Applications › New Entry','f-docapp':'Students › Aarti Pawar › Documentation','f-profile':'My Settings','a-home':'Home','a-verify':'Document Verification','a-catalogue':'Scholarship Catalogue','a-students':'Students','a-student':'Students › Student','a-fellows':'Users','a-profile':'My Settings'};
```

- [ ] **Step 4: Verify wiring (passing check)**

Run (Grep tool):
- pattern `onclick="go\('a-verify'\)"` → expect **0 matches**.
- pattern `goStudent\(this\)` → expect **6 matches**.
- pattern `'a-student':'Students › Student'` → expect **1 match**.

- [ ] **Step 5: Commit**

```bash
git add wireframe/index.html
git commit -m "Route Admin All-Students rows to the new student-detail view

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 4: Manual browser acceptance + heartbeat/memory update

**Files:**
- Modify: `wireframe/BUILD_STATUS.md` (heartbeat).
- Modify: `C:\Users\Jitesh (DHW-L06)\.claude\projects\C--Users-Jitesh--DHW-L06--Desktop-Samavesh\memory\samavesh_wireframe_scope.md` (remove the "Admin lacks a dedicated student-detail view" caveat).

- [ ] **Step 1: Open the prototype in a browser**

Open `wireframe/index.html` directly (or the live Pages URL after pushing). Sign in as Admin: mobile `9800000001` (or the Program Admin quick-login button).

- [ ] **Step 2: Run the acceptance checklist** (this is the real gate)

Navigate to **All Students** and verify each item:
1. Click **Aarti Pawar** (Submitted) → lands on the record; header shows "Aarti Pawar", blue **Submitted** pill, journey **current** step = Submitted, sidebar **Assigned Fellow = Rahul More**, breadcrumb `Home › Students › Aarti Pawar`.
2. Back-link `‹ Back to All Students` returns to the table.
3. Click **Ganesh Pawar** (Onboarded, Dhanashree O) → header now reads "Ganesh Pawar", grey **Onboarded** pill, journey current = Onboarded, sidebar fellow = Dhanashree O, avatar initial = G with its row gradient. (Confirms the header is truly dynamic, not stuck on Aarti.)
4. Tabs switch (Profile / Scholarships / Documents / Applications) via `ftab`.
5. **Documents** tab: the **Domicile Certificate** row has `[View file] [Reject] [Accept]`. Click **Accept** → pill flips to green Accepted, buttons clear (reuses `vAct`). Click **Reject** on a fresh open → prompts for a reason, flips to red Rejected.
6. **Profile** shows **Edit profile** (alert). Sidebar shows **Reassign Fellow** (alert). **Scholarships** and **Applications** show **no** Upload/Submit/Update/Disburse buttons.
7. Re-open the student detail → it always opens on the **Profile** tab (tab reset works).
8. **No regression:** sign in as Fellow (`9800000011`), open a student via My Students → the Fellow record still shows its operator buttons and tabs work; Document Verification queue (Admin) still shows its pending count correctly.

- [ ] **Step 3: Update the heartbeat**

In `wireframe/BUILD_STATUS.md`, under the Program Admin section add:

```markdown
- [x] **Student Detail (a-student)** — dynamic per-row header (name/avatar/stage/fellow/journey), oversight + admin-only actions: Accept/Reject docs (reuses vAct), Edit profile, Reassign Fellow; read-only Scholarships/Applications
```

And in "Still to do / next", remove the line:
`- [ ] Admin: build a real student-detail view (currently admin rows route to Verify)`

- [ ] **Step 4: Update the wireframe-scope memory**

In `memory/samavesh_wireframe_scope.md`, change the sentence
`Admin lacks a dedicated student-detail view (rows route to Verify for now).`
to note it is now built:
`Admin student-detail view (a-student) is built — dynamic header from the clicked row, oversight + Accept/Reject + Edit profile + Reassign Fellow.`

- [ ] **Step 5: Commit**

```bash
git add wireframe/BUILD_STATUS.md
git commit -m "Heartbeat: Admin student-detail view complete

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

(The memory file lives outside the repo and is saved by the Write tool, not committed.)

---

## Self-Review (completed by plan author)

**Spec coverage:**
- New `a-student` section → Task 1. ✓
- `goStudent(row)` dynamic header + breadcrumb + tab reset + journey → Task 2. ✓
- Row re-routing + CRUMBS fallback → Task 3. ✓
- Profile (Edit), Scholarships (read-only), Documents (vAct Accept/Reject via `.vrow`), Applications (read-only) → Task 1 markup. ✓
- Sidebar Assigned Fellow + Reassign, Tags, Details; Activity timeline → Task 1 markup. ✓
- No new CSS; reuses `ftab`/`vAct`/`go`/`setCrumb`/`CRUMBS` → confirmed by design. ✓
- Manual verification (no test runner) → Task 4. ✓

**Placeholder scan:** No TBD/TODO; all markup and JS shown in full; the empty `.sub` divs in the journey are intentional (no per-step date in the dynamic view). ✓

**Type/name consistency:** Element IDs are consistent across tasks — `asAvatar`, `asName`, `asSub`, `asStagePill`, `asPath`, `asFellow`, `asFellowAv`, and panes `ap-profile`/`ap-sch`/`ap-docs`/`ap-apps` (Task 1 markup) are exactly the IDs read/written by `goStudent` (Task 2). `STAGE_TO_STEP` keys (`onboarded`/`documents`/`submitted`/`approved`/`disbursed`) match the `data-stage` values on the rows (Task 3). `vAct` requires `.vrow`/`.vacts`/`.vi`/`.vb small` — all present in the Documents-tab rows. ✓
