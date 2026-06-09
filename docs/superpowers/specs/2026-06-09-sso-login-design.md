# SSO Login — Design Spec

**Date:** 2026-06-09
**File touched:** `wireframe/index.html` (single-file clickable prototype)
**Scope:** Replace the mobile-number login with a unified, role-agnostic, passwordless
auth screen — **Google SSO** (primary) + **email magic link** (fallback for non-Google
emails). Applies the 3 organisational Frappe skills: `frappe-doctype-skill` (data-model
layer), `frappe-dashboard-design` (visual rules), and `frappe-custom-html-block`
(deployment patterns — non-binding here; login isn't a CHB).

**Reference (authoritative):** BRD v3.0 §7 (RBAC roles), §10.1 (Onboarding form fields),
§9.5 ONB-04 / BR-01 (duplicate-student rule). Director feedback (2026-06-09): "We will
not use contact number to login; use Google email or SSO. For students with non-Google
email (Outlook / Hotmail), use magic sign-in." Memories: [[samavesh-frappe-ui]],
[[samavesh-wireframe-scope]], [[samavesh-forms-spec]], [[samavesh-brd-discrepancies]].

## Problem / goal

The current login uses a 10-digit mobile number lookup against a hard-coded `ACCOUNTS`
map (`wireframe/index.html` line 2639). This was a wireframe convenience that does not
match the production auth model. The Director wants the wireframe to reflect the real
production login so the client demo doubles as a sign-off artefact.

**Production model:** all four roles (Student, Fellow, Program Admin, Mentor)
authenticate via **email** — Google OAuth as primary, magic link as fallback. Email is
already mandatory at onboarding (decision today) and uniquely identifies a user. The
role is derived from the Frappe User's Role Profile after authentication — the login
surface does not know or ask about role.

The new login must keep the wireframe **fully clickable** for client demos (i.e., the
existing one-tap `loginAs(role)` shortcuts continue to work, repositioned as explicit
"Demo shortcuts").

## Decisions (user, 2026-06-09)

- **Auth methods:** Google SSO + magic link (Notion / Vercel / Linear pattern). No
  passwords anywhere. No "forgot password" flow.
- **Roles:** all 4 roles use the same login surface; role is derived post-auth from the
  Frappe User Role Profile, not selected on the screen.
- **Onboarding form impact:** email becomes a **new compulsory question** in §10.1. The
  duplicate-check (ONB-04 / BR-01) primary key shifts from `mobile` to `email`; mobile
  becomes secondary (still captured for SMS notifications). **This spec documents the
  change but does NOT implement the form change** — that's a follow-on round.
- **Brand pitch panel kept:** the existing left-pane teal+marigold pitch is a brand
  element (consistent with modern auth — Linear, Notion, Vercel all do this); not a
  dashboard surface, so the `frappe-dashboard-design` "light theme + white surfaces"
  rule does not apply to it.
- **Demo mechanism kept:** the 4 quick `loginAs(role)` buttons remain but are
  re-positioned and re-styled as an explicit **"Demo shortcuts"** section below the real
  auth (not mixed with it). Each maps to a demo email.
- **Magic-link flow:** standard 2-screen pattern (enter email → "Check your email"
  confirmation → click link → app). 10-minute expiry, single-use. No OTP fallback
  (over-engineering for Phase-1 scope).

## Architecture & wiring

### Surface

The login is a **Frappe Web Portal** page (pre-auth), so it uses the `.fp` portal scope
already established for Student and Mentor (memory `samavesh-frappe-ui`). No `.fd`
(Desk) styling here. The page sits outside any role app — it is the entry-point shown
when no role is active, identical to today.

### Two screens + one fake OAuth overlay

| Screen | DOM id | When shown | Notes |
|---|---|---|---|
| Sign-in | `loginWrap` (existing) | App entry, before login | Replaces existing content. Google SSO button + email field + magic-link button + Demo shortcuts. |
| Check-your-email confirmation | `checkEmailScreen` (NEW) | After "Send magic link" | Shows the email entered, Resend, Use a different email. For the wireframe, includes a "Simulate link click" button to advance into the app. |
| Fake Google OAuth picker | `googleOAuthOverlay` (NEW, modal) | After "Continue with Google" | 600ms overlay showing a faked Google account picker; on selection, advances into the role's app. Mock-only — production uses real Frappe Google OAuth Provider redirect. |

### JS wiring (5 touches in the existing `<script>`)

1. **`ACCOUNTS` map** (line 2639): re-key from mobile to email.
   ```js
   const ACCOUNTS = {
     'aarti.pawar@gmail.com':       {role:'student', name:'Aarti Pawar',       provider:'google'},
     'rahul.more@samavesh.org':     {role:'fellow',  name:'Rahul More',        provider:'google'},
     'meera.nair@samavesh.org':     {role:'admin',   name:'Meera Nair',        provider:'google'},
     'dr.meera.joshi@gmail.com':    {role:'mentor',  name:'Dr. Meera Joshi',   provider:'google'},
     // example non-Google for magic-link demo:
     'aarti.outlook@outlook.com':   {role:'student', name:'Aarti (Outlook)',   provider:'magic'},
   };
   ```
2. **`roleFor(mobile)`** (line 2641): rename to `roleFor(email)`, normalise to lowercase
   trim. Default fallback stays `'student'`.
3. **New `doGoogleSSO()`**: show `googleOAuthOverlay` for 600ms (faked picker UI listing
   the 4 demo Google emails), then `loginAs(ACCOUNTS[picked].role)`.
4. **New `doMagicLink()`**: read email from `#loginEmail`, write it into
   `#checkEmailEmail`, hide `loginWrap`'s sign-in card, show `#checkEmailScreen`. The
   "Simulate link click" button calls `loginAs(roleFor(email))`.
5. **`loginAs(role)`** (line 2657): unchanged. The deep-link handler at line 2903 also
   unchanged — `?role=&screen=` continues to bypass auth.

### HTML structure (the new login card)

```
#loginWrap (existing 2-pane shell)
├── .login-art  (left pitch panel — KEPT unchanged)
└── .login-form
    └── .login-card
        ├── .mobile-mark (mobile brand pill — KEPT)
        ├── h1 "Welcome to Samavesh"
        ├── .lsub "Sign in to continue"
        ├── button.btn-google     "Continue with Google"          ← primary
        ├── .divider-or           "──── or ────"
        ├── label "Email"
        ├── input#loginEmail type=email
        ├── button.btn.btn-outline "Send magic link"              ← secondary
        ├── .demo-shortcuts
        │   ├── .dsh-label "Demo shortcuts (this wireframe)"
        │   └── .dsh-grid  (4 buttons, each calls loginAs)
        └── .login-foot "View the role workflows →"               ← KEPT, links to workflow.html

#checkEmailScreen (NEW, sibling of .login-card, hidden by default)
├── icon.envelope (64px, gray-300, per dashboard-design §9.2)
├── h2 "Check your email"
├── p   "We sent a sign-in link to" + strong#checkEmailEmail
├── p.help "The link expires in 10 minutes."
├── button.btn.btn-outline    "Resend link"   ← wireframe: shows a transient toast/inline "Link resent" for 2s
├── a.link                    "Use a different email"   ← wireframe: hides #checkEmailScreen, re-shows .login-card with #loginEmail focused
└── button.btn.btn-ghost.demo "Simulate link click (demo)"        ← wireframe-only; runs loginAs(roleFor(currentEmail))

#googleOAuthOverlay (NEW, fixed-position modal)
├── .gpicker-card
│   ├── img.glogo (Google G mark, inline SVG)
│   ├── h3 "Choose an account"
│   ├── .gpicker-sub "to continue to Samavesh"
│   ├── .gacct (×4 — one per demo Google email; click → loginAs)
│   └── .gpicker-foot "Use another account"
```

## Visual treatment — applying `frappe-dashboard-design`

The login is **portal pre-auth**, so the rules apply selectively. From the 8 critical rules:

| Rule | Applies to login? | How |
|---|---|---|
| 1. Zero whitespace waste | Yes | All `gap` / `padding` multiples of 4 using `--space-*` tokens |
| 2. Names not IDs | N/A | No data shown |
| 3. Indian number formatting | N/A | No numbers |
| 4. India map fitBounds | N/A | No map |
| 5. Frappe sidebar + top tabs | N/A | Pre-auth; no app shell |
| 6. Mock-data banner | N/A | Login isn't mock data |
| 7. Universal filter strip | N/A | Not a list view |
| 8. Light theme + subtle motion | Partial | Form panel + check-email screen = light/white. The pitch panel stays brand (teal gradient) — it's a brand element, not a dashboard surface. |

**Tokens applied** (from `frappe-dashboard-design/REFERENCE.md`):

- Type scale: `h1` = 30px / weight 700 / `--gray-900`; `.lsub` = 14px / `--gray-500`;
  field label = 11px overline, uppercase, letter-spacing 0.06em; help text = 13px / `--gray-500`.
- Spacing: card padding 24px (`--space-6`); inter-field gap 16px (`--space-4`); divider
  margin 20px (`--space-5`); section gap 32px (`--space-8`) between auth block and Demo
  shortcuts.
- Shadow: `.login-card` uses `--shadow-1`; lifts to `--shadow-2` on `:focus-within`.
- Radius: 12px on the card; 8px on buttons & inputs (matches `frappe-dashboard-design`
  button spec §10).
- Animation: card fade-in 300ms (`--ease-out`); button hover 150ms; Google overlay
  fade+scale-in 200ms (`--ease-spring`).
- Font: Inter (already declared at body). No new font import.

**Button hierarchy** (per `frappe-dashboard-design` "one primary CTA per section"):

| Button | Role | Style |
|---|---|---|
| Continue with Google | **Primary auth** (visually first, largest) | White bg + 1px `--gray-300` border + 18px Google G logo (inline SVG) + 14px medium `--gray-800` text. Hover: `--shadow-2`, border `--gray-400`. This is a deliberate override of the dashboard-design "filled blue primary" rule — Google's brand guidelines mandate this exact treatment for any "Continue with Google" button. The primacy is conveyed by position and size, not color. |
| Send magic link | Secondary outlined | bg #fff, 1px border `--teal-700`, color `--teal-700`. Hover: bg `--teal-50`. |
| Demo shortcut tile (×4) | Tertiary chip | bg `--gray-100`, color `--gray-700`, 12px font, fully rounded 9999px. Hover: `--gray-200`. |
| Resend link (check-email screen) | Secondary outlined | Same as magic-link button |
| Simulate link click (demo) | Ghost | Transparent, color `--gray-500`. Small italic "(demo)" suffix. |

**Brand pitch panel (left):** unchanged structurally. Keep the teal gradient
background, marigond accents, the same pitch points. The dashboard-design
"light theme" rule applies to dashboard surfaces, not pre-auth brand canvases —
Linear / Notion / Vercel all use this exact pattern.

## Data-model impact (the `frappe-doctype-skill` layer)

Documented here for the developer who will implement the live Frappe build; the
wireframe itself does not implement these changes this round.

### Frappe User (built-in)

| Field | Change |
|---|---|
| `name` (primary key) | Set to lowercase email. Already the Frappe default — confirm in System Settings. |
| `username` | Derive from email local-part; no UI input. |
| `enabled` | Default 1 (active). |
| `role_profile_name` | NEW per-user assignment — points to one of `Student`, `Fellow`, `Program Admin`, `Mentor` (defined as Role Profile records). |
| `send_welcome_email` | 0 — we issue the welcome via the magic-link flow itself. |

### Auth providers (System Settings)

| Provider | Setting | Value |
|---|---|---|
| Google OAuth | enable_google_login | 1 |
| Google OAuth | google_client_id / google_client_secret | from Google Cloud Console, env-vars |
| Email Login Key (magic link) | enable_email_login_link | 1 |
| Email Login Key | login_with_email_link_expiry_minutes | 10 |
| Email Login Key | login_with_email_link_single_use | 1 |

### Onboarding form §10.1 — change documented, NOT implemented this round

| Position | Question | EN label | MR label | Required | Notes |
|---|---|---|---|---|---|
| After existing "Full name" | NEW Q: Email address | "Email address (Gmail preferred for one-click sign-in) *" | "ईमेल पत्ता *" | **Yes** | Becomes the Frappe User's `name`. Validate format. If non-Gmail, the student gets magic-link auth automatically. |
| Existing "Contact No" | unchanged | unchanged | unchanged | Yes | Kept for SMS notifications only; no longer the auth identifier. |

Source-of-truth file to update later: `Onboarding Form MD File.md` (memory
[[samavesh-forms-spec]]).

### Duplicate check (ONB-04 / BR-01)

**Was:** "Name + DOB" OR "Mobile" — flagged as potential duplicate.

**Becomes:**
1. **Primary key:** `email` — exact match = blocked, not a duplicate flag (email is unique).
2. **Secondary check:** "Name + DOB" OR "Mobile" — flagged as potential duplicate
   (covers the rare sibling-sharing-family-email case).

Source-of-truth file to update later: BRD memory `samavesh-brd-discrepancies` — add a new
resolved-decision entry for auth + email-compulsory.

## Demo mechanism (wireframe stays clickable)

The 4 quick-demo buttons are retained but visually demoted to a small
**"Demo shortcuts (this wireframe)"** section below the real auth, separated by a
32px gap and a small caption. Each shortcut is labelled with the role + demo email:

```
─────── Demo shortcuts (this wireframe) ───────
 [Student] aarti.pawar@gmail.com
 [Fellow]  rahul.more@samavesh.org
 [Admin]   meera.nair@samavesh.org
 [Mentor]  dr.meera.joshi@gmail.com
```

Clicking any shortcut runs `loginAs(role)` directly — no fake OAuth, no magic link, no
wait. This is the path the user (and the client during demo) will mostly use.

The deep-link mechanism (`?role=&screen=` handler at line 2903) is unchanged and
continues to bypass auth — `workflow.html` deep-links still land correctly.

## Estimated diff (wireframe/index.html)

| Layer | Lines added | Lines removed |
|---|---|---|
| CSS | ~80 | ~10 |
| HTML | ~60 | ~35 |
| JS | ~30 | ~10 |
| **Total** | **~170** | **~55** |

Single file. Scoped to the login section + ACCOUNTS map. No changes to role-app shells,
the workflow diagram, the forms, or any role-specific screen. No new external assets
(Google G logo is inline SVG).

## Out of scope (this round)

- Onboarding form §10.1 update (add email Q) — separate round.
- BRD memory update — separate round.
- Real Frappe System Settings configuration — production deployment, not wireframe.
- Onboarding flow rewrite around email-as-primary-key — separate round.
- Password-anything (forgot password, change password) — not part of the auth model.

## Open questions

None remaining at this scope. (Onboarding form changes will be addressed in a follow-on
brainstorm.)
