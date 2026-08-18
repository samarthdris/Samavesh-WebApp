# BUSINESS REQUIREMENTS DOCUMENT — SAMAVESH

**Student Scholarship & Mentorship Platform**
Phase-Wise Implementation · Forms · Mentorship & LMS · Validated Clickable Prototype

**Prepared by** Dhwani Rural Information Systems Pvt. Ltd.
**For** Samavesh Action For Impact
**Version 4.0 | July 2026**

---

# 1. Executive Summary

Samavesh Action For Impact is an education-focused organization that supports students from underserved communities in accessing higher education through structured mentoring and government scholarship enablement. Samavesh has supported over 3,000 students and unlocked approximately ₹8 crore in government scholarship funds to date. The program is delivered on-ground by Fellows — trained field staff — who work one-on-one with students through onboarding, eligibility mapping, document support, application submission, and post-disbursement tracking, with mentors supporting higher-education readiness in parallel.

Current operations rely on Google Forms, spreadsheets, WhatsApp groups, and manual coordination. As the program scales, this fragmented stack creates bottlenecks — duplicate data, missed scholarships, lost documents, incomplete status visibility, and a limited audit trail.

This BRD defines the requirements for a unified, role-based digital platform — the **Samavesh Platform** — that replaces fragmented tools, automates eligibility matching, tracks applications end-to-end, delivers a structured mentorship and LMS experience, and provides a single MIS view to leadership. The platform is built on the **Frappe Framework**, deployed on the **AWS India Region** for DPDP compliance, and delivered in **four sequential phases** over an indicative timeline of 14–16 weeks from kick-off.

**What is new in Version 4.0.** Version 4.0 aligns the requirements to a **validated, clickable prototype** of the platform that Dhwani has built and demonstrated to Samavesh, and it incorporates the **screen-by-screen feedback** Samavesh recorded during the wireframe walkthrough. Status vocabulary, document-handling rules, the student-journey funnel, and the application-tracking model have been harmonized across every section so that the specification reads as a single consistent source of truth. Section 24 (**Current Prototype Scope**) states precisely which capabilities are demonstrated in the prototype today and which remain on the delivery roadmap.

### 1.1 Document Objective

- Provide a single, authoritative reference of business requirements, validations, and business rules for both the Samavesh program team and the Dhwani delivery team.
- Establish a phase-wise release plan with clearly stated deliverables, acceptance criteria, and sign-off owners for each milestone.
- Define the form structures (student profiling, document verification, scholarship application) as validated in the prototype.
- Detail the mentorship flow and the LMS journey to be implemented in delivery Phase 3.
- Record the wireframe feedback incorporated from Samavesh and the resulting design decisions.
- Capture assumptions, dependencies, risks, and out-of-scope items so both parties remain aligned through delivery.

---

# 2. Document Control

## 2.1 Document Metadata

| Field | Detail |
|---|---|
| Document Title | Samavesh – Business Requirements Document (BRD) |
| Document Version | 4.0 (Prototype-Aligned, Phase-Wise Release Plan) |
| Date | July 2026 |
| Client | Samavesh Action For Impact |
| Prepared By | Dhwani Rural Information Systems Pvt. Ltd. |
| Reviewed By | Project Manager – Dhwani |
| Status | Draft for Client Review and Sign-off |
| Distribution | Samavesh Leadership, Samavesh SPOC, Dhwani Project Team, Dhwani Pre-Sales |

## 2.2 Revision History

| Version | Date | Author | Summary of Changes |
|---|---|---|---|
| 1.0 | March 2026 | Dhwani Team | Initial BRD with module-wise requirements |
| 2.0 | May 2026 | Dhwani Team | Added phase-wise release plan, detailed form specifications, business rules, validations, mentorship & LMS flow, sign-off matrix, acceptance criteria |
| 3.0 | May 2026 | Dhwani Team | Expanded Mentorship (Section 12) and LMS (Section 13) with detailed use cases, functional requirements, business rules, data dictionary, and acceptance criteria |
| 4.0 | July 2026 | Dhwani Team | Aligned the specification to the validated clickable prototype; incorporated Samavesh's screen-by-screen wireframe feedback; harmonized application-status vocabulary, the student-journey funnel, and document file-handling rules across all sections; added the unified Applications caseload model, Fellow attendance, student self-onboarding and self-upload, notes, and the current-prototype-scope inventory. Document version label corrected and made consistent throughout. |

## 2.3 How to Read This Document

This BRD serves two audiences. The Samavesh program team should read Sections 1 through 9 to validate scope, forms, and business rules, and Sections 14 through 17 to plan around the phase-wise release, sign-off cadence, and dependencies on Samavesh. The Dhwani delivery team should treat Sections 9 through 13 as the primary specification for build, and Sections 18 through 20 as the operating envelope (assumptions, risks, out-of-scope). **Section 24 (Current Prototype Scope)** tells any reader — including a prospective partner organization — exactly what the clickable prototype demonstrates today versus what is planned.

Where a requirement is marked **Must Have**, it is in scope for the earliest applicable release. **Should Have** items are scheduled within a later phase. **Could Have** items are explicitly deferred to a possible future enhancement window.

**A note on the word "Phase."** This document uses "Phase" in two distinct senses. The **program workflow phases** (Section 6/8: Onboarding → Eligibility → Hands-on Support → Tracking) describe how the Samavesh program runs. The **delivery release phases** (Section 14: P1–P4) describe the build schedule. Where ambiguity is possible, the sense is stated explicitly.

---

# 3. Glossary & Acronyms

| Term | Definition |
|---|---|
| BRD | Business Requirements Document – this document |
| DPDP | Digital Personal Data Protection Act, 2023 (India) |
| Fellow | Samavesh field staff member who supports students 1:1 through the scholarship journey; the platform's primary operator |
| Mentor | Subject-matter expert who supports a student with higher-education guidance and career readiness |
| SPOC | Single Point of Contact – the Samavesh nominee for daily project coordination |
| SOP | Standard Operating Procedure followed by Samavesh Fellows |
| NSP | National Scholarship Portal – Government of India scholarship submission portal |
| LMS | Learning Management System – module that delivers structured mentoring content to students |
| mForm | Mobile-friendly dynamic form module on the platform |
| UAT | User Acceptance Testing – formal testing window before each release |
| MIS | Management Information System – consolidated reporting dashboard |
| RBAC | Role-Based Access Control |
| PWA | Progressive Web App – mobile-browser experience without a native app |
| SSO | Single Sign-On (Google) used as the primary authentication method |
| OTP | One-Time Password – six-digit code used to verify a student's email at sign-up |
| KPI | Key Performance Indicator |
| Caseload | The consolidated list of a Fellow's active scholarship applications, each advanced along a single status ladder |
| Triage | A derived (computed) view of caseload attention: Action needed / Awaiting / Done / On Hold |
| Sub-document | A supporting attachment required to procure a primary document (e.g., Aadhaar to obtain a Ration Card) |

---

# 4. Organizational Context

## 4.1 About Samavesh

Samavesh Action For Impact works at the intersection of education equity and scholarship access. The organization deploys Fellows — trained field staff — who work one-on-one with students to navigate complex government scholarship systems, procure documents, and stay enrolled in higher education institutions. Alongside scholarship access, Samavesh provides structured mentoring to support higher-education readiness.

### Key Program Metrics

- 3,000+ students supported to date
- Approximately ₹8 crore in government scholarships unlocked
- Multiple geographies served by a distributed Fellow team
- Active mentoring journey running in parallel to the scholarship workflow

## 4.2 Current Operating Model (As-Is)

| Step | Current Tool / Method | Pain Point |
|---|---|---|
| Student Onboarding | Google Forms | No persistent student profile; data siloed per submission; duplicate entries |
| Scholarship Matching | Manual lookup by Fellows | No automated eligibility logic; prone to missed scholarships |
| Document Collection | WhatsApp / physical collection | No status tracking; documents lost; format issues caught too late |
| Application Submission | Government portals (manual) | Fellow logs into each portal separately; no central record of application IDs |
| Application Tracking | Excel / Google Sheets | Status not real-time; high manual effort; no reminders |
| Mentoring Coordination | WhatsApp / calls | No interaction log, no content delivery, no progress view |
| Reporting & MIS | Manual dashboards | Time-consuming; error-prone; donor reporting delayed |

## 4.3 Strategic Drivers for Digitization

- Scale to 20,000+ student records without growing operational overhead proportionally.
- Reduce scholarship leakage by automating eligibility detection per student profile.
- Provide leadership with real-time MIS instead of stitched-together monthly reports.
- Meet DPDP Act 2023 obligations for student consent, data residency, and auditability.
- Standardize the Fellow SOP into a system-enforced workflow.

---

# 5. Problem Statement & Solution Overview

## 5.1 Problem Statement

Samavesh's scholarship-access program is delivered through a Fellow-led model that depends on a heterogeneous stack — Google Forms for capture, WhatsApp for documents, Excel for tracking, and government portals for submission. This stack does not scale beyond the current operational size, makes scholarship leakage hard to detect, prevents real-time leadership visibility, leaves student documents at risk, and does not meet DPDP-grade audit and consent requirements.

## 5.2 Solution Overview

Dhwani delivers a unified Samavesh Platform that digitizes the full student lifecycle — onboarding, eligibility, documents, application, tracking, and mentorship — with role-based access and a leadership MIS. The platform is built on the Frappe Framework, hosted on the AWS India Region, and released in four phases. A validated clickable prototype of the Student, Fellow, and Program Admin experiences has been built and demonstrated to Samavesh (see Section 24).

### 5.3 Solution Architecture Summary

| Layer | Component | Rationale |
|---|---|---|
| Application Framework | Frappe Framework v15 (LTS) | Open-source, low-code, DocType-driven, built-in RBAC and audit trail; LMS app available natively |
| LMS | Frappe LMS App | Native integration with Frappe, content delivery, course progression tracking |
| Data Capture | Frappe Web Form / mForm | Dynamic forms for onboarding, document verification, scholarship application |
| Hosting | AWS Mumbai (ap-south-1) | Data residency required for DPDP compliance; managed by Dhwani |
| Database | MariaDB (managed) | Default Frappe DB; encrypted at rest |
| File Storage | AWS S3 (Mumbai) | Encrypted object storage for student documents |
| Authentication | Google SSO (primary) + email OTP verification at student sign-up | Fellows/Admins use organizational Google accounts; students verify email ownership via a six-digit OTP |
| Notifications | Email + SMS (via gateway) + WhatsApp deep-links | Status alerts, deadline reminders, and direct student-to-Fellow / student-to-helpline reach-out |
| Frontend | Frappe web + PWA (mobile browser) | No native mobile app in Phase 1; PWA covers Fellow field use |

## 5.4 High-Level Module Map

- Module 1 – Student Onboarding & Profile Management
- Module 2 – Scholarship Catalogue & Eligibility Engine
- Module 3 – Document Collection, Verification & Vault
- Module 4 – Scholarship Application & Tracking (unified caseload)
- Module 5 – Mentorship Coordination & LMS *(delivery Phase 3 — roadmap)*
- Module 6 – Role-Based Access Control & Audit
- Module 7 – MIS Dashboards & Reporting
- Module 8 – Data Privacy & DPDP Compliance
- Module 9 – Fellow Field Operations (attendance, notes, WhatsApp reach-out)

---

# 6. As-Is Program Workflow

Samavesh's program follows four sequential phases with a parallel mentoring journey. This section reproduces the current workflow that the platform digitizes, drawn from the Samavesh workflow diagram and the Fellow SOP.

## 6.1 Phase 1 – Student Onboarding

A Fellow initiates contact with a student and onboards them by collecting socio-economic, educational, and personal background information through a digital onboarding form. The form output becomes the master student profile used across all downstream modules.

## 6.2 Phase 2 – Eligibility & Discovery

Based on the student profile, the platform identifies the scholarships the student is eligible for and auto-generates a document checklist for each. This stage replaces the manual lookup that Fellows currently perform.

## 6.3 Phase 3 – Hands-On Support

The Fellow provides one-on-one support to procure missing documents and submit scholarship applications. Documents are uploaded into the platform, verified, and the application is then submitted on the relevant government portal. Submission details (date, application reference) are recorded back into the platform.

## 6.4 Phase 4 – Tracking, Liaising & Re-Application

The Fellow monitors application status on each government portal and updates the platform. Applications redirected for correction are routed through a re-application loop. On disbursal, the platform marks the case closed and updates the outcome dashboard.

## 6.5 Parallel – Mentoring Journey

In parallel to the scholarship workflow, the student is engaged in a structured mentoring journey for higher-education readiness. Students access a mentoring experience with learning content and can book sessions with mentors. The platform provides this through the Mentorship module and the LMS (delivery Phase 3).

---

# 7. Stakeholders & User Roles

| User Role | Description | Primary Actions on the Platform |
|---|---|---|
| Student | Young person from an underserved community seeking scholarship & mentoring support | View own profile, view submitted onboarding form (read-only) and request changes, upload additional/follow-up documents, attach sub-documents, view scholarship status, reach out to Fellow/helpline on WhatsApp, browse LMS content, give consent |
| Fellow | Samavesh field staff; the primary operator of the platform | Onboard students, fill all forms, manage documents, submit and track scholarship applications on a unified caseload, log attendance, record notes from calls/meetings, respond to student change requests |
| Mentor | Subject-matter expert providing higher-education guidance | Publish availability, conduct booked sessions, log interactions *(delivery Phase 3)* |
| Program Manager / Admin | Samavesh HQ staff overseeing program operations | Monitor MIS, verify documents (Accept/Reject), manage the scholarship catalogue, manage users, reassign cases, view attendance, generate reports |
| Super Admin | System administrator (Dhwani and designated Samavesh staff) | Full system access, configuration, user & role-profile management, data administration including the DPDP deletion workflow, audit-log access |

Each role has a corresponding role profile in the platform's RBAC layer (Section 9.6). Role profiles control which DocTypes, fields, and reports each user may access. In the current prototype, the Student, Fellow, and Program Admin seats are demonstrated end-to-end; the Mentor experience is on the Phase-3 roadmap and the Super Admin seat is a defined role profile whose dedicated administration screens are planned (see Section 24).

---

# 8. To-Be Platform Workflow

The to-be workflow mirrors the four-phase Samavesh program, but every step is system-mediated. Fellows operate from a single workspace; Admins see a single MIS; students see only their own data.

## 8.1 Phase 1 – Digital Onboarding

- A student can be onboarded in one of two ways: (a) the **Fellow** initiates a new onboarding and completes the Student Profiling Form on the student's behalf, or (b) the student receives a **shareable onboarding link**, fills the same form themselves, and submits it for Fellow review.
- Before creating a new record, the system checks for an existing profile (by email or full name) to avoid duplicates.
- The form captures personal, family, socio-economic, educational, and demographic data.
- The student (or guardian, for minors) gives explicit digital consent for data processing under DPDP.
- When a student self-submits, the submission enters a **Fellow approval gate**: the Fellow reviews the entire form, edits if needed, and approves it before the student's full portal unlocks.
- The profile is saved with a unique Student ID and becomes the master record for all downstream stages. The student can subsequently view their submitted form read-only and **request a change** (which notifies the Fellow); the student never edits an approved profile directly.

## 8.2 Phase 2 – Automated Eligibility & Document Checklist

- On profile save (and on every profile edit), the eligibility engine evaluates the profile against the scholarship catalogue.
- The platform displays the scholarships the student is eligible for, with metadata: governing body, portal URL, benefit amount, deadline.
- For each eligible scholarship, the platform auto-generates a document checklist mapped to the scholarship's required documents.
- An Admin can manually override eligibility for edge cases, with a mandatory reason captured for audit.

## 8.3 Phase 3 – Document Support, Verification & Application Submission

- The Fellow procures and uploads each required document; the student may also upload their own additional/follow-up documents and, for a document they do not have, attach supporting **sub-documents** so the Fellow can procure it.
- The system validates format and file size at upload time (Section 10.5).
- A Program Admin reviews each document and marks it **Accepted** or **Rejected** (with reason).
- Once all required documents are **Accepted** for a scholarship, that application becomes **Ready to Submit**.
- The Fellow submits the application on the government portal and records submission details — portal application ID, submission date, submitted PDF — into the platform.

## 8.4 Phase 4 – Status Tracking, Re-Application & Disbursement

- The Fellow checks each government portal and updates the application status in the platform from a single unified caseload.
- If an application needs correction, its status is set to **Re-apply** and it is worked and re-submitted.
- On **Application Approved** and then **Benefits Received**, the Fellow records the disbursed amount and date (and uploads proof); the outcome dashboard updates automatically.
- Automated reminders are sent to Fellows on configurable thresholds.

## 8.5 Parallel – Mentorship & LMS Journey

Detailed in Section 12 (Mentorship Flow) and Section 13 (LMS Journey). Both are delivery-Phase-3 roadmap capabilities.

---

# 9. Detailed Business Requirements

Each module lists requirements with a unique Req ID, priority (Must / Should / Could), and the delivery phase. The phase column corresponds to the release plan in Section 14. Requirements introduced or revised in v4.0 to reflect Samavesh's wireframe feedback are noted in the requirement text.

## 9.1 Module 1 – Student Onboarding

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| ONB-01 | A Fellow shall create and submit a digital Student Onboarding Form capturing personal, educational, socio-economic, and family background data. | Must Have | Phase 2 |
| ONB-02 | Each student shall have a unique profile (Student ID) acting as the master record across all modules. | Must Have | Phase 2 |
| ONB-03 | The onboarding form shall capture, at minimum: name, DOB (with auto-computed age), gender, caste/category, religion, annual family income, state, district, block, school/college, class/year, %/CGPA, contact details, and guardian information. | Must Have | Phase 2 |
| ONB-04 | Before creating a new profile, the system shall search for an existing student (by email or full name) and block duplicate creation. | Must Have | Phase 2 |
| ONB-05 | The system shall capture digital consent (typed-name signature) from the student or guardian at onboarding, in compliance with DPDP; consent gates submission. | Must Have | Phase 2 |
| ONB-06 | A student profile shall be editable only by the assigned Fellow and Program Admin. The system shall maintain an audit trail (field, old value, new value, user, timestamp). | Must Have | Phase 2 |
| ONB-07 | The onboarding form shall be mobile-responsive (PWA) so Fellows can onboard students on a phone in the field. | Must Have | Phase 2 |
| ONB-08 | Aadhaar and bank-account numbers, if captured, shall be masked in list views and visible only on the detail page with role permission; each reveal is logged. | Must Have | Phase 2 |
| ONB-09 | The system shall allow tagging of students by geography (state / district / block) and institution to enable segmented MIS. | Should Have | Phase 4 |
| ONB-10 | *(v4.0 — feedback ID 1)* At student sign-up, the system shall verify email ownership via a six-digit email OTP before creating the account. Authentication is Google SSO (primary) with email OTP verification for the student self-service path. | Must Have | Phase 2 |
| ONB-11 | *(v4.0 — feedback ID 10)* The system shall provide a shareable onboarding-form link that a student can use to complete the same onboarding form independently; the submission enters a Fellow approval gate before the student's full portal unlocks. | Must Have | Phase 2 |
| ONB-12 | *(v4.0 — feedback ID 10)* A Fellow shall review the complete self-submitted onboarding form, edit if required, and Approve it; approval unlocks the student's full portal and inserts the student into the Fellow's My Students list. | Must Have | Phase 2 |
| ONB-13 | *(v4.0)* A student shall be able to view their submitted onboarding form read-only and raise a "request a change" note to the assigned Fellow; the student shall not edit an approved profile directly. | Must Have | Phase 2 |

## 9.2 Module 2 – Scholarship Catalogue & Eligibility

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| SCH-01 | The platform shall maintain a configurable Scholarship Catalogue (DocType) with criteria per scholarship: eligible states, eligible categories, income limit, educational level, minimum academic performance, age limit, and gender. | Must Have | Phase 2 |
| SCH-02 | On profile create or update, the system shall auto-compute the list of scholarships the student is eligible for. | Must Have | Phase 2 |
| SCH-03 | For each eligible scholarship, the system shall auto-generate a document checklist from the scholarship's required-document configuration. | Must Have | Phase 2 |
| SCH-04 | The scholarship catalogue shall be CRUD-editable by a Program Admin without developer intervention. | Must Have | Phase 2 |
| SCH-05 | Each scholarship record shall carry metadata: governing body, portal URL, deadline, benefit amount, application window. | Should Have | Phase 2 |
| SCH-06 | Fellows and Admins shall be able to manually override eligibility with a mandatory reason field captured for audit. | Should Have | Phase 3 |
| SCH-07 | The system shall notify the assigned Fellow and student when a scholarship application deadline is approaching (configurable, e.g., 7 days). | Should Have | Phase 3 |
| SCH-08 | *(v4.0 — feedback ID 9)* The catalogue shall retain an "Other" free-text scheme-name option, with strict fellow-facing guidance, so a newly encountered scholarship can be recorded and later added to the master. | Should Have | Phase 2 |

## 9.3 Module 3 – Document Collection & Verification

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| DOC-01 | The system shall present a document checklist per student per eligible scholarship, grouped Must Have / Good to Have. | Must Have | Phase 2 |
| DOC-02 | Fellows shall upload documents per checklist item. Document-vault files shall be **PDF only, between 1 MB and 2 MB** (per Fellow SOP). | Must Have | Phase 2 |
| DOC-03 | Each document shall carry a status: **Pending → Uploaded → Under Review → Accepted**, or **Rejected** (re-uploadable). "Under Review" is a document status only. | Must Have | Phase 2 |
| DOC-04 | A Program Admin shall review uploaded documents and Accept or Reject them; rejection shall require a reason. | Must Have | Phase 2 |
| DOC-05 | The system shall track overall document completeness (% complete) per scholarship per student. | Must Have | Phase 2 |
| DOC-06 | A scholarship application shall move to "Ready to Submit" only when 100% of required documents are Accepted. | Must Have | Phase 2 |
| DOC-07 | Documents shall be access-controlled — only the assigned Fellow, Program Admin, and Super Admin may access them (the assigned Mentor sees no documents). | Must Have | Phase 2 |
| DOC-08 | The system shall support re-upload of a rejected document; previous versions are retained via a Re-upload Version field. | Should Have | Phase 3 |
| DOC-09 | Document file storage shall enforce DPDP-compliant encryption at rest (AES-256) on AWS S3 Mumbai. | Must Have | Phase 2 |
| DOC-10 | *(v4.0 — feedback ID 2)* A student shall be able to upload their own additional/follow-up documents from their profile (to replace WhatsApp as the document channel). Onboarding- and scholarship-form uploads remain with the Fellow. | Must Have | Phase 2 |
| DOC-11 | *(v4.0 — feedback ID 5)* For a document the student does not have (marked "Don't have" — mutually exclusive with "Have it"), the system shall present a sub-document dropdown so the student can attach supporting documents; the Fellow can view and download these. Sub-documents are configured for Ration Card, Caste, Domicile, and Income certificates. | Must Have | Phase 2 |
| DOC-12 | *(v4.0 — feedback ID 11 of the June MoM)* A Fellow shall be able to download any document on record for a student at any time (regardless of status), to re-upload on the government portal. | Must Have | Phase 2 |
| DOC-13 | *(v4.0)* Any document-review surface shall present the complete document (preview) before an Accept/Reject decision, not a summary alone. | Must Have | Phase 2 |

## 9.4 Module 4 – Scholarship Application & Tracking (Unified Caseload)

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| TRK-01 | When all documents are Accepted, a Fellow shall record an Application via the Scholarship Data Entry form: scheme, government portal reference / application ID, submission date, submitted PDF, benefit breakdown, notes. | Must Have | Phase 2 |
| TRK-02 | Application status shall follow the canonical state ladder (Section 11): Ready to Submit → Under Scrutiny → Application Approved · funds awaited → Benefits Received; with Re-apply and Rejected as alternate outcomes. | Must Have | Phase 2 |
| TRK-03 | A Fellow shall update application status after checking the government portal, with date stamp and notes. | Must Have | Phase 2 |
| TRK-04 | *(v4.0 — feedback ID 8)* The Fellow's "My Cases" and "Applications" surfaces shall be a single unified **Applications caseload**: one status ladder (single source of truth) advanced by the real work, plus two manual exception flags — **On Hold** (+reason) and **Discarded** (+reason). Work-state is not a second stored field. | Must Have | Phase 2 |
| TRK-05 | The caseload shall present a **derived triage** filter — Action needed / Awaiting / Done / On Hold — computed from status, not hand-set. | Must Have | Phase 2 |
| TRK-06 | On "Benefits Received", the Fellow shall record disbursed amount and date and upload proof; the MIS funnel updates automatically. | Must Have | Phase 2 |
| TRK-07 | The system shall support multiple concurrent applications per student (one per eligible scholarship), each with its own checklist and status. | Must Have | Phase 2 |
| TRK-08 | A Program Admin shall view all applications (oversight), filterable by Fellow and triage, with status read-only (Fellows drive status) and **Reassign** as the admin action. | Must Have | Phase 2 |
| TRK-09 | Automated reminders shall be sent to Fellows when an application has been awaiting a portal decision beyond a configurable threshold. | Should Have | Phase 4 |
| TRK-10 | *(v4.0 — feedback ID 11)* Once a student is onboarded, the Scholarship Data Entry form shall be available on the student's profile (Fellow view), and the resulting application record and status shall be reflected on the student's own view — **never** the government-portal credentials. *(Roadmap: reflection on the student view is planned; see Section 24.)* | Must Have | Phase 2 |

## 9.5 Module 5 – Mentorship Coordination & LMS *(Delivery Phase 3 — Roadmap)*

Detailed functional requirements, business rules, data model, and acceptance criteria are in Sections 12 (Mentorship) and 13 (LMS). These capabilities are planned for delivery Phase 3 and are not part of the current prototype (Section 24).

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| MENT-01 | The platform shall provide a self-service mentorship experience: students discover mentors by filters, view availability, and book one-on-one sessions. | Must Have | Phase 3 |
| MENT-02 | The platform shall generate a video-conferencing link per booking and dispatch confirmations and reminders by email/SMS. | Must Have | Phase 3 |
| MENT-03 | The platform shall capture post-session feedback that is confidential to the Program Administrator and Mentoring Lead. | Must Have | Phase 3 |
| LMS-01 | The platform shall deliver a structured, self-paced LMS (Frappe LMS) with multi-modal content, per-module quizzes, baseline/endline assessments, badges, and a completion certificate. | Must Have | Phase 3 |
| LMS-02 | LMS access in Phase 1 shall be invite-only (partner-nominated); public self-signup is deferred. | Must Have | Phase 3 |

## 9.6 Module 6 – Role-Based Access Control & Audit

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| RBAC-01 | The system shall implement distinct role profiles: Student, Fellow, Mentor, Program Admin, Super Admin. | Must Have | Phase 2 |
| RBAC-02 | Fellows shall see only students assigned to them; cross-Fellow visibility shall be blocked and audited. | Must Have | Phase 2 |
| RBAC-03 | Mentors shall see only assigned mentees, and only mentoring-relevant fields (no income, no Aadhaar, no documents, no student feedback). | Must Have | Phase 3 |
| RBAC-04 | Students shall see only their own profile, documents, application status, and assigned learning content; the student's sole write permissions are uploading their own additional documents/sub-documents and raising change requests. | Must Have | Phase 2 |
| RBAC-05 | A Program Admin shall have full read-write access to all student records, the scholarship catalogue, and reports. | Must Have | Phase 2 |
| RBAC-06 | A Super Admin shall have system-level access including user & role management, configuration, data administration, and the audit log. | Must Have | Phase 2 |
| RBAC-07 | All write actions shall be logged with user ID, role, timestamp, DocType, record name, field, old value, new value. | Must Have | Phase 2 |

## 9.7 Module 7 – MIS Dashboards & Reporting

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| MIS-01 | The Program Admin dashboard shall surface these KPIs: total students onboarded, scholarships identified, applications submitted, applications approved, total scholarship value disbursed. | Must Have | Phase 4 |
| MIS-02 | The dashboard shall support filtering by date range, geography (state/district), Fellow, scholarship, and student category. | Must Have | Phase 4 |
| MIS-03 | A Fellow-wise performance view shall show number of students, documents pending, applications submitted, and approvals achieved. | Must Have | Phase 4 |
| MIS-04 | A scholarship-wise view shall show number of eligible students, applicants, approvals, and total value disbursed. | Must Have | Phase 4 |
| MIS-05 | The system shall export student and application data to Excel / CSV with the active filters applied. | Must Have | Phase 4 |
| MIS-06 | The dashboard shall present the **five-stage funnel** (Section 11.3): Onboarded → Eligibility Identified → Documents Complete → Submitted → Decision, with a Decision breakdown (Approved · funds awaited / Approved · Disbursed / Re-apply / Rejected). | Should Have | Phase 4 |
| MIS-07 | Dashboards shall auto-refresh on data save; no manual refresh required. | Should Have | Phase 4 |
| MIS-08 | *(v4.0 — feedback ID 7)* The Admin shall have an Attendance view with a live "Working now" panel and filters (Fellow / date / status) and export, fed by Fellow log-in/log-out events. | Should Have | Phase 4 |

## 9.8 Module 8 – Data Privacy & DPDP Compliance

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| PVT-01 | Student data collection shall require explicit digital consent at onboarding (DPDP-compliant). | Must Have | Phase 2 |
| PVT-02 | Sensitive personal data (caste, income, disability) shall be accessible strictly on a need-to-know basis per role. | Must Have | Phase 2 |
| PVT-03 | The system shall support a data deletion / anonymization workflow for students who withdraw consent. | Should Have | Phase 4 |
| PVT-04 | All data shall be hosted on the AWS India Region (Mumbai) for data-residency compliance. | Must Have | Phase 1 |
| PVT-05 | Data shall be encrypted at rest (AES-256) and in transit (TLS 1.2+). | Must Have | Phase 1 |

## 9.9 Module 9 – Fellow Field Operations *(v4.0 — new)*

| Req ID | Requirement | Priority | Phase |
|---|---|---|---|
| FLD-01 | *(feedback ID 7)* A Fellow shall log attendance via Log In / Log Out on the home dashboard, capturing time, date, and browser geolocation; a My Attendance screen shall list personal history with date/status filters. Supervisory tiles belong to the Admin view only. | Must Have | Phase 2 |
| FLD-02 | *(feedback ID 12)* A Fellow shall record notes (type: Call / In-person / Virtual, with date-time) on a student's profile; these notes are visible read-only to the student on their own profile. | Must Have | Phase 2 |
| FLD-03 | *(feedback ID 3)* The student home shall provide WhatsApp reach-out actions — one to the assigned Fellow/IP and one to the central Samavesh helpline — via deep-links that pre-fill a message. | Must Have | Phase 2 |

---

# 10. Form Specifications & Business Rules

This section specifies the three primary data-entry forms as validated in the prototype: the **Student Profiling (Onboarding) Form**, the **Document Verification Form**, and the **Scholarship Application (Data Entry) Form**. Field types, mandatory flags, and validations are listed for each. Values marked "to be confirmed by Samavesh" are finalized during the discovery workshop (Section 14.2).

**Design conventions applied to every form** (validated in the prototype): single-page layout with horizontal anchor tabs at the top (no multi-step "Continue" wizards); Date-of-Birth picker with auto-computed age; mobile numbers validated as Indian 10-digit with an inline ✓/✗ chip; Gender driven by a shared master (one source, many consumers); any choice list with three or more options rendered as a dropdown, two-option lists as radios; documents captured through a child-table. These conventions map directly to Frappe DocType field types (Date, Phone-validated Data, Link-to-master, Select, Select-with-optgroup, Table).

## 10.1 Student Profiling (Onboarding) Form

Captured at program Phase 1 (Onboarding). Drives eligibility and the document checklist downstream. Mobile-responsive (PWA). Filled by the Fellow, or self-filled by the student via the shareable link and then Fellow-approved. Approximately 42 questions across eight sections (Welcome & Support → Personal → Location → College & Reference → Education → Documents → Socio-economic → Aspirations & Consent).

| Field Name | Type | Mandatory | Validation / Business Rule |
|---|---|---|---|
| Full Name | Text | Yes | Letters/spaces; min 3 chars |
| Date of Birth | Date | Yes | Auto-computes age; if age < 18, guardian consent is mandatory |
| Gender | Dropdown (master) | Yes | From the shared Gender master (Male / Female / Non-binary / Other / Prefer not to say) |
| Caste / Social Category | Dropdown | Yes | General / OBC / SC / ST / Other; drives category-specific eligibility |
| Sub-Caste | Text | Conditional | Mandatory if category is OBC or SC |
| Religion | Dropdown | Yes | Hindu / Muslim / Christian / Sikh / Buddhist / Jain / Other |
| Annual Family Income (₹) | Number | Yes | ≥ 0; income certificate backs the value if > 0 |
| State | Dropdown | Yes | Bound to a government state master; determines state-portal eligibility |
| District | Dropdown | Yes | Filtered by selected State |
| Block / Taluka | Dropdown | Yes | Filtered by District |
| Pincode | Number | Yes | Six digits |
| Domicile State | Dropdown | Yes | May differ from current State; backed by Domicile Certificate |
| Current Course | Dropdown | Yes | Class 9 to PG levels; controls which scholarships apply |
| Course Year | Dropdown | Conditional | Conditional on Course |
| Institution Name | Link / Text | Yes | Search-as-you-type; new institutions require Admin approval |
| Last Qualifying Exam % | Number | Yes | 0–100; some scholarships have minimum thresholds |
| Mobile Number | Number | Yes | 10 digits; OTP-verified |
| Alternate Contact | Number | No | 10 digits; cannot equal primary |
| Email | Email | Yes | Standard email; primary identifier; OTP-verified at student sign-up |
| Aadhaar Number | Number | Conditional | 12 digits; masked in list views |
| Guardian Name | Text | Yes | Sole consent signatory if student < 18 |
| Guardian Relationship | Dropdown | Yes | Father / Mother / Guardian |
| Guardian Occupation | Dropdown | Yes | Drives socio-economic profiling |
| Housing Status | Dropdown | Yes | Owned / Rented / Government / Other |
| Disability Status | Radio | No | Yes / No; if Yes, Disability Certificate required |
| BPL Status | Radio | No | Yes / No; backed by BPL / Ration card |
| Documents checklist | Child-table | Yes | Have it / Don't have / N/A per document; grouped Must Have (mandatory) and Good to Have; "Have it" makes the attachment compulsory |
| Assigned Fellow | Link | Yes | Auto-set to the logged-in Fellow; editable by Admin only |
| Consent Captured | Typed-name signature | Yes | DPDP consent statement; date-stamped; gates submission |

## 10.2 Document Verification Form

Used by Fellows to upload documents and by Program Admins to verify them. One record per (Student × Document Type).

| Field Name | Type | Mandatory | Validation / Business Rule |
|---|---|---|---|
| Document Name | Link (master) | Yes | From the master list of 18 standard documents (Annexure A) |
| Document Type | Dropdown | Auto | Identity / Income / Caste / Academic / Domicile / Other |
| Document File | File Upload | Yes | **PDF only; 1 MB–2 MB** (per Fellow SOP); single upload per slot |
| Sub-documents | Child-table | Conditional | For a "Don't have" primary document, supporting attachments the Fellow can view/download (Ration, Caste, Domicile, Income) |
| Issued By | Text | Conditional | Mandatory for Caste, Income, Domicile certificates |
| Issue Date | Date | Conditional | Not in the future |
| Validity Date | Date | Conditional | Required for expiry-bound documents |
| Document Number | Text | Conditional | Required for Aadhaar, PAN, Ration Card |
| Uploaded By | Link (User) | Auto | Logged-in Fellow (or student, for self-uploaded follow-up docs) |
| Upload Date | DateTime | Auto | System timestamp |
| Verification Status | Dropdown | Auto | Pending / Uploaded / Under Review / Accepted / Rejected |
| Rejection Reason | Text | Conditional | Mandatory if status = Rejected |
| Reviewer | Link (User) | Auto | Program Admin who set the status |
| Review Date | DateTime | Auto | System timestamp on verification |
| Re-upload Version | Number | Auto | Increments on each re-upload |

## 10.3 Scholarship Application (Data Entry) Form

Filled by the Fellow immediately after the student's application is submitted on the government portal. Approximately 46 questions, English, with a branch at the "Type of Service" question (Scholarship Support vs Technical Support). The submitted application PDF is attached as proof.

| Field Name | Type | Mandatory | Validation / Business Rule |
|---|---|---|---|
| Student | Link | Yes | Auto-linked from profile (auto-fills student details on entering the student's email) |
| Type of Service | Branch selector | Yes | Scholarship Support or Technical Support; toggles the relevant sections |
| Scholarship / Scheme | Dropdown (optgroup) | Yes | Schemes grouped by category; includes an "Other" free-text option (SCH-08) |
| Government Portal Username | Text (masked) | Conditional | Masked with a reveal toggle; security-sensitive — never shown to the student |
| Government Portal Password | Password (masked) | Conditional | Masked with a reveal toggle; security-sensitive — never shown to the student |
| Application Portal | Text / URL | Yes | NSP / State portal URL; defaulted from the scholarship master |
| Application Submission Date | Date | Yes | Cannot be future-dated |
| Government Application ID | Text | Yes | Unique per scholarship |
| Submitted Application PDF | File Upload | Yes | **PDF; max 10 MB** — final submitted application as proof |
| Sanctioned / Expected Amount (₹) | Number | Yes | Expected initially; updated to sanctioned post-approval |
| Application Status | Dropdown | Yes | Ready to Submit / Under Scrutiny / Application Approved / Benefits Received / Re-apply / Rejected |
| Installment grid | 2×2 grid | Conditional | Student / Institute × First / Second, each with amount and disbursement status (Funds Disbursed / Fund Disbursement in Process / Awaiting Approval) |
| Proof of Benefits Received | File Upload | Conditional | Appears when status = Benefits Received; **PDF/JPG/PNG; max 10 MB** — screenshot/document showing funds received |
| Last Status Check Date | Date | Yes | Updated after each portal check |
| Notes | Long Text | No | Additional remarks |

## 10.4 Cross-Form Business Rules

| Rule ID | Business Rule |
|---|---|
| BR-01 | A student can be onboarded only once. Duplicate detection by email or full name blocks creation. |
| BR-02 | Consent is mandatory at onboarding; the profile cannot be submitted without DPDP consent. |
| BR-03 | If the student is a minor (< 18), the guardian is the consenting party and guardian fields are mandatory. |
| BR-04 | Eligibility re-runs automatically on any profile change that is part of an eligibility rule. |
| BR-05 | A scholarship cannot move to "Ready to Submit" unless every mandatory document is "Accepted". |
| BR-06 | Rejecting a document requires a Rejection Reason; the Fellow is notified and the slot becomes re-uploadable. |
| BR-07 | Application status transitions follow the canonical state machine (Section 11); illegal transitions are blocked at UI and API. |
| BR-08 | A student may have multiple concurrent applications (one per eligible scholarship), each with its own checklist and status. |
| BR-09 | On "Benefits Received", disbursed amount and date are mandatory and proof is uploaded; the MIS funnel updates automatically. |
| BR-10 | Fellows can access only their assigned students; cross-Fellow access is blocked and audited. |
| BR-11 | Aadhaar, bank-account numbers, and government-portal credentials are masked by default; portal credentials are never exposed to the student; every reveal of a masked value is logged. |
| BR-12 | Reminders trigger on configurable thresholds (deadlines, ageing awaiting a portal decision). |
| BR-13 | All edits to a student profile create an audit-trail entry; the audit log is read-only and exportable. |
| BR-14 | Withdrawal of consent triggers a DPDP data-deletion / anonymization workflow. |
| BR-15 | Document-vault files outside 1 MB–2 MB, or not PDF, are rejected at upload. The submitted application PDF (≤ 10 MB) and the benefits-received proof (PDF/JPG/PNG, ≤ 10 MB) are separate upload contexts with their own limits. |
| BR-16 | *(v4.0)* "Have it" and "Don't have" are mutually exclusive on any outstanding document; "Have it" enables the attachment and hides sub-documents, "Don't have" reveals the sub-document dropdown. Accepted / Under-Review documents show neither. |
| BR-17 | *(v4.0)* A self-submitted onboarding form is not live until a Fellow approves it; the student remains view-only on an approved profile and effects changes only through a change request. |

## 10.5 File-Handling Rules (Consolidated)

To remove any ambiguity, the platform recognizes three distinct upload contexts, each with its own rule:

| Upload Context | Where | Accepted Formats | Size |
|---|---|---|---|
| Document vault (scholarship certificates & their sub-documents) | Student Documents; Fellow document section | **PDF only** | **1 MB – 2 MB** |
| Submitted application PDF (proof of portal submission) | Scholarship Data Entry form | **PDF** | **≤ 10 MB** |
| Proof of Benefits Received (funds-received screenshot/document) | Scholarship Data Entry form (conditional) | **PDF / JPG / PNG** | **≤ 10 MB** |

---

# 11. Application Status State Machine & Lifecycle Model

All status transitions are validated by the platform; illegal transitions are blocked at the UI and API layers. Version 4.0 uses a single canonical vocabulary — the vocabulary Fellows actually use on the Scholarship Data Entry form and in the Fellow SOP — consistently across the Student, Fellow, and Admin surfaces.

## 11.1 Application Status Ladder (canonical)

| Status | Meaning | Permitted Next States |
|---|---|---|
| Ready to Submit | All required documents are Accepted; the Fellow can submit on the portal. | Under Scrutiny |
| Under Scrutiny | Application filed on the government portal; reference ID captured; portal is processing. | Application Approved · funds awaited / Re-apply / Rejected |
| Application Approved · funds awaited | Application sanctioned; disbursement awaited. | Benefits Received |
| Benefits Received | Funds credited to the student; disbursed amount, date, and proof recorded. | Closed (terminal) |
| Re-apply | Portal flagged the application for correction; the Fellow works and re-submits. | Under Scrutiny |
| Rejected | Application rejected by the portal; reason captured. | Closed (terminal) |

> **Vocabulary note.** "Under Review" is used **only** as a *document* status (Section 10.2 / DOC-03). When a government portal returns an application for correction, the application status is **Re-apply**. Application status, journey stage, and document status are deliberately maintained as three separate vocabularies to keep every surface consistent.

## 11.2 Unified Applications Caseload (Fellow) & Oversight (Admin)

A **Case = one Student × one Scholarship** — the same entity as an Application. The Fellow works a single **Applications caseload** rather than maintaining two parallel screens.

- **One status ladder (single source of truth), advanced by the real action:** Eligibility Identified → Documents Pending → Ready to Submit → Under Scrutiny → Application Approved · funds awaited → Benefits Received / Re-apply / Rejected. Status advances as a byproduct of real work (e.g., submitting and recording the application moves it to Under Scrutiny; recording disbursement moves it to Benefits Received).
- **Two manual exception flags — the only states a Fellow hand-sets:** **On Hold** (+reason) and **Discarded** (+reason), because the ladder cannot infer them.
- **Derived triage (a computed filter, not a stored field):** **Action needed** (Eligibility Identified / Documents Pending / Ready to Submit / Re-apply) · **Awaiting** (Under Scrutiny / Application Approved · funds awaited) · **Done** (Benefits Received / Rejected / Discarded) · **On Hold** (flag set, overrides). This drives the Fellow's caseload filter chips, the dashboard cards, and the actionable-count badge.
- **Admin oversight** presents the same caseload cross-Fellow, with **read-only status** (Fellows drive status) and **Reassign** as the admin action, filterable by Fellow and triage.

**Production data model (Frappe):** one `Case` DocType — `student`, `scholarship`, `assigned_fellow`, `status` (the ladder Select = single source of truth), `on_hold` (Check) + `on_hold_reason`, `discarded` (Check) + `discarded_reason`. Triage/attention is computed, never stored.

## 11.3 Student-Journey Funnel (five stages, canonical)

The MIS funnel and the per-student mini-funnel use five stages. Disbursement is a **sub-status of the terminal Decision stage**, not a stage of its own.

| # | Stage | Meaning |
|---|---|---|
| 1 | Onboarded | Onboarding complete (form filled, consent captured, profile saved) |
| 2 | Eligibility Identified | The eligibility engine has matched scholarships from the catalogue |
| 3 | Documents Complete | All required documents Accepted by a Program Admin (ready to submit) |
| 4 | Submitted | Application filed on the government portal; tracking begins |
| 5 | Decision | Terminal stage; sub-statuses: Approved · funds awaited / Approved · Disbursed / Re-apply / Rejected |

The relationship between the two views: every application has its own status from the ladder (11.1); a student's overall journey stage (11.3) rolls up to the highest-progress application. Document status (11 vocabulary note) is a third, independent axis. Keeping these three axes distinct — **application status**, **journey stage**, **document status** — is what keeps every surface consistent.

---

# 12. Mentorship Flow *(Delivery Phase 3 — Roadmap; not in the current prototype)*

The mentorship journey runs in parallel to the scholarship workflow. Mentors are subject-matter experts (alumni, professionals, academic counsellors) who guide a student through higher-education readiness, career choices, and life-skills. This section specifies the planned Phase-3 capability; it is documented here as the product vision and is **not** part of the clickable prototype described in Section 24.

## 12.1 Module Overview

The Mentoring module digitizes one-on-one guidance historically delivered in person. It enables students to discover qualified mentors, book sessions, conduct them over a video-conferencing service, and obtain a verifiable record of each engagement. Mentoring is delivered **self-service**: any verified student can discover mentors and book — no Fellow/Admin assignment is required to start. (Whether mentor availability is published by the mentor directly or maintained centrally by a Program Admin is a Phase-3 design detail to be finalized with Samavesh; both models are supported by the same booking experience.)

### In Scope (Phase 3)
- Mentor onboarding, profile management, and skill-tag taxonomy administration.
- Mentor availability calendar (recurring and one-off slots).
- Multi-criteria mentor discovery (search and filter) for students.
- One-on-one session booking, rescheduling, and cancellation.
- Automated session-link delivery and reminder notifications.
- Post-session transcription and dispatch of a session summary (MoM).
- Confidential post-session feedback capture.
- Administrative reporting on session volumes, ratings, and outcomes.

### Out of Scope (Phase 1)
- Paid/fee-bearing mentoring (all sessions free in Phase 1); group/cohort sessions; public display of mentor ratings or written feedback; mentor billing/payouts.

## 12.2 Roles & Responsibilities

| Role | Responsibilities | Permissions |
|---|---|---|
| Student | Discover mentors; book, attend, cancel sessions; submit feedback. | Read own bookings and session MoMs; write bookings and feedback. |
| Mentor | Maintain profile and availability; conduct sessions; review post-session record. | Read own profile/bookings/own-session MoMs; write profile, availability, mentor notes. Mentors see only the mentee's first name and topic — no income, no Aadhaar, no documents, no student feedback. |
| Program Administrator | Onboard/verify mentors; manage taxonomy; review feedback; produce reports. | Read/write all mentor data, bookings, MoMs, feedback. |
| Partner Organisation Administrator | View aggregate mentoring usage of nominated students. | Read aggregated counts/outcomes for own-org students. |

## 12.3 Key Functional Requirements (Phase 3)

| Req ID | Requirement | Priority |
|---|---|---|
| FR-MENT-01 | Onboard a mentor with name, contact, photo, bio, skill/stream/geography tags, languages, mode (Volunteer/Facilitator), and monthly hour commitment. | Must |
| FR-MENT-02 | Enforce an Approved-status workflow before a mentor is publicly listed. | Must |
| FR-MENT-03 | Publish recurring and one-off availability; present only future, un-booked, approved slots to students. | Must |
| FR-MENT-04 | Provide a mentor directory with combinable filters (skill, stream, geography, language, mode) and sorting. | Must |
| FR-MENT-05 | Allow a student to book by selecting a slot, entering a one-line topic, and confirming (≤ 3 clicks). | Must |
| FR-MENT-06 | Generate a unique video-conferencing link per booking and dispatch confirmations + reminders (T-24h, T-1h). | Must |
| FR-MENT-07 | Transcribe the session and dispatch a key-outcome summary and transcript to student, mentor, and Admin. | Should |
| FR-MENT-08 | Capture post-session feedback accessible only to the Program Administrator and Mentoring Lead — never public, never to the mentor. | Must |
| FR-MENT-09 | Enforce the mentor's monthly hour commitment as a hard cap. | Must |
| FR-MENT-10 | Permit cancellation up to T-4h without penalty; later cancellations/no-shows are recorded; two no-shows soft-block a student until an Admin lifts it. | Must |
| FR-MENT-11 | Mentoring is functionally independent of the LMS and open to any verified student (no partner nomination required). | Must |

## 12.4 Non-Functional & Acceptance Highlights
- Support ≥ 2,000 registered students and ~500 bookings/month in Year 1; directory renders < 2s for 200+ mentors; booking completes < 3s.
- Each video link is single-use; transcripts encrypted at rest; feedback and MoMs access-controlled per RBAC (Section 9.6).
- A student can book in ≤ 3 clicks; a booked slot is never selectable by another student; feedback is visible only to Admin/Mentoring Lead.

---

# 13. LMS Journey & Flow *(Delivery Phase 3 — Roadmap; not in the current prototype)*

The LMS module is implemented using the Frappe LMS app, integrated with the platform's user, RBAC, and mentor modules. It delivers Samavesh's curriculum as a structured, self-paced, mobile-friendly learning journey. In the current prototype, "Learning" is a navigation placeholder; the full LMS below is the Phase-3 vision.

## 13.1 Curriculum Architecture
The curriculum is organized on two orthogonal axes, fully configurable by the Program Administrator (no hard-coded module counts or eligibility rules):

| Axis | Value | Definition |
|---|---|---|
| Level | Level 1 – Foundational | Self-awareness, career awareness, pathways, problem solving, critical thinking, decision making, English speaking. All audiences. |
| Level | Level 2 – Advanced | Domain-specific, more demanding modules building on Level 1 (UG/PG by stream). |
| Track | Generic | Mandatory modules every enrolled student completes. |
| Track | Domain | Elective modules in thematic clusters, selected by the student. |

Audience groups: **G-SCH** (classes 9–12), **G-UG** (undergraduate years 1–3), **G-PG** (postgraduate years 1–2).

## 13.2 Module Structure & Sequencing
Every module follows a fixed template: Reading Introduction (mandatory) → Image(s) → Primary Video (mandatory) → Secondary Video → Additional Resources → Audio → **End-of-Module Quiz (mandatory)**. A module cannot be published without its End-of-Module Quiz. Modules unlock sequentially within a track; an optional diagnostic placement test can grant skip-ahead.

## 13.3 Assessment Framework
- **Baseline Assessment** — mandatory at enrolment, before the first module unlocks.
- **End-of-Module Quiz** — on each module; passing gates progression; configurable pass mark (default 60%); retries permitted; responses immutable post-submission.
- **Endline Assessment** — mandatory at ≥ 70% curriculum completion, before certificate issuance.

## 13.4 Key Functional Requirements (Phase 3)

| Req ID | Requirement | Priority |
|---|---|---|
| FR-LMS-01 | Author modules (title, level, track, audience, multi-modal content blocks, quiz with stored answers) under a Draft → Published workflow. | Must |
| FR-LMS-02 | Enforce sequential progression; support an optional diagnostic for skip-ahead. | Must |
| FR-LMS-03 | Track per-student overall completion %, per-module status, partial progress, and a timeline. | Must |
| FR-LMS-04 | Award badges on configurable criteria; issue a completion certificate at ≥ configured threshold (default 70%) with Endline complete. | Must |
| FR-LMS-05 | Restrict access to nominated students (invite-only in Phase 1); provide a Partner-Organisation dashboard of nominees' progress and baseline-vs-endline shift. | Must |
| FR-LMS-06 | Provide the Program Administrator an Impact view (students onboarded, modules completed, badges, baseline-vs-endline shift, retention, partner drill-down). | Must |
| FR-LMS-07 | Support content blocks: rich text, image, uploaded/embedded video, audio, PDF, external link; a "Continue Learning" resume entry point. | Must |

## 13.5 Non-Functional Highlights
Support ~1,000 concurrent learners; module first-paint ≤ 3s on 3G; quiz submission < 2s; WCAG 2.1 AA for reading content and captions; strings externalized for later Hindi/regional localization; assessment data access-controlled per RBAC.

---

# 14. Phase-Wise Release Plan & Milestones

Delivery is staged into four sequential release phases. Each phase is a self-contained milestone with its own deliverables, acceptance criteria, and sign-off owner. Durations are indicative and assume the assumptions in Section 18 hold; the largest swing factor is the speed of Samavesh inputs in Phase 1 (forms, scholarship catalogue, eligibility criteria).

## 14.1 Release Plan at a Glance

| # | Phase | Duration | Outcome | Sign-off |
|---|---|---|---|---|
| P1 | Discovery, Wireframes & Foundation | Weeks 1–3 | Workshops, finalized forms, **validated clickable prototype (delivered & demoed)**, scholarship catalogue intake, AWS provisioning, DPDP architecture | Program Head + SPOC |
| P2 | Core Platform MVP (Onboarding → Eligibility → Documents → Application Tracking) | Weeks 4–8 | Modules 1, 2, 3, 4, RBAC core, audit trail, Fellow attendance & notes, Fellow PWA | SPOC + Operations Lead |
| P3 | Mentorship & LMS Release | Weeks 9–11 | Mentor module, LMS, student mentoring dashboard, content authoring | SPOC + Mentoring Lead |
| P4 | MIS, Reporting, UAT, Hardening & Go-Live | Weeks 12–16 | MIS dashboards, reports, reminders, DPDP deletion workflow, UAT, training, production cutover | Program Head + SPOC |

## 14.2 Phase 1 – Discovery, Wireframes & Foundation (Weeks 1–3)

A joint discovery sprint between Dhwani and Samavesh. **Status: substantially complete** — a clickable prototype of the Student, Fellow, and Program Admin experiences has been built and demonstrated, Samavesh's screen-by-screen feedback has been captured and incorporated (Section 25), and this BRD v4.0 baselines the result.

### Deliverables
| ID | Deliverable | Status |
|---|---|---|
| P1-D1 | Kick-off & stakeholder workshops | Done |
| P1-D2 | Finalized Student Profiling, Document Verification, Scholarship Application forms | Validated in prototype; formal sign-off pending |
| P1-D3 | Final eligibility-criteria sheet (per scholarship) | Received (in review) |
| P1-D4 | Final scholarship catalogue (required documents per scholarship) | In progress with Samavesh |
| P1-D5 | Clickable prototype of all Phase 2 screens | Done & demoed |
| P1-D6 | Mentorship flow & LMS journey documented | Done (roadmap; Sections 12–13) |
| P1-D7 | KPI dictionary for the MIS dashboard | Pending sign-off |
| P1-D8 | AWS India Region environment (Dev / UAT / Prod), DPDP-aligned | To provision |
| P1-D9 | Sign-off-ready BRD (this document, baselined) | This document (v4.0) |
| P1-D10 | Project plan, RACI, communication cadence | Agreed |

### Acceptance Criteria
- All three forms signed off in writing by the Samavesh SPOC.
- Scholarship catalogue and eligibility-criteria sheet received in the agreed template.
- Prototype approved with no open major issues.
- AWS environment health-check green for Dev and UAT.
- Mentorship and LMS flows approved by the Samavesh Mentoring Lead.

## 14.3 Phase 2 – Core Platform MVP (Weeks 4–8)

Delivers the operational backbone — the four core modules that replace the Google Forms + Excel + WhatsApp stack, plus Fellow attendance and notes.

### Deliverables
| ID | Deliverable |
|---|---|
| P2-D1 | Module 1 – Student Onboarding (DocType, form, consent, audit log, self-onboarding link + Fellow approval gate, email OTP) |
| P2-D2 | Module 2 – Scholarship Catalogue & Eligibility Engine (admin CRUD + auto-eligibility) |
| P2-D3 | Module 3 – Document Collection & Verification (upload, sub-documents, student self-upload, status, admin verification, completeness) |
| P2-D4 | Module 4 – Unified Applications caseload (status ladder, On Hold/Discarded, derived triage, correction/re-apply, multiple concurrent applications) |
| P2-D5 | RBAC core (Student, Fellow, Mentor, Admin, Super Admin), Fellow-scoped data access |
| P2-D6 | Audit trail across modules |
| P2-D7 | Fellow attendance (log-in/out + geolocation) and notes |
| P2-D8 | Fellow-facing PWA (mobile-responsive) |
| P2-D9 | Internal QA cycle with regression suite; fortnightly sprint demos |

### Acceptance Criteria
- Student profile created/edited/viewed with full audit trail.
- Eligibility engine returns the correct scholarship list for 20 agreed sample profiles.
- Document upload → verification → completeness demonstrably blocks application creation until 100% Accepted.
- Application status ladder prevents illegal transitions; caseload triage is derived correctly.
- Fellow sees only assigned students; cross-Fellow access blocked and logged.
- PWA usable on mid-range Android on 4G with no crash in a 30-minute session.

## 14.4 Phase 3 – Mentorship & LMS (Weeks 9–11)

Delivers the parallel mentoring journey and the LMS (Frappe LMS configured and integrated). See Sections 12–13. Deliverables: mentor module, availability & booking, session links, interaction/feedback capture, LMS content library/courses/quizzes, cohort publication, student mentoring dashboard, engagement analytics, certificates.

## 14.5 Phase 4 – MIS, Reporting, UAT & Go-Live (Weeks 12–16)

Delivers leadership MIS, hardens the platform, runs UAT, conducts training, and takes the system live with hypercare. Deliverables: MIS dashboard (P0 KPIs), filters & five-stage funnel, Fellow-wise and scholarship-wise views, Attendance admin view, Excel/CSV exports, automated reminders, DPDP deletion/anonymization workflow, UAT cycle, training (train-the-trainer + cohorts), production cutover, and two-week hypercare.

---

# 15. Deliverables Matrix (Phase × Module)

| Deliverable | Phase 1 | Phase 2 | Phase 3 | Phase 4 |
|---|---|---|---|---|
| Discovery workshops | ✔ | | | |
| Clickable prototype | ✔ | | | |
| Form specs final + validations | ✔ | | | |
| Scholarship catalogue loaded | Drafted | ✔ Loaded | | |
| Student onboarding (+ self-onboarding, OTP) | | ✔ | | |
| Eligibility engine | | ✔ | | |
| Document verification (+ sub-docs, student upload) | | ✔ | | |
| Unified Applications caseload | | ✔ | | |
| Fellow attendance & notes | | ✔ | | |
| RBAC + audit trail | | ✔ Core | Extended (Student/Mentor) | |
| Fellow PWA | | ✔ | | |
| Mentor module | | | ✔ | |
| LMS – content, courses, quizzes | | | ✔ | |
| Student mentoring dashboard | | | ✔ | |
| Certificates | | | ✔ | |
| MIS dashboard (P0 KPIs) + Attendance admin | | | | ✔ |
| Filters & five-stage funnel | | | | ✔ |
| Excel / CSV exports | | | | ✔ |
| Automated reminders | | | | ✔ |
| DPDP consent capture | | ✔ | | |
| DPDP data-deletion workflow | | | | ✔ |
| UAT · Training · Cutover & Hypercare | | | | ✔ |

---

# 16. Project Governance & Sign-Off Matrix

## 16.1 Governance Cadence
- Daily 15-minute stand-up between the Dhwani delivery lead and the Samavesh SPOC during active sprints.
- Weekly 60-minute steering review between the Dhwani Project Manager and the Samavesh Program Head.
- Fortnightly sprint demo to the wider Samavesh team.
- All scope/change requests raised via the Change Control process (Section 18.3).

## 16.2 Sign-Off Matrix

| Deliverable | Owned By (Dhwani) | Signed Off By (Samavesh) | Target |
|---|---|---|---|
| BRD v4.0 (this document) | Project Manager | Program Head & SPOC | End of Phase 1 |
| Finalised forms (Profiling / Document / Application) | BA + Tech Lead | SPOC + Operations Lead | End of Phase 1 |
| Scholarship catalogue & eligibility criteria | BA (handshake) | SPOC (owns content) | End of Phase 1 |
| Clickable prototype | UX | SPOC + Operations Lead | End of Phase 1 |
| Mentorship & LMS flow | BA + Tech Lead | Mentoring Lead | End of Phase 1 |
| KPI dictionary | BA | Program Head | End of Phase 1 |
| Phase 2 MVP | Tech Lead + QA Lead | SPOC + Operations Lead | End of Phase 2 |
| Phase 3 Mentorship/LMS | Tech Lead | Mentoring Lead | End of Phase 3 |
| Phase 4 UAT closure & Go-Live | Project Manager + QA Lead | Program Head + SPOC | End of Phase 4 |

## 16.3 Communication Channels
Project updates via the kick-off email thread (single source of written updates); day-to-day coordination via a shared chat channel; a Dhwani-managed document repository for signed-off documents; issue tracking in the Dhwani tracker with shared visibility during UAT.

---

# 17. Non-Functional Requirements

| Category | Requirement |
|---|---|
| Performance | Page load within 3 seconds for up to 500 concurrent users on a standard broadband connection. |
| Availability | 99.5% uptime excluding scheduled maintenance (announced 24 hours in advance). |
| Scalability | Architecture shall support 20,000+ active student records without re-platforming. |
| Mobile Accessibility | Responsive PWA functional on Chrome / Firefox / Safari on Android and iOS. |
| Notifications | Email and SMS for key events; WhatsApp deep-link reach-out from the student home. |
| Data Backup | Automated daily backups with 30-day retention; restore tested quarterly. |
| Security | TLS 1.2+ in transit; AES-256 at rest; role-based access; full audit log; masked sensitive fields (Aadhaar, bank, portal credentials). |
| DPDP Compliance | Architecture and data handling comply with the India DPDP Act 2023. |
| Browser Support | Last two versions of Chrome, Firefox, Safari, Edge. |
| Accessibility | Reasonable WCAG 2.1 AA conformance for student-facing pages (Phase 3 onward). |
| Language | English UI in Phase 1 (Marathi/bilingual UI deferred to a later phase). |

---

# 18. Assumptions, Dependencies, Scope & Change Control

## 18.1 Assumptions
- Samavesh nominates a dedicated SPOC for the duration of implementation.
- At least two senior Samavesh staff are authorized to provide milestone sign-offs.
- The scholarship catalogue (criteria, required documents) is provided by Samavesh in the agreed template.
- Fellows and students have access to internet-connected devices.
- Samavesh provides feedback within 48 hours during review and UAT cycles.
- Samavesh participates in training and owns day-to-day system administration post go-live.
- AWS hosting costs are passed through to Samavesh; Dhwani provisions and operates the environment.

## 18.2 Dependencies
- Availability of Samavesh operations, mentoring, and M&E staff for Phase 1 workshops.
- Finalization of the scholarship catalogue and eligibility criteria before eligibility-engine build.
- KPI sign-off before MIS dashboard development.
- AWS India Region account setup and credentials (Dhwani provisions; costs billed to Samavesh).
- Mentor roster, expertise tags, and seed LMS content provided in Phase 3.

## 18.3 Change Control
Any change beyond the baselined requirements follows the Change Control process: a Change Request is raised, Dhwani provides an impact assessment (effort, timeline, cost), and the Samavesh Program Head approves before it enters a sprint. Approved CRs are appended with a new version number.

## 18.4 Out of Scope (Phase 1 Project)
- Integration with government scholarship portals (NSP, state portals) — Fellows continue manual status updates.
- Native mobile application (Android / iOS) — replaced by the PWA.
- Full offline functionality.
- Legacy data cleaning, deduplication, or transformation — Samavesh owns data preparation.
- Integration with any third-party CRM, ERP, payments, or HRMS systems.
- Multi-language UI — English in this project.
- **Campaign creation with a per-campaign form builder** (partner-specific onboarding forms) — flagged operationally useful by Samavesh but out of Phase 1 scope; a "Bulk Upload" surface is stubbed for future use, and campaign/form-builder scope is subject to separate scoping and budgeting.
- **DigiLocker / third-party document-verification API** — under internal Dhwani–Samavesh discussion; not committed for Phase 1.

Items listed as out of scope may be considered in a subsequent phase subject to separate scoping.

---

# 19. Risks & Mitigations

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Scope creep during development | High | High | Strict change-request process with impact assessment |
| Delayed feedback / sign-off by Samavesh | Medium | High | Weekly reviews; escalation path; sign-off SLAs (Section 16) |
| Scholarship catalogue / eligibility data not ready on time | Medium | High | Template shared early; dependency tracked at daily stand-up |
| Low Fellow adoption | Medium | High | Human-centred, mobile-first PWA; validated prototype; training cohorts; in-app help |
| Government portal changes invalidating manual tracking | Low | Medium | Portal integration out of scope; SOP updated if portals change |
| DPDP compliance gaps | Low | High | Consent module, data residency, deletion workflow, and masking built from day one |
| Content readiness for LMS (Phase 3) | Medium | Medium | Seed content agreed in Phase 1; Samavesh nominates a content owner |
| UAT defect leakage to production | Low | Medium | Two-week UAT + two-week hypercare; severity-based exit criteria |

---

# 20. Open Items Awaiting Samavesh Input

| ID | Open Item | Owner (Samavesh) | Status |
|---|---|---|---|
| OI-01 | Final scholarship catalogue with eligibility criteria | SPOC | Eligibility criteria received (in review); catalogue in progress |
| OI-02 | Final list of required documents per scholarship (vs Annexure A) | SPOC | In progress |
| OI-03 | Sign-off on the Student Profiling Form (Section 10.1) | SPOC | Validated in prototype; sign-off pending |
| OI-04 | Sign-off on the Document Verification Form (Section 10.2) | SPOC | Validated in prototype; sign-off pending |
| OI-05 | Sign-off on the Scholarship Application Form (Section 10.3) | SPOC | Validated in prototype; sign-off pending |
| OI-06 | Approval of the Mentorship flow (Section 12) | Mentoring Lead | Open (roadmap) |
| OI-07 | Approval of the LMS journey (Section 13) | Mentoring Lead | Open (roadmap) |
| OI-08 | KPI dictionary for the MIS dashboard | Program Head | Open |
| OI-09 | Mentor roster, expertise tags, content owners | Mentoring Lead | Open (Phase 3) |
| OI-10 | UAT user nominations (Fellow, Mentor, Admin) | SPOC | Open (Phase 4) |
| OI-11 | Confirm whether the mentor availability is mentor-published or admin-maintained (Section 12.1) | Mentoring Lead | Open (Phase 3 design detail) |
| OI-12 | Confirm the Frappe amendment flow for post-submission edits (request → Admin approve → cancel → re-edit → resubmit) | SPOC | Open |

---

# 21. UAT, Training & Knowledge Transfer

## 21.1 UAT Approach
UAT runs for two weeks in Phase 4. Test cases are co-authored by Dhwani QA and the Samavesh SPOC, organized by module. Samavesh nominates three UAT testers (Fellow, Mentor, Admin personas). Defects are logged in the Dhwani tracker with shared visibility; Severity-1 and -2 must clear before go-live. Exit criteria: zero open S1, an agreed plan for open S2, all critical journeys signed off.

## 21.2 Training Plan
Train-the-trainer for two Samavesh champions (4 hours, hands-on); two live Fellow cohorts (2 hours each, recorded); one Mentor cohort (1 hour); one Admin cohort (2 hours); written manuals and short how-to videos.

## 21.3 Hypercare
Two weeks post go-live with a daily stand-up and same-day defect triage; Severity-1 incidents addressed within 4 business hours.

---

# 22. Post Go-Live Support Model

Support SLAs are governed by the signed MSA/SOW and summarized here for context: Tier 1 support by Samavesh in-house champions; Tier 2 by the Dhwani support desk (ticket-based, email + portal). Severity-1 (production down): response within 4 business hours, target resolution within 1 business day. Severity-2 (major feature degraded): response within 1 business day, target resolution within 3 business days. Enhancements follow the Change Control process (Section 18.3).

---

# 23. Annexures

## Annexure A – Document Master List
The master list of documents supported for scholarship applications; final inclusion is confirmed during Phase 1. New documents can be added by an Admin without developer involvement.

| # | Document Name | Document Type | Remarks / Applicability |
|---|---|---|---|
| 1 | Domicile Certificate | Identity / Domicile | Mandatory for most schemes |
| 2 | Income Certificate | Income | Mandatory; validity-bound |
| 3 | Aadhaar Card | Identity | Identity proof |
| 4 | Bank Passbook | Banking | For fund transfer |
| 5 | Caste Certificate | Caste | For reserved-category students |
| 6 | Rent Agreement | Address | For select schemes |
| 7 | Caste Validity Certificate | Caste | For professional courses |
| 8 | Non-Creamy Layer Certificate | Caste | For OBC in professional courses |
| 9 | College Fee Receipt | Academic / Institutional | Mandatory |
| 10 | Bonafide Certificate | Academic | Issued by institution |
| 11 | Leaving Certificate (10 & 12) | Academic | Academic proof |
| 12 | Graduation Marksheet | Academic | For PG applicants |
| 13 | 10th & 12th Marksheet | Academic | Mandatory |
| 14 | Self-Declaration | Other | As per scheme |
| 15 | Ration Card | Socio-economic | For selected schemes |
| 16 | Alp Bhu Dharak Certificate | Land / Socio-economic | For selected schemes |
| 17 | BOCW Card | Welfare | For selected schemes |
| 18 | Hostel Certificate | Institutional | For selected schemes |

**Sub-document dependencies (v4.0).** Four primary documents carry a sub-document list the student can attach when they mark "Don't have": **Ration Card, Caste Certificate, Domicile Certificate, Income Certificate**. The exact sub-document lists are maintained as a master and applied as validations.

## Annexure B – Document Upload Guidelines (per Fellow SOP)
- Document vault (scholarship certificates & sub-documents): **PDF only; 1 MB – 2 MB**.
- Submitted application PDF: **PDF; ≤ 10 MB**.
- Proof of Benefits Received: **PDF / JPG / PNG; ≤ 10 MB**.
- Documents must be clear, legible, and properly scanned. Any unclear or invalid document must be re-uploaded before submission.

## Annexure C – Sources Used to Prepare This BRD
- Samavesh BRD v3.0 (May 2026) – baseline content for organizational context, modules, mentorship, and LMS.
- The validated clickable prototype (`wireframe/index.html`) and the client-facing workflow diagram (`wireframe/workflow.html`) – source of the as-built capabilities and screen inventory in Section 24.
- Samavesh's screen-by-screen wireframe feedback tracker (filled) – source of the feedback register in Section 25.
- Minutes of the 12-Jun-2026 wireframe walkthrough – source of confirmed changes, scope decisions, and blockers.
- SAMAVESH Fellow SOP – source of the document master list, upload guidelines, and status states.
- Samavesh Onboarding Form and Scholarship Data Entry form specifications – source of the form field lists in Section 10.

## Annexure D – Document End
This Business Requirements Document constitutes the agreed basis for the Samavesh platform build. Once signed off, any change to scope, schedule, or cost is governed by the Change Control process (Section 18.3).

---

# 24. Current Prototype Scope & Screen Inventory

This section states precisely what the validated clickable prototype demonstrates today, so that any reader — Samavesh, Dhwani pre-sales, or a prospective partner organization — has an accurate picture of maturity. The prototype is a high-fidelity, fully clickable single-application prototype of the platform's Student, Fellow, and Program Admin experiences; production is built on the Frappe Framework per Section 5.

## 24.1 Demonstrated End-to-End (in the prototype)

**Student (Web Portal):**
- Login with Google SSO and student sign-up with six-digit email OTP verification.
- Home — scholarship-journey overview, three stat tiles (Scholarships identified · Documents Accepted · Expected amount) surfaced above the applications list, and WhatsApp reach-out to the assigned Fellow and to the central helpline.
- Self-onboarding — gated portal that opens the onboarding form; on submission, "pending review" until Fellow approval.
- My Onboarding — read-only view of the submitted onboarding form with a "request a change" action that notifies the Fellow.
- Scholarships — one unified "My Scholarships" list of all eligible schemes with status/stage, including eligible-but-not-applied schemes.
- Documents — checklist grouped Must Have / Good to Have with document history; "Have it / Don't have" toggle; sub-document attachment when "Don't have"; student self-upload of additional documents.
- Profile — read-only, including read-only notes recorded by the Fellow.
- Learning — LMS navigation placeholder (Phase-3 roadmap).

**Fellow (Frappe Desk):**
- Dashboard — caseload triage cards and the attendance Log In / Log Out card (time, date, geolocation).
- My Students — status tabs and date filter.
- Onboarding Approvals — inbox of self-submitted onboarding forms; opening one shows the entire form for review/edit/approve.
- Onboarding Form (≈42 questions) — single-page, DOB→age, mobile validation, gender master, documents child-table, typed-name consent.
- Student Detail — Profile (+ notes), Scholarships (eligible schemes), Documents (with sub-document view/download and download-any-document), Applications.
- Applications caseload — one status ladder, On Hold / Discarded flags, derived triage filters.
- Scholarship Data Entry (≈46 questions) — branch by service type, masked portal credentials with reveal toggle, grouped scheme dropdown, application status, installment grid, submitted-PDF and benefits-proof uploads.
- Documentation Application — per-document application and follow-up timeline.
- My Attendance — personal attendance history with filters.
- My Profile.

**Program Admin (Frappe Desk):**
- MIS Dashboard — five KPIs, the five-stage funnel with a Decision breakdown, Fellow-wise and scholarship-wise tables, filters, and export.
- Document Verification — queue with Accept / Reject (reason required) and full-document preview.
- Scholarship Catalogue — list with add/edit (CRUD).
- All Students — cross-Fellow table with stage and Fellow filters.
- Student Detail — oversight with inline Accept/Reject, edit profile, and reassign Fellow.
- Applications oversight — cross-Fellow, read-only status, Reassign.
- Fellows & Users — performance table and Add User (incl. Super Admin role).
- Attendance — live "Working now" panel, filters, and export.
- My Profile.

**Client-facing workflow diagram:** a standalone branded flowchart of the four program phases and the four role flows, with screen nodes that deep-link into the prototype.

## 24.2 On the Roadmap (not in the current prototype)
- **Mentorship** (Section 12) and **LMS** (Section 13) — delivery Phase 3.
- **Super Admin** dedicated administration screens — the role profile is defined; the admin UI is planned.
- **Government-portal integration** — out of scope; status updates remain manual.
- **Scholarship Data Entry reflected on the student view** (application record and status shown to the student, never the portal credentials) — planned.
- **Bulk Upload** — a stubbed admin surface for a future release.
- **DigiLocker / API document verification** and **campaign creation / form builder** — under discussion / out of Phase 1 scope (Section 18.4).

---

# 25. Wireframe Feedback Incorporated

This register maps Samavesh's screen-by-screen wireframe feedback to the resolution reflected in this BRD and the prototype. It is included to make the responsiveness of the build explicit.

| ID | Screen | Feedback | Resolution |
|---|---|---|---|
| 1 | Login | Add email-OTP verification when onboarding a student. | Implemented — sign-up verifies email via a six-digit OTP (ONB-10). |
| 2 | Navigation | Hide the role buttons; log in by ID/password and route by role profile. | Production model: role derived from the user's role profile post-authentication; the prototype keeps quick role buttons only as a demo convenience. |
| 3 | Student Home | Add a WhatsApp reach-out button; move the three stat tiles above "My Scholarship Applications". | Implemented — two WhatsApp deep-links (Fellow, helpline) and the stat tiles reordered (FLD-03). |
| 4 | Scholarship tab | Remove "Also eligible" and "Ready to submit"; show one list of all eligible scholarships with status/stage. | Implemented — single unified "My Scholarships" list. |
| 5 | Documents tab | Add a sub-document dropdown behind each missing primary document (viewable/downloadable by the Fellow), enabled when the student marks "Don't have". | Implemented for Ration, Caste, Domicile, Income (DOC-11, BR-16). |
| 6 | Student Profile | (No change requested.) | No change. |
| 7 | Fellow Home | Add Log In / Log Out for attendance. | Implemented — Fellow attendance + Admin "Working now" view (FLD-01, MIS-08). |
| 8 | My Students / My Cases | Merge the two into a single page. | Addressed by unifying the duplicate Applications and My Cases surfaces into one Applications caseload (single status ladder + triage); My Students remains the roster (TRK-04). |
| 9 | (Catalogue) | Retain an "Other" scheme-name option with strict guidance. | Implemented — "Other" free-text retained (SCH-08). |
| 10 | Onboarding Form | Provide a shareable link for students to self-fill the onboarding form. | Implemented — self-onboarding link + Fellow approval gate (ONB-11, ONB-12). |
| 11 | Scholarship Data Entry | Enable the form on the student's profile once onboarded, and reflect it on the student view. | Form available on the Fellow's student profile; reflecting the resulting application record/status on the student view (never the portal credentials) is planned (TRK-10, Section 24.2). |
| 12 | Student Profile | Add a notes section for Fellows to record call/meeting notes with date-time. | Implemented — Fellow notes (Call / In-person / Virtual), visible read-only to the student (FLD-02). |
| 13–18 | Admin screens | (No change requested — MIS, Cases reassignment, Catalogue, Manage Students, Fellows & Users, Document Verification.) | No change. |

**End of Document.**
