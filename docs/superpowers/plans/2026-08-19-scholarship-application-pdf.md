# Scholarship Application PDF — Implementation Plan

**Goal:** One shared PDF per scholarship application (`pm`/`ms`/`ab`/`smm`), attached by the Fellow via the Scholarship Data Entry form, downloadable with zero extra navigation from the Student's Scholarships screen, the Fellow's Applications tab, and the Admin's oversight Applications tab (read-only). Removes the second, redundant upload box in the Fellow's "record submission" mini-form.

**Architecture:** One JS object `APP_PDF` keyed by the 4 existing `data-app` values, one render function, one `.app-pdf-host[data-app]` container added to each of the 4 applications on each of the 3 surfaces (12 insertions total), plus the Data Entry form's existing (currently inert) upload field wired to write into `APP_PDF.pm` (documented simplification — see design doc).

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build. Verification = render + click-through.

## Global constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** — ask before every commit, ask again before any push.
- **Single upload point:** only the Data Entry form attaches a file; Fellow's Applications-tab view gets a "Replace" link that jumps to that form, never its own file input.
- **Student and Admin are download-only** — no attach control on either surface.
- **No inert UI, no `prompt()`.**

---

## Task 1: `APP_PDF` engine + CSS

**Files:** `wireframe/index.html` — CSS immediately before `</style>` (line 1154, same anchor Module 2 used — insert after Module 2's block if that's already landed, otherwise before `</style>`); JS immediately before `var NOTIF = {` (line 4940, same anchor as Module 2 — insert after Module 2's `DOC_NOTES` block if present).

- [ ] **Step 1: CSS.**
```css
.app-pdf-row{display:flex;align-items:center;gap:10px;flex-wrap:wrap;margin-top:10px;padding-top:10px;border-top:1px dashed var(--line);font-size:13px}
.app-pdf-row .apn{color:var(--muted)}
```

- [ ] **Step 2: JS engine.**
```js
// Scholarship Application PDF — one file per application (pm/ms/ab/smm), Fellow-attached via the Data Entry form.
var APP_PDF = {
  pm: {name:'PMS-2026-0042-submitted.pdf', on:'28 May'}
};
function renderAppPdf(key, canReplace){
  var f = APP_PDF[key];
  if(!f){
    return '<div class="app-pdf-row"><span class="apn">📄 Application PDF: Not yet uploaded'+(canReplace?' — attach it via the Scholarship Data Entry form':'')+'</span></div>';
  }
  return '<div class="app-pdf-row"><span>📄 Application PDF: <b>'+f.name+'</b></span><span class="apn">Uploaded '+f.on+'</span>'+
    '<a href="#" onclick="event.preventDefault();downloadDoc(\''+f.name+'\')">Download</a>'+
    (canReplace ? ' <a href="#" onclick="event.preventDefault();go(\'f-scholarship\')">Replace</a>' : '')+
    '</div>';
}
function refreshAppPdf(key){
  document.querySelectorAll('.app-pdf-host[data-app="'+key+'"]').forEach(function(host){
    host.innerHTML = renderAppPdf(key, host.hasAttribute('data-can-replace'));
  });
}
function initAppPdfAll(){
  document.querySelectorAll('.app-pdf-host').forEach(function(host){
    host.innerHTML = renderAppPdf(host.getAttribute('data-app'), host.hasAttribute('data-can-replace'));
  });
}
// Called from the Scholarship Data Entry form's Upload field (demo always targets 'pm' — see design doc's stated simplification).
function saveSchFormPdf(input){
  if(!input.files || !input.files[0]) return;
  APP_PDF.pm = {name:input.files[0].name, on:'today'};
  refreshAppPdf('pm');
}
```

- [ ] **Step 3: Wire init on load.** Add `initAppPdfAll();` next to the other one-time init calls (same load hook Module 2's `initDocNotesAll()` uses, if already landed — otherwise the same `DOMContentLoaded`/init block used by `initTurnPills()`/`initFellowSubdocs()`).

- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Application PDF: APP_PDF engine + render/refresh/save"` — ask first.

---

## Task 2: Wire the Data Entry form's upload field + remove the redundant box

**Files:** `wireframe/index.html`.

- [ ] **Step 1: Wire the real upload field.** At line 3373, change:
  `<div class="q full" id="uploadPdfQ"><div class="qen">Upload Submitted PDF <span class="req">*</span></div><input type="file" accept="application/pdf" /><div class="qhelp">Final submitted application PDF (proof). PDF, max 10 MB.</div></div>`
  to:
  `<div class="q full" id="uploadPdfQ"><div class="qen">Upload Submitted PDF <span class="req">*</span></div><input type="file" accept="application/pdf" onchange="saveSchFormPdf(this)" /><div class="qhelp">Final submitted application PDF (proof). PDF, max 10 MB. This becomes the one file downloadable from Applications and the student's Scholarships screen.</div></div>`

- [ ] **Step 2: Remove the redundant box in the MS record-submission form.** At line 2993, replace:
  `<div class="fg full"><label>Submitted Application PDF<span class="req">*</span></label><input type="file" /><span class="hint">Final submitted PDF · mandatory proof per Fellow SOP</span></div>`
  with:
  `<div class="fg full"><label>Submitted Application PDF<span class="req">*</span></label><div class="app-pdf-host" data-app="ms" data-can-replace></div><span class="hint">Attach this from the Scholarship Data Entry form — same file as shown on Applications.</span></div>`

- [ ] **Step 3 (GATED): commit.** `git add wireframe/index.html && git commit -m "Application PDF: wire Data Entry upload, remove redundant record-submission box"` — ask first.

---

## Task 3: Fellow `#fp-apps` — all 4 applications

**Files:** `wireframe/index.html`, lines ~2925–3061.

- [ ] **Step 1 — PM** (`app-acc-body`, ~line 2951, immediately after the `.tl` block's closing `</div>` and before the `.btnrow` div at line 2952): insert
  `<div class="app-pdf-host" data-app="pm" data-can-replace></div>`
- [ ] **Step 2 — MS.** Already handled by Task 2 Step 2 (the record-submission form's own box now doubles as this application's PDF host) — no separate insert needed. Add a note comment `<!-- Application PDF for MS lives in the record-submission form above -->` for future-reader clarity.
- [ ] **Step 3 — AB** (`app-acc-body`, ~line 3018, after the `<p>` description and before the `.btnrow`): insert
  `<div class="app-pdf-host" data-app="ab" data-can-replace></div>`
- [ ] **Step 4 — SMM** (`app-acc-body`, ~line 3052, after the `.tl` block and before the `.btnrow` at line 3053): insert
  `<div class="app-pdf-host" data-app="smm" data-can-replace></div>`
- [ ] **Step 5: Render — verify.** `?role=fellow&screen=f-student` → Applications tab. Expect: PM shows Download (seeded file) + Replace; MS shows "Not yet uploaded — attach it via the Scholarship Data Entry form" inside its own record-submission form (no second box); AB and SMM show the same "Not yet uploaded" line in their body.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Application PDF: Fellow Applications tab (4/4)"` — ask first.

---

## Task 4: Admin `#ap-apps` — all 4 applications, read-only

**Files:** `wireframe/index.html`, lines ~3894–3992.

- [ ] **Step 1 — PM** (`app-acc-body`, ~line 3918, after the `.tl` block, before the body's closing `</div>` at 3919): insert
  `<div class="app-pdf-host" data-app="pm"></div>` (no `data-can-replace` — Admin is read-only).
- [ ] **Step 2 — MS** (`app-acc-body`, ~line 3941, after the `<p>` description, before the body's closing `</div>` at 3942): insert
  `<div class="app-pdf-host" data-app="ms"></div>`
- [ ] **Step 3 — AB** (~line 3964, after the `<p>`, before closing at 3965): insert
  `<div class="app-pdf-host" data-app="ab"></div>`
- [ ] **Step 4 — SMM** (~line 3989, after the oversight `<p>`, before closing at 3990): insert
  `<div class="app-pdf-host" data-app="smm"></div>`
- [ ] **Step 5: Render — verify.** `?role=admin&screen=a-student` → Applications tab. Expect: PM downloadable, MS/AB/SMM "Not yet uploaded" (no "attach via…" wording — that's Fellow-facing only), no Replace/attach control anywhere on this surface.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Application PDF: Admin Applications tab (4/4, read-only)"` — ask first.

---

## Task 5: Student `#scholarships` — all 4 applications

**Files:** `wireframe/index.html`, lines ~1532–1609.

- [ ] **Step 1 — PM** (`.sch2-side`, ~line 1547, after `.s-next`, before the `.card`'s closing `</div>` at 1549): insert
  `<div class="app-pdf-host" data-app="pm"></div>`
- [ ] **Step 2 — MS** (`.sch2-side`, ~line 1566, before closing at 1568): insert
  `<div class="app-pdf-host" data-app="ms"></div>`
- [ ] **Step 3 — AB** (`.sch2-side`, ~line 1587, after the existing "View documents" button, before closing at 1588): insert
  `<div class="app-pdf-host" data-app="ab"></div>`
- [ ] **Step 4 — SMM** (`.sch2-side`, ~line 1607, before closing at 1609): insert
  `<div class="app-pdf-host" data-app="smm"></div>`
- [ ] **Step 5: Render — verify.** `?role=student&screen=scholarships`. Expect: PM's card shows a Download link with the seeded filename; MS/AB/SMM show "Not yet uploaded" (no "attach via…" wording — that's Fellow-facing only); no upload control anywhere on this surface. Confirm the GP and EBC "eligible, not applied" cards (no `data-app`, not real applications yet) correctly get nothing inserted.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Application PDF: Student Scholarships screen (4/4)"` — ask first.

---

## Task 6: Full-sweep verify + heartbeat + commit

**Files:** `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Consistency check.** Grep confirms `app-pdf-host` appears exactly 12 times (4 applications × 3 surfaces, counting MS's Task 2 box as its Fellow-surface instance) with matching `data-app` values everywhere; exactly 4 `data-can-replace` attributes, all on the Fellow surface only.
- [ ] **Step 2: Behaviour render sweep.** Re-render + click through: the live Data-Entry-form upload → `APP_PDF.pm` updates → confirm it now reflects on Student/Fellow/Admin without a page reload (same in-memory object, all three DOM regions present at once); zero-file empty state on MS/AB/SMM everywhere.
- [ ] **Step 3: Update heartbeat.** Add a dated section to `wireframe/BUILD_STATUS.md` summarising the Scholarship Application PDF feature (engine + Data Entry wiring + redundant-box removal + 3-surface rollout) as done + render-verified. Note the source: `New Feedbacks/Module 3.md`.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html wireframe/BUILD_STATUS.md && git commit -m "Heartbeat: Scholarship Application PDF complete"` — then ask whether to **push** (separate gate).

---

## Self-Review (against the design doc)

- **Spec coverage:** engine + demo data → Task 1. Single upload point + redundant-box removal → Task 2. Fellow/Admin/Student rollout → Tasks 3–5. Verification → Tasks 3–6.
- **Placeholder scan:** every insertion has concrete code and a real `data-app` value; no TBD.
- **Stated simplification carried through:** the Data Entry form always writes to `APP_PDF.pm` — flagged in the design doc, not silently assumed.
- **Constraint:** every commit GATED; push is a separate, later gate.
