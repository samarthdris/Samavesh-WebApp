# Point 2 — Complete PDF Upload and Download Option in Scholarship Data Entry Form
*(Module 4 in the master FRD)*

**Scope Status:** In Scope
**Priority:** Medium

---

**1. Feature/Module Name:** Complete PDF Upload and Download Option in Scholarship Data Entry Form

**2. Current State (Existing Wireframe Behavior):** The scholarship data entry form does not currently have a confirmed provision to attach the completed application PDF for later retrieval.

**3. Requested Change (verbatim, as confirmed):** "Upload the completed scholarship application PDF within the data entry form, with the option for student or fellow to download/access it at any time."

**4. Business Objective:** The submitted application PDF is often needed later (for the scholarship provider, for audits, or for the student's own record) — attaching it once at entry time avoids re-collecting or regenerating it later.

**5. Detailed Functional Requirements:**
- One PDF attachment per scholarship application entry.
- Confirmed: a re-upload replaces the existing file — no version history. This follows directly from a single-attachment field design; the requirement doesn't call for versioning.
- **[ASSUMPTION]** Upload is performed by Fellow/Admin via the data entry form; student has download/access only — per the requirement's wording ("option for student or fellow to download/access") and consistent with the read-only pattern in Module 1. Flag if students should also be able to upload.
- Validation: restrict upload to PDF file type.
- **[ASSUMPTION]** File-size cap defaults to a sensible platform-wide limit (e.g., 10MB) — not specified in the requirement.
- Edge case: an application record with no PDF yet attached should show a clear "not yet uploaded" state to both roles, not an error.

**6. UI/UX Changes Required:**
- An attach/upload control within the scholarship data entry form.
- A download/view link wherever the application record is displayed to student or fellow — including on the Application Status Tracker's details screen, as an exception to that screen's general document-exclusion rule, since "access it at any time" implies availability wherever the application record is viewed.

**7. Data Requirements:** A single `Attach` field (e.g., `application_pdf`) on the Scholarship Application DocType. No child table needed — cardinality is one file per application; a re-upload overwrites the field's reference.

**8. Acceptance Criteria:**
- A fellow can upload one PDF to a scholarship application; the same file is downloadable by both fellow and student roles from the application record, including from the application-details screen.
- Re-uploading a new PDF replaces the previous file — only the latest is retrievable.

**9. Technical Considerations:** Standard Frappe file attachment (`Attach` fieldtype) — no custom storage needed. Confirm the field's read permission includes the Student role (attachments follow the parent document's permission by default in Frappe, but worth an explicit check given other modules already restrict some fields from students).

**10. Priority Level:** **Medium** — self-contained, low-complexity; not a blocker for other modules.

**11. Implementation Recommendation:** A single `Attach` field on the existing Scholarship Application DocType; no new DocType, child table, or versioning logic required. Fully resolved — ready for CLI.
