# Point 3 — Financial Tracking Section – Image Upload + Transaction-wise Detail Entry
*(Module 5 in the master FRD)*

**Scope Status:** In Scope | **Priority:** High
**Note:** Fully resolved — zero open items.

---

**1. Feature/Module Name:** Financial Tracking Section (Tranche-wise Disbursement + Proof Upload)

**2. Current State (Existing Wireframe Behavior):** Not built. No financial/disbursement tracking currently exists; this is a new section, active once an application reaches Approved status.

**3. Requested Change (verbatim, as confirmed):** "Tranche-wise entry (Tranche 1 mandatory, Tranche 2 non-mandatory) capturing amount, date, and proof of benefit received via image/PDF upload; tracked separately for student-level and institution-level disbursement status."

**4. Business Objective:** Scholarship disbursement often happens in stages (tranches) and to two different recipients (student directly, or the institution on the student's behalf) — the program needs auditable proof and status per tranche per recipient level to confirm funds actually reached their destination, not just that an application was approved.

**5. Detailed Functional Requirements:**
- Section becomes active/visible only when the parent application's status = Approved (`depends_on` condition); hidden/disabled before that.
- Tranche 1: amount, date, proof upload — all mandatory once the section is active.
- Tranche 2: same three fields. Confirmed: **non-mandatory** (optional) — matches the original requirement wording exactly. Confirmed: no scholarship-master flag gates it — Tranche 2 is simply available on every application, and a fellow can choose whether to fill it in.
- Tracking is duplicated across two levels — Student and Institution — each with its own amount/date/proof/status per tranche: up to 4 distinct entries per application (Tranche 1–Student, Tranche 1–Institution, Tranche 2–Student, Tranche 2–Institution).
- Confirmed: institution-level tracking is always shown, for every scholarship — not conditional on scholarship type.
- Confirmed: disbursement status values are a simple two-value set, **Pending → Disbursed**, per entry.
- Validation: proof upload restricted to image or PDF, consistent with the "screenshot/PDF" language used in the discussion.

**6. UI/UX Changes Required:** A new "Financial Tracking" tab/section on the application detail view, structured as a compact grid (2 tranches × 2 levels) rather than 4 free-standing forms — keeps it scannable, consistent with the project's low-clutter visual preference.

**7. Data Requirements:** A new child table, "Disbursement Entry," on the Scholarship Application:

| Field | Type | Notes |
|---|---|---|
| `tranche` | Select | Tranche 1 / Tranche 2 |
| `level` | Select | Student / Institution |
| `amount` | Currency | |
| `disbursement_date` | Date | |
| `proof` | Attach | Image or PDF only |
| `status` | Select | Pending / Disbursed |

Relationship: Scholarship Application (1) → Disbursement Entry (0–4, typically). No `has_second_tranche` flag needed — Tranche 2 is available on every application and remains non-mandatory throughout.

**8. Acceptance Criteria:**
- The Financial Tracking section is not visible/editable on an application until its status is Approved.
- Tranche 1 (both levels) cannot be saved without amount, date, and proof.
- Tranche 2 is available on every application with no mandatory validation — it can be left empty.
- Each of the up to 4 entries carries its own independent status, defaulting to Pending until marked Disbursed.

**9. Technical Considerations:** Since Tranche 2 is non-mandatory with no conditional gating, only Tranche 1's straightforward mandatory-field check (amount, date, proof) needs server-side enforcement in the DocType's `validate` controller hook. File-type validation should also be server-side.

**10. Priority Level:** **High** — the MIS Dashboard's "funds unlocked" metric and the Consolidated Profile Summary both depend on this data existing.

**11. Implementation Recommendation:** New child table ("Disbursement Entry") on the existing Scholarship Application DocType — repeating structured data belonging to a parent, the textbook child-table case, not a new parent DocType. No flag or conditional logic needed on the Scholarship master — Tranche 2 is simply optional, same as any non-mandatory field. Fully resolved — ready for CLI.
