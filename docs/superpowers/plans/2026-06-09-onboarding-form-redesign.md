# Onboarding Form Redesign — Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development or superpowers:executing-plans. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Convert the Fellow's Onboarding Form (`#onbForm`) from an 8-step wizard into a single-page Frappe DocType form. Per spec.

**Architecture:** New `.dt-form` class scope on `#onbForm` only — Forms 2 & 3 keep `.fstep`/`formGo` untouched. Sticky `.dt-anchors` sidebar (scroll-spy) + 8 `<section class="dt-section">` blocks + sticky `.dt-actions` footer. DOB→auto-age, mobile-verify chip, gender master dropdown, document child-table (Must Have / Good to Have), all 4+ option radio lists → dropdowns, email/name dup-check.

**Tech Stack:** Single-file static HTML/CSS/vanilla-JS prototype (`wireframe/index.html`). Frequent commits to `dev`.

**Reference spec:** `docs/superpowers/specs/2026-06-09-onboarding-form-redesign.md` (full code blocks and tokens there).

**Verification convention:** Grep-based failing/passing checks. Final task is a manual desktop-only browser acceptance (mobile out of scope per `samavesh-web-focus`).

---

### Task 1: CSS — Frappe DocType form pattern (append, don't replace)

**Files:** Modify `wireframe/index.html` — append a new `/* DocType form (onboarding redesign) */` block immediately after the existing `.fnav` rule (around line 517). Do NOT touch existing `.fstep`/`.formwrap`/`.matrix` rules.

- [ ] **Step 1: Failing checks** — Grep counts expected 0: `\.dt-form`, `\.dt-anchors`, `\.dt-section`, `\.doc-ctable`, `\.mobile-verify`, `\.dob-age`, `\.dt-actions`.

- [ ] **Step 2: Append new CSS block** — see spec §"Visual treatment" + "Section anchor sidebar" + "Mobile validate" + "DOB + auto-age" + "Documents child-table" subsections. Roughly 140 new lines:
  - `.dt-form` grid (220px anchor sidebar | 1fr body)
  - `.dt-anchors` sticky, scroll-spy `.on` state with teal left-border + teal-50 bg
  - `.dt-body` white card with shadow
  - `.dt-section` with bottom-border separators; `.dt-sec-title` (Fraunces 18px) + `.dt-sec-lead` (13px)
  - `.dt-grid` 2-col with `.q.full` for full-width fields
  - `.dt-help` and `.dt-help.note` styled hints
  - `.dt-banner` (teal) for the optional-questions notice
  - `.dt-actions` sticky bottom bar with `.dt-act-meta` + `.dt-act-btns`
  - `.dob-inline` + `.dob-age` chip (teal-tinted on valid, gray-tinted on empty)
  - `.mobile-input` + `.mobile-verify.ok/.bad` absolute-positioned chips
  - `.doc-ctable` with grouped `<tbody class="doc-group">` + `.doc-group-h` subheads with `.doc-pill.must/.good` markers + `.doc-stat` select + `.doc-att` attach button (disabled when status≠"Have it")

- [ ] **Step 3: Passing checks** — Grep counts expected ≥1 each: `\.dt-form`, `\.dt-anchors`, `\.dt-section`, `\.doc-ctable`, `\.mobile-verify`, `\.dob-age`, `\.dt-actions`, `\.doc-group-h`, `\.dob-inline`.

- [ ] **Step 4: Commit**
```bash
git add wireframe/index.html
git commit -m "Onboarding redesign (1/4): CSS — DocType form pattern + doc child-table + mobile/DOB chips"
```

---

### Task 2: HTML — rewrite `#onbForm` interior + update `.dupbox`

**Files:** Modify `wireframe/index.html` — replace lines 1711-1874 (the `.dupbox` + the entire `<div class="formwrap" id="onbForm">…</div>` block).

- [ ] **Step 1: Failing checks** — Grep counts expected 0 each:
  - `class="dt-form"`
  - `id="onbDob"`
  - `id="onbAge"`
  - `id="onbMobile"`
  - `class="dt-anchors"`
  - `class="doc-ctable"`
  - `id="sec-personal"`

- [ ] **Step 2: Replace `.dupbox` content** — change copy and search placeholder:
  - Headline: "First, check for an existing profile" (unchanged)
  - Body: "A student can be onboarded only once. Search by **email** or **full name** — duplicates are blocked (BR-01)."
  - Input placeholder: "Enter email or student's full name…"

- [ ] **Step 3: Replace the entire `<div class="formwrap" id="onbForm">…</div>` block** with new `<div class="dt-form" id="onbForm">…</div>` containing:
  - `<aside class="dt-anchors">` — 8 anchor links, each with `.anum` numeric badge: Welcome / Personal / Location / College / Education / Documents / Socio-economic / Consent. Last anchor includes optional "Aspirations" note.
  - `<div class="dt-body">` — 8 `<section class="dt-section" id="sec-{key}">` blocks (sec-support, sec-personal, sec-location, sec-college, sec-edu, sec-docs, sec-socio, sec-consent). Each section opens with `<h3 class="dt-sec-title">` (with matching `.anum` chip) and `<div class="dt-sec-lead">`. NO Continue buttons. Section bodies preserve all 42 original questions, retitled into `.dt-grid` 2-column rows per spec's "Layout — Frappe Column Break equivalents" table.
  - Special field changes (per spec):
    - **Personal section:** REMOVE old Age `<select>`. ADD new DOB question (`<input type="date" id="onbDob">` + `<span class="dob-age empty" id="onbAge">Age: —</span>` + qhelp). KEEP "Date of Form Submission" Q. Mobile question wraps the input in `<div class="mobile-input">` with `<span class="mobile-verify" id="onbMobileVerify">` placeholder; input id="onbMobile". Gender Q becomes `<select id="onbGender">` (populated by JS from `GENDER_MASTER`).
    - **College & Reference section:** "How did you hear about SAMAVESH?" → `<select>` dropdown (10 options).
    - **Education section:** unchanged (already dropdowns).
    - **Documents section:** REPLACE the entire `<div class="matrix-wrap">…</div>` with `<table class="doc-ctable">…</table>`. Two `<tbody class="doc-group">` blocks: "Must Have / Mandatory" (4 docs — Aadhar, Annual Income, 10th SLC, 12th SLC) and "Good to Have (scholarship-specific)" (5 docs — Caste, Non-Creamy Layer, Domicile, Ration, Orphan). Each row has 3 td's: `td.doc-name`, `td.doc-stat <select>`, `td.doc-att <button class="attach-btn" disabled><svg>📎</svg>Attach</button>`.
    - **Socio-economic section:** Social Background → `<select>` (10 options). Annual Family Income → `<select>` (7 options). Family Occupation → `<select>` (6 options). Drug-addiction stays radio (4 options but conceptually a "yes/no/maybe/PNS" scale). Other 2/3-option questions stay as radios.
    - **Aspirations & Consent section:** all 3 textareas unchanged. Consent typed-name input unchanged.
  - `<div class="dt-actions">` sticky footer: `.dt-act-meta` ("Section <b id="dtSecCur">1</b> of 8 · optional fields skipped") + `.dt-act-btns` (`<button class="btn btn-ghost">Save draft</button>` + `<button class="btn btn-primary" onclick="onbSubmit()">Submit Onboarding ✓</button>`).

- [ ] **Step 4: Passing checks** — Grep counts expected ≥1 each: `id="onbDob"`, `id="onbAge"`, `id="onbMobile"`, `id="onbGender"`, `id="sec-personal"`, `id="sec-docs"`, `class="dt-form"`, `class="dt-anchors"`, `class="doc-ctable"`, `doc-group-h`, `Must Have / Mandatory`, `Good to Have`.
- Old-construct removal — expected 0 each: `data-step="1"`, `data-step="8"`, `class="fstep on"`, `onclick="formGo\('onbForm'`, `name="d_aadhar"`, `name="onb_gender"`, `name="onb_caste"`, `name="onb_income"`. (Note: `formGo` JS function remains for Forms 2/3; we're only removing onboarding's calls to it.)
- Expected 1 each: `Search by <b>email</b>`, `Enter email or student's full name`.

- [ ] **Step 5: Commit**
```bash
git add wireframe/index.html
git commit -m "Onboarding redesign (2/4): HTML — single-page DocType form, doc child-table, email/name dup-check"
```

---

### Task 3: JS — DOB→age, mobile validate, gender master, anchor scroll-spy, onbSubmit

**Files:** Modify `wireframe/index.html` — append new JS functions inside the main `<script>` block, after the existing `onState` function (around line 2979). Do NOT touch existing `formGo`/`formSubmit`/`sameAddr`/`onState`/`onAppStatus` — Forms 2 & 3 still use them.

- [ ] **Step 1: Failing checks** — expected 0 each: `function onbInit`, `function onbDobChange`, `function onbMobileBlur`, `function onbDocStatus`, `function onbSubmit`, `const GENDER_MASTER`, `function onbScrollSpy`.

- [ ] **Step 2: Append new JS** — roughly 70 lines:
```js
// ===== Onboarding form (Frappe DocType pattern) =====
const GENDER_MASTER = [
  {code:'M',   en:'Male',           mr:'पुरुष'},
  {code:'F',   en:'Female',         mr:'स्त्री'},
  {code:'NB',  en:'Non-Binary',     mr:'नॉन-बायनरी'},
  {code:'O',   en:'Other',          mr:'इतर'},
  {code:'PNS', en:'Prefer not to say', mr:'सांगू इच्छित नाही'}
];
function onbInit(){
  // Populate gender dropdown from master (matches future Frappe Link → Gender DocType).
  const sel=document.getElementById('onbGender');
  if(sel && !sel.dataset.populated){
    sel.innerHTML='<option value="">Select / निवडा…</option>' +
      GENDER_MASTER.map(g=>`<option value="${g.code}">${g.en} / ${g.mr}</option>`).join('');
    sel.dataset.populated='1';
  }
  // Wire DOB change, mobile blur, document status changes, scroll-spy.
  const dob=document.getElementById('onbDob');
  if(dob && !dob.dataset.wired){ dob.addEventListener('change',onbDobChange); dob.dataset.wired='1'; }
  const mob=document.getElementById('onbMobile');
  if(mob && !mob.dataset.wired){
    mob.addEventListener('blur',onbMobileBlur);
    mob.addEventListener('input',function(){ if(document.getElementById('onbMobileVerify').classList.contains('ok')||document.getElementById('onbMobileVerify').classList.contains('bad')) onbMobileBlur(); });
    mob.dataset.wired='1';
  }
  document.querySelectorAll('#onbForm .doc-ctable td.doc-stat select').forEach(function(s){
    if(s.dataset.wired) return;
    s.addEventListener('change',onbDocStatus);
    s.dataset.wired='1';
  });
  document.querySelectorAll('#onbForm .dt-anchors a').forEach(function(a){
    if(a.dataset.wired) return;
    a.addEventListener('click',function(e){
      e.preventDefault();
      const id=a.getAttribute('href').slice(1);
      const sec=document.getElementById(id);
      if(sec) sec.scrollIntoView({behavior:'smooth',block:'start'});
    });
    a.dataset.wired='1';
  });
  // Initial scroll-spy on load + on scroll.
  if(!document._onbSpyBound){
    const main=document.querySelector('#fellowApp .content');
    (main||window).addEventListener('scroll',onbScrollSpy,{passive:true});
    document._onbSpyBound=true;
  }
  onbScrollSpy();
}
function onbDobChange(){
  const dob=document.getElementById('onbDob').value;
  const out=document.getElementById('onbAge');
  if(!dob){ out.textContent='Age: —'; out.classList.add('empty'); return; }
  const ms=Date.now()-new Date(dob).getTime();
  const years=ms/(365.25*24*3600*1000);
  if(years<0||years>120){ out.textContent='Age: invalid date'; out.classList.add('empty'); return; }
  out.textContent='Age: '+years.toFixed(1)+' years';
  out.classList.remove('empty');
}
function onbMobileBlur(){
  const v=(document.getElementById('onbMobile').value||'').replace(/\D/g,'');
  const chip=document.getElementById('onbMobileVerify');
  chip.classList.remove('ok','bad');
  if(!v) return; // empty = no chip
  const valid=/^[6-9]\d{9}$/.test(v);
  if(valid){ chip.innerHTML='<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M20 6 9 17l-5-5"/></svg>Valid'; chip.classList.add('ok'); }
  else     { chip.innerHTML='<svg viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="3"><path d="M6 6l12 12M18 6 6 18"/></svg>Invalid'; chip.classList.add('bad'); }
}
function onbDocStatus(e){
  // Enable/disable the Attach button next to this status select.
  const sel=e.target;
  const tr=sel.closest('tr');
  const btn=tr.querySelector('.doc-att .attach-btn');
  if(!btn) return;
  btn.disabled = sel.value!=='have';
}
function onbScrollSpy(){
  const sections=document.querySelectorAll('#onbForm .dt-section');
  const anchors=document.querySelectorAll('#onbForm .dt-anchors a');
  let curIdx=0;
  sections.forEach(function(s,i){
    const r=s.getBoundingClientRect();
    if(r.top<=120) curIdx=i;
  });
  anchors.forEach(function(a,i){ a.classList.toggle('on',i===curIdx); });
  const meta=document.getElementById('dtSecCur'); if(meta) meta.textContent=String(curIdx+1);
}
function onbSubmit(){
  // Validation: consent + mobile valid (if entered).
  const c=document.getElementById('onbConsent');
  if(c && !c.value.trim()){ alert('Consent is required — please type the student/guardian name to give consent (BR-02).'); document.getElementById('sec-consent').scrollIntoView({behavior:'smooth'}); return; }
  const chip=document.getElementById('onbMobileVerify');
  if(chip && chip.classList.contains('bad')){ alert('Mobile number is invalid. Please correct it before submitting.'); document.getElementById('sec-personal').scrollIntoView({behavior:'smooth'}); return; }
  alert('✅ Onboarding submitted. A unique Student ID is generated and the eligibility engine has matched scholarships automatically.');
  go('f-student');
}
// Run onbInit whenever the onboarding screen becomes active (handled in go() observer below).
document.addEventListener('DOMContentLoaded',function(){ if(document.getElementById('onbDob')) setTimeout(onbInit,0); });
```

Place this block immediately after `function onState(sel){…}` and its closing brace, before `function onAppStatus(sel){…}`.

- [ ] **Step 3: Hook onbInit into screen activation** — find the existing `go()` function (around line 2745) and ensure it calls `onbInit()` when navigating to `#f-onboard`. Cleanest: add at the bottom of `go(id)` function:
```js
  if(id==='f-onboard' && typeof onbInit==='function') setTimeout(onbInit,0);
```

- [ ] **Step 4: Passing checks** — expected 1 each: `function onbInit`, `function onbDobChange`, `function onbMobileBlur`, `function onbDocStatus`, `function onbSubmit`, `function onbScrollSpy`, `const GENDER_MASTER`. Existing `function formGo` still present (Forms 2/3 keep working).

- [ ] **Step 5: Commit**
```bash
git add wireframe/index.html
git commit -m "Onboarding redesign (3/4): JS — DOB→age, mobile validate, gender master, anchor scroll-spy, submit"
```

---

### Task 4: BUILD_STATUS heartbeat update

**Files:** Modify `wireframe/BUILD_STATUS.md` — update top date + add new section.

- [ ] **Step 1:** Change `_Last updated: 2026-06-09 — SSO login redesign (Google + magic link)._` to `_Last updated: 2026-06-09 — Onboarding form redesign (Frappe DocType pattern)._`

- [ ] **Step 2:** Insert this section immediately before the `## SSO login (2026-06-09) — done` section:
```markdown
## Onboarding form redesign (2026-06-09) — done
- [x] Single-page Frappe DocType form replaces the 8-step "Continue" wizard
- [x] Left sticky anchor sidebar with scroll-spy (8 sections preserved as anchors, not steps)
- [x] DOB picker → auto-calculated **age in years (1 decimal)** displayed inline
- [x] Mobile field validated as Indian 10-digit (regex `/^[6-9]\d{9}$/`); shows ✓ Valid / ✗ Invalid chip on blur
- [x] **Gender as master** — dropdown driven by `GENDER_MASTER` array (5 entries, codes M/F/NB/O/PNS); same array will become the future Frappe Gender DocType, reusable across all forms
- [x] **Documents child-table** — grouped Must Have / Mandatory (4 docs) and Good to Have / Scholarship-specific (5 docs); each row has Status dropdown + Attach button (enabled only when status=Have it)
- [x] All radio lists with ≥4 options converted to dropdowns (Social Background, Annual Income, Family Occupation, How heard, etc.)
- [x] Duplicate-check changed: Name+DOB-or-Mobile → **Email OR full Name** (matches SSO email-as-primary-identifier)
- [x] Sticky bottom action bar: Save draft (secondary) + Submit Onboarding (primary)
- [x] Forms 2 (Scholarship Data Entry) and 3 (Documentation Application) untouched — they keep the `.fstep`/`formGo` wizard for now
- Spec+plan: `docs/superpowers/specs/2026-06-09-onboarding-form-redesign.md`, `docs/superpowers/plans/2026-06-09-onboarding-form-redesign.md`
- Skills applied: `frappe-doctype-skill` (Date / Phone-validated Data / Link-to-master / Select / Table child-table fieldtypes; Section Break + Column Break form layout), `frappe-dashboard-design` (status chips for doc rows, Inter type scale, light theme, 4px spacing, button hierarchy).
```

- [ ] **Step 3: Commit**
```bash
git add wireframe/BUILD_STATUS.md
git commit -m "Onboarding redesign (4/4): heartbeat — BUILD_STATUS.md"
```

---

### Task 5: Push + browser acceptance gate (desktop only)

- [ ] **Step 1:** Push the 4 commits:
```bash
git push origin dev
```

- [ ] **Step 2:** Manual desktop browser check at https://samarthdris.github.io/Samavesh-WebApp/wireframe/ (after ~1-2 min Pages rebuild) or via local file. Verify:
  1. **Dup-box** — placeholder now reads "Enter email or student's full name…"; body says "Search by email or full name".
  2. **Form shell** — left anchor sidebar visible, 8 items numbered 1-8; clicking each smooth-scrolls to the matching section and highlights the anchor (teal left-border + teal-50 bg).
  3. **Scroll-spy** — scrolling the form body updates the highlighted anchor.
  4. **Personal section** — Full Name (full-width row), then row of (DOB | Gender dropdown), then row of (Mobile | Form Submission Date), etc. Picking a date in DOB → "Age: NN.N years" chip appears. Typing "9876543210" in mobile and tabbing out → green ✓ Valid chip. Typing "123" → red ✗ Invalid chip. Gender dropdown contains 5 options Male/Female/Non-Binary/Other/Prefer not to say.
  5. **Documents section** — child-table with 2 group headers. Must Have group has Aadhar / Annual Income Cert / 10th SLC / 12th SLC. Good to Have has Caste / Non-Creamy Layer / Domicile / Ration / Orphan. Each row: name + Status dropdown + Attach button. Attach button is disabled until Status = "Have it".
  6. **Socio-economic** — Social Background, Annual Income, Family Occupation all rendered as dropdowns (not vertical radio lists).
  7. **No "Continue" buttons** anywhere in the onboarding form.
  8. **Sticky footer** — "Save draft" (ghost) + "Submit Onboarding ✓" (primary teal) always visible. Clicking Submit without typing the consent name shows the consent validation alert and scrolls to the Consent section.
  9. **Forms 2 & 3 unaffected** — open f-scholarship and f-docapp screens; the multi-step Continue stepper still works.
  10. **Deep-link** — `?role=fellow&screen=f-onboard` still works.

- [ ] **Step 3:** If anything looks off → tell the user, do NOT auto-fix. If all pass → done.

---

## Out of scope (this plan)

- Forms 2 (Scholarship Data Entry) and 3 (Documentation Application) — same patterns will be applied later.
- Real Frappe DocType creation (Gender master, Onboarding Document child DocType) — production deployment.
- Welcome / email verification email post-onboarding — separate plan.
- Mobile viewport polish — per `samavesh-web-focus`.
- Other Fellow screens (My Students, Applications) — separate plans.
