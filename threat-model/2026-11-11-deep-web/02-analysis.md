# 02 Threat analysis (TM_ANALYSIS, revision r02)

## Inspection coverage

- **Read in full in this run (revision `10b7ddf`):** `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `SECURITY_EVENTS.md`, `AGENTS.md`, `README.md`.
- **Partly read:** `DESIGN.md` (prose lines 181-330 in full; the design-token front matter as key names only) and `design/preview.html` (a search for external references only).
- **Not read:** `REQUIREMENT_TEMPLATE.md` and `logo.svg` (not system evidence); the external sources S1-S6 that `SECURITY.md` cites (no network use was authorized).
- **No implementation exists.** Every finding is a design hypothesis about the specifications, and every control is `designed`, not implemented or tested.
- **Unmapped boundaries:** none beyond TB-1 to TB-7. The route namespace inside TB-1 is an open topology gap (G-002).
- **Working tree:** not checked with Git, because globally configured clean/process filters trigger the runbook's skip rule.

All 17 threats are design hypotheses about the specifications. None is an evidenced weakness. None is rated; no rubric was supplied. No method ran; the baseline lenses carry no method reference.

| ID | Arch. | Confidence | Scenario | Gaps |
|---|---|---|---|---|
| T-001 | web | low | A page from another origin makes the signed-in user's browser send a state-changing request (create, edit or delete a link, revoke a token, delete the account, an admin delete) that the server accepts on the strength of the session cookie alone. | G-001 |
| T-002 | web | low | Sign-up and login routes are not authenticated by the session cookie, so the wording of SC-015 does not cover them. A cross-site page submits a login completion for the attacker's own account, or a second party completes a ceremony against a challenge issued to someone else. The victim ends up signed in as the attacker and saves links (target URLs) into the attacker's account. | G-003 |
| T-003 | web, api | medium | Sign-up and login are limited to 10 attempts per minute per socket address. On the localhost demo every browser tab and local process shares the loopback address, and the ceremony routes are reachable cross-site. Any page the user visits, or any local process, can spend the allowance and keep the user locked out of login. | G-012 |
| T-004 | api, web | medium | Admin authority is granted to whoever holds an internal user ID listed in configuration. The specs do not say how IDs are generated or that they are never reused. If IDs are sequential or reusable, a new sign-up can receive an ID that is still in the admin list after the original admin account was deleted, and becomes an admin. Reuse would also mis-attribute retained security events. | G-004 |
| T-005 | api, web | medium | One JSON API accepts a session cookie or a bearer token, but the specs do not list which routes accept which. A leaked or over-shared API token could reach account, passkey, token-management or admin routes, and a request carrying both credentials could be checked under one and acted on under the other. | G-005 |
| T-006 | api, web | low | A signed-in user or API client changes the short code in a view, edit or delete call to reach a link owned by someone else, or calls an admin route without the admin role. The admin list-all route is the one bulk path; it returns every user's links and IDs. | G-001 |
| T-007 | web | low | A user stores a title or target URL crafted to be interpreted as markup or a live link. It is later shown in another user's context: the admin's list of all links, a delete-confirmation dialog that names the link and owner, or a tooltip. If interpreted, it runs with the admin's session. | G-001 |
| T-008 | web | medium | A target URL is accepted and displayed in a form that misleads the reader about where it goes: a trusted-looking name before an @ sign, an internationalized look-alike host shown in Unicode, or a title containing direction-override or control characters that reorders what is displayed. DESIGN.md relies on the visible host to expose look-alike domains. | G-017 |
| T-009 | web, api | low | Sign-up is open and uncapped, and any http/https target is allowed. Anyone who can reach the site can create an account and use shrink-182 links as a trusted-looking front for phishing or malware destinations. Removal depends on an admin noticing. | G-009 |
| T-010 | api, web | medium | FR-2.10 rejects targets on the site's own domain, and SC-003 compares the target host with the configured site host. A target that names the same server another way (a different spelling of the loopback host, a trailing dot, another port on the same host) passes the comparison and lets links point back at the service, including at each other in a loop. | G-017 |
| T-011 | api | medium | API rate limits are keyed by the hash of the presented token. A caller who sends a different random token on each request never reaches a limit. Each rejected request also writes an auth.api_token_rejected event, so the caller can fill the 10 MiB x 5 event file and rotate audit-required events out before the operator reads them. | G-007, G-016 |
| T-012 | web | medium | Every anonymous request for an existing short link performs a synchronous database write to count the click, and redirects are deliberately not rate-limited. A flood of redirect requests can occupy the single process and also inflate any link's click count. | - |
| T-013 | web | low | Short codes are the only thing protecting a link's target from anonymous readers. Someone guesses codes against the unlimited redirect route to harvest target URLs, which may include private document links. | G-015 |
| T-014 | api | low | Account deletion spans the Auth module (user, passkeys, sessions, tokens) and the Links module (links, retired codes). If it is not atomic, a failure midway leaves live links without an owner or tokens and sessions that still work for a deleted user, and a request already in flight may act for an account that is being deleted. | G-008 |
| T-015 | web | medium | Someone who briefly controls a signed-in browser (an unattended laptop, a stolen session) adds their own passkey. The session alone authorizes it. The owner cannot see or remove passkeys, so the intruder keeps access after the session ends; the only remedy is deleting the account. The same session can create an API token or delete the account. | G-013 |
| T-016 | web | low | Authenticated JSON responses (the link list, the admin lists, the one-time token value) carry no cache directive. On a shared machine they may remain in the browser's cache or history after logout or session expiry. | G-014 |
| T-017 | web, api | low | Several controls are justified only by the localhost demo: no TLS or HSTS, a Secure cookie that works because localhost is treated as secure, trustProxy off, rate limits keyed by socket address, no Host check. If the same build is put on a network or behind a proxy without revisiting them, the controls weaken silently: all clients may share one rate-limit key, cookies travel without TLS guarantees, and open sign-up meets real outsiders. | G-009, G-011, G-012 |

## Lens coverage

| Lens | Status | Reason |
|---|---|---|
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

## Gaps added in this stage

- **G-013**: The specs give no way to list or remove a single passkey and require no fresh passkey check before adding a passkey, creating a token or deleting the account.
- **G-014**: Cache directives are specified only for the redirect. Nothing says how authenticated JSON responses, including the one-time token, are kept out of the browser cache.
- **G-015**: No secure-coding rule says that short codes come from a cryptographic random source with unbiased selection; only the event ledger implies it.
- **G-016**: Nothing bounds how many security events one unauthenticated caller can cause, and audit-required events share one rotating file with high-volume denial events.
- **G-017**: SC-003 does not say what is stored and shown for a target: the submitted string or the parsed, serialized URL; nor whether embedded credentials or control characters are allowed.
