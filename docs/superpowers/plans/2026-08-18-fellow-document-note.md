# Fellow-to-Student Document Note — Implementation Plan

**Goal:** A Fellow can add a short, timestamped note against one specific document (of the 9-document checklist); the Student sees it read-only with a bell notification; the Admin sees it read-only too (per Shweta's 2026-08-18 decision). Sub-documents are untouched.

**Architecture:** One JS data object `DOC_NOTES` (keyed by document display name) + one render function (`renderDocNotes`) populate a `.doc-notes-host[data-doc="…"]` container inserted after every one of the 9 documents on all three surfaces (Student `#documents`, Fellow `#fp-docs`, Admin `#ap-docs` — three separately-authored markups, no shared loop today, so each gets its own insert). Only the Fellow surface also gets a `.doc-note-add` button per document. Adding a note re-renders every matching host across all three screens (they're all in the DOM at once, just hidden) and pushes one entry to `NOTIF.student`.

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build. Verification = headless-Chrome render, not unit tests.

## Global constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** — ask before every commit, ask again (separately) before any push.
- **No new vocabulary** — document names must match exactly what's already on screen.
- **No inert UI** — every button does something visible; no `prompt()`.
- **Only the 9 main documents** — do not touch the Ration/Caste/Domicile/Income sub-document lists (`SUBDOCS`, `renderSubdocList`, etc.) — out of scope per the design doc.
- **Read-only stays read-only** — Student and Admin never get an add-note control, only Fellow does.

### Render snippet (reuse, adjust path to your machine)
```
Copy-Item "wireframe/index.html" "/tmp/swf.html" -Force
# open /tmp/swf.html?role=<role>&screen=<screen> in a headless-Chrome render, or just open it in a normal browser tab and click through — this repo's usual recipe assumes Windows/PowerShell + headless Chrome; on this Mac, a plain browser open + manual click-through is an acceptable substitute as long as every interaction below is actually clicked, not just eyeballed in the HTML.
```

---

## Task 1: CSS + JS engine

**Files:** `wireframe/index.html` — CSS immediately before `</style>` (line 1154); JS immediately before the `var NOTIF = {` block (line 4940).

- [ ] **Step 1: CSS.** Immediately before `</style>` (line 1154), insert:
```css
.doc-notes{margin-top:8px;padding-top:8px;border-top:1px dashed var(--line)}
.doc-note{padding:6px 0;font-size:12.5px}
.doc-note b{color:var(--ink)}
.doc-note-date{color:var(--muted);margin-left:6px;font-size:11.5px}
.doc-note-text{color:var(--ink-soft);margin-top:2px}
.doc-note-form textarea{width:100%;min-height:56px;padding:8px 10px;border:1px solid var(--line);border-radius:8px;font-family:inherit;font-size:13px;box-sizing:border-box;margin-top:8px}
.doc-note-form{margin-top:6px}
```

- [ ] **Step 2: JS engine.** Immediately before the line `var NOTIF = {` (line 4940), insert:
```js
// Fellow-to-Student document notes. Keyed by exact document display name. Fellow writes, Student + Admin read.
var DOC_NOTES = {
  'Non-Creamy Layer Certificate': [
    {by:'Rahul More', on:'16 May', text:"Sent back for a clearer scan — the caste-category line wasn't legible. Re-uploading a fresh copy now."}
  ]
};
function renderDocNotes(key){
  var notes = DOC_NOTES[key];
  if(!notes || !notes.length) return '';
  return '<div class="doc-notes">'+notes.map(function(n){
    return '<div class="doc-note"><b>'+n.by+'</b><span class="doc-note-date">'+n.on+'</span><div class="doc-note-text">'+n.text+'</div></div>';
  }).join('')+'</div>';
}
function refreshDocNotes(key){
  document.querySelectorAll('.doc-notes-host[data-doc="'+key+'"]').forEach(function(host){ host.innerHTML = renderDocNotes(key); });
}
function initDocNotesAll(){
  document.querySelectorAll('.doc-notes-host').forEach(function(host){ host.innerHTML = renderDocNotes(host.getAttribute('data-doc')); });
}
function renderAddDocNote(btn){
  var key = btn.getAttribute('data-doc');
  var wrap = document.createElement('div'); wrap.className='doc-note-form';
  wrap.innerHTML = '<textarea placeholder="Add a note for the student…"></textarea><div style="display:flex;gap:8px;justify-content:flex-end;margin-top:6px"><button class="btn btn-ghost btn-sm" type="button" onclick="this.closest(\'.doc-note-form\').remove()">Cancel</button><button class="btn btn-primary btn-sm" type="button" onclick="sendDocNote(this,\''+key+'\')">Send</button></div>';
  btn.insertAdjacentElement('afterend', wrap);
  btn.style.display='none';
}
function sendDocNote(sendBtn, key){
  var form = sendBtn.closest('.doc-note-form');
  var ta = form.querySelector('textarea');
  var text = (ta.value||'').trim();
  if(!text){ ta.focus(); return; }
  DOC_NOTES[key] = DOC_NOTES[key] || [];
  DOC_NOTES[key].push({by:'Rahul More', on:'today', text:text});
  refreshDocNotes(key);
  NOTIF.student.unshift({t:'Rahul More added a note on your '+key, s:'today'});
  form.remove();
  var addBtn = document.querySelector('.doc-note-add[data-doc="'+key.replace(/"/g,'\\"')+'"]');
  if(addBtn) addBtn.style.display='';
}
```

- [ ] **Step 3: Wire init on load.** Find the `DOMContentLoaded`/init block that already calls other one-time setup (e.g. wherever `initFellowSubdocs`/`initTurnPills` are invoked on load — grep `initTurnPills()` call site). Add `initDocNotesAll();` alongside it, so all three surfaces' hosts render on first paint (harmless before login — hosts just render empty or with the seeded NCL note; visibility is still gated by the normal role/screen `.hidden` classes).

- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Document notes: DOC_NOTES engine + render/add/send"` — ask first.

---

## Task 2: Student `#documents` — read-only threads on all 9 documents

**Files:** `wireframe/index.html`, lines ~1673–1839 (the `#documents` section).

Insert `<div class="doc-notes-host" data-doc="EXACT NAME"></div>` as the last child inside each document's containing block (flat `.card` → right after its `.doc-row` closing `</div>`, before the `.card`'s own closing `</div>`; `.arr-card` → inside `.arr-body`, after the last existing child).

- [ ] **Step 1 — Aadhaar Card** (flat card, ~line 1678). After the `.doc-row`'s closing `</div>` and before the card's closing `</div>` (line 1679), insert:
  `<div class="doc-notes-host" data-doc="Aadhaar Card"></div>`
- [ ] **Step 2 — Annual Income Certificate** (`arr-card`, inside `.arr-body`, after the `.tl` block closes, ~line 1710, before `</div></div>` that closes `.arr-body`/`.arr-card`): insert
  `<div class="doc-notes-host" data-doc="Annual Income Certificate"></div>`
- [ ] **Step 3 — 10th School Leaving Certificate** (flat card, ~line 1720, before its card's closing `</div>` at 1721): insert
  `<div class="doc-notes-host" data-doc="10th School Leaving Certificate"></div>`
- [ ] **Step 4 — 12th School Leaving Certificate** (flat card, ~line 1729, before closing `</div>` at 1730): insert
  `<div class="doc-notes-host" data-doc="12th School Leaving Certificate"></div>`
- [ ] **Step 5 — Caste Certificate** (flat card, ~line 1741, before closing `</div>` at 1742): insert
  `<div class="doc-notes-host" data-doc="Caste Certificate"></div>`
- [ ] **Step 6 — Non-Creamy Layer Certificate** (`arr-card` id `arr1`, inside `.arr-body` after the `.fellow-help` block, ~line 1775, before `</div></div>` closing `.arr-body`/`.arr-card` at 1776): insert
  `<div class="doc-notes-host" data-doc="Non-Creamy Layer Certificate"></div>`
  → **this is the one that should render the seeded note** (Task 1's `DOC_NOTES` entry).
- [ ] **Step 7 — Domicile Certificate** (`arr-card` id `arr2`, inside `.arr-body` after `.fellow-help`, ~line 1809, before `</div></div>` closing at 1810): insert
  `<div class="doc-notes-host" data-doc="Domicile Certificate"></div>`
- [ ] **Step 8 — Ration Card** (card with have-choice, ~line 1829, after the `#ration-subdocs` block, before the card's closing `</div>` at 1830): insert
  `<div class="doc-notes-host" data-doc="Ration Card"></div>`
- [ ] **Step 9 — Orphan Certificate** (flat card, ~line 1838, before closing `</div>` at 1839): insert
  `<div class="doc-notes-host" data-doc="Orphan Certificate"></div>`
- [ ] **Step 10: Render — verify.** Open `?role=student&screen=documents`. Expect: Non-Creamy Layer Certificate shows the seeded note ("Rahul More · 16 May · Sent back for a clearer scan…"); all other 8 documents show nothing extra (no empty-state box). No student-facing control anywhere.
- [ ] **Step 11 (GATED): commit.** `git add wireframe/index.html && git commit -m "Document notes: read-only threads on Student Documents (9/9)"` — ask first.

---

## Task 3: Fellow `#fp-docs` — writable threads + Add-note on all 9 documents

**Files:** `wireframe/index.html`, lines ~2853–2915 (`#fp-docs` tabpane, inside a student's Fellow detail view).

Each Fellow `.doc-row` gets, immediately after its own closing `</div>`: an "Add note" button (`.doc-note-add`) + a `.doc-notes-host`. Skip the "Recently uploaded by student" strip's Ration Card entry (line ~2840–2852) — that's a different, already-functional review-action card, not the checklist row; the checklist's own Ration Card row (line ~2899) is what gets the note controls.

- [ ] **Step 1 — Aadhaar Card** (~line 2860, after its `.doc-row` closes): insert
  `<div style="margin-left:58px"><button class="btn btn-ghost btn-sm doc-note-add" type="button" data-doc="Aadhaar Card" onclick="renderAddDocNote(this)">Add note</button><div class="doc-notes-host" data-doc="Aadhaar Card"></div></div>`
- [ ] **Step 2 — Annual Income Certificate** (~line 2866): same pattern, `data-doc="Annual Income Certificate"`.
- [ ] **Step 3 — 10th School Leaving Certificate** (~line 2872): same pattern, `data-doc="10th School Leaving Certificate"`.
- [ ] **Step 4 — 12th School Leaving Certificate** (~line 2878): same pattern, `data-doc="12th School Leaving Certificate"`.
- [ ] **Step 5 — Caste Certificate** (~line 2886): same pattern, `data-doc="Caste Certificate"`.
- [ ] **Step 6 — Non-Creamy Layer Certificate** (~line 2892): same pattern, `data-doc="Non-Creamy Layer Certificate"` → should immediately show the seeded note beneath the Add-note button.
- [ ] **Step 7 — Domicile Certificate** (~line 2898): same pattern, `data-doc="Domicile Certificate"` → **this is the live demo row** for Step 9 below.
- [ ] **Step 8 — Ration Card** (~line 2909, after the `.subdoc-host` block that follows Ration's own `.doc-row`, so the note control sits below the sub-document panel, not inside it): insert the same pattern, `data-doc="Ration Card"`.
  Note: Orphan Certificate (N/A) is intentionally **skipped** — it's not applicable to this student, nothing to ever note against it; matches the "no inert UI on things that can't happen" principle. Flag to Shweta if it should be included anyway for future students where it *is* applicable.
- [ ] **Step 9: Render — Add-note flow (live demo).** On `?role=fellow&screen=f-student` → Documents tab, click "Add note" under Domicile Certificate → expect an inline textarea + Cancel/Send. Type "Program Admin flagged the address doesn't match Aadhaar — re-verifying with the block office." → click Send → expect: the note appears in the thread immediately, attributed to Rahul More, dated "today"; the Add-note button reappears; `NOTIF.student[0].t` now reads "Rahul More added a note on your Domicile Certificate".
- [ ] **Step 10: Render — cross-surface propagation.** Without reloading, switch to `?role=student&screen=documents` in the same session (or re-render fresh and re-run the Step 9 injection via script) → confirm the Domicile Certificate note from Step 9 also appears on the Student's screen. This is the actual test of "one `DOC_NOTES` object, three surfaces."
- [ ] **Step 11 (GATED): commit.** `git add wireframe/index.html && git commit -m "Document notes: Add-note + threads on Fellow Documents tab (8/9, Orphan N/A skipped)"` — ask first.

---

## Task 4: Admin `#ap-docs` — read-only threads on all 9 documents

**Files:** `wireframe/index.html`, lines ~3836–3885 (`#ap-docs` tabpane).

Same as Task 2 (read-only, no Add-note), inserted after each `.vrow`'s closing `</div>`.

- [ ] **Step 1 — Aadhaar Card** (~line 3842): insert `<div class="doc-notes-host" data-doc="Aadhaar Card" style="margin-left:58px"></div>`.
- [ ] **Step 2 — Annual Income Certificate** (~line 3847): same, `data-doc="Annual Income Certificate"`.
- [ ] **Step 3 — 10th School Leaving Certificate** (~line 3852): same, `data-doc="10th School Leaving Certificate"`.
- [ ] **Step 4 — 12th School Leaving Certificate** (~line 3857): same, `data-doc="12th School Leaving Certificate"`.
- [ ] **Step 5 — Caste Certificate** (~line 3864): same, `data-doc="Caste Certificate"`.
- [ ] **Step 6 — Domicile Certificate** (~line 3869, note: appears before NCL in this surface's ordering): same, `data-doc="Domicile Certificate"` → should show the Task 3 live-demo note once Task 3 is done.
- [ ] **Step 7 — Non-Creamy Layer Certificate** (~line 3874): same, `data-doc="Non-Creamy Layer Certificate"` → should show the seeded note.
- [ ] **Step 8 — Ration Card** (~line 3879): same, `data-doc="Ration Card"`.
- [ ] **Step 9 — Orphan Certificate** (~line 3884): same, `data-doc="Orphan Certificate"` (included here for full read-only symmetry even though Task 3 skipped the Fellow add-control on it — it'll just always render empty, which is correct).
- [ ] **Step 10: Render — verify.** Open `?role=admin&screen=a-student` → Documents tab. Expect: Non-Creamy Layer Certificate shows the seeded note; Domicile Certificate shows the Task 3 demo note (if Task 3's test data is still in memory in the same render) or is empty (if rendered fresh, since `DOC_NOTES` only persists for the life of one page load — that's expected wireframe behaviour, not a bug). No Admin-facing add control anywhere.
- [ ] **Step 11 (GATED): commit.** `git add wireframe/index.html && git commit -m "Document notes: read-only threads on Admin Documents tab (9/9)"` — ask first.

---

## Task 5: Full-sweep verify + heartbeat + commit

**Files:** `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Consistency check.** Grep confirms `data-doc` appears exactly 9 times on Student, 8 times on Fellow (Orphan skipped) + the special-case count, 9 times on Admin, all using the exact same 9 document-name strings verbatim across surfaces (no typos/casing drift). Confirm no `prompt()` was added and every `.doc-note-add` has a matching `.doc-notes-host` with the same `data-doc`.
- [ ] **Step 2: Behaviour render sweep.** Re-render + click through: Student read-only view, Fellow add-note + cross-surface propagation, Admin read-only view, zero-note empty state (e.g. Aadhaar Card) on all three surfaces — confirm no empty-state clutter anywhere.
- [ ] **Step 3: Update heartbeat.** Add a dated section to `wireframe/BUILD_STATUS.md` summarising the Fellow-to-Student Document Note feature (engine + 3-surface rollout + Orphan-Certificate exception) as done + render-verified. Note the client feedback source: `New Feedbacks/Module_02_Fellow_Document_Revert_Communication.md`.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html wireframe/BUILD_STATUS.md && git commit -m "Heartbeat: Fellow-to-Student document notes complete"` — then ask whether to **push** (separate gate).

---

## Self-Review (against the design doc)

- **Spec coverage:** engine + demo data → Task 1. Student read-only → Task 2. Fellow write + notification → Task 3. Admin read-only (Shweta's 2026-08-18 decision) → Task 4. Verification → Tasks 2–5.
- **Placeholder scan:** every insertion has concrete code and an exact document name; no TBD.
- **Named-consistency:** `DOC_NOTES`, `renderDocNotes`, `refreshDocNotes`, `initDocNotesAll`, `renderAddDocNote`, `sendDocNote`, classes `doc-notes-host`/`doc-note-add`/`doc-note-form` — defined once in Task 1, reused identically in Tasks 2–4.
- **Known, stated exception:** Orphan Certificate has no Add-note control on the Fellow surface (N/A document, nothing to ever note) — flagged for Shweta's confirmation in Task 3 Step 8, not silently done.
- **Constraint:** every commit GATED; push is a separate, later gate.
