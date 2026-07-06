# Unified Applications Caseload — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Collapse the Fellow's (and Admin's) duplicate "Applications" + "My Cases" screens into one data-driven caseload with a single status ladder, two manual exception flags (On Hold / Discarded), and a derived triage — so the Fellow updates one status at the real moment instead of maintaining two.

**Architecture:** One JS array `CASES` (one record per student × scholarship) is the single source of truth. A small engine (`caseAttention`, `casePill`, `renderCaseList`, filter + action handlers) renders that array into the Fellow screen (`#f-applications`, operator actions) and the Admin screen (`#a-tasks` → relabelled "Applications", oversight + Reassign, read-only status). The old static `#f-applications` rows, the whole `#f-tasks` screen, and the static `#a-tasks` table are retired. Everything is in the single file `wireframe/index.html`.

**Tech Stack:** Vanilla HTML/CSS/JS in one file. No build, no framework. Verification is **headless-Chrome render** (not unit tests).

## Global Constraints

- **Single file:** all edits are in `wireframe/index.html`.
- **Per-task GATED commit:** the user chose per-task commits; **ask before pushing** (never push without explicit approval) — memory `samavesh-git-workflow`. Commit only `wireframe/index.html` (+ `BUILD_STATUS.md` in the last task); never `git add -A` (many untracked reference files must stay untracked).
- **Reuse existing vocabulary only** (memory `samavesh-use-only-context-terms`): the 8 statuses and both flag names are already in the wireframe. Invent no new domain terms.
- **No inert UI** (memory `samavesh-no-half-baked`): every button does something visible; no `prompt()` — use inline reason forms.
- **No new nav confusion:** exactly ONE caseload rail item per role after this ("Applications").
- **Render-verify behavior, not structure** (memory `samavesh-render-verify`): kill chrome first, use `--headless=new` + `--user-data-dir="C:\Temp\cprof"` (the default `--headless` produced no screenshot this environment; the isolated profile avoids a lock). Read each PNG.
- **Ripple consistency** (memory `samavesh-ripple-check`): Fellow and Admin stay consistent; update every sibling surface named here.

### The status ladder (single source of truth) — order + pill color

| # | status string (exact) | pill class | terminal |
|---|---|---|---|
| 1 | `Eligibility Identified` | grey | no |
| 2 | `Documents Pending` | amber | no |
| 3 | `Ready to Submit` | amber | no |
| 4 | `Under Scrutiny` | blue | no |
| 5 | `Application Approved · funds awaited` | teal | no |
| 6 | `Benefits Received` | green | yes |
| 7 | `Re-apply` | amber | no |
| 8 | `Rejected` | red | yes |

Flags (layered, manual): `On Hold` (+reason), `Discarded` (+reason) — rendered with existing `.case-state.hold` / `.case-state.discarded` pill classes.

### Render-verify snippet (reuse in every task)

```powershell
Copy-Item "C:\Users\Jitesh (DHW-L06)\Desktop\Samavesh\wireframe\index.html" "C:\Temp\swf.html" -Force
try { Get-Process chrome -ErrorAction Stop | Stop-Process -Force } catch {}
Start-Sleep -Milliseconds 500
& "C:\Program Files\Google\Chrome\Application\chrome.exe" --headless=new --disable-gpu --no-sandbox --hide-scrollbars --user-data-dir="C:\Temp\cprof" --window-size=1440,1400 --virtual-time-budget=4000 --screenshot="C:\Temp\shot.png" "file:///C:/Temp/swf.html?role=fellow&screen=f-applications" 2>&1 | Out-String
```
For flows needing interaction, inject a `<script>window.addEventListener("load",function(){setTimeout(function(){ ...calls... },700);});</script>` before `</body>` into a copy, as in the approval-review plan.

---

## Task 1: Case engine + data + rebuilt Fellow `#f-applications` (data-driven)

**Files:**
- Modify: `wireframe/index.html` — add CSS in the `<style>` block (before `</style>`, line ~1141); replace the `#f-applications` body (currently lines ~3070–3137); add the engine + data + handlers in the `<script>` (near the other Fellow functions, e.g. after `fellowDocAction`); add a `renderCaseList('fellow')` call to the DOMContentLoaded that already calls `onbInit`/`renderApprovalList`.

**Interfaces:**
- Produces (consumed by Task 4 Admin + Task 2/5 dashboards):
  - `CASES` (array), `CASE_STATUS` (array of the 8 status strings in order).
  - `caseAttention(c) -> 'action'|'awaiting'|'done'|'hold'`
  - `casePill(status) -> classname` and `caseStatusHtml(c) -> html`
  - `renderCaseList(scope)` where `scope ∈ {'fellow','admin'}` — renders into `#caseListFellow` (fellow) or `#caseListAdmin` (admin, built in Task 4).
  - `filterCaseList(scope, bucket, btn)`, `caseAdvance(id)`, `caseSetStatus(id,status)`, `caseHold(id)`, `caseResume(id)`, `caseDiscard(id)`, `caseReopen(id)`, `casePendReason(id,type)`, `caseSaveReason(id)`, `caseCancelReason(id)`.
- Consumes: existing `go`, `.stu-row/.sa/.si/.smeta/.chev2` classes, `.pill.*` classes, `.case-state.hold/.discarded` classes.

- [ ] **Step 1: Add CSS.** Insert immediately before `</style>` (line ~1141):

```css
/* Unified caseload */
.case-menu{position:relative;display:inline-block}
.case-menu>summary{list-style:none;cursor:pointer;border:1px solid var(--line);border-radius:8px;padding:5px 9px;font-size:13px;background:#fff;color:var(--ink-soft)}
.case-menu>summary::-webkit-details-marker{display:none}
.case-menu[open]>.cm-pop{display:block}
.cm-pop{display:none;position:absolute;right:0;top:calc(100% + 4px);z-index:30;background:#fff;border:1px solid var(--line);border-radius:10px;box-shadow:0 12px 30px rgba(0,0,0,.14);min-width:170px;padding:6px}
.cm-pop button{display:block;width:100%;text-align:left;border:none;background:none;padding:8px 10px;border-radius:7px;font-size:13px;cursor:pointer;color:var(--ink)}
.cm-pop button:hover{background:var(--paper)}
.case-flags{display:flex;gap:6px;flex-wrap:wrap;margin-top:4px}
.case-reason{display:flex;gap:8px;align-items:center;flex-wrap:wrap;margin-top:8px;width:100%}
.case-reason input{flex:1;min-width:160px;padding:7px 10px;border:1px solid var(--line);border-radius:8px;font-size:13px}
.cl-row .smeta{display:flex;flex-direction:column;align-items:flex-end;gap:4px}
```

- [ ] **Step 2: Replace the `#f-applications` body.** Replace the block from `<div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:18px">` (the old status chips, line ~3075) through the closing `</div>` of `#appList` (line ~3136) — i.e. everything between the `page-h` and the section close — with:

```html
        <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:18px" id="fCaseChips">
          <span class="tagk chip on" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('fellow','all',this)">All</span>
          <span class="tagk chip" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('fellow','action',this)">Action needed</span>
          <span class="tagk chip" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('fellow','awaiting',this)">Awaiting</span>
          <span class="tagk chip" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('fellow','hold',this)">On Hold</span>
          <span class="tagk chip" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('fellow','done',this)">Done</span>
        </div>
        <div class="listbar"><span class="lbsel"></span><span class="cnt"><b id="fCaseCnt">0</b> Applications</span><div class="lb-tools"><button><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 5h18M6 12h12M10 19h4"/></svg>Filter</button><button><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 6h13M3 12h9M3 18h5M18 9l3 3-3 3"/></svg>Sort</button></div></div>
        <div id="caseListFellow"></div>
        <div class="empty" id="fCaseEmpty" style="display:none">No applications in this view.</div>
```

Also update the `page-h` subtitle (line ~3072) from "Track and update every scholarship application. Log into the government portal, then update the status here." to: `One row per student × scholarship — your whole caseload. Do the real work (collect docs, submit, record disbursement) and the status advances itself; flag On Hold or Discard from the ⋯ menu.` Keep the `+ New Scholarship Entry` button.

- [ ] **Step 3: Add the data + engine + handlers.** Insert in the `<script>` after `fellowDocAction` (find `function fellowDocAction`); paste:

```js
/* ===== Unified caseload (one record per student × scholarship) ===== */
var CASE_STATUS = ['Eligibility Identified','Documents Pending','Ready to Submit','Under Scrutiny','Application Approved · funds awaited','Benefits Received','Re-apply','Rejected'];
var CASES = [
  {id:'CASE-2026-001', student:'Aarti Pawar', av:'A', avStyle:'background:linear-gradient(140deg,#e8821e,#f6b24a);color:#3a2400', scheme:'Post-Matric SC', appId:'MH-PM-2026-44871', fellow:'Rahul More', status:'Under Scrutiny', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'2 Jun'},
  {id:'CASE-2026-002', student:'Aarti Pawar', av:'A', avStyle:'background:linear-gradient(140deg,#e8821e,#f6b24a);color:#3a2400', scheme:'Maharashtra State Minority', appId:'—', fellow:'Rahul More', status:'Ready to Submit', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'28 Apr'},
  {id:'CASE-2026-003', student:'Aarti Pawar', av:'A', avStyle:'background:linear-gradient(140deg,#e8821e,#f6b24a);color:#3a2400', scheme:'Dr. Babasaheb Ambedkar', appId:'—', fellow:'Rahul More', status:'Documents Pending', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'22 May'},
  {id:'CASE-2026-004', student:'Kavya Deshmukh', av:'K', avStyle:'background:linear-gradient(140deg,#2f6fd6,#1c4ea8);color:#fff', scheme:'OBC Post-Matric', appId:'MH-OBC-2026-30912', fellow:'Rahul More', status:'Under Scrutiny', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'30 May'},
  {id:'CASE-2026-005', student:'Sneha Patil', av:'S', avStyle:'background:linear-gradient(140deg,#0f857a,#0a3f3a);color:#fff', scheme:'Ambedkar Swadhar Yojana', appId:'MH-AB-2026-11902', fellow:'Rahul More', status:'Application Approved · funds awaited', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'3 Jun'},
  {id:'CASE-2026-006', student:'Priya Sonawane', av:'P', avStyle:'background:linear-gradient(140deg,#2f6fd6,#1c4ea8);color:#fff', scheme:'GoI Post-Matric', appId:'MH-PM-2026-20551', fellow:'Rahul More', status:'Benefits Received', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'20 Mar'},
  {id:'CASE-2026-007', student:'Suresh Jadhav', av:'S', avStyle:'background:linear-gradient(140deg,#0f857a,#0a3f3a);color:#fff', scheme:'OBC Post-Matric', appId:'—', fellow:'Rahul More', status:'Ready to Submit', onHold:true, onHoldReason:'Family unreachable since 22 May', discarded:false, discardedReason:'', updated:'1 Mar'},
  {id:'CASE-2026-008', student:'Suresh Jadhav', av:'S', avStyle:'background:linear-gradient(140deg,#0f857a,#0a3f3a);color:#fff', scheme:'Shahu Maharaj Merit', appId:'MH-SM-2026-77120', fellow:'Rahul More', status:'Re-apply', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'4 Jun'},
  {id:'CASE-2026-009', student:'Imran Shaikh', av:'I', avStyle:'background:linear-gradient(140deg,#e8821e,#f6b24a);color:#3a2400', scheme:'VJNT Post-Matric', appId:'MH-VJ-2026-51004', fellow:'Rahul More', status:'Rejected', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'6 Jun'},
  {id:'CASE-2026-010', student:'Ravi Khade', av:'R', avStyle:'background:linear-gradient(140deg,#9ca3af,#6b7280);color:#fff', scheme:'Aadhar Seeding (KYC)', appId:'—', fellow:'Rahul More', status:'Eligibility Identified', onHold:false, onHoldReason:'', discarded:true, discardedReason:'Student dropped out 14 May', updated:'12 May'},
  {id:'CASE-2026-011', student:'Sneha Patil', av:'S', avStyle:'background:linear-gradient(140deg,#0f857a,#0a3f3a);color:#fff', scheme:'Post-Matric SC', appId:'MH-PM-2026-33471', fellow:'Sandip K', status:'Benefits Received', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'10 Feb'},
  {id:'CASE-2026-012', student:'Ganesh Pawar', av:'G', avStyle:'background:linear-gradient(140deg,#0f857a,#0a3f3a);color:#fff', scheme:'Post-Matric SC', appId:'—', fellow:'Dhanashree O', status:'Documents Pending', onHold:false, onHoldReason:'', discarded:false, discardedReason:'', updated:'3 Jun'}
];
var CASE_FELLOW = 'Rahul More';   // the logged-in Fellow (demo)
function caseAttention(c){
  if(c.discarded) return 'done';
  if(c.onHold) return 'hold';
  if(c.status==='Benefits Received'||c.status==='Rejected') return 'done';
  if(c.status==='Under Scrutiny'||c.status==='Application Approved · funds awaited') return 'awaiting';
  return 'action';
}
function casePill(status){
  return {'Eligibility Identified':'grey','Documents Pending':'amber','Ready to Submit':'amber','Under Scrutiny':'blue','Application Approved · funds awaited':'teal','Benefits Received':'green','Re-apply':'amber','Rejected':'red'}[status]||'grey';
}
function caseById(id){ return CASES.filter(function(c){return c.id===id;})[0]; }
function caseFlagsHtml(c){
  var h='';
  if(c.onHold) h+='<span class="case-state hold">On Hold — '+c.onHoldReason+'</span>';
  if(c.discarded) h+='<span class="case-state discarded">Discarded — '+c.discardedReason+'</span>';
  return h?('<div class="case-flags">'+h+'</div>'):'';
}
// Fellow-scope primary action per status (the single status-advancing action).
function casePrimaryHtml(c){
  if(c.discarded||c.onHold) return '';
  switch(c.status){
    case 'Eligibility Identified': return '<button class="btn btn-primary btn-sm" onclick="event.stopPropagation();caseAdvance(\''+c.id+'\')">Start — collect documents</button>';
    case 'Documents Pending': return '<button class="btn btn-ghost btn-sm" onclick="event.stopPropagation();go(\'f-student\')">Open documents</button>';
    case 'Ready to Submit': return '<button class="btn btn-primary btn-sm" onclick="event.stopPropagation();caseAdvance(\''+c.id+'\')">Submit &amp; record</button>';
    case 'Under Scrutiny': return '<details class="case-menu" onclick="event.stopPropagation()"><summary>Update outcome</summary><div class="cm-pop">'+
      '<button onclick="caseSetStatus(\''+c.id+'\',\'Application Approved · funds awaited\')">Approved · funds awaited</button>'+
      '<button onclick="caseSetStatus(\''+c.id+'\',\'Re-apply\')">Returned — Re-apply</button>'+
      '<button onclick="caseSetStatus(\''+c.id+'\',\'Rejected\')">Rejected</button></div></details>';
    case 'Application Approved · funds awaited': return '<button class="btn btn-gold btn-sm" onclick="event.stopPropagation();caseAdvance(\''+c.id+'\')">Record disbursement</button>';
    case 'Re-apply': return '<button class="btn btn-primary btn-sm" onclick="event.stopPropagation();caseSetStatus(\''+c.id+'\',\'Ready to Submit\')">Re-apply</button>';
    default: return '';
  }
}
// The ⋯ overflow: the two manual exception flags + reopen.
function caseMenuHtml(c){
  var items='';
  if(c.discarded){ items+='<button onclick="caseReopen(\''+c.id+'\')">Reopen case</button>'; }
  else {
    if(c.onHold) items+='<button onclick="caseResume(\''+c.id+'\')">Resume</button>';
    else items+='<button onclick="casePendReason(\''+c.id+'\',\'hold\')">Put on hold…</button>';
    items+='<button onclick="casePendReason(\''+c.id+'\',\'discard\')">Discard case…</button>';
    if(c.status==='Benefits Received'||c.status==='Rejected') items+='<button onclick="caseReopen(\''+c.id+'\')">Reopen</button>';
  }
  return '<details class="case-menu" onclick="event.stopPropagation()"><summary>⋯</summary><div class="cm-pop">'+items+'</div></details>';
}
function caseRowHtml(c, scope){
  var pend = c._pend ?
    '<div class="case-reason"><input id="cr_'+c.id+'" placeholder="'+(c._pend==='hold'?'Reason for hold (e.g. family unreachable)':'Reason for discarding (e.g. student withdrew)')+'" /><button class="btn btn-primary btn-sm" onclick="event.stopPropagation();caseSaveReason(\''+c.id+'\')">Save</button><button class="btn btn-ghost btn-sm" onclick="event.stopPropagation();caseCancelReason(\''+c.id+'\')">Cancel</button></div>' : '';
  var sub = c.scheme+' · '+(c.appId&&c.appId!=='—'?('App ID '+c.appId):'no App ID yet')+(scope==='admin'?(' · '+c.fellow):'');
  var actions = scope==='admin'
    ? '<button class="btn btn-ghost btn-sm" onclick="event.stopPropagation();caseReassign2(\''+c.id+'\')">Reassign</button>'
    : (casePrimaryHtml(c)+caseMenuHtml(c));
  return '<div class="stu-row cl-row" style="cursor:pointer" onclick="go(\''+(scope==='admin'?'a-student':'f-student')+'\')">'+
    '<div class="sa" style="'+c.avStyle+'">'+c.av+'</div>'+
    '<div class="si"><b>'+c.student+'</b><small>'+sub+'</small>'+caseFlagsHtml(c)+pend+'</div>'+
    '<div class="smeta"><span class="pill '+casePill(c.status)+'"><span class="pdot"></span>'+c.status+'</span><span style="font-size:11.5px;color:var(--muted)">Updated '+c.updated+'</span></div>'+
    '<div style="display:flex;gap:6px;align-items:center">'+actions+'</div>'+
    '</div>';
}
var _caseFilter = {fellow:'all', admin:'all'};
function caseScopeRows(scope){
  return CASES.filter(function(c){ return scope==='admin' ? true : c.fellow===CASE_FELLOW; });
}
function renderCaseList(scope){
  var host=document.getElementById(scope==='admin'?'caseListAdmin':'caseListFellow'); if(!host) return;
  var bucket=_caseFilter[scope];
  var rows=caseScopeRows(scope).filter(function(c){ return bucket==='all' ? true : caseAttention(c)===bucket; });
  host.innerHTML = rows.map(function(c){ return caseRowHtml(c,scope); }).join('');
  var cnt=document.getElementById(scope==='admin'?'aCaseCnt':'fCaseCnt'); if(cnt) cnt.textContent=rows.length;
  var empty=document.getElementById(scope==='admin'?'aCaseEmpty':'fCaseEmpty'); if(empty) empty.style.display=rows.length?'none':'';
  if(scope==='fellow'){ var b=document.getElementById('fApplicationsBadge'); if(b){ var n=caseScopeRows('fellow').filter(function(c){return caseAttention(c)==='action';}).length; b.textContent=n; b.style.display=n?'':'none'; } }
  if(scope==='admin'){ var ab=document.getElementById('aApplicationsBadge'); if(ab){ var an=caseScopeRows('admin').filter(function(c){return caseAttention(c)==='action';}).length; ab.textContent=an; ab.style.display=an?'':'none'; } }
}
function filterCaseList(scope, bucket, btn){
  _caseFilter[scope]=bucket;
  var bar=document.getElementById(scope==='admin'?'aCaseChips':'fCaseChips'); if(bar) bar.querySelectorAll('.chip').forEach(function(x){x.classList.remove('on');});
  if(btn) btn.classList.add('on');
  renderCaseList(scope);
}
function _rerenderAll(){ renderCaseList('fellow'); if(document.getElementById('caseListAdmin')) renderCaseList('admin'); }
function caseAdvance(id){ var c=caseById(id); if(!c) return; var i=CASE_STATUS.indexOf(c.status); if(i>=0&&i<3) c.status=CASE_STATUS[i+1]; else if(c.status==='Application Approved · funds awaited') c.status='Benefits Received'; _rerenderAll(); }
function caseSetStatus(id,status){ var c=caseById(id); if(!c) return; c.status=status; _rerenderAll(); }
function casePendReason(id,type){ var c=caseById(id); if(!c) return; c._pend=type; _rerenderAll(); setTimeout(function(){ var el=document.getElementById('cr_'+id); if(el) el.focus(); },0); }
function caseCancelReason(id){ var c=caseById(id); if(!c) return; delete c._pend; _rerenderAll(); }
function caseSaveReason(id){ var c=caseById(id); if(!c) return; var v=(document.getElementById('cr_'+id)||{}).value||''; if(c._pend==='hold'){ c.onHold=true; c.onHoldReason=v||'On hold'; } else { c.discarded=true; c.discardedReason=v||'Discarded'; } delete c._pend; _rerenderAll(); }
function caseResume(id){ var c=caseById(id); if(!c) return; c.onHold=false; c.onHoldReason=''; _rerenderAll(); }
function caseReopen(id){ var c=caseById(id); if(!c) return; c.discarded=false; c.discardedReason=''; _rerenderAll(); }
function caseReassign2(id){ var c=caseById(id); if(!c) return; c.fellow = c.fellow==='Rahul More'?'Sandip K':'Rahul More'; _rerenderAll(); }
```

- [ ] **Step 4: Call the renderer on load.** Find the `DOMContentLoaded` handler that calls `renderApprovalList()` (added in the approval-review cluster) and append: `if(document.getElementById('caseListFellow')) renderCaseList('fellow');`

- [ ] **Step 5: Render-verify (Fellow list).** Use the render snippet at `?role=fellow&screen=f-applications`. Read the PNG. Expect: 9 rows for Rahul, status pills, Suresh (OBC) showing an amber "On Hold — Family unreachable…" flag, Ravi Khade showing "Discarded — …", each active row with one primary action + ⋯. Then verify a filter and an action with an injected script (e.g. `loginAs('fellow');go('f-applications');filterCaseList('fellow','action',document.querySelectorAll('#fCaseChips .chip')[1]);`) — expect only Action-needed rows. Then inject `caseAdvance('CASE-2026-002')` and confirm its pill moves Ready to Submit → Under Scrutiny.

- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Unified caseload: data-driven Fellow Applications (status ladder + triage + hold/discard)"` — ask first.

---

## Task 2: Retire `#f-tasks`, fix Fellow rail + dashboard, remove dead JS

**Files:**
- Modify: `wireframe/index.html` — delete the `#f-tasks` section (lines ~3139–3220); remove the "My Cases" rail entry (lines ~2134–2136); relabel the `#f-home` "My Cases inbox" cards (lines ~2201–2216); add the `id="fApplicationsBadge"` span to the Applications rail entry (line ~2140–2142); remove dead JS (`filterMyCases`, `transitionCase`, `applyCaseState`); fix `CRUMBS`.

- [ ] **Step 1: Remove the "My Cases" rail entry.** Delete the whole `<a ... data-go="f-tasks"> … My Cases … </a>` block (lines ~2134–2136).

- [ ] **Step 2: Add the actionable badge to the Applications rail entry.** In the `<a ... data-go="f-applications">` block (line ~2140–2142), append before `</a>` a badge span: `<span class="soon" style="background:rgba(246,178,74,.2);color:var(--marigold-soft)" id="fApplicationsBadge">0</span>` (populated by `renderCaseList`).

- [ ] **Step 3: Delete the `#f-tasks` section.** Remove the entire `<!-- ===== MY CASES ... --> <section class="screen" id="f-tasks"> … </section>` (from line ~3139 comment through the section close at ~3220).

- [ ] **Step 4: Relabel the `#f-home` "My Cases inbox".** Replace the section-h + the four `.csum` cards (lines ~2201–2216) with a triage version:

```html
        <div class="section-h" style="display:flex;justify-content:space-between;align-items:flex-end;flex-wrap:wrap;gap:8px"><h3>My caseload</h3><a href="#" data-go="f-applications" style="font-size:13px;color:var(--teal-700);font-weight:600;text-decoration:none">Open Applications →</a></div>
        <div class="case-summary">
          <div class="csum" onclick="go('f-applications');setTimeout(function(){var c=document.querySelectorAll('#fCaseChips .chip')[1];if(c)filterCaseList('fellow','action',c);},150)"><div class="num"><span class="dot" style="background:#B45309"></span><span id="fhAction">0</span></div><div class="cap">Action needed</div></div>
          <div class="csum" onclick="go('f-applications');setTimeout(function(){var c=document.querySelectorAll('#fCaseChips .chip')[2];if(c)filterCaseList('fellow','awaiting',c);},150)"><div class="num"><span class="dot" style="background:#1D4ED8"></span><span id="fhAwaiting">0</span></div><div class="cap">Awaiting</div></div>
          <div class="csum" onclick="go('f-applications');setTimeout(function(){var c=document.querySelectorAll('#fCaseChips .chip')[3];if(c)filterCaseList('fellow','hold',c);},150)"><div class="num"><span class="dot" style="background:#374151"></span><span id="fhHold">0</span></div><div class="cap">On Hold</div></div>
          <div class="csum" onclick="go('f-applications');setTimeout(function(){var c=document.querySelectorAll('#fCaseChips .chip')[4];if(c)filterCaseList('fellow','done',c);},150)"><div class="num"><span class="dot" style="background:#15803D"></span><span id="fhDone">0</span></div><div class="cap">Done</div></div>
        </div>
```

Add to `renderCaseList` (in the `scope==='fellow'` branch, Task 1 Step 3) counts for these — append inside that branch:
```js
    ['action','awaiting','hold','done'].forEach(function(k){ var el=document.getElementById('fh'+k.charAt(0).toUpperCase()+k.slice(1)); if(el) el.textContent=caseScopeRows('fellow').filter(function(c){return caseAttention(c)===k;}).length; });
```

- [ ] **Step 5: Remove dead JS.** Delete the functions `filterMyCases`, `transitionCase`, `applyCaseState` (grep them). Leave `filterCases`/`filterCasesByFellow`/`filterCasesByStage`/`caseChangeState`/`caseReassign` for now (Task 4 handles Admin).

- [ ] **Step 6: Fix `CRUMBS`.** In the `CRUMBS` object remove the `'f-tasks':...` entry; ensure `'f-applications':'Applications'` stays.

- [ ] **Step 7: Render-verify.** Render `?role=fellow&screen=f-home` (triage cards show counts: Action needed 4, Awaiting 3, On Hold 1, Done 3 — verify against the data) and confirm the rail shows a single "Applications" entry with a badge = actionable count (4), and NO "My Cases". Grep the file: zero `f-tasks`, zero `My Cases`, zero `filterMyCases`.

- [ ] **Step 8 (GATED): commit.** `... commit -m "Retire My Cases screen; Fellow rail + dashboard use unified Applications triage"` — ask first.

---

## Task 3: Align Fellow `#f-student` Applications tab (`fp-apps`)

**Files:**
- Modify: `wireframe/index.html` — the `fp-apps` tab pane (per-student application cards). Read it first (`grep -n 'id="fp-apps"'`, then read the pane).

**What to change (read the current pane, then apply):**
- [ ] **Step 1:** Align each per-student application card's status wording to the ladder strings (e.g. "Under Scrutiny", "Application Approved · funds awaited", "Benefits Received", "Re-apply", "Rejected"). Keep the existing per-card actions (`updateAppStatus` dropdown, `recordDisbursement`, `addCorrectionNote`).
- [ ] **Step 2:** If any card carries a work-state control ("Mark Complete" / To Do / In-progress), remove it — the single status is authoritative.
- [ ] **Step 3:** Add an On Hold / Discard affordance consistent with the list: a small `⋯`-style ghost button on each card offering "Put on hold…" / "Discard case…" with an inline reason (reuse `.case-reason` styling; a local per-card handler is fine — this pane is per-student detail, not driven by `CASES`). If the effort is disproportionate, at minimum show any On Hold/Discarded state as a `.case-state` badge for consistency and note the action lives on the Applications list.
- [ ] **Step 4:** Render-verify: `?role=fellow&screen=f-student`, activate the Applications tab (inject `var t=document.querySelectorAll('#f-student .tabs button');ftab(t[3],'fp-apps')`), confirm wording aligned + no orphan work-state control.
- [ ] **Step 5 (GATED): commit** `-m "Fellow student-detail Applications tab aligned to unified status ladder"` — ask first.

---

## Task 4: Admin `#a-tasks` → "Applications" (cross-Fellow oversight, data-driven)

**Files:**
- Modify: `wireframe/index.html` — Admin rail entry (line ~3630); `#a-tasks` body (lines ~4150–~4290, the whole summary/filters/table); remove dead Admin case JS (`filterCases`, `filterCasesByFellow`, `filterCasesByStage`, `caseChangeState`); keep behavior via the shared engine.

- [ ] **Step 1: Relabel the Admin rail entry** (line ~3630): change the label text `Cases &amp; Tasks` → `Applications`, and give the badge `id="aApplicationsBadge"` (keep the `.soon` styling; value populated by `renderCaseList('admin')`).

- [ ] **Step 2: Replace the `#a-tasks` body.** Keep the `page-h` but change eyebrow/title/subtitle to: eyebrow `Oversight`, `<h1>Applications</h1>`, subtitle `All student × scholarship cases across your Fellows. Status is driven by the Fellows; reassign here.` Keep the `+ Create Case` button. Replace everything from the `<!-- 5-state summary cards -->` block through the `</table></div>` + `#myCaseEmpty`-style empty (lines ~4156–~4290) with:

```html
        <div class="filterbar" style="margin-bottom:14px">
          <select onchange="adminCaseFellow(this.value)" id="aCaseFellowFilter"><option value="all">All Fellows</option><option>Rahul More</option><option>Sandip K</option><option>Dhanashree O</option></select>
        </div>
        <div style="display:flex;gap:8px;flex-wrap:wrap;margin-bottom:18px" id="aCaseChips">
          <span class="tagk chip on" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('admin','all',this)">All</span>
          <span class="tagk chip" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('admin','action',this)">Action needed</span>
          <span class="tagk chip" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('admin','awaiting',this)">Awaiting</span>
          <span class="tagk chip" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('admin','hold',this)">On Hold</span>
          <span class="tagk chip" style="padding:8px 14px;cursor:pointer" onclick="filterCaseList('admin','done',this)">Done</span>
        </div>
        <div class="listbar"><span class="lbsel"></span><span class="cnt"><b id="aCaseCnt">0</b> Applications</span><div class="lb-tools"><button><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M3 5h18M6 12h12M10 19h4"/></svg>Filter</button></div></div>
        <div id="caseListAdmin"></div>
        <div class="empty" id="aCaseEmpty" style="display:none">No applications in this view.</div>
```

- [ ] **Step 3: Add the Admin Fellow filter helper** (near the engine): 
```js
function adminCaseFellow(f){ _caseAdminFellow=f; renderCaseList('admin'); }
```
and add `var _caseAdminFellow='all';` next to `_caseFilter`, and in `caseScopeRows` change the admin branch to honor it:
```js
function caseScopeRows(scope){ return CASES.filter(function(c){ return scope==='admin' ? (_caseAdminFellow==='all'||c.fellow===_caseAdminFellow) : c.fellow===CASE_FELLOW; }); }
```

- [ ] **Step 4: Call `renderCaseList('admin')` on load** — append to the same DOMContentLoaded: `if(document.getElementById('caseListAdmin')) renderCaseList('admin');`

- [ ] **Step 5: Remove dead Admin case JS** — delete `filterCases`, `filterCasesByFellow`, `filterCasesByStage`, `caseChangeState` (grep). The old `caseReassign(btn)` (row-DOM based) is superseded by `caseReassign2(id)` — delete `caseReassign` if nothing else references it (grep to confirm; the old a-tasks table is gone). Fix `CRUMBS` `'a-tasks'` label to `'Applications'`.

- [ ] **Step 6: Render-verify.** `?role=admin&screen=a-tasks` — expect the cross-Fellow list (12 rows), triage chips, Fellow dropdown; inject `adminCaseFellow('Rahul More')` → only Rahul's rows; confirm rows show the Fellow name in the subtitle and a **Reassign** action (no Submit/Record actions); status pills read-only. Confirm rail shows "Applications" with actionable badge.

- [ ] **Step 7 (GATED): commit** `-m "Admin Cases & Tasks → unified Applications oversight (shared caseload engine)"` — ask first.

---

## Task 5: Admin `#a-student` (`ap-apps`) alignment + full sweep + heartbeat

**Files:**
- Modify: `wireframe/index.html` — `ap-apps` pane (read-only per-student oversight); `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Align `ap-apps`.** Read the pane (`grep -n 'id="ap-apps"'`); align application status wording to the ladder; show any On Hold/Discarded as read-only `.case-state` badges (Admin sets nothing here — oversight). Keep it read-only (no operator buttons), consistent with the existing design.
- [ ] **Step 2: Consistency grep sweep.** Confirm: no `f-tasks` / `My Cases` / `filterMyCases` / `transitionCase` / `applyCaseState` / `filterCasesByStage` remain; exactly one caseload rail entry per role; `caseAttention` used by both scopes; no orphaned function referenced by deleted markup. Fix anything that grep surfaces.
- [ ] **Step 3: Behavior render sweep.** Re-render + read: Fellow `f-applications` (list + a filter + an advance + a hold-with-reason + a discard-with-reason), Fellow `f-home` (triage cards), Fellow `fp-apps`, Admin `a-tasks` (list + Fellow filter + Reassign), Admin `ap-apps`. All per `samavesh-render-verify`.
- [ ] **Step 4: Update the heartbeat.** Add a dated section to `wireframe/BUILD_STATUS.md` summarising the unified caseload (Fellow + Admin) as done + render-verified, and noting `#f-tasks` retired + `#a-tasks` relabelled.
- [ ] **Step 5 (GATED): commit** `-m "Heartbeat + Admin student-detail alignment: unified Applications caseload complete"` — then ask whether to **push** (separate gate).
- [ ] **Step 6 (after user OK): update memory** — revise `samavesh-cases-tasks` to the new single-status + flags model (work_state removed; attention derived), and update `samavesh-status-model` note that application status is now the single per-case axis with On Hold/Discarded as flags. Add/adjust MEMORY.md pointers as needed.

---

## Self-Review (against the spec)

- **Spec coverage:** §A status ladder → Task 1 (`CASE_STATUS`, `casePill`). §B exception flags → Task 1 (`caseHold`/`caseDiscard`/`caseResume`/`caseReopen` + inline reason). §C derived triage → Task 1 (`caseAttention`, filters) + Task 2 (dashboard). §D merged Fellow screen → Tasks 1–2. §E Fellow ripple (dashboard, fp-apps, rail) → Tasks 2–3. Admin screen → Task 4. Admin ripple (rail, ap-apps, MIS untouched) → Tasks 4–5. Data model / cleanup → Tasks 2,4,5. Verification → every task + Task 5 sweep. Decisions (Applications name, Admin included, Discarded, read-only Admin, actionable badge) → Tasks 1,2,4.
- **Placeholder scan:** engine + data + all handlers are complete code; the two per-student panes (Tasks 3,5) are "read current, align wording + add badge" edits (small, explicitly scoped) rather than transcribed here because they are per-student detail not driven by `CASES` — flagged, not silent.
- **Type/name consistency:** `renderCaseList(scope)`, `filterCaseList(scope,bucket,btn)`, `caseAttention`, `casePill`, `caseAdvance`, `caseSetStatus`, `casePendReason/caseSaveReason/caseCancelReason`, `caseResume/caseReopen`, `caseReassign2`, `caseScopeRows`, `_caseFilter`/`_caseAdminFellow` are defined in Task 1/4 and used consistently. Fellow badge `fApplicationsBadge` (Task 2) / Admin badge `aApplicationsBadge` (Task 4) match their `renderCaseList` writers.
- **Scope:** one coherent feature (one entity, two views) in one file — one plan is correct.
- **Constraint:** every commit GATED; push is a separate gate.
