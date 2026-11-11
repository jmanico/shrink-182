# 03 Challenge and consolidation (TM_REVIEWED_MODEL, revision r03)

## Inspection coverage

- **Read in full in this run (revision `10b7ddf`):** `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `SECURITY_EVENTS.md`, `AGENTS.md`, `README.md`.
- **Partly read:** `DESIGN.md` (prose lines 181-330 in full; the design-token front matter as key names only) and `design/preview.html` (a search for external references only).
- **Not read:** `REQUIREMENT_TEMPLATE.md` and `logo.svg` (not system evidence); the external sources S1-S6 that `SECURITY.md` cites (no network use was authorized).
- **No implementation exists.** Every finding is a design hypothesis about the specifications, and every control is `designed`, not implemented or tested.
- **Unmapped boundaries:** none beyond TB-1 to TB-7. The route namespace inside TB-1 is an open topology gap (G-002).
- **Working tree:** not checked with Git, because globally configured clean/process filters trigger the runbook's skip rule.

## Outcome

- All 17 threats stay open. None was merged, retired or resolved. No confidence level changed; seven rationales were revised (T-001, T-002, T-003, T-004, T-012, T-014, T-015).
- T-012 was narrowed: click-count inflation is accepted behavior under FR-3.7; the availability effect remains.
- Two specification conflicts were recorded and left unresolved:
  - **CF-001:** FR-1.11 lets any logged-in user delete their own account; SC-016, FR-7.5 and SEC-049 say an admin cannot delete themself.
  - **CF-002:** SC-015 says no GET route may change state; FR-3.4 and FR-3.7 require the redirect GET to count a click.
- New gaps: **G-018** (cleanup and caps without background jobs) and **G-019** (no control could be challenged by observation).
- No method ran, so method coverage is not applicable. No identities were imported or merged.

## Counter-evidence per threat

| Threat | Examined on | Result |
|---|---|---|
| T-001 | TB-2 | Challenged: SC-015 would be defeated by a route that changes state on GET, by a body parser that accepts a non-JSON content type, or by the check running after the handler. Its wording rules out the first two for cookie-authenticated routes; nothing observed rules out the third (G-019). SameSite=Lax is a second barrier for cross-site POSTs. Stays open at low confidence until tested. |
| T-002 | EP-ceremony | Challenged: a preflight requirement would stop a cross-site JSON POST only if the ceremony routes refuse other content types, which SC-015 does not require of them. A single-use challenge does not by itself tie the ceremony to one browser. The premise stands; the practical path is narrow. Stays open at low confidence. |
| T-003 | EP-ceremony | Challenged: looked for a specified protection that keeps another origin's or another local process's requests from spending the allowance. None exists: the limiter key is the socket address and no origin check covers ceremony routes. Simple cross-site POSTs are sent by browsers without a preflight. Confidence stays medium for the localhost demo. |
| T-004 | D-user-id | Challenged: the scenario fails if IDs are unguessable and never reused, or if admins never delete their accounts. Neither is stated; CF-001 shows the specs disagree on admin self-deletion, and FR-7.5 lets one admin delete another. Confidence stays medium until an ID rule exists. |
| T-005 | EP-api-account | Sought a statement limiting bearer tokens to link routes; found only indirect support (FR-5.2, data flows 5 and 6, SC-014). Controls examined: CTL-SC-014. |
| T-006 | TB-3 | Compared single-object routes with the admin list-all path and with token access; SC-016 covers every read, edit and delete in one module. Controls examined: CTL-SC-016, CTL-SC-021, CTL-SC-002. |
| T-007 | TB-5 | Checked the admin list, dialogs and tooltip contexts against SC-009 and SC-010; both apply to every view. Uploads and previews do not exist. Controls examined: CTL-SC-009, CTL-SC-010. |
| T-008 | AS-links | Sought any rule on embedded credentials, host display form or title characters; none found. Controls examined: CTL-SC-003, CTL-SC-009. |
| T-009 | TB-7 | Counter-evidence is the localhost-only deployment and admin deletion. The premise holds only if hosted. |
| T-010 | AS-links | Sought a normalization rule for the own-host comparison; none found. Controls examined: CTL-SC-003. |
| T-011 | AS-event-log | Sought a limit that applies when the token is unknown or missing, and any bound on event volume; none found. Controls examined: CTL-SC-018, CTL-SC-025. |
| T-012 | EP-redirect | Challenged and narrowed: click-count inflation is behavior the owner accepted in FR-3.7, so it is not treated as a defect. The availability effect remains: unlimited anonymous requests each cause a synchronous write in a single process. CF-002 notes that SC-015's 'no GET changes state' wording does not hold for this route. |
| T-013 | EP-redirect | Counter-evidence is the size of the code space. The remaining premise is the random source (G-015). Controls examined: CTL-SC-020. |
| T-014 | AS-acct-del | Challenged: a single synchronous process makes interleaving unlikely within one transaction, and SEC-052's rolled_back outcome implies one. The specs still do not require it, and DR-5 splits the work across two modules. Stays open at low confidence. |
| T-015 | D-passkey | Challenged: registration needs a user gesture and, under SC-011, user verification on the new authenticator, but that verifies the intruder's authenticator, not the account owner. The event file records the addition but only the operator reads it. Nothing lets the owner detect or undo it. Confidence stays medium. |
| T-016 | Z-browser | No shared cache, other tenant or account switch exists; the only recipient mismatch is a later user of the same browser profile. Controls examined: CTL-SC-020, CTL-SC-014. |
| T-017 | AS-config | The specs disclose each localhost assumption; no rule forces a review before hosting. Controls examined: CTL-SC-013, CTL-SC-010. |
