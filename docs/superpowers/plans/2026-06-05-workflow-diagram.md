# Role Workflow Diagram Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build a client-facing, branded `wireframe/workflow.html` — an algorithmic flowchart (in the style of the provided `Samavesh workflow.pdf`) showing a Program Overview + the four role journeys (Student / Fellow / Program Admin / Mentor), with screen-nodes that deep-link into the live wireframe.

**Architecture:** A self-contained HTML page reusing the wireframe's brand tokens/fonts, with a small set of `.wf-*` flowchart CSS classes (color-coded nodes: green action / yellow screen / blue decision / pink note / teal title; vertical lanes + split branches + connector arrows). Screen-nodes are `<a href="index.html?role=&screen=" target="_blank">`; a 6-line handler added to `index.html` reads those params on load and auto-logs-in to that role/screen via the existing `loginAs`/`go`.

**Tech Stack:** Static HTML/CSS, no JS framework, no build, no test runner. Verification = Grep structural checks + manual browser. Frequent commits to `dev`.

**Reference spec:** `docs/superpowers/specs/2026-06-05-workflow-diagram-design.md`

**Verification convention (no test runner):** each task's "failing check" is a Grep confirming the target is absent; the "passing check" confirms it's present. Task 6 is the manual browser acceptance gate. Commit after each task. All work on branch `dev` — do not create/switch branches. Windows + PowerShell (pwsh 7, `&&` works).

---

### Task 1: Add the deep-link handler to index.html

**Files:**
- Modify: `wireframe/index.html` — append a 6-line IIFE at the very end of the `<script>`.

- [ ] **Step 1: Failing check** — Grep `wireframe/index.html` for `deep-link from workflow` → expect 0 matches.

- [ ] **Step 2: Insert the handler**

Find this exact block at the end of the file:

```html
</script>
</body>
</html>
```

Replace it with:

```html
// ---- deep-link from workflow.html (?role=&screen=) ----
(function(){
  const p=new URLSearchParams(location.search);
  const role=p.get('role'); const screen=p.get('screen');
  if(role && ({student:1,fellow:1,admin:1,mentor:1})[role]){ loginAs(role); if(screen) go(screen); }
})();
</script>
</body>
</html>
```

This runs after `loginAs`/`showApp`/`go` are defined and the DOM exists (it's the last thing in the script). `loginAs(role)` shows the role's app and hides the login overlay; the trailing `go(screen)` overrides the default home screen. With no params, nothing runs and the login screen shows as before.

- [ ] **Step 3: Passing checks** — Grep `deep-link from workflow` → 1; `URLSearchParams` → 1; confirm exactly one `</script>` immediately followed by `</body>`.

- [ ] **Step 4: Commit**

```bash
cd "C:\Users\Jitesh (DHW-L06)\Desktop\Samavesh"
git add wireframe/index.html
git commit -m "index.html: deep-link handler (?role=&screen=) for workflow page

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 2: Create workflow.html skeleton (head + CSS + legend + section anchors + footer)

**Files:**
- Create: `wireframe/workflow.html`.

- [ ] **Step 1: Failing check** — confirm the file does not exist: Grep for `id="wf-overview"` across `wireframe/` → 0 matches.

- [ ] **Step 2: Find the font links to copy**

In `wireframe/index.html` `<head>`, Grep for `fonts.googleapis` to locate the `<link>` font tags (Mukta + Fraunces). Copy those exact `<link ...>` lines — you'll paste them into the new file's `<head>` in Step 3 (shown as `<!-- PASTE font <link> tags from index.html here -->`). If none are found, omit them (the CSS has `system-ui` fallbacks).

- [ ] **Step 3: Write the file**

Create `wireframe/workflow.html` with exactly this content (paste the font links where indicated):

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<title>Samavesh — Role Workflows</title>
<!-- PASTE font <link> tags from index.html here -->
<style>
:root{
  --teal-900:#0a3f3a; --teal-700:#0e6e66; --teal-600:#0f857a; --teal-500:#14a99a;
  --teal-100:#d6efec; --teal-50:#eef8f6; --teal-200:#bce4de; --teal-400:#3cbcae; --teal-800:#0c534c;
  --marigold:#e8821e; --marigold-soft:#f6b24a; --marigold-50:#fdf2e2;
  --cream:#faf6ef; --cream-2:#f3ece0; --paper:#ffffff;
  --ink:#1b2b29; --ink-soft:#4d605d; --ink-faint:#869693;
  --muted:#7c8c89;
  --line:#e7ddcf;
  --green:#2f9e63; --green-50:#e7f4ec;
  --blue:#2f6fd6; --blue-50:#e7eefb;
  --red:#d2553f; --red-50:#fbeae6;
  --amber:#d99413; --amber-50:#fbf1de;
}
*{box-sizing:border-box}
body{margin:0;font-family:'Mukta',system-ui,sans-serif;color:var(--ink);background:var(--cream);
  background-image:radial-gradient(1200px 600px at 110% -10%,var(--teal-50),transparent 60%),radial-gradient(900px 500px at -10% 110%,var(--marigold-50),transparent 55%);
  background-attachment:fixed;padding:34px 22px 90px}
h1,h2{font-family:'Fraunces',Georgia,serif;font-weight:600;letter-spacing:-.01em}
.wf-wrap{max-width:1200px;margin:0 auto}
.wf-head{display:flex;align-items:center;gap:16px;margin-bottom:6px}
.wf-head .mark{width:46px;height:46px;border-radius:13px;background:linear-gradient(140deg,var(--marigold),var(--marigold-soft));display:grid;place-items:center;color:#3a2400;font-family:'Fraunces';font-weight:700;font-size:24px}
.wf-head h1{font-size:30px;margin:0}
.wf-head p{margin:3px 0 0;color:var(--ink-soft);font-size:14px}
.wf-legend{display:flex;flex-wrap:wrap;gap:16px;margin:18px 0 12px;padding:14px 18px;background:rgba(255,255,255,.72);border:1px solid var(--line);border-radius:14px}
.wf-legend .lg{display:flex;align-items:center;gap:8px;font-size:12.5px;color:var(--ink-soft);font-weight:600}
.wf-legend .sw{width:22px;height:15px;border-radius:5px;border:1.6px solid}
.wf-legend .sw.a{background:var(--green-50);border-color:var(--green)}
.wf-legend .sw.s{background:var(--marigold-50);border-color:var(--marigold)}
.wf-legend .sw.d{background:var(--blue-50);border-color:var(--blue)}
.wf-legend .sw.n{background:#fdeaf0;border-color:#dd6c97;border-style:dashed}
.wf-legend .sw.t{background:var(--teal-700);border-color:var(--teal-900)}
.wf-tip{font-size:12px;color:var(--muted);margin:0 0 24px}
.wf-section{margin:26px 0;padding:22px 22px 26px;background:rgba(255,255,255,.55);border:1px solid var(--line);border-radius:18px}
.wf-section>h2{font-size:21px;margin:0 0 10px;color:var(--teal-900)}
.wf-rolebar{display:inline-flex;gap:8px;align-items:center;font-size:12.5px;color:var(--ink-soft);background:var(--teal-50);border:1px solid var(--teal-200);border-radius:30px;padding:6px 14px;margin-bottom:22px}
.wf-rolebar b{color:var(--teal-800)}
.wf-lane{display:flex;flex-direction:column;align-items:center;gap:0}
.wf-node{padding:11px 16px;border-radius:12px;font-size:13px;font-weight:600;text-align:center;max-width:250px;line-height:1.3;border:1.6px solid;text-decoration:none;display:inline-block;color:var(--ink)}
.wf-node.wf-title{background:linear-gradient(135deg,var(--teal-900),var(--teal-700));color:#fff;border-color:var(--teal-900);font-family:'Fraunces';font-size:12.5px;letter-spacing:.04em;text-transform:uppercase;border-radius:10px}
.wf-node.wf-action{background:var(--green-50);border-color:var(--green);color:#1f6b43;border-radius:30px}
.wf-node.wf-screen{background:var(--marigold-50);border-color:var(--marigold);color:#8a4b07;cursor:pointer;transition:.15s}
.wf-node.wf-screen:hover{transform:translateY(-2px);box-shadow:0 8px 18px rgba(232,130,30,.22)}
.wf-node.wf-screen::after{content:" \2197";font-size:11px;opacity:.55}
.wf-node.wf-decision{background:var(--blue-50);border-color:var(--blue);color:#1f4c95;border-radius:8px}
.wf-node.wf-decision::before{content:"\25C6  ";color:var(--blue);font-size:11px}
.wf-note{background:#fdeaf0;border:1.4px dashed #dd6c97;color:#9a3a5e;border-radius:10px;padding:9px 13px;font-size:11.5px;max-width:300px;text-align:center;margin:8px auto 0;line-height:1.35}
.wf-link{width:2px;height:24px;background:#cdd6d3;position:relative;flex:0 0 auto}
.wf-link::after{content:"";position:absolute;bottom:-1px;left:50%;transform:translateX(-50%);border:5px solid transparent;border-top-color:#cdd6d3}
.wf-split{display:flex;gap:26px;align-items:flex-start;justify-content:center;flex-wrap:wrap;margin-top:4px}
.wf-col{display:flex;flex-direction:column;align-items:center;gap:0}
.wf-blabel{font-size:10.5px;font-weight:700;letter-spacing:.06em;color:var(--muted);text-transform:uppercase;margin-bottom:8px;background:#fff;border:1px solid var(--line);border-radius:20px;padding:2px 10px}
.wf-phases{display:flex;gap:14px;align-items:flex-start;overflow-x:auto;padding-bottom:10px}
.wf-phase{flex:1 1 230px;min-width:215px;background:var(--paper);border:1px solid var(--line);border-radius:14px;padding:16px 14px}
.wf-phase>.ph-h{font-family:'Fraunces';font-size:12px;text-transform:uppercase;letter-spacing:.05em;color:var(--teal-700);border-bottom:1px solid var(--line);padding-bottom:8px;margin-bottom:14px;text-align:center}
.wf-foot{margin-top:34px;text-align:center}
.wf-foot a{display:inline-flex;align-items:center;gap:8px;background:var(--teal-700);color:#fff;text-decoration:none;font-weight:600;font-size:14px;padding:12px 24px;border-radius:30px}
.wf-foot a:hover{background:var(--teal-900)}
@media(max-width:760px){.wf-phases{flex-direction:column}.wf-split{flex-direction:column;align-items:center;gap:8px}}
@media print{body{background:#fff;background-image:none;padding:0}.wf-section{break-inside:avoid;border:none;background:none}.wf-node.wf-screen::after{content:""}}
</style>
</head>
<body>
<div class="wf-wrap">
  <div class="wf-head">
    <div class="mark">स</div>
    <div>
      <h1>Samavesh — Role Workflows</h1>
      <p>How each role moves through the platform. Click any <b>amber screen</b> to open it live in the wireframe.</p>
    </div>
  </div>

  <div class="wf-legend">
    <div class="lg"><span class="sw t"></span>Phase / section</div>
    <div class="lg"><span class="sw a"></span>Action (a step the user performs)</div>
    <div class="lg"><span class="sw s"></span>Screen (click to open it live ↗)</div>
    <div class="lg"><span class="sw d"></span>Decision ◆ (a branch)</div>
    <div class="lg"><span class="sw n"></span>Note</div>
  </div>
  <p class="wf-tip">Modelled on the Samavesh program workflow. Screen links open the prototype already signed-in as that role.</p>

  <section class="wf-section" id="wf-overview"><h2>Program Overview — the 4 phases</h2><!-- WF: overview --></section>
  <section class="wf-section" id="wf-student"><h2>1 · Student journey</h2><!-- WF: student --></section>
  <section class="wf-section" id="wf-fellow"><h2>2 · Fellow journey</h2><!-- WF: fellow --></section>
  <section class="wf-section" id="wf-admin"><h2>3 · Program Admin journey</h2><!-- WF: admin --></section>
  <section class="wf-section" id="wf-mentor"><h2>4 · Mentor journey</h2><!-- WF: mentor --></section>

  <div class="wf-foot"><a href="index.html">← Open the Samavesh wireframe</a></div>
</div>
</body>
</html>
```

- [ ] **Step 4: Passing checks** — Grep `wireframe/workflow.html` for: `id="wf-overview"` → 1; `id="wf-student"` → 1; `id="wf-fellow"` → 1; `id="wf-admin"` → 1; `id="wf-mentor"` → 1; `.wf-node.wf-screen` → 1; `<!-- WF: overview -->` → 1. Confirm the 5 section anchors and the legend (5 `.sw` swatches) are present.

- [ ] **Step 5: Commit**

```bash
git add wireframe/workflow.html
git commit -m "workflow.html: skeleton, brand CSS, legend, section anchors

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 3: Program Overview content

**Files:**
- Modify: `wireframe/workflow.html` — replace `<!-- WF: overview -->`.

- [ ] **Step 1: Failing check** — Grep `Phase 1 · Onboarding` in `wireframe/workflow.html` → 0.

- [ ] **Step 2: Replace the anchor**

Find `<!-- WF: overview -->` and replace it with:

```html
<div class="wf-phases">
    <div class="wf-phase"><div class="ph-h">Phase 1 · Onboarding</div>
      <div class="wf-lane">
        <div class="wf-node wf-action">Fellow initiates onboarding</div>
        <div class="wf-link"></div>
        <a class="wf-node wf-screen" href="index.html?role=fellow&screen=f-onboard" target="_blank">Digital Onboarding Form</a>
        <div class="wf-note">For this we require an mForm</div>
        <div class="wf-link"></div>
        <div class="wf-node wf-action">Fills form + DPDP consent</div>
        <div class="wf-link"></div>
        <div class="wf-node wf-action">Profile created</div>
      </div>
    </div>
    <div class="wf-phase"><div class="ph-h">Phase 2 · Eligibility &amp; Discovery</div>
      <div class="wf-lane">
        <div class="wf-node wf-action">Automated eligibility engine</div>
        <div class="wf-link"></div>
        <div class="wf-node wf-action">List of eligible scholarships</div>
        <div class="wf-link"></div>
        <div class="wf-node wf-action">Document checklist generated</div>
      </div>
    </div>
    <div class="wf-phase"><div class="ph-h">Phase 3 · Hands-On Support</div>
      <div class="wf-lane">
        <div class="wf-node wf-decision">Documents complete?</div>
        <div class="wf-split">
          <div class="wf-col"><span class="wf-blabel">No</span>
            <div class="wf-node wf-action">Procure &amp; upload</div>
            <div class="wf-link"></div>
            <a class="wf-node wf-screen" href="index.html?role=admin&screen=a-verify" target="_blank">Admin verifies</a>
            <div class="wf-note">Added: verification gate (implicit in the original)</div>
          </div>
          <div class="wf-col"><span class="wf-blabel">Yes</span>
            <div class="wf-node wf-action">Submit on govt portal</div>
            <div class="wf-link"></div>
            <div class="wf-node wf-action">Record application details</div>
          </div>
        </div>
      </div>
    </div>
    <div class="wf-phase"><div class="ph-h">Phase 4 · Tracking &amp; Re-application</div>
      <div class="wf-lane">
        <div class="wf-node wf-action">Logs into govt portal</div>
        <div class="wf-link"></div>
        <div class="wf-node wf-decision">Application status?</div>
        <div class="wf-split">
          <div class="wf-col"><span class="wf-blabel">Re-apply</span>
            <div class="wf-node wf-action">Correction loop → re-submit</div>
          </div>
          <div class="wf-col"><span class="wf-blabel">Approved</span>
            <div class="wf-node wf-decision">Funds disbursed?</div>
            <div class="wf-link"></div>
            <div class="wf-node wf-action">Record disbursement</div>
          </div>
        </div>
        <div class="wf-link"></div>
        <div class="wf-node wf-action">MIS dashboard auto-updates</div>
        <div class="wf-note">Vocabulary reconciled to the build: Under Review → Under Scrutiny; Redirected for Correction → Re-apply</div>
      </div>
    </div>
  </div>
  <div class="wf-note" style="max-width:none;margin-top:18px">Parallel — Mentoring Journey: once the Profile is created, the student self-books a mentor → mentoring sessions → Learning (Frappe LMS, planned).</div>
```

- [ ] **Step 3: Passing check** — Grep `Phase 1 · Onboarding` → 1; `Documents complete?` → 1; `Vocabulary reconciled` → 1; `<!-- WF: overview -->` → 0 (gone).

- [ ] **Step 4: Commit**

```bash
git add wireframe/workflow.html
git commit -m "workflow.html: Program Overview (4 phases + mentoring parallel)

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 4: Student + Mentor role flows

**Files:**
- Modify: `wireframe/workflow.html` — replace `<!-- WF: student -->` and `<!-- WF: mentor -->`.

- [ ] **Step 1: Failing check** — Grep `RBAC-04` in `wireframe/workflow.html` → 0; `RBAC-03` → 0.

- [ ] **Step 2: Replace the Student anchor**

Find `<!-- WF: student -->` and replace it with:

```html
<div class="wf-rolebar"><b>Web Portal</b> · demo login <b>9800000021</b> · the student <b>watches</b> the journey</div>
  <div class="wf-lane">
    <div class="wf-node wf-action">Logs in as Student</div>
    <div class="wf-link"></div>
    <a class="wf-node wf-screen" href="index.html?role=student&screen=home" target="_blank">Home — journey overview</a>
    <div class="wf-link"></div>
    <span class="wf-blabel">Explore from Home</span>
    <div class="wf-split">
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=student&screen=scholarships" target="_blank">Scholarships</a><div class="wf-link"></div><div class="wf-node wf-action">View matched schemes &amp; application status</div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=student&screen=documents" target="_blank">Documents</a><div class="wf-link"></div><div class="wf-node wf-action">View status &amp; history (view-only)</div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=student&screen=mentorship" target="_blank">Mentorship</a><div class="wf-link"></div><div class="wf-node wf-decision">Slot available?</div><div class="wf-link"></div><div class="wf-node wf-action">Book a session</div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=student&screen=learning" target="_blank">Learning</a><div class="wf-note">Planned — Frappe LMS</div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=student&screen=profile" target="_blank">Profile</a><div class="wf-note">Read-only · managed by Fellow</div></div>
    </div>
    <div class="wf-note" style="max-width:none">Student sees status only — the Fellow performs every action (RBAC-04).</div>
  </div>
```

- [ ] **Step 3: Replace the Mentor anchor**

Find `<!-- WF: mentor -->` and replace it with:

```html
<div class="wf-rolebar"><b>Web Portal</b> · demo login <b>9800000031</b> · a <b>parallel</b> lane (Dr. Meera Joshi)</div>
  <div class="wf-lane">
    <div class="wf-node wf-action">Logs in as Mentor</div>
    <div class="wf-link"></div>
    <a class="wf-node wf-screen" href="index.html?role=mentor&screen=m-home" target="_blank">Home — next session &amp; stats</a>
    <div class="wf-link"></div>
    <div class="wf-split">
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=mentor&screen=m-availability" target="_blank">My Availability</a><div class="wf-link"></div><div class="wf-node wf-action">Publish recurring &amp; one-off slots</div><div class="wf-link"></div><div class="wf-node wf-decision">Profile Approved?</div><div class="wf-link"></div><div class="wf-node wf-action">Slots visible to students</div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=mentor&screen=m-sessions" target="_blank">Sessions</a><div class="wf-link"></div><div class="wf-node wf-action">Join · Mark complete · + Mentor note</div><div class="wf-link"></div><div class="wf-node wf-decision">Cancel &gt; T-4h?</div><div class="wf-split"><div class="wf-col"><span class="wf-blabel">Yes</span><div class="wf-node wf-action">Penalty-free</div></div><div class="wf-col"><span class="wf-blabel">No</span><div class="wf-node wf-action">Recorded no-show</div></div></div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=mentor&screen=m-profile" target="_blank">Profile</a><div class="wf-link"></div><div class="wf-node wf-action">Bio · tags · mode · hour cap</div></div>
    </div>
    <div class="wf-note" style="max-width:none">Mentor sees mentee first name + topic only — no PII, no student feedback (RBAC-03).</div>
  </div>
```

- [ ] **Step 4: Passing checks** — Grep `RBAC-04` → 1; `RBAC-03` → 1; `screen=documents` → 1; `screen=m-availability` → 1; `<!-- WF: student -->` → 0; `<!-- WF: mentor -->` → 0.

- [ ] **Step 5: Commit**

```bash
git add wireframe/workflow.html
git commit -m "workflow.html: Student + Mentor role flows

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 5: Fellow + Program Admin role flows

**Files:**
- Modify: `wireframe/workflow.html` — replace `<!-- WF: fellow -->` and `<!-- WF: admin -->`.

- [ ] **Step 1: Failing check** — Grep `Duplicate exists?` in `wireframe/workflow.html` → 0; `RBAC-05` → 0.

- [ ] **Step 2: Replace the Fellow anchor**

Find `<!-- WF: fellow -->` and replace it with:

```html
<div class="wf-rolebar"><b>Frappe Desk</b> · demo login <b>9800000011</b> · the Fellow <b>runs the engine</b> (all 4 phases)</div>
  <div class="wf-lane">
    <div class="wf-node wf-action">Logs in as Fellow</div>
    <div class="wf-link"></div>
    <a class="wf-node wf-screen" href="index.html?role=fellow&screen=f-home" target="_blank">Dashboard — caseload &amp; tasks</a>
    <div class="wf-link"></div>
    <a class="wf-node wf-screen" href="index.html?role=fellow&screen=f-students" target="_blank">My Students</a>
    <div class="wf-link"></div>
    <div class="wf-node wf-decision">Duplicate exists?</div>
    <div class="wf-split">
      <div class="wf-col"><span class="wf-blabel">Yes</span><div class="wf-node wf-action">Open existing profile</div></div>
      <div class="wf-col"><span class="wf-blabel">No</span><a class="wf-node wf-screen" href="index.html?role=fellow&screen=f-onboard" target="_blank">Onboarding Form (mForm)</a><div class="wf-link"></div><div class="wf-node wf-action">Capture DPDP consent → profile created</div></div>
    </div>
    <div class="wf-link"></div>
    <a class="wf-node wf-screen" href="index.html?role=fellow&screen=f-student" target="_blank">Student Detail</a>
    <div class="wf-link"></div>
    <div class="wf-node wf-decision">Documents complete?</div>
    <div class="wf-split">
      <div class="wf-col"><span class="wf-blabel">No</span><div class="wf-node wf-action">Procure &amp; upload</div><div class="wf-note">Program Admin verifies → loop until all Accepted</div></div>
      <div class="wf-col"><span class="wf-blabel">Yes</span><a class="wf-node wf-screen" href="index.html?role=fellow&screen=f-scholarship" target="_blank">Scholarship Data Entry</a><div class="wf-link"></div><div class="wf-node wf-action">Submit on portal &amp; record</div></div>
    </div>
    <div class="wf-link"></div>
    <div class="wf-node wf-decision">Application status?</div>
    <div class="wf-split">
      <div class="wf-col"><span class="wf-blabel">Re-apply</span><a class="wf-node wf-screen" href="index.html?role=fellow&screen=f-docapp" target="_blank">Documentation Application</a><div class="wf-link"></div><div class="wf-node wf-action">Correct &amp; re-submit (loop)</div></div>
      <div class="wf-col"><span class="wf-blabel">Approved</span><div class="wf-node wf-action">Record disbursement → Benefits Received</div></div>
    </div>
    <div class="wf-link"></div>
    <a class="wf-node wf-screen" href="index.html?role=fellow&screen=f-applications" target="_blank">Applications — track all</a>
    <div class="wf-note" style="max-width:none">The Fellow performs every operator action; the student only sees the resulting status.</div>
  </div>
```

- [ ] **Step 3: Replace the Admin anchor**

Find `<!-- WF: admin -->` and replace it with:

```html
<div class="wf-rolebar"><b>Frappe Desk</b> · demo login <b>9800000001</b> · the Admin <b>gates &amp; oversees</b> (Meera Nair)</div>
  <div class="wf-lane">
    <div class="wf-node wf-action">Logs in as Program Admin</div>
    <div class="wf-link"></div>
    <a class="wf-node wf-screen" href="index.html?role=admin&screen=a-home" target="_blank">MIS Dashboard — KPIs / funnel / export</a>
    <div class="wf-link"></div>
    <span class="wf-blabel">Oversee &amp; gate</span>
    <div class="wf-split">
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=admin&screen=a-verify" target="_blank">Document Verification</a><div class="wf-link"></div><div class="wf-node wf-decision">Document valid?</div><div class="wf-split"><div class="wf-col"><span class="wf-blabel">Accept</span><div class="wf-node wf-action">Unlocks "Ready to Submit"</div></div><div class="wf-col"><span class="wf-blabel">Reject</span><div class="wf-node wf-action">Back to Fellow (re-upload)</div></div></div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=admin&screen=a-students" target="_blank">All Students</a><div class="wf-link"></div><a class="wf-node wf-screen" href="index.html?role=admin&screen=a-student" target="_blank">Student Detail</a><div class="wf-link"></div><div class="wf-node wf-action">Edit profile · Reassign Fellow</div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=admin&screen=a-catalogue" target="_blank">Scholarship Catalogue</a><div class="wf-link"></div><div class="wf-node wf-action">Add / edit schemes → feeds eligibility</div></div>
      <div class="wf-col"><a class="wf-node wf-screen" href="index.html?role=admin&screen=a-fellows" target="_blank">Fellows &amp; Users</a><div class="wf-link"></div><div class="wf-node wf-action">Manage users</div></div>
    </div>
    <div class="wf-note" style="max-width:none">Admin gates verification &amp; approval and oversees all Fellows (RBAC-05).</div>
  </div>
```

- [ ] **Step 4: Passing checks** — Grep `Duplicate exists?` → 1; `RBAC-05` → 1; `screen=f-scholarship` → 1; `screen=a-verify` → 2 (one in Program Overview from Task 3, one here); `<!-- WF: fellow -->` → 0; `<!-- WF: admin -->` → 0.

- [ ] **Step 5: Commit**

```bash
git add wireframe/workflow.html
git commit -m "workflow.html: Fellow + Program Admin role flows

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

---

### Task 6: Manual browser acceptance + heartbeat/memory update

**Files:**
- Modify: `wireframe/BUILD_STATUS.md`.
- Modify: `C:\Users\Jitesh (DHW-L06)\.claude\projects\C--Users-Jitesh--DHW-L06--Desktop-Samavesh\memory\samavesh_wireframe_scope.md`.

- [ ] **Step 1: Open `wireframe/workflow.html`** in a browser (or the live Pages URL after pushing).

- [ ] **Step 2: Acceptance checklist** (the real gate):
1. Page renders: title header, legend with 5 node types, **Program Overview** (4 phase columns + a mentoring-parallel note), and 4 role sections (Student / Fellow / Program Admin / Mentor) each with a role bar.
2. Node coding matches the PDF: green action pills, amber screen rectangles (with ↗), blue ◆ decision nodes, pink dashed notes, teal phase headers.
3. Decisions show labeled branches (Documents complete? Yes/No; Application status? Re-apply/Approved; Document valid? Accept/Reject; Cancel > T-4h? Yes/No; Slot available?; Profile Approved?; Duplicate exists?; Funds disbursed?).
4. Click one screen node per role and confirm it opens the wireframe **already signed-in** on the right screen:
   - Student → Documents (`index.html?role=student&screen=documents`)
   - Fellow → Student Detail (`...role=fellow&screen=f-student`)
   - Admin → Document Verification (`...role=admin&screen=a-verify`)
   - Mentor → My Availability (`...role=mentor&screen=m-availability`)
5. Pink annotations present: the two Program-Overview reconciliation notes (verification gate added; vocabulary reconciled), mForm + LMS notes, and the per-role RBAC notes (RBAC-04 Student, RBAC-05 Admin, RBAC-03 Mentor).
6. The footer "← Open the Samavesh wireframe" link opens `index.html` at the normal login screen.
7. Print preview shows the whole diagram without clipping; narrow viewport wraps the phase strip and split branches vertically.
8. **No regression:** opening `index.html` with NO query string still shows the login screen; the four demo logins still work.

- [ ] **Step 3: Update the heartbeat** — in `wireframe/BUILD_STATUS.md`:
  - Update the `_Last updated_` line to `2026-06-05 — added client workflow diagram (workflow.html).`
  - Add a new section before `## Verification audit (2026-06-05) — done`:

```markdown
## Workflow diagram (NEW) — client-facing
- [x] `wireframe/workflow.html` — branded algorithmic flowchart in the style of Samavesh workflow.pdf
- [x] Program Overview (4 phases + mentoring parallel; verification gate added, status vocab reconciled — both pink-annotated)
- [x] 4 role flows (Student/Fellow/Program Admin/Mentor) mapped to built screens, with per-role RBAC notes
- [x] Screen-nodes deep-link into the live wireframe via index.html?role=&screen= (6-line handler in index.html)
- Spec+plan: docs/superpowers/*2026-06-05-workflow-diagram*.
```

- [ ] **Step 4: Update the scope memory** — in `memory/samavesh_wireframe_scope.md`, append to the "Built so far" area a line:
  `**Client workflow diagram BUILT 2026-06-05:** standalone branded `wireframe/workflow.html` (PDF-style flowchart) — Program Overview (4 phases) + Student/Fellow/Admin/Mentor flows; amber screen-nodes deep-link into the live wireframe (index.html?role=&screen=, handled by a 6-line on-load reader in index.html). Spec+plan in docs/superpowers/*2026-06-05-workflow-diagram*.`

- [ ] **Step 5: Commit the heartbeat**

```bash
git add wireframe/BUILD_STATUS.md
git commit -m "Heartbeat: client workflow diagram (workflow.html) complete

Co-Authored-By: Claude Opus 4.8 <noreply@anthropic.com>"
```

(The memory file lives outside the repo and is saved by the Write tool, not committed.)

---

## Self-Review (completed by plan author)

**Spec coverage:**
- Deep-link handler in index.html → Task 1. ✓
- Self-contained workflow.html with copied brand tokens/fonts + flowchart CSS + legend → Task 2. ✓
- Node taxonomy (green action / amber screen / blue decision / pink note / teal title) → Task 2 CSS + used throughout. ✓
- Program Overview (4 phases, mentoring parallel, 2 reconciliation annotations) → Task 3. ✓
- Student flow (5 screens, book decision, RBAC-04 note) → Task 4. ✓
- Mentor flow (availability/sessions/profile, approval + cancel decisions, RBAC-03 note) → Task 4. ✓
- Fellow flow (dashboard→students→duplicate?→onboard→detail→docs?→data-entry→status?→docapp/disburse→applications) → Task 5. ✓
- Admin flow (MIS→verify(valid?)/all-students→detail/catalogue/users, RBAC-05 note) → Task 5. ✓
- Deep-link targets use exact screen ids from the spec table → Tasks 3-5 hrefs. ✓
- Manual verification + heartbeat/memory → Task 6. ✓

**Placeholder scan:** No TBD/TODO. All CSS and markup shown in full. The `<!-- WF: * -->` and `<!-- PASTE font <link> tags ... -->` are intentional, explicitly-resolved anchors (Task 2 says copy the exact font lines from index.html; Tasks 3-5 replace the section anchors).

**Type/name consistency:** CSS classes (`.wf-node`, `.wf-title`, `.wf-action`, `.wf-screen`, `.wf-decision`, `.wf-note`, `.wf-link`, `.wf-split`, `.wf-col`, `.wf-blabel`, `.wf-phases`, `.wf-phase`, `.ph-h`, `.wf-rolebar`, `.wf-lane`, `.wf-legend .sw.a/.s/.d/.n/.t`) are all defined in Task 2 and used verbatim in Tasks 3-5. Section anchors created in Task 2 (`<!-- WF: overview/student/fellow/admin/mentor -->`) match exactly the find-targets in Tasks 3-5. Deep-link role values (student/fellow/admin/mentor) match the `ACCOUNTS`/`showApp` roles and the Task-1 handler's allow-list; screen ids match the spec's target table and the real `index.html` screen ids. The Task-5 grep note correctly anticipates `screen=a-verify` appearing twice (Task 3 overview + Task 5 admin).
