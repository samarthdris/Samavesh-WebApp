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

## How to work (this part is not optional)

**Brainstorm → plan → execute.** Never jump straight to code.

1. **Brainstorm.** Restate what's being asked, in your own words. Surface the open questions and the options, with a recommendation. Get agreement on the *approach* before designing.
2. **Write a spec + plan** under `docs/superpowers/specs/` and `docs/superpowers/plans/`, dated, matching the shape of the existing ones there — those are the worked examples, read one before writing yours.
3. **Get an explicit yes**, then implement.
4. **Render-verify** (see rule 5 and `docs/CONTEXT.md` § Verifying), then ask before committing.

**Ask. Do not assume.** If a requirement is ambiguous, if you can't tell which of two screens is meant, if a client comment could be read two ways — **stop and ask**. A wrong assumption that reaches the client costs far more than a question. Never silently pick an interpretation and build on it. If you must proceed, state the assumption out loud in your reply, clearly labelled, so it can be corrected.

**Never invent. Verify against the file.** This wireframe is one 465 KB file — do not answer from memory or guess at what a function, id, or status label is. `grep` it and read it first. If you catch yourself writing "it probably…" or "this should be…", that's the signal to go and check. Every claim about what the wireframe currently does must come from having just read that code. Same for the client documents: quote them, don't paraphrase from recall.

**Say when you don't know.** "I couldn't find this — where should I look?" is a good answer. A confident wrong answer is the worst one.

**Suggest improvements.** If you spot a bug, an inconsistency, a screen the ripple check missed, a simpler approach than the one asked for — say so. Raise it, recommend, and let Samarth decide. Don't quietly widen the scope and build it, and don't stay silent about a problem because it wasn't in the ticket.

**Report honestly.** If something didn't work, or you skipped part of the task, or you couldn't verify it — say that plainly, with the evidence. Never claim done without having actually checked.

**Keep the docs current.** When a decision gets locked or a rule changes, update `CLAUDE.md` / `docs/CONTEXT.md` / `wireframe/BUILD_STATUS.md` **in the same commit** as the change. Drift between the code and these files is treated as a defect.

> These practices are the same ones used to build everything already in this repo. The rework in this project has come almost entirely from skipping them — assuming instead of asking, and declaring done without clicking.
