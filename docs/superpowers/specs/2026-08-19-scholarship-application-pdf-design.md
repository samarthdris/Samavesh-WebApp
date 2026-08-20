# Scholarship Application PDF — Upload & Download — Design

**Date:** 2026-08-19
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` only. Touches the Scholarship Data Entry form (`#f-scholarship` / `#schForm`), the Applications accordion on Fellow (`#fp-apps`) and Admin (`#ap-apps`), and the Student's Scholarships screen (`#scholarships`).
**Source:** `New Feedbacks/Module 3.md` ("Point 2 — Complete PDF Upload and Download Option in Scholarship Data Entry Form," "Module 4" in the client's master FRD). Confirmed in scope, Medium priority. Placement/UX decisions below confirmed with Shweta on 2026-08-19.

## Problem

The submitted scholarship application PDF is needed later — by the scholarship provider, for audits, or by the student — but today there's no reliable single place to attach or retrieve it. Worse, the wireframe currently has **two separate, both non-functional** file-upload boxes that appear to represent the same document:

1. Inside the Scholarship Data Entry form (`#schForm`, "Upload Submitted PDF").
2. Inside the Fellow's Applications tab, in the "Ready to Submit → record submission" mini-form ("Submitted Application PDF").

Neither is wired to anything; clicking either does nothing today.

## Goal (confirmed with Shweta)

- **One file per scholarship application.** The Fellow attaches it in the **Scholarship Data Entry form** — that is the single upload point and "source of all reference."
- **Download access, with no deep navigation, from every place that already shows that application:**
  - **Student** — on the Scholarships screen, on that scholarship's own card.
  - **Fellow** — on the student's profile, Applications tab, that scholarship's detail panel (also where they can replace the file, since it's the same underlying record).
  - **Admin** — same spot, their oversight version of the Applications tab — **download only**, matching Admin's existing read-only posture everywhere else in this repo.
- Re-uploading replaces the file — no version history (confirmed in the client spec).
- PDF only; reuse the file-size cap already used for the same kind of upload elsewhere in this form (10 MB, matching the neighbouring "Proof of Benefits Received" field).
- If nothing's uploaded yet, show "Not yet uploaded" — never a broken link or an error.

## Non-goals

- No redesign of the Scholarship Data Entry form beyond this one field's behaviour.
- No change to who can edit the rest of an application's data — this is scoped to the one PDF.
- No Frappe build — wireframe mock only; the production note below is carried forward, not built now.

## One shared data source, four applications, three surfaces

The demo has 4 existing scholarship applications, each already carrying a stable `data-app` key on every surface: `pm` (Post-Matric, Under Scrutiny), `ms` (Minority, Ready to Submit), `ab` (Ambedkar, Documents Pending), `smm` (Shahu Maharaj Merit, Re-apply). A single JS object, `APP_PDF` (keyed by these same 4 strings), backs a `.app-pdf-host[data-app="…"]` container added to:

| Surface | Container already present | Behaviour |
|---|---|---|
| Student `#scholarships` | each `.card.sch2[data-app]`'s `.sch2-side` | Download / "Not yet uploaded" — no upload control |
| Fellow `#fp-apps` | each `.app-acc-item[data-app]`'s `.app-acc-body` | Download / "Not yet uploaded — attach it via the Scholarship Data Entry form" — no upload control *here* (single upload point rule) |
| Admin `#ap-apps` | each `.app-acc-item[data-app]`'s `.app-acc-body` | Download / "Not yet uploaded" — read-only |

All 4 applications get this treatment, not just the demo one — matches the repo's ripple-check rule (a new pattern reaches every sibling row, not a cherry-picked example).

## The redundant second upload box

The Fellow's "Submit & Record Application" mini-form for MS (`#fp-apps`, Ready-to-Submit state) currently has its own "Submitted Application PDF" `<input type="file">`. This gets **replaced** by the same `.app-pdf-host[data-app="ms"]` (download/attach status), so there is only ever one upload mechanism in the whole file, matching Shweta's "source of all reference" instruction.

## Demo data & a stated wireframe simplification

- **Seeded:** `APP_PDF.pm` starts with a demo file already attached (`"PMS-2026-0042-submitted.pdf"`, dated 28 May — matching PM's existing "Submitted 28 May" copy). This is realistic: PM is already "Under Scrutiny," so a submitted PDF should already exist.
- **Live demo:** the Scholarship Data Entry form's "Upload Submitted PDF" field is wired to write into `APP_PDF.pm` when a Fellow fills it and saves. This is a **stated simplification**, not a production design: the Data Entry form here is a single generic form (not scoped to one specific application in this mock — its own "Scheme Name" dropdown has 26 unrelated options and doesn't cleanly map to the 4 demo cases, I checked). In the real Frappe build this is a non-issue — a Data Entry form is always opened against one specific Scholarship Application record, so the file always attaches to the right one automatically. For the wireframe demo, every save from this form targets PM specifically so the interaction is verifiable; MS, AB, and SMM are demonstrated as their permanent "Not yet uploaded" read state (also correct — none of the three has been submitted yet in the story, so an empty state is the accurate depiction anyway).

## UI

- **Present:** "📄 Application PDF" row with the filename + "Download" (Fellow/Admin/Student) + "Replace" (Fellow only, opens the Data Entry form to the relevant field).
- **Absent:** "Application PDF: Not yet uploaded" in muted text; Fellow's copy adds "— attach it via the Scholarship Data Entry form" so it's clear where to go, without another upload box appearing in a second place.

## Production note (Frappe) — carried forward from the client spec

A single `Attach` field (e.g. `application_pdf`) on the Scholarship Application DocType — no child table, no versioning; a re-upload overwrites the field's file reference. Attachments follow the parent document's permissions by default in Frappe; explicitly confirm the Student role's read permission includes this field at implementation time (worth double-checking since other modules in this project restrict some fields from students).

## Verification (render-verify)

1. Student `?role=student&screen=scholarships` — PM's card shows a Download link with the seeded filename; MS/AB/SMM show "Not yet uploaded."
2. Fellow `?role=fellow&screen=f-student` → Applications tab — same 4-way check, PM downloadable + Replace, others "Not yet uploaded — attach via Scholarship Data Entry form"; the old MS record-submission file input is gone, replaced by the shared control.
3. Admin `?role=admin&screen=a-student` → Applications tab — same 4-way check, download-only, no Replace/attach control anywhere.
4. Live: open the Scholarship Data Entry form, fill the Upload field, save → confirm `APP_PDF.pm`'s filename updates and every one of the three surfaces reflects the new file.
