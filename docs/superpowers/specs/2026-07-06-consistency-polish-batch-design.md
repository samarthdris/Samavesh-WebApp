# Wireframe Consistency + Polish Batch — Design

**Date:** 2026-07-06
**Status:** Approved (design) — awaiting spec review
**Scope:** `wireframe/index.html`. Student + Fellow (ripple one item to Admin for consistency).
**Origin:** End-to-end Student+Fellow audit (2026-07-06). Items A1–A5 (discrepancies) + B2–B6 (UX). B1 skipped per user.
**Related memory:** `samavesh-cases-tasks` (caseload data), `samavesh-ripple-check`, `samavesh-no-half-baked`, `samavesh-render-verify`, `samavesh-git-workflow`, `samavesh-use-only-context-terms`.

## Problem

The audit found the wireframe holds each student's data in several independent places (static HTML per screen + JS arrays), which drifted after recent features. Plus a few UX gaps. The client clicks through Student + Fellow, so cross-surface consistency matters.

## Canonical data (the source of truth to align everything to)

**Aarti Ramesh Pawar** (the primary demo student):
- **DOB:** 14 August 2005. **Category:** SC. **Course:** B.Com · Year 2. **Income:** ₹1,80,000. **State/District:** Maharashtra · Pune. **Marks:** current 78%, previous 74%. **Mobile:** …21. **Aadhaar:** …4821.
- **Applications (exactly 4):**
  | Scheme | Status |
  |---|---|
  | Post-Matric Scholarship for SC Students (PM) | Under Scrutiny |
  | Maharashtra State Minority Scholarship (MS) | Ready to Submit |
  | Dr. Babasaheb Ambedkar Scholarship (AB) | **Documents Pending** |
  | Rajarshi Chhatrapati Shahu Maharaj Merit (SM) | Re-apply |
- **Documents:** 9-doc checklist → 5 Accepted · 1 Under Review · 1 Pending · 2 N/A.

Every surface below must reflect these exact values.

## Changes

### 1. Applications consistency (A1)
- **Caseload (`CASES`):** the Shahu-Maharaj Merit *Re-apply* case is currently assigned to **Suresh Jadhav** — retarget it to **Aarti** (student name + avatar) so the caseload shows Aarti's 4 apps, matching her per-student tabs and student view. (Suresh keeps his On-Hold OBC case.)
- **Fellow student→Applications tab (`fp-apps`):** the AB accordion currently shows **"Application Approved · awaiting disbursement" (stage 5)** — the outlier. Change to **Documents Pending (stage 2)** with docs-pending detail, matching the student view + caseload. (A doc-pending scholarship cannot read "approved".)
- After these two edits, Aarti's PM/MS/AB/SM statuses match across: caseload, `fp-apps`, `fp-sch` (Eligible Schemes), and student **Scholarships**.

### 2. DOB + marks consistency (A2)
- The onboarding read-only seed (`MY_ONBOARDING`) currently has DOB **18 Jun 2005 / marks 76%** — change to **14 Aug 2005 / 78% (prev 74%)** so it matches My Profile + the Fellow student-detail profile.

### 3. Document counts (A3 / B4)
- Home stat tile **"5/8 Documents Accepted" → "5 of 9"** (matches the 9-doc checklist and the Home hero's "9 documents"). The Documents screen ("5 Accepted") already agrees.
- The Ambedkar card's **"6 of 8 Accepted" is a per-scheme required-docs count** — a legitimately different metric; it stays as-is.

### 4. Fellow student count (A4)
- Keep **"24 students"** on the Fellow dashboard hero + My Profile (realistic caseload size).
- The **My Students** list count reads **"Showing 10 of 24"** (rest paginated in production). The approval-flow increment (10→11) still works — it becomes "Showing 11 of 24" after an approval.

### 5. Real inline "Edit profile" (A5 / B2)
- Replace the `alert()` stub on the Fellow student-detail (`f-student`) **Edit profile** with **inline edit-in-place**: clicking flips the profile fields (DOB, category, income, course, state/district, marks, mobile) into inputs with **Save / Cancel**; Save writes the values back into the display grid + a green **"Saved · audit-logged"** flash. Reuses the wireframe's existing inline-form interaction language (no `prompt()`, no new modal).
- **Ripple:** apply the same inline edit to the Admin student-detail (`a-student`) Edit profile (same stub exists there) — per the ripple-check rule.

### 6. Fellow My Profile hub (B5)
- **Add detail:** email, joined date, and "focus areas" (the scholarships/regions this Fellow handles) to the currently-thin card.
- **Editable contact:** make **mobile + region** inline-editable (Edit → inputs → Save + "Saved · audit-logged" flash), same pattern as #5. Role / Students-assigned stay read-only.

### 7. Request-a-change discoverability (B6)
- The Home **"Your onboarding submission"** card sub-text gains **"— view or request a change"** so students discover the request action without opening the form. (The read-only screen already carries the actual Request-a-change control.)

## Non-goals (YAGNI)

- No structural single-source refactor — **value-alignment only** (user's chosen depth). The residual "future features can still drift" risk is recorded in the heartbeat as a watch-item.
- No Mentor / Super-Admin changes. ID 11 (Scholarship Data Entry on student view) stays parked. B1 (rename "Join (link soon)") skipped per user.

## Verification (per `samavesh-render-verify`)

Headless-Chrome (`--headless=new` + isolated `--user-data-dir`), read each PNG:
1. **Aarti consistency:** render caseload (`f-applications`), `f-student`→Applications tab, `f-student`→Eligible Schemes, student **Scholarships** — Aarti shows the same 4 apps with the same statuses (esp. AB = Documents Pending) on all.
2. **DOB:** onboarding read-only (`s-my-onboarding`) shows 14 Aug 2005; matches My Profile.
3. **Docs:** Home tile reads "5 of 9".
4. **Count:** My Students header reads "Showing 10 of 24".
5. **Edit profile:** Fellow (+ Admin) → Edit profile → fields become inputs → Save → updated values + flash.
6. **Fellow profile:** email/joined/focus visible; mobile/region editable inline.
7. **Home card:** sub-text includes "view or request a change".

## Decisions (record)

1. Data-unify depth = **value-align across surfaces** (not a refactor).
2. A4 = keep 24, label list **"Showing 10 of 24"**.
3. A5/B2 = **inline edit-in-place**; rippled to Admin.
4. B5 = **more detail + editable contact** (no attendance duplicate).
5. Canonical AB status = **Documents Pending** (fix the `fp-apps` outlier); Aarti's Shahu-Maharaj case moves to her in the caseload.
