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
