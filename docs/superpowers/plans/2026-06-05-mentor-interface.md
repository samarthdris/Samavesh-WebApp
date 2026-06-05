# Mentor Interface Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add the **Mentor** role app to the wireframe — a `.fp` Web Portal surface (Home / My Availability / Sessions / My Profile) for the self-service facilitator journey in BRD §12, demo identity Dr. Meera Joshi (`9800000031`).

**Architecture:** New `#mentorApp` + `#mentorBot` portal shell mirroring the Student app, wired into the existing `ACCOUNTS`/`showApp`/`loginAs` role system. Wiring lands first so the app is reachable, then each screen is filled (each independently visible in-browser). Interaction helpers live in the same task as the screen that uses them. The RBAC-03 privacy boundary (no income/Aadhaar, no student feedback shown) is baked into the markup.

**Tech Stack:** Single-file static HTML/CSS/vanilla-JS prototype (`wireframe/index.html`). No build, no test framework. Frequent commits to `dev`.

**Reference spec:** `docs/superpowers/specs/2026-06-05-mentor-interface-design.md`

**Verification convention (no test runner):** each task's "failing check" is a Grep confirming the target is absent; the "passing check" is a Grep confirming it's present. Task 6 is the manual browser acceptance gate. Commit after each task. All work stays on branch `dev` — do not create/switch branches.

---

### Task 1: Mentor shell + bottom-nav + stub screens + JS wiring (makes the app reachable)

**Files:**
- Modify: `wireframe/index.html` — insert a new app shell before the Fellow app; add a `loginAs('mentor')` demo button; extend `ACCOUNTS` and `showApp`.

- [ ] **Step 1: Confirm nothing exists yet (failing checks)**

Grep the file (counts):
- `id="mentorApp"` → 0
- `loginAs\('mentor'\)` → 0
- `'9800000031'` → 0

- [ ] **Step 2: Insert the mentor shell + bottom-nav before the Fellow app**

Find this exact block (the Fellow app divider):

```html
<!-- ===================================================================== -->
<!-- ============================ FELLOW APP ============================== -->
<!-- ===================================================================== -->
<div class="app hidden fd" id="fellowApp">
```

Replace it with the mentor block PLUS the same Fellow divider (i.e. prepend the mentor markup, keep the Fellow lines intact):

```html
<!-- ===================================================================== -->
<!-- ============================ MENTOR APP ============================= -->
<!-- ===================================================================== -->
<div class="app hidden fp" id="mentorApp">
  <aside class="rail">
    <div class="brand"><div class="mark">स</div><div><div class="name">Samavesh</div><div class="tag">Mentor</div></div></div>
    <nav class="nav">
      <div class="nav-label">Mentoring</div>
      <a href="#" class="nav-link active" data-go="m-home"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M3 10.5 12 3l9 7.5"/><path d="M5 9.5V21h14V9.5"/></svg>Home</a>
      <a href="#" class="nav-link" data-go="m-availability"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="3" y="4.5" width="18" height="16" rx="2"/><path d="M3 9h18M8 3v3M16 3v3"/><path d="M12 12.5V15l1.6 1"/></svg>My Availability</a>
      <a href="#" class="nav-link" data-go="m-sessions"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M15 10l5-3v10l-5-3v2a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h8a2 2 0 0 1 2 2Z"/></svg>Sessions</a>
      <div class="nav-label">Account</div>
      <a href="#" class="nav-link" data-go="m-profile"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="12" cy="8" r="3.6"/><path d="M4.5 20a7.5 7.5 0 0 1 15 0"/></svg>My Profile</a>
      <a href="#" class="nav-link" onclick="logout();return false"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M9 21H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h4"/><path d="m16 17 5-5-5-5M21 12H9"/></svg>Sign out</a>
    </nav>
    <div class="rail-foot"><div class="av" style="background:linear-gradient(140deg,#7a5cd0,#5538a8)">M</div><div class="who">Dr. Meera Joshi<small>Career Counsellor</small></div></div>
  </aside>
  <div class="main">
    <header class="topbar">
      <div class="mobile-brand"><div class="mark">स</div><b class="serif" style="font-size:18px">Samavesh</b></div>
      <div class="search"><svg viewBox="0 0 24 24" width="18" height="18" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="11" cy="11" r="7"/><path d="m20 20-3.2-3.2"/></svg><input placeholder="Search sessions, mentees…" /></div>
      <div class="spacer"></div>
      <div class="lang">🌐 English</div>
      <button class="icon-btn" title="Notifications"><span class="dot"></span><svg viewBox="0 0 24 24" width="20" height="20" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M18 8a6 6 0 0 0-12 0c0 7-3 9-3 9h18s-3-2-3-9"/><path d="M13.7 21a2 2 0 0 1-3.4 0"/></svg></button>
    </header>
    <main class="content">
      <section class="screen" id="m-home"><div class="page-h"><div class="eyebrow">Mentoring</div><h1>Welcome, Dr. Meera Joshi</h1><p>Your mentoring home — upcoming sessions and this month at a glance.</p></div><!-- m-home body --></section>
      <section class="screen" id="m-availability"><div class="page-h"><div class="eyebrow">Mentoring · Availability</div><h1>My Availability</h1><p>Publish the times you're open to mentor. Students can book only future, un-booked slots once your profile is Approved.</p></div><!-- m-availability body --></section>
      <section class="screen" id="m-sessions"><div class="page-h"><div class="eyebrow">Mentoring · Sessions</div><h1>Sessions</h1><p>Your booked sessions. You see only the mentee's first name and their discussion topic — no personal or financial details.</p></div><!-- m-sessions body --></section>
      <section class="screen" id="m-profile"><div class="page-h"><div class="eyebrow">Account</div><h1>My Mentor Profile</h1><p>Your public mentor card. Published after Program Admin approval.</p></div><!-- m-profile body --></section>
    </main>
  </div>
</div>

<nav class="botnav hidden" id="mentorBot">
  <a href="#" class="nav-link active" data-go="m-home"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M3 10.5 12 3l9 7.5"/><path d="M5 9.5V21h14V9.5"/></svg>Home</a>
  <a href="#" class="nav-link" data-go="m-availability"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="3" y="4.5" width="18" height="16" rx="2"/><path d="M3 9h18M8 3v3M16 3v3"/></svg>Slots</a>
  <a href="#" class="nav-link" data-go="m-sessions"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M15 10l5-3v10l-5-3v2a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h8a2 2 0 0 1 2 2Z"/></svg>Sessions</a>
  <a href="#" class="nav-link" data-go="m-profile"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="12" cy="8" r="3.6"/><path d="M4.5 20a7.5 7.5 0 0 1 15 0"/></svg>Profile</a>
</nav>

<!-- ===================================================================== -->
<!-- ============================ FELLOW APP ============================== -->
<!-- ===================================================================== -->
<div class="app hidden fd" id="fellowApp">
```

- [ ] **Step 3: Add the Mentor demo login button**

Find this exact line (the Program Admin demo button):

```html
          <button onclick="loginAs('admin')"><span class="da" style="background:linear-gradient(140deg,#2f6fd6,#1c4ea8)">M</span><span class="dt">Program Admin<small>Meera Nair</small></span></button>
```

Replace it with itself + the mentor button:

```html
          <button onclick="loginAs('admin')"><span class="da" style="background:linear-gradient(140deg,#2f6fd6,#1c4ea8)">M</span><span class="dt">Program Admin<small>Meera Nair</small></span></button>
          <button onclick="loginAs('mentor')"><span class="da" style="background:linear-gradient(140deg,#7a5cd0,#5538a8)">M</span><span class="dt">Mentor<small>Dr. Meera Joshi</small></span></button>
```

- [ ] **Step 4: Add the Mentor account to `ACCOUNTS`**

Find:

```javascript
const ACCOUNTS = { '9800000021':{role:'student'}, '9800000011':{role:'fellow'}, '9800000001':{role:'admin'} };
```

Replace with:

```javascript
const ACCOUNTS = { '9800000021':{role:'student'}, '9800000011':{role:'fellow'}, '9800000001':{role:'admin'}, '9800000031':{role:'mentor'} };
```

- [ ] **Step 5: Register the mentor app in `showApp`**

Find:

```javascript
function showApp(role){
  [['student','studentApp','studentBot'],['fellow','fellowApp','fellowBot'],['admin','adminApp','adminBot']].forEach(function(t){
    document.getElementById(t[1]).classList.toggle('hidden',role!==t[0]);
    document.getElementById(t[2]).classList.toggle('hidden',role!==t[0]);
  });
  go(role==='fellow'?'f-home':role==='admin'?'a-home':'home');
}
```

Replace with:

```javascript
function showApp(role){
  [['student','studentApp','studentBot'],['fellow','fellowApp','fellowBot'],['admin','adminApp','adminBot'],['mentor','mentorApp','mentorBot']].forEach(function(t){
    document.getElementById(t[1]).classList.toggle('hidden',role!==t[0]);
    document.getElementById(t[2]).classList.toggle('hidden',role!==t[0]);
  });
  go(role==='fellow'?'f-home':role==='admin'?'a-home':role==='mentor'?'m-home':'home');
}
```

- [ ] **Step 6: Verify (passing checks)**

Grep (counts): `id="mentorApp"` → 1; `id="mentorBot"` → 1; `id="m-home"` → 1; `id="m-availability"` → 1; `id="m-sessions"` → 1; `id="m-profile"` → 1; `loginAs\('mentor'\)` → 1; `'9800000031':\{role:'mentor'\}` → 1; `\['mentor','mentorApp','mentorBot'\]` → 1; `role==='mentor'\?'m-home'` → 1. Also confirm `id="fellowApp"` still → 1 (Fellow app intact).

- [ ] **Step 7: Commit**

```bash
cd "C:\Users\Jitesh (DHW-L06)\Desktop\Samavesh"
git add wireframe/index.html
git commit -m "Add Mentor portal app shell + login wiring (reachable, stub screens)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 2: Fill m-home (next-session banner + stats + quick links)

**Files:**
- Modify: `wireframe/index.html` — replace the `<!-- m-home body -->` placeholder.

- [ ] **Step 1: Failing check** — Grep `Next session` → 0 matches.

- [ ] **Step 2: Replace the placeholder**

Find `<!-- m-home body -->` and replace it with:

```html
<div class="card" style="margin-bottom:20px;background:linear-gradient(135deg,var(--teal-700),var(--teal-900));color:#eafaf7;border:none">
          <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:12px">
            <div>
              <span class="pill" style="background:rgba(255,255,255,.16);color:#fff"><span class="pdot"></span>Next session</span>
              <h3 style="color:#fff;font-size:19px;margin:10px 0 4px">Aarti P. · "Choosing between B.Com and BBA"</h3>
              <div style="color:#cfeae6;font-size:14px">Friday, 6 June · 4:00 PM · 45 min · Video call</div>
            </div>
            <button class="btn btn-gold btn-sm">Join (link soon)</button>
          </div>
        </div>
        <div class="grid stagger" style="grid-template-columns:repeat(auto-fit,minmax(150px,1fr));gap:18px;margin-bottom:20px">
          <div class="card stat hoverable"><div class="top"><div class="ic teal"><svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.8"><path d="M15 10l5-3v10l-5-3v2a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V8a2 2 0 0 1 2-2h8a2 2 0 0 1 2 2Z"/></svg></div></div><div class="num">6</div><div class="cap">Sessions this month</div></div>
          <div class="card stat hoverable"><div class="top"><div class="ic gold"><svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.8"><circle cx="12" cy="12" r="9"/><path d="M12 7v5l3 2"/></svg></div></div><div class="num">4.5 / 8 h</div><div class="cap">Hours used / monthly cap</div></div>
          <div class="card stat hoverable"><div class="top"><div class="ic green"><svg viewBox="0 0 24 24" width="22" height="22" fill="none" stroke="currentColor" stroke-width="1.8"><path d="m12 3 2.6 5.3 5.9.9-4.2 4.1 1 5.8L12 16.9 6.7 19.2l1-5.8L3.5 9.2l5.9-.9Z"/></svg></div></div><div class="num">4.9</div><div class="cap">My average rating</div></div>
        </div>
        <div class="card">
          <p style="font-size:13px;color:var(--ink-soft);margin-bottom:6px">Student feedback is confidential to the Program team and is never shown to mentors — only your aggregate rating appears here.</p>
          <div style="display:flex;gap:10px;flex-wrap:wrap;margin-top:6px">
            <button class="btn btn-primary btn-sm" data-go="m-availability">Manage availability</button>
            <button class="btn btn-ghost btn-sm" data-go="m-sessions">View sessions</button>
          </div>
        </div>
```

Note: the `data-go` buttons work via the existing delegated `[data-go]` click handler (bound at load). The "My average rating" being a bare number — not the written feedback — is the privacy boundary (FR-MENT-011).

- [ ] **Step 3: Passing check** — Grep `Next session` → 1; `Hours used / monthly cap` → 1; `My average rating` → 1.

- [ ] **Step 4: Commit**

```bash
git add wireframe/index.html
git commit -m "Mentor Home: next-session banner, monthly stats, quick links

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 3: Fill m-availability (weekly grid + one-off slots) + grid CSS + helpers

**Files:**
- Modify: `wireframe/index.html` — add grid CSS before `</style>`; replace the `<!-- m-availability body -->` placeholder; add `mSlot`/`toggleOneOff`/`saveOneOff` JS helpers.

- [ ] **Step 1: Failing check** — Grep `class="availgrid"` → 0; `function mSlot` → 0.

- [ ] **Step 2: Add the grid CSS**

Find the closing `</style>` tag (there is exactly one). Insert this block immediately BEFORE `</style>`:

```css
/* ---- mentor weekly availability grid ---- */
.availgrid{display:grid;grid-template-columns:54px repeat(7,1fr);gap:7px;margin-top:6px}
.availgrid .day{text-align:center;font-size:11px;font-weight:700;color:var(--muted);text-transform:uppercase;letter-spacing:.05em}
.availgrid .time{font-size:11.5px;font-weight:700;color:var(--ink-soft);display:flex;align-items:center}
.slot-cell{border:1.5px solid var(--line);background:var(--paper);border-radius:9px;padding:8px 4px;font-size:11.5px;font-weight:600;color:var(--ink-soft);cursor:pointer;transition:.14s;text-align:center}
.slot-cell:hover{border-color:var(--teal-500)}
.slot-cell.on{background:var(--teal-50);border-color:var(--teal-500);color:var(--teal-700)}
.slot-cell.booked{background:var(--amber-50);border-color:var(--amber);color:var(--amber);cursor:not-allowed}
@media(max-width:640px){.availgrid{grid-template-columns:40px repeat(7,1fr);gap:4px}.slot-cell{padding:6px 2px;font-size:10px}.availgrid .time{font-size:10px}}
```

- [ ] **Step 3: Replace the placeholder**

Find `<!-- m-availability body -->` and replace it with:

```html
<div class="card" style="margin-bottom:18px">
          <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:10px;margin-bottom:6px">
            <h3 style="font-size:17px">Recurring weekly availability</h3>
            <span style="font-size:12.5px;color:var(--muted)">Tap a slot to open or close it · amber = already booked</span>
          </div>
          <div class="availgrid">
            <div></div>
            <div class="day">Mon</div><div class="day">Tue</div><div class="day">Wed</div><div class="day">Thu</div><div class="day">Fri</div><div class="day">Sat</div><div class="day">Sun</div>
            <div class="time">4 PM</div>
            <button class="slot-cell on" onclick="mSlot(this)">4:00</button>
            <button class="slot-cell" onclick="mSlot(this)">4:00</button>
            <button class="slot-cell on" onclick="mSlot(this)">4:00</button>
            <button class="slot-cell" onclick="mSlot(this)">4:00</button>
            <button class="slot-cell booked">Booked</button>
            <button class="slot-cell" onclick="mSlot(this)">4:00</button>
            <button class="slot-cell" onclick="mSlot(this)">4:00</button>
            <div class="time">5 PM</div>
            <button class="slot-cell" onclick="mSlot(this)">5:00</button>
            <button class="slot-cell on" onclick="mSlot(this)">5:00</button>
            <button class="slot-cell" onclick="mSlot(this)">5:00</button>
            <button class="slot-cell on" onclick="mSlot(this)">5:00</button>
            <button class="slot-cell on" onclick="mSlot(this)">5:00</button>
            <button class="slot-cell" onclick="mSlot(this)">5:00</button>
            <button class="slot-cell" onclick="mSlot(this)">5:00</button>
          </div>
        </div>
        <div class="card">
          <div style="display:flex;justify-content:space-between;align-items:center;flex-wrap:wrap;gap:10px;margin-bottom:10px">
            <h3 style="font-size:17px">One-off slots</h3>
            <button class="btn btn-gold btn-sm" onclick="toggleOneOff()">+ Add one-off slot</button>
          </div>
          <div id="oneOffForm" style="display:none;gap:10px;flex-wrap:wrap;align-items:flex-end;margin-bottom:14px">
            <div class="fg"><label>Date</label><input type="date" id="ooDate" /></div>
            <div class="fg"><label>Start time</label><input type="time" id="ooTime" /></div>
            <div class="fg"><label>Duration</label><select id="ooDur"><option>30</option><option selected>45</option><option>60</option></select></div>
            <button class="btn btn-primary btn-sm" onclick="saveOneOff()">Add slot</button>
          </div>
          <div id="oneOffList">
            <div style="display:flex;align-items:center;gap:12px;padding:10px 0;border-bottom:1px solid var(--line);font-size:13.5px"><span class="pill teal"><span class="pdot"></span>Open</span><b>Sat, 7 Jun</b><span style="color:var(--ink-soft)">10:00 · 60 min</span></div>
            <div style="display:flex;align-items:center;gap:12px;padding:10px 0;font-size:13.5px"><span class="pill amber"><span class="pdot"></span>Booked</span><b>Sun, 8 Jun</b><span style="color:var(--ink-soft)">11:00 · 45 min · Imran S.</span></div>
          </div>
        </div>
```

- [ ] **Step 4: Add the JS helpers**

Find this exact line:

```javascript
document.querySelectorAll('[data-go]').forEach(el=>el.addEventListener('click',e=>{e.preventDefault();go(el.dataset.go);}));
```

Insert the following immediately AFTER it:

```javascript

// ---- Mentor app interactions ----
function mSlot(el){ if(el.classList.contains('booked')) return; el.classList.toggle('on'); }
function toggleOneOff(){ const f=document.getElementById('oneOffForm'); f.style.display = f.style.display==='none' ? 'flex' : 'none'; }
function saveOneOff(){
  const d=document.getElementById('ooDate').value, t=document.getElementById('ooTime').value, dur=document.getElementById('ooDur').value;
  if(!d || !t){ alert('Pick a date and start time for the slot.'); return; }
  const row=document.createElement('div');
  row.style.cssText='display:flex;align-items:center;gap:12px;padding:10px 0;border-top:1px solid var(--line);font-size:13.5px';
  row.innerHTML='<span class="pill teal"><span class="pdot"></span>Open</span><b>'+d+'</b><span style="color:var(--ink-soft)">'+t+' · '+dur+' min</span>';
  document.getElementById('oneOffList').appendChild(row);
  document.getElementById('ooDate').value=''; document.getElementById('ooTime').value='';
  toggleOneOff();
}
```

- [ ] **Step 5: Passing checks** — Grep `class="availgrid"` → 1; `function mSlot` → 1; `function toggleOneOff` → 1; `function saveOneOff` → 1; `.slot-cell\{` → 1 (CSS present).

- [ ] **Step 6: Commit**

```bash
git add wireframe/index.html
git commit -m "Mentor Availability: weekly grid + one-off slots (mSlot/saveOneOff)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 4: Fill m-sessions (Upcoming/Past tabs + cards) + session helpers

**Files:**
- Modify: `wireframe/index.html` — replace the `<!-- m-sessions body -->` placeholder; add `mSession`/`mNote` helpers.

- [ ] **Step 1: Failing check** — Grep `id="ms-upcoming"` → 0; `function mSession` → 0.

- [ ] **Step 2: Replace the placeholder**

Find `<!-- m-sessions body -->` and replace it with:

```html
<div class="tabs">
          <button class="on" onclick="ftab(this,'ms-upcoming')">Upcoming</button>
          <button onclick="ftab(this,'ms-past')">Past</button>
        </div>
        <div class="tabpane on" id="ms-upcoming">
          <div class="readonly-note" style="margin-bottom:14px">
            <svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.8"><rect x="5" y="11" width="14" height="9" rx="2"/><path d="M8 11V7a4 4 0 0 1 8 0v4"/></svg>
            You see only the mentee's first name and discussion topic — no income, Aadhaar or documents (RBAC-03).
          </div>
          <div class="card" style="margin-bottom:12px">
            <div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px;align-items:center"><h4 class="serif" style="font-size:17px">Aarti P.</h4><span class="pill blue"><span class="pdot"></span>Scheduled</span></div>
            <div style="font-size:13.5px;color:var(--ink-soft);margin:6px 0 4px">Topic: "Choosing between B.Com and BBA"</div>
            <div style="font-size:13px;color:var(--muted)">Fri, 6 Jun · 4:00 PM · 45 min · Video call</div>
            <div class="sess-acts" style="margin-top:14px;display:flex;gap:10px;flex-wrap:wrap">
              <button class="btn btn-gold btn-sm">Join (link soon)</button>
              <button class="btn btn-primary btn-sm" onclick="mSession(this,'done')">Mark complete</button>
              <button class="btn btn-ghost btn-sm" onclick="mSession(this,'cancel')">Cancel / Reschedule</button>
              <button class="btn btn-ghost btn-sm" onclick="mNote(this)">+ Mentor note</button>
            </div>
            <div style="font-size:12px;color:var(--muted);margin-top:8px">Penalty-free up to 4 hours before start; later cancellations and no-shows are recorded (BR-MENT-07).</div>
          </div>
          <div class="card">
            <div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px;align-items:center"><h4 class="serif" style="font-size:17px">Imran S.</h4><span class="pill blue"><span class="pdot"></span>Scheduled</span></div>
            <div style="font-size:13.5px;color:var(--ink-soft);margin:6px 0 4px">Topic: "Engineering entrance prep + scholarships"</div>
            <div style="font-size:13px;color:var(--muted)">Sun, 8 Jun · 11:00 AM · 45 min · Video call</div>
            <div class="sess-acts" style="margin-top:14px;display:flex;gap:10px;flex-wrap:wrap">
              <button class="btn btn-gold btn-sm">Join (link soon)</button>
              <button class="btn btn-primary btn-sm" onclick="mSession(this,'done')">Mark complete</button>
              <button class="btn btn-ghost btn-sm" onclick="mSession(this,'cancel')">Cancel / Reschedule</button>
              <button class="btn btn-ghost btn-sm" onclick="mNote(this)">+ Mentor note</button>
            </div>
          </div>
        </div>
        <div class="tabpane" id="ms-past">
          <div class="card" style="margin-bottom:12px">
            <div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px;align-items:center"><h4 class="serif" style="font-size:17px">Sneha P.</h4><span class="pill green"><span class="pdot"></span>Completed</span></div>
            <div style="font-size:13.5px;color:var(--ink-soft);margin:6px 0 4px">Topic: "Scholarship interview preparation"</div>
            <div style="font-size:13px;color:var(--muted)">28 May · 4:00 PM · 45 min</div>
            <div style="font-size:12.5px;color:var(--green);margin-top:8px"><svg viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="currentColor" stroke-width="2" style="vertical-align:-2px"><path d="M20 6 9 17l-5-5"/></svg> Minutes &amp; transcript dispatched</div>
          </div>
          <div class="card">
            <div style="display:flex;justify-content:space-between;flex-wrap:wrap;gap:8px;align-items:center"><h4 class="serif" style="font-size:17px">Ganesh P.</h4><span class="pill grey"><span class="pdot"></span>No-show</span></div>
            <div style="font-size:13.5px;color:var(--ink-soft);margin:6px 0 4px">Topic: "Career options after B.Sc"</div>
            <div style="font-size:13px;color:var(--muted)">22 May · 5:00 PM · recorded against student</div>
          </div>
        </div>
```

Note: reuses the existing `ftab(btn,id)` (toggles `.tabpane.on` globally — harmless since other screens are hidden). The mentee-first-name-only + topic-only content is the RBAC-03 privacy boundary; no feedback is shown on Past cards.

- [ ] **Step 3: Add the JS helpers**

Find this exact line (added in Task 3):

```javascript
  document.getElementById('ooDate').value=''; document.getElementById('ooTime').value='';
  toggleOneOff();
}
```

Insert the following immediately AFTER the closing `}` of `saveOneOff`:

```javascript
function mSession(btn, action){
  const acts=btn.closest('.sess-acts');
  if(action==='done'){
    acts.innerHTML='<span class="pill green"><span class="pdot"></span>Completed</span>';
  } else {
    const why=prompt('Reason for cancellation (recorded; penalty-free up to 4 hours before start):');
    if(why===null) return;
    acts.innerHTML='<span class="pill grey"><span class="pdot"></span>Cancelled</span>';
  }
}
function mNote(btn){
  const n=prompt('Private note for this session (visible only to you and the Program Admin):');
  if(!n) return;
  const card=btn.closest('.card');
  let note=card.querySelector('.mnote');
  if(!note){ note=document.createElement('div'); note.className='mnote'; note.style.cssText='margin-top:10px;font-size:12.5px;color:var(--ink-soft)'; card.appendChild(note); }
  note.textContent='🔒 Note saved (private): '+n;
}
```

- [ ] **Step 4: Passing checks** — Grep `id="ms-upcoming"` → 1; `id="ms-past"` → 1; `function mSession` → 1; `function mNote` → 1; `RBAC-03` → 1 (privacy note present).

- [ ] **Step 5: Commit**

```bash
git add wireframe/index.html
git commit -m "Mentor Sessions: Upcoming/Past tabs, privacy boundary, mSession/mNote

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 5: Fill m-profile (editable mentor profile)

**Files:**
- Modify: `wireframe/index.html` — replace the `<!-- m-profile body -->` placeholder.

- [ ] **Step 1: Failing check** — Grep `Monthly hour commitment` → 0.

- [ ] **Step 2: Replace the placeholder**

Find `<!-- m-profile body -->` and replace it with:

```html
<div class="card">
          <div class="prof-head">
            <div class="pa" style="background:linear-gradient(140deg,#7a5cd0,#5538a8)">M</div>
            <div style="flex:1">
              <h3 style="font-size:23px">Dr. Meera Joshi</h3>
              <div style="color:var(--ink-soft);font-size:14px">Career Counsellor · 12 yrs experience</div>
              <div style="margin-top:8px"><span class="pill green"><span class="pdot"></span>Approved · publicly listed</span></div>
            </div>
            <button class="btn btn-primary btn-sm" onclick="alert('Profile saved. Changes are re-published after Program Admin review (BR-MENT-01).')">Save profile</button>
          </div>
          <p style="font-size:12.5px;color:var(--muted);margin:12px 0 4px">Your profile is publicly listed to students only after Program Admin approval. Your monthly hour commitment is a hard cap on bookings.</p>
          <div class="form-grid">
            <div class="fg full"><label>Bio</label><input value="Career counsellor with 12 years guiding first-generation students into higher education and scholarships." /></div>
            <div class="fg"><label>Skill areas</label><input value="Career, Higher Studies" /></div>
            <div class="fg"><label>Stream / Domain</label><input value="Commerce, Humanities" /></div>
            <div class="fg"><label>Geography</label><input value="Pune, Maharashtra" /></div>
            <div class="fg"><label>Languages</label><input value="Marathi, Hindi, English" /></div>
            <div class="fg"><label>Mode</label><select><option>Volunteer</option><option selected>Facilitator</option></select></div>
            <div class="fg"><label>Monthly hour commitment</label><input type="number" value="8" /><span class="hint">Hard cap — bookings beyond this are blocked (FR-MENT-012).</span></div>
          </div>
        </div>
```

- [ ] **Step 3: Passing check** — Grep `Monthly hour commitment` → 1; `Facilitator` → 1 (mode select present).

- [ ] **Step 4: Commit**

```bash
git add wireframe/index.html
git commit -m "Mentor Profile: editable bio/tags/mode/hour-cap

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 6: Manual browser acceptance + heartbeat/memory update

**Files:**
- Modify: `wireframe/BUILD_STATUS.md`.
- Modify: `C:\Users\Jitesh (DHW-L06)\.claude\projects\C--Users-Jitesh--DHW-L06--Desktop-Samavesh\memory\samavesh_wireframe_scope.md`.

- [ ] **Step 1: Open the prototype** (open `wireframe/index.html` locally, or the live Pages URL after pushing).

- [ ] **Step 2: Acceptance checklist** (the real gate):
1. From login, click the **Mentor** demo button (or sign in with `9800000031`) → lands on `m-home` in the mentor portal shell (rail: Home / My Availability / Sessions / My Profile).
2. **Home:** next-session banner (Aarti P. + topic), 3 stat cards incl. "Hours used / monthly cap" and "My average rating" (a number only). Quick-link buttons jump to Availability/Sessions.
3. **My Availability:** tapping an open weekly slot toggles teal on/off; the **Booked** cell does not toggle; "+ Add one-off slot" reveals the form; filling date+time and "Add slot" appends a row and hides the form; missing date/time shows the alert.
4. **Sessions:** Upcoming/Past tabs switch; cards show mentee first name + topic only (no income/Aadhaar); **Mark complete** → green Completed pill; **Cancel/Reschedule** → prompts reason → grey Cancelled pill; **+ Mentor note** → prompts → "🔒 Note saved (private)" line appears; Past tab shows Completed + "Minutes dispatched" and a No-show; **no feedback content anywhere**.
5. **My Profile:** fields render incl. Mode (Facilitator selected) and Monthly hour commitment; Save shows the confirm alert.
6. Narrow the viewport → mentor **bottom-nav** appears and navigates.
7. **No regression:** Student (`9800000021`), Fellow (`9800000011`), Admin (`9800000001`) logins each open their own app; the student `#mentorship` booking screen + modal still work; sign-out returns to login.

- [ ] **Step 3: Update the heartbeat** — in `wireframe/BUILD_STATUS.md`:
  - Update the `_Last updated_` line to `2026-06-05 — added Mentor portal app.`
  - Add a new section after the Program Admin section:

```markdown
## Mentor section (NEW)
- [x] Mentor portal app (.fp) — demo login 9800000031 (Dr. Meera Joshi); rail + bottom-nav
- [x] Home (m-home) — next-session banner, monthly stats (incl. hours-used/cap), my-rating number
- [x] My Availability (m-availability) — recurring weekly grid (toggle slots) + one-off slots (add)
- [x] Sessions (m-sessions) — Upcoming/Past tabs; Join/Mark complete/Cancel/Mentor note; privacy boundary (first name + topic only, NO feedback shown)
- [x] My Profile (m-profile) — editable bio/tags/mode/monthly-hour-cap
```

  - In "Still to do / next", change the Mentor line to: `- [x] ~~Mentor role app~~ — DONE 2026-06-05 · [ ] Super Admin role app (not built yet)`

- [ ] **Step 4: Update the wireframe-scope memory** — in `memory/samavesh_wireframe_scope.md`, change `**Roles still to build:** Mentor, Super Admin.` to:
  `**Roles still to build:** Super Admin. (Mentor BUILT 2026-06-05 — .fp portal app, demo 9800000031 Dr. Meera Joshi; Home/Availability/Sessions/Profile; RBAC-03 privacy boundary: mentee first name + topic only, no feedback shown. Spec+plan in docs/superpowers/*2026-06-05-mentor-interface*.)`

- [ ] **Step 5: Commit the heartbeat**

```bash
git add wireframe/BUILD_STATUS.md
git commit -m "Heartbeat: Mentor portal app complete

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

(The memory file lives outside the repo and is saved by the Write tool, not committed.)

---

## Self-Review (completed by plan author)

**Spec coverage:**
- New `#mentorApp`/`#mentorBot` `.fp` shell + 4 screens → Task 1. ✓
- JS wiring (ACCOUNTS / showApp array + home-routing / loginAs button) → Task 1. ✓
- m-home (banner, stats incl. hour cap, rating-number-only) → Task 2. ✓
- m-availability (recurring weekly grid + one-off + grid CSS) → Task 3. ✓
- m-sessions (Upcoming/Past, Join/Mark complete/Cancel/Note, status pills, minutes-dispatched) → Task 4. ✓
- m-profile (bio/tags/mode/hour-cap, approval note) → Task 5. ✓
- Privacy boundary (first name + topic only, no income/Aadhaar, no feedback shown) → Tasks 2 (rating note) + 4 (RBAC-03 note, no feedback). ✓
- Interaction helpers mSlot/toggleOneOff/saveOneOff/mSession/mNote, ftab reuse → Tasks 3-4. ✓
- Reuse-first CSS; only new CSS = the grid → Task 3. ✓
- Manual verification + heartbeat/memory → Task 6. ✓

**Placeholder scan:** No TBD/TODO. All markup, CSS, and JS shown in full. The `<!-- m-* body -->` comments are intentional Task-1 anchors that Tasks 2-5 replace.

**Type/name consistency:** Element IDs are consistent across tasks — `oneOffForm`/`oneOffList`/`ooDate`/`ooTime`/`ooDur` (Task 3 markup ↔ Task 3 helpers); `sess-acts`/`.mnote` (Task 4 markup ↔ Task 4 helpers); panes `ms-upcoming`/`ms-past` (Task 4 markup ↔ `ftab` reuse). Screen ids `m-home`/`m-availability`/`m-sessions`/`m-profile` match between the shell nav (Task 1), the stub sections (Task 1), and the `data-go` quick links (Task 2). `showApp`'s `['mentor','mentorApp','mentorBot']` matches the shell/bottom-nav ids (Task 1). `loginAs('mentor')` resolves through the unchanged `loginAs(role)` → `showApp(role)`. CSS classes `availgrid`/`slot-cell`/`slot-cell.on`/`slot-cell.booked` match between Task 3 CSS and Task 3 markup. ✓
