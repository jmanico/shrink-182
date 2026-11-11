# Change review (TM_REVIEWED_MODEL, model shrink-182-specs, revision r06, parent r05)

This reviews a specification change. It is not an approval, and it does not show the system is secure.

## Inspection coverage

- **Baseline:** `threat-model/2026-11-11-deep-web/05-report.json` (r05, deep), read in full.
- **Change evidence:** the diff `10b7ddf..e8e2b87`, endpoint to endpoint, of `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `SECURITY_EVENTS.md` and `DESIGN.md`, read in full with surrounding text.
- **Not reviewed as change evidence:** `README.md`, `THREAT_MODEL.md` and the run folder in the same range.
- **Carried, not re-read:** baseline evidence E-001 to E-038.
- **No implementation exists.** Controls stay `designed`; no threat is resolved.

## Changed scenarios

| Threat | Confidence | Result of the change |
|---|---|---|
| T-001 | low | Change reviewed: SC-015 now also covers ceremony routes and runs before rate limiting. Its new sentence exempting 'bearer-token routes' is ambiguous for link routes that accept either credential (G-020). Still a designed control only. |
| T-002 | low | Change reviewed: SC-015 now gates sign-up and login, and SC-011 binds each challenge to the requesting browser and deletes it before verifying. Both premises are addressed in the design. Open until implemented and tested (V-002). |
| T-003 | low | Change reviewed: cross-site requests are refused before they count (SC-015), foreign Host names are refused (SC-026) and the server listens only on loopback (SC-032). A local process on the same machine can still spend the allowance; that is inherent to an address key. Confidence lowered from medium. |
| T-004 | low | Change reviewed: FR-7.9 and SC-027 require random, never-reused IDs and report unmatched admin entries; FR-7.10 stops admins deleting themselves. The premise now fails by design. Confidence lowered from medium. Open until implemented (V-004). |
| T-005 | low | Change reviewed: FR-5.9, SC-028 and the route map limit tokens to link routes and refuse mixed credentials. Confidence lowered from medium. Open until implemented (V-005). |
| T-008 | low | Change reviewed: FR-2.12 to FR-2.14 and SC-003 refuse embedded credentials and control characters and store the serialized URL. Confidence lowered from medium. Open until implemented (V-008). |
| T-009 | low | Change reviewed: NFR-1.4 and SC-032 keep the system on localhost, so outsiders cannot reach it. The scenario returns only if that rule is lifted. The answer is a proposed default (DEC-002). |
| T-010 | low | Change reviewed: SC-003 now compares normalized host names and treats loopback names as the site. Confidence lowered from medium. Open until implemented (V-010). |
| T-011 | low | Change reviewed: unknown-token calls are keyed by address (SC-018) and denial events have a ceiling (SC-025). The flush path is addressed in the design, but the ceiling itself opens T-018. Confidence lowered from medium. |
| T-012 | medium | Change reviewed: SC-034 limits each redirect to one statement and NFR-1.5 records the flood risk as accepted for the demo. That acceptance is a proposed default, not a caller decision (DEC-004), and it does not remove the scenario. |
| T-013 | low | Change reviewed: SC-029 requires a cryptographic source without bias. Open until implemented (V-013). |
| T-014 | low | Change reviewed: FR-1.12 and SC-030 require one transaction with full rollback. Open until implemented (V-014). |
| T-015 | low | Change reviewed: FR-1.13 to FR-1.15 and SC-033 add passkey listing and removal and require a passkey assertion from the last 5 minutes for sensitive actions. Residual: someone who takes over a session within 5 minutes of the owner's own assertion can still add a passkey, and can then remove the owner's. Confidence lowered from medium. |
| T-016 | low | Change reviewed: SC-031 requires no-store on every API response. Open until implemented (V-016). |
| T-017 | low | Change reviewed: SC-032 refuses any origin but localhost, so the weakened-control condition cannot arise without a deliberate rule change. Proposed default (DEC-002). |

## New scenario

- **T-018 (api, medium):** The new volume ceiling writes at most 60 denied or failure records per minute for each event name and only counts the rest. Someone first triggers 60 cheap denials of one event name (for example authorization denials or rejected input), then performs the probing they want hidden in the same minute. Those later records are counted but not written, so the operator cannot see who did what.
  - **Proposed treatment (TR-018):** Suppression applies only to records with no identified actor. A denial or failure by a signed-in user or a valid token is always written, so an identified caller cannot hide their own actions, and anonymous noise cannot hide them either.
  - **Verification:** V-018, not run.

## Carried findings

T-006 and T-007 are unchanged. Treatments TR-001 to TR-017 stay proposed and V-001 to V-017 stay not run.

## Conflicts

- **CF-001** resolved in the text: FR-7.10 and SC-016 now say an admin cannot delete their own account by any route, and FR-1.11 names that exception. The answer is a proposed default awaiting the owner.
- **CF-002** resolved in the text: SC-015 now excepts the click count on following a short link from its rule that no GET changes state.

## Gaps

- **Closed by the new text:** G-002, G-003, G-004, G-005, G-008, G-009, G-012, G-013, G-014, G-015, G-016, G-017.
- **Still open:** G-001, G-006, G-007 (narrowed to session-authenticated API calls), G-010, G-011, G-018, G-019.
- **New:** G-020 (SC-015 says bearer-token routes are exempt from the origin check, but the link routes accept a cookie or a token. Read literally, cookie-authenticated link requests could be exempt.) and G-021 (The specs do not say whether the re-verification ceremony counts against the login rate limit, or how the 5-minute window behaves when a session is used from two browsers.)

## Decisions

The caller supplied no decision. DEC-002 to DEC-005 stay `proposed` with no owner or date; the specs now carry proposed-default answers to OD-008 to OD-011.

## Checks

The JSON parses, has no duplicate keys, has unique IDs and no dangling references, and its changes were computed against r05 (no record was dropped). No schema validator exists, so contract validation did not run. No test ran.
