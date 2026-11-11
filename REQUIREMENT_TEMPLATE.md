# Requirement Template - Web Applications and APIs

Use one copy of this template for each atomic requirement.

Replace bracketed placeholders. Use `N/A` only for a field you evaluated and found not applicable, and `TO BE DECIDED` for an unresolved answer. Uppercase normative terms follow BCP 14 (RFC 2119 and RFC 8174). Split compound behavior into separate requirements.

## Metadata

- **ID**: [REQ-MODULE-###, e.g., REQ-AUTH-010]
- **Title**: [Concise requirement name]

## Requirement

- **Statement**: [One atomic, unambiguous, independently testable statement of what the system MUST, MUST NOT, SHOULD, SHOULD NOT, or MAY do.]
- **Traceability**: [`REQUIREMENTS.md` IDs; `DESIGN.md` section or `N/A`; `ARCHITECTURE.md` component, boundary, flow, or `DR-*`; `SECURITY.md` `SC-*` IDs or `N/A`; `SECURITY_EVENTS.md` `SEC-*` IDs or `N/A`; `THREAT_MODEL.md` `T-*` IDs or `N/A`]

Keep project context in the linked specs rather than repeating it here.

## Acceptance Criteria

Each criterion describes one observable pass/fail outcome, can be tested on its own, stays within this requirement, and adds no new behavior. Use Given/When/Then.

1. **AC-01 — Expected behavior**: Given [valid preconditions], when [action], then [observable result].
2. **AC-02 — Boundary or failure**: Given [boundary, invalid input, unauthorized action, or dependency failure], when [action], then [safe, observable result with no unauthorized state change or sensitive side effect].
3. **AC-03 — Prohibited behavior**: Given [preconditions], when [action], then [what MUST NOT happen].
4. **AC-04 — Additional criterion**: [Only when needed. Remove otherwise.]

## Failure Behavior

- **Failure Responses**: [Applicable invalid-input, authentication, authorization, dependency, and system failures: response, disclosure, state changes, and recovery. Deny by default on security-decision failure unless a documented risk decision says otherwise.]
- **Logging / Audit**: [Relevant event contracts from `SECURITY_EVENTS.md`, or `N/A`]

## Test Strategy

- **Tests**: [Unit, integration, and security checks covering each `AC-*`, including positive and negative tests for every security-critical decision and error path]
- **UI Checks**: [Screens and states to screenshot against `DESIGN.md`, at which widths, or `N/A`]
- **Required Test Environment**: [Fixtures, identities, roles, simulators, configuration, or data]

## Dependencies

- **Upstream Requirements**: [IDs that MUST be satisfied first, or `None`]
- **External Dependencies**: [Third-party services, identity providers, APIs, or `None`]
- **Open Decisions**: [Unresolved decisions that block approval or implementation, or `None`]
