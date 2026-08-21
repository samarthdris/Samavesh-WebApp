# Point 4 — MIS Dashboard
*(Module 10 in the master FRD)*

**Scope Status:** Out of Scope for the current phase | **Priority:** High *(if scoped in)*
**Note:** Every originally-discussed indicator is preserved below, plus 5 Dhwani-suggested additions for Admin (approved, folded into the main list rather than kept separate). The screenshot shared this round matches the original description with no new client indicators visible — still pending, flagged in chat, not blocking this document.

---

**1. Feature/Module Name:** Admin MIS / M&E Dashboard

**2. Current State (Existing Wireframe Behavior):** Not built. No program-monitoring dashboard currently exists.

**3. Requested Change (confirmed indicators — none removed, 5 added):** A program-monitoring dashboard with filters (year/fellow/student/college-wise) and now approximately 27–28 indicators, covering:
- Overview metrics: total applications, students, funds unlocked, schemes tapped
- Gender-wise breakdown
- Month-wise timeline
- Education stream-wise breakdown
- Total colleges and grade-wise count
- Socio-economic breakdowns: caste-category-wise, annual-income-wise
- Scheme-wise application breakdown
- Documents-support dashboard
- Geography-wise breakdown (incl. migrant/non-migrant)
- Fellow-wise performance and scholarship-wise cuts
- Conversion funnel: Eligible → Applied → Approved → Disbursed, with drop-off rate at each stage *(Dhwani-suggested)*
- Average turnaround time: time-to-approval and time-to-disbursement *(Dhwani-suggested)*
- Repeat/renewal rate: % of students supported across more than one year *(Dhwani-suggested)*
- Document friction rate: which document types sit longest in "Follow-up in Progress" *(Dhwani-suggested)*
- Scheme utilization: proportion of a scheme's available slots/funds actually used *(Dhwani-suggested)*

Indicators are largely independent metrics (not cross-tabulated), per the original discussion — except the conversion funnel, which is inherently sequential; noted separately in Technical Considerations.

**4. Business Objective:** Program leadership and possibly funders need aggregate visibility into reach, equity (gender/caste/income/geography), and fellow performance — data that individual student/application records don't surface on their own.

**5. Detailed Functional Requirements:**
- Indicators are largely independent metrics (drawn from individual fields), not cross-tabulated — e.g., "% female" and "% rural" reported separately, not as a combined "% rural female" cut. **[OPEN QUESTION]** Confirm this holds for all ~22–23 original indicators, or whether specific cross-tabs were intended.
- A shared filter strip (year / fellow / student / college) applies consistently across all indicator views, not per-page custom filters.
- The exact final indicator list (~22–23 original) was not fully enumerated field-by-field in the original discussion — this document lists the categories discussed, not a locked field-level spec. **[OPEN QUESTION]** The additional client-shared indicators mentioned this round haven't been received yet — pending.
- The conversion funnel (Eligible → Applied → Approved → Disbursed) is buildable now as a **snapshot** funnel — current counts at each stage, computed from existing status fields. A **cohort** funnel (e.g., "of students who became eligible in January, how many reached disbursement by March") would need status-change history that doesn't exist yet — see Technical Considerations.
- Turnaround time and document friction rate both need to know *when* a status was reached, not just the current status — this data isn't captured yet by any module in this document. See Technical Considerations for the recommended fix.
- Repeat/renewal rate depends on reliably matching the same student across years, including historical records (Module 9) — needs a stable student identifier that survives the migration; not yet confirmed as existing.
- Scheme utilization needs a "capacity" or "budget" figure per scheme to measure against — this field doesn't exist in the Scholarship master as currently speced (Module 8); would need to be added.

**6. UI/UX Changes Required:** A dedicated dashboard/workspace with a persistent filter strip and a grid of independent charts/number cards per indicator category; given the volume, organized into sub-sections (Demographics, Geography, Scheme Performance, Fellow Performance, Process Efficiency) rather than one long scroll — the last group holds the 5 newly added indicators.

**7. Data Requirements:** Pulls from Scholarship Application, Student, Disbursement Entry (Module 5 — confirmed), and Application Document (Module 1 — confirmed) records. "Funds unlocked" is defined precisely: SUM(Disbursement Entry.amount) where status = Disbursed, per Point 3's resolution. Geography and socio-economic fields (caste category, annual income, migrant status) must exist as structured fields on the Student record. **[OPEN QUESTION]** Do these fields already exist on the student profile, or need to be added? Not confirmed. **New requirements from the 5 added indicators:** a scheme-level `capacity`/`budget` field on the Scholarship master (for utilization); a stable cross-year student identifier (for renewal rate); and status-change timestamps on Scholarship Application and Application Document (for turnaround time and friction rate) — see Technical Considerations for how to capture the latter without new fields on every DocType.

**8. Acceptance Criteria:** Each of the ~27–28 indicators renders a figure/chart matching a manual query against the same filters; applying the shared filter strip updates all indicators consistently. The conversion funnel's per-stage counts sum consistently with the underlying status distributions.

**9. Technical Considerations:** At this indicator count, evaluate Frappe's native Query Report / Dashboard Chart / Number Card building blocks before considering a custom-built reporting layer; a shared filter strip across many independent widgets is the main non-trivial technical piece. For turnaround time and document friction rate, recommend enabling Frappe's built-in change tracking (`track_changes: 1`) on Scholarship Application and Application Document rather than adding bespoke timestamp fields — Frappe's native Version doctype then holds a queryable history of every status change, which both indicators can read from directly.

**10. Priority Level:** Not prioritized — Out of Scope for the current phase. If scoped in: **High**, given stakeholder visibility, but also high-effort given the indicator count — recommend phased delivery (demographics first, fellow-performance later).

**11. Implementation Recommendation:** Do not begin building until Dhwani's effort estimate is reviewed and scope is formally confirmed. When it is, sequence by sub-section rather than building all ~27–28 indicators at once — the Process Efficiency sub-section (the 5 added indicators) has real prerequisites (change tracking, a scheme capacity field, a stable student identifier) and should be sequenced after the original categories, not alongside them.

*Student-facing note: MIS-style dashboards are typically admin/fellow tools. The one student-relevant idea from this round — a simple "what's next" progress line — has been folded into the Consolidated Profile Summary (Module 6) instead of this dashboard, since that's where students actually look.*
