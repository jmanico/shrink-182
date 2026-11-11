# shrink-182 — Secure-Coding Rules

This file owns the implementation rules for writing shrink-182 securely. WHAT the system does is in `REQUIREMENTS.md`, how it looks is in `DESIGN.md`, and how it is built is in `ARCHITECTURE.md`. Rule IDs (`SC-###`) and decision IDs (`OD-###`) are permanent and are never renumbered or reused. This is a demo, so rules are kept to what the confirmed stack needs.

> **Note on answers:** The owner asked for the open decisions to be answered as best as possible. OD-001 to OD-007 were answered with proposed defaults.

## Secure-Coding Inputs

- **Documents read:** `REQUIREMENTS.md` (FR-1 to FR-7, NFR-1), `DESIGN.md` (security-relevant interactions), `ARCHITECTURE.md` (stack, components, TB-1 to TB-7, DR-1 to DR-8). No earlier `SECURITY.md` existed.
- **Owner notes:** demo, keep it simple; passkeys are the only authentication; two roles, user and admin. These match FR-1.4 and FR-7.1.
- **Confirmed components and versions:** see [`ARCHITECTURE.md` → Stack](ARCHITECTURE.md#stack).
- **Applicability decisions:**
  - Applies: request and resource limits, rate limits, event-loop blocking, input validation, output encoding, authentication (passkeys, API tokens), authorization (ownership, admin role), sessions, CSRF, errors, logging, dependencies, prototype pollution, unsafe code execution, security headers, open redirect (TB-7).
  - Runtime permissions: the Node permission model is not used for this localhost demo, to keep it simple (OD-003, proposed default). SC-024 covers file protection instead.
  - Not applicable: password storage (FR-1.4, no passwords); file uploads (FR-6.2); payments (FR-6.1); server-side rendering and SSR serialization (SPA only); outbound server requests / SSRF (no external integrations, and the redirect handler never fetches the target); account recovery (FR-1.8, none exists); `npm publish` hygiene (app is not published); HTTP/2 and proxy smuggling (no reverse proxy in the localhost demo, `trustProxy` stays `false`).
- **Gaps and conflicts:**
  - **Conflict:** Fastify's default request logging includes the client address, while FR-3.5 forbids storing visitor-identifying data on redirects. Resolved by SC-019 (requirement wins).
  - **Gap:** the React cheat sheet is a draft with no published revision date; its guidance is used only where it agrees with the OWASP XSS and CSRF cheat sheets or React's own behavior.
  - Example packages in the guides (helmet, hpp, acl, DOMPurify, csurf-style middleware) are not approved dependencies and are not used.

## Sources

- **S1 — Node.js, "Security Best Practices"** — https://nodejs.org/en/learn/getting-started/security-best-practices — sections: DoS of HTTP server, DNS rebinding, Timing attacks, Malicious third-party modules / Supply chain attacks, Prototype pollution, Node.js permission model, Experimental features in production. Retrieved 2026-11-11. No revision date on page. Supports: Node.js server, npm toolchain.
- **S2 — OWASP Cheat Sheet Series, "Node.js Security Cheat Sheet"** — https://cheatsheetseries.owasp.org/cheatsheets/Nodejs_Security_Cheat_Sheet.html — sections: Application Security (request size limits, event loop, input validation, output escaping, logging, brute-forcing, anti-CSRF, only return what is necessary), Error & Exception Handling, Server Security (cookie flags, security headers), Platform Security (packages, dangerous functions, evil regexes, strict mode). Retrieved 2026-11-11. Supports: Fastify server.
- **S3 — OWASP, "React Security Cheat Sheet" (draft)** — https://github.com/OWASP/CheatSheetSeries/blob/master/cheatsheets_draft/React_Security_Cheat_Sheet.md — sections: XSS Prevention (dangerouslySetInnerHTML, Validate URLs before rendering, Direct DOM manipulation, Prop spreading, Dynamic code execution), Sensitive Data Exposure (HttpOnly cookies, data out of URLs, environment variables), Authentication and Authorization, Other Considerations (CSP). Retrieved 2026-11-11, `master` branch, no commit date shown. Supports: React SPA, Vite build.
- **S4 — OWASP, "Cross-Site Request Forgery Prevention Cheat Sheet"** — https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html — sections: Employing Custom Request Headers, Fetch Metadata headers, Using Standard Headers to Verify Origin, SameSite, Disallowing simple requests. Retrieved 2026-11-11. Supports: Fastify JSON API with session cookie.
- **S5 — Fastify, "Server" reference (v5.12.8)** — https://fastify.dev/docs/latest/Reference/Server/ — options `bodyLimit` (default 1 MiB), `connectionTimeout` (0), `requestTimeout` (0), `maxParamLength` (100), `trustProxy` (false), `onProtoPoisoning` / `onConstructorPoisoning` (`'error'`). Retrieved 2026-11-11. Supports: HTTP layer.
- **S6 — SimpleWebAuthn, "@simplewebauthn/server" docs** — https://simplewebauthn.dev/docs/packages/server — sections: Registration (`generateRegistrationOptions` `excludeCredentials`, `verifyRegistrationResponse`), Authentication (`verifyAuthenticationResponse`, counter handling). Retrieved 2026-11-11, current docs (v14 line). Supports: Auth module.

## Secure-Coding Rules

**SC-001 — Request size and time limits**
- **Status:** ACTIVE
- **Rule:** The Fastify server MUST keep `bodyLimit` at or below 16 KiB for JSON routes and MUST set non-zero `connectionTimeout` and `requestTimeout` (project choice: 10 s and 30 s).
- **Enforcement:** HTTP layer, Fastify factory options.
- **Evidence:** ARCHITECTURE.md HTTP layer ("request shape and size limits"); FR-2.9, FR-2.11. S2 "Set request size limits"; S1 "DoS of HTTP server" (timeouts); S5 defaults (1 MiB, 0, 0). Exact values are a project decision.
- **Verification:** A test posting a 32 KiB JSON body gets 413; a server config test asserts both timeouts are non-zero.

**SC-002 — Schema on every route**
- **Status:** ACTIVE
- **Rule:** Every route that accepts a body, params, or query MUST declare a Fastify JSON schema with `additionalProperties: false`, string `maxLength`s, and the short-code pattern `^[A-Za-z0-9]{7}$`; handlers MUST NOT read unvalidated fields.
- **Enforcement:** HTTP layer route definitions (TB-1).
- **Evidence:** ARCHITECTURE.md TB-1; FR-2.7, FR-5.3, FR-5.4. S2 "Perform input validation"; S1 "Prototype pollution" (validate with JSON Schema).
- **Verification:** A request with an extra property, an array where a string is expected, or an 8-character code is rejected with a JSON 400 (or 404 for redirects) and no state change.

**SC-003 — Target URL rules in the Links module**
- **Status:** ACTIVE
- **Rule:** The Links module MUST parse target URLs with the WHATWG `URL` class and MUST accept only `http:`/`https:` schemes, length ≤ 2,048, and a host different from the configured site host, on both create and edit.
- **Enforcement:** Links module (DR-2), independent of route schemas.
- **Evidence:** FR-2.8 to FR-2.10, FR-4.10; ARCHITECTURE.md TB-7. S3 "Validate URLs before rendering" (scheme allow-list); S2 "Perform input validation".
- **Verification:** Unit tests reject `javascript:alert(1)`, `data:text/html,x`, `ftp://a.b`, a 2,049-character URL, and a URL on the site's own host; `https://example.com` is accepted.

**SC-004 — No regex backtracking on input**
- **Status:** ACTIVE
- **Rule:** Validation MUST NOT use regular expressions with nested or overlapping quantifiers on user input; URL checks MUST use `URL` parsing, not a URL regex.
- **Enforcement:** Links module, HTTP schemas.
- **Evidence:** S2 "Stay away from evil regexes"; S1 "DoS of HTTP server".
- **Verification:** Code review finds no URL regex; a test validating a 2,048-character crafted URL finishes in under 50 ms.

**SC-005 — Do not block the event loop**
- **Status:** ACTIVE
- **Rule:** Request handlers MUST NOT use synchronous `fs` or `crypto` calls other than `randomBytes`/`randomInt`/`createHash`/`timingSafeEqual` on small inputs, and MUST NOT loop over unbounded user-controlled data.
- **Enforcement:** All server modules. `node:sqlite` is synchronous by design; queries MUST be indexed lookups on bounded data (NFR-1.1).
- **Evidence:** S2 "Do not block the event loop"; ARCHITECTURE.md stack (`node:sqlite`).
- **Verification:** Grep finds no `*Sync(` call in request paths; short-code and token lookups use indexed columns (schema test).

**SC-006 — Bound parameters only**
- **Status:** ACTIVE
- **Rule:** The store MUST build every SQL statement with `prepare()` and bound parameters, and MUST NOT concatenate or template user data into SQL.
- **Enforcement:** SQLite store (DR-6, TB-6).
- **Evidence:** ARCHITECTURE.md TB-6 ("bound parameters only"); S2 "Perform input validation" (injection). Project decision, already in ARCHITECTURE.md.
- **Verification:** Grep finds no SQL string containing `${` or `+` with variables; a link titled `'); DROP TABLE links;--` is stored and read back unchanged.

**SC-007 — Prototype-pollution safety**
- **Status:** ACTIVE
- **Rule:** The server MUST keep Fastify `onProtoPoisoning` and `onConstructorPoisoning` at `'error'` and MUST NOT deep-merge or `Object.assign` request data into existing objects; lookups keyed by user data MUST use `Map` or `Object.hasOwn`.
- **Enforcement:** HTTP layer config; all modules.
- **Evidence:** S1 "Prototype pollution"; S5 option defaults.
- **Verification:** A body containing `"__proto__": {"admin": true}` or `"constructor": {...}` is rejected with 400, and `({}).admin` is still `undefined` afterward.

**SC-008 — No dynamic code execution**
- **Status:** ACTIVE
- **Rule:** Server and client code MUST NOT use `eval`, `new Function`, string `setTimeout`, `node:vm`, or `child_process`.
- **Enforcement:** All code; checked in review.
- **Evidence:** S2 "Do not use dangerous functions"; S3 "Avoid dynamic code execution".
- **Verification:** Grep of `src/` for these names returns nothing.

**SC-009 — Plain-text rendering in React**
- **Status:** ACTIVE
- **Rule:** The web client MUST render titles, target URLs, and tokens only as JSX text; it MUST NOT use `dangerouslySetInnerHTML`, `innerHTML`/`outerHTML`/`insertAdjacentHTML`, or spread server objects into element props, and MUST NOT put target URLs in `href`.
- **Enforcement:** Web client components (TB-5).
- **Evidence:** DESIGN.md "User-supplied content" and Don'ts (no live links); ARCHITECTURE.md TB-5. S3 "Avoid unsafe HTML injection", "Avoid direct DOM manipulation", "Avoid prop injection via spread syntax". Not linking targets is a project decision from DESIGN.md.
- **Verification:** A link titled `<img src=x onerror=alert(1)>` shows as literal text; grep finds none of the banned APIs; the list's DOM has no `<a>` whose `href` is a target URL.

**SC-010 — Security headers**
- **Status:** ACTIVE
- **Rule:** Every response MUST carry `Content-Security-Policy: default-src 'self'; script-src 'self'; style-src 'self'; img-src 'self' data:; font-src 'self'; object-src 'none'; base-uri 'none'; frame-ancestors 'none'; form-action 'self'`, `X-Content-Type-Options: nosniff`, and `Referrer-Policy: no-referrer`, set in one Fastify `onSend` hook without extra packages.
- **Enforcement:** HTTP layer hook.
- **Evidence:** S2 "Use appropriate security headers"; S3 "Other considerations" (CSP); DESIGN.md self-hosted fonts. Header values and "no helmet" are project decisions. HSTS is not set because the demo runs on `http://localhost` (ARCHITECTURE.md).
- **Verification:** A test checks all three headers on `/`, an API route, a 302 redirect, and a 404; a page with an injected inline script is blocked by CSP in the browser console.

**SC-011 — Passkey ceremony verification**
- **Status:** ACTIVE
- **Rule:** The Auth module MUST verify every registration and login with `@simplewebauthn/server`, passing a server-stored single-use challenge (expiring after 5 minutes), `expectedOrigin` and `expectedRPID` from configuration, and `requireUserVerification: true` (with `userVerification: 'required'` in the options sent to the browser), and MUST create no user or session when verification fails.
- **Enforcement:** Auth module (DR-6).
- **Evidence:** FR-1.1 to FR-1.3, FR-1.7; ARCHITECTURE.md data flow 1 and Configuration. S6 Registration and Authentication. The 5-minute expiry and required user verification are project decisions (OD-002, resolved as a proposed default).
- **Verification:** Replaying a used challenge, using a wrong origin, an expired challenge, or an assertion without the user-verified flag returns the generic failure and creates no session row.

**SC-012 — Passkey credential binding and counter**
- **Status:** ACTIVE
- **Rule:** Login MUST look up the stored credential by its ID, verify against that credential's public key, store `newCounter` on success, and reject an assertion whose counter is not higher than a non-zero stored counter; registration MUST pass the user's existing credentials as `excludeCredentials`.
- **Enforcement:** Auth module.
- **Evidence:** FR-1.6, FR-1.7; S6 counter handling and `excludeCredentials`.
- **Verification:** A test with a stored counter of 5 and an assertion counter of 5 fails; a credential belonging to user A cannot log in as user B.

**SC-013 — Session cookie and lifecycle**
- **Status:** ACTIVE
- **Rule:** Sessions MUST use a 32-byte `crypto.randomBytes` ID in a cookie named `__Host-session` with `HttpOnly`, `Secure`, `SameSite=Lax`, `Path=/`; the server MUST store only a SHA-256 hash of the ID, issue a new ID at every login, delete it on logout, and reject it after 24 h idle or 7 days absolute.
- **Enforcement:** Auth module; `@fastify/cookie` in the HTTP layer.
- **Evidence:** FR-1.9, FR-1.10; ARCHITECTURE.md (server-side sessions). S2 "Set cookie flags appropriately"; S3 "Store authentication tokens in HttpOnly cookies"; S4 SameSite (`__Host-` prefix). Hashing and cookie name are project decisions. Browsers treat `http://localhost` as a secure context, so `Secure` works in the demo.
- **Verification:** Set-Cookie shows all attributes; the DB holds no raw session ID; a session older than 7 days or idle 24 h, or after logout, gets 401.

**SC-014 — API tokens**
- **Status:** ACTIVE
- **Rule:** API tokens MUST be 32 random bytes, shown once, stored only as a SHA-256 hash, accepted only in `Authorization: Bearer`, and rejected when revoked or when the owner is deleted; tokens MUST NOT be created with an API token.
- **Enforcement:** Auth module.
- **Evidence:** FR-5.5 to FR-5.8; ARCHITECTURE.md data flow 5. S1 "Timing attacks" (lookup by hash avoids comparing raw secrets); S3 "Keep sensitive data out of URLs". Hash choice is a project decision (high-entropy tokens do not need a slow hash).
- **Verification:** The DB holds no raw token; a revoked token, a token in a query string, and a token of a deleted user each get 401; a token-authenticated call to create a token gets 401/403.

**SC-015 — CSRF for cookie-authenticated requests**
- **Status:** ACTIVE
- **Rule:** For every non-GET route authenticated by the session cookie, the server MUST reject the request unless `Sec-Fetch-Site` is `same-origin` (or, when absent, `Origin` exactly equals the configured site origin) and `Content-Type` is `application/json`; no GET route MAY change state.
- **Enforcement:** HTTP layer `onRequest` hook, before the Auth module.
- **Evidence:** ARCHITECTURE.md TB-2. S4 Fetch Metadata, Verify Origin, SameSite, Disallowing simple requests; S2 "Use anti-CSRF tokens" (don't rely on SameSite alone). No CORS is enabled, so cross-origin scripts cannot send JSON with credentials.
- **Verification:** A POST with `Sec-Fetch-Site: cross-site`, with `Origin: null`, or with `Content-Type: text/plain` gets 403 and changes nothing; the same request from the SPA succeeds.

**SC-016 — Ownership and role checks on the server**
- **Status:** ACTIVE
- **Rule:** The Links module MUST filter or check by the caller's user ID on every link read, edit, and delete, returning the same not-found result for "missing" and "not yours"; the Admin module MUST check the caller's ID against the configured admin list on every admin action and MUST NOT accept a role from the client.
- **Enforcement:** Links module (TB-3), Admin module entry (TB-4).
- **Evidence:** FR-4.3, FR-4.6, FR-4.13, FR-7.1 to FR-7.6; ARCHITECTURE.md TB-3, TB-4. S3 "Authentication and authorization"; S2 "Use access control lists".
- **Verification:** User B's edit/delete of user A's link returns the same 404 body as a random code, and the link is unchanged; a non-admin calling an admin route gets 403; an admin cannot edit another user's link or delete themself.

**SC-017 — Errors**
- **Status:** ACTIVE
- **Rule:** The server MUST return errors as JSON `{ error, message }` with no stack traces, SQL, or internal IDs of other users, and MUST log the detail server-side; on `uncaughtException` it MUST log and exit.
- **Enforcement:** HTTP layer `setErrorHandler`; process entry point.
- **Evidence:** FR-5.4; DESIGN.md "Error messages never show internal details". S2 "Error & exception handling".
- **Verification:** A forced store error returns a generic 500 JSON body with no `stack` field; the log contains the stack.

**SC-018 — Rate limits on sign-up, login, and API**
- **Status:** ACTIVE
- **Rule:** The HTTP layer MUST apply NFR-1.3 with `@fastify/rate-limit` using its in-memory store: sign-up and login routes (options and verify) keyed by the socket address, and API routes keyed by the SHA-256 token hash already computed for lookup. The address key MUST stay in memory only, MUST NOT be logged or written to the database, and MUST expire with the 1-minute window. The redirect handler MUST NOT be rate-limited. `trustProxy` stays `false`.
- **Enforcement:** HTTP layer.
- **Evidence:** NFR-1.3; FR-3.5; SC-019. S2 "Take precautions against brute-forcing". The limits and keys are project decisions (OD-001, resolved as a proposed default).
- **Verification:** The 11th login-options request in a minute from one address gets 429 with a JSON error; the 61st API call with one token gets 429; 100 redirects in a minute all succeed; the logs and database contain no address.

**SC-019 — Logging without visitor data or secrets**
- **Status:** ACTIVE
- **Rule:** The pino logger MUST use serializers that omit the client IP (the only in-memory use of it is SC-018), `User-Agent`, `Cookie`, `Authorization`, and request bodies, and MUST NOT log session IDs, API tokens, WebAuthn challenges, or credential public keys.
- **Enforcement:** HTTP layer logger configuration.
- **Evidence:** FR-3.5 (overrides Fastify's default request log fields); S2 "Perform application activity logging"; S3 "Keep sensitive data out of URLs".
- **Verification:** Following a short link and calling the API with a token, then grepping the log, finds no IP, user agent, cookie, or token value. The never-log list in `SECURITY_EVENTS.md` also applies to security events (SC-025).

**SC-020 — Redirect handler output**
- **Status:** ACTIVE
- **Rule:** The redirect handler MUST respond with 302 only to a target stored by the Links module, MUST set `Cache-Control: no-store`, and MUST NOT read or store any request header or address.
- **Enforcement:** Redirect handler (DR-8).
- **Evidence:** FR-3.1 to FR-3.6; ARCHITECTURE.md TB-7. S2 "Only return what is necessary". `no-store` is a project decision supporting FR-3.6.
- **Verification:** A request for a valid code returns 302 with that exact `Location` and `no-store`; an unknown code returns 404 with no `Location`.

**SC-021 — Minimal API responses**
- **Status:** ACTIVE
- **Rule:** API responses MUST be built with Fastify response schemas listing only allowed fields; users, sessions, tokens (except the one-time value), and credentials MUST NOT be returned whole.
- **Enforcement:** HTTP layer response schemas.
- **Evidence:** S2 "Only return what is necessary"; S3 "Minimize sensitive data". FR-5.5.
- **Verification:** The token list response contains no hash; the admin user list contains no credential data.

**SC-022 — Configuration and secrets**
- **Status:** ACTIVE
- **Rule:** Site origin, admin list, and database path MUST come from server environment variables validated at startup (fail closed if missing); client code MUST NOT read any env var except ones prefixed `VITE_` that are public.
- **Enforcement:** Configuration component; Vite config.
- **Evidence:** ARCHITECTURE.md Configuration; FR-7.2. S3 "Do not expose secrets through environment variables".
- **Verification:** Starting without the origin variable exits with an error; the built client bundle contains no admin IDs or DB path.

**SC-023 — Dependencies and install**
- **Status:** ACTIVE
- **Rule:** The project MUST commit `package-lock.json`, install with `npm ci`, use only dependencies listed in `ARCHITECTURE.md`, set `ignore-scripts=true` in a project `.npmrc`, and pass `npm audit --audit-level=high`.
- **Enforcement:** Toolchain.
- **Evidence:** ARCHITECTURE.md Toolchain. S1 "Supply chain attacks"; S2 "Keep your packages up-to-date". `ignore-scripts` is source guidance adopted by the project.
- **Verification:** `npm ci && npm audit --audit-level=high` exits 0; `package.json` lists no package absent from ARCHITECTURE.md.

**SC-024 — Database file and inspector**
- **Status:** ACTIVE
- **Rule:** The SQLite file MUST live outside the static-file root and be created with mode `0600`; the server MUST NOT be started with `--inspect` outside local debugging.
- **Enforcement:** Store startup; `npm start` script.
- **Evidence:** ARCHITECTURE.md TB-6 ("file protection owned by SECURITY.md"). S1 "DNS rebinding" (inspector); S2 "Do not use dangerous functions" (unsafe `fs`). The Node permission model (S1, S2 "Permissions") is not used for this localhost demo (OD-003, resolved as a proposed default).
- **Verification:** `GET /<db filename>` returns 404; `stat` shows mode 600; `npm start` has no `--inspect`.

**SC-025 — Security events follow the ledger**
- **Status:** ACTIVE
- **Rule:** Security events MUST be emitted only through one helper that builds each record from the per-event allowlist in `SECURITY_EVENTS.md` (Event Contract), and MUST never throw into the request. They MUST be written as described in [Security Event Policy](#security-event-policy).
- **Enforcement:** Event helper used by the Auth, Links, Admin, and Account deletion modules, and by the process entry point.
- **Evidence:** `SECURITY_EVENTS.md`; Dread Ledger `7d38c75` event contract.
- **Verification:** Contract tests per event: presence and absence, field allowlist, secret canaries, CR/LF and oversize input, and helper failure leaving the business result unchanged.

## Security Event Policy

The owner asked for these decisions (OD-004 to OD-007) to be answered as best as possible, so all four are *(proposed default)* answers for the owner to review.

- **Destination and consumer (OD-004):** security events go to a separate local JSON-lines file (pino destination) with mode `0600`, outside the static-file root and separate from the diagnostic log. The consumer is the operator (project owner), who reviews the file by hand. There are no alerts, tickets, or chat forwarding.
- **Retention (OD-005):** the file rotates at 10 MiB and keeps at most 5 files. Events are kept only while the demo checkout exists, and deleting the checkout disposes of them. There is no legal hold.
- **Personal data (OD-006):** events may carry the opaque internal user ID as `actor_ref` and `target_ref`. They are kept after account deletion for attribution, but no other profile data is kept. No IP, user agent, or `session_audit_ref` is collected.
- **Audit durability (OD-007):** `audit_required=true` for `account.deleted`, `link.deleted` (`action=admin`), `admin.read`, `config.loaded`, and `config.admins_loaded`. Writing them is best-effort and never blocks or rolls back the business change; losses are counted by `logging.health`. There is no transactional outbox. The 10 MiB × 5 rotation caps how long evidence survives.

## Open Security Decisions

None open. Resolved and removed: OD-001 (rate limits, SC-018, NFR-1.3), OD-004 to OD-007 (see [Security Event Policy](#security-event-policy)), OD-002 (passkey user verification is required, SC-011) and OD-003 (Node permission model not used in the demo, SC-024). The owner asked for these to be answered as best as possible, so all are *(proposed default)* answers for the owner to review.
