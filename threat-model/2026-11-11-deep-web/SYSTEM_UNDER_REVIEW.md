# SYSTEM_UNDER_REVIEW

Bundle: shrink-182, revision `10b7ddf` (branch `main`); working tree `not_checked: Git clean/process filters are configured globally (LFS), so the runbook's filter rule skips status and working-tree diff. The agent session's start-of-session snapshot reported a clean tree; that is a carried claim, not a check made in this run.`

Run: `MODE=deep`, `REVIEW_SCOPE=security`, `METHODS=[]`, `PRIVACY_PROFILES=[]`. Roots: `PROJECT_ROOT` = the shrink-182 checkout, `LIBRARY_ROOT` = a sibling Grim Prompter checkout at library revision `d7c9082` (2026-11-08). `RUN_DIR` = `threat-model/2026-11-11-deep-web/`. Prompts: `deep/web/00`–`05` (primary) and `deep/api/00`–`05` (secondary), combined under `reference/composition-guide.md`. No architecture selector ran because the caller named both architectures. No library registry was supplied, so architecture identities were checked only against the library's file listing.

Purpose: threat model the specifications before any code is written, so supported findings can be folded back into the specs. Primary architecture: `web`. Secondary: `api` (bounded to the JSON API's caller identity, object/function/property authority, and sequencing).

Scope: the single shrink-182 process as specified: React SPA, Fastify HTTP layer, Auth, Links, Redirect handler, Admin, Account deletion, Configuration, SQLite store, security event file; the browser/session boundary; the bearer-token JSON API; the anonymous redirect.

Out of scope: application code (none exists); the host OS, GitHub repository settings and CI (outside the app boundary per SECURITY_EVENTS.md); a hosted deployment (the specs define only a localhost demo); privacy review scope (not requested); supply-chain architecture analysis beyond what the web/API lenses touch.

Boundary ownership for the combined run: `web` owns TB-1, TB-2, TB-5, TB-7. `api` owns TB-3, TB-4, TB-6.

Absent surfaces, recorded as not applicable from the evidence: file uploads and previews (FR-6.2), payments (FR-6.1), shared caches or reverse proxies, third-party scripts or external resources (fonts are self-hosted), background jobs and asynchronous exports, callbacks/webhooks and outbound requests, identity federation/OAuth, multi-tenancy.

## Evidence

All spec evidence is documentation of a proposed design. None of it is an implementation or runtime observation.

E-001 — documentation claim. Locator: `REQUIREMENTS.md:22-32`. FR-1: passkey-only sign-up and login, open sign-up, additional passkeys, no recovery, logout, 24 h idle / 7 day absolute session expiry, self-service account deletion that removes the user's links.

E-002 — documentation claim. Locator: `REQUIREMENTS.md:36-46`. FR-2: link creation by a logged-in user; server-generated 7-character random code; only well-formed `http`/`https` targets up to 2,048 characters, not on the site's own domain; title up to 200 characters.

E-003 — documentation claim. Locator: `REQUIREMENTS.md:50-57`. FR-3: anonymous 302 redirect, click count on every request, no visitor-identifying data stored, no filtering of repeat or automated visits, links never expire.

E-004 — documentation claim. Locator: `REQUIREMENTS.md:61-73`. FR-4: own-link list, edit, delete; other users' links rejected and unchanged; identical not-found for missing and not-owned; deleted codes never reused.

E-005 — documentation claim. Locator: `REQUIREMENTS.md:77-84`. FR-5: web UI and JSON API with the same rules; personal API tokens shown once, listable, revocable; API identifies the caller by token; account deletion revokes tokens.

E-006 — documentation claim. Locator: `REQUIREMENTS.md:88-89`. FR-6: no payments, no file uploads.

E-007 — documentation claim. Locator: `REQUIREMENTS.md:93-100`. FR-7: two roles; admin assigned only in deployment configuration by internal user ID; admin lists all links and users, deletes any link, deletes other users; the user's own internal ID is shown on their account page.

E-008 — documentation claim. Locator: `REQUIREMENTS.md:104-106`. NFR-1: up to 100 users, single deployment; 10 sign-up/login attempts per minute per network address, 60 API calls per minute per token; redirects are not rate-limited; sign-ups are not capped.

E-009 — documentation claim. Locator: `ARCHITECTURE.md:9-21`. One Node.js process at `http://localhost`; one JSON API serves the web UI (session cookie) and API clients (bearer token); server-side sessions; one SQLite file; pino logging.

E-010 — documentation claim. Locator: `ARCHITECTURE.md:23-40`. Stack and versions: Node 24, Fastify 5, React 19, `node:sqlite`, `@simplewebauthn/*`, `@fastify/cookie`, `@fastify/static`, `@fastify/rate-limit`, Vite, TypeScript, Vitest. No router, ORM or CSS framework.

E-011 — documentation claim. Locator: `ARCHITECTURE.md:42-54`. Component table: responsibilities, rules enforced and data owned per component.

E-012 — documentation claim. Locator: `ARCHITECTURE.md:56-63`. Six main data flows: passkey sign-up/login, create link, follow link, manage own links, API tokens (created with a session only), admin actions (session caller, configured admin list).

E-013 — documentation claim. Locator: `ARCHITECTURE.md:65-73`. Trust boundaries TB-1 to TB-7 and their enforcement points.

E-014 — documentation claim. Locator: `ARCHITECTURE.md:75-84`. Dependency rules DR-1 to DR-8, including Auth as the sole owner of identity tables and the Redirect handler not depending on Auth.

E-015 — diagram. Locator: `ARCHITECTURE.md:86-169`. Component/trust-boundary flowchart and the redirect and passkey-login sequence diagrams.

E-016 — documentation claim. Locator: `SECURITY.md:7-19`. Applicability decisions; items declared not applicable (passwords, uploads, payments, SSR, SSRF, recovery, proxies); `trustProxy` stays `false`; the Node permission model is not used.

E-017 — documentation claim. Locator: `SECURITY.md:32-65`. SC-001 to SC-005: body and time limits, a schema on every route, WHATWG URL parsing with scheme, length and own-host checks, no backtracking regexes, no event-loop blocking.

E-018 — documentation claim. Locator: `SECURITY.md:67-86`. SC-006 to SC-008: bound parameters, prototype-pollution safety, no dynamic code execution.

E-019 — documentation claim. Locator: `SECURITY.md:88-100`. SC-009 and SC-010: plain-text rendering, no target URL in `href`, a fixed CSP limited to `'self'`, `nosniff`, `no-referrer`.

E-020 — documentation claim. Locator: `SECURITY.md:102-114`. SC-011 and SC-012: ceremony verification with a server-stored single-use 5-minute challenge, configured origin and RP ID, required user verification; credential lookup by ID, counter handling, `excludeCredentials`.

E-021 — documentation claim. Locator: `SECURITY.md:116-121`. SC-013: 32-byte random session ID in a `__Host-session` cookie (HttpOnly, Secure, SameSite=Lax), stored hashed, rotated at login, deleted on logout, idle and absolute expiry.

E-022 — documentation claim. Locator: `SECURITY.md:123-128`. SC-014: 32-byte tokens, shown once, stored hashed, bearer header only, rejected when revoked or when the owner is deleted, never created with a token.

E-023 — documentation claim. Locator: `SECURITY.md:130-135`. SC-015: for non-GET routes authenticated by the session cookie, require `Sec-Fetch-Site: same-origin` (or a matching `Origin` when absent) and a JSON content type; no GET changes state; no CORS.

E-024 — documentation claim. Locator: `SECURITY.md:137-142`. SC-016: ownership filter in the Links module with a uniform not-found; admin check against the configured list on every admin action; the verification text says an admin cannot delete themself.

E-025 — documentation claim. Locator: `SECURITY.md:144-149`. SC-017: uniform JSON errors without internals.

E-026 — documentation claim. Locator: `SECURITY.md:151-156`. SC-018: in-memory rate limits; sign-up and login keyed by socket address; API routes keyed by the token hash already computed for lookup; redirects not limited.

E-027 — documentation claim. Locator: `SECURITY.md:158-170`. SC-019 and SC-020: logs omit IP, user agent, cookies, authorization and bodies; the redirect returns 302 only to a stored target with `Cache-Control: no-store`.

E-028 — documentation claim. Locator: `SECURITY.md:172-198`. SC-021 to SC-024: response schemas, configuration from validated environment variables, dependency and install rules, database file outside the static root with mode `0600`.

E-029 — documentation claim. Locator: `SECURITY.md:200-216`. SC-025 and the security event policy: one local JSON-lines file, rotation at 10 MiB × 5, best-effort audit events with counted loss; no open decisions.

E-030 — documentation claim. Locator: `SECURITY_EVENTS.md:13-69`. Event contract: envelope, allowlisted conditional fields, 2 KiB limit, never-log list.

E-031 — documentation claim. Locator: `SECURITY_EVENTS.md:71-161`. Ledger of SEC-001 to SEC-099 and SEC-APP-001/002 with dispositions; every applicable event is `missing` because nothing is built.

E-032 — documentation claim. Locator: `DESIGN.md:296-326`. Security-relevant interactions: signed-in state, confirmed destructive actions, uniform error wording, user content as plain text, target URLs not live links.

E-033 — documentation claim. Locator: `DESIGN.md:232`, `DESIGN.md:286-289`. Fonts are self-hosted; link rows show the full host, shorten long paths, and show the full URL as a tooltip.

E-034 — documentation claim. Locator: `README.md:11`. "Nothing is built yet. The project is in the design phase."

E-035 — configuration observation. Locator: tracked-file listing at `10b7ddf`. The repository tracks only `AGENTS.md`, `ARCHITECTURE.md`, `DESIGN.md`, `README.md`, `REQUIREMENTS.md`, `REQUIREMENT_TEMPLATE.md`, `SECURITY.md`, `SECURITY_EVENTS.md`, `design/preview.html` and `logo.svg`. There is no application source, manifest, lockfile, test or configuration file.

E-036 — caller claim. Locator: caller request for this run. Nothing is implemented; record that as an evidence gap; run deep web and API with one report and one set of `T-nnn` IDs; a missing detail is a gap, not a vulnerability.

E-037 — implementation observation (partial). Locator: `design/preview.html` (pattern search only). A search for URLs, `src=`, `href=`, `<script`, `@import` and `url(` found only example URL strings inside form fields and one `href="#"`; no external resource load was found by that search.

E-038 — documentation claim. Locator: `SECURITY_EVENTS.md:6-9`. The ledger targets specifications only; both gates are NOT RUN.

## Gaps

G-001 — Unknown: whether any specified control works. No implementation exists (E-034, E-035, E-036). Settled by: source code and test results once built.

G-002 — Unknown: the URL namespace. The specs show `GET /<code>` and one example route template `/api/links/:code`, but not how short codes, SPA screens, static files and API routes are kept apart. Settled by: a route map.

G-003 — Unknown: how a WebAuthn challenge is tied to the browser that asked for it, and whether it is consumed atomically. Settled by: a rule or design for challenge binding and consumption.

G-004 — Unknown: the format of the internal user ID, how it is generated, and whether an ID can be reused after deletion. Settled by: an ID rule.

G-005 — Unknown: which routes accept a bearer token and which accept only a session, and what happens when a request carries both. Settled by: a per-route credential map.

G-006 — Unknown: no exact, resolved secure-coding pack was supplied for the stack. Technology-specific prescriptions stay unverified. Settled by: a resolved pack with provenance.

G-007 — Unknown: how the rate limiter treats API calls with a missing or unknown token, and API calls made with a session cookie. Settled by: a rule naming the key for each case.

G-008 — Unknown: whether account deletion across the Auth and Links modules is one transaction. Settled by: a rule or design.

G-009 — Unknown: anything about a hosted deployment (TLS, proxy, HSTS, real client addresses). Settled by: a deployment design; until then hosted exposure is out of scope.

G-010 — Unknown: whether login uses discoverable credentials (users have no username). Settled by: the registration and login option settings.

## Inspection ledger

Inspection coverage: discovery by the tracked-file listing at `10b7ddf` (10 files, complete listing). Untracked files were not enumerated beyond a directory listing of the root, which showed only an empty `.claude/` folder besides tracked files.

Files inspected:
- `REQUIREMENTS.md:1-110` — full read.
- `ARCHITECTURE.md:1-177` — full read.
- `SECURITY.md:1-218` — full read.
- `SECURITY_EVENTS.md:1-165` — full read.
- `DESIGN.md` — partial: lines 181-330 (prose) read in full; lines 1-180 (design-token front matter) read as key names and a URL search only, values mostly unread.
- `AGENTS.md:1-62` — full read.
- `README.md:1-17` — full read.
- `design/preview.html` — partial: pattern search for external references only (E-037); markup and styles otherwise unread.

Files or areas not inspected:
- `REQUIREMENT_TEMPLATE.md` — skipped: an issue template, not system evidence.
- `logo.svg` — skipped: a 313-byte image, not system evidence.
- `.git/` — excluded by the runbook.
- External sources S1–S6 cited by `SECURITY.md` — unavailable: no network use is authorized.

Unmapped trust boundaries: none suspected beyond TB-1 to TB-7. The route namespace (G-002) is an open topology gap inside TB-1.

Redactions: none found after inspection. The specs contain no secrets, addresses or personal data.
