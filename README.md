# Samavesh — Student Scholarship & Mentorship Platform

A unified, role-based platform for **Samavesh Action For Impact** that digitises the full student scholarship lifecycle — onboarding, eligibility, documents, application, tracking — plus a parallel mentorship & LMS journey. Built by **Dhwani Rural Information Systems**.

## ▶️ Live Wireframe (clickable prototype)

**https://samarthdris.github.io/Samavesh-WebApp/wireframe/**

Open the link — no download needed. You land on a login screen; use the one-tap demo buttons:

| Role | Demo sign-in | Mobile (manual) |
|------|--------------|-----------------|
| **Student** | "Sign in as Student (Aarti)" | `9800000021` |
| **Fellow** | "Sign in as Fellow (Rahul)" | `9800000011` |

> The prototype uses **dummy data only**. It auto-redeploys on every push to `dev`.
> Instant fallback (no build): [htmlpreview link](https://htmlpreview.github.io/?https://github.com/samarthdris/Samavesh-WebApp/blob/dev/wireframe/index.html)

## What's in the wireframe

- **Student section** — scholarship journey path, eligibility & application status tracking, documents (status + history, read-only), self-service mentorship booking, LMS placeholder, read-only profile.
- **Fellow section** — dashboard (KPIs + workflow tasks), My Students (working status tabs + date filter), Applications tracking, student detail with action tabs, and three fully fillable forms:
  - **Onboarding Form** — 42 questions, bilingual EN/Marathi, 8 steps, document matrix, consent gate.
  - **Scholarship Data Entry** — 46 questions, service-type branch, masked credentials, schemes by category, 2×2 installment grid.
  - **Documentation Application + follow-up**.

## Repository structure

| Path | What |
|------|------|
| `wireframe/index.html` | The single-file clickable prototype (source of truth for development) |
| `wireframe/BUILD_STATUS.md` | Live build checklist / status |
| `Onboarding Form MD File.md` | Onboarding form spec |
| `SAMAVESH_SCHOLARSHIP_DATA_ENTRY_ANALYSIS_AND_SPEC.md` | Scholarship data-entry form spec |
| `Samavesh_BRD_v3.0 copy.docx` | Business Requirements Document |
| `*.pdf` | Source workflow + Google Form references |

## Status

Active development on the **`dev`** branch. Tech target: Frappe Framework v15 + Frappe LMS on AWS Mumbai (DPDP-compliant).

## Contributing

Read [`CLAUDE.md`](CLAUDE.md) first — it carries the client-locked rules, the branch workflow, and the required way of working. Fuller domain context is in [`docs/CONTEXT.md`](docs/CONTEXT.md); what shipped when is in [`wireframe/BUILD_STATUS.md`](wireframe/BUILD_STATUS.md).

`dev` is the live branch and is **protected**. Work on a `feature/*` branch and open a PR into `dev`; it deploys to GitHub Pages once merged and approved.

```bash
git checkout dev && git pull
git checkout -b feature/<short-name>
# ...work, then...
gh pr create --base dev
```

### First-time setup

```bash
# 1. Accept the collaborator invite at github.com/samarthdris/Samavesh-WebApp

# 2. Authenticate the GitHub CLI (needed to open PRs)
gh auth login

# 3. Clone and get on dev
git clone https://github.com/samarthdris/Samavesh-WebApp.git
cd Samavesh-WebApp
git checkout dev

# 4. Install the superpowers plugin (brainstorming / planning / TDD skills)
claude
/plugin install superpowers@claude-plugins-official
```

Then open the live wireframe once to see what you're working on:
**https://samarthdris.github.io/Samavesh-WebApp/wireframe/**

### Starting a session

`CLAUDE.md` loads automatically, so the rules are already in context. Open with an orientation pass before touching anything:

> Read `CLAUDE.md`, then `docs/CONTEXT.md`, then `wireframe/BUILD_STATUS.md`, then open `Feedbacks.xlsx`. Follow the working agreement in `CLAUDE.md` exactly, especially the "How to work" section.
>
> I'm picking up the Samavesh wireframe to work through client feedback. Before any code: tell me in your own words what this project is, what has already shipped, what is parked, and what the hard rules are — so I can confirm you have it right. Then list every open question you have, and flag anything in the repo that looks inconsistent or out of date.
>
> Do not write any code yet, and do not assume anything — ask me.

For each piece of work after that:

> Feedback item `<ID>` from `Feedbacks.xlsx`: `<paste the item>`.
>
> Brainstorm this with me first — restate what's being asked, the options, and what you recommend. Do not assume which screen or which behaviour is meant; ask. Once I agree on the approach, write a spec + plan under `docs/superpowers/`, get my yes, then implement, render-verify, and ask before committing.
