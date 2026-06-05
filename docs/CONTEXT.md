# Samavesh — Project Context (portable)

> This file mirrors the key working context so any machine/teammate (or a fresh Claude Code session) can get up to speed without the local `.claude/` memory. Live status checklist: [`wireframe/BUILD_STATUS.md`](../wireframe/BUILD_STATUS.md).

## What this is
**Samavesh** = a unified, role-based digital platform for **Samavesh Action For Impact** (education NGO, Pune — "from the margins to the mainstream"), built by **Dhwani Rural Information Systems**. It digitises the student scholarship lifecycle + a parallel mentorship/LMS journey, replacing Google Forms + Excel + WhatsApp.

- **Tech target:** Frappe Framework v15 + Frappe LMS, MariaDB, AWS Mumbai (ap-south-1, DPDP residency), S3 docs, PWA. English in Phase 1.
- **Roles:** Student, Fellow (primary operator), Mentor, Program Admin, Super Admin.
- **Delivery:** 4 phases (~14–16 wks): P1 discovery/wireframes → P2 core MVP (onboarding/eligibility/docs/applications) → P3 mentorship+LMS → P4 MIS/UAT/go-live.

## Current deliverable: the wireframe
Single self-contained file **`wireframe/index.html`** — a clickable prototype, treated as the **source of truth for development** (everything must be clickable, not a throwaway mockup).
- **Live:** https://samarthdris.github.io/Samavesh-WebApp/wireframe/ (GitHub Pages from `dev`; root URL redirects there).
- **Architecture:** one login + multiple role app-shells; only the logged-in role renders (`.hidden` toggling). Each role = `#xApp` + `#xBot` (bottom-nav). JS router `go(id)` shows/hides `.screen` sections (unique ids per app). Demo creds in JS `ACCOUNTS` + one-tap demo buttons. To add a role: new `#xApp`+`#xBot` + ACCOUNTS entry + demo button.
- **Demo logins:** Student `9800000021` (Aarti Pawar) · Fellow `9800000011` (Rahul More) · Program Admin `9800000001`.

## Roles & responsibilities (who does what)
- **Student** = read/transparency surface only. Sees journey, scholarship status, document status+history (no actions), self-service mentor booking, LMS placeholder, read-only profile. Does NOT fill any form.
- **Fellow** = the operator. Onboards students (fills the form on their behalf), procures/uploads documents, fills the scholarship data-entry form, follows up on documents and applications, records disbursement.
- **Program Admin** = verifies documents (Accept/Reject — the action neither student nor fellow does), manages the scholarship catalogue & KPIs, sees the MIS dashboards, manages users.
- **Mentor** = self-service mentee sessions (not yet built).

## Document lifecycle (faithful to BRD DOC-03 + workflow)
`Pending → Uploaded → Under Review → Accepted` (or `Rejected` → re-upload, tracked by *Re-upload Version*). Procurement is folded into **Pending** (Fellow procures; workflow "Supports document procurement"). Only when ALL docs Accepted can the application be submitted (BR-05). Student sees status+history only; Fellow uploads; **Admin verifies**.

## Status model (drives the Fellow tabs)
Student lifecycle: **Onboarded → Documents Pending → Submitted (Under Scrutiny) → Approved → Disbursed (Benefits Received)**. Application statuses use the **data-entry form Q33 vocabulary** (Under Scrutiny / Application Approved / Benefits Received / Re-apply / Rejected) — the forms win over BRD §11.

## Source-of-truth documents (in repo)
- `Samavesh_BRD_v3.0 copy.docx` — Business Requirements Document.
- `Onboarding Form MD File.md` — Onboarding form spec (42 Q, bilingual EN/Marathi, 8 steps).
- `SAMAVESH_SCHOLARSHIP_DATA_ENTRY_ANALYSIS_AND_SPEC.md` — Scholarship Data Entry spec (46 Q, branched at Q25, masked credentials, 2×2 installment grid).
- `Samavesh workflow.pdf`, `Scholarship Data Entry - Google Forms.pdf`, `Student Onboarding Form Scholarships.pdf` — source references.

## Hard rules (decisions locked with the client)
1. **Use ONLY terminology/data from the context files** (BRD/SOP/forms). Never invent status labels or pull from outside web research (e.g. no real govt-portal names that aren't in the files).
2. Documents: **PDF only, 1–2 MB**.
3. Mentorship = **self-service booking** model; student self-books. LMS = curriculum model (§13.6–13.16) — nav placeholder for now, not built.
4. Specific scholarship/college names in the wireframe are **sample placeholders** (real catalogue is open item OI-01, awaited from Samavesh).
5. Git: work stays on **`dev`**; no PR/merge to `main` unless asked. Repo is PUBLIC.

## Open questions parked for the client
- Documentation Application form has **no dedicated spec** — currently a derived design; confirm.
- Form-level questions from the MD specs (e.g. "Binary" gender option, duplicate scheme names, single-installment handling, plaintext-password security in the data-entry form).
