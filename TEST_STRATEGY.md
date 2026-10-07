# Risk assessment — before adding tests

## Source reality
The supplied archive is Next.js 16 / React 19, Spring Boot 4 / Java 21, PostgreSQL,
Keycloak, Redis and Kafka. It already includes Vitest / Testing Library and JUnit
unit tests. The assessment prose says Angular and no tests; this submission follows
the actual source. Baseline imported unchanged in the first Git commit.

## Priorities
| Priority | Risk and impact | Planned evidence |
|---|---|---|
| P0 | A claimant sees another person's claim or performs admin changes | Real-stack API tests with distinct user identities; WebSocket recipient isolation |
| P0 | Invalid amounts, dates or transitions are accepted | Domain boundary and complete transition matrix tests; schema tests |
| P1 | A submission loses data, creates duplicates or fails silently | Component tests using the real wizard store, pending / failed HTTP responses |
| P1 | A persisted status never reaches its owner | Kafka consumer → real WebSocket handler integration and stack WebSocket test |
| P1 | UI and API validation disagree | Trimmed text boundary tests and executable defect reproductions |

Read: domain Claim / ClaimStatus; creation, ownership and status use cases;
HTTP controllers; Kafka publisher, transaction listener and consumer;
WebSocket handler / authentication; UI wizard schemas, store, components,
auth middleware and clients; Compose and startup scripts.

## Scope decisions
Extend existing tests rather than duplicating all of them. Use Testing Library
with React because this repository is not Angular. Keep a small Chromium stack
suite, use real login / signup and cookies, do not mock backend routes in E2E.
Use bounded assertions rather than sleeps and unique data rather than global resets.
One message-path integration uses real consumer + handler with simulated transport:
it verifies routing and payload. A separate embedded-Kafka integration verifies
actual broker delivery to the production annotated listener, with transport mocked.

Deliberately deferred: exhaustive cross-browser and visual coverage, Pact,
load testing, CDC connector integration, Kafka crash / replay recovery, Redis
invalidation timing and concurrent database updates. Financial correctness and
privacy come first. Broker-backed replay and atomic persistence/event delivery
are the next priorities. Load numbers without a controlled environment would
be misleading.

## Early findings
- Compose sets CDC true, despite README saying direct Kafka is default. No
  Debezium in normal startup: writes can succeed without real-time notifications.
- UI text schemas count whitespace; Java trims before checking lengths.
- Wizard store has no in-flight guard when submitClaim is called twice.
- No optimistic locking; Kafka sends after commit without a durable outbox.

See BUG_REPORT.md and VALIDATION.md for evidence and execution limits. No claim
of complete application soundness is made from passing unit tests.
