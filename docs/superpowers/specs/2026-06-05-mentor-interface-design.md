# Mentor Interface — Design Spec

**Date:** 2026-06-05
**File touched:** `wireframe/index.html` (single-file clickable prototype)
**Scope:** Add the **Mentor** role app — the self-service facilitator surface from BRD §12
(Mentoring module). One of the two remaining unbuilt roles (the other is Super Admin).

**Reference (authoritative):** BRD v3.0 §7 (roles), §9.6 RBAC-03, §12.4–12.10 (Mentoring:
roles & permissions, use cases UC-MENT-01..09, functional requirements FR-MENT-001..018,
business rules BR-MENT-01..09, data entities). Resolved mentorship model =
self-service booking (memory `samavesh-brd-discrepancies` item 4/6).

## Problem / goal

The platform has five roles (Student, Fellow, Mentor, Program Admin, Super Admin; RBAC-01).
The wireframe currently has Student, Fellow, and Program Admin apps. The **Mentor** app does
not exist. The student side already lets a student discover mentors and book a session
(`#mentorship` screen + booking modal). This spec adds the **inverse**: the mentor's own
surface to publish availability, run booked sessions, keep private notes, and manage their
profile — under a strict privacy boundary.

## Decisions (user, 2026-06-05)

- **Surface:** Web Portal (`.fp`) — the same branded portal skin as the Student app, because a
  mentor is an external volunteer/facilitator (§7), not HQ Desk staff.
- **Scope:** Full journey — Home, My Availability, Sessions, My Profile.
- **Build approach (A):** A dedicated `#mentorApp` + `#mentorBot` portal shell mirroring the
  Student app; availability rendered as a recurring weekly grid + one-off slots list.
- **Demo identity:** **Dr. Meera Joshi** at login mobile `9800000031` — the same mentor name a
  student books on the student side, tying the two sides of the journey together.

## Architecture & wiring

New portal app following the established role-app pattern (each role = its own `#xApp`/`#xBot`
shell + an `ACCOUNTS`/`showApp`/`loginAs` entry).

1. **Shell:** `<div class="app hidden fp" id="mentorApp">` inserted after the Student app block
   (`#studentApp` + `#studentBot`) and before the Fellow app, OR after the Admin app — placement
   is cosmetic; insert it after `#studentBot`'s closing `</nav>` so portal apps sit together.
   Structure mirrors `#studentApp`:
   - `<aside class="rail">` — brand (स / Samavesh / "Mentor"), nav links (`data-go`):
     `m-home` (Home), `m-availability` (My Availability), `m-sessions` (Sessions),
     `m-profile` (My Profile), and a Sign out link (`onclick="logout();return false"`).
     `rail-foot` shows the mentor avatar "M" + name.
   - `<div class="main">` → `<header class="topbar">` (mobile-brand + search + lang + bell) →
     `<main class="content">` holding the four `<section class="screen">` blocks.
2. **Bottom nav:** `<nav class="botnav hidden" id="mentorBot">` with 4 links (Home / Availability
   / Sessions / Profile) — mirrors `#studentBot`. (Student bottom-nav uses `.botnav` without
   `hidden`; mentor must include `hidden` because it starts hidden until login, matching the
   Fellow/Admin bottom-nav which carry `hidden`.)
3. **Screen ids:** `m-home`, `m-availability`, `m-sessions`, `m-profile`. The first screen
   carries `class="screen active"` only if it would be the default-shown app; since login
   controls visibility, give `m-home` `class="screen"` and rely on `showApp` calling
   `go('m-home')` (consistent with how Fellow/Admin first screens are handled).
4. **JS wiring (4 touches in the existing `<script>`):**
   - `ACCOUNTS`: add `'9800000031':{role:'mentor'}`.
   - `showApp`: add `['mentor','mentorApp','mentorBot']` to the role array; and change the
     final `go(...)` line to send mentor → `'m-home'`, i.e.
     `go(role==='fellow'?'f-home':role==='admin'?'a-home':role==='mentor'?'m-home':'home');`
   - Login screen: add a `loginAs('mentor')` demo button in the `.demo .drow` block
     (after the Program Admin button).
   - No change needed to the `[data-go]` delegated handler — it binds at load over all
     `[data-go]` elements including the new ones.

## Screens

### m-home
- Greeting header (eyebrow "Mentoring" + "Welcome, Dr. Meera Joshi").
- **Next-session banner** — a teal gradient card (same idiom as the student `#mentorship`
  upcoming card) showing the next scheduled mentee first name, topic, date/time, duration,
  and a "Join (link soon)" gold button.
- **Stat row** (3 cards): "Sessions this month" (e.g. 6); "Hours used / cap" (e.g. 4.5 / 8 h —
  reflects FR-MENT-012 monthly hour commitment as a hard cap); "My rating" (a bare number, e.g.
  4.9 — see privacy boundary: number only, never the written feedback).
- Quick links to My Availability and Sessions.

### m-availability
- Intro note: published slots become visible to students only after Program Admin approval
  (BR-MENT-01) and only future, un-booked slots are shown to students (FR-MENT-004).
- **Recurring weekly grid:** rows = time bands, columns = Mon–Sun (or a compact day list with
  toggleable time chips). Each slot cell is a button toggled open/closed via `mSlot(el)`;
  booked slots are shown in a distinct "booked" state and are not toggleable.
- **One-off slots list:** existing one-off slots + an "Add one-off slot" affordance with
  date, start time, and duration select (30 / 45 / 60 min, per data entity default).
- Slot-duration and recurring/one-off map to FR-MENT-003 and the MentorAvailability entity.

### m-sessions
- Tabs **Upcoming / Past** (reuse the existing `ftab(btn,id)` with panes `ms-upcoming`,
  `ms-past` — `ftab` toggles `.tabpane.on` globally, harmless across hidden screens).
- **Upcoming** session cards: mentee **first name only** + one-line **topic**, date/time,
  duration, video link. Actions: **Join (link soon)**, **Mark complete** (`mSession(btn,'done')`),
  **Cancel / Reschedule** (`mSession(btn,'cancel')`) with an inline note that cancellation is
  penalty-free up to T-4h and later cancels/no-shows are recorded (FR-MENT-013 / BR-MENT-07),
  and **+ Mentor note** (`mNote(btn)` → simple prompt/inline, note is private/admin-visible).
- **Past** session cards: status pill (Completed / Cancelled / No-show), the topic, and a
  "Minutes dispatched" indicator (transcript + key-outcome auto-sent, FR-MENT-009). No feedback
  is shown (see privacy boundary).

### m-profile
- Editable mentor profile card: avatar "M", name, status pill (Approved).
- Fields (MentorSkillTag / Mentor entities): Bio, Skill tags (Area), Stream/Domain tags,
  Geography tags, Languages, **Mode** (Volunteer / Facilitator), **Monthly hour commitment**
  (the binding cap). A "Save profile" button (prototype alert/inline confirm).
- Note that the profile is publicly listed only after Program Admin approval (BR-MENT-01).

## Privacy boundary (core design constraint — must be visible in the UI)

- Sessions surface **only the mentee's first name + the discussion topic** — **no income, no
  Aadhaar, no documents, no full profile** (RBAC-03). An inline note on m-sessions states this.
- **No student feedback is shown anywhere** in the mentor app — rating/written feedback is
  confidential to Program Admin + Mentoring Lead (FR-MENT-011 / BR-MENT-05). The mentor's own
  aggregate rating may appear as a bare number on Home, never the written content.
- **Mentor Notes** are the mentor's own private notes (MentorNote entity; admin-visible only) —
  a write affordance, not a place to view student data.

## Interactions (fillable, matching prototype conventions)

New JS helpers in the existing `<script>` (self-contained; depend only on `go`/`ftab`):
- `mSlot(el)` — toggle a weekly-grid availability cell between open and closed (skip if
  `.booked`). Visual class toggle only.
- `mSession(btn, action)` — on an Upcoming session card: `'done'` → replace actions with a
  green "Completed" pill; `'cancel'` → replace with a grey "Cancelled" pill. Mirrors the
  `vAct`/`fileAct` in-place mutation pattern.
- `mNote(btn)` — `prompt()` for a note; on submit show a small "Note saved (private)" inline
  confirmation on the card.
- `addOneOff()` — reveal a hidden inline mini-form (date / start time / duration select),
  matching the catalogue's `toggleCatForm` show/hide pattern; on "Add", append a one-off slot
  row to the list and re-hide the form.
- Sessions tabs reuse the existing `ftab(btn,id)` (no new tab function).

## Reuse summary

- **CSS:** prefer reusing existing classes (`.app.fp`, `.rail`, `.nav`, `.nav-link`, `.botnav`,
  `.topbar`, `.content`, `.screen`, `.card`, `.pill`, `.pdot`, `.tabs`, `.tabpane`, `.grid`,
  `.field`, `.tagk`, `.btn*`, `.page-h`, `.eyebrow`, `.stat`, `.mentor`, `.ph`, `.stars`).
  A **small amount of new CSS is permitted** only for the weekly availability grid
  (e.g. `.availgrid`, `.slot-cell`, `.slot-cell.booked`) if no existing class fits — keep it
  scoped and minimal, in the existing `<style>` block, matching the brand tokens
  (teal/marigold CSS vars already defined).
- **JS:** reuse `go`, `ftab`, `showApp`, `loginAs`, `logout`, `setCrumb` (no breadcrumb needed
  for portal — `setCrumb` only targets `crumbAdmin`/`crumbFellow`, so portal screens are
  unaffected). New: `mSlot`, `mSession`, `mNote`, `addOneOff`.

## Out of scope

- Real video conferencing, real transcription, real scheduling backend (prototype only).
- Program-Admin mentor onboarding/approval screens (UC-MENT-01, FR-MENT-017) and the Partner
  Org aggregate view (FR-MENT-018) — those belong to the Admin/Partner roles, not the Mentor app.
- LMS, Super Admin role — unaffected.
- Student-side `#mentorship` screen — unchanged (already built).

## Verification

Manual, in-browser (clickable prototype, treated as final dev source of truth):
1. From login, click the **Mentor** demo button (or sign in with `9800000031`) → lands on
   `m-home` with the mentor portal shell (rail nav Home/Availability/Sessions/Profile).
2. Home shows the next-session banner + stat row (incl. hours used / cap) + my-rating number.
3. **My Availability:** toggling a weekly-grid slot flips its open/closed state; booked slots
   don't toggle; "Add one-off slot" reveals/appends a slot row.
4. **Sessions:** Upcoming/Past tabs switch; an Upcoming card shows mentee first name + topic
   only (no income/Aadhaar); **Mark complete** flips it to a Completed pill; **Cancel** flips to
   Cancelled; **+ Mentor note** prompts and confirms privately; Past cards show status +
   "Minutes dispatched"; **no feedback content is visible anywhere**.
5. **My Profile:** fields render incl. Mode + Monthly hour commitment; Save shows a confirm.
6. Bottom-nav works on a narrow viewport.
7. **No regression:** Student / Fellow / Admin apps and their logins behave exactly as before;
   `showApp` still routes the other three roles correctly; the student `#mentorship` screen and
   booking modal are unchanged.
