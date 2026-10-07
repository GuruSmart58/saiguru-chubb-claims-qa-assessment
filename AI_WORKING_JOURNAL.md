# AI working journal

This is a record of this ChatGPT-assisted session, not a reconstructed prompt history.
The candidate must review every generated test before submission. Candidate sign-off
and personal challenge/override decisions are still pending; do not present the
assistant's decisions as your own past actions.

1. User supplied the CHUBB assessment and asked to build a project.
   Assistant proposed reviewing source before implementation and requested it.
2. User supplied demo-app-qa-main(1).zip.
   Assistant unpacked the source and compared it with the brief.
3. Initial Angular/no-tests assumption was overridden based on package.json and
   existing test directories. Retain React Testing Library / Vitest / JUnit.
4. Read domain, use cases, UI validation/store, auth, messaging and Compose.
   Accepted existing boundaries and transition rules as the source-based oracle.
   Challenged README's default Kafka mode using the actual Compose setting.
5. Baseline test run exposed absent generated clients and storage expectations.
   Prefer generating clients from the supplied OpenAPI schema; do not handwrite
   a fake API client or mock away a production import failure.
6. Chose real wizard/store behaviour tests rather than another mocked store
   suite. Document known failures explicitly; do not weaken assertions to pass.
7. Chose real-stack API/Playwright coverage with unique users and cookies.
   Broker/Compose execution depends on local prerequisites; record blocked tests
   honestly instead of claiming they ran.


8. Corrected provided theme tests to use actual sessionStorage. Challenged the
   existing rehydration test because it only parsed JSON; replaced it with real
   store rehydration assertions. Corrected generated-model test fixtures.
9. Type checking exposed an unsupported react-hot-toast onClick option. Removed
   the ineffective callback instead of adding an unsupported route. Fixed the
   local client's default BFF port and added a fetch-URL regression test.
10. Added direct-Kafka Compose override rather than changing the base CDC mode.
    Added real embedded-Kafka delivery to distinguish broker evidence from mocked
    transport routing. Added real UI/API/WS Playwright tests with no backend mocks.
11. Frontend tests, type checking and Next production build were run. Stack test
    discovery succeeded; actual full-stack execution is blocked without Docker.
    Java tooling was installed in scratch for verification, not added to the repo.

## Candidate review to complete
For each meaningful choice above: mark accepted / challenged / overridden and
briefly explain your own reasoning. Record additional prompts as you work locally.
Verify bug expectations, run Docker tests and explain the limits of mocks.

## Final backend validation decisions
- Compilation exposed the provided UpdateClaimStatusUseCaseTest constructor missing
  its ApplicationEventPublisher dependency. Added the collaborator rather than
  disabling the test. Creation tests now explicitly select CDC mode for assertions
  about retained events, and their names reflect what they actually verify.
- Full Claims suite exposed mocked ObjectMapper.createObjectNode returning null.
  The provided publisher tests stubbed serialization of the event, while production
  serializes an envelope. Reworked tests around real JSON construction and exact
  envelope/partition/routing fields; kept serialization and validation failure checks.
- Explicit Mockito javaagent was added to both Surefire configs. This removes the
  restricted environment's automatic-attachment failure and supports Java 21.
- Tests live in their relevant layer packages, consistent with existing architecture
  rules; no architecture rule was relaxed for the new tests.
- Completed runs: 289 frontend checks (three strict expected failures), 229 Claims
  tests, 213 BFF tests and five message/broker integration tests. Docker/browser
  execution is blocked at health preflight; candidate must run it before sign-off.
