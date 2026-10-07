# Findings

## BUG-001 — UI and backend disagree on whitespace (medium, reproduced)
UI schemas accept ten spaces as a description and five spaces as a location.
Java Claim trims before enforcing lengths and rejects both. A claimant can
advance through the wizard with values the backend will reject.

Evidence: tests/qa/claim-schema.test.ts has two strict `it.fails` checks;
ClaimRiskRulesTest verifies Java's trimmed rule. Reproduce: enter a past date,
five spaces for location, a valid amount, then ten spaces for description.
Expected: Next disabled / meaningful field validation. Actual: UI schema accepts.
Suggested fix: trim before length validation and align client/server rules.
Production rule is left unchanged to preserve the assessment defect evidence.

## BUG-002 — Submission store lacks an in-flight guard (medium, reproduced at store boundary)
Calling submitClaim twice before its first request resolves issues two API calls.
No idempotency key appears in the request or domain creation path. The server
creates a fresh UUID on every accepted request. This is a duplicate-request risk,
not proof that a normal browser double-click bypasses the disabled button.

Evidence: claim-submission-concurrency.test.ts strict `it.fails` expects one call
and receives two. Wizard behaviour test separately confirms disabled UI controls.
Suggested fix: in-flight guard plus server idempotency; do not rely only on a button.

## BUG-003 — Default Compose suppresses direct events without CDC (high, code review)
README says direct Kafka is default; docker-compose.apps.yml sets
APP_EVENTS_CDC_ENABLED=true. CreateClaimUseCase and UpdateClaimStatusUseCase
then rely on Debezium. Normal start-infra/start-app does not start Debezium.
Expected: normal startup provides live updates. Likely actual: writes persist but
notifications are absent. Not reproduced against Docker here.

Mitigation: docker-compose.qa.yml explicitly sets false for the direct event mode.
The base application file is unchanged. Full-stack live-update test verifies this
path locally. CDC mode requires its own connector-backed test.

## BUG-004 — UI defaults API calls to the wrong service (high, fixed)
The client fell back to localhost:8080 (Claims) while BFF runs on 8090. Login uses
Next rewrites to BFF but generated claim calls used the wrong default for local dev.
Changed fallback to 8090; WireMock mode keeps its explicit 8080 default.
Evidence: bff-base-url.test.ts tests the actual generated client's fetch URL.

## BUG-005 — Unsupported toast callback blocks TypeScript build (medium, fixed)
use-claim-realtime-handler passed onClick to react-hot-toast options, which do not
support that property. It also navigated to /claims/{id}, for which no route exists.
Removed the ineffective callback and its unused router dependency. Claimant
notifications remain informational; View Details already opens a modal.
Evidence: original TypeScript output and successful type check after repair.

## Existing test defects corrected
Theme assertions targeted localStorage though production uses sessionStorage.
Rehydration test merely parsed stored JSON; it now invokes persist.rehydrate and
asserts store state. Existing TypeScript test fixtures used `id` instead of
`userId`, string dates instead of generated Date models, and missing hook imports.
These are test defects, not evidence of a broken application feature.

## Concerns requiring further validation
- No optimistic locking: concurrent admin updates can overwrite each other.
- Kafka publish occurs after transaction commit and does not observe the send
  future; there is no durable outbox. Broker failure can lose a notification.
- The consumer logs malformed JSON and returns; retry / dead-letter behaviour
  is not demonstrated. Replayed event IDs are not explicitly deduplicated.
- UI's UTC-derived date max may differ from a claimant's local calendar near
  midnight. Test across timezones before deciding the date policy.
- Signup spans Keycloak and Claims without a visible compensation path.
- Test users/claims remain in local DB because no delete API exists; use a
  disposable assessment stack and do not point this suite at shared production.

Severity reflects potential impact; review findings are not claimed runtime bugs.

Additional baseline test repairs: UpdateClaimStatusUseCaseTest used a three-argument
constructor after production added ApplicationEventPublisher. Creation tests did
not choose a CDC mode explicitly. Kafka publisher tests mocked serialization of
the event object while production constructs an envelope; the mocked JSON tree
returned null. Reworked those tests with real JSON and exact payload assertions.
No production event implementation was changed to accommodate stale tests.
