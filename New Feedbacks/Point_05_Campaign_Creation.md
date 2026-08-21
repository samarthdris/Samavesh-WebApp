# Point 5 — Campaign Creation
*(Module 11 in the master FRD)*

**Scope Status:** Out of Scope for the current phase (per Aug 18 scope confirmation) | **Priority:** Medium *(if scoped in)*
**Note:** Fully resolved — zero open items. Sourced from two detailed client documents (Campaign Creation & Camp Management; Student Assign Flow) rather than the original call transcript, which only covered this at a high level. "Campaign" and "Camp" are confirmed as the same single entity, used interchangeably in the source material — modeled below as one DocType, referred to as **Camp**.

---

**1. Feature/Module Name:** Campaign Creation & Camp Management (incl. Student Assignment Flow)

**2. Current State (Existing Wireframe Behavior):** Not built. No concept of a camp currently exists in the wireframe.

**3. Requested Change:** An operational feature for managing on-ground student onboarding drives — Admin creates a camp (venue, schedule, fellow assignment), the system generates a unique onboarding link and QR code, and every student onboarded through that link is automatically mapped to the correct camp and fellow. Confirmed flow: **Admin creates a camp → assigns operational details and fellow(s) → system generates a unique onboarding link → fellow visits the venue and onboards students → Admin monitors progress from the camp dashboard.**

**4. Business Objective:** Serves as the operational backbone for onboarding drives at scale — reduces manual data entry and coordination errors, gives Admin real-time visibility into camp-level progress (scheduled/active/onboarded/follow-up needed), and gives fellows a simple, focused workflow.

**5. Detailed Functional Requirements:**

**5.1 Camp creation fields**
- Basic info: camp name, auto-generated ID, program (Select: Scholarship/Documentation/Mentoring/etc.), start date, end date, duration (auto-calculated, see below), camp owner, camp status (Select: Draft/Scheduled/Active/Completed/Closed), overall student target, description (optional).
- Duration is auto-calculated from start/end dates, **excluding Sundays** — confirmed. Admin never enters this manually.
- Venue & institution: venue type (College/Partner Org), institution/partner name (searchable dropdown with free-text fallback if not listed), address, city, district, state, venue contact person, contact number.
- Partner organisation is conditional: a Yes/No toggle; if Yes, reveals partner name, contact person, contact number, partnership type, and notes.

**5.2 Fellow assignment (on the camp)**
- Primary Fellow (required) and optional Supporting Fellow(s) (multiple allowed).
- The fellow picker should show workload context, not just a name — e.g. "Rohan — 3 active camps — 72 students" — so Admin can make an informed assignment.
- **Key rule:** a camp cannot be published unless at least one Primary Fellow is assigned.

**5.3 Unique onboarding link & QR code**
- Generated automatically once camp details and fellow are saved.
- Camp ID format: first 3 letters of institution name + date (dd/mm/yy) + auto-calculated camp number.
- Link format: `samavesh.org/onboard/CAMP-00124`.
- The link is permanently tied to Camp → Partner → Fellow, so a student registering through it never needs to select camp or fellow manually.
- Copy / Download QR / Share actions on the link.

**5.4 Publishing**
- Before publishing, show a summary (camp name, dates, duration, venue, partner, target, assigned fellows, status, link/QR generation confirmation).
- Actions: Save as Draft, or Publish Camp.

**5.5 Camp dashboard (Admin view, after creation)**
Header: camp name, live/status indicator, date/time, assigned fellows. Seven cards:
1. **Student registrations** — count vs. target, % of target, progress bar.
2. **Onboarding completed** — completed count + pending/draft count.
3. **Today's registrations** — day-wise count, most useful while the camp is live.
4. **Fellow** — primary fellow's students onboarded, applications count (from schemes mapped via eligibility matching, Point 7), documents-support count (from Module 1). Actions: View Fellow, View Students.
5. **Camp link** — the unique link, registration count through it, Copy/Download/Share. Called out in the source as one of the most prominent cards — the bridge between the physical camp and the digital system.
6. **Camp timeline** — a derived checklist, not a separate stored status: Camp Created, Fellow Assigned, Link Generated, Camp Published, Camp Live, Camp Completed, Follow-up Closed. Each tick is computed from existing fields (see Data Requirements) rather than manually set.
7. **Pending actions** — action-oriented, e.g. "9 student forms saved as draft," "4 students require document support." Links to View Pending Actions.

Below the cards: a camp-level student table — Student, Mobile, Onboarding Date, Documents, No. of Scholarships Mapped, Scholarship Application Status, Final Status, Assigned Fellow. All of these reuse existing modules (Module 1 documents, Point 7 eligibility matching, Module 3 application status) rather than introducing new tracking.

**5.6 Program-level overview (Admin view, all camps)**
A level above the individual Camp Dashboard: aggregate totals across all camps (total camps, target, registered, applications identified/completed, documents identified/completed, completed students), plus a table of individual camps (dates, venue, fellow, onboarded, completed, status) — clicking a row opens that camp's dashboard. Since Camp and Campaign are the same entity, this is a filtered/aggregated list view over Camp records (likely scoped by Program), not a separate parent-record rollup.

**5.7 Fellow's own view**
Deliberately simpler than Admin's: "My Camps" (upcoming camps with name/date/venue/target, Open Camp action). Once opened: venue & directions, timing, partner contact, the unique link/QR, and onboarded/completed/pending counts. A fellow's only actions are to work the assigned camp and onboard students — **fellows do not create camps, self-assign, select a campaign, or manually map students to a camp.**

**5.8 Student assignment — Track 1: via camp link**
- Student (or fellow, on the student's behalf) opens the camp's unique link and lands directly on the onboarding form, with camp, location, and institution already populated from the link — no manual selection.
- On submission, the system auto-maps Student → Camp.
- Fellow assignment then follows one of two paths:
  - **One fellow on the camp:** student is auto-assigned to that fellow — no manual step.
  - **Multiple fellows on the camp:** the person completing the form picks from a dropdown scoped to only that camp's fellows (Primary and Supporting both included — confirmed), not the full org fellow list.
- Assignment confirmation shows: student name, camp, camp date, assigned fellow, onboarding status, application status.

**5.9 Student assignment — Track 2: via regular onboarding form**
- Used when a student is onboarded outside any camp context — no auto camp mapping.
- The regular form includes an "Assign Fellow" dropdown (the full org fellow list); the fellow completing the onboarding simply selects their own name.
- Student appears immediately in that fellow's dashboard/student list.

**5.10 Assignment status & reassignment**
- Every student carries an assignment status: Not Assigned / Assigned / Reassigned / Inactive-Closed.
- Admin/authorised users can reassign a student's fellow after onboarding. The system keeps the current assignment as the active one while maintaining a history/audit trail of prior assignments for administrative tracking.

**5.11 Key system rules (carried directly from the source document)**
- Every camp link is unique to exactly one camp.
- Students via camp link are automatically mapped to that camp and its assigned fellow(s).
- One fellow on a camp → auto-assign; multiple fellows → dropdown scoped to that camp only.
- Both the regular form and the camp-specific form include an Assign Fellow control.
- A fellow can select themselves during regular onboarding.
- Admin/program users can reassign students when required.
- Unassigned students must be clearly identifiable.
- Assignment changes reflect immediately in the fellow's dashboard.
- The system maintains Student → Fellow → Camp relationships for reporting.
- Reassignment changes are captured in an assignment history/audit trail.

**6. UI/UX Changes Required:**
- New Camp creation form (multi-section, per 5.1–5.4).
- New Camp Dashboard (7 cards + student table, per 5.5) and Program-level overview (per 5.6).
- New "My Camps" view for fellows (per 5.7), deliberately lighter than Admin's.
- The onboarding form gains conditional pre-fill behavior when reached via a camp link (camp/location/institution hidden and auto-set, vs. shown and manual for the regular form).
- An "Assign Fellow" dropdown appears on both onboarding paths, scoped differently (camp's fellows only vs. full org list).

**7. Data Requirements:**

**Camp DocType (new):**

| Field | Type | Notes |
|---|---|---|
| `camp_name` | Data | |
| `name` (ID) | Auto-generated | Institution (first 3 letters) + date + camp number |
| `program` | Select | Scholarship / Documentation / Mentoring / etc. |
| `start_date`, `end_date` | Date | |
| `duration` | Read Only | Computed: calendar days, excluding Sundays |
| `camp_owner` | Link → User | Filtered to Admin/Program Manager roles |
| `camp_status` | Select | Draft / Scheduled / Active / Completed / Closed |
| `overall_student_target` | Int | |
| `description` | Small Text | Optional |
| `venue_type` | Select | College / Partner Org |
| `institution_name` | Link | Searchable; free-text fallback creates a new record |
| `venue_address`, `city`, `district`, `state` | Data/Link | |
| `venue_contact_person`, `venue_contact_number` | Data | |
| `has_partner_organisation` | Check | Gates the 5 partner fields below |
| `partner_organisation_name`, `partner_contact_person`, `partner_contact_number`, `partnership_type`, `partner_notes` | Data/Select/Small Text | Conditional on above |
| `primary_fellow` | Link → User | Required before publish |
| `supporting_fellows` | Table MultiSelect | Optional, multiple |
| `unique_link` | Read Only | Auto-generated on save |
| `qr_code` | Attach | Generated from `unique_link` |

**Student DocType — new fields:**

| Field | Type | Notes |
|---|---|---|
| `assigned_camp` | Link → Camp | Set automatically for Track 1; null unless later linked for Track 2 |
| `assigned_fellow` | Link → User | Set via either track |
| `assignment_status` | Select | Not Assigned / Assigned / Reassigned / Inactive-Closed |

**Fellow Assignment Log (new child table, on Student):**

| Field | Type | Notes |
|---|---|---|
| `previous_fellow`, `new_fellow` | Link → User | |
| `changed_by` | Link → User | Defaults to session user |
| `changed_on` | Datetime | Defaults to now |
| `reason` | Small Text | Optional |

The Camp Timeline (5.5, card 6) is derived, not stored: Fellow Assigned ← `primary_fellow` is set; Link Generated ← `unique_link` is set (happens together with the above); Camp Published ← `camp_status` ≥ Scheduled; Camp Live ← `camp_status` = Active; Camp Completed ← `camp_status` = Completed; Follow-up Closed ← `camp_status` = Closed.

**8. Acceptance Criteria:**
- A camp cannot be published without a Primary Fellow assigned.
- Duration always reflects calendar days minus Sundays, never manually entered.
- A student onboarded via a camp link is automatically tagged to that camp with no manual camp/location selection.
- If a camp has one fellow, every student from that camp's link is auto-assigned to them; if multiple, the assignment dropdown shows only that camp's fellows.
- A student onboarded via the regular form always has an Assign Fellow control, defaulting to no pre-fill.
- Reassigning a student's fellow updates their current assignment immediately and logs the change in the assignment history.
- The Camp Timeline checklist always reflects the camp's actual field values — never an independently-editable status.

**9. Technical Considerations:**
- `camp_owner`'s "Admin / Program Manager" dropdown is modeled as a `Link` to User filtered by role, rather than a plain role-name selector — confirm this reading is correct once implementation starts, though it's low-risk either way.
- Institution/partner name-matching should reuse a shared Institution master (Link field with quick-create) rather than free text, to avoid duplicate/near-duplicate institution records over time.
- QR code generation can be handled with any standard QR library server-side, keyed off `unique_link` — no custom encoding needed.
- The Program-level overview (5.6) is a filtered aggregate view over Camp records, not a join across a separate parent entity, since Camp and Campaign are the same DocType.

**10. Priority Level:** Not prioritized — Out of Scope for the current phase. If scoped in: **Medium** — high operational value for scaled onboarding, but depends on Point 7 (eligibility matching) and Module 1 (documents) already being in place for the dashboard cards to show real numbers.

**11. Implementation Recommendation:** New DocType "Camp" (no separate Campaign parent needed). Extend Student with the 3 new fields plus the Fellow Assignment Log child table. Build the Camp Dashboard's 7 cards as read-only aggregate views over existing data (Module 1, Point 7, Module 3) rather than duplicating any of that tracking on the Camp record itself. Do not begin until this module is formally scoped in and an effort estimate is reviewed, per the existing Aug 12 MoM action item — but the spec itself is complete and ready for that point.
