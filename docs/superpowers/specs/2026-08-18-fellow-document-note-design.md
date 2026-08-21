# Fellow-to-Student Document Note ("Document Notes") — Design

**Date:** 2026-08-18
**Status:** Draft — awaiting Shweta's yes
**Scope:** `wireframe/index.html` only. Touches the document lists on Student (`#documents`), Fellow (`#fp-docs`), and — pending decision below — Admin (`#ap-docs`).
**Source:** `New Feedbacks/Module_02_Fellow_Document_Revert_Communication.md` (client-confirmed spec; scope re-opened by Shweta on 2026-08-18 after the file's own header marked it out-of-scope).

## Problem

A document's status label (Pending / Uploaded / Under Review / Accepted / Rejected) is the only thing a student sees. If a document is rejected or needs a correction, there is no in-app way for the Fellow to explain *why* or *what to do* — that conversation happens over phone/WhatsApp today, with no record of it.

## Goal

A Fellow can leave a short, timestamped note against one specific document. The student sees it (with the Fellow's real name), gets a notification-bell alert, and cannot edit or delete it. The note is a separate log, never written into the document's status field.

## Non-goals (per the client spec + YAGNI)

- No SMS/email — in-app notification only.
- No automatic note on every status change — a note is only created when the Fellow deliberately writes one.
- Not applied to sub-documents (the Ration/Caste/Domicile/Income sub-doc attachments) — only to the main 9-document checklist, matching the client spec's "mirrors Module 1's `document_type`."
- No Frappe build — this is the wireframe mock; the client spec's DocType/child-table/Notification-Log section is carried forward as the production note below, not built now.

## Reused patterns (no new UX invented)

- **Inline form, no `prompt()`** — same shape as `sendOnbChange` (Request a change) and `addCorrectionNote`.
- **Notification push** — reuse the existing `NOTIF.student` array + bell (`showNotifications`), same mechanism `sendOnbChange` already uses for `NOTIF.fellow`.
- **Attribution** — the existing demo Fellow name, Rahul More, same as everywhere else in the wireframe.

## The three surfaces (ripple check)

The document list is **not** one shared component today — it's three separately-authored markup blocks (a known repo watch-item): Student `#documents` (`.card`/`.arr-card` rows), Fellow `#fp-docs` (`.doc-row` rows), Admin `#ap-docs` (`.vrow` rows). Per the repo's ripple-check rule, a new pattern must reach every sibling surface, so this touches all three markups independently (there is no single array to change once).

| Surface | Role | Behaviour |
|---|---|---|
| Student `#documents` | Read | Sees the note thread under the matching document. No add/edit/delete. |
| Fellow `#fp-docs` | Write | Sees the thread + an "Add note" action per document. |
| Admin `#ap-docs` | Read (proposed) | Sees the thread, read-only — **open question below.** |

Each of the ~9 documents gets a `data-doc="<Document Name>"` attribute added to its row (a stable key, matching the document's display name — no new vocabulary), and a small `.doc-notes-host` container is inserted after it. One shared data object, `DOC_NOTES` (keyed by document name), and one render function populate all three surfaces' containers on page load — this is the "single JS source of truth" *for the notes themselves*, even though the surrounding document rows remain three separate markups.

## Demo data

- **Seeded note (pre-existing, not live-added):** on **Non-Creamy Layer Certificate** (currently Rejected — "Not clearly legible"), one note from Rahul More explaining the same rejection in plain language, dated to match the existing rejection timeline (16 May). This shows the read-only thread rendering correctly without needing an interaction.
- **Live "Add note" demo:** on **Domicile Certificate** (currently Under Review) in the Fellow's `#fp-docs`. Clicking "Add note" reveals a textarea + Send/Cancel (mirrors `startOnbChange`/`sendOnbChange`); Send appends to `DOC_NOTES['Domicile Certificate']`, re-renders the thread on all surfaces that show it, and pushes one entry into `NOTIF.student`.

## UI

- **Fellow row:** existing `.doc-row` gains a small "Add note" ghost-button (next to Download, where present) + a collapsed note-count chip if notes exist (e.g. "1 note"). Clicking either opens/reveals the thread.
- **Thread:** a compact list — Fellow's name (bold) + relative-ish date + note text, oldest first, newest last (matches the onboarding-note / activity-timeline convention already in the file).
- **Student row:** same thread, no controls. If zero notes, nothing renders (no empty-state clutter, per the client spec's stated edge case).
- **Notification:** reuses the existing bell; entry text e.g. "Rahul More added a note on your Domicile Certificate."

## Open question for Shweta

**Should Admin's `#ap-docs` (the read-only oversight view inside Admin's Student Detail) also show the note thread?** The client spec never mentions Admin. Arguments for including it: Admin is the one who Accepts/Rejects, so seeing why a Fellow flagged something could inform that decision, and every other doc-related feature in this repo (sub-documents, turn-pills) already ripples to all three surfaces. Argument against: it's scope not asked for. **My recommendation: include it, read-only, no add-control** — same low-cost ripple as everything else here. Flagging it rather than assuming, per the working agreement.

## Assumptions (stated, not blocking — flag if wrong)

1. Only the 9 main documents get this; sub-documents do not.
2. A note is Fellow-only to create; Student and (if included) Admin are always read-only.
3. One notification is pushed per note (never zero, never more than one) — matches the client's acceptance criteria.

## Production note (Frappe) — carried forward from the client spec

New child table **"Document Communication Log"** on the Scholarship Application (sibling to the existing "Application Document" child table): `document_type` (Select/Link), `note` (Small Text), `logged_by` (Link → User, defaults to session user), `logged_on` (Datetime), `notification_sent` (Check, prevents double-notify on re-save). Use Frappe's built-in `Notification Log` for the in-app alert rather than custom infra. Historical notes stay attributed to whoever logged them even after a Fellow reassignment — no reassignment or hiding of old notes.

## Verification (render-verify, per the repo's standing rule)

1. Render Student `?role=student&screen=documents` — confirm the seeded note appears under Non-Creamy Layer Certificate, attributed to Rahul More, no controls.
2. Render Fellow `?role=fellow&screen=f-student` → Documents tab — confirm "Add note" on Domicile Certificate; inject the add-note flow; confirm the thread updates and `NOTIF.student` gains one new entry.
3. Render Admin `?role=admin&screen=a-student` → Documents tab — confirm the decision from the open question above is reflected correctly (present read-only, or absent).
4. Confirm a document with zero notes (e.g. Aadhaar Card) renders with no empty-state clutter on all surfaces.
