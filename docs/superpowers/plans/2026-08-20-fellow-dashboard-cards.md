# Fellow Interface Dashboard Cards — Implementation Plan

**Goal:** Replace the Fellow home's 4-card KPI grid with the client's confirmed 12 cards, 8 of them computed live from `CASES` (scoped to `CASE_FELLOW`) and Module 5's `FIN`, the rest from one declared mock object. Card 11 carries a configurable day-threshold; card 12 shows month-over-month completions.

**Architecture:** One `FDASH` mock object for the document-support and history figures, one `fdashCounts()` that derives the live numbers from `CASES` + `FIN`, one `renderFellowCards()` writing into a single `#fDashCards` host, called on load and re-called whenever the caseload or a disbursement changes.

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build. Verification = render + click-through.

**Design doc:** `docs/superpowers/specs/2026-08-20-fellow-dashboard-cards-design.md`

## Global constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** — ask before every commit, ask again before any push.
- **Locked definitions:** completed = Benefits Received · in process = Ready to Submit / Under Scrutiny / Application Approved · funds awaited · document completed = Accepted.
- **Fellow scoping is absolute:** every count filters `CASES` by `CASE_FELLOW`. No card may ever include Sandip K's or Dhanashree O's rows.
- **No inert UI:** card 11's threshold select must re-render; nothing on screen may be decoration.
- **Do not touch** the caseload triage tiles, the attendance card, or `CASES` itself.

---

## Task 1: `FDASH` mock + live count engine

**Files:** `wireframe/index.html` — CSS before `</style>`; JS after the Module 5 `FIN` block, before `var NOTIF = {`.

- [ ] **Step 1: CSS.** `.fdash-grid` (responsive `repeat(auto-fit,minmax(150px,1fr))`), `.fdash-mini` (card 12's bar strip), `.fdash-thr` (card 11's threshold select). Reuse `.card.stat` for the cards themselves — no new card component.
- [ ] **Step 2: `FDASH` mock object** — document-support figures (assigned / completed / in process / documents supported), per-case mock ages in days for card 11, and 6 months of completion counts for card 12. One object, clearly commented as the mock half.
- [ ] **Step 3: `fdashCounts()`** — returns every figure. Live from `CASES.filter(c => c.fellow === CASE_FELLOW)`: distinct students, total cases, Benefits Received, the three in-process statuses, Rejected. Live from `FIN`: sum of entries whose status is "Funds Disbursed", plus `FDASH.fundsBaseline` for the Fellow's other students.
- [ ] **Step 4: `renderFellowCards()`** — builds all 12 `.card.stat` tiles into `#fDashCards`. Card 11 renders its threshold select with the current value; card 12 renders 6 bars.
- [ ] **Step 5: `setFdashThreshold(sel)`** — stores the choice and re-renders.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Fellow cards: FDASH engine + live counts from CASES and FIN"` — ask first.

---

## Task 2: Swap the 4-card grid for the 12-card grid

**Files:** `wireframe/index.html`, `#f-home` line 2252.

- [ ] **Step 1:** Replace the existing 4-card grid (lines 2252–2257) with `<div class="fdash-grid stagger" id="fDashCards"></div>`, keeping the `<!-- KPIs (MIS-03 fellow performance) -->` comment updated to name the source document.
- [ ] **Step 2: Init on load.** Add `if(typeof renderFellowCards==='function') renderFellowCards();` beside `initFinAll()` in the existing load hook.
- [ ] **Step 3: Render — verify.** `?role=fellow&screen=f-home` — 12 cards, none blank; the 4 original figures still present under their new card names.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Fellow cards: 12-card grid replaces the 4-card KPI strip"` — ask first.

---

## Task 3: Keep the cards live

**Files:** `wireframe/index.html`.

- [ ] **Step 1:** Call `renderFellowCards()` from the places that change the underlying data — `caseSetStatus`, `caseAdvance`, `caseReopen` (caseload status changes) and `saveFinInstallment` (a recorded disbursement). Read each function first; each must keep doing everything it already does, with the re-render appended.
- [ ] **Step 2: Render — verify the loop.** Caseload → advance a case to Benefits Received → home: "Applications completed" is up by one and "in process" down by one. Financial Tracking → record ₹25,000 → home: "Total funds unlocked" is up by ₹25,000.
- [ ] **Step 3: Render — verify scoping.** Probe that the sum of cards 4, 6, 10 plus Eligibility Identified / Documents Pending / Re-apply equals this Fellow's case count exactly, and that the total is strictly less than `CASES.length`.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html && git commit -m "Fellow cards: re-render on caseload and disbursement changes"` — ask first.

---

## Task 4: Card 11 threshold + card 12 trend

**Files:** `wireframe/index.html`.

- [ ] **Step 1: Render — verify** the threshold select moves the count across 7 / 15 / 30 days, and that card 12's 6 bars scale to their own maximum (no bar overflowing its track, no zero-height bar for a non-zero month).
- [ ] **Step 2 (GATED): commit** only if Task 1's implementation needed correction here; otherwise fold into Task 5.

---

## Task 5: Full-sweep verify + heartbeat + commit

**Files:** `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Behaviour sweep** — all 6 checks in the design doc's Verification section, with a screenshot of the grid.
- [ ] **Step 2: Consistency grep** — exactly one `#fDashCards` host; the old 4-card grid markup is gone, not orphaned.
- [ ] **Step 3: Heartbeat** — dated section in `wireframe/BUILD_STATUS.md`: the 12 cards, which are live vs mock, the two document claims the repo disproved (cards 4/6/10 were never blocked; the 5-value document status set doesn't exist here), the locked definitions, and `track_changes` as the shared production prerequisite with the MIS dashboard. Source: `New Feedbacks/Point_08_Fellow_Interface_Cards.md`.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html wireframe/BUILD_STATUS.md docs/superpowers/specs/2026-08-20-fellow-dashboard-cards-design.md docs/superpowers/plans/2026-08-20-fellow-dashboard-cards.md "New Feedbacks/Point_08_Fellow_Interface_Cards.md"` — ask first. Push is a separate, later gate.

---

## Self-Review (against the design doc)

- **Spec coverage:** engine → Task 1. Grid swap → Task 2. Liveness → Task 3. Card 11/12 behaviour → Tasks 1 and 4. Docs + verification → Task 5.
- **Placeholder scan:** every card has a named source (live query or `FDASH` key); no TBD.
- **Mock is declared, not disguised:** the document-support and history figures live in one clearly-commented object, and BUILD_STATUS says which cards are mock.
- **Ripple check:** the existing 4 figures are absorbed into the new grid rather than duplicated beside it; caseload tiles and attendance card deliberately untouched.
- **Constraint:** every commit GATED; push separate.
