# Point 6 — Target Assign to Fellow Flow
*(Module 12 in the master FRD)*

**Scope Status:** Out of Scope for the current phase | **Priority:** Low *(if scoped in)*
**Note:** Fully resolved — zero open items. Still Out of Scope, but documented ready for CLI whenever this gets picked up.

---

**1. Feature/Module Name:** Fellow Target Assignment Flow (Target Assign to Fellow Flow)

**2. Current State (Existing Wireframe Behavior):** Not built. No performance-target concept currently exists for fellows.

**3. Requested Change:** Ability to set performance targets for fellows across multiple metric types (confirmed: applications submitted, documents collected, students onboarded — not restricted to applications alone) and track performance against target; targets may vary by fellow, with Admin able to set and override them. **Confirmed Out of Scope for the current phase.**

**4. Business Objective:** Without targets, fellow performance can only be reviewed qualitatively; a target-vs-actual view gives program management an objective, per-fellow performance signal across the metrics that matter to the program, not just one.

**5. Detailed Functional Requirements (Deferred — drafted for future scoping only):**
- Admin can set a target value per fellow per period, for any of three confirmed metric types: applications submitted, documents collected, students onboarded.
- An admin creates a separate target record per fellow/period/metric-type combination — e.g., one record for a fellow's monthly application target, a separate record for that same fellow's monthly documents-collected target — rather than one record trying to hold every metric at once.
- Targets vary by fellow — no single platform-wide default target is assumed.
- Admin can override/edit a previously set target.
- Actual performance is computed from existing data, with the source depending on metric type (see Data Requirements) — not manually entered.
- Edge case: a fellow with no target set for a given metric should show as "no target set," not a 0-value target — these are different states and shouldn't be visually conflated.

**6. UI/UX Changes Required (Deferred):** An Admin-facing screen to set/edit targets per fellow per period per metric type; a target-vs-actual view, likely alongside or feeding into the Fellow/Admin dashboard cards (Point 8/Module 7).

**7. Data Requirements (Deferred):** New DocType "Fellow Target": `fellow` (Link → Fellow/User), `period` (Month, or a Date range pair), `metric_type` (Select: Applications Submitted / Documents Collected / Students Onboarded), `target_value` (Int). `actual_value` is not stored — computed on read, with the source table depending on `metric_type`: Scholarship Application (Applications Submitted), Application Document (Documents Collected), Student (Students Onboarded) — each filtered by fellow + period.

**8. Acceptance Criteria (Deferred — draft):** Admin can set, view, and edit a target for any fellow, for any of the three metric types, for a given period; the actual-vs-target comparison for a fellow matches a manual count against the correct source table for that metric type.

**9. Technical Considerations (Deferred):** No workflow/approval needed — this is Admin-only data entry with a computed comparison; a plain DocType with write permission restricted to the Admin role is sufficient. The "actual value" computation branches on `metric_type` — three separate lookup queries, one per metric type, rather than one generic query, since each metric type reads from a different DocType.

**10. Priority Level:** Not prioritized — Out of Scope for the current phase. If scoped in: **Medium** — self-contained; depends on Modules 1/3 application data existing, but doesn't block anything else in this document.

**11. Implementation Recommendation:** Do not begin until scoped in. When it is: new DocType "Fellow Target" (Master-type, no submit/workflow needed) — a target is reference data Admin maintains, not a transactional decision record. Fully resolved — ready for CLI whenever this gets scoped in.
