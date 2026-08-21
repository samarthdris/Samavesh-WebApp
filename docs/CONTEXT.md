# Samavesh — Project Context (portable)

> The full working context, so any machine or teammate (or a fresh Claude Code session) can get up to speed without the local `.claude/` memory. Working agreement + hard rules: [`../CLAUDE.md`](../CLAUDE.md). Live build checklist: [`../wireframe/BUILD_STATUS.md`](../wireframe/BUILD_STATUS.md).
>
> _Last refreshed: 2026-08-18._

## What this is

**Samavesh** = a unified, role-based digital platform for **Samavesh Action For Impact** (education NGO, Pune — "from the margins to the mainstream"), built by **Dhwani Rural Information Systems**. It digitises the student scholarship lifecycle plus a parallel mentorship/LMS journey, replacing Google Forms + Excel + WhatsApp.

- **Tech target:** Frappe Framework v15 + Frappe LMS, MariaDB, AWS Mumbai (ap-south-1, DPDP residency), S3 docs, PWA. English only in Phase 1.
- **Roles:** Student, Fellow (the primary operator), Mentor, Program Admin, Super Admin.
- **Delivery:** 4 phases (~14–16 wks): P1 discovery/wireframes → P2 core MVP (onboarding/eligibility/docs/applications) → P3 mentorship + LMS → P4 MIS/UAT/go-live.

## The deliverable: the wireframe

Single self-contained file **`wireframe/index.html`** — a clickable prototype, treated as the **source of truth for development** (everything must be clickable, not a throwaway mockup).

- **Live:** https://samarthdris.github.io/Samavesh-WebApp/wireframe/ (GitHub Pages from `dev` root; the repo-root `index.html` redirects there).
- **Fallback (no build):** [htmlpreview](https://htmlpreview.github.io/?https://github.com/samarthdris/Samavesh-WebApp/blob/dev/wireframe/index.html)
- **Architecture:** one login + multiple role app-shells; only the logged-in role renders (`.hidden` toggling). Each role = `#xApp` + `#xBot` (bottom-nav). JS router `go(id)` shows/hides `.screen` sections (unique ids per app). Demo creds in the JS `ACCOUNTS` object + one-tap demo buttons. To add a role: new `#xApp` + `#xBot` + an `ACCOUNTS` entry + a demo button.
- **Deep links (used for verification):** `index.html?role=student|fellow|admin|mentor&screen=<screenId>` auto-logs-in and navigates.
- **Demo logins:** Student `9800000021` (Aarti Pawar) · Fellow `9800000011` (Rahul More) · Program Admin `9800000001`.

### Visual target

Custom-branded **Frappe UI** (the modern Frappe CRM/Helpdesk "espresso" look), **not** stock grey Desk — wearing Samavesh **teal + marigold**. Student = Frappe **Web Portal** (friendly, public-facing). Fellow + Admin = Frappe **Desk** (`.fd` scoped theme: workspace sidebar, list views with dot-style status indicators, sectioned form views, number-card dashboard). Keep the Frappe-credible signals: Inter font, dot indicators (not chips), list rows, sidebar + navbar shell.

## Roles — who does what

- **Student** = read/transparency surface, plus self-onboarding. Sees journey, scholarship status, document status + history, self-service mentor booking, LMS placeholder, read-only profile, and a read-only view of the onboarding form they submitted (with a "request a change" action). Fills **only** the onboarding form.
- **Fellow** = the operator. Approves student self-onboarding, onboards students directly (fills the form on their behalf), procures/uploads documents, fills the scholarship data-entry form, follows up on documents and applications, records disbursement. Also: personal attendance punch, student notes, WhatsApp contact.
- **Program Admin** = verifies documents (Accept/Reject — the action neither student nor fellow does), manages the scholarship catalogue & KPIs, sees MIS dashboards, manages users, supervisory attendance view.
- **Mentor** = self-service mentee sessions (role app exists; deliberately excluded from BRD v4.0's as-built recall).
- **Super Admin** = defined in the BRD (RBAC-06), **role app not yet built** — the next major piece.

## Document lifecycle (BRD DOC-03 + workflow)

`Pending → Uploaded → Under Review → Accepted` (or `Rejected` → re-upload, tracked by *Re-upload Version*). Procurement is folded into **Pending** (the Fellow procures). Only when ALL docs are Accepted can the application be submitted (BR-05). Student sees status + history; Fellow uploads; **Admin verifies**.

- **Sub-documents:** only 4 main docs have them — Ration (6) · Caste (9) · Domicile (7) · Income (11). The sub-doc list stays locked until the student marks **"Don't have"** on the main doc. Have it / Don't have are **mutually exclusive** (`setHave()`), and only appear on docs the student must still provide.
- **Fellow can download any document on record at any time** — status never gates download.
- **Whose-turn chips** on each student doc: Complete / With reviewer / Action needed.
- **File limits — 3 distinct contexts, do not collapse them:** document vault (certificates + sub-docs) = **PDF only, 1–2 MB**; the submitted application PDF = **PDF, ≤10 MB**; proof of Benefits Received = **PDF/JPG/PNG, ≤10 MB**.

## Status model — three distinct axes

**1. Student journey funnel — 5 stages** (disbursement is a sub-status of the terminal stage, not its own):

`Onboarded → Eligibility Identified → Documents Complete → Submitted → Decision`

**2. Application status — the Form/SOP Q33 vocabulary, canonical on every surface** (student, Fellow, Admin):

`Ready to Submit → Under Scrutiny → Application Approved · funds awaited → Benefits Received` · plus `Re-apply` · `Rejected`

> The forms win over BRD §11. Do **not** use "Under Review / Approved / Disbursed / Redirected for Correction" as application statuses anywhere. "Under Review" is a **document** status only; "Redirected for Correction" = `Re-apply`.

**3. Document status** — the DOC-03 ladder above.

## Applications caseload (the tracking model)

A **case = one student × one scholarship** — the same entity as an "application". The old separate "My Cases" screen and its second work-state axis were **retired** (2026-07-06) as a confusing double-update; there is now ONE unified Applications caseload.

- **One status ladder**, advanced as a byproduct of real work (Submit & record → Under Scrutiny; Record disbursement → Benefits Received).
- **Two manual exception flags** — the only things a Fellow hand-sets, because the ladder can't infer them: **On Hold** (+ reason) and **Discarded** (+ reason). Set via inline reason forms, never `prompt()`.
- **Derived triage** (a computed filter, never a stored field): *Action needed* · *Awaiting* · *Done* · *On Hold* (overrides).
- Surfaces: Fellow `#f-applications` (rail badge = "Action needed" count), Admin `#a-tasks` (cross-Fellow, read-only status, Reassign only), Fellow dashboard caseload cards (deep-link pre-filtered), per-student Applications tabs.
- Key JS: `CASES`, `CASE_STATUS`, `caseAttention`, `casePill`, `renderCaseList(scope)`, `filterCaseList`, `caseAdvance`, `casePendReason` / `caseSaveReason`, `caseResume`, `caseReopen`, `caseReassign2`.
- **Production data model (Frappe):** one `Case` DocType — `student`, `scholarship`, `assigned_fellow`, `status` (8-step Select = single source of truth), `on_hold` (Check) + reason, `discarded` (Check) + reason. `attention` is computed, never stored.

**Design principle worth reusing:** make status a byproduct of the real work, and let the user hand-set only what the workflow can't infer. Single writer, derive the rest.

## Form pattern (canonical — apply to every form)

Validated 2026-06-09 across all three Fellow forms. The multi-step "Continue" wizard was **rejected** by the client's director; real Frappe DocType forms are single-page with section breaks.

1. Container `<div class="dt-form" id="xForm">`; small forms use `.dt-form.simple`.
2. **Horizontal sticky pill-tab bar at the TOP** of `.dt-body` (`<aside class="dt-anchors">` — semantic name kept for compat), numbered links, scroll-spy highlight, click smooth-scrolls. Not a left sidebar.
3. Body = `<section class="dt-section" id="sec-x">` blocks with `<h3 class="dt-sec-title">`; `<div class="dt-grid">` for 2-col, `.q.full` for full-width.
4. Sticky bottom action bar `.dt-actions` (Save draft + Submit). No "Continue" buttons anywhere.
5. Conditional branches: `data-branch="A"` sections toggled with `.hidden`, matching anchor links toggled too.
6. **Field conversions (mandatory):** age dropdown → **DOB picker + auto-age chip**; mobile → **verify chip** (validated on blur, Indian 10-digit starting 6–9); gender → dropdown from the shared `GENDER_MASTER`; **≥3 options → `<select>`**, only strictly-2-option questions stay radios; long grouped lists → `<select>` with `<optgroup>`.

**The onboarding form is ONE source of truth.** The same `#onbForm` markup renders three ways: Fellow fill, student self-fill (`buildStudentOnboard()`, `s_` prefix), and student read-only (`lockOnbClone`, `myonb_` prefix). Never build a separate simplified student copy.

## Canonical demo data — Aarti Ramesh Pawar (SMV-2026-01182)

⚠️ Her data lives in **multiple independent places** (static HTML per screen + the JS arrays `CASES` and `MY_ONBOARDING`). Every new surface adds another copy, so values drift. The 2026-07-06 audit hand-aligned them; the architecture was **not** refactored. **When you add or edit any student-facing surface, update all of her copies.**

- DOB **14 Aug 2005** · Category **SC** · Religion Buddhist · **B.Com, Year 2** · income **₹1,80,000** · Maharashtra / Pune · marks **78%** (prev 74%) · mobile …21 · Aadhaar …4821 · Guardian Ramesh Pawar (Father).
- **4 applications:** Post-Matric SC = *Under Scrutiny* · Maharashtra State Minority = *Ready to Submit* · Dr. Babasaheb Ambedkar = *Documents Pending* · Rajarshi Chhatrapati Shahu Maharaj Merit = *Re-apply*. These must match across `CASES`, `fp-apps`, `fp-sch`, and the student Scholarships tab.
- **Documents:** 9-doc checklist → 5 Accepted / 1 Under Review / 1 Pending / 2 N/A. Home tile reads **"5 of 9"**.
- My Students reads **"Showing N of 24"** (`#fStuShown`, incremented by `bumpStudentCounts`).

**Reusable pattern — generic inline profile edit:** `editProfile(btn)` / `saveProfile(btn)` / `cancelProfile(btn)` work on any `.card` with a `.field/.k/.v` grid, flipping `.v:not(.mask):not([data-noedit])` into inputs and flashing "✓ Saved · audit-logged" on save. Add `data-noedit` (or `.mask`) to make a value non-editable.

## Verifying (do this before saying done)

Structural checks (tag/brace balance) pass on broken features — a filter that only updates a note, a screen built on a stale tab. Render it and click it.

> ## ⛔ NEVER kill Chrome by name
>
> Do **not** run `pkill -f "Google Chrome"`, `killall Chrome`, `taskkill /IM chrome.exe`, or any other
> kill-by-name. On a working machine that closes the developer's **own browser window and tabs** — it
> happened repeatedly on 2026-08-21 and Shweta had to restart Chrome each time.
>
> Kill **only the PID you launched yourself**, and always pass a throwaway `--user-data-dir` so the real
> profile is never touched. The macOS recipe below does both.

### macOS recipe (verified 2026-08-21 — use this one on a Mac)

```bash
SP=/tmp/samavesh-render                    # scratch dir outside the repo
mkdir -p "$SP" && rm -rf "$SP/prof"
cp "wireframe/index.html" "$SP/swf.html"   # repo path has spaces, which breaks file:// URLs

"/Applications/Google Chrome.app/Contents/MacOS/Google Chrome" \
  --headless=new --disable-gpu --no-sandbox --hide-scrollbars \
  --user-data-dir="$SP/prof" --window-size=1440,2200 --virtual-time-budget=8000 \
  --dump-dom "file://$SP/swf.html?role=fellow&screen=f-attendance" > "$SP/dom.html" 2>/dev/null &
CHROME_PID=$!                              # <- the only process you may kill

until grep -q "PROBE|" "$SP/dom.html" 2>/dev/null; do sleep 2; done
kill "$CHROME_PID" 2>/dev/null             # never pkill, never killall
grep -o 'PROBE|[^<]*' "$SP/dom.html"
```

Swap `--dump-dom … > dom.html` for `--screenshot="$SP/shot.png"` and poll with `until [ -s "$SP/shot.png" ]`
when you want a picture instead of a probe. `--headless=new` does not always exit on its own, which is why
the PID is captured — not a reason to reach for a broad kill.

**One more macOS gotcha:** `getGeo()` (used by `fellowPunch`) waits on `navigator.geolocation` with a
**2-second** timeout before falling back to a static Pune location, so anything asserting on a check-in
must wait **> 2.5 s** after the click. A shorter wait reports a working punch as broken.

### PowerShell recipe (Windows, verified earlier on this project)

1. `Copy-Item wireframe/index.html C:\Temp\swf.html -Force` — the repo path has spaces, which breaks `file://` URLs.
2. Close only Chrome instances you started yourself — see the warning above; do not blanket-kill.
3. Render:

```powershell
& $chrome --headless=new --disable-gpu --no-sandbox --hide-scrollbars `
  --user-data-dir="C:\Temp\cprof" --window-size=1440,2200 --virtual-time-budget=4000 `
  --screenshot="C:\Temp\shot.png" `
  "file:///C:/Temp/swf.html?role=fellow&screen=f-attendance" 2>&1 | Out-String
```

**Flags that matter:** `--headless=new` (plain `--headless` silently produces nothing on this Chrome) · an isolated `--user-data-dir` (a locked default profile fails silently) · `2>&1 | Out-String`, not `2>$null` · `--virtual-time-budget=4000` so `.stagger` entrance animations finish, otherwise cards look faint.

**For assertions on values, counts, or class flips**, prefer a DOM probe over eyeballing a screenshot: inject a script that writes results into `<div id="probe">`, then `--dump-dom | Select-String "PROBE"`. Note that `go(screen)` builds some screens on a `setTimeout(0)` — defer probe checks ~300 ms after calling `go()`, or you get false negatives.

Several `--screenshot` runs in one PowerShell block execute sequentially, which is fine. Avoid *concurrent* Chrome.

## Source-of-truth documents (in repo)

| File | What |
|---|---|
| `SAMAVESH_BRD_v4.0.md` | **Current BRD** (2026-07-27) — as-built + roadmap, pitch-ready, 25 sections. §25 maps every feedback tracker ID to its resolution. |
| `Samavesh_BRD_v3.0 copy.docx` | Prior BRD, superseded by v4.0 but retained. |
| `Onboarding Form MD File.md` | Onboarding form spec — 42 Q, 8 steps, document matrix, consent gate. |
| `SAMAVESH_SCHOLARSHIP_DATA_ENTRY_ANALYSIS_AND_SPEC.md` | Scholarship Data Entry spec — 46 Q, branched at Q25, masked credentials, 2×2 installment grid. |
| `Feedbacks.xlsx` | **The authoritative client feedback tracker** (arrived 2026-07-03), 11 substantive items. |
| `Scholarships Eligibility Criteria … .xlsx` | Scheme eligibility criteria, all schemes. |
| `Sub-Documents List/`, `Login Logout Fellow/` | Client reference screenshots. |
| `Samavesh workflow.pdf` + the two Google-Form PDFs | Source references. |
| `docs/superpowers/{specs,plans}/` | Per-feature design specs and implementation plans — the worked examples to match. |

## Feedback tracker state (`Feedbacks.xlsx`)

11 substantive items: General (2) + Student (4) + Fellow (5). **All Admin screens (IDs 13–18) and Student Profile (ID 6) received no feedback** — the client is happy with them as they are.

- **Batch 1 — done & live** (IDs 1, 3, 4, 5): email-OTP sign-up toggle; Student Home reorder + dual WhatsApp buttons (assigned Fellow + central helpline); a unified "My Scholarships" list including eligible-but-not-applied schemes; sub-documents.
- **Batch 2 — done & live** (IDs 7, 10, 12 + global English-only): Fellow attendance; student self-onboarding with document to-and-fro; Fellow notes. ID 8 was dropped; ID 2 was a dev-time removal.
- **Parked / next: ID 11** — reflect the *Scholarship Data Entry* form on the student view. **Landmine:** show the resulting application record and status, **never the government-portal credentials**.

## Hard rules

The client-locked decisions live in [`../CLAUDE.md`](../CLAUDE.md) in full. Summarised: only context-file vocabulary; nothing half-baked; review gates show the whole artifact; ripple-check every new pattern; verify behaviour, not structure; desktop only; keep Aarti's data consistent.

Plus:

- Documents in the vault are **PDF only, 1–2 MB**.
- Mentorship = **self-service booking**; the student self-books. LMS = the §13.6–13.16 curriculum model — currently a **nav placeholder**, not built.
- Specific scholarship and college names in the wireframe are **sample placeholders** — the real catalogue is open item **OI-01**, still awaited from Samavesh.

## Git & deployment

- **Canonical repo:** `samarthdris/Samavesh-WebApp` (public). A mirror exists at `samarthrana/Samavesh-WebApp`; it is not the working repo.
- **`dev` is the live branch** — GitHub Pages deploys from `dev` root. It is protected: work on `feature/*` and open a PR into `dev`.
- `main` is dormant, behind `dev`, and deploys nothing. No PRs to `main` unless asked.
- One push per feature — two pushes inside a minute make the first Pages deployment report a (benign) failure.

## Open questions parked for the client

- The scholarship catalogue (OI-01) — real scheme and college names.
- The Documentation Application form has **no dedicated spec** — currently a derived design; confirm.
- Form-level questions from the MD specs: the "Binary" gender option, duplicate scheme names, single-installment handling, and plaintext-password storage in the data-entry form.

## Backlog (confirm scope before starting any of these)

1. **Feedback ID 11** — Scholarship Data Entry on the student view (credentials landmine above).
2. **Super Admin role app** — the last unbuilt role. Scope from BRD RBAC-06 + §7: user & role-profile management, system configuration, data administration (incl. the DPDP deletion workflow), audit log. Mostly list/config Desk (`.fd`) surfaces, not a program-operations seat.
3. **LMS** — currently a student nav placeholder; the resolved scope is the §13.6–13.16 curriculum model.
4. Fellow rail nav could gain `f-scholarship` / `f-docapp` (today reached contextually).
5. `workflow.html` niceties: an optional root choice page (Wireframe vs Workflows), minor wording, Mentor bio as a `<textarea>`.

## Known watch-items

- **Data duplication** (above) — values were aligned, the architecture was not. If drift recurs, single-source per student.
- The Scholarship Data Entry form's `onAppStatus` dropdown still reads "Application Approved" (a separate surface, was out of scope).
- Audit item B1, the "Join (link soon)" rename — skipped deliberately.
