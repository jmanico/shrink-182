# 01 System model (TM_SYSTEM_MODEL, revision r01)

## Inspection coverage

- **Read in full in this run (revision `10b7ddf`):** `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `SECURITY_EVENTS.md`, `AGENTS.md`, `README.md`.
- **Partly read:** `DESIGN.md` (prose lines 181-330 in full; the design-token front matter as key names only) and `design/preview.html` (a search for external references only).
- **Not read:** `REQUIREMENT_TEMPLATE.md` and `logo.svg` (not system evidence); the external sources S1-S6 that `SECURITY.md` cites (no network use was authorized).
- **No implementation exists.** Every finding is a design hypothesis about the specifications, and every control is `designed`, not implemented or tested.
- **Unmapped boundaries:** none beyond TB-1 to TB-7. The route namespace inside TB-1 is an open topology gap (G-002).
- **Working tree:** not checked with Git, because globally configured clean/process filters trigger the runbook's skip rule.

Everything below is reconstructed from specification text. Each element has basis `proposed_design`; each control has status `designed`.

## Zones
| ID | Description |
|---|---|
| Z-internet | Untrusted network side: anything that can send a request to the server. |
| Z-browser | The user's browser running the React SPA, alongside pages from other origins. |
| Z-server | The single Node.js process (Fastify and the domain modules). |
| Z-host | Local host resources the process uses: SQLite file, event and diagnostic log files, environment configuration. |
| Z-external | External target sites that short links redirect to. |

## Actors
| ID | Description |
|---|---|
| ACT-visitor | Anonymous visitor who follows short links or opens the signed-out page. |
| ACT-user | Signed-in user who owns links, passkeys and API tokens. |
| ACT-admin | User whose internal ID is in the configured admin list; can list everything and delete links and other users. |
| ACT-api-client | Program calling the JSON API with a user's bearer token. |
| ACT-operator | Project owner who sets environment configuration and reads the security event file. |
| ACT-other-origin | A page from another origin open in the user's browser, able to send cross-site requests to the server. |

## Components and stores
| ID | Description |
|---|---|
| AS-spa | React single-page app: sign-up, login, links, tokens, account and admin screens. |
| AS-http | Fastify HTTP layer: routing, schemas, cookies, static files, rate limits, headers, error shape. |
| AS-auth | Auth module: passkeys, sessions, API tokens, caller identity and role. |
| AS-links | Links module: create, list, edit, delete, code generation, URL rules, ownership. |
| AS-redirect | Redirect handler: resolves a code, counts the click, returns 302 or not-found. |
| AS-admin | Admin module: role check, list all links and users, delete links and other users. |
| AS-acct-del | Account deletion: removes a user with their links, tokens and sessions by coordinating Auth and Links. |
| AS-config | Configuration read from environment variables at startup: admin list, site origin (also the RP ID), database path. |
| AS-db | One SQLite file holding users, passkeys, sessions, tokens, links and retired codes. |
| AS-event-log | Local JSON-lines security event file (mode 0600, rotating 10 MiB x 5). |
| AS-vendor | Third-party packages running in the process or the page: Fastify and its plugins, @simplewebauthn, React; plus the built-in node:sqlite (release candidate). |

## Data
| ID | Description |
|---|---|
| D-session | Session ID (cookie value; stored as a SHA-256 hash). |
| D-passkey | Passkey credential record: credential ID, public key, counter. |
| D-token | API token (shown once; stored as a SHA-256 hash). |
| D-challenge | Single-use WebAuthn challenge, valid 5 minutes. |
| D-link | Short link: code, target URL, optional title, owner, click count. |
| D-user-id | Internal user ID; shown to its owner, listed to admins, used in the admin list and in events. |
| D-admin-list | Admin list of internal user IDs in deployment configuration. |
| D-events | Security events with opaque actor and target references. |

## Entry points
| ID | Description |
|---|---|
| EP-redirect | Anonymous GET /<code> short-link redirect. |
| EP-static | Static files and the SPA shell, including the signed-out page. |
| EP-ceremony | Unauthenticated sign-up and login routes (options and verify). |
| EP-api-links | JSON API for creating, listing, editing and deleting links; session cookie or bearer token. |
| EP-api-account | Account routes: logout, add passkey, create/list/revoke tokens, delete account. |
| EP-api-admin | Admin routes: list all links and users, delete any link, delete another user. |

## Trust boundaries
| ID | Owner | Description |
|---|---|---|
| TB-1 | web | Internet to server: all requests are untrusted input; validated in the HTTP layer and again in the Links module. |
| TB-2 | web | Anonymous to authenticated: authentication is enforced once in the Auth module; session cookies for the web UI, bearer tokens for the JSON API. |
| TB-3 | api | User to another user's data: ownership is enforced in the Links module on every read, edit and delete. |
| TB-4 | api | User to admin: the role is enforced at the Admin module entry using the configured admin list. |
| TB-5 | web | Server to browser rendering: user titles and URLs are output as plain text by React. |
| TB-6 | api | Server to SQLite file: trusted local store, bound parameters only. |
| TB-7 | web | Redirect to external target: visitors are sent to user-supplied http/https URLs that are not the site's own domain. |

## Flows
| ID | Description |
|---|---|
| F-1 | Passkey sign-up or login: options request, challenge, WebAuthn in the browser, verify, session cookie set. |
| F-2 | Create a link from the web UI (cookie) or an API client (token). |
| F-3 | Follow a short link: look up the code, increment the count, 302 to the target or not-found. |
| F-4 | List, edit and delete own links; not-owned and missing give the same not-found. |
| F-5 | Create (session only), list and revoke API tokens; the value is shown once. |
| F-6 | Admin actions: list all links and users, delete a link, delete another user. |
| F-7 | API client calls the links API with a bearer token. |
| F-8 | Delete an account: remove the user, their links, tokens and sessions. |
| F-9 | Modules write security events to the local event file. |
| F-10 | Operator sets the admin list, origin and database path; the process reads them at startup. |
| F-11 | A page from another origin sends a cross-site request to the server from the user's browser. |
| F-12 | The SPA renders user-supplied titles and target URLs in the owner's list and in the admin list. |

## States
| ID | Description |
|---|---|
| ST-signed-out | No session; only sign-up, login, redirect, signed-out page and static files are reachable. |
| ST-signed-in | Valid session within the idle and absolute limits. |
| ST-session-ended | Session ended by logout, idle expiry or absolute expiry; the ID is no longer accepted. |
| ST-account-deleted | Account deleted by its owner or an admin; links removed, tokens revoked, codes stay retired. |
| ST-token-revoked | API token revoked by its owner or by account deletion. |
| ST-link-deleted | Link deleted; its code answers not-found and is never reassigned. |

## Dependencies
| ID | Description |
|---|---|
| DEP-1 | Auth relies on @simplewebauthn/server for attestation and assertion verification. |
| DEP-2 | The HTTP layer relies on Fastify, @fastify/cookie, @fastify/static and @fastify/rate-limit. |
| DEP-3 | The store relies on node:sqlite, a release-candidate built-in. |
| DEP-4 | Admin authority depends on the admin list in configuration. |

## Supplied controls (all `designed`)
| ID | Description |
|---|---|
| CTL-SC-001 | SC-001: request size and time limits. |
| CTL-SC-002 | SC-002: JSON schema on every route, with the short-code pattern. |
| CTL-SC-003 | SC-003: target URL rules in the Links module: WHATWG parsing, http/https only, length limit, not the site's own host. |
| CTL-SC-009 | SC-009: titles, URLs and tokens rendered only as JSX text; no target URL in href. |
| CTL-SC-010 | SC-010: CSP limited to 'self', nosniff and no-referrer on every response. |
| CTL-SC-011 | SC-011: passkey ceremony verification with a single-use 5-minute challenge, configured origin and RP ID, required user verification. |
| CTL-SC-012 | SC-012: credential lookup by ID, counter check, excludeCredentials. |
| CTL-SC-013 | SC-013: __Host-session cookie (HttpOnly, Secure, SameSite=Lax), hashed storage, new ID at login, deletion at logout, idle and absolute expiry. |
| CTL-SC-014 | SC-014: API tokens: random, shown once, hashed, bearer header only, rejected when revoked or owner deleted, not creatable with a token. |
| CTL-SC-015 | SC-015: Fetch-metadata/Origin and JSON content-type check on non-GET routes authenticated by the session cookie. |
| CTL-SC-016 | SC-016: ownership filter with uniform not-found in the Links module; admin-list check on every admin action. |
| CTL-SC-018 | SC-018: in-memory rate limits: sign-up/login by socket address, API by token hash; redirects unlimited. |
| CTL-SC-019 | SC-019: logs without client address, user agent, cookies, authorization or bodies. |
| CTL-SC-020 | SC-020: redirect returns 302 only to a stored target, with Cache-Control: no-store. |
| CTL-SC-021 | SC-021: response schemas listing only allowed fields. |
| CTL-SC-022 | SC-022: origin, admin list and database path from validated environment variables; fail closed. |
| CTL-SC-024 | SC-024: database file outside the static root with mode 0600; no inspector. |
| CTL-SC-025 | SC-025: security events only through one allowlisting helper that never throws. |

## Gaps added in this stage
- **G-011** and **G-012** (see JSON). Gaps G-002 to G-010 were linked to the system records they affect.

## Not done in this stage
No threat analysis. No method ran.
