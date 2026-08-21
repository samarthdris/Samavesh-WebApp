# Financial Tracking — Installment-wise Disbursement + Proof Upload — Design

**Date:** 2026-08-20
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` only. Adds a new tab to Fellow (`#f-student`) and Admin (`#a-student`) student-detail, a read-only block to the Student's Scholarships screen (`#scholarships`), and (see "The duplication problem") wires the Data Entry form's existing §7A grid to the same data.
**Source:** `New Feedbacks/Point 4_Financial_Tracking_Section.md` ("Point 3 — Financial Tracking Section", "Module 5" in the client's master FRD). In Scope, **High** priority. Placement/visibility/vocabulary decisions below confirmed with Shweta on 2026-08-20.

## Problem

Scholarship money arrives in stages and to two different recipients — the student directly, and the institute on the student's behalf. The programme needs auditable proof per installment per recipient, because "application approved" is not the same as "funds actually landed". The MIS "₹ Unlocked" column and the Consolidated Profile Summary both depend on this data existing.

**Correction to the client note.** The feedback states "Not built. No financial/disbursement tracking currently exists." That is not accurate against the current wireframe. What exists today:

| Already in the file | Where | State |
|---|---|---|
| §7 Financial Tracking — the 2×2 installment grid (Student/Institute × First/Second Installment), amount + status per cell | Scholarship Data Entry form, `#sch-fin` (line 3411–3426), plus its form-nav link (line 3197) | Renders, but **inert** — typing in it changes nothing anywhere |
| "Proof of Benefits Received" upload | Same form, `#benefitsProofQ` (line 3407) | One shared field for the whole application, revealed only when status = Benefits Received |
| `recordDisbursement()` — validates amount + date, flips the pill to Benefits Received | JS line 5296 | **Dead code** — defined, never called from any markup |
| "₹ Unlocked" MIS column · "2 scholarships · ₹71,000" | Admin line 4060 · Student-list line 2376 | Hardcoded |

So this module is an **extend**, and the genuinely missing pieces are: a **date** per entry, **proof per entry** (not one per application), the **Approved gate**, the **mandatory/optional split**, a place to see it **outside the entry form**, and above all **state** — so that what a Fellow types is visible to anyone else.

## Goal (confirmed with Shweta)

- A **separate "Financial Tracking" tab** on the application detail view, as the feedback literally describes — *not* an extension of the data-entry form's section. Shweta chose this shape explicitly on 2026-08-20 after the duplication risk below was raised.
- **Fellow enters. Admin and Student see it read-only.** (Matches Modules 2 and 3, and the student is already shown ₹ figures elsewhere in the product.)
- **Locked vocabulary.** First / Second Installment · Student / Institute · the three existing statuses: *Awaiting Approval · Fund Disbursement in Process · Funds Disbursed*.
- Up to **4 entries per application**: 2 installments × 2 levels, each carrying amount, date, proof, status.
- **First Installment mandatory** (amount + date + proof) once the section is active; **Second Installment optional**, always available, never validated.
- Section is **inactive until the application is Approved** — shown as a plain locked message, never an error.

## Vocabulary — three deliberate deviations from the feedback wording

The feedback's words and the client-locked documents disagree. Hard rule 1 says the locked documents win, and Shweta confirmed this on 2026-08-20:

| Feedback says | We build | Why |
|---|---|---|
| Tranche 1 / Tranche 2 | **First / Second Installment** | "Tranche" appears **zero** times in the BRD, form specs, SOP or wireframe. Q36–Q43 of the Data Entry spec are all "Installment". |
| Institution | **Institute** | The locked sources and the existing grid say Institute. |
| Pending / Disbursed | **Awaiting Approval · Fund Disbursement in Process · Funds Disbursed** | The three-value set is already in the form spec (Q37/39/41/43) and on screen. Collapsing to two would drop values the client has already reviewed. |

Everything else in the feedback is adopted as written. One item is an improvement on the locked spec and is adopted gladly: making the Second Installment optional **resolves an open issue the Data Entry spec itself raised** (its issue #14 / open question #5 — all 8 installment fields marked required, with no way to record a single-installment scheme).

## Non-goals

- Not making the Admin "₹ Unlocked" column or the student-list "₹71,000" compute from this data. Those are hardcoded and belong to a **different** student (Asha Kamble) — they don't contradict Aarti's numbers. Logged as a watch-item, not fixed here.
- No redesign of the Data Entry form's layout, and no change to the branch logic (Path A/B) that reveals §7A.
- No new status values, no new application statuses, no change to who may edit anything else.
- No Frappe build — wireframe mock only.

## One shared data source, four applications, three surfaces

Same engine-plus-hosts pattern as Modules 2 and 3, keyed on the `data-app` values that already exist on every surface (`pm`, `ms`, `ab`, `smm`):

```
FIN = { pm: { s1:{amount,date,status,proof}, i1:{…}, s2:{…}, i2:{…} }, ms:{…}, ab:{…}, smm:{…} }
       s = Student level, i = Institute level; 1 = First Installment, 2 = Second
```

| Surface | Container | Behaviour |
|---|---|---|
| Fellow — new tab `#fp-fin` | one block per application | Full 2×2 grid, editable, proof upload, Save per installment |
| Admin — new tab `#ap-fin` | one block per application | Same grid, read-only, proof viewable, no Save |
| Student — `#scholarships` | `.fin-host[data-app]` in each card's `.sch2-side`, beside the Module 3 PDF row | Compact read-only summary: amount + status + date per recorded entry |

All 4 applications get a block, not just the demo one — ripple-check rule.

## The Approved gate, and what the client will actually see

The gate is `Application Approved · funds awaited` **or** `Benefits Received`. Anything earlier shows: *"Financial Tracking unlocks once this application is Approved."*

**None of Aarti's 4 applications is currently at or past Approved** — PM is Under Scrutiny, MS Ready to Submit, AB Documents Pending, SMM Re-apply. So on first load every block is legitimately locked, which is the honest depiction of her story but shows the client an empty feature.

**Decision:** demonstrate the gate live rather than fake the data. PM's Applications panel already has an "Update status…" dropdown (line 2984) containing "Application Approved · funds awaited". Moving PM to that status unlocks its Financial Tracking block in-session, the Fellow fills the First Installment, and the Student and Admin views immediately show it. Four clicks, nothing hardcoded, and it demonstrates the gate itself rather than hiding it.

**Alternative if Shweta prefers a pre-filled demo:** promote PM to Approved permanently, or add a 5th Approved application. Both are bigger ripples — PM's status is woven through its timeline, pill, caseload row and Module 3 PDF row on three surfaces, and Aarti is the canonical demo student (hard rule 7). Not recommended, but it's a one-word decision.

## The duplication problem — raised, and how it's contained

Shweta's chosen shape means money can be typed in **two** places: the Data Entry form's §7A grid and the new tab. Two independent grids for the same four numbers is exactly the drift the July audit flagged as a watch-item.

**Containment (needs Shweta's confirmation):** leave §7A **visually untouched** — same position, same layout, no redesign — but have its inputs read and write the same `FIN` object the new tab uses. The two views then physically cannot disagree, because there is only one set of numbers underneath. This is the only edit proposed to the form, and it is behavioural, not visual.

If Shweta would rather §7A stay fully inert, that's her call — but then the wireframe ships two grids that show different numbers, and the demo has to avoid one of them. Flagging, not deciding.

Also: `recordDisbursement()` (line 5296) is dead code with exactly the validation this module needs. The new tab reuses it rather than writing a second validator.

## UI

- **Block per application:** scheme name + current status pill as the header, then the 2×2 grid — rows Student / Institute, columns First / Second Installment.
- **Cell:** ₹ amount · status dropdown (3 locked values) · disbursement date · proof (upload for Fellow; "View proof" link for everyone once attached). Reuses the existing `.instgrid` / `.amt` / `.rs` styles so it looks native.
- **Mandatory:** First Installment cells marked `*`; Save refuses with a plain message naming the missing field. Second Installment carries no `*` and never blocks.
- **Empty:** "No disbursement recorded yet" — not an error.
- **Locked:** "Financial Tracking unlocks once this application is Approved." **Exception, found during render-verification:** on the Student's card this message rendered a bordered block repeating the scheme name and status already shown at the top of the same card — a hard-rule-2 duplication. The Student's compact view therefore renders **nothing** while locked; the message stays on the Fellow and Admin tabs, where the block is not a duplicate of its surroundings.
- **Student's compact view:** one line per recorded entry — "First Installment (Student): ₹25,000 · Funds Disbursed · 12 Jul · View proof". Nothing shown for un-recorded entries.

## Production note (Frappe) — carried forward from the client spec

New child table **"Disbursement Entry"** on the Scholarship Application: `tranche` (Select), `level` (Select Student/Institute), `amount` (Currency), `disbursement_date` (Date), `proof` (Attach, image or PDF), `status` (Select). Relationship 1 → 0..4. No `has_second_tranche` flag — the Second Installment is simply optional. Server-side: the `validate` controller hook enforces First-Installment completeness and the file-type restriction; visibility uses a `depends_on` on the parent's approved status. Note the DocType field names keep the client's `tranche` wording even though the UI labels read "Installment" — worth confirming at build time which layer the client cares about.

## Verification (render-verify)

1. Fellow `?role=fellow&screen=f-student` → **Financial Tracking** tab — 4 blocks, all locked with the unlock message; no grid editable.
2. Same screen → Applications tab → PM "Update status…" → "Application Approved · funds awaited" → back to Financial Tracking: PM's block is now unlocked and editable, the other three still locked.
3. Fellow fills First Installment (Student) amount + date + proof → Save. Then try saving with the date cleared → refuses, names the field. Second Installment left empty → saves fine.
4. Student `?role=student&screen=scholarships` — PM's card shows the recorded entry read-only; the other three cards show nothing added.
5. Admin `?role=admin&screen=a-student` → **Financial Tracking** tab — same numbers as the Fellow entered, read-only, no Save button and no upload control anywhere.
6. Data Entry form `?role=fellow&screen=f-scholarship` → §7 Financial Tracking — the grid shows the same numbers (if the containment above is approved), and editing there is reflected back on the three surfaces.
7. Grep consistency: `.fin-host` count matches 4 applications × the surfaces that use hosts; exactly one editable grid per application on the Fellow surface and zero on Admin/Student.
