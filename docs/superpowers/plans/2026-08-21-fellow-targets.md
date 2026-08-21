# Fellow Target Assignment — Implementation Plan

**Goal:** Admin sets per-fellow, per-period, per-metric targets from the Fellows & Users screen; the fellow sees their own target-vs-actual on their home screen. Three metrics — Applications Submitted (live from `CASES`), Documents Collected and Students Onboarded (declared mock). "No target set" renders distinctly from a target of 0.

**Architecture:** One `FELLOW_TARGETS` object keyed `period → fellow → metric`, one `TARGETS_ACTUAL` mock for the two metrics with no live source, `targetActual(fellow, metric)` resolving live-or-mock, `renderTargetPanel(fellow)` for the Admin inline row and `renderMyTargets()` for the Fellow strip. Both re-render from the same hooks that already refresh the workload cards.

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build. Verification = render + click-through.

**Design doc:** `docs/superpowers/specs/2026-08-21-fellow-targets-design.md`

## Global constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** — ask before every commit, ask again before any push.
- **No target set ≠ 0.** Enforced in both renderers and checked by probe.
- **Actuals are never typed by a user** — computed, per the source document.
- **No inert UI:** the period selector, Save and Clear must all visibly do something.
- **Do not touch** the existing Fellows & Users figures, the MIS tables, or the Point 8 cards.

---

## Task 1: Data + actual resolution + CSS

**Files:** `wireframe/index.html` — CSS before `</style>`; JS after the `FDASH` block.

- [ ] **Step 1: CSS.** `.tgt-panel` (inline panel row), `.tgt-row` (metric line), `.tgt-bar`/`.tgt-fill` (progress, reusing the MIS bar gradient), `.tgt-none` (muted "no target set"), `.tgt-strip` (Fellow home strip), `.tgt-gap`.
- [ ] **Step 2: `FELLOW_TARGETS`** — two seeded periods (Jul and Aug 2026) so switching period visibly changes values; Aug deliberately leaves one metric unset on one fellow so the "no target set" state is visible without the reviewer having to create it.
- [ ] **Step 3: `TARGETS_ACTUAL`** — per-fellow mock for Documents Collected and Students Onboarded, students seeded to the values already in the Fellows & Users table (24 / 19 / 22 / 17).
- [ ] **Step 4: `targetActual(fellow, metric)`** — for `apps`, count that fellow's `CASES` rows with `appId !== '—'` (an application ID exists only once submitted on the portal); otherwise read `TARGETS_ACTUAL`.
- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html && git commit -m "Fellow targets: data model, actual resolution, styles"` — ask first.

---

## Task 2: Admin panel on Fellows & Users

**Files:** `wireframe/index.html`, `#a-fellows` lines 4191–4199.

- [ ] **Step 1:** Add a `Targets` header cell and a per-row button (`toggleTargetPanel(this,'<fellow>')`) to all four rows.
- [ ] **Step 2:** `toggleTargetPanel()` inserts/removes a `<tr class="tgt-panel">` immediately after that fellow's row, rendered by `renderTargetPanel(fellow)`: period select, three metric rows (target input · actual · progress · gap), Save and Clear.
- [ ] **Step 3:** `saveTargets(btn, fellow)` writes the three inputs into `FELLOW_TARGETS[period][fellow]`, treating an empty input as **no target** and `0` as a real zero; re-renders the panel and the Fellow strip. `clearTarget(fellow, metric)` deletes one metric.
- [ ] **Step 4:** `setTargetPeriod(sel, fellow)` switches period and re-renders that panel only.
- [ ] **Step 5: Render — verify.** Open a panel; actuals present; set/Save; Clear returns "No target set"; period switch preserves each period's own values.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Fellow targets: Admin panel on Fellows & Users"` — ask first.

---

## Task 3: Fellow home strip

**Files:** `wireframe/index.html`, `#f-home` after `#fDashCards`.

- [ ] **Step 1:** Add `<div id="myTargets"></div>` under the workload card grid; `renderMyTargets()` renders the "My targets — <period>" strip for `CASE_FELLOW`, three progress bars, actual vs target, gap, and "No target set" where unset.
- [ ] **Step 2:** Call it from the load hook beside `renderFellowCards()`, and from `_rerenderAll()` so a caseload change moves the fellow's own progress too.
- [ ] **Step 3: Render — verify.** Strip matches the Admin panel's numbers for Rahul More; advancing one of his cases to a submitted state raises the Applications Submitted actual on both surfaces.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Fellow targets: my-targets strip on Fellow home"` — ask first.

---

## Task 4: Full-sweep verify + heartbeat + commit

**Files:** `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Behaviour sweep** — all 7 checks in the design doc's Verification section, plus a screenshot of the Admin panel and the Fellow strip.
- [ ] **Step 2: Consistency check** — the Students Onboarded actuals equal the Students column already in the Fellows & Users table; the Applications Submitted actual equals a manual `CASES` count; no target figure appears on a third surface.
- [ ] **Step 3: Heartbeat** — dated section in `wireframe/BUILD_STATUS.md`: what shipped, the live-vs-mock split, the "no target set ≠ 0" handling, the Frappe DocType note, and that this module was built despite its own out-of-scope header on Shweta's standing instruction. Source: `New Feedbacks/Point_06_Target_Assign_to_Fellow_Flow.md`.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html wireframe/BUILD_STATUS.md docs/superpowers/specs/2026-08-21-fellow-targets-design.md docs/superpowers/plans/2026-08-21-fellow-targets.md "New Feedbacks/Point_06_Target_Assign_to_Fellow_Flow.md"` — ask first. Push is a separate gate.

---

## Self-Review (against the design doc)

- **Spec coverage:** data + actuals → Task 1. Admin entry → Task 2. Fellow visibility → Task 3. Docs + verification → Task 4.
- **Placeholder scan:** every metric has a named source; no TBD.
- **Edge case built, not just noted:** the seed data deliberately leaves one metric unset so "No target set" is visible on first load.
- **Ripple check:** targets appear on exactly two surfaces; the MIS Fellow-wise table is deliberately left alone to avoid a third copy of the same numbers.
- **Mock declared:** two of three actuals are mock, stated in the design doc and the heartbeat, seeded to agree with figures already on screen.
- **Constraint:** every commit GATED; push separate.
