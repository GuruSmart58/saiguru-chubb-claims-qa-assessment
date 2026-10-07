# Hiring-panel walkthrough

## 15–20 minute presentation
1. **2 minutes — application reality.** Show package.json and architecture. The
   brief mentions Angular/no tests; this archive has React and unit tests. Explain
   why extending the actual stack is more valuable than replacing it.
2. **3 minutes — risk choices.** Privacy, amounts, state transitions, then reliable
   delivery and submission UX. Show TEST_STRATEGY.md. Explain deferred coverage.
3. **5 minutes — tests.** Run npm run test:qa, focused ClaimRiskRulesTest and
   message-routing test. Show real components/store and the independent transition
   matrix. Explain each test's oracle and what is mocked.
4. **4 minutes — real stack demo.** Start direct Kafka QA mode before presentation;
   run Playwright. Show claim submission, forbidden ownership access and status
   updates without a refresh. If this has not run locally, say so explicitly.
5. **3 minutes — findings.** Explain whitespace mismatch, overlapping store calls,
   CDC default discrepancy, corrected BFF port and build blocker. Distinguish
   reproduced failures from review concerns.
6. **2 minutes — AI and limits.** Show genuine commit sequence and journal. Explain
   a specific test you reviewed or rejected. Do not claim decisions that the
   assistant made as your own past experience.

## Likely panel questions
**Why React Testing Library rather than Angular Testing Library?** The supplied
source is React. It preserves the requested user-perspective testing principle.

**Why not only Playwright?** Domain checks make boundary failures cheap and exact;
component tests cover slow/error responses deterministically; E2E verifies actual
cookies, proxying, persistence and asynchronous events that mocks cannot prove.

**Why expected-failure tests?** They keep known defects executable and visible.
Vitest fails on an unexpected pass, so fixing the defect requires removing .fails.
These three checks must never be counted as working feature behaviour.

**Why not a generic helper for every endpoint?** ClaimsClient owns authentication,
CSRF and claim operations. Tests retain expected outcomes in their own assertions.
A universal helper that auto-accepts any 2xx/4xx would hide the business expectation.

**Does embedded Kafka prove the whole real-time feature?** No. It proves broker to
production listener. Routing integration proves recipient isolation with simulated
transport. Playwright verifies the actual stack and browser update separately.

**How do you handle asynchronous updates?** Wait for connection readiness, arm the
matching claim/status frame wait before mutation, then assert the visible row.
No arbitrary sleeps or reruns that turn a failure into a pass.

**Is a double-click actually broken?** The component correctly disables controls.
The reproduced defect is missing store-level concurrency protection; server
idempotency is still needed for retries and duplicated network submissions.

**Can the status event leak to another claimant?** Tests verify the unrelated
claimant receives no message. Actual HTTP JWT/role enforcement is separately
covered by real-stack API tests; route-cookie presence is only a UI guard.

## Next 10 minutes — priorities with more time
1. Execute all stack tests on the candidate's Docker host and resolve failures.
2. Fix whitespace and duplicate submission defects; remove expected-failure marks.
3. Broker outage/replay tests and durable outbox/idempotency design.
4. Concurrent admin updates with database optimistic locking and conflict tests.
5. CDC mode separately, Redis cache freshness and signup compensation failures.
6. Contract drift, accessibility keyboard flows, then measured load/visual coverage.

## Questions to ask
- Which claim defects have the greatest business or regulatory impact here?
- What guarantees are expected for event delivery and duplicate handling?
- How are component/API tests and slower stack tests used in CI today?

## Before submitting
Read every new test and helper, run the documented commands locally, complete the
candidate-review journal section, and update VALIDATION.md with your real results.
Never claim Docker execution based only on test discovery or mocked integration.
