# Samavesh — working agreement for Claude Code

Read this first, every session. Full domain context: [`docs/CONTEXT.md`](docs/CONTEXT.md).
Current build state / what shipped when: [`wireframe/BUILD_STATUS.md`](wireframe/BUILD_STATUS.md).

## What this repo is

A **clickable HTML wireframe** for the Samavesh scholarship & mentorship platform (built by Dhwani Rural Information Systems for Samavesh Action For Impact). It is the **source of truth for development** — the client reviews it *by clicking it*, and the eventual Frappe build is written from it.

Effectively one file: **`wireframe/index.html`** (~465 KB, all HTML + CSS + one inline `<script>`). Also `wireframe/workflow.html` (client workflow diagram) and `index.html` (root redirect).

**Live:** https://samarthdris.github.io/Samavesh-WebApp/wireframe/ — GitHub Pages, served from **`dev`** root. A merge to `dev` deploys within ~1 min.

## Branch workflow

`dev` is the live branch. It is protected — **do not push to it directly.**

```
git checkout dev && git pull
git checkout -b feature/<short-name>
# ...work...
gh pr create --base dev
```

Samarth reviews and merges. `main` is dormant (behind `dev`, deploys nothing) — do not open PRs to `main` unless asked.

**Ask before every `git commit` and every `git push`.** Not "ask once for the batch" — ask each time. This is a hard gate; a plan that lists commit steps is not commit approval.

Avoid two pushes within ~a minute: GitHub Pages allows one active deployment, so the first reports "Deployment failed" and emails the owner. One push per feature — fold the BUILD_STATUS heartbeat into the same commit.

## Hard rules (client-locked — do not re-litigate)

1. **Use only vocabulary from the context files** (BRD, Fellow SOP, workflow PDF, the two form specs). Never invent a status label, never pull names from web research. If a value genuinely isn't in the files, mark it a sample placeholder.
2. **Nothing half-baked.** Every button, dropdown and card on screen must visibly do something when used. Mock state is fine; missing state is not. No block that just repeats data shown elsewhere on the same screen.
3. **Review gates show the whole artifact.** Any surface where someone approves / verifies / accepts / rejects must render the *complete* record (reusing the component it was captured in) before the decision button. A 6-field teaser above an Approve button is a defect, even if the button works.
4. **Ripple check.** Introduce a pattern (a taxonomy, a layout, a master list) → immediately enumerate every sibling surface that should match, and update them in the same round. Most past rework came from skipping this.
5. **Verify behaviour before saying done** — not just that the HTML parses. Render the screen headlessly and click the thing. Recipe in `docs/CONTEXT.md` § Verifying.
6. **Desktop/web only.** Don't polish mobile viewports or add mobile variants unless asked. Leave existing `@media` rules alone.
7. **Aarti Ramesh Pawar is the canonical demo student.** Her data lives in several places (static HTML per screen + JS arrays). Change one, change all — see `docs/CONTEXT.md` § Canonical demo data.

## Files to leave alone

Never read, reference, or surface these — permanently out of scope:
`CERE_UAP_BRD_v3 5.md` · `SAMAVESH_BRD_GENERATION_PROMPT.md` · `SAMAVESH_BRD_v1.0.md` · `wireframe/frappe-desk.html` · `Samavesh_Wireframe_Feedback_Tracker.xlsx` (the *empty* old tracker — **not** `Feedbacks.xlsx`, which is the real one).

## Process

Brainstorm → write a spec + plan under `docs/superpowers/` → implement → render-verify → ask to commit. Existing specs and plans there are the worked examples; match their shape.

**Confirm scope before building anything.** Propose the approach and wait for an explicit yes — don't kick off implementation off the back of a discussion.
