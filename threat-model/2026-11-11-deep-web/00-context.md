# 00 Scope and evidence (TM_CONTEXT, revision r00)

## Inspection coverage

- **Read in full in this run (revision `10b7ddf`):** `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `SECURITY_EVENTS.md`, `AGENTS.md`, `README.md`.
- **Partly read:** `DESIGN.md` (prose lines 181-330 in full; the design-token front matter as key names only) and `design/preview.html` (a search for external references only).
- **Not read:** `REQUIREMENT_TEMPLATE.md` and `logo.svg` (not system evidence); the external sources S1-S6 that `SECURITY.md` cites (no network use was authorized).
- **No implementation exists.** Every finding is a design hypothesis about the specifications, and every control is `designed`, not implemented or tested.
- **Unmapped boundaries:** none beyond TB-1 to TB-7. The route namespace inside TB-1 is an open topology gap (G-002).
- **Working tree:** not checked with Git, because globally configured clean/process filters trigger the runbook's skip rule.

## Scope

Specification-level threat model of shrink-182 at revision `10b7ddf`. Architectures: `web` (primary) and `api` (secondary). Review scope: security. No methods, privacy profiles, rubric, prior model or secure-coding pack were supplied.

Out of scope: code (none exists), host and CI, a hosted deployment, privacy.

## Evidence

38 evidence records (E-001 to E-038). 35 are specification claims about a proposed design. E-035 observes that the repository contains no code. E-036 is the caller's statement. E-037 is a partial observation of the static preview.

## Open gaps

- **G-001** (evidence): No implementation exists, so no specified control can be observed working. Needed: Source code and test results once the system is built.
- **G-002** (topology): The URL namespace is not specified: how short codes, SPA screens, static files and API routes are kept apart. Needed: A route map with the path prefix for each surface.
- **G-003** (evidence): The specs do not say how a WebAuthn challenge is tied to the browser that requested it, or that it is consumed atomically. Needed: A rule for challenge binding and one-time consumption.
- **G-004** (evidence): The format of the internal user ID, how it is generated and whether it can be reused after deletion are not specified. Needed: A rule for user ID generation and reuse.
- **G-005** (evidence): The specs do not list which routes accept a bearer token and which accept only a session, or what happens when a request carries both. Needed: A per-route credential map.
- **G-006** (technology): No exact, resolved secure-coding pack was supplied for the stack, so technology-specific prescriptions stay unverified. Needed: A resolved pack with group, technology, version and resolver provenance.
- **G-007** (evidence): The rate-limit key for API calls with a missing or unknown token, and for API calls made with a session cookie, is not specified. Needed: A rule naming the key for each case.
- **G-008** (evidence): The specs do not say whether account deletion across the Auth and Links modules is one transaction. Needed: A rule or design for the deletion's atomicity.
- **G-009** (evidence): Nothing is specified about a hosted deployment (TLS, proxy, HSTS, real client addresses). Hosted exposure is out of scope. Needed: A deployment design, if the system is ever hosted.
- **G-010** (evidence): The specs do not say whether login uses discoverable credentials, although users have no username. Needed: The registration and login option settings.

## Not done in this stage

No system model and no threat analysis; those belong to stages 01 and 02. No baseline comparison ran because stage 00 has no parent.
