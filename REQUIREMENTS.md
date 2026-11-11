# shrink-182 — Requirements

This file owns WHAT the system must do. Requirement IDs are permanent and are never renumbered or reused.

> **Note on answers:** Every answer once marked as a proposed default was confirmed by the owner on 2026-11-11 and is now a decision.

## Required Requirement Inputs

- **Project purpose:** A small link shortener for a team.
- **Users and actors:** Anonymous visitors (can sign up and follow short links); users (own and manage links); admins (oversee all links and users).
- **Core workflows:** Sign up with passkey; log in with passkey; register more passkeys; log out; create a short link from a long URL with an optional title; follow a short link to its target; list, edit, and delete own links with click counts; manage API tokens; delete own account; admin review and removal of links and users.
- **Data entities:** User (with role); passkey credential; session; API token; short link (short code, target URL, optional title, owner, click count).
- **External integrations:** None.
- **Authentication and roles:** Passkey sign-up and login; personal API tokens for the JSON API. Roles: user, admin.
- **Regulatory or privacy constraints:** No visitor-identifying data is stored when a link is followed (see FR-3.5). No other constraints stated.
- **Target surfaces (web UI, APIs, background jobs):** Web UI and JSON API. No background jobs.

## Functional Requirements

### Account registration and login

- **FR-1.1** The system MUST allow a visitor to create a user account by registering a passkey.
- **FR-1.2** The system MUST allow a registered user to log in by authenticating with their passkey.
- **FR-1.3** The system MUST reject a login attempt whose passkey authentication fails, and MUST NOT start a logged-in session for it.
- **FR-1.4** The system MUST NOT offer password-based sign-up or login.
- **FR-1.5** The system MUST allow anyone who reaches the site to sign up, without an invite or any email-domain restriction.
- **FR-1.6** The system MUST allow a logged-in user to register additional passkeys on their account.
- **FR-1.7** The system MUST allow a user to log in with any passkey registered on their account.
- **FR-1.8** The system MUST NOT provide any account recovery path. A user who has lost access to all of their passkeys MUST NOT be able to regain that account and must sign up again as a new user.
- **FR-1.9** The system MUST allow a logged-in user to log out, after which their session MUST no longer be accepted.
- **FR-1.10** The system MUST end a logged-in session after 24 hours without activity, and MUST end any session 7 days after login regardless of activity.
- **FR-1.11** The system MUST allow a logged-in user to delete their own account. Deleting an account MUST also delete all links that user owns, and those short links MUST then behave as not found (FR-3.3). An admin is the one exception (FR-7.10).
- **FR-1.12** Deleting an account MUST be all-or-nothing. If any part fails, the account and its links, API tokens, and sessions MUST stay as they were. After a successful deletion, none of them may work. *(T-014)*
- **FR-1.13** The system MUST show a logged-in user a list of the passkeys on their account, with the date each was added and the date it was last used. *(resolves OD-009, T-015)*
- **FR-1.14** The system MUST allow a logged-in user to remove a passkey from their account, after which it MUST no longer be accepted for login. The system MUST refuse to remove the last remaining passkey, because there is no recovery path (FR-1.8). *(resolves OD-009, T-015)*
- **FR-1.15** Before a user adds a passkey, removes a passkey, creates an API token, or deletes their own account, the system MUST require that the user authenticated with one of their passkeys within the last 5 minutes, and MUST ask them to do so again if not. The 5-minute window belongs to the session, not to the browser. *(resolves OD-009, T-015; G-021)*

### Creating short links

- **FR-2.1** The system MUST allow a logged-in user to create a short link by submitting a long target URL.
- **FR-2.2** The system MUST allow the user to include an optional title when creating a short link.
- **FR-2.3** On successful creation, the system MUST return the new short link to the user, and MUST record that user as the link's owner.
- **FR-2.4** The system MUST reject a create request that has no target URL, and MUST tell the user why.
- **FR-2.5** The system MUST reject a create request from a user who is not logged in.
- **FR-2.6** Each short link MUST be unique; two links MUST NOT share the same short link.
- **FR-2.7** The system MUST generate the short code itself as 7 random letters and digits (A–Z, a–z, 0–9). Users MUST NOT be able to choose a custom short code.
- **FR-2.8** The system MUST accept only target URLs that start with `http://` or `https://` and are well-formed, and MUST reject any other target URL, telling the user why.
- **FR-2.9** The system MUST reject a target URL longer than 2,048 characters, telling the user why.
- **FR-2.10** The system MUST reject a target URL that points at shrink-182's own domain, telling the user why. This covers every name that reaches the shrink-182 server itself, not only the configured one. *(widened by T-010)*
- **FR-2.11** The system MUST reject a title longer than 200 characters, telling the user why.
- **FR-2.12** The system MUST reject a target URL that contains a username or password, telling the user why. *(T-008)*
- **FR-2.13** The system MUST reject a title that contains control characters or text-direction override characters, telling the user why. *(T-008)*
- **FR-2.14** The system MUST store and show a target URL in its normalized form, so the host a user sees is the host the redirect goes to. *(T-008)*

### Following short links

- **FR-3.1** When anyone requests an existing short link, the system MUST redirect them to that link's target URL.
- **FR-3.2** The system MUST NOT require a visitor to be logged in to follow a short link.
- **FR-3.3** When a short link is requested that does not exist, the system MUST NOT redirect, and MUST respond with a not-found response.
- **FR-3.4** The system MUST increase a link's click count when that short link is followed.
- **FR-3.5** The system MUST NOT store the visitor's IP address, browser details, or any other data that identifies a visitor when a short link is followed. Only the click count is recorded.
- **FR-3.6** The redirect MUST be temporary (HTTP 302), not permanent, so that browsers do not cache it and every visit reaches the system.
- **FR-3.7** Every request for an existing short link MUST count as one click; repeat visits and automated clients MUST NOT be filtered out.
- **FR-3.8** Short links MUST NOT expire. A link MUST remain active until it is deleted.

### Managing own links

- **FR-4.1** The system MUST allow a logged-in user to list the short links they own.
- **FR-4.2** The list MUST show, for each link, its short link, target URL, title (if any), and click count.
- **FR-4.3** The list MUST NOT include links owned by other users.
- **FR-4.4** The system MUST allow a logged-in user to edit a link they own.
- **FR-4.5** The system MUST allow a logged-in user to delete a link they own.
- **FR-4.6** The system MUST reject any attempt by a user to view the details of, edit, or delete a link they do not own, and MUST leave that link unchanged. Admin powers in FR-7 are the only exception.
- **FR-4.7** The system MUST reject list, edit, and delete requests from a user who is not logged in.
- **FR-4.8** After a link is deleted, it MUST no longer appear in its owner's list.
- **FR-4.9** When editing a link, the owner MUST be able to change the title and the target URL. The short code MUST NOT be changeable.
- **FR-4.10** An edited target URL or title MUST pass the same validation as on creation (FR-2.8 to FR-2.14).
- **FR-4.11** Editing a link MUST NOT reset its click count.
- **FR-4.12** After a link is deleted, its short link MUST respond as not found, and its short code MUST NOT ever be assigned to a new link.
- **FR-4.13** When a user asks to view, edit, or delete a link that does not exist or that they do not own, the system MUST give the same not-found response in both cases, so it never confirms that another user's link exists. *(moved from a DESIGN.md assumption)*

### Web UI and JSON API

- **FR-5.1** The system MUST provide a web UI for sign-up, login, creating links, listing own links, editing own links, and deleting own links.
- **FR-5.2** The system MUST provide a JSON API for creating links, listing own links, editing own links, and deleting own links.
- **FR-5.3** The JSON API MUST apply the same validation and ownership rules as the web UI.
- **FR-5.4** The JSON API MUST return failures as JSON responses that say what went wrong.
- **FR-5.5** The system MUST allow a logged-in user to create personal API tokens in the web UI. The token value MUST be shown only once, at creation.
- **FR-5.6** The system MUST allow a user to list and revoke their own API tokens. A revoked token MUST no longer be accepted.
- **FR-5.7** The JSON API MUST identify the caller by API token and MUST act as the token's owner. Requests with a missing, invalid, or revoked token MUST be rejected.
- **FR-5.8** Deleting a user account MUST revoke all of that user's API tokens.
- **FR-5.9** An API token MUST be accepted only for creating, listing, editing, and deleting the token owner's links (FR-5.2). Account, passkey, API-token, and admin actions MUST require a logged-in session and MUST reject an API token. *(T-005)*

### Out of scope

- **FR-6.1** The system MUST NOT take payments.
- **FR-6.2** The system MUST NOT accept file uploads.

### Roles and admin

- **FR-7.1** The system MUST support exactly two roles: user and admin. Every new sign-up MUST get the user role.
- **FR-7.2** The admin role MUST be assigned only by the operator in the deployment's configuration. The web UI and the JSON API MUST NOT let anyone grant or remove the admin role.
- **FR-7.3** The system MUST allow an admin to list all links from all users, with each link's owner and click count.
- **FR-7.4** The system MUST allow an admin to delete any link. An admin MUST NOT be able to edit another user's link.
- **FR-7.5** The system MUST allow an admin to list all user accounts and delete any user account other than their own. The deletion follows FR-1.11 and FR-5.8.
- **FR-7.6** The system MUST reject admin actions from anyone who does not have the admin role.
- **FR-7.7** The system MUST show a logged-in user their own internal user ID on their account page, so the operator can copy it into the admin configuration (FR-7.2). *(moved from ARCHITECTURE.md AQ-5)*
- **FR-7.8** Signed-in screens MUST identify the user only as "Signed in". The admin user list MUST show each user's internal user ID and sign-up date. Users have no display name or email. *(resolves DQ-1)*
- **FR-7.9** An internal user ID MUST NOT be guessable and MUST NOT ever be given to another account, including after the account is deleted. *(T-004)*
- **FR-7.10** The system MUST reject an admin's attempt to delete their own account, from the account page as well as from the admin screens, for as long as their ID is in the admin configuration. The operator removes the ID first (FR-7.2); the person can then delete the account as an ordinary user. *(resolves OD-011, T-004)*

## Non-Functional Requirements

- **NFR-1.1** The system MUST support up to 100 registered users.
- **NFR-1.2** The system MUST run as a single deployment.
- **NFR-1.3** The system MUST limit sign-up and login to 10 attempts per minute from one network address, and JSON API calls to 60 per minute per API token. JSON API calls made with a valid session cookie MUST be limited to 60 per minute per session (G-007). JSON API calls that carry no valid API token or session MUST be limited to 60 per minute from one network address (T-011). Passkey re-verification (FR-1.15) counts as login and shares the 10-per-minute limit for one network address (G-021). A request over the limit MUST be refused with HTTP 429 and a JSON or on-screen message saying to wait and try again. Following short links MUST NOT be rate-limited. Sign-ups are not capped at a fixed number of accounts. *(resolves OQ-15; G-007 and G-021 answered 2026-11-11)*
- **NFR-1.4** The system MUST run only on a developer machine at `http://localhost` and MUST refuse to start with any other site origin. Making it reachable by other people is out of scope and needs these specifications and the threat model to be revised first. *(resolves OD-008, T-009, T-017)*
- **NFR-1.5** Unlimited short-link traffic (NFR-1.3, FR-3.7) is accepted for the localhost demo, including the risk that a flood of requests slows the service. *(resolves OD-010, T-012)*

## Open Questions

None open here. The decisions raised by the threat model (`THREAT_MODEL.md`), OD-008 to OD-011, were answered: NFR-1.4, FR-1.13 to FR-1.15, NFR-1.5, and FR-7.10. OQ-15 was resolved by NFR-1.3; no uptime or redirect-speed targets are set for the demo.
