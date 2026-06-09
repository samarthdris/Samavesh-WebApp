# Onboarding Form Redesign — Design Spec

**Date:** 2026-06-09
**File touched:** `wireframe/index.html` (Fellow section only — `#f-onboard` screen and its `#onbForm` formwrap)
**Scope:** Convert Form 1 (Student Onboarding) from an 8-step "Continue"-driven wizard into a **single-page Frappe DocType form** with a left section-anchor sidebar, applying every Frappe-specific UX pattern the user called out. **Forms 2 (Scholarship Data Entry) and 3 (Documentation Application) are NOT touched in this round** — they continue to use the existing `.fstep`/`formGo` infrastructure.

**Reference (authoritative):**
- Director feedback (2026-06-09): five screenshot-anchored changes — search by email/name only, DOB→age auto-calc, mobile verify chip, gender master, document child-table with Must-Have/Good-to-Have groups, all radio-lists with ≥4 options → dropdowns, eliminate Continue stepper.
- `frappe-doctype-skill`: fieldtype reference (Date, Phone, Link, Select, Table), Section Break / Column Break layout pattern, master DocType pattern.
- `frappe-dashboard-design`: light theme, 4px spacing, Inter type, status chip mappings, button hierarchy.
- BRD v3.0 §10.1 (Form fields), BR-01 (duplicate-student rule).
- `Onboarding Form MD File.md` (the 42-Q source-of-truth).
- Memories: [[samavesh-forms-spec]], [[samavesh-frappe-ui]], [[samavesh-web-focus]] (desktop only — no mobile polish this round).

## Problem / goal

The current onboarding form (`#onbForm`, lines 1717–1874) is a Google-Form-style 8-step wizard with a "Continue" button between every section. Director feedback:

1. **Multi-step "Continue" is hostile UX** — the Fellow has to click 7 buttons across 8 screens to finish one form. Frappe's actual DocType forms are single-page with **collapsible Section Breaks** and **Column Breaks** for 2-column layouts — much faster to scan and fill.
2. **Radio-button lists with 4+ options are wasteful** — Social Background (9 options), Annual Income (7 options), Stream (10), etc. each occupy 250–500 vertical pixels of radio buttons. The Frappe equivalent (Select dropdown) takes 40px.
3. **Age dropdown is a calculation Frappe can do for free** — given DOB, age in years (decimal) is derived. Storing both is duplication; storing DOB and computing age is the canonical pattern.
4. **Mobile field has no validity feedback** — the Fellow can type "123" and proceed. Frappe v16's `Data` fieldtype with `options: "Phone"` does format validation; we mock the same client-side.
5. **Gender as a free-radio is brittle** — every form that asks for gender re-defines the same 5 options. Single source of truth = a Frappe **master DocType** that all forms Link to.
6. **Document matrix lacks priority signal AND attachment** — a Fellow can't tell which docs block submission vs which are scholarship-conditional, and there's no way to upload during the Have-it path.
7. **Duplicate check anchored to Mobile is now wrong** — SSO login made email the primary identifier (per [[samavesh-brd-discrepancies]]); the dup-check should be email-first.

## Decisions (user, 2026-06-09)

- **Single-page form** with **collapsible Section Breaks** (8 sections kept as anchors, not steps).
- **Left section-anchor sidebar**, sticky, scroll-spy highlights current section.
- **No "Continue" buttons.** Single "Save draft" + "Submit Onboarding" at the bottom.
- **Two-column layout within sections** wherever fields naturally pair (Frappe Column Break pattern).
- **DOB picker** replaces Age dropdown; age (years, 1-decimal) shown read-only beside it.
- **Mobile** gets a ✓/✗ chip on the right; validates Indian 10-digit format (starts with 6/7/8/9) on blur.
- **Gender** becomes a single dropdown driven by a **Gender master** (5 entries: Male/Female/Non-Binary/Other/Prefer not to say, codes M/F/NB/O/PNS).
- **All single-select questions with ≥4 options** convert from radio lists to dropdowns. 2–3 option questions stay as radio buttons (cheaper scan).
- **Document matrix** restructured as a **child-table** with two grouped tbodies: **Must Have (Mandatory)** and **Good to Have (Scholarship-specific)**. Each row has Document name + Status dropdown (Have it / Don't have / N/A) + Attach control (📎 file input, enabled only when status = "Have it"). Status renders as a **colored chip** in saved-row preview (green / amber / gray per dashboard-design).
- **Duplicate-check** search bar simplified to **Email OR Name** (mobile no longer the primary key).
- **Mobile-viewport polish out of scope** (per [[samavesh-web-focus]]). Existing responsive CSS rules preserved but not enhanced.

## Architecture & wiring

### Form surface (Frappe DocType pattern)

```
#onbForm.dt-form (replaces existing .formwrap stepper)
├── .dt-anchors           ← sticky left sidebar, anchor list
│    ├── a[href="#sec-support"]  Welcome & Support
│    ├── a[href="#sec-personal"] Personal Details
│    ├── a[href="#sec-location"] Location
│    ├── a[href="#sec-college"]  College & Reference
│    ├── a[href="#sec-edu"]      Education
│    ├── a[href="#sec-docs"]     Documents
│    ├── a[href="#sec-socio"]    Socio-economic
│    └── a[href="#sec-consent"]  Aspirations & Consent
├── .dt-body              ← scroll container
│    ├── section#sec-support     .dt-section
│    │    ├── h3 .dt-sec-title   "Welcome & Support Type"
│    │    ├── .dt-grid           (2-column or single, per content)
│    │    │    └── .q × N (existing bilingual question blocks, restyled)
│    │    └── (no Continue button)
│    ├── section#sec-personal    ... (similar)
│    │    ├── DOB picker + auto-calculated age (read-only inline)
│    │    ├── Mobile w/ validate chip
│    │    ├── Gender dropdown (Master-driven)
│    │    └── ...
│    ├── section#sec-docs        ... (special — uses .doc-ctable, see below)
│    └── ... (5 more sections, all under one scroll)
└── .dt-actions           ← sticky bottom bar
     ├── .dt-act-meta     "8 of 8 sections · 0 blocking validations"
     ├── button.btn-ghost "Save draft"
     └── button.btn-primary "Submit Onboarding"
```

The 8 sections from the existing form are preserved (no loss of fields), just rendered continuously. The user scrolls; the sidebar shows current position via scroll-spy.

### Documents child-table (the headline change)

```
#sec-docs
├── h3 "Documents"
├── .dt-help "Mark availability and attach if you have the document in hand."
├── .doc-ctable                ← Frappe child-table styling
│    ├── thead
│    │    ├── th  Document
│    │    ├── th  Status
│    │    └── th  Attachment
│    ├── tbody.doc-group       ← MUST HAVE / MANDATORY
│    │    ├── tr.doc-group-h   "Must Have / Mandatory"  (subhead row, full-width)
│    │    ├── tr × 4           (Aadhar / Annual Income Cert / 10th SLC / 12th SLC)
│    │    │    ├── td.doc-name "Aadhar Card"
│    │    │    ├── td.doc-stat <select Status>  + chip preview
│    │    │    └── td.doc-att  <input type="file"> (disabled until Status="Have it")
│    │    └── ...
│    └── tbody.doc-group       ← GOOD TO HAVE / SCHOLARSHIP-SPECIFIC
│         ├── tr.doc-group-h   "Good to Have (scholarship-specific)"
│         ├── tr × 5           (Caste / Non-Creamy Layer / Domicile / Ration / Orphan)
│         └── ...
└── .dt-help.note "Any document marked 'Don't have' becomes a Documentation Application task for the Fellow to procure."
```

Status-chip mapping (per `frappe-dashboard-design` §6.2):

| Status | Chip class | Color |
|---|---|---|
| Have it | `.chip.active` | `#DCFCE7` / `#15803D` |
| Don't have | `.chip.pending` | `#FEF3C7` / `#B45309` |
| N/A | `.chip.inactive` | `#F3F4F6` / `#374151` |

### Default categorisation

| Must Have (4) | Good to Have (5) |
|---|---|
| Aadhar Card | Caste Certificate (scholarship-dependent) |
| Annual Income Certificate (FY 25/26) | Non-Creamy Layer Certificate (scholarship-dependent) |
| 10th School Leaving Certificate | Domicile Certificate (Maharashtra-domicile scholarships) |
| 12th School Leaving Certificate | Ration Card |
| | Orphan Certificate (only if applicable) |

### DOB + auto-age

```
.q.dob-q
├── .qen "Date of Birth"        .qmr "जन्मतारीख"
├── .dob-inline
│    ├── input#onbDob type="date"      (Frappe Date fieldtype)
│    └── .dob-age                       (read-only chip)
│         └── "Age: 18.5 years"        ← live-computed from DOB
└── .qhelp "Age in years auto-calculated."
```

Compute: `age = (today - dob) / 365.25`, formatted with 1 decimal place. Computed on input/change event.

### Mobile validate

```
.q.mobile-q
├── .qen "Contact Number (preferably WhatsApp)"  .qmr "..."
├── .mobile-input
│    ├── input#onbMobile type="tel" maxlength="10"
│    └── .mobile-verify       (positioned absolute right; hidden by default)
│         └── ✓ Valid          (or ✗ Invalid)
└── .qhelp "..."
```

Validation regex: `/^[6-9]\d{9}$/` (Indian 10-digit mobile starting with 6/7/8/9). Runs on `blur` and re-runs on `input` if the chip is currently showing.

### Gender master

```js
// Mocked in JS as a static list — production would Link to the Gender DocType.
const GENDER_MASTER = [
  {code:'M',   label_en:'Male',           label_mr:'पुरुष'},
  {code:'F',   label_en:'Female',         label_mr:'स्त्री'},
  {code:'NB',  label_en:'Non-Binary',     label_mr:'नॉन-बायनरी'},
  {code:'O',   label_en:'Other',          label_mr:'इतर'},
  {code:'PNS', label_en:'Prefer not to say', label_mr:'सांगू इच्छित नाही'}
];
```

Rendered as a single `<select>` populated from this array. Same array becomes the future Frappe DocType "Gender" — codes match, labels match. Other forms (Scholarship Data Entry, Mentor profile) will eventually consume this same array.

### Section anchor sidebar (scroll-spy)

```css
.dt-anchors { position: sticky; top: 80px; width: 200px; align-self: flex-start; }
.dt-anchors a { display:block; padding:8px 12px; font-size:13px; color:var(--ink-soft); border-left:2px solid transparent; }
.dt-anchors a.on { color: var(--teal-700); border-left-color: var(--teal-700); font-weight:600; background:var(--teal-50); }
```

JS: on scroll within `.dt-body`, find the topmost `.dt-section` whose bounding rect's top ≤ 100px, mark its anchor `.on`.

### Layout — Frappe Column Break equivalents

Where pairs of fields are natural, use 2-column `.dt-grid`:

| Section | 2-col pairs |
|---|---|
| Personal | (Full Name, DOB+Age) ─ (Mobile, Gender) |
| Location | (State, District-Permanent) ─ Permanent Address (full width) ─ "Same as permanent" (full width) ─ Current Address (full width) ─ (District-Current, blank) |
| College & Reference | (College, How heard) |
| Education | (Current Grade, Stream) ─ Current Marks (full width) ─ (Previous Grade, Previous Marks) |
| Socio-economic | (Social Background, Annual Income) ─ Identities (full width) ─ (Single Parent, Parent Disability) ─ (Family Occupation, House Type) ─ (Drug-addiction, WiFi) ─ Computer (full width) |
| Welcome+Support | Email (full width) ─ Support Required (full width) |
| Documents | Always full-width child-table |
| Consent | Always full-width |

### Duplicate-check (top of form)

```html
<div class="dupbox">
  <div class="dt">First, check for an existing profile</div>
  <p>A student can be onboarded only once. Search by <b>email</b> or <b>full name</b> — duplicates are blocked (BR-01).</p>
  <div class="sbar">
    <input placeholder="Enter email or student's full name…" />
    <button class="btn btn-ghost btn-sm">Search</button>
  </div>
</div>
```

## Field-type mapping (Frappe DocType layer)

| Field | Old | New | Frappe fieldtype |
|---|---|---|---|
| Email | input type=email | input type=email | `Data` with `options:"Email"` (v16) |
| Full Name | input type=text | unchanged | `Data` with `options:"Name"` (v16) |
| Form Submission Date | input type=date | unchanged | `Date` |
| Age | select 1–30+ | **REMOVED** | (derived) |
| DOB | (not present) | **NEW** input type=date + age read-out | `Date` |
| Mobile | input type=tel | input type=tel + validate chip | `Data` with `options:"Phone"` (v16) |
| Gender | radio × 5 | **select dropdown** (master-driven) | `Link` → `Gender` DocType |
| Support Required | checkbox × 2 | unchanged | `Check` (2× Boolean) OR Table MultiSelect |
| State | select | unchanged | `Select` |
| District (Perm/Curr) | select | unchanged | `Select` (would be `Link` → `District` in prod) |
| Address Perm/Curr | textarea | unchanged | `Small Text` |
| College Name | select | unchanged | `Select` (would be `Link` → `College` in prod) |
| How heard | radio × 8 | **select dropdown** | `Select` |
| Current Grade | select | unchanged | `Select` |
| Stream | select | unchanged | `Select` |
| Current Marks | input text | unchanged | `Data` |
| Previous Grade | select | unchanged | `Select` |
| Previous Marks | input text | unchanged | `Data` |
| Documents (9-row matrix) | radio matrix | **child-table** (status + attach, grouped) | `Table` → `Onboarding Document` child DocType |
| Social Background | radio × 10 | **select dropdown** | `Select` (would be `Link` → `Social Background` master in prod) |
| Identities | checkbox × 6 | unchanged (multi-select) | `Table MultiSelect` |
| Annual Family Income | radio × 7 | **select dropdown** | `Select` |
| Single Parent Child | radio × 2 | unchanged | `Check` |
| Parent Disability | radio × 3 | unchanged | `Select` (Yes/No/Maybe) |
| Family Occupation | radio × 6 | **select dropdown** | `Select` |
| House Type | checkbox × 4 | unchanged | `Table MultiSelect` |
| Drug-addiction | radio × 4 | unchanged | `Select` |
| WiFi / Internet | radio × 2 | unchanged | `Check` |
| Computer/Laptop | radio × 2 | unchanged | `Check` |
| Goals / Aspirations / Notes | textarea × 3 | unchanged | `Small Text` × 3 |
| Consent — type name | input text | unchanged | `Data` |

## Visual treatment — `frappe-dashboard-design` rules applied

| Rule | How |
|---|---|
| Light theme + white surfaces | `.dt-form` body uses `var(--paper)` + `var(--shadow-sm)`; sections separated by `border-bottom: 1px solid var(--line)` |
| 4px spacing grid | All paddings/gaps in multiples of 4 |
| Inter type | already declared globally; reuse |
| Type scale | Section title 19px / weight 600; field label 13px / 500; input 14.5px; help 12px |
| Status chips | Document status renders as chip per §6.2 (`.chip.active/.pending/.inactive`) |
| Button hierarchy | Submit = primary teal; Save draft = secondary outlined; Cancel/Back = ghost |
| Subtle motion | Section anchor highlight fades 150ms; mobile verify chip slides in 200ms; smooth-scroll to section on anchor click |
| Frappe sidebar + sub-nav | Form is inside `.fd` (Desk) shell; the dt-anchors are SUB-navigation within the form, not the main rail |

## Out of scope (this round)

- Form 2 (Scholarship Data Entry, `#schForm`) — same patterns will be applied later; keep using `formGo` for now.
- Form 3 (Documentation Application, `#docForm`) — same.
- Other Fellow screens (My Students list, Applications list, etc.) — same patterns will be applied later.
- Other roles' forms (Admin, Mentor) — separate rounds.
- Real Frappe DocType / master creation — production deployment, not wireframe.
- Mobile viewport polish — per [[samavesh-web-focus]].
- Email verification flow (welcome email post-onboarding) — flagged separately, not this round.

## Estimated diff (wireframe/index.html)

| Layer | Lines added | Lines removed |
|---|---|---|
| CSS (new `.dt-*`, `.doc-ctable`, `.mobile-verify`, `.dob-age`, `.dt-anchors`) | ~140 | ~5 |
| HTML (rewrite of `#onbForm` interior + `.dupbox` copy) | ~250 | ~165 |
| JS (DOB→age, mobile validate, gender master, anchor scroll-spy, onbSubmit) | ~70 | ~5 |
| **Total** | **~460** | **~175** |

Big change (~300 net lines added), but single-file, single-feature, surgically scoped to `#f-onboard`.

## Open questions

None. User gave broad authority ("make whatever changes necessary overall as per the frappe build and to improve the user journey and experience"). Defaults documented above.
