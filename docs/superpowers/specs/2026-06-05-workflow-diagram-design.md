# Role Workflow Diagram — Design Spec

**Date:** 2026-06-05
**Deliverable:** A client-facing, branded, algorithmic workflow diagram of the Samavesh wireframe,
modeled on the style of the provided `Samavesh workflow.pdf`, covering the 4 built roles.
**Audience:** Non-technical client management (program/leadership), for presenting/printing.

**Reference:** `Samavesh workflow.pdf` (the original program flowchart) and the built wireframe
`wireframe/index.html` (Student, Fellow, Program Admin, Mentor apps).

## Decisions (user, 2026-06-05)

- **Medium:** a new **self-contained branded HTML page** `wireframe/workflow.html` (own styles, no
  external runtime deps beyond the same web-fonts the wireframe already uses; works offline, prints).
- **Structure:** a **Program Overview** (faithful recreation of the original 4-phase PDF flow,
  on-brand) **+ one flowchart per role** (Student, Fellow, Program Admin, Mentor).
- **Interactivity:** flowchart **screen-nodes deep-link into the live wireframe** — clicking a screen
  node opens `index.html?role=<r>&screen=<id>` (new tab), which auto-logs-in as that role and
  navigates to that screen. Action/decision/annotation nodes are static.

## Visual language (mirrors the PDF)

A legend at the top defines five node types, matching the PDF's coding:

| Node type | Meaning | Style | Clickable? |
|---|---|---|---|
| **Title block** | Section / phase header | dark-teal fill, white text (`--teal-900`→`--teal-700`) | no |
| **Action** (green pill) | A step the user performs | green fill (`--green-50` / `--green` border), pill radius | no |
| **Screen** (yellow rect) | A wireframe screen | marigold fill (`--marigold-50` / `--marigold` border), rounded rect | **yes — deep-links** |
| **Decision** (blue diamond) | A branch point | blue (`--blue-50` / `--blue`), diamond-styled (◆ marker), branch labels on the outgoing arrows | no |
| **Annotation** (pink callout) | A note (RBAC rule, tech need) | soft-pink dashed callout | no |

Connectors are CSS (centered vertical line + ▼ arrowhead between stacked nodes); decision branches
show a small label (YES / NO / status) on the outgoing arrow; loop-backs are drawn as labeled
return arrows. No fragile absolute-positioned geometry — flows are vertical lanes; the Program
Overview is a horizontal 4-phase strip that wraps to vertical on narrow screens.

## Files

1. **New: `wireframe/workflow.html`** — self-contained page:
   - `<head>`: copy the exact Google-Fonts `<link>` tags and the `:root{…}` brand-variable block
     from `index.html` so colors/fonts match; plus a scoped flowchart CSS block (`.wf-*` classes).
   - `<body>`: title header → legend → Program Overview → 4 role sections → footer "back to wireframe".
2. **Edit: `wireframe/index.html`** — add a deep-link handler at the very end of the existing
   `<script>` (after `loginAs`/`showApp`/`go` are defined and the DOM exists):

```javascript
// ---- deep-link from workflow.html (?role=&screen=) ----
(function(){
  const p=new URLSearchParams(location.search);
  const role=p.get('role'); const screen=p.get('screen');
  if(role && ({student:1,fellow:1,admin:1,mentor:1})[role]){ loginAs(role); if(screen) go(screen); }
})();
```

   - `loginAs(role)` already shows the role's app and hides the login overlay; the trailing
     `go(screen)` overrides the default home screen. Invalid/missing `screen` → stays on the role
     home (no error). This is the ONLY change to `index.html`.

## Deep-link targets (role → screen ids)

Screen-node links use these exact `index.html` screen ids:
- **student:** `home`, `scholarships`, `documents`, `mentorship`, `learning`, `profile`
- **fellow:** `f-home`, `f-students`, `f-onboard`, `f-student`, `f-docapp`, `f-scholarship`, `f-applications`, `f-profile`
- **admin:** `a-home`, `a-verify`, `a-catalogue`, `a-students`, `a-student`, `a-fellows`, `a-profile`
- **mentor:** `m-home`, `m-availability`, `m-sessions`, `m-profile`

Link form: `<a class="wf-node screen" href="index.html?role=student&screen=documents" target="_blank">Documents</a>`.

## Content — Program Overview (faithful recreation of the PDF, lightly reconciled)

Horizontal 4-phase strip + a parallel mentoring branch, on-brand. Two reconciliations vs the
original are shown as **pink annotations** so the change is transparent to the client:

- **Phase 1 — Onboarding:** Fellow *initiates onboarding* → **Digital Onboarding Form** *(pink: "mForm")* → *fills form + captures DPDP consent* → *Profile created* → eligibility auto-runs. Parallel arrow → Mentoring Journey.
- **Phase 2 — Eligibility & Discovery:** *Automated eligibility engine* → *List of eligible scholarships* → *Document checklist generated*.
- **Phase 3 — Hands-On Support:** `Documents complete?` ◆ — **No** → *Supports procurement* → *Uploads documents* → **Program Admin verifies (Accept/Reject)** *(pink: "added — verification gate, implicit in the original")* → loop until complete; **Yes** → *Fellow submits on government portal* → *Records application details*.
- **Phase 4 — Tracking, Liaisoning & Re-application:** *Logs into government portal* → `Application status?` ◆ → **Under Scrutiny** (track) / **Re-apply** → *Identifies correction* → *Provides corrected info* → *Re-submits* (loop) / **Approved** → `Funds disbursed?` ◆ → **Yes** → *Records disbursement* → **MIS dashboard auto-updates**. *(pink: "vocabulary reconciled to the build: Under Review→Under Scrutiny, Redirected for Correction→Re-apply")*
- **Parallel — Mentoring Journey:** *Profile created* → *student self-books a mentor* → *Mentoring sessions* → **Learning (LMS)** *(pink: "Planned — Frappe LMS")*.

## Content — the 4 role flows (mapped to built screens)

Each role section has a header band: role name · which Frappe surface (Web Portal / Desk) · demo
login mobile. Bold names below are **screen nodes (clickable)**; italics are action nodes; `◆` are
decisions; “»” notes are pink annotations.

**Student** — Web Portal · `9800000021` · *watches the journey*
Login → **Home** (journey path) → branches: **Scholarships** (view matched schemes + application
status) · **Documents** (view status + history; `Document status?` shown: Pending→Uploaded→Under
Review→Accepted/Rejected) · **Mentorship** (`Slot available?` ◆ → *book a session* → confirmed) ·
**Learning** (» Planned — Frappe LMS) · **Profile** (read-only).
» *Pink: "Student sees status only — the Fellow performs every action (RBAC-04)."*

**Fellow** — Frappe Desk · `9800000011` · *runs the engine (all 4 phases)*
Login → **Dashboard** (caseload KPIs + task list) → **My Students** → `Duplicate exists?` ◆ (search
Name+DOB / Mobile) — Yes → open existing; No → **Onboarding Form** → *capture DPDP consent* →
*profile created (eligibility runs)* → **Student Detail**: Documents tab `Documents complete?` ◆ — No
→ *procure + upload* (loop, Admin verifies) → Yes → **Scholarship Data Entry** → *submit on portal +
record* → `Application status?` ◆ — Re-apply → **Documentation Application** / correction loop;
Approved → *record disbursement* → Benefits Received. Also: **Applications** (cross-student tracking),
**Profile**.

**Program Admin** — Frappe Desk · `9800000001` · *gates & oversees*
Login → **MIS Dashboard** (KPIs / funnel / tables / export) → branches: **Document Verification**
`Document valid?` ◆ — Accept → unlocks "Ready to Submit"; Reject (reason) → back to Fellow (loop) ·
**All Students** → **Student Detail** (oversight; *Edit profile*; *Reassign Fellow*; Accept/Reject) ·
**Scholarship Catalogue** (*add/edit schemes* → feeds eligibility engine) · **Fellows & Users**
(*manage users*).
» *Pink: "Admin gates verification & approval and oversees all Fellows (RBAC-05)."*

**Mentor** — Web Portal · `9800000031` · *parallel lane*
Login → **Home** (next session + monthly stats) → branches: **My Availability** (*publish recurring +
one-off slots* → `Profile Approved?` ◆ → slots visible to students) · **Sessions**: Upcoming → *Join* /
*Mark complete* / `Cancel > T-4h?` ◆ (penalty-free vs recorded no-show) / *+ Mentor note*; Past →
*minutes & transcript dispatched* · **Profile** (bio / tags / mode / monthly hour cap).
» *Pink: "Mentor sees mentee first name + topic only — no PII, no student feedback (RBAC-03)."*

## Reuse / consistency

- Brand tokens and fonts copied verbatim from `index.html` (`:root` vars + font `<link>`s) so the page
  is visually one family with the wireframe.
- Status vocabulary follows the canonical model (memory `samavesh-status-model`): application =
  Under Scrutiny / Application Approved / Benefits Received / Re-apply / Rejected; documents =
  Pending / Uploaded / Under Review / Accepted / Rejected; funnel = MIS-06 stages.
- All terms drawn from the BRD/SOP/workflow vocabulary only (memory `samavesh-use-only-context-terms`).

## Out of scope

- No backend, no real auth; the deep-link "login" is the existing demo `loginAs`.
- No changes to any wireframe screen other than the 6-line deep-link handler.
- Super Admin (not built) is not included.
- Editing the source `Samavesh workflow.pdf`.

## Verification

Structural (Grep) + manual browser:
1. `wireframe/workflow.html` exists; opens standalone; shows title, legend (5 node types), Program
   Overview (4 phases + mentoring parallel), and 4 role sections.
2. Legend node styles visually match the PDF coding (green action / yellow screen / blue decision /
   pink note / teal title).
3. Each role section lists the correct screens; every **screen node** is an `<a>` whose href is
   `index.html?role=<r>&screen=<id>` with a valid id from the target table; `target="_blank"`.
4. Clicking a screen node opens the wireframe already logged-in as that role on that screen
   (e.g. Student › Documents; Fellow › Student Detail; Admin › Document Verification; Mentor ›
   My Availability). Test at least one node per role.
5. Decisions render with branch labels (Documents complete? YES/NO; Application status?; etc.);
   loops shown for procurement and correction.
6. Pink annotations present (RBAC notes per role; the two Program-Overview reconciliation notes;
   mForm/LMS tech notes).
7. The page prints cleanly (no clipped flow) and is readable on a narrow screen (phases wrap).
8. **No regression:** opening `index.html` with NO query params still shows the login screen as
   before; the four demo logins still work; deep-link with an unknown role is ignored (login shows).
