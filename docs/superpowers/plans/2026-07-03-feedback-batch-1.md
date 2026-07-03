# Feedback Batch 1 (IDs 1, 3, 4, 5) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Apply the four confirmed client-tracker items (`Feedbacks.xlsx`) to the student-facing surfaces of the wireframe: email-OTP sign-up, dual WhatsApp reach-out + home reorder, a single unified scholarship list with right-side status, and student-uploadable sub-documents gated behind a "Don't have" toggle (with Fellow download).

**Architecture:** Single-file wireframe (`wireframe/index.html`, ~5031 lines) — inline `<style>` + inline `<script>` + role-app `<div>`s. All changes stay inside this file. No build step, no test runner. Verification is **manual browser click-through** (open the file, exercise the flow, confirm the expected visual/behaviour) plus the project's standing rules: `samavesh-no-half-baked` (every control must work), `samavesh-ripple-check` (update every sibling surface), `samavesh-use-only-context-terms`.

**Tech Stack:** Hand-written HTML/CSS/vanilla JS. Reuse existing tokens (`var(--teal-700)`, `.btn`, `.pill`, `.card`, `.arr-card`/`toggleArr`, `.doc-ctable .attach-btn`, `ftab`, `go`, `loginAs`).

## Global Constraints

- **Scope lock:** touch ONLY the login screen, Student Home (`#home`), Student Scholarships (`#scholarships`), Student Documents (`#documents`), and the Fellow's student-detail Documents tab (`#fp-docs`). Do NOT touch Admin, Mentor, or any other Fellow screen.
- **Vocabulary lock:** document status stays **Pending → Uploaded → Under Review → Accepted (+ N/A)**. Do not introduce "Missing/Verified" from the client screenshots.
- **Terminology lock:** use only BRD/SOP/tracker vocabulary. "IP" = Implementation Partner (the Fellow). No invented scheme names.
- **Doc-list lock:** keep every document currently on the Student Documents tab. Only ADD sub-doc capability to the 4 named docs. Do NOT add Bank Passbook / Marksheet as new main docs.
- **Every control must be functional** (no inert buttons, no `prompt()`), per `samavesh-no-half-baked`.
- **Files marked ignore-forever** (per `samavesh-ignored-files`) must not be read or referenced.
- Work stays on branch `dev`. Commit per task; do not push or PR unless the user asks.

---

### Task 1: ID 1 — Sign In / Sign Up buttons + email-OTP at account creation

**Files:**
- Modify: `wireframe/index.html` — login card `#loginCardSignin` (anchor: `<div class="login-card" id="loginCardSignin">`), the login `<style>` block, and the login JS (near `doGoogleSSO`/`doMagicLink`, ~line 4020).

**Interfaces:**
- Consumes: existing `loginAs(role)`, `doGoogleSSO()`, `doMagicLink()`.
- Produces: `showAuthMode(mode)` where `mode ∈ {'signin','signup'}`; `sendSignupOtp()`; `verifySignupOtp()`. New DOM ids: `authTabs`, `signupPane`, `otpRow`, `otpInputs`, `signupEmail`.

**Design:** Keep the existing sign-in card interface intact. Add a 2-button segmented toggle at the top of the card — **Sign in** (default) / **Sign up**. "Sign in" shows the current content (Google + magic-link + demo shortcuts). "Sign up" swaps the card body to: email field → `Send OTP` → a 6-box OTP entry (`otpRow`, hidden until OTP sent) → `Verify & create account`. On verify, flash "✓ Email verified — account created" then route via `loginAs('student')`. OTP copy must say it works for **any email provider (Gmail, Outlook, other)**.

- [ ] **Step 1: Add the auth-toggle + sign-up pane CSS**

Add to the login `<style>` region (near `.demo-shortcuts`):

```css
.auth-tabs{display:flex;gap:6px;background:var(--cream);border:1px solid var(--line);border-radius:10px;padding:4px;margin-bottom:18px}
.auth-tabs button{flex:1;padding:8px 10px;border:none;background:transparent;font-family:inherit;font-size:13.5px;font-weight:600;color:var(--ink-soft);border-radius:7px;cursor:pointer;transition:.15s}
.auth-tabs button.on{background:#fff;color:var(--teal-700);box-shadow:var(--shadow-sm)}
.otp-row{display:flex;gap:8px;justify-content:space-between;margin:12px 0}
.otp-row input{width:44px;height:52px;text-align:center;font-size:20px;font-weight:600;border:1.5px solid var(--line);border-radius:9px;font-family:inherit;outline:none;transition:.15s}
.otp-row input:focus{border-color:var(--teal-500);box-shadow:0 0 0 3px var(--teal-50)}
.otp-sent-note{font-size:12.5px;color:var(--ink-soft);margin:2px 0 4px}
.auth-ok{display:none;align-items:center;gap:8px;background:var(--green-50);color:var(--green);font-weight:600;font-size:13.5px;padding:10px 14px;border-radius:10px;margin-top:10px}
```

- [ ] **Step 2: Insert the auth toggle + sign-up pane HTML**

Immediately after `<div class="lhead">…</div>` inside `#loginCardSignin`, insert the toggle; wrap the existing Google/divider/magic-link/demo-shortcuts block in `<div id="signinPane">…</div>`; add the sign-up pane after it.

```html
<div class="auth-tabs" id="authTabs">
  <button class="on" id="tabSignin" onclick="showAuthMode('signin')">Sign in</button>
  <button id="tabSignup" onclick="showAuthMode('signup')">Sign up</button>
</div>
<!-- existing Google + or + magic-link + demo-shortcuts markup moves inside: -->
<div id="signinPane"> … existing content unchanged … </div>
<div id="signupPane" style="display:none">
  <div class="lfield">
    <label>Email</label>
    <div class="inp"><input id="signupEmail" type="email" autocomplete="email" placeholder="you@example.com" /></div>
    <div class="qhelp" style="font-size:12px;color:var(--muted);margin-top:6px">We'll email a one-time code to confirm you own this address. Works with Gmail, Outlook or any provider.</div>
  </div>
  <button class="btn-magic" onclick="sendSignupOtp()">Send OTP</button>
  <div id="otpBlock" style="display:none">
    <div class="otp-sent-note">Enter the 6-digit code sent to <b id="otpTo"></b></div>
    <div class="otp-row" id="otpRow">
      <input maxlength="1" inputmode="numeric"><input maxlength="1" inputmode="numeric"><input maxlength="1" inputmode="numeric"><input maxlength="1" inputmode="numeric"><input maxlength="1" inputmode="numeric"><input maxlength="1" inputmode="numeric">
    </div>
    <button class="btn-google" style="justify-content:center" onclick="verifySignupOtp()">Verify &amp; create account</button>
  </div>
  <div class="auth-ok" id="authOk"><svg viewBox="0 0 24 24" width="16" height="16" fill="none" stroke="currentColor" stroke-width="2"><path d="M20 6 9 17l-5-5"/></svg>Email verified — account created</div>
</div>
```

- [ ] **Step 3: Add the auth JS**

Add near `doMagicLink` (~4045):

```javascript
function showAuthMode(mode){
  var si = mode==='signin';
  document.getElementById('tabSignin').classList.toggle('on', si);
  document.getElementById('tabSignup').classList.toggle('on', !si);
  document.getElementById('signinPane').style.display = si ? '' : 'none';
  document.getElementById('signupPane').style.display = si ? 'none' : '';
}
function sendSignupOtp(){
  var e = document.getElementById('signupEmail').value.trim();
  if(!/^\S+@\S+\.\S+$/.test(e)){ alert('Enter a valid email address.'); return; }
  document.getElementById('otpTo').textContent = e;
  document.getElementById('otpBlock').style.display = '';
  var boxes = document.querySelectorAll('#otpRow input');
  boxes.forEach(function(b,i){ b.value=''; b.oninput=function(){ if(b.value && i<5) boxes[i+1].focus(); }; });
  boxes[0].focus();
}
function verifySignupOtp(){
  var code = Array.from(document.querySelectorAll('#otpRow input')).map(function(b){return b.value;}).join('');
  if(code.length<6){ alert('Enter all 6 digits (any digits work in this demo).'); return; }
  document.getElementById('authOk').style.display='flex';
  setTimeout(function(){ loginAs('student'); }, 900);
}
```

- [ ] **Step 4: Verify in browser**

Open `wireframe/index.html`. On the login card: click **Sign up** → card swaps to email + Send OTP. Enter `test@outlook.com` → Send OTP → 6 boxes appear, auto-advance on typing → enter 6 digits → Verify → green "Email verified" → lands on the student portal. Click **Sign in** → original Google/magic-link/demo card returns. Expected: no console errors; both toggles work.

- [ ] **Step 5: Commit**

```bash
git add wireframe/index.html
git commit -m "ID 1: Sign in/Sign up toggle + email-OTP account creation"
```

---

### Task 2: ID 3 — Student Home: reorder stat tiles above applications + dual WhatsApp reach-out

**Files:**
- Modify: `wireframe/index.html` — `#home` section (anchors: `<!-- My Applications` comment ~1203, `<!-- Stats -->` comment ~1276), plus a small CSS add for the WhatsApp buttons.

**Interfaces:**
- Consumes: existing `.hero`, `.grid.g-3`, `.scholar-apps`, `.btn`.
- Produces: `.wa-cta` button style; two `wa.me` deep-links.

**Design:** (a) Move the `<div class="grid g-3 stagger">…</div>` **stats block** to sit *directly under the hero and above* the `<div class="scholar-apps">` block. (b) Add a WhatsApp reach-out card with **two buttons** — "Message my Fellow (IP)" and "Samavesh helpline" — each a `wa.me` deep-link with a prefilled query. In the wireframe use demo numbers; add a comment noting the IP number is per-student in production.

- [ ] **Step 1: Add WhatsApp button CSS**

```css
.wa-cta{display:inline-flex;align-items:center;gap:8px;background:#25D366;color:#062e16;border:none;border-radius:9px;padding:11px 16px;font-family:inherit;font-size:13.5px;font-weight:700;cursor:pointer;text-decoration:none;transition:.15s}
.wa-cta:hover{filter:brightness(.96)}
.wa-cta.alt{background:#fff;color:#0b6b3a;border:1.5px solid #25D366}
.wa-cta svg{width:17px;height:17px}
```

- [ ] **Step 2: Reorder — move the stats grid above `.scholar-apps`**

Cut the `<!-- Stats -->` block (`<div class="grid g-3 stagger"> … Expected amount … </div>`) and paste it immediately after the `.hero` `</div>` and before `<!-- My Applications -->`. Leave the applications block otherwise unchanged.

- [ ] **Step 3: Insert the WhatsApp reach-out card**

Place after the stats grid (or after the "What's next" grid — choose directly under stats so it's high on the page). Use a WhatsApp glyph.

```html
<div class="card" style="margin:14px 0;border-left:4px solid #25D366">
  <h4 class="serif" style="font-size:17px;margin:0 0 4px">Have a question? Reach out on WhatsApp</h4>
  <p style="color:var(--ink-soft);font-size:13.5px;margin:0 0 12px">Message your Fellow for anything about your scholarships or documents, or the Samavesh helpline for general help.</p>
  <div style="display:flex;gap:10px;flex-wrap:wrap">
    <!-- Production: IP number is the student's assigned Fellow (dynamic). -->
    <a class="wa-cta" href="https://wa.me/919800000011?text=Hi%20Rahul%2C%20I%20have%20a%20question%20about%20my%20scholarship" target="_blank" rel="noopener">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 2a10 10 0 0 0-8.6 15l-1.4 5 5.1-1.3A10 10 0 1 0 12 2Zm5.3 14.2c-.2.6-1.3 1.2-1.8 1.2-.5.1-1 .1-1.7-.1-.4-.1-.9-.3-1.6-.6-2.8-1.2-4.6-4-4.7-4.2-.1-.2-1.1-1.5-1.1-2.8 0-1.3.7-2 .9-2.2.2-.3.5-.3.7-.3h.5c.2 0 .4 0 .6.5.2.5.7 1.8.8 1.9.1.1.1.3 0 .5-.3.6-.6.8-.8 1-.1.2-.3.3-.1.6.2.3.8 1.3 1.7 2.1 1.2 1 2.1 1.4 2.4 1.5.3.1.5.1.6-.1.2-.2.7-.8.9-1.1.2-.3.4-.2.6-.1.2.1 1.5.7 1.7.8.2.1.4.2.4.3.1.1.1.6-.1 1.2Z"/></svg>
      Message my Fellow (IP)
    </a>
    <a class="wa-cta alt" href="https://wa.me/919800000000?text=Hi%20Samavesh%2C%20I%20need%20some%20help" target="_blank" rel="noopener">
      <svg viewBox="0 0 24 24" fill="currentColor"><path d="M12 2a10 10 0 0 0-8.6 15l-1.4 5 5.1-1.3A10 10 0 1 0 12 2Zm5.3 14.2c-.2.6-1.3 1.2-1.8 1.2-.5.1-1 .1-1.7-.1-.4-.1-.9-.3-1.6-.6-2.8-1.2-4.6-4-4.7-4.2-.1-.2-1.1-1.5-1.1-2.8 0-1.3.7-2 .9-2.2.2-.3.5-.3.7-.3h.5c.2 0 .4 0 .6.5.2.5.7 1.8.8 1.9.1.1.1.3 0 .5-.3.6-.6.8-.8 1-.1.2-.3.3-.1.6.2.3.8 1.3 1.7 2.1 1.2 1 2.1 1.4 2.4 1.5.3.1.5.1.6-.1.2-.2.7-.8.9-1.1.2-.3.4-.2.6-.1.2.1 1.5.7 1.7.8.2.1.4.2.4.3.1.1.1.6-.1 1.2Z"/></svg>
      Samavesh helpline
    </a>
  </div>
</div>
```

- [ ] **Step 4: Verify in browser**

Log in as student. Home now shows, in order: hero → 3 stat tiles → WhatsApp card → My Scholarship Applications. Both WhatsApp buttons open `wa.me/…` in a new tab with prefilled text. Expected: order correct, both links live.

- [ ] **Step 5: Commit**

```bash
git add wireframe/index.html
git commit -m "ID 3: Student Home — stat tiles above applications + dual WhatsApp reach-out"
```

---

### Task 3: ID 4 — Student Scholarships: one unified list, status on the right of each card

**Files:**
- Modify: `wireframe/index.html` — `#scholarships` section (~1312–1441), plus a CSS add for the 2-column scholarship card.

**Interfaces:**
- Consumes: existing `.sch-card`, `.pill`, `.tl` timeline.
- Produces: `.sch2` (2-column card: `.sch2-main` left, `.sch2-side` right) style.

**Design:** Replace the three sections ("My Active Applications" / "Ready to submit" / "Also Eligible") with **one** section titled **"My Scholarships"** listing **all** eligible schemes. Each card is 2-column: **left** = logo + name + dept + amount + application ID + (optional) short timeline; **right side** = status pill + stage/step + one-line next-step. Eligible-but-not-applied schemes stay in the list with a **grey "Eligible — Not Applied"** pill on the right and criteria on the left. Keep the existing `data-app` ids (`pm/ms/ab/smm`) and `studentSelectApp` deep-link targets intact.

- [ ] **Step 1: Add the 2-column card CSS**

```css
.sch2{display:grid;grid-template-columns:1fr 232px;gap:0;align-items:stretch}
.sch2-main{padding:16px 18px;min-width:0}
.sch2-side{border-left:1px solid var(--line);background:var(--cream);padding:16px 16px;display:flex;flex-direction:column;gap:8px;justify-content:center}
.sch2-side .s-stage{font-size:12px;color:var(--ink-soft)}
.sch2-side .s-step{font-size:12px;font-weight:600;color:var(--ink)}
.sch2-side .s-next{font-size:12px;color:var(--ink-soft);border-top:1px dashed var(--line);padding-top:8px;margin-top:2px}
@media(max-width:640px){.sch2{grid-template-columns:1fr}.sch2-side{border-left:none;border-top:1px solid var(--line)}}
```

- [ ] **Step 2: Rewrite the `#scholarships` body to a single list**

Replace everything between the `.page-h` and the section close with one `<div class="section-h"><h3>My Scholarships</h3><span>…all schemes you qualify for…</span></div>` followed by one card per scheme using `.sch2`. Convert the 4 applied schemes (PM/MS/AB/SMM) into `.sch2` cards (keep their `data-app`, logo, name, amount, timeline on the left; move the status pill + stage/step + next-step into `.sch2-side`). Convert the 2 eligible-not-applied schemes (GP/EBC) into `.sch2` cards with left = name + amount + `.elig-criteria`, right = grey `Eligible — Not Applied` pill. Example shape for one applied card:

```html
<div class="card sch2" data-app="pm" style="margin-bottom:14px;padding:0;scroll-margin-top:100px">
  <div class="sch2-main">
    <div class="sch-card" style="padding:0">
      <div class="logo" style="background:linear-gradient(140deg,#0f857a,#0a3f3a)">PM</div>
      <div class="body">
        <h4>Post-Matric Scholarship for SC Students</h4>
        <div style="font-size:13px;color:var(--muted)">Govt. of Maharashtra · NSP</div>
        <div class="meta"><span><b class="amount">₹50,000</b>/year</span><span>ID: MH-PM-2026-44871</span></div>
      </div>
    </div>
  </div>
  <div class="sch2-side">
    <span class="pill blue" style="align-self:flex-start"><span class="pdot"></span>Under Scrutiny</span>
    <div class="s-stage">Stage 4 of 5 · Submitted</div>
    <div class="s-step">Dept. reviewing since 2 Jun</div>
    <div class="s-next">Next: await portal decision</div>
  </div>
</div>
```

And one eligible-not-applied card:

```html
<div class="card sch2" style="margin-bottom:14px;padding:0">
  <div class="sch2-main">
    <div class="sch-card" style="padding:0">
      <div class="logo" style="background:linear-gradient(140deg,#0a3f3a,#0e6e66)">GP</div>
      <div class="body">
        <h4>Government of India Post-Matric Scholarship</h4>
        <div style="font-size:13px;color:var(--muted)">Ministry of Social Justice · Central</div>
        <div class="meta"><span><b class="amount">₹48,000</b>/year</span></div>
        <div class="elig-criteria">You qualify: <b>SC</b> · <b>UG enrolled</b> · <b>Income &lt; ₹2.5L</b></div>
      </div>
    </div>
  </div>
  <div class="sch2-side">
    <span class="pill grey" style="align-self:flex-start"><span class="pdot"></span>Eligible — Not Applied</span>
    <div class="s-next">Your Fellow can apply on your behalf</div>
  </div>
</div>
```

- [ ] **Step 3: Update the page-h copy**

Change the `.page-h` `<p>` to reflect one list, e.g. "Every scholarship you qualify for, with its current status. Your Fellow applies and submits on your behalf." Remove references to the deleted sub-sections.

- [ ] **Step 4: Verify in browser + deep-link**

As student → Scholarships: one list, 6 cards, each with status on the right; the 2 not-applied ones show the grey pill. From Home, click a `.scholar-row` (e.g. MS) → still scrolls to and outlines the matching `data-app` card (`studentSelectApp` intact). Resize narrow → side column stacks below (mobile fallback only; web is primary). Expected: no orphaned "Ready to submit"/"Also Eligible" headings remain.

- [ ] **Step 5: Commit**

```bash
git add wireframe/index.html
git commit -m "ID 4: Student Scholarships — single unified list with right-side status"
```

---

### Task 4: ID 5a — Student Documents: "Don't have" gating + sub-document dropdown (attach)

**Files:**
- Modify: `wireframe/index.html` — `#documents` section (4 target docs: Income `arr3` ~1470, Caste flat ~1524, Domicile `arr2` ~1566, Ration flat ~1600), a CSS add, and a JS add (near `toggleArr` ~4205).

**Interfaces:**
- Consumes: `.card`, `.pill`, `.attach-btn` styling.
- Produces (used by Task 5): global `SUBDOCS` object keyed `ration|caste|domicile|income`, each an array of sub-doc label strings; `renderSubdocList(mainKey, mode)` returning row HTML where `mode ∈ {'student','fellow'}`; `toggleDontHave(mainKey, btn)`; `attachSubdoc(btn)`.

**Design:** Give the 4 target docs a **"Don't have" toggle**. When ON, an unlocked **Sub-documents** dropdown appears listing that doc's sub-docs (from `SUBDOCS`) with per-row `Attach → filename + Replace/Delete`, an "X / N attached" counter, and the blue helper banner. When OFF (student has the doc), the dropdown is locked/hidden. Demo seed: **Ration Card** and **Caste Certificate** start as "Don't have" (Ration with 1 attached to mirror the "1/6" screenshot, Caste with 0); **Income** (Accepted) and **Domicile** (Under Review) keep their status with the toggle available but OFF.

- [ ] **Step 1: Define the SUBDOCS data + render/attach JS**

Add near `toggleArr` (~4205). This is the authoritative sub-doc content (verbatim from client screenshots):

```javascript
var SUBDOCS = {
  ration: ["Aadhaar Card (Student and Father)","Electricity Bill","Income Certificate","Salary Slip / Employer Certificate","Passport-size Photographs of all family members","Aadhaar Card of all Family Members"],
  caste: ["1950 Identity Proof SC (Grandfather/mother) — Birth or School Leaving Cert.","1960 Identity Proof ST (Grandfather/mother) — Birth or School Leaving Cert.","1970 Identity Proof OBC (Grandfather/mother) — Birth or School Leaving Cert.","Aadhaar Card (Student and Father)","Ration Card","Electricity / Water Bill","Birth Certificate (Student and Father)","School Leaving Certificate (Student and Father)","Passport-size Photograph (Student and Father)"],
  domicile: ["Ration Card","Electricity Bill / Water Bill (last 10 years)","Aadhaar Card (Student and Father)","Birth Certificate / School Leaving Cert. (Student and Father)","Affidavit (self-declaration of residence duration)","Passport-size Photograph (Student and Father)","Parent's Domicile Certificate (if minor)"],
  income: ["Aadhaar Card (Student and Father)","Ration Card","Electricity / Water Bill","Rent Agreement","Salary Slip (for salaried individuals)","Employer Certificate (if employed)","Affidavit / Self Declaration stating income details","Bank Passbook Copy","Passport-size Photograph (Student and Father)","Income Tax Return (ITR or Form 16)","Agricultural Income Proof (if farmer)"]
};
var SUBDOC_LABEL = { ration:"Ration Card", caste:"Caste Certificate", domicile:"Domicile Certificate", income:"Income Certificate" };

// preAttached: indices (per mainKey) already uploaded, for demo realism
var SUBDOC_PRESET = { ration:[4], caste:[], domicile:[], income:[] };

function renderSubdocList(mainKey, mode){
  var items = SUBDOCS[mainKey] || [];
  var pre = SUBDOC_PRESET[mainKey] || [];
  var rows = items.map(function(label,i){
    var attached = pre.indexOf(i)>-1;
    if(mode==='fellow'){
      return '<div class="subdoc-row"><span class="subdoc-label">'+label+'</span>'+
        (attached
          ? '<a class="subdoc-dl" href="#" onclick="event.preventDefault();alert(\'Download '+label.replace(/'/g,"")+' (demo)\')"><svg viewBox="0 0 24 24" width="14" height="14" fill="none" stroke="currentColor" stroke-width="2"><path d="M12 3v12m0 0 4-4m-4 4-4-4M4 21h16"/></svg>Download</a>'
          : '<span class="subdoc-none">Not attached</span>')+'</div>';
    }
    return '<div class="subdoc-row">'+
      '<span class="subdoc-label">'+label+'</span>'+
      (attached
        ? '<span class="subdoc-done"><b>'+label.split(" ")[0].toLowerCase()+'.pdf</b> <button class="subdoc-mini" onclick="attachSubdoc(this)">Replace</button><button class="subdoc-mini danger" onclick="removeSubdoc(this)">Delete</button></span>'
        : '<button class="attach-btn" onclick="attachSubdoc(this)"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 8v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/><path d="M14 3v5h5M12 11v6M9 14h6"/></svg>Attach</button>')+
      '</div>';
  }).join('');
  var n = items.length, got = pre.length;
  var banner = '<div class="subdoc-banner"><b>Sub-documents required to procure '+SUBDOC_LABEL[mainKey]+'</b><span>Attach each sub-document you have. Your Fellow will use these to apply for the main document on your behalf.</span></div>';
  var counter = '<div class="subdoc-count">'+got+' / '+n+' sub-docs attached</div>';
  return banner + counter + rows;
}
function toggleDontHave(mainKey, btn){
  var host = btn.closest('.subdoc-host');
  var panel = host.querySelector('.subdoc-panel');
  var on = btn.classList.toggle('on');
  btn.textContent = on ? "Don't have ✓" : "Don't have";
  if(on && !panel.dataset.filled){ panel.innerHTML = renderSubdocList(mainKey,'student'); panel.dataset.filled='1'; }
  panel.style.display = on ? 'block' : 'none';
}
function attachSubdoc(btn){
  var row = btn.closest('.subdoc-row');
  row.querySelector('.subdoc-label').insertAdjacentHTML('afterend','');
  btn.outerHTML = '<span class="subdoc-done"><b>attachment.pdf</b> <button class="subdoc-mini" onclick="attachSubdoc(this)">Replace</button><button class="subdoc-mini danger" onclick="removeSubdoc(this)">Delete</button></span>';
  bumpSubdocCount(row, +1);
}
function removeSubdoc(btn){
  var row = btn.closest('.subdoc-row');
  var done = btn.closest('.subdoc-done');
  done.outerHTML = '<button class="attach-btn" onclick="attachSubdoc(this)"><svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="2"><path d="M21 8v11a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V5a2 2 0 0 1 2-2h11"/><path d="M14 3v5h5M12 11v6M9 14h6"/></svg>Attach</button>';
  bumpSubdocCount(row, -1);
}
function bumpSubdocCount(row, delta){
  var panel = row.closest('.subdoc-panel');
  var c = panel.querySelector('.subdoc-count');
  var m = c.textContent.match(/(\d+)\s*\/\s*(\d+)/);
  if(m){ var got=Math.max(0,parseInt(m[1])+delta); c.textContent = got+' / '+m[2]+' sub-docs attached'; }
}
```

- [ ] **Step 2: Add the sub-doc CSS**

```css
.subdoc-host{margin-top:10px}
.dont-have-btn{font-family:inherit;font-size:12px;font-weight:600;color:var(--ink-soft);background:#fff;border:1.5px solid var(--line);border-radius:20px;padding:5px 12px;cursor:pointer;transition:.15s}
.dont-have-btn:hover{border-color:var(--amber);color:var(--amber)}
.dont-have-btn.on{background:var(--amber-50);border-color:var(--amber);color:#9a5b00}
.subdoc-panel{display:none;margin-top:10px;border:1px solid var(--line);border-radius:11px;overflow:hidden}
.subdoc-banner{background:var(--blue-50);padding:11px 14px;border-bottom:1px solid var(--line)}
.subdoc-banner b{display:block;color:var(--blue);font-size:13px}
.subdoc-banner span{font-size:12px;color:var(--ink-soft)}
.subdoc-count{font-size:11.5px;font-weight:600;color:var(--ink-soft);padding:8px 14px;background:var(--cream);border-bottom:1px solid var(--line)}
.subdoc-row{display:flex;align-items:center;justify-content:space-between;gap:12px;padding:10px 14px;border-bottom:1px solid var(--line)}
.subdoc-row:last-child{border-bottom:none}
.subdoc-label{font-size:13px;color:var(--ink)}
.subdoc-done{display:inline-flex;align-items:center;gap:8px;font-size:12px;color:var(--teal-700)}
.subdoc-mini{font-family:inherit;font-size:11px;border:1px solid var(--line);background:#fff;border-radius:6px;padding:3px 8px;cursor:pointer}
.subdoc-mini.danger{color:var(--red);border-color:#f2c0c0}
.subdoc-dl{display:inline-flex;align-items:center;gap:5px;font-size:12px;color:var(--teal-700);font-weight:600;text-decoration:none}
.subdoc-none{font-size:12px;color:var(--muted)}
```

- [ ] **Step 3: Wire the 4 target docs**

For each of Ration, Caste, Domicile, Income in `#documents`, append a `.subdoc-host` after the doc's status row containing the "Don't have" button + empty `.subdoc-panel`. For flat `.doc-row` cards (Ration, Caste), add it inside the card after the `.doc-row`. For `arr-card` cards (Income `arr3`, Domicile `arr2`), add it inside `.arr-body`. Example (Ration, pre-seeded ON):

```html
<div class="subdoc-host">
  <button class="dont-have-btn on" onclick="toggleDontHave('ration',this)">Don't have ✓</button>
  <div class="subdoc-panel" style="display:block"></div>
</div>
<script>document.currentScript.previousElementSibling.querySelector('.subdoc-panel').innerHTML = renderSubdocList('ration','student');document.currentScript.previousElementSibling.querySelector('.subdoc-panel').dataset.filled='1';</script>
```

For Caste use the same but seeded ON with empty attachments; for Income/Domicile use `<button class="dont-have-btn" onclick="toggleDontHave('income',this)">Don't have</button>` with an empty hidden panel (no inline script; fills on first toggle).

> Note: inline `<script>` seeding is acceptable in this wireframe. Alternatively seed all four in a single DOMReady pass at the end of the script block — pick whichever the surrounding code favors; prefer a single init function `initSubdocs()` called on load if inline scripts feel fragile.

- [ ] **Step 4: Verify in browser**

As student → Documents. Ration Card shows "Don't have ✓" with an open panel: blue banner, "1 / 6 sub-docs attached", one row showing `<file>.pdf` + Replace/Delete, others showing Attach. Click an Attach → becomes filename + counter increments. Click Delete → reverts + counter decrements. On Income (Accepted): "Don't have" is OFF, no panel; click it → panel unlocks and lists 11 income sub-docs. Toggle off → panel hides. Expected: all 4 docs behave; no console errors.

- [ ] **Step 5: Commit**

```bash
git add wireframe/index.html
git commit -m "ID 5a: Student Documents — Don't-have gating + sub-document attach dropdown"
```

---

### Task 5: ID 5b — Fellow view: student-attached sub-documents visible + downloadable

**Files:**
- Modify: `wireframe/index.html` — `#fp-docs` tabpane inside `#f-student` (~2550–2620). Reuses `SUBDOCS`/`renderSubdocList` from Task 4.

**Interfaces:**
- Consumes: `SUBDOCS`, `renderSubdocList(mainKey,'fellow')`, `SUBDOC_PRESET` (from Task 4).
- Produces: nothing new (read-only Fellow surface).

**Design:** In the Fellow's Documents tab, under the same 4 docs (Ration/Caste/Domicile/Income rows), show a **collapsible "Sub-documents from student" list** rendered in `'fellow'` mode — each attached sub-doc has a **Download** link; unattached ones read "Not attached". This is the download/view side the client asked for; Fellow does not attach here.

- [ ] **Step 1: Add a sub-doc block under each of the 4 Fellow doc rows**

For the 4 target `.doc-row`s in `#fp-docs`, append a `.subdoc-host` (always visible, no "Don't have" toggle — Fellow just views). Render on load in `'fellow'` mode. Example markup:

```html
<div class="subdoc-host" style="margin:0 0 6px 58px">
  <details class="subdoc-fellow"><summary>Sub-documents from student</summary>
    <div class="subdoc-panel" data-subkey="ration" style="display:block"></div>
  </details>
</div>
```

Add a light style + one init pass:

```css
.subdoc-fellow summary{font-size:12.5px;font-weight:600;color:var(--teal-700);cursor:pointer;padding:6px 0}
```

```javascript
function initFellowSubdocs(){
  document.querySelectorAll('#fp-docs .subdoc-panel[data-subkey]').forEach(function(p){
    p.innerHTML = renderSubdocList(p.dataset.subkey,'fellow');
  });
}
```

Call `initFellowSubdocs()` when the student-detail Documents tab is shown (inside `ftab` when target is `fp-docs`, or once on load).

- [ ] **Step 2: Verify in browser**

Log in as Fellow → My Students → open a student → Documents tab. Under Ration Card, expand "Sub-documents from student" → shows the 6 rows; the pre-attached one (index 4, "Passport-size Photographs of all family members") shows a **Download** link, the rest show "Not attached". Click Download → demo alert. Confirm Caste/Domicile/Income also list their sub-docs in fellow mode. Expected: counts/labels match the student side exactly (same `SUBDOCS` source).

- [ ] **Step 3: Commit**

```bash
git add wireframe/index.html
git commit -m "ID 5b: Fellow student-detail — view + download student-attached sub-documents"
```

---

## Self-Review

**Spec coverage:**
- ID 1 (email OTP at account creation) → Task 1 ✓ (Sign up → email → OTP → verify, any provider).
- ID 3 (WhatsApp + reorder) → Task 2 ✓ (two buttons IP + helpline; stats moved above applications).
- ID 4 (one list, status on right, keep not-applied) → Task 3 ✓.
- ID 5 (sub-docs gated by "Don't have"; keep vocab; keep doc list; Fellow download) → Tasks 4 + 5 ✓.
- ID 2 (role buttons) → intentionally untouched (dev-time removal) ✓.

**Placeholder scan:** All sub-doc content is spelled out verbatim in `SUBDOCS`; all CSS/JS/HTML shown. No "TBD"/"add handling".

**Type consistency:** `SUBDOCS` keys `ration|caste|domicile|income` used identically in Task 4 (`renderSubdocList`, `toggleDontHave`, `SUBDOC_PRESET`) and Task 5 (`data-subkey`, `initFellowSubdocs`). `renderSubdocList(mainKey, mode)` signature consistent across both tasks. `attachSubdoc`/`removeSubdoc`/`bumpSubdocCount` defined and referenced consistently.

**Ripple check (per `samavesh-ripple-check`):** ID 5 spans BOTH student (`#documents`) and Fellow (`#fp-docs`) — both covered. ID 4's `data-app` deep-link from Home (`studentSelectApp`) preserved. No Admin `a-student` doc surface is in scope this round (documented as out-of-scope).

## Open follow-ups (not in this batch, note for later)
- Admin `a-student` Documents tab does not get the sub-doc view this round (only Student + Fellow were requested). Flag if the client wants Admin parity.
- Production WhatsApp IP number must resolve to the student's assigned Fellow (dynamic); helpline is a constant.
