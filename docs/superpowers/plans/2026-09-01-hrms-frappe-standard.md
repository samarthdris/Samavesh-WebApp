# HRMS — replace the custom build with standard Frappe HR — Implementation Plan

**Goal:** Remove every custom attendance/leave surface from the wireframe and put one "delivered by the standard Frappe HR app" page in its place — showing the real stock screens, and stating on the page where vanilla Frappe HR differs from what Point 9 asked for.

**Architecture:** One renderer, `renderHrStub(role)`, mounted into both `#f-attendance` and `#a-attendance`. Screen ids, nav entries and `CRUMBS` unchanged. Net deletion: ~330 lines of custom HTML/JS out, ~70 in.

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build. Verification = render + click-through.

**Design doc:** `docs/superpowers/specs/2026-09-01-hrms-frappe-standard-design.md`

## Global constraints

- **Single file** for the wireframe edits: `wireframe/index.html`. Plus three PNGs copied into `wireframe/img/`.
- **Per-task GATED commit** — ask before every commit, ask again before any push.
- **Keep `hrFellow()`** — Camps calls it (line 5950). Everything else in the 6117–6340 block goes.
- **No inert UI** — `#fAttCard` keeps a working link, not a dead button (hard rule 2).
- **No Frappe desk mock**, no reference to `wireframe/frappe-desk.html` (out of scope).
- **Render-verify with hard rule 7** — headless Chrome with `&`, capture `CHROME_PID=$!`, throwaway `--user-data-dir`, kill only that PID. Never by name.

---

## Task 1: Assets + stub renderer

**Files:** `wireframe/img/` (new), `wireframe/index.html` — CSS before `</style>`, JS near the old Point 9 block.

- [ ] **Step 1:** `mkdir wireframe/img`; copy the three tracked screenshots from `Login Logout Fellow/` to URL-safe names — `frappe-hr-checkin-form.png`, `frappe-hr-checkin-list.png`, `frappe-hr-attendance-list.png`. Originals stay put.
- [ ] **Step 2: CSS.** `.hrstub` (page wrapper), `.hrstub-note` (the delivery statement), `.hrstub-shots` / `.hrstub-shot` (screenshot cards with captions), `.hrstub-map` (the Point-9-to-Frappe table), `.hrstub-diff` (the row style for the three differences).
- [ ] **Step 3: `renderHrStub(role)`** — delivery statement, role-appropriate "what you get" list, the three captioned screenshots, and the mapping table with the WFH / holiday-list / who's-working-now / export rows. Called for both hosts at boot.
- [ ] **Step 4 (GATED): commit.** — ask first.

---

## Task 2: Strip the custom surfaces

**Files:** `wireframe/index.html` — `#fAttCard` (2350), `#f-attendance` (3289–3324), `#a-attendance` (4346–4397).

- [ ] **Step 1:** Replace the body of `#f-attendance` with the page header + `<div id="hrStubFellow"></div>`; rewrite the header copy (the current sub-line promises status-at-login, leave balance and holidays — all removed).
- [ ] **Step 2:** Same for `#a-attendance` → `<div id="hrStubAdmin"></div>`. The Export button, "Working now" panel and the three stat cards go with it — all custom, none stock.
- [ ] **Step 3:** `#fAttCard` — drop `#fAttStatusSel` and the `fellowPunch` button; keep the card as a pointer with the existing `data-go="f-attendance"` link, reworded to "Attendance and leave run in Frappe HR".
- [ ] **Step 4: Render — verify.** Both screens load, both nav entries work, no console error.
- [ ] **Step 5 (GATED): commit.** — ask first.

---

## Task 3: Delete the dead JS and CSS

**Files:** `wireframe/index.html`.

- [ ] **Step 1:** Delete `fellowPunch` (5006), `attFilter`/`attClear` (5067, 5082), `fAttFilter`/`fAttClear` (5089, 5103).
- [ ] **Step 2:** Delete the Point 9 block 6117–6340 **except `hrFellow()` at 6154**, which moves up out of the block and keeps its comment. Delete the two boot calls at 7216–7217.
- [ ] **Step 3:** For each CSS class the deletion orphans (`.att-card`, `.att-btn`, `.att-status-sel`, `.wn-list`, `.wn-row`, `.hr-sec`, `.leave-bal`, `.lb-tile`, `.leave-form`, `.lreq`, `.hol-list`, `.hol-row`, `.sample-tag`) — grep for remaining users **first**, delete only those at zero. Report any that survive and why.
- [ ] **Step 4: Render — verify.** Fellow home, Fellow attendance, Admin attendance, Admin home and Camps all render; console clean on both roles; Camps still lists the fellow's camps (proves `hrFellow` survived).
- [ ] **Step 5 (GATED): commit.** — ask first.

---

## Task 4: Docs + full sweep

**Files:** `wireframe/BUILD_STATUS.md`, `New Feedbacks/Point_09_HRMS_Attendance_Leave.md`, the two 2026-08-21 HRMS docs.

- [ ] **Step 1: Greps, expecting zero** — `fellowPunch|renderMyLeave|renderAdminLeave|applyLeave|decideLeave|adjustBalance|renderHolidays|setHolidayState|addHoliday|LEAVE_REQUESTS|LEAVE_BALANCE|LEAVE_TYPES|HOLIDAY_STATE_SEL|FELLOW_STATE|ATT_STATUS_CHOICES|fAttTable|attTable|fAttStatusSel`. Then confirm `hrFellow` is still defined and called.
- [ ] **Step 2: Behaviour sweep** — all 6 checks in the design doc's Verification section, with screenshots of the Fellow and Admin stub pages.
- [ ] **Step 3: Mark the 2026-08-21 spec and plan superseded** — a dated header line pointing at this pair. They stay in the repo as the record of why the module was removed.
- [ ] **Step 4: Update `Point_09_HRMS_Attendance_Leave.md`** — §11 records the 2026-09-01 decision: standard Frappe HR, no custom build, and the three vanilla differences. The requirements themselves are not rewritten.
- [ ] **Step 5: `BUILD_STATUS.md` heartbeat** — dated section: what was removed (both the 2026-08-21 leave module and the older attendance layer that was already on `dev`), why, what the client sees instead, that the cost is now install + configure with **zero custom development**, and that the "who's working now" panel is gone and would be a custom report if wanted back.
- [ ] **Step 6 (GATED): commit.** All doc files in one commit with the heartbeat. Push is a separate gate — and a PR to `dev` is Samarth's call, not something this plan does.

---

## Self-Review (against the design doc)

- **Spec coverage:** stub page → Task 1–2. Deletion → Task 2–3. The vanilla-difference table → Task 1 Step 3, and it is on the page, not only in the docs.
- **Nothing invented:** every screen named on the stub is a real Frappe HR doctype, and the three screenshots are stock Frappe HR from a Dhwani bench, already tracked in git — no drawn Frappe UI (hard rule 1).
- **Ripple check:** deleting the surfaces means the header copy, the `#fAttCard` control, both nav entries, `CRUMBS`, the boot calls and the orphaned CSS all move in the same round — and both 2026-08-21 docs get marked superseded rather than left contradicting the code.
- **Nothing half-baked:** the stub is read-only by design and says so; the one interactive element left on `#fAttCard` is a link that works.
- **Risk stated, not absorbed:** this removes attendance screens the client has already reviewed on `dev`. That is the intent of the change, and BUILD_STATUS records it explicitly so the removal is not discovered as a surprise.
- **Constraint:** every commit GATED; push separate.
