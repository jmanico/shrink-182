# shrink-182 — Security Events

This file is the security event ledger the build has to satisfy. It records which events the code must emit, with what fields, and for whom. Requirements are in `REQUIREMENTS.md`, components in `ARCHITECTURE.md`, and secure-coding rules and owner decisions (`OD-###`) in `SECURITY.md`. IDs (`SEC-###`, `SEC-APP-###`) are permanent and are never renumbered or reused.

- **Library revision:** Dread Ledger `7d38c755e0943fea52f32f293521d522c20aaeae` (2026-11-09), from https://github.com/jmanico/dread-ledger. Inputs: `runbook.md`, `prompts/00-shared-instruction.md`, `prompts/01-assess.md`, `reference/coverage.md`, `reference/event-contract.md`, `reference/security-events.md`, `reference/verification.md`.
- **Target revision:** specifications only, at `e7e99c7` (`REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, `DESIGN.md`). No application code exists.
- **Scope:** the single shrink-182 process (Fastify HTTP layer, Auth, Links, Redirect, Admin, Account deletion, Configuration, SQLite store) as specified. The host, the GitHub repository, and any CI are outside this boundary and are recorded as `unavailable_signal`.
- **Threat model update (2026-11-11):** rows SEC-019, SEC-026, SEC-031, SEC-034, SEC-037, SEC-062 and SEC-086 and the event contract were extended from `THREAT_MODEL.md` (T-002, T-003, T-004, T-005, T-008, T-011, T-017). No event was removed or renamed and no ID changed. Each change cites its threat ID. The answers to OD-008 to OD-011 then extended SEC-001, SEC-008, SEC-015 and SEC-049 (T-015, T-004).
- **Gate local:** NOT RUN. No code to verify.
- **Gate deployment:** NOT RUN. The policy is set; nothing is built yet.
- **Checklist reconciliation:** done for all 99 SEC IDs against the specifications. Nothing is `covered`, because nothing has passed verification.
- **No real logs were read.** This file contains no secrets, log content, or operational details.

## Event Contract

**Logger:** Fastify's built-in pino logger (`ARCHITECTURE.md` Stack and "Logging library"). Security events are pino records written through one Auth/Links/Admin helper (`securityEvent(name, fields)`). The helper builds the record from a fixed per-event allowlist. No route or module calls `log.info` with security data directly. SC-019 serializers still apply to every other log line.

**Schema version:** `schema_version: 1`. Any change to a field's meaning bumps the version. Consumers reject or quarantine unknown versions visibly and never coerce them.

### Envelope (every event)

| Field | Type and limit | Trusted source | Notes |
|---|---|---|---|
| `schema_version` | integer, `1` | Constant in the helper | |
| `event_id` | string, UUID v4, 36 chars | `crypto.randomUUID()` in the helper | Never taken from a request header or client ID |
| `event` | string, `^[a-z][a-z0-9_.]{2,63}$` | Constant from the event catalog below | Never concatenated with data |
| `timestamp` | string, ISO 8601 UTC with ms, 24 chars | Server clock at the decision point | pino `timestamp: isoTime` |
| `service` | string, `"shrink-182"` | Constant | |
| `environment` | string, ≤ 16 chars, enum `local` \| `test` | Configuration at startup (SC-022) | Not request input |
| `level` | pino level | Event catalog | `info` = success or attempt, `warn` = denied or failure, `error` = unknown outcome or producer fault. Not tied to wave or confidence. *(proposed default)* |
| `outcome` | enum: `success`, `failure`, `denied`, `attempted`, `rolled_back`, `unknown` | Code at the authoritative decision or commit | Meanings as in Dread Ledger: login rejection = `failure`, ownership or role rejection = `denied`, a mutation is `success` only after the SQLite commit |
| `security_relevant` | boolean, `true` | Constant | |
| `audit_required` | boolean | Event catalog | Set per event by [policy](SECURITY.md#security-event-policy). Never chosen by a caller and never defaulted to `false`. |

### Conditional fields (allowlisted per event in the Ledger)

| Field | Type and limit | Trusted source |
|---|---|---|
| `actor_ref` | string, ≤ 64 chars, opaque internal user ID | Auth module only, from the verified session or validated API token. Never from a body, param, or header. Omitted when no identity is established. Logging it at all is set by [policy](SECURITY.md#security-event-policy). |
| `actor_kind` | enum: `user`, `admin`, `anonymous`, `unknown` | Auth module (role from the configured admin list, FR-7.2) |
| `actor_provenance` | enum: `verified_session`, `validated_token` | Auth module. Omitted when no identity is established. |
| `target_ref` | string, ≤ 64 chars: a 7-char short code, or an internal user ID | Links or Auth module, after the record is loaded *by the caller's own permission*. A denied request never triggers a lookup only to enrich the event. |
| `target_kind` | enum: `link`, `user`, `api_token`, `passkey`, `session` | Event catalog |
| `action` | enum per event, ≤ 32 chars | Event catalog |
| `reason_code` | enum per event, ≤ 48 chars (for example `challenge_expired`, `origin_mismatch`, `uv_missing`, `counter_regressed`, `unknown_credential`, `token_revoked`, `not_found_or_not_owned`, `not_admin`, `url_scheme`, `url_too_long`, `url_self_domain`, `title_too_long`, `schema_violation`, `fetch_site_cross`, `content_type`, `rate_limited`, `url_credentials`, `title_characters`, `host_mismatch`, `token_not_allowed`, `mixed_credentials`, `admin_unmatched`, `origin_not_local`, `reverify_required`, `last_passkey`) | Code at the decision point. Never an exception message. |
| `token_record_ref` | string, ≤ 64 chars | The API token's own opaque row ID. Never the token value or its hash. |
| `credential_record_ref` | string, ≤ 64 chars | The passkey record's own opaque row ID. Never the WebAuthn credential ID, public key, or a hash of them. |
| `route_template` | string, ≤ 64 chars, e.g. `/api/links/:code` | Fastify `routeOptions.url`. Never the raw URL. |
| `request_id` | string, ≤ 36 chars | Fastify `genReqId` set to a server-generated UUID. Incoming `request-id` headers are ignored. This is correlation, not identity. |
| `count` | integer ≥ 0 | Server-computed, e.g. links removed by an account deletion |
| `context_status` | enum: `optional_context_omitted`, `optional_context_invalid` | Helper, when optional fields fail validation or the event exceeds its size limit |

**Tenant fields:** not used. shrink-182 has one team and no tenants (FR-1.5, NFR-1.2), so `tenant_ref`, `actor_tenant_ref`, and `resource_tenant_ref` are omitted. Reassess if multi-team support is added.

**Size:** every encoded event is ≤ 2 KiB. If optional context would exceed that, the helper drops the optional fields and sets `context_status`. It never truncates the envelope or the serialized JSON.

**Volume:** for each event name, the helper writes at most 60 records per minute that have no `actor_ref` and whose outcome is `denied` or `failure`. Further ones are counted, not written, and the count is reported as `suppressed` in `logging.health` (SEC-086). Records with an `actor_ref` and events with `audit_required=true` are never suppressed (T-018). This keeps an anonymous caller from rotating audit evidence out of the file (SC-025, T-011).

**Failure:** the helper never throws into the request. Encoding failures increment a bounded in-process counter, reported as described in SEC-086. They are never re-logged recursively. Audit failure policy: best-effort with counted loss ([policy](SECURITY.md#security-event-policy)).

### Never log

Including in errors, pino serializers, fallbacks, and test fixtures:

- Session IDs, the `__Host-session` cookie, or their SHA-256 hashes.
- API token values or their hashes, and the `Authorization` header.
- WebAuthn challenges, `clientDataJSON`, attestation or assertion objects, credential IDs, public keys, and signature counters.
- Client IP addresses, `User-Agent`, `Referer`, or any other request header. This applies everywhere, not only on redirects (FR-3.5, SC-019).
- Target URLs, link titles, query strings, raw URLs, and request or response bodies.
- Raw exception messages or stacks inside security events (SC-017 diagnostic logs are a separate stream).
- Rejected input values, arbitrary serialized objects, configuration values, the database path, and the admin list contents.
- Any hash or fingerprint of the items above, used as a stand-in.

## Ledger

Disposition rules: `missing` = applicable to the specs but not yet built and verified; `not_applicable` = the specs show the feature is absent (with the change that would reopen it); `unavailable_signal` = applicability or signal is uncertain, needs a detector that does not exist, or sits outside the app boundary. **Consumer:** the operator reading the local security event file ([policy](SECURITY.md#security-event-policy)). Waves: 0 foundation, 1 critical, 2 identity and access, 3 validation and abuse, 4 context.

| ID | Disposition | Trigger (requirement / component) | Proposed event and fields | Consumer | Wave |
|---|---|---|---|---|---|
| SEC-001 | missing | FR-1.2, FR-1.3, FR-1.7, FR-5.7 / Auth module (passkey login verify, bearer token check) | `auth.login` (success, failure): `actor_ref` (success only), `actor_kind`, `actor_provenance`, `credential_record_ref` (success only), `action` (`login` \| `reverify`; a re-verification before a sensitive account action, SC-033, T-015), `reason_code`, `request_id`. `auth.api_token_rejected`: see SEC-026. | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-002 | unavailable_signal | Auth module | No lockout or reset exists (FR-1.8). "Success after repeated failures" needs a correlation detector that does not exist. | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-003 | unavailable_signal | Auth module, HTTP layer | No passwords (FR-1.4). There is no spraying or enumeration detector. Throttled attempts are recorded by SEC-040 `rate_limited`. | Operator ([policy](SECURITY.md#security-event-policy)) | 3 |
| SEC-004 | not_applicable | Auth module | No lockout, suspension, or disable state exists. Deleted accounts are removed (FR-1.11), so a later attempt shows up as a SEC-001 `unknown_credential` failure. Reopen if suspension is added. | — | — |
| SEC-005 | not_applicable | FR-1.4, FR-1.8 | No passwords and no recovery. Reopen if either is added. | — | — |
| SEC-006 | not_applicable | FR-1.2, SC-011 | A passkey with required user verification is the only factor. There is no separate MFA challenge or push. Reopen if a second factor is added. | — | — |
| SEC-007 | not_applicable | FR-1.4, FR-1.8 | No MFA enrollment and no recovery codes. User verification is fixed to required in code (SC-011), so it cannot be downgraded. Passkey registration is SEC-008. | — | — |
| SEC-008 | missing | FR-1.1, FR-1.6 / Auth module registration | `auth.passkey_registered` (success, failure): `actor_ref`, `actor_provenance` (FR-1.6 only), `credential_record_ref`, `action` (`signup` \| `add`), `reason_code`. Removal (FR-1.14, T-015): `auth.passkey_removed` (success, denied): `actor_ref`, `actor_provenance`, `credential_record_ref`, `reason_code` (`last_passkey` when the only passkey is refused). Removals during account deletion are counted in SEC-052. | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-009 | not_applicable | FR-1.8, REQUIREMENTS data entities | No recovery email, phone, or ownership transfer. Users have no contact fields. Adding a passkey is SEC-008. | — | — |
| SEC-010 | unavailable_signal | FR-7.2 / Configuration | No service accounts or shared accounts are specified. Dormant-account use would need a detector that does not exist. | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-011 | unavailable_signal | FR-3.5, SC-019 | Location and IP are excluded by policy, so the signal cannot exist. Reopen only through a personal-data decision. | — | — |
| SEC-012 | unavailable_signal | SC-019 | Device and network context is excluded by policy (same as SEC-011). | — | — |
| SEC-013 | unavailable_signal | Auth module sessions | No concurrent-session or anomalous-token detector exists. | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-014 | unavailable_signal | Auth module | No phishing or stolen-cookie detector. Passkeys are origin-bound (SC-011). | — | — |
| SEC-015 | missing | FR-4.6, FR-4.7, FR-7.6 / Links module (TB-3), Admin entry (TB-4), Auth (TB-2) | `authz.denied` (denied): `actor_ref`, `actor_kind`, `actor_provenance`, `action` (`link_read` \| `link_edit` \| `link_delete` \| `admin_*`), `route_template`, `reason_code` (`not_found_or_not_owned` \| `not_admin` \| `not_authenticated` \| `reverify_required` with `action` `passkey_add` \| `passkey_remove` \| `token_create` \| `account_delete`; SC-033, T-015), `request_id`. No `target_ref` for not-owned links, so another user's link is never confirmed. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-016 | not_applicable | NFR-1.2, FR-1.5 | Single team with no tenant boundary. Reopen if multi-team support is added. | — | — |
| SEC-017 | missing | FR-4.6 / Links module | Mapped to `authz.denied` with `reason_code=not_found_or_not_owned` (shared event, no duplicate emission). Unknown and not-owned codes give the same result by design, so the event cannot tell manipulation from a typo. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-018 | missing | FR-7.2, FR-7.6 / Admin entry | Attempts: `authz.denied` with `reason_code=not_admin`. Successful elevation can only happen through configuration and is recorded in SEC-019. No temporary grants exist. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-019 | missing | FR-1.1, FR-7.1, FR-7.2, FR-1.11, FR-7.5 / Auth, Configuration | `account.created` (success): `actor_ref`, `actor_kind=user`. Role changes: `config.admins_loaded` at startup (success, failure): `count` only, never the IDs. When admin-list entries match no current user, a second `config.admins_loaded` record is emitted with `outcome=failure`, `reason_code=admin_unmatched` and `count` of the unmatched entries; startup continues (SC-027, T-004). Deletions: SEC-052. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-020 | not_applicable | FR-7.4 | No impersonation, support access, or break-glass mode. Admins act as themselves. | — | — |
| SEC-021 | missing | FR-7.2 / Configuration, Admin and Links checks | `config.admins_loaded` (SEC-019). `authz.check_failed` (unknown): `action`, `route_template`, `reason_code=evaluation_error`, emitted when a store error occurs during an ownership or role check. The request still fails closed (SC-016). | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-022 | unavailable_signal | SC-016 | Needs an independent verification signal that does not exist. Covered by tests, not runtime events. | — | — |
| SEC-023 | missing | FR-1.9, FR-1.10 / Auth sessions | `session.created`, `session.ended` (`action`: `logout` \| `idle_expired` \| `absolute_expired` \| `account_deleted`): `actor_ref`, `actor_provenance`, `outcome`. No session ID or hash; no `session_audit_ref` (not approved, [policy](SECURITY.md#security-event-policy)). | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-024 | unavailable_signal | SC-013 | Logged-out sessions are deleted, so a revoked ID cannot be told apart from a random one. No hijack detector. Expired-session use is SEC-023. | — | — |
| SEC-025 | missing | FR-5.5, FR-5.6, FR-5.8 / Auth tokens | `api_token.created`, `api_token.revoked` (`action`: `user` \| `account_deleted`): `actor_ref`, `actor_provenance=verified_session`, `token_record_ref`. No token value or hash, and no scope field (tokens have no scopes). | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-026 | missing | FR-5.7, SC-014 / Auth bearer check | `auth.api_token_rejected` (failure): `reason_code` (`missing` \| `unknown` \| `revoked` \| `owner_deleted` \| `query_string` \| `token_not_allowed` for a token on a session-only route \| `mixed_credentials` for a cookie and token together; SC-028, T-005), `route_template`, `request_id`. `token_record_ref` only when a revoked record was matched. Subject to the volume ceiling (T-011). | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-027 | missing | SC-011, SC-012 / Auth WebAuthn | OAuth subcases are not_applicable. Challenge replay and counter regression are recorded on `auth.login` or `auth.passkey_registered` failure with `reason_code` `challenge_reused` \| `challenge_expired` \| `counter_regressed`. | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-028 | not_applicable | ARCHITECTURE (no OAuth) | No OAuth or OIDC. WebAuthn origin and RP ID failures are SEC-001 `reason_code=origin_mismatch`. | — | — |
| SEC-029 | not_applicable | ARCHITECTURE (no OAuth) | No OAuth applications or consent. | — | — |
| SEC-030 | not_applicable | REQUIREMENTS (no integrations) | No SSO or federation. The RP ID comes from startup config (SEC-062). | — | — |
| SEC-031 | missing | FR-2.4, FR-2.8 to FR-2.11, FR-4.10, SC-002, SC-003 / HTTP schemas, Links rules | `input.rejected` (denied): `actor_ref` (if known), `route_template`, `reason_code` (`schema_violation` \| `url_scheme` \| `url_malformed` \| `url_too_long` \| `url_self_domain` \| `title_too_long` \| `proto_poisoning` \| `url_credentials` \| `title_characters`; FR-2.12, FR-2.13, T-008). Never the rejected value. | Operator ([policy](SECURITY.md#security-event-policy)) | 3 |
| SEC-032 | unavailable_signal | SC-006 | Bound parameters prevent SQL injection. No injection detector exists, and none will be built. | — | — |
| SEC-033 | unavailable_signal | SC-009, `@fastify/static` | No XSS or traversal detector. Static traversal is refused by the library and appears only as a 404. | — | — |
| SEC-034 | missing | SC-015 / HTTP `onRequest` hook, including sign-up and login routes (T-002, T-003) | `csrf.rejected` (denied): `route_template`, `reason_code` (`fetch_site_cross` \| `origin_mismatch` \| `content_type`), `request_id`. No header values. | Operator ([policy](SECURITY.md#security-event-policy)) | 3 |
| SEC-035 | unavailable_signal | SC-010 | CSP has no `report-to` endpoint. Adding one would be a new collection point, which needs an owner decision. | — | — |
| SEC-036 | not_applicable | SECURITY.md applicability (no outbound requests) | The server never fetches a target. Reopen if link previews or URL checks that fetch are added. | — | — |
| SEC-037 | missing | SC-001, SC-002, SC-007 / Fastify | `request.rejected` (denied): `reason_code` (`body_too_large` \| `bad_json` \| `method_not_allowed` \| `timeout` \| `rate_limited` \| `host_mismatch` from SC-026, T-003), `route_template` where matched. Smuggling: not_applicable (no proxy, `trustProxy=false`). | Operator ([policy](SECURITY.md#security-event-policy)) | 3 |
| SEC-038 | not_applicable | FR-6.2 | No file uploads. | — | — |
| SEC-039 | unavailable_signal | FR-3.3, FR-3.5 / Redirect handler | Short-code guessing produces 404s that cannot be attributed (FR-3.5), and no enumeration detector exists. See SEC-APP-001. | — | — |
| SEC-040 | missing | SC-001 (size and timeout); SC-018, NFR-1.3 (rate limits) | `request.rejected` (SEC-037, shared) for size and timeout, and with `reason_code=rate_limited` and `route_template` for SC-018 refusals. No address or token value. | Operator ([policy](SECURITY.md#security-event-policy)) | 3 |
| SEC-041 | missing | FR-7.3, FR-7.5 / Admin module | `admin.read` (success): `actor_ref`, `actor_kind=admin`, `action` (`list_links` \| `list_users`), `count`. Denied attempts: `authz.denied`. A user listing their own links is not a sensitive read. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-042 | unavailable_signal | FR-7.3 | Admin list-all is bulk by design (SEC-041). There is no volume detector. | — | — |
| SEC-043 | missing | FR-2.3, FR-4.4, FR-4.5, FR-7.4 / Links module | `link.created`, `link.updated` (`action`: `title` \| `target` \| `both`), `link.deleted` (`action`: `owner` \| `admin` \| `account_deleted`): `actor_ref`, `actor_kind`, `actor_provenance`, `target_ref` (short code), `outcome` after commit. No URL or title. | Operator ([policy](SECURITY.md#security-event-policy)) | 4 (admin deletes: 1) |
| SEC-044 | missing | FR-3.2 / Links module | Every short link is public. Creating one is public-link creation, mapped to `link.created` (SEC-043, shared). Changing a public link's target is `link.updated` `action=target`. | Operator ([policy](SECURITY.md#security-event-policy)) | 4 |
| SEC-045 | unavailable_signal | — | No DLP control. Targets are never fetched or scanned. | — | — |
| SEC-046 | not_applicable | ARCHITECTURE (embedded `node:sqlite`) | No database users, grants, or audit settings. File mode is set at startup (SC-024, SEC-068). | — | — |
| SEC-047 | unavailable_signal | — | Backups are not specified. They belong to the operator or host. | — | — |
| SEC-048 | unavailable_signal | FR-1.11, FR-7.5 | Mass deletion is recorded per action (SEC-052, with `count`). There is no pattern detector. | — | — |
| SEC-049 | missing | FR-7.4, FR-7.5, SC-014 / Admin, Auth | `authz.denied` with `reason_code` `admin_self_delete` (from the admin route or the admin's own account page, FR-7.10) \| `admin_edit_foreign_link` \| `token_via_token`. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-050 | unavailable_signal | FR-2.6, FR-4.12 | No idempotency keys and no replay detector. Code collisions are SEC-APP-002. | — | — |
| SEC-051 | not_applicable | FR-6.1 | No payments. | — | — |
| SEC-052 | missing | FR-1.11, FR-5.8, FR-7.4, FR-7.5 / Account deletion, Admin | `account.deleted` (success, rolled_back): `actor_ref`, `actor_kind`, `target_ref` (deleted user ID, per [policy](SECURITY.md#security-event-policy)), `action` (`self` \| `admin`), `count` (links removed), plus `api_token.revoked` and `session.ended` with `action=account_deleted`. Admin link delete: `link.deleted` `action=admin`. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-053 | unavailable_signal | SC-018 | No fraud controls. Generic throttling is SEC-040. | — | — |
| SEC-054 | missing | SC-011, SC-012 / Auth | WebAuthn signature or attestation failures: `auth.login` / `auth.passkey_registered` failure with `reason_code=signature_invalid` (shared event). | Operator ([policy](SECURITY.md#security-event-policy)) | 2 |
| SEC-055 | not_applicable | ARCHITECTURE | The app manages no keys. Passkey public keys are user credentials (SEC-008). | — | — |
| SEC-056 | not_applicable | ARCHITECTURE | No key management service. | — | — |
| SEC-057 | not_applicable | SC-022 | No secret store. Config comes from environment variables at startup. | — | — |
| SEC-058 | not_applicable | ARCHITECTURE (`http://localhost`) | No TLS in the demo. Reopen when hosted. | — | — |
| SEC-059 | not_applicable | ARCHITECTURE | No certificates. Reopen when hosted. | — | — |
| SEC-060 | not_applicable | SC-011, SC-013 | Crypto settings are fixed in code with no runtime toggle. | — | — |
| SEC-061 | missing | FR-2.7, SC-013, SC-014 / Links, Auth | A `crypto.randomBytes` or `randomInt` throw produces `system.fault` (unknown) with `reason_code=rng_failure`. Nonce reuse: not_applicable. | Operator ([policy](SECURITY.md#security-event-policy)) | 4 |
| SEC-062 | missing | SC-022 / Configuration | `config.loaded` (success, failure) at startup: `reason_code` on failure (`origin_missing` \| `admins_invalid` \| `db_path_missing` \| `origin_not_local` from SC-032, T-017), with no values. No security control can be changed at runtime. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-063 | not_applicable | NFR-1.2, ARCHITECTURE | No cloud control plane. Reopen when hosted. | — | — |
| SEC-064 | unavailable_signal | — | Host network config is outside the app boundary. | — | — |
| SEC-065 | unavailable_signal | — | Host-level. The app makes no outbound calls. | — | — |
| SEC-066 | unavailable_signal | SC-008 | The app spawns no processes. Process creation is host-level. | — | — |
| SEC-067 | unavailable_signal | — | Host-level. | — | — |
| SEC-068 | missing | SC-017, SC-024 / process entry | `system.started` (success, failure), `system.stopping`, `system.fault` (`reason_code=uncaught_exception`, no message). No fabricated shutdown event after a crash. | Operator ([policy](SECURITY.md#security-event-policy)) | 0 |
| SEC-069 | not_applicable | NFR-1.2 | No containers or Kubernetes. | — | — |
| SEC-070 | not_applicable | NFR-1.2 | No admission control. | — | — |
| SEC-071 | unavailable_signal | — | GitHub repository settings are provider-owned. | — | — |
| SEC-072 | unavailable_signal | ARCHITECTURE Toolchain | No CI/CD is specified. | — | — |
| SEC-073 | unavailable_signal | SC-023 | Toolchain checks, not runtime events. | — | — |
| SEC-074 | unavailable_signal | SC-023 | `npm ci` and `npm audit` results are build output, not app events. | — | — |
| SEC-075 to SEC-083 | not_applicable | REQUIREMENTS (no AI or agent features) | No models, agents, tools, or MCP. Reopen if any AI feature is added. (One summary row per ID: 075, 076, 077, 078, 079, 080, 081, 082, 083 are each not_applicable on this evidence.) | — | — |
| SEC-084 | unavailable_signal | — | The file is protected by mode `0600` ([policy](SECURITY.md#security-event-policy)). There is no app-level read audit, and file access is host-level. | — | — |
| SEC-085 | missing | SC-019 / logger setup | `system.started` includes `action=logger_config` and a constant `logger_profile` version (≤ 16 chars). The destination is fixed in config ([policy](SECURITY.md#security-event-policy)). | Operator ([policy](SECURITY.md#security-event-policy)) | 0 |
| SEC-086 | missing | Event helper | Bounded counters for `encode_failed`, `dropped`, `oversize_omitted`, and `suppressed` (volume ceiling, T-011), written as `logging.health` every 60 s and at shutdown with counts only. An independent silence monitor is the operator's file review ([policy](SECURITY.md#security-event-policy)). | Operator ([policy](SECURITY.md#security-event-policy)) | 0 |
| SEC-087 | unavailable_signal | — | Host clock. | — | — |
| SEC-088 | unavailable_signal | — | No detection rules exist. Retention is in [policy](SECURITY.md#security-event-policy). | — | — |
| SEC-089 | missing | SC-019 / event helper and serializers | No runtime event. The requirement is verified by tests: CR/LF, NUL, and oversize values in titles and URLs produce one parsable record; secret canaries (session, token, challenge) are absent from all output; `input.rejected` never echoes values. | Operator ([policy](SECURITY.md#security-event-policy)) | 0 |
| SEC-090 | missing | FR-7.4, FR-7.5 | Containment by an admin (deleting a user or link) is recorded by SEC-052 events (shared, no duplicates). There are no automated alerts or isolation. | Operator ([policy](SECURITY.md#security-event-policy)) | 1 |
| SEC-091 to SEC-099 | not_applicable | REQUIREMENTS (no AI features) | No model calls, prompts, retrieval, or guardrails. (091, 092, 093, 094, 095, 096, 097, 098, 099 are each not_applicable.) | — | — |
| SEC-APP-001 | missing | FR-3.1 to FR-3.5, SC-019, SC-020 / Redirect handler | **Absence requirement.** The redirect path emits no event and no request log that contains visitor data. The click count is the only record. Verified by a test that follows and misses codes and checks that no IP, UA, Referer, or header appears in any output. | Operator ([policy](SECURITY.md#security-event-policy)) | 0 |
| SEC-APP-002 | missing | FR-2.6, FR-2.7, FR-4.12 / Links code generation | `link.code_generation_failed` (failure): `reason_code=retries_exhausted`, `count` (attempts). Signals code-space pressure or a fault. | Operator ([policy](SECURITY.md#security-event-policy)) | 4 |

## Open Decisions

None open for events. OD-008 to OD-011 in `SECURITY.md` came from the threat model and are answered there as proposed defaults; OD-009 added `auth.passkey_removed` (SEC-008). OD-004 to OD-007 are answered in [SECURITY.md → Security Event Policy](SECURITY.md#security-event-policy). OD-001 (rate limits) is answered by SC-018, which feeds SEC-040.
