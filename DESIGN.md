---
version: alpha
name: shrink-182
description: A cutting-mat design system for shrink-182, a small team link shortener. All the small links.
colors:
  primary: "#1E5A43"
  primary-strong: "#15402F"
  on-primary: "#FFFFFF"
  ink: "#17242C"
  slate: "#4A5B63"
  rule: "#6B7D76"
  surface: "#FFFFFF"
  mat: "#E9F1EC"
  snip: "#FFD23F"
  snip-wash: "#FFF8DC"
  error: "#A3201B"
  error-wash: "#FBEAE8"
  admin: "#3D2C7A"
  admin-wash: "#ECE8F7"
typography:
  display:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 36px
    fontWeight: 700
    lineHeight: 1.15
    letterSpacing: -0.01em
  headline:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 24px
    fontWeight: 700
    lineHeight: 1.25
  title:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 18px
    fontWeight: 700
    lineHeight: 1.35
  body:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.5
  body-sm:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 14px
    fontWeight: 400
    lineHeight: 1.45
  label:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 14px
    fontWeight: 700
    lineHeight: 1.3
  code-lg:
    fontFamily: Atkinson Hyperlegible Mono
    fontSize: 24px
    fontWeight: 700
    lineHeight: 1.2
  code:
    fontFamily: Atkinson Hyperlegible Mono
    fontSize: 16px
    fontWeight: 500
    lineHeight: 1.4
  count:
    fontFamily: Atkinson Hyperlegible Next
    fontSize: 18px
    fontWeight: 700
    lineHeight: 1.2
    fontFeature: '"tnum" 1'
rounded:
  none: 0px
  sm: 3px
  md: 6px
spacing:
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 40px
  page-max: 760px
  gutter: 16px
  mat-grid: 24px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label}"
    rounded: "{rounded.sm}"
    padding: 12px
    height: 44px
  button-primary-hover:
    backgroundColor: "{colors.primary-strong}"
    textColor: "{colors.on-primary}"
  button-secondary:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.primary}"
    typography: "{typography.label}"
    rounded: "{rounded.sm}"
    padding: 12px
    height: 44px
  button-danger:
    backgroundColor: "{colors.error}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label}"
    rounded: "{rounded.sm}"
    padding: 12px
    height: 44px
  input:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.ink}"
    typography: "{typography.body}"
    rounded: "{rounded.sm}"
    padding: 10px
    height: 44px
  input-error:
    backgroundColor: "{colors.error-wash}"
    textColor: "{colors.ink}"
  create-mat:
    backgroundColor: "{colors.mat}"
    textColor: "{colors.ink}"
    rounded: "{rounded.md}"
    padding: 24px
  snipped-result:
    backgroundColor: "{colors.snip-wash}"
    textColor: "{colors.ink}"
    typography: "{typography.code-lg}"
    rounded: "{rounded.sm}"
    padding: 16px
  link-row:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.ink}"
    typography: "{typography.body}"
    padding: 16px
  short-code:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.primary}"
    typography: "{typography.code}"
  click-count:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.ink}"
    typography: "{typography.count}"
  alert-error:
    backgroundColor: "{colors.error-wash}"
    textColor: "{colors.ink}"
    typography: "{typography.body}"
    rounded: "{rounded.sm}"
    padding: 16px
  account-bar:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.ink}"
    typography: "{typography.body-sm}"
    height: 56px
  admin-banner:
    backgroundColor: "{colors.admin}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label}"
    padding: 8px
  admin-badge:
    backgroundColor: "{colors.admin-wash}"
    textColor: "{colors.admin}"
    typography: "{typography.label}"
    rounded: "{rounded.sm}"
    padding: 4px
  helper-text:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.slate}"
    typography: "{typography.body-sm}"
  cut-line:
    backgroundColor: "{colors.rule}"
    height: 1px
  snip-edge:
    backgroundColor: "{colors.snip}"
    textColor: "{colors.primary}"
    width: 4px
  token-reveal:
    backgroundColor: "{colors.snip-wash}"
    textColor: "{colors.ink}"
    typography: "{typography.code}"
    rounded: "{rounded.sm}"
    padding: 16px
---

# shrink-182 Design

## Overview

**Brief.**
- **Subject:** shrink-182, a link shortener for one team (under 100 users). It cuts long URLs into 7-character codes.
- **Audience:** Team members who sign in with passkeys, and anyone who follows a short link.
- **Primary jobs:**
  - **Your links (home):** paste a long URL, get a short link to copy, and see each link's click count.
  - **Sign in / sign up:** one passkey action.
  - **Admin:** review and delete links or accounts.
  - **Not found:** tell visitors plainly that a short link doesn't exist.

**First plan and what changed.** The first token plan was white pages, a single blue accent, Inter type, and each link in a rounded shadowed card with a "Copy →" button. That fits any SaaS tool equally well, so it was revised:

- **Color:** the blue became **cutting-mat green**. A self-healing cutting mat is where things get trimmed, and this is shrink-182's subject.
- **Type:** Inter became **Atkinson Hyperlegible**. Codes are 7 random mixed-case letters and digits (FR-2.7), and that family is drawn so `l`/`I`/`1` and `O`/`0` can't be confused. The mono variant is used only for codes and tokens, never for labels.
- **List:** cards became plain ruled rows. The link list is a list, not a gallery.

**The one bold element** is the *cut*. The create form sits on a green mat with a faint 24 px grid, and a dashed cut line separates the long URL from the result. The new short link appears below the line on a marker-yellow strip. Everything else stays quiet: white pages, ink text, left-aligned single column, no decorative motion.

**Answers.** The owner confirmed the design answers on 2026-11-11.

**Voice.** Plain, sentence case, active verbs. The action is named "Shorten link", and its confirmation says "Link shortened". Errors say what happened and how to fix it, and never apologize. Product tagline: *All the small links.*

## Colors

- **Mat green (`primary` #1E5A43):** the only action color: primary buttons, links, short codes, and focus rings. It is 8.08:1 on white and 7.03:1 on the mat.
- **Deep mat (`primary-strong` #15402F):** hover and pressed state for primary buttons. It is 11.63:1 under white text.
- **Ink (`ink` #17242C):** body text and headings, 15.85:1 on white. It is a blue-black chosen to sit with the green, not a neutral near-black.
- **Slate (`slate` #4A5B63):** secondary text such as target URLs, hosts, and helper text. It is 7.08:1 on white and 6.15:1 on the mat.
- **Rule (`rule` #6B7D76):** input borders, row dividers, the dashed cut line, and the mat grid. It is 4.36:1 on white and 3.79:1 on the mat, so it meets the 3:1 rule for UI components.
- **Surface (`surface` #FFFFFF):** page background.
- **Mat (`mat` #E9F1EC):** the create-panel background only.
- **Snip yellow (`snip` #FFD23F) and Snip wash (`snip-wash` #FFF8DC):** mark something that was just made:
  - The wash sits behind a new short link or a one-time API token. Ink is 14.88:1 on it, and mat green is 7.59:1.
  - Solid snip yellow is used only as a 4 px left edge and in the logo. Against the mat green it is 5.6:1.
- **Error (`error` #A3201B) and Error wash (`error-wash` #FBEAE8):** validation messages, the danger button (7.55:1 under white text), and error alerts (ink on the wash is 13.61:1).
- **Admin violet (`admin` #3D2C7A) and Admin wash (`admin-wash` #ECE8F7):** used only inside the admin area, so privileged screens never look like personal ones. White on violet is 11.41:1, and violet on its wash is 9.49:1.

All text, component, and focus pairs above meet WCAG 2.2 AA. Color is never the only signal: errors also carry an icon and text, and admin screens also carry a written banner.

## Typography

- **Atkinson Hyperlegible Next** sets all interface text.
  - The scale steps through 14, 16, 18, 24, and 36 px.
  - Bold (700) is used for headings, labels, and counts. Regular (400) is used for body text.
  - Labels are sentence case, never all caps.
- **Atkinson Hyperlegible Mono** is used only where a person must read or retype exact characters: short codes, short URLs, and API tokens.
- **Click counts** use tabular figures so the counts line up down the list.
- **Line length:** body text stays under 75 characters.
- **Font hosting:** shrink-182 serves both Atkinson Hyperlegible families itself, so pages make no requests to font services. They are licensed under the SIL Open Font License. The fallback stack is the system UI font, and `ui-monospace` for codes.

## Layout

- **Page:** a single left-aligned column, at most 760 px wide, with a 16 px gutter on phones.
- **Spacing:** a 4/8/16/24/40 px scale.
- **Home screen**, top to bottom:
  1. Account bar
  2. Create mat
  3. "Your links" list

```
shrink-182 logo ............................ Signed in · Log out
┌ mat ───────────────────────────────────────────────┐
│ Long URL [_____________________________] [Shorten link] │
│ Title (optional) [______________]                  │
│ - - - - - - - - - - ✂ - - - - - - - - - - - - - - -│
│ ▌ shrink-182.example/k7Qm2Xa      [Copy]                 │
└────────────────────────────────────────────────────┘
Your links (12)
k7Qm2Xa   Q3 planning doc            214 clicks  Edit Delete
          docs.example.com/…/q3-plan
```

- **Wide screens:** link rows show code, title and target, count, and actions in a row.
- **Below 600 px:** each row stacks code, then title, then target, then a line with the count and actions.

- **Theme:** light only. There is no dark theme in the first version.
- **Signed-out root:** a visitor at the site root sees the logo, the line "All the small links.", one sentence on what shrink-182 does, and two buttons: "Sign in with passkey" (primary) and "Sign up" (secondary).

## Elevation & Depth

The design is flat with no shadows. Depth comes from tone: the green mat panel sits on a white page, and the yellow strip marks new results. Confirmation dialogs are the only layer. They use a 2 px ink border and dim the page behind them.

## Shapes

- **Corners:** small and functional. Buttons, inputs, and alerts use 3 px. The mat uses 6 px, because it's the one larger surface.
- **List rows:** no rounding; they're separated by a 1 px rule.
- **The cut line:** a 1 px dashed rule with a small scissors glyph.
- **Focus:** a 2 px solid mat-green outline at a 2 px offset on every interactive element. On the mat background it stays 7.03:1.

## Components

- **Buttons** come in three variants:
  - **Primary:** mat green. There is one per view ("Shorten link", "Sign in with passkey").
  - **Secondary:** white with a 1 px rule border and green text ("Copy", "Edit").
  - **Danger:** error red, shown only inside a confirmation step.

  All buttons are 44 px tall and come in default, hover, focus, pressed, and disabled states. Disabled buttons keep their label and show a "not allowed" cursor. Labels are verbs: "Copy" becomes "Copied" for 2 seconds after use.
- **Input:** a 1 px rule border with a label above it.
  - Helper text goes below, in slate.
  - Error state: the field turns to the error wash with a 2 px error border, and an error message starting with "⚠" appears under the field. The message says what to fix, for example "Use a link that starts with http:// or https://", "Titles can be up to 200 characters", "Remove the username and password from this link" (FR-2.12), "Titles can't contain hidden control characters" (FR-2.13), or "This link points back at shrink-182" (FR-2.10).
- **Create mat:** URL and optional title inputs plus the primary button, on the gridded mat.
- **Snipped result:** appears below the cut line after a link is created. It shows the full short URL in `code-lg` with a 4 px snip-yellow left edge, then a Copy button.
- **Link row:**
  - Shows the short code (mono, green), the title or "Untitled", and the target URL in slate.
  - The target's host is shown in full. A long path is shortened in the middle, and the full URL appears as a tooltip and on the edit screen.
  - Also shows the click count and the Edit and Delete actions.
- **Empty list:** "No links yet. Paste a long URL above to make your first one."
- **Account bar:** logo on the left, signed-in state on the right (see below).
- **Admin banner:** a violet full-width strip reading "Admin view: you're seeing every user's links" at the top of each admin screen, plus an "Admin" badge in the account bar.
- **Token reveal:** shows a new API token once, on the snip wash, with Copy and the line "You won't see this token again. Store it now."
- **Alert:** for page-level errors such as "This short link doesn't exist." It uses the error wash with ink text.

### Security-relevant interactions

- **Signed-in state:**
  - Signed-in pages show the account bar with "Signed in" and a "Log out" button.
  - Signed-out pages show only "Sign in" and "Sign up".
  - Users are identified only as "Signed in" (FR-7.8). The admin user list shows each user's internal ID in the mono face, plus their sign-up date.
  - When a session expires (FR-1.10), the user is returned to sign-in with "Your session ended. Sign in again to continue."
- **Destructive actions are confirmed in a dialog** that names the exact thing affected:
  - **Delete link:** "Delete shrink-182.example/k7Qm2Xa? It will stop working for everyone, and this code won't be reused." The buttons are "Delete link" (danger) and "Keep link". Focus starts on "Keep link".
  - **Revoke token:** a similar dialog.
  - **Delete account:** the dialog states the number of links that will be deleted, and that there is no recovery (FR-1.8, FR-1.11). For an admin, the button is disabled with the helper text "Admins can't delete their own account. Ask the operator to remove your admin access first." (FR-7.10)
  - **Remove passkey:** "Remove the passkey added on 3 March 2026? You won't be able to sign in with it again." The buttons are "Remove passkey" (danger) and "Keep passkey". The last passkey has no Remove button and shows "This is your only passkey, so it can't be removed." (FR-1.14)
  - **Admin deleting another user's link or account:** the dialog also names the owner and is styled with the admin banner.
- **Passkeys on the account page** (FR-1.13): a plain ruled list, one row per passkey, showing "Added" and "Last used" dates, with "Add a passkey" as a secondary button below it. Passkeys have no names.
- **Confirming it's you** (FR-1.15): when an account action needs a recent passkey check, the page shows "Confirm it's you with your passkey to continue" with one primary button, "Confirm with passkey". After it succeeds, the original action carries on without being asked for again. If it fails: "That didn't work. Try again with a passkey saved for this account."
- **What errors reveal:**
  - A link that doesn't exist and a link the user doesn't own produce the same message, "Link not found" (FR-4.13).
  - A failed passkey sign-in says only "Sign-in didn't work. Try again with a passkey saved for shrink-182."
  - When a limit is hit (NFR-1.3): "Too many tries. Wait a minute and try again."
  - Error messages never show internal details.
- **User-supplied content** (titles and target URLs) is always shown as plain text, never as formatted or clickable rich content.
  - Target URLs in the list are shown as text, not live links. The full host is always visible, so lookalike domains can't hide.
  - The URL shown is the normalized one the server stored (FR-2.14), never the text the user typed. An internationalized host appears in its ASCII `xn--` form, and a URL never contains a username or password (THREAT_MODEL.md T-008).
  - The same plain-text rules apply in the admin lists, in confirmation dialogs, and in tooltips (T-007).
  - Long values are shortened with an ellipsis and are never allowed to overflow the layout.

## Do's and Don'ts

- Do use mat green for one primary action per view.
- Do show short codes and tokens in the mono face, so every character is unambiguous.
- Do use the snip yellow only for things just created.
- Do use admin violet only on admin screens.
- Don't put link rows in cards or add shadows.
- Don't use all-caps labels, arrows on buttons, or decorative motion.
- Don't render titles or URLs as HTML or as live links in the list.
- Don't ask for confirmation in a way that hides what will be deleted.

## Open Questions

None open. DQ-1 (how users are identified) was resolved by FR-7.8.
