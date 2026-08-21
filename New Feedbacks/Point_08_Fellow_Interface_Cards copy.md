# Point 8 — Fellow Interface Cards Changes
*(Module 7 in the master FRD — Fellow portion only)*

**Scope Status:** In Scope | **Priority:** Medium
**Note:** This resolves the previously pending card list. 2 Dhwani-suggested cards are folded directly into the list below (marked inline) rather than kept separate. Admin dashboard cards remain a separate, still-pending item.

---

**1. Feature/Module Name:** Fellow Interface Dashboard Cards

**2. Current State (Existing Wireframe Behavior):** A draft set of dashboard cards was discussed on the original call but not finalized — Dhwani proposed a curated set rather than one card per metric, pending Samavesh's specific list.

**3. Requested Change (confirmed card list, 12 total):**
1. Total number of students assigned
2. Total number of scholarship applications assigned
3. Total number of document support cases assigned
4. Total number of scholarship applications completed / successfully submitted
5. Total number of document support cases completed / successfully submitted
6. Total number of scholarship applications in process
7. Total number of document support cases in process
8. Total funds unlocked
9. Total documents supported
10. Total number of rejected applications
11. Applications/documents pending beyond a set number of days *(Dhwani-suggested — aging indicator)*
12. Month-over-month view of the fellow's own completions *(Dhwani-suggested — personal trend)*

**4. Business Objective:** Give fellows a single at-a-glance view of their own workload and progress — assigned vs. completed vs. in-process vs. rejected — without running a report or scrolling through individual student records.

**5. Detailed Functional Requirements:**
- All 12 cards are scoped to the logged-in fellow (their own assigned students/applications only), matching the fellow-scoping pattern used throughout this document.
- Module 3's application-level status values are now confirmed: **Submitted → Under Review → Approved / Rejected**. Mapping cards 4/6/10 onto these (derived, not verbatim — flag if this split isn't what you meant): Card 4 (completed/successfully submitted) = status **Approved**; Card 6 (in process) = status **Submitted or Under Review**; Card 10 (rejected) = status **Rejected**. Together these three cleanly partition all 4 statuses with no gaps or overlap.
- Cards 5 and 7 (document cases completed / in process) use Module 1's already-confirmed 5-value status set: "completed" = Submitted to Scholarship; "in process" = any of the other four values.
- Card 8 (Total funds unlocked): SUM(Disbursement Entry.amount) where status = Disbursed, scoped to the fellow's students — same definition used in the MIS Dashboard (Point 4).
- Card 9 (Total documents supported): reuses the existing definition from the Consolidated Profile Summary (Module 6) — count of documents the fellow uploaded on the student's behalf.
- Card 10 (rejected applications): see the status mapping above — status = Rejected.
- Cards 11 and 12 both need to know *when* a status was reached, not just the current status — this timestamp history doesn't exist yet in any module in this document. Recommend enabling Frappe's built-in change tracking (`track_changes: 1`) on Scholarship Application and Application Document rather than adding bespoke timestamp fields — the same fix already recommended for the MIS Dashboard's turnaround-time indicator (Point 4), so this becomes one shared prerequisite instead of two separate ones.

**6. UI/UX Changes Required:** A 12-card grid on the Fellow dashboard — likely 2 rows of 6, or a responsive wrap, larger than the "curated 4–6" size originally anticipated for this module. Cards read as plain counts, consistent with the project's minimal visual style — no color-coded status indicators unless requested.

**7. Data Requirements:** All 12 cards are aggregate queries scoped by `frappe.session.user` → assigned-fellow lookup, reading from: Student, Scholarship Application (Module 3 — status values now confirmed), Application Document (Module 1 — confirmed), Disbursement Entry (Module 5 — confirmed). Cards 11–12 additionally need Frappe's change-tracking history (see above) rather than a new DocType.

**8. Acceptance Criteria:** Each of the 12 cards, for a given fellow, matches a manual count/sum against the same filters; no card ever shows another fellow's data. Card 11's "pending beyond X days" threshold is configurable, not hardcoded.

**9. Technical Considerations:** Implement as Frappe Number Cards or a small custom Page with a Server Script endpoint, per the pattern already used elsewhere in this document. With 12 simultaneous aggregate queries per dashboard load, consider a brief cache (a few minutes) if load time becomes noticeable — not needed at current expected volumes.

**10. Priority Level:** **Medium** — mostly a reporting layer over data other modules already produce; 10 of 12 cards are ready to build now, cards 11–12 are blocked only on the change-tracking prerequisite.

**11. Implementation Recommendation:** Build cards 1–10 now — all read from already-confirmed data (status values, disbursement status, document status). Sequence cards 11–12 once change tracking is enabled — which also unblocks two of the MIS Dashboard's added indicators (Point 4), so it's worth doing once for both documents rather than twice.
