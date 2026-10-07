# QA assessment — start here

This is the supplied claims application with focused QA additions. It is React /
Next.js, not Angular. See TEST_STRATEGY.md, BUG_REPORT.md, AI_WORKING_JOURNAL.md,
VALIDATION.md and WALKTHROUGH.md. The archive includes `.git` and real staged
commits from this session. No remote repository has been created.

## Prerequisites
Java 21, Maven 3.9+, Node 20+ (22/24 also tested for local frontend), Docker with
Compose. Run commands from repository root unless indicated. Git Bash or WSL is
recommended on Windows for the supplied shell scripts.

## Install and generate UI client
```bash
cd code/demo-app-ui
npm ci
npm run generate:api
npm test
npm run test:qa
npx tsc --noEmit
cd ../..
```
The generated client is excluded from Git and derived from the provided OpenAPI
schema with a pinned generator. Generate before tests or starting the UI.
Three QA tests use Vitest `it.fails`: they reproduce BUG-001 (two assertions) and
BUG-002 (one). They are not skipped and fail if the defect stops reproducing.
After fixing a defect, remove `.fails` and keep the original expected assertion.
Do not describe them as passing feature behaviour.

## Backend tests
```bash
mvn -f code/claims-service/pom.xml test
mvn -f code/bff-service/pom.xml test

# Focused deterministic domain checks
mvn -f code/claims-service/pom.xml test -Dtest=ClaimRiskRulesTest

# Real event parsing + WebSocket recipient routing, simulated transport
mvn -f code/bff-service/pom.xml test -Dtest=ClaimMessageRoutingIntegrationTest -Dsurefire.excludedGroups=

# Real embedded Kafka broker + production annotated listener
mvn -f code/bff-service/pom.xml test -Dtest=ClaimKafkaDeliveryIntegrationTest -Dsurefire.excludedGroups=
```
Integration tag is excluded by default in the supplied Maven config; the explicit
empty excludedGroups property is necessary. Embedded Kafka does not need Docker.
Other supplied Spring context integration tests use Testcontainers and need Docker.

## Start actual stack and run Playwright
```bash
./qa/start-stack.sh

cd code/demo-app-ui
npx playwright install chromium
npm run test:e2e:list
npm run test:e2e
npx playwright show-report
```
qa/start-stack.sh installs/generates the UI client, starts infrastructure, packages
both services (test execution is separate), and starts the base Compose stack with
the explicit direct-Kafka override. The UI runs in dev mode. Run tests separately
as above. The generated client is copied into the UI image.

If preferred, build the two JARs with Maven and use:
```bash
docker compose -f docker-compose.apps.yml -f docker-compose.qa.yml up --build -d
```
Use the existing README for health checks and safe stop commands. Do not start
Debezium alongside the direct mode for this suite.

Default URLs: UI http://localhost:3001, BFF http://localhost:8090.
Optional environment variables: QA_UI_URL, QA_BFF_URL, QA_ADMIN_EMAIL,
QA_ADMIN_PASSWORD. Set environment variables in your shell before Playwright;
no dotenv dependency is used. Never commit credentials or browser storage state.
The demo seeded admin defaults are intended only for the disposable local stack.

## What Playwright proves
11 tests: owner persistence/list isolation, forbidden cross-owner detail access,
claimant/admin role enforcement, legal lifecycle and rejected transitions, three
amount boundaries, CSRF enforcement, anonymous API denial, real UI submission,
real Kafka→WebSocket→UI status change, and protected-route redirect.
There are no backend mocks. Fresh claimant identities isolate tests. No arbitrary
sleeps, no hidden retries and no shared destructive database reset.
Trace/screenshots/JUnit/HTML reports are retained on failure. Traces may include
cookies and claim data; keep these local and redact before sharing.

Users and claims remain because the supplied app has no delete API. Clean the
local disposable stack only when you intend to wipe its data. The source archive
is preserved separately; do not confuse this deliverable with the original.
