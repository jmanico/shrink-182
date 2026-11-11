# shrink-182 Threat Model

This is a design-level threat model of the shrink-182 specifications. It is not a security assessment of running software, and it does not show that the system is secure, compliant or ready for production. Treat it as private review material.

- **Method:** Grim Prompter deep mode, six stages (00 to 05), `web` as the primary architecture and `api` as a bounded secondary, combined under its composition guide. Library revision `d7c9082`.
- **Target:** specifications only, at revision `10b7ddf`. Review scope: security.
- **Model:** `shrink-182-specs`, revision `r05`. The JSON baseline for change reviews is `threat-model/2026-11-11-deep-web/05-report.json`.
- **Status:** all 17 threats are open design hypotheses. All treatments are proposed, all verification is `not_run`, all risks are unrated.

## Inspection coverage

- **Read in full in this run (revision `10b7ddf`):** `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `SECURITY_EVENTS.md`, `AGENTS.md`, `README.md`.
- **Partly read:** `DESIGN.md` (prose lines 181-330 in full; the design-token front matter as key names only) and `design/preview.html` (a search for external references only).
- **Not read:** `REQUIREMENT_TEMPLATE.md` and `logo.svg` (not system evidence); the external sources S1-S6 that `SECURITY.md` cites (no network use was authorized).
- **No implementation exists.** Every finding is a design hypothesis about the specifications, and every control is `designed`, not implemented or tested.
- **Unmapped boundaries:** none beyond TB-1 to TB-7. The route namespace inside TB-1 is an open topology gap (G-002).
- **Working tree:** not checked with Git, because globally configured clean/process filters trigger the runbook's skip rule.

## Scope

**In scope:** React SPA and browser session; Fastify HTTP layer and JSON API (session cookie and bearer token); Auth, Links, Redirect, Admin and Account deletion modules; Configuration, SQLite store, security event file; Anonymous short-link redirect.

**Out of scope:** Application code (none exists); Host OS, GitHub repository settings and CI; A hosted deployment (only a localhost demo is specified); Privacy review scope (not requested); Supply-chain architecture analysis beyond what the web and API lenses touch; REQUIREMENT_TEMPLATE.md and logo.svg (not system evidence).

**Not applicable surfaces (absent from the specs):** file uploads and previews (FR-6.2), payments (FR-6.1), shared caches and reverse proxies, third-party scripts and external resources, background jobs and exports, callbacks and outbound requests, identity federation, multi-tenancy.

## System model

The model has 6 actors, 11 components and stores, 8 data types, 6 entry points, 12 flows and 7 trust boundaries. All come from specification text (basis `proposed_design`). Full records are in the JSON.

| Boundary | Owning architecture | Description |
|---|---|---|
| TB-1 | web | Internet to server: all requests are untrusted input; validated in the HTTP layer and again in the Links module. |
| TB-2 | web | Anonymous to authenticated: authentication is enforced once in the Auth module; session cookies for the web UI, bearer tokens for the JSON API. |
| TB-3 | api | User to another user's data: ownership is enforced in the Links module on every read, edit and delete. |
| TB-4 | api | User to admin: the role is enforced at the Admin module entry using the configured admin list. |
| TB-5 | web | Server to browser rendering: user titles and URLs are output as plain text by React. |
| TB-6 | api | Server to SQLite file: trusted local store, bound parameters only. |
| TB-7 | web | Redirect to external target: visitors are sent to user-supplied http/https URLs that are not the site's own domain. |

| Entry point | Description |
|---|---|
| EP-redirect | Anonymous GET /<code> short-link redirect. |
| EP-static | Static files and the SPA shell, including the signed-out page. |
| EP-ceremony | Unauthenticated sign-up and login routes (options and verify). |
| EP-api-links | JSON API for creating, listing, editing and deleting links; session cookie or bearer token. |
| EP-api-account | Account routes: logout, add passkey, create/list/revoke tokens, delete account. |
| EP-api-admin | Admin routes: list all links and users, delete any link, delete another user. |

## Threats

Confidence describes how well the specs support the scenario. It is not severity. No risk is rated, because no rubric was supplied.

| ID | Threat | Arch. | Confidence | Designed controls that bear on it | Open gaps |
|---|---|---|---|---|---|
| T-001 | Cross-site request changes state using the session cookie | web | low | SC-013, SC-015 | G-001, G-019 |
| T-002 | Sign-up and login routes fall outside the CSRF rule; challenge not tied to one browser | web | low | SC-011, SC-012 | G-003 |
| T-003 | Another page or local process spends the login rate limit and locks the user out | web, api | medium | SC-018 | G-012 |
| T-004 | A reused or predictable user ID inherits admin authority from configuration | api, web | medium | SC-016, SC-022 | G-004 |
| T-005 | An API token reaches account, token or admin routes; mixed credentials | api, web | medium | SC-014 | G-005 |
| T-006 | A changed short code reaches another user's link; non-admin reaches admin routes | api, web | low | SC-002, SC-016, SC-021 | G-001 |
| T-007 | A stored title or URL is interpreted in the admin's view | web | low | SC-009, SC-010 | G-001 |
| T-008 | A target or title is displayed in a way that hides the real destination | web | medium | SC-003, SC-009 | G-017 |
| T-009 | Open sign-up plus any target turns the service into a phishing redirector (hosted only) | web, api | low | SC-003 | G-009 |
| T-010 | A target names the service by another host spelling and bypasses the own-domain rule | api, web | medium | SC-003 | G-017 |
| T-011 | Random tokens bypass the API rate limit and flush the security event file | api | medium | SC-018, SC-025 | G-007, G-016 |
| T-012 | A redirect flood, one synchronous write per request, starves the single process | web | medium | SC-001, SC-018, SC-020 | - |
| T-013 | Guessing short codes discloses target URLs | web | low | SC-020 | G-015 |
| T-014 | A partly failed account deletion leaves working links, tokens or sessions | api | low | none | G-008, G-018 |
| T-015 | Brief use of a session adds a passkey the owner cannot see or remove | web | medium | SC-013 | G-013 |
| T-016 | Authenticated responses, including the one-time token, stay in the browser cache after sign-out | web | low | SC-014, SC-020 | G-014 |
| T-017 | Localhost-only assumptions weaken controls if the system is ever hosted | web, api | low | SC-010, SC-013 | G-009, G-011, G-012 |

### T-001 — Cross-site request changes state using the session cookie

- **Scenario:** A page from another origin makes the signed-in user's browser send a state-changing request (create, edit or delete a link, revoke a token, delete the account, an admin delete) that the server accepts on the strength of the session cookie alone.
- **At stake:** Integrity of the user's links, tokens and account; for an admin, of every user's links and accounts.
- **Needs:** The victim has a valid session; The victim opens an attacker-controlled page in the same browser; The fetch-metadata/Origin and content-type check is missing or bypassable on the targeted route.
- **Evidence and counter-evidence:** Challenged: SC-015 would be defeated by a route that changes state on GET, by a body parser that accepts a non-JSON content type, or by the check running after the handler. Its wording rules out the first two for cookie-authenticated routes; nothing observed rules out the third (G-019). SameSite=Lax is a second barrier for cross-site POSTs. Stays open at low confidence until tested.
- **Evidence records:** E-021, E-023. **Affected:** ACT-other-origin, F-11, EP-api-links, EP-api-account, EP-api-admin, TB-2, D-session.
- **Proposed treatment (TR-001):** Every state-changing request made with the session cookie is accepted only when it demonstrably comes from the application's own origin and is a JSON request. Keep SC-015 and SC-013 as specified and prove them on every non-GET route.
- **Verification:** V-001, not run.

### T-002 — Sign-up and login routes fall outside the CSRF rule; challenge not tied to one browser

- **Scenario:** Sign-up and login routes are not authenticated by the session cookie, so the wording of SC-015 does not cover them. A cross-site page submits a login completion for the attacker's own account, or a second party completes a ceremony against a challenge issued to someone else. The victim ends up signed in as the attacker and saves links (target URLs) into the attacker's account.
- **At stake:** Confidentiality of URLs the victim shortens; integrity of which account a browser is signed in to.
- **Needs:** The ceremony routes accept a cross-site request; A challenge is not tied to the browser that requested it, or can be used twice under concurrent requests; The attacker has their own registered passkey.
- **Evidence and counter-evidence:** Challenged: a preflight requirement would stop a cross-site JSON POST only if the ceremony routes refuse other content types, which SC-015 does not require of them. A single-use challenge does not by itself tie the ceremony to one browser. The premise stands; the practical path is narrow. Stays open at low confidence.
- **Evidence records:** E-020, E-023. **Affected:** ACT-other-origin, EP-ceremony, F-1, D-challenge, TB-2.
- **Proposed treatment (TR-002):** Ceremony routes get the same origin and content-type gate as authenticated routes, and a challenge can be completed only by the browser that requested it, exactly once.
- **Verification:** V-002, not run.

### T-003 — Another page or local process spends the login rate limit and locks the user out

- **Scenario:** Sign-up and login are limited to 10 attempts per minute per socket address. On the localhost demo every browser tab and local process shares the loopback address, and the ceremony routes are reachable cross-site. Any page the user visits, or any local process, can spend the allowance and keep the user locked out of login.
- **At stake:** Availability of sign-up and login.
- **Needs:** Ceremony option or verify requests count against the limit before any origin check; The attacker's requests arrive from the same socket address as the user's.
- **Evidence and counter-evidence:** Challenged: looked for a specified protection that keeps another origin's or another local process's requests from spending the allowance. None exists: the limiter key is the socket address and no origin check covers ceremony routes. Simple cross-site POSTs are sent by browsers without a preflight. Confidence stays medium for the localhost demo.
- **Evidence records:** E-008, E-023, E-026. **Affected:** ACT-other-origin, EP-ceremony, AS-http, F-11.
- **Proposed treatment (TR-003):** Requests from other origins, or addressed to another host name, are refused before they count against the sign-up and login allowance, so another page cannot spend the user's attempts.
- **Verification:** V-003, not run.

### T-004 — A reused or predictable user ID inherits admin authority from configuration

- **Scenario:** Admin authority is granted to whoever holds an internal user ID listed in configuration. The specs do not say how IDs are generated or that they are never reused. If IDs are sequential or reusable, a new sign-up can receive an ID that is still in the admin list after the original admin account was deleted, and becomes an admin. Reuse would also mis-attribute retained security events.
- **At stake:** Admin authority (TB-4): every user's links and accounts; attribution in the event file.
- **Needs:** An ID in the admin list no longer belongs to the intended person (the account was deleted, or the ID was listed in advance); A new account can obtain that ID.
- **Evidence and counter-evidence:** Challenged: the scenario fails if IDs are unguessable and never reused, or if admins never delete their accounts. Neither is stated; CF-001 shows the specs disagree on admin self-deletion, and FR-7.5 lets one admin delete another. Confidence stays medium until an ID rule exists.
- **Evidence records:** E-007, E-024, E-029. **Affected:** D-user-id, D-admin-list, AS-config, AS-admin, TB-4, DEP-4.
- **Proposed treatment (TR-004):** An internal user ID is unguessable, is never issued twice, and an admin-list entry that matches no current user is reported at startup, so admin authority cannot pass to a later sign-up.
- **Verification:** V-004, not run.

### T-005 — An API token reaches account, token or admin routes; mixed credentials

- **Scenario:** One JSON API accepts a session cookie or a bearer token, but the specs do not list which routes accept which. A leaked or over-shared API token could reach account, passkey, token-management or admin routes, and a request carrying both credentials could be checked under one and acted on under the other.
- **At stake:** Limit of a token's authority to link management; integrity of account and admin functions.
- **Needs:** A token is obtained by someone other than its owner, or a request carries both a cookie and a token; A non-link route accepts the bearer credential, or the CSRF check is skipped when an Authorization header is present.
- **Evidence and counter-evidence:** Supported by ARCHITECTURE.md describing one API for both credentials and by G-005. Against it: FR-5.2 lists only link operations for the API, data flows 5 and 6 name the session for tokens and admin, and SC-014 forbids creating tokens with a token. A per-route credential rule would decide it.
- **Evidence records:** E-005, E-009, E-012, E-022. **Affected:** D-token, EP-api-account, EP-api-admin, EP-api-links, AS-auth, TB-2, TB-4, F-7.
- **Proposed treatment (TR-005):** Each route accepts exactly one kind of credential. Bearer tokens work only on the link operations of FR-5.2; account, passkey, token-management and admin routes accept only a session; a request carrying both is refused.
- **Verification:** V-005, not run.

### T-006 — A changed short code reaches another user's link; non-admin reaches admin routes

- **Scenario:** A signed-in user or API client changes the short code in a view, edit or delete call to reach a link owned by someone else, or calls an admin route without the admin role. The admin list-all route is the one bulk path; it returns every user's links and IDs.
- **At stake:** Confidentiality and integrity of other users' links (TB-3); admin-only functions (TB-4).
- **Needs:** The ownership filter or role check is missing on one route or one operation; Or a response returns fields beyond the allowed set.
- **Evidence and counter-evidence:** SC-016 places the ownership check in the Links module for every read, edit and delete with a uniform not-found, and SC-021 limits response fields. Nothing is built. A two-account comparison across every operation, by cookie and by token, would decide it.
- **Evidence records:** E-004, E-007, E-024, E-028. **Affected:** EP-api-links, EP-api-admin, AS-links, AS-admin, TB-3, TB-4, D-link, F-4, F-6.
- **Proposed treatment (TR-006):** Every link operation binds the caller, the link's owner and the action in the Links module, and every admin operation checks the configured role, whichever credential is used. Keep SC-016 and SC-021 and prove them per operation.
- **Verification:** V-006, not run.

### T-007 — A stored title or URL is interpreted in the admin's view

- **Scenario:** A user stores a title or target URL crafted to be interpreted as markup or a live link. It is later shown in another user's context: the admin's list of all links, a delete-confirmation dialog that names the link and owner, or a tooltip. If interpreted, it runs with the admin's session.
- **At stake:** Integrity of the admin's session and actions; TB-5.
- **Needs:** User content reaches a sink that interprets it (HTML, a URL attribute) instead of text; The CSP does not stop the result.
- **Evidence and counter-evidence:** SC-009 (text only, no target in href) and SC-010 (CSP limited to 'self') are both specified and each would defeat it. Nothing is built. Rendering hostile titles and URLs in the admin list, the dialogs and the tooltip would decide it.
- **Evidence records:** E-019, E-032, E-033. **Affected:** D-link, F-12, AS-spa, TB-5, ACT-admin, EP-api-admin.
- **Proposed treatment (TR-007):** User-supplied titles and URLs stay inert text in every view, including the admin list, confirmation dialogs and tooltips. Keep SC-009 and SC-010 and prove them in the privileged views.
- **Verification:** V-007, not run.

### T-008 — A target or title is displayed in a way that hides the real destination

- **Scenario:** A target URL is accepted and displayed in a form that misleads the reader about where it goes: a trusted-looking name before an @ sign, an internationalized look-alike host shown in Unicode, or a title containing direction-override or control characters that reorders what is displayed. DESIGN.md relies on the visible host to expose look-alike domains.
- **At stake:** Users' and admins' ability to judge a link's destination from the list and from confirmation dialogs.
- **Needs:** SC-003 accepts URLs with embedded credentials, or the display shows the submitted string instead of the parsed host; Or titles may contain control or direction-override characters.
- **Evidence and counter-evidence:** Supported by SC-003 listing only scheme, length and own-host checks, and by G-017. No specified control addresses it. A rule on normalization and allowed characters would decide it.
- **Evidence records:** E-002, E-017, E-032, E-033. **Affected:** D-link, AS-links, AS-spa, F-12, TB-5, TB-7.
- **Proposed treatment (TR-008):** What is stored and shown for a target is the parsed, normalized URL, so the visible host is the real destination. Targets with embedded credentials are refused, and titles cannot carry control or direction-override characters.
- **Verification:** V-008, not run.

### T-009 — Open sign-up plus any target turns the service into a phishing redirector (hosted only)

- **Scenario:** Sign-up is open and uncapped, and any http/https target is allowed. Anyone who can reach the site can create an account and use shrink-182 links as a trusted-looking front for phishing or malware destinations. Removal depends on an admin noticing.
- **At stake:** Trust in links served from the team's short-link host; safety of visitors.
- **Needs:** The site is reachable by people outside the team; No admin has removed the link or account.
- **Evidence and counter-evidence:** FR-1.5, FR-2.8 and NFR-1.3 make this possible by design. Against it: the specified deployment is a localhost demo, so outsiders cannot reach it, and admins can delete links and users (FR-7.4, FR-7.5). It becomes live only if the system is hosted (G-009).
- **Evidence records:** E-001, E-002, E-003, E-007, E-008. **Affected:** ACT-visitor, EP-ceremony, EP-redirect, AS-links, TB-7, D-link, F-3.
- **Proposed treatment (TR-009):** Before the system is reachable by anyone outside the team, the owner decides how sign-up and link targets are constrained (for example invitations, a warning page before leaving, or an allow-list). Until then the system stays on localhost.
- **Verification:** V-009, not run.

### T-010 — A target names the service by another host spelling and bypasses the own-domain rule

- **Scenario:** FR-2.10 rejects targets on the site's own domain, and SC-003 compares the target host with the configured site host. A target that names the same server another way (a different spelling of the loopback host, a trailing dot, another port on the same host) passes the comparison and lets links point back at the service, including at each other in a loop.
- **At stake:** The FR-2.10 rule; availability for a visitor caught in a redirect loop; inflated click counts.
- **Needs:** The own-host comparison is an exact string match against one configured name.
- **Evidence and counter-evidence:** Supported by the wording of SC-003 ('a host different from the configured site host'). Browsers stop redirect loops after a small number of hops, which limits harm. A rule on host normalization and loopback names would decide it.
- **Evidence records:** E-002, E-017. **Affected:** AS-links, EP-redirect, TB-7, D-link.
- **Proposed treatment (TR-010):** The own-domain rule compares normalized host names and treats every name for the local machine as the site itself while the site runs on localhost, so a link cannot point back at the service.
- **Verification:** V-010, not run.

### T-011 — Random tokens bypass the API rate limit and flush the security event file

- **Scenario:** API rate limits are keyed by the hash of the presented token. A caller who sends a different random token on each request never reaches a limit. Each rejected request also writes an auth.api_token_rejected event, so the caller can fill the 10 MiB x 5 event file and rotate audit-required events out before the operator reads them.
- **At stake:** Availability of the API; retention of audit evidence in the event file.
- **Needs:** Requests with missing or unknown tokens are limited per token hash, or not at all; Rejection events are written without a volume bound.
- **Evidence and counter-evidence:** Supported by SC-018 (key is the token hash 'already computed for lookup'), SEC-026 (an event per rejection) and the rotation policy. G-007 and G-016 are open. A rule for unauthenticated API failures and for event volume would decide it.
- **Evidence records:** E-008, E-026, E-029, E-031. **Affected:** EP-api-links, AS-http, AS-event-log, D-events, F-7, F-9.
- **Proposed treatment (TR-011):** API calls that present no valid token share the address-keyed allowance used for login, and each kind of denial event has a per-minute ceiling with a count of suppressed events, so an anonymous caller can neither bypass the limit nor flush audit evidence quickly.
- **Verification:** V-011, not run.

### T-012 — A redirect flood, one synchronous write per request, starves the single process

- **Scenario:** Every anonymous request for an existing short link performs a synchronous database write to count the click, and redirects are deliberately not rate-limited. A flood of redirect requests can occupy the single process and also inflate any link's click count.
- **At stake:** Availability of the whole service (one event loop); integrity of click counts.
- **Needs:** The attacker can send requests at a rate the process cannot absorb.
- **Evidence and counter-evidence:** Challenged and narrowed: click-count inflation is behavior the owner accepted in FR-3.7, so it is not treated as a defect. The availability effect remains: unlimited anonymous requests each cause a synchronous write in a single process. CF-002 notes that SC-015's 'no GET changes state' wording does not hold for this route.
- **Evidence records:** E-003, E-008, E-017, E-026. **Affected:** EP-redirect, AS-redirect, AS-db, F-3, D-link.
- **Proposed treatment (TR-012):** The owner decides whether unlimited anonymous redirects, each with a synchronous write, are acceptable for the demo or whether the write cost per request must be bounded. Click-count inflation stays accepted under FR-3.7.
- **Verification:** V-012, not run.

### T-013 — Guessing short codes discloses target URLs

- **Scenario:** Short codes are the only thing protecting a link's target from anonymous readers. Someone guesses codes against the unlimited redirect route to harvest target URLs, which may include private document links.
- **At stake:** Confidentiality of target URLs.
- **Needs:** Codes are predictable or drawn with bias, or the attacker can afford a very large number of requests.
- **Evidence and counter-evidence:** FR-2.7 gives 62^7 possible codes for a few thousand links, so blind guessing is impractical if codes are drawn from a cryptographic source. No SC rule states that source (G-015). The generation rule decides it.
- **Evidence records:** E-002, E-003, E-031. **Affected:** EP-redirect, D-link, AS-links, F-3.
- **Proposed treatment (TR-013):** Short codes are drawn from a cryptographic random source with every character equally likely, so codes cannot be predicted from earlier ones.
- **Verification:** V-013, not run.

### T-014 — A partly failed account deletion leaves working links, tokens or sessions

- **Scenario:** Account deletion spans the Auth module (user, passkeys, sessions, tokens) and the Links module (links, retired codes). If it is not atomic, a failure midway leaves live links without an owner or tokens and sessions that still work for a deleted user, and a request already in flight may act for an account that is being deleted.
- **At stake:** FR-1.11 and FR-5.8: a deleted account leaves no working link, token or session.
- **Needs:** Deletion is done as separate steps without one transaction; A step fails or a concurrent request interleaves.
- **Evidence and counter-evidence:** Challenged: a single synchronous process makes interleaving unlikely within one transaction, and SEC-052's rolled_back outcome implies one. The specs still do not require it, and DR-5 splits the work across two modules. Stays open at low confidence.
- **Evidence records:** E-001, E-005, E-011, E-014, E-031. **Affected:** AS-acct-del, AS-auth, AS-links, AS-db, F-8, D-token, D-session, D-link.
- **Proposed treatment (TR-014):** Deleting an account removes or revokes the user, passkeys, sessions, tokens and links as one all-or-nothing change, and a request that started before the deletion cannot act for the account after it.
- **Verification:** V-014, not run.

### T-015 — Brief use of a session adds a passkey the owner cannot see or remove

- **Scenario:** Someone who briefly controls a signed-in browser (an unattended laptop, a stolen session) adds their own passkey. The session alone authorizes it. The owner cannot see or remove passkeys, so the intruder keeps access after the session ends; the only remedy is deleting the account. The same session can create an API token or delete the account.
- **At stake:** Long-term control of the account.
- **Needs:** The attacker can act within a valid session for a short time.
- **Evidence and counter-evidence:** Challenged: registration needs a user gesture and, under SC-011, user verification on the new authenticator, but that verifies the intruder's authenticator, not the account owner. The event file records the addition but only the operator reads it. Nothing lets the owner detect or undo it. Confidence stays medium.
- **Evidence records:** E-001, E-021, E-031. **Affected:** EP-api-account, D-passkey, D-session, AS-auth, ST-signed-in.
- **Proposed treatment (TR-015):** The owner decides how an account holder can detect and undo an unwanted passkey, and whether adding a passkey, creating a token or deleting the account needs a fresh passkey check.
- **Verification:** V-015, not run.

### T-016 — Authenticated responses, including the one-time token, stay in the browser cache after sign-out

- **Scenario:** Authenticated JSON responses (the link list, the admin lists, the one-time token value) carry no cache directive. On a shared machine they may remain in the browser's cache or history after logout or session expiry.
- **At stake:** Confidentiality of link data and of a freshly created API token after sign-out.
- **Needs:** Another person uses the same browser profile after the user signs out; The browser stored the response.
- **Evidence and counter-evidence:** Supported by Cache-Control: no-store being specified only for redirects (G-014). No shared cache or proxy exists, so exposure is limited to the local browser. A header check on API responses would decide it.
- **Evidence records:** E-022, E-027. **Affected:** EP-api-account, EP-api-links, EP-api-admin, D-token, D-link, ST-session-ended, Z-browser.
- **Proposed treatment (TR-016):** Authenticated responses, above all the one-time token, are never stored by the browser, so nothing remains readable after sign-out.
- **Verification:** V-016, not run.

### T-017 — Localhost-only assumptions weaken controls if the system is ever hosted

- **Scenario:** Several controls are justified only by the localhost demo: no TLS or HSTS, a Secure cookie that works because localhost is treated as secure, trustProxy off, rate limits keyed by socket address, no Host check. If the same build is put on a network or behind a proxy without revisiting them, the controls weaken silently: all clients may share one rate-limit key, cookies travel without TLS guarantees, and open sign-up meets real outsiders.
- **At stake:** Effectiveness of the session, rate-limit and redirect-abuse controls outside the demo.
- **Needs:** The system is deployed somewhere other than a developer's localhost; The localhost-only decisions are not revisited.
- **Evidence and counter-evidence:** The specs state these assumptions openly (SC-010, SC-013, SC-018, SEC-058). A hosted deployment is out of scope (G-009), so this is a condition to guard, not a present exposure.
- **Evidence records:** E-009, E-016, E-019, E-021, E-026. **Affected:** AS-http, AS-config, EP-ceremony, D-session, TB-1.
- **Proposed treatment (TR-017):** The localhost-only decisions are named in one place and block any other deployment until the owner has decided transport security, proxy trust, the rate-limit key and exposure of open sign-up.
- **Verification:** V-017, not run.

## Protective evidence

The specs already design controls that would defeat several scenarios if built as written. All have status `designed`; none is implemented or tested.

| Control | Bears on | Planned verification |
|---|---|---|
| SC-001: request size and time limits. | T-012 | V-012 |
| SC-002: JSON schema on every route, with the short-code pattern. | T-006 | V-006 |
| SC-003: target URL rules in the Links module: WHATWG parsing, http/https only, length limit, not the site's own host. | T-008, T-009, T-010 | V-008, V-009, V-010 |
| SC-009: titles, URLs and tokens rendered only as JSX text; no target URL in href. | T-007, T-008 | V-007, V-008 |
| SC-010: CSP limited to 'self', nosniff and no-referrer on every response. | T-007, T-017 | V-007 |
| SC-011: passkey ceremony verification with a single-use 5-minute challenge, configured origin and RP ID, required user verification. | T-002 | V-002 |
| SC-012: credential lookup by ID, counter check, excludeCredentials. | T-002 | - |
| SC-013: __Host-session cookie (HttpOnly, Secure, SameSite=Lax), hashed storage, new ID at login, deletion at logout, idle and absolute expiry. | T-001, T-015, T-017 | V-001, V-015, V-017 |
| SC-014: API tokens: random, shown once, hashed, bearer header only, rejected when revoked or owner deleted, not creatable with a token. | T-005, T-016 | V-005, V-014 |
| SC-015: Fetch-metadata/Origin and JSON content-type check on non-GET routes authenticated by the session cookie. | T-001 | V-001, V-002 |
| SC-016: ownership filter with uniform not-found in the Links module; admin-list check on every admin action. | T-004, T-006 | V-004, V-006, V-014 |
| SC-018: in-memory rate limits: sign-up/login by socket address, API by token hash; redirects unlimited. | T-003, T-011, T-012 | V-003, V-011, V-012 |
| SC-019: logs without client address, user agent, cookies, authorization or bodies. | - | - |
| SC-020: redirect returns 302 only to a stored target, with Cache-Control: no-store. | T-012, T-013, T-016 | V-013, V-016 |
| SC-021: response schemas listing only allowed fields. | T-006 | V-006 |
| SC-022: origin, admin list and database path from validated environment variables; fail closed. | T-004 | V-004, V-017 |
| SC-024: database file outside the static root with mode 0600; no inspector. | - | - |
| SC-025: security events only through one allowlisting helper that never throws. | T-011 | V-011 |

## Conflicts

Both are unresolved. Neither was settled by preferring one document.

- **CF-001 — Whether an admin can delete their own account.**
  - FR-1.11 lets any logged-in user delete their own account, with no exception for admins. (E-001)
  - SC-016's verification says an admin cannot delete themself, and FR-7.5 lets an admin delete any account other than their own; SEC-049 defines an admin_self_delete denial. (E-007, E-024, E-031)
- **CF-002 — Whether a GET request may change state.**
  - SC-015 says no GET route may change state. (E-023)
  - FR-3.4 and FR-3.7 require every GET of a short link to increase its click count. (E-003)

## Gaps

A gap is missing evidence or an unspecified detail. It is not a vulnerability.

| ID | Kind | Gap | What would settle it |
|---|---|---|---|
| G-001 | evidence | No implementation exists, so no specified control can be observed working. | Source code and test results once the system is built. |
| G-002 | topology | The URL namespace is not specified: how short codes, SPA screens, static files and API routes are kept apart. | A route map with the path prefix for each surface. |
| G-003 | evidence | The specs do not say how a WebAuthn challenge is tied to the browser that requested it, or that it is consumed atomically. | A rule for challenge binding and one-time consumption. |
| G-004 | evidence | The format of the internal user ID, how it is generated and whether it can be reused after deletion are not specified. | A rule for user ID generation and reuse. |
| G-005 | evidence | The specs do not list which routes accept a bearer token and which accept only a session, or what happens when a request carries both. | A per-route credential map. |
| G-006 | technology | No exact, resolved secure-coding pack was supplied for the stack, so technology-specific prescriptions stay unverified. | A resolved pack with group, technology, version and resolver provenance. |
| G-007 | evidence | The rate-limit key for API calls with a missing or unknown token, and for API calls made with a session cookie, is not specified. | A rule naming the key for each case. |
| G-008 | evidence | The specs do not say whether account deletion across the Auth and Links modules is one transaction. | A rule or design for the deletion's atomicity. |
| G-009 | evidence | Nothing is specified about a hosted deployment (TLS, proxy, HSTS, real client addresses). Hosted exposure is out of scope. | A deployment design, if the system is ever hosted. |
| G-010 | evidence | The specs do not say whether login uses discoverable credentials, although users have no username. | The registration and login option settings. |
| G-011 | technology | Whether every supported browser accepts a Secure, __Host- cookie and runs WebAuthn at http://localhost is a claim in SC-013, not an observation. | A browser test once the app runs. |
| G-012 | evidence | The specs do not say how the server treats a request whose Host header is not the configured site host. | A rule for Host handling. |
| G-013 | evidence | The specs give no way to list or remove a single passkey and require no fresh passkey check before adding a passkey, creating a token or deleting the account. | An owner decision on passkey management and on re-verification for sensitive account actions. |
| G-014 | evidence | Cache directives are specified only for the redirect. Nothing says how authenticated JSON responses, including the one-time token, are kept out of the browser cache. | A rule for cache directives on API responses. |
| G-015 | evidence | No secure-coding rule says that short codes come from a cryptographic random source with unbiased selection; only the event ledger implies it. | A rule for short-code generation. |
| G-016 | evidence | Nothing bounds how many security events one unauthenticated caller can cause, and audit-required events share one rotating file with high-volume denial events. | A rule for event volume, or a decision to separate audit events. |
| G-017 | evidence | SC-003 does not say what is stored and shown for a target: the submitted string or the parsed, serialized URL; nor whether embedded credentials or control characters are allowed. | A rule for target normalization and for characters allowed in titles. |
| G-018 | evidence | The specs have no background jobs, yet expired sessions and challenges, and rows created by repeated sign-ups or token creation, must be bounded somehow. No rule says when they are removed or capped. | A rule for cleanup of expired rows and for per-user caps. |
| G-019 | coverage | Challenge of the designed controls could only be done against their wording. No bypass or failure case could be ruled out by observation, because nothing is built. | Tests of each control once implemented (see the verification plans). |

## Coverage

No named method ran. The baseline used STRIDE-style questions and the web and API lenses below.

| Lens | Status | Reason |
|---|---|---|
| inspection-coverage | partial | Six documents read in full; DESIGN.md front matter and design/preview.html partly read; REQUIREMENT_TEMPLATE.md and logo.svg skipped; cited external sources and any implementation unavailable. All reads were made fresh in stage 00 of this run at revision 10b7ddf. |
| baseline:impersonation | assessed | Baseline lens applied without a registered method. Passkey ceremonies, sessions and tokens examined: T-002, T-005, T-015. |
| baseline:unauthorized-change | assessed | Baseline lens applied without a registered method. State-changing requests and object ownership examined: T-001, T-006, T-010, T-014. |
| baseline:accountability | assessed | Baseline lens applied without a registered method. Event file and identifier reuse examined: T-004, T-011. |
| baseline:disclosure | assessed | Baseline lens applied without a registered method. Target URLs, tokens and cached responses examined: T-002, T-013, T-016. |
| baseline:availability | assessed | Baseline lens applied without a registered method. Rate limits, redirect writes and event volume examined: T-003, T-011, T-012. |
| baseline:privilege-crossing | assessed | Baseline lens applied without a registered method. User-to-admin and token-to-account crossings examined: T-004, T-005, T-006, T-007. |
| web:state-changing-requests | assessed | Family 1 (unintended state change through session authority): T-001, T-002, T-003. |
| web:identity-object-binding | assessed | Normal, API-token and admin routes compared: T-005, T-006. |
| web:content-in-another-users-view | assessed | Family 2 (user content in the admin's view): T-007, T-008. |
| web:private-response-reuse | assessed | Family 3. No shared cache or proxy is supplied, so recipient mismatch through a shared cache has no subject. Browser-side retention is T-016. |
| web:upload-preview-download | not_applicable | FR-6.2: the system accepts no file uploads. |
| web:third-party-code-and-resources | not_applicable | No third-party script or external resource is specified: fonts are self-hosted and the CSP allows only 'self'. The preview page search (E-037) found no external load. |
| web:signout-expiry-recovery | assessed | Logout, expiry and account deletion examined: T-014, T-015, T-016. No recovery path exists (FR-1.8). |
| web:delegated-async-work | not_applicable | The specs state there are no background jobs. |
| api:caller-identity-and-audience | assessed | Bearer and cookie identities examined: T-005, T-011. No gateway or external issuer is supplied. |
| api:object-identifier-paths | assessed | Family 1 (another owner's object): T-006. The only bulk path is the admin list; no export or deferred retrieval exists. |
| api:object-function-property-authority | assessed | Object (T-006), function (T-005, T-004) and property (SC-021 under T-006) authority considered separately. |
| api:replay-concurrency-order | partial | Family 2. Examined for challenge reuse (T-002) and account deletion (T-014). The supplied invariants are unique codes, never-reused codes and single-use challenges; no commitment-point detail is supplied, so concurrency analysis stays at the design level. |
| api:callback-destinations | not_applicable | Family 3. The server makes no outbound requests and registers no callbacks; the redirect never fetches the target. |
| api:notification-payload-and-response | not_applicable | No notifications or external status feeds are specified. |
| api:bulk-export-and-callback-work | not_applicable | No exports, jobs or retained results are specified. |
| api:federation | not_applicable | No identity federation or OAuth is specified. |
| method-coverage | not_applicable | selected_methods is empty and no execution record exists, so there is nothing to compare. All analysis used baseline lenses with a null method reference. |
| secure-coding-pack | not_assessed | No resolved pack was supplied. TR-001, TR-002, TR-003 and TR-016 depend on technology behavior and carry an unavailable pack dependency linked to G-006. |

## Decisions

The caller supplied no decisions. These are suggestions awaiting the owner; none has an owner or a date.

- **DEC-001** (subjects: RK-001, RK-002, RK-003, RK-004 and others): Proposed for the owner's approval, not applied: rate each risk on impact (low: one user's convenience; medium: one user's data or the service's availability; high: any user's account or admin authority) and on how many preconditions the specs leave open (few, some, many). No rating was calculated from this scale.
- **DEC-002** (subjects: TR-009, TR-017, T-009, T-017): Awaiting the owner: what must be decided before the system is reachable beyond localhost (sign-up gating, target constraints, transport security, proxy trust, rate-limit key).
- **DEC-003** (subjects: TR-015, T-015): Awaiting the owner: passkey listing and removal, and whether sensitive account actions need a fresh passkey check.
- **DEC-004** (subjects: TR-012, T-012): Awaiting the owner: accept unlimited redirect load for the demo, or bound it in a way that keeps FR-3.5.
- **DEC-005** (subjects: RQ-FR-1.11, CTL-SC-016): Awaiting the owner: CF-001, whether an admin may delete their own account through FR-1.11.
- **DEC-006** (subjects: TR-011): Awaiting the owner (optional alternative): whether audit-required events should go to a separate file with their own retention, which would change OD-004 and OD-005.

## Proposed requirements

- **RQP-001**: Internal user IDs are never reused, including after account deletion. (T-004)
- **RQP-002**: A target URL that contains embedded credentials is rejected with a reason. (T-008)
- **RQP-003**: A title that contains control or direction-override characters is rejected with a reason. (T-008)
- **RQP-004**: The own-domain rule covers every name that reaches the service itself. (T-010)
- **RQP-005**: API tokens are accepted only for the link operations of the JSON API. (T-005)
- **RQP-006**: API calls without a valid token are limited per network address like sign-up and login. (T-011)
- **RQP-007**: Account deletion is all-or-nothing. (T-014)

## Action register

Owners and dates are unknown for every row.

| ID | Kind | Status | Threat | Next evidence or action |
|---|---|---|---|---|
| TR-001 | treatment | proposed | T-001 | Owner reviews; if agreed, fold into the specs. |
| TR-002 | treatment | proposed | T-002 | Owner reviews; if agreed, fold into the specs. |
| TR-003 | treatment | proposed | T-003 | Owner reviews; if agreed, fold into the specs. |
| TR-004 | treatment | proposed | T-004 | Owner reviews; if agreed, fold into the specs. |
| TR-005 | treatment | proposed | T-005 | Owner reviews; if agreed, fold into the specs. |
| TR-006 | treatment | proposed | T-006 | Owner reviews; if agreed, fold into the specs. |
| TR-007 | treatment | proposed | T-007 | Owner reviews; if agreed, fold into the specs. |
| TR-008 | treatment | proposed | T-008 | Owner reviews; if agreed, fold into the specs. |
| TR-009 | treatment | proposed | T-009 | Owner reviews; if agreed, fold into the specs. |
| TR-010 | treatment | proposed | T-010 | Owner reviews; if agreed, fold into the specs. |
| TR-011 | treatment | proposed | T-011 | Owner reviews; if agreed, fold into the specs. |
| TR-012 | treatment | proposed | T-012 | Owner reviews; if agreed, fold into the specs. |
| TR-013 | treatment | proposed | T-013 | Owner reviews; if agreed, fold into the specs. |
| TR-014 | treatment | proposed | T-014 | Owner reviews; if agreed, fold into the specs. |
| TR-015 | treatment | proposed | T-015 | Owner reviews; if agreed, fold into the specs. |
| TR-016 | treatment | proposed | T-016 | Owner reviews; if agreed, fold into the specs. |
| TR-017 | treatment | proposed | T-017 | Owner reviews; if agreed, fold into the specs. |
| V-001 | verification | not_run | T-001 | Run after implementation, in an isolated setup, when authorized. |
| V-002 | verification | not_run | T-002 | Run after implementation, in an isolated setup, when authorized. |
| V-003 | verification | not_run | T-003 | Run after implementation, in an isolated setup, when authorized. |
| V-004 | verification | not_run | T-004 | Run after implementation, in an isolated setup, when authorized. |
| V-005 | verification | not_run | T-005 | Run after implementation, in an isolated setup, when authorized. |
| V-006 | verification | not_run | T-006 | Run after implementation, in an isolated setup, when authorized. |
| V-007 | verification | not_run | T-007 | Run after implementation, in an isolated setup, when authorized. |
| V-008 | verification | not_run | T-008 | Run after implementation, in an isolated setup, when authorized. |
| V-009 | verification | not_run | T-009 | Run after implementation, in an isolated setup, when authorized. |
| V-010 | verification | not_run | T-010 | Run after implementation, in an isolated setup, when authorized. |
| V-011 | verification | not_run | T-011 | Run after implementation, in an isolated setup, when authorized. |
| V-012 | verification | not_run | T-012 | Run after implementation, in an isolated setup, when authorized. |
| V-013 | verification | not_run | T-013 | Run after implementation, in an isolated setup, when authorized. |
| V-014 | verification | not_run | T-014 | Run after implementation, in an isolated setup, when authorized. |
| V-015 | verification | not_run | T-015 | Run after implementation, in an isolated setup, when authorized. |
| V-016 | verification | not_run | T-016 | Run after implementation, in an isolated setup, when authorized. |
| V-017 | verification | not_run | T-017 | Run after implementation, in an isolated setup, when authorized. |
| G-001 | gap | open | - | Build the system, then run V-001 to V-017. |
| G-002 | gap | open | - | Add a route map to ARCHITECTURE.md. |
| G-003 | gap | open | - | Specify challenge binding and one-time consumption (TR-002). |
| G-004 | gap | open | - | Specify user ID generation and non-reuse (TR-004). |
| G-005 | gap | open | - | Specify the credential type each route accepts (TR-005). |
| G-006 | gap | open | - | Supply a resolved secure-coding pack, or accept that technology details stay unverified. |
| G-007 | gap | open | - | Specify the rate-limit key for unauthenticated and session API calls (TR-011). |
| G-008 | gap | open | - | Specify atomic account deletion (TR-014). |
| G-009 | gap | open | - | Owner decision before any hosting (DEC-002). |
| G-010 | gap | open | - | State the discoverable-credential setting in SC-011. |
| G-011 | gap | open | - | Browser test once the app runs. |
| G-012 | gap | open | - | Specify Host handling (TR-003). |
| G-013 | gap | open | - | Owner decision (DEC-003). |
| G-014 | gap | open | - | Specify cache directives for API responses (TR-016). |
| G-015 | gap | open | - | Specify the short-code random source (TR-013). |
| G-016 | gap | open | - | Specify event volume bounds (TR-011); optional owner decision DEC-006. |
| G-017 | gap | open | - | Specify target normalization and title characters (TR-008, TR-010). |
| G-018 | gap | open | - | Specify cleanup of expired rows and per-user caps. |
| G-019 | gap | open | - | Run the verification plans once the system is built. |

## What did not run

- No code was analyzed, built or executed; none exists.
- No named threat-modeling method, privacy analysis or regulatory profile ran.
- No verification plan ran. No rating was calculated.
- No secure-coding pack was resolved, so technology-specific details are unverified (G-006).
- The cited external sources S1 to S6 were not opened.
- JSON checks that ran: each envelope parses, has no duplicate keys or non-finite numbers, has unique record IDs and has no dangling references; each stage's changes were computed against its parent. No schema validator exists in the library, so full contract validation did not run.

## Folded into the specs

This section was added after the run finished. It records where each finding went. The run's JSON does not contain it. Everything below is still a specification; nothing is implemented.

| Threat | Spec changes |
|---|---|
| T-001 | SECURITY.md SC-015 (evidence; GET wording fixed for CF-002). No new rule; the designed control stands. |
| T-002 | SECURITY.md SC-011 (challenge bound to the browser, deleted before verify), SC-015 (covers sign-up and login); ARCHITECTURE.md data flow 1, TB-2, login diagram; SECURITY_EVENTS.md SEC-034. |
| T-003 | SECURITY.md SC-015 (runs before rate limiting), SC-026 (Host check); ARCHITECTURE.md TB-1; SECURITY_EVENTS.md SEC-034, SEC-037. |
| T-004 | REQUIREMENTS.md FR-7.9; SECURITY.md SC-027; ARCHITECTURE.md Auth row, TB-4; SECURITY_EVENTS.md SEC-019. |
| T-005 | REQUIREMENTS.md FR-5.9; SECURITY.md SC-028; ARCHITECTURE.md route map, TB-2, data flow 5, component diagram; SECURITY_EVENTS.md SEC-026. |
| T-006 | SECURITY.md SC-016 (evidence). No new rule; the designed control stands. |
| T-007 | SECURITY.md SC-009 (evidence); DESIGN.md user-supplied content. No new rule. |
| T-008 | REQUIREMENTS.md FR-2.12, FR-2.13, FR-2.14; SECURITY.md SC-003; ARCHITECTURE.md data flow 2, TB-7; DESIGN.md input errors and user-supplied content; SECURITY_EVENTS.md SEC-031. |
| T-009 | SECURITY.md SC-032 (BLOCKED), OD-008; ARCHITECTURE.md TB-7. |
| T-010 | REQUIREMENTS.md FR-2.10; SECURITY.md SC-003; ARCHITECTURE.md TB-7; DESIGN.md input errors. |
| T-011 | REQUIREMENTS.md NFR-1.3; SECURITY.md SC-018, SC-025; SECURITY_EVENTS.md event contract (Volume), SEC-026, SEC-086. |
| T-012 | SECURITY.md SC-034 (BLOCKED), OD-010. |
| T-013 | SECURITY.md SC-029. |
| T-014 | REQUIREMENTS.md FR-1.12; SECURITY.md SC-030; ARCHITECTURE.md Account deletion row, DR-5. |
| T-015 | SECURITY.md SC-033 (BLOCKED), OD-009. |
| T-016 | SECURITY.md SC-031. |
| T-017 | SECURITY.md SC-026, SC-032 (BLOCKED), OD-008; SECURITY_EVENTS.md SEC-062. |
| CF-001 | SECURITY.md OD-011 (open). |
| CF-002 | SECURITY.md SC-015 wording. |

Not folded in: G-010 (discoverable credentials), G-011 (browser behavior on localhost) and G-018 (cleanup and caps) are gaps with no threat that requires a rule yet. DEC-001 (a rating scale) and DEC-006 (a separate audit file) remain suggestions.

## Owner decisions answered after the run

OD-008 to OD-011 were answered as proposed defaults for the owner to review. The run's JSON still shows these threats and the conflict as open, with decisions DEC-002 to DEC-005 proposed; the next change review should reconcile them.

| Decision | Answer | Spec changes | Threats |
|---|---|---|---|
| OD-008 | Localhost only. The server refuses any other origin; hosting needs a spec and threat-model revision first. | REQUIREMENTS.md NFR-1.4; SECURITY.md SC-032 (now ACTIVE); ARCHITECTURE.md Configuration row, TB-7. | T-009, T-017 |
| OD-009 | Users can list and remove passkeys, except the last. Sensitive account actions need a passkey check from the last 5 minutes. | REQUIREMENTS.md FR-1.13 to FR-1.15; SECURITY.md SC-033 (now ACTIVE); ARCHITECTURE.md Auth row, data flow 7, route map; DESIGN.md account page; SECURITY_EVENTS.md SEC-001, SEC-008, SEC-015. | T-015 |
| OD-010 | Unlimited redirect traffic is accepted for the localhost demo. The redirect is held to one database statement per request. | REQUIREMENTS.md NFR-1.5; SECURITY.md SC-034 (now ACTIVE). | T-012 |
| OD-011 | An admin cannot delete their own account by any route while their ID is in the admin configuration. | REQUIREMENTS.md FR-1.11, FR-7.10; SECURITY.md SC-016; ARCHITECTURE.md Admin row; DESIGN.md delete-account dialog; SECURITY_EVENTS.md SEC-049. | CF-001, T-004 |
