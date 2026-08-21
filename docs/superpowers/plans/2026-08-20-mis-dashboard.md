# MIS / M&E Dashboard — Implementation Plan

**Goal:** Extend the Admin Program Dashboard (`#a-home`) into the five sub-sections the client asked for — Overview, Demographics, Geography, Scheme Performance, Fellow Performance, Process Efficiency — adding ~22 indicators as CSS bar charts and number cards off one `MIS` mock object, adding drop-off rates to the existing funnel, adding Student and College filters, and introducing a new **Migrant status** field across its seven ripple points.

**Architecture:** One `MIS` data object (one key per indicator, each an array of `{label, value}` rows), one generic `renderMisBars(hostId, rows, opts)` reusing the existing `.funnel` bar styling, one `renderMisAll()` on load, and `misFilter()` extended to set `MIS_FILTER`, show a "Filtered view" chip and re-render with scaled figures (stated simplification — see design doc).

**Tech stack:** Vanilla HTML/CSS/JS, one file, no build, no charting library. Verification = render + click-through.

**Design doc:** `docs/superpowers/specs/2026-08-20-mis-dashboard-design.md`

## Global constraints

- **Single file:** all edits in `wireframe/index.html`.
- **Per-task GATED commit** — ask before every commit, ask again before any push.
- **One funnel only.** Extend the existing 5-stage funnel with drop-off rates; never add a second funnel.
- **Migrant field must land in all 7 places in the same round** (hard rule 4) — Aarti's value identical in every one (hard rule 7).
- **No inert UI:** both new filter selects must visibly change the view.
- **Do not touch** the "₹ Unlocked" column, the student-list "₹71,000", the existing 5 number cards' figures, or the funnel's stage names.
- **Mock is declared:** every mock figure lives in `MIS`, and BUILD_STATUS says what is mock and what the real prerequisite is.

---

## Task 1: `MIS` data + bar renderer + sub-section CSS

**Files:** `wireframe/index.html` — CSS before `</style>`; JS near the other dashboard helpers, before `var NOTIF = {`.

- [ ] **Step 1: CSS.** `.mis-sec` (sub-section card), `.mis-sec h3`, `.mis-grid` (two charts per row, wrapping), `.misbars` + `.mb-row`/`.mb-lbl`/`.mb-track`/`.mb-fill`/`.mb-val` (reusing the funnel's gradient and track colours), `.mis-chip` (the Filtered-view chip), `.mis-note` (per-chart footnote for mock-dependent indicators).
- [ ] **Step 2: `MIS` object** — one key per indicator: `gender`, `caste`, `income`, `stream`, `grade`, `colleges`, `geo`, `migrant`, `schemeUtil`, `docSupport`, `months`, `ttApproval`, `ttDisbursement`, `renewal`, `docFriction`, `schemesTapped`. Values consistent with the existing headline figures (480 students onboarded, 264 submitted, 182 approved) so no chart contradicts a card already on screen.
- [ ] **Step 3: `renderMisBars(hostId, rows, opts)`** — labelled horizontal bars, each scaled to the row set's max, value printed at the end; `opts.suffix` for "days"/"%" indicators, `opts.note` for the footnote.
- [ ] **Step 4: `renderMisAll()`** — renders every host, applying `misScale()` to each figure.
- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html && git commit -m "MIS: data object + bar renderer + sub-section styles"` — ask first.

---

## Task 2: Funnel drop-off + 6th overview card + month timeline

**Files:** `wireframe/index.html`, `#a-home` lines 3611–3650.

- [ ] **Step 1: 6th number card** — "Schemes tapped" in the existing card grid (line 3611), same `.card.stat` component.
- [ ] **Step 2: Funnel drop-off.** Read the funnel markup first. For every stage after the first, add a drop-off percentage against the previous stage, computed from the stage counts already there — not typed in by hand, so the numbers cannot contradict the bars.
- [ ] **Step 3: Month-wise timeline** — 12-bar chart in an Overview sub-section, off `MIS.months`.
- [ ] **Step 4: Render — verify.** Funnel shows a drop-off on stages 2–5 and the percentages match the counts; the new card and timeline render.
- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html && git commit -m "MIS: funnel drop-off rates, Schemes tapped card, month-wise timeline"` — ask first.

---

## Task 3: Demographics, Scheme Performance, Fellow Performance, Process Efficiency

**Files:** `wireframe/index.html`, inserted before `#a-home` closes (line 3664).

- [ ] **Step 1: Demographics sub-section** — gender, caste category, annual income, education stream, grade-wise + a "Total colleges" figure.
- [ ] **Step 2: Scheme Performance** — scheme utilisation (with a footnote that a scheme `capacity` field doesn't exist yet) and the documents-support dashboard. Move the existing Scholarship-wise card under this heading rather than rebuilding it.
- [ ] **Step 3: Fellow Performance** — move the existing fellow-wise table under this heading, unchanged.
- [ ] **Step 4: Process Efficiency** — average time-to-approval, average time-to-disbursement, repeat/renewal rate, document friction rate. Each carries a footnote naming its real prerequisite (`track_changes`, cross-year student id).
- [ ] **Step 5: Render — verify.** Every sub-section renders under its heading; no empty card; nothing duplicated between the moved cards and their new headings.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "MIS: Demographics, Scheme, Fellow and Process Efficiency sub-sections"` — ask first.

---

## Task 4: Migrant field — all 7 ripple points

**Files:** `wireframe/index.html`.

- [ ] **Step 1: Onboarding form** — new `Migrant Family?` question (`onbMigrant`; Migrant / Non-migrant / Prefer not to answer) beside Annual Family Income (line ~2637). Verify it reaches all three render modes via `cloneOnbForm`.
- [ ] **Step 2: `MY_ONBOARDING.fields`** (line ~4387) — Aarti's value.
- [ ] **Step 3: `PENDING_ONBOARDINGS`** (lines ~4449, ~4466) — both submissions.
- [ ] **Step 4: Three profile views** — student's own (line ~1972), Fellow's (line ~2765), Admin's (line ~3808), beside Annual Family Income in each.
- [ ] **Step 5: Render — verify all three form modes.** Student self-fill, Fellow approval review (value prefilled), student read-only view (value shown, disabled). Then all three profiles show the same value for Aarti.
- [ ] **Step 6 (GATED): commit.** `git add wireframe/index.html && git commit -m "Migrant status field: onboarding form (3 modes), prefill data, 3 profile views"` — ask first.

---

## Task 5: Geography sub-section + Student/College/Migrant filters

**Files:** `wireframe/index.html`.

- [ ] **Step 1: Geography sub-section** — state/district-wise plus the migrant / non-migrant split.
- [ ] **Step 2: Filter strip** — add Student, College and Migrant selects to `.misfilters` (line 3601).
- [ ] **Step 3: Extend `misFilter()`** — read it first; keep its existing note behaviour, add `MIS_FILTER` state, the "Filtered view: …" chip and a `renderMisAll()` re-render with `misScale()` applied.
- [ ] **Step 4: Render — verify.** Change each of the three new selects and confirm the chip names the selection and figures visibly change; clear back to "All" and figures return.
- [ ] **Step 5 (GATED): commit.** `git add wireframe/index.html && git commit -m "MIS: Geography sub-section + Student/College/Migrant filters"` — ask first.

---

## Task 6: Full-sweep verify + heartbeat + commit

**Files:** `wireframe/BUILD_STATUS.md`.

- [ ] **Step 1: Behaviour sweep** — all 6 checks in the design doc's Verification section, with screenshots of the dashboard and of the onboarding form's new question.
- [ ] **Step 2: Consistency greps** — `onbMigrant` present in markup once and in all three prefill datasets; exactly one funnel on the screen; no chart whose figures contradict the headline cards.
- [ ] **Step 3: Heartbeat** — dated section in `wireframe/BUILD_STATUS.md`: what was already built vs added, the one-funnel decision, the filter-scaling simplification, the migrant field's 7 ripple points, which indicators are mock and their real prerequisites, and the fact that this module was **built on Shweta's instruction despite its own out-of-scope note**. Source: `New Feedbacks/Point_5_MIS_Dashboard.md`.
- [ ] **Step 4 (GATED): commit.** `git add wireframe/index.html wireframe/BUILD_STATUS.md docs/superpowers/specs/2026-08-20-mis-dashboard-design.md docs/superpowers/plans/2026-08-20-mis-dashboard.md "New Feedbacks/Point_5_MIS_Dashboard.md"` — ask first. Push is a separate, later gate.

---

## Self-Review (against the design doc)

- **Spec coverage:** engine → Task 1. Overview + funnel → Task 2. Three sub-sections → Task 3. Migrant field → Task 4. Geography + filters → Task 5. Docs + verification → Task 6.
- **Placeholder scan:** every indicator has a named `MIS` key; every prerequisite-dependent indicator has a footnote instead of a fake solution.
- **Ripple check:** the migrant field's 7 targets are enumerated as steps, not left to memory; existing fellow-wise and scholarship-wise cards are re-homed under headings rather than duplicated.
- **Stated simplification carried through:** filter scaling is declared in the design doc, the heartbeat and this plan — never presented as real filtering.
- **Sequencing note preserved:** the design doc and heartbeat both record that the client document marked this out of scope and Shweta directed the build anyway.
- **Constraint:** every commit GATED; push separate.
