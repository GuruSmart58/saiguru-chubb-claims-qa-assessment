# Validation record

Executed during this assistant session on 7 October 2026. Actual wall-clock effort
was far below the assessment hard cap. Commands below are portable equivalents of
the local runs; temporary Java 21 / Maven tooling and proxy configuration were
used outside the repository. There is no fabricated Docker execution.

| Check | Result | Evidence |
|---|---|---|
| Supplied frontend baseline after client generation | 264 passed; 2 theme-storage assertions failed | frontend-baseline.log |
| Final frontend `npm test` | 289 checks: 286 normal passes, 3 strict expected failures | frontend-final.log |
| Focused frontend `npm run test:qa` | 23 checks: 20 normal passes, 3 strict expected failures | frontend-qa.log |
| TypeScript `npx tsc --noEmit` | Pass | typescript.log |
| Next production `npm run build` | Pass | frontend-build.log |
| Claims default test group | 229 passed, no failures/errors/skips | claims-tests.log |
| Added Claims domain matrix/boundaries | 33 passed (included in 229 above) | claims-domain.log |
| BFF default test group | 213 passed, no failures/errors/skips | bff-tests.log |
| Added routing + embedded Kafka integration | 5 passed, no failures/errors/skips | bff-integration.log |
| Playwright `--list` | 11 tests discovered in 2 files | playwright-list.log |
| Full-stack Playwright execution | Blocked: BFF health ECONNREFUSED; Docker unavailable | playwright-preflight.log |
| Claims Checkstyle | Pass; supplied warning-level style messages remain | claims-checkstyle.log |
| Shell syntax / Git whitespace | Pass | `bash -n qa/start-stack.sh`, `git diff --check` |

A total of **733 normal checks passed**, plus **3 explicitly expected defect
reproductions**. The 33 focused domain checks are not counted a second time.
Passing checks are not a claim of exhaustive coverage or whole-stack soundness.

## Exact backend suite scope
Claims: `mvn -f code/claims-service/pom.xml test -Dcheckstyle.skip -Dpmd.skip`.
BFF: `mvn -f code/bff-service/pom.xml test`.
Added integration: `mvn -f code/bff-service/pom.xml test
-Dtest=ClaimMessageRoutingIntegrationTest,ClaimKafkaDeliveryIntegrationTest
-Dsurefire.excludedGroups= -Dcheckstyle.skip -Dpmd.skip`.

Default Maven excludes the integration tag. The 213 BFF checks include its
provided default-selected tests; this is not a count of only isolated unit tests.
The embedded Kafka test exercises a real broker and production listener, with
WebSocket transport mocked. The routing test uses real consumer and handler with
mock sessions. No PostgreSQL/Testcontainers or actual browser/backend run is
claimed. Static analysis `verify`, load and visual coverage were not completed.

## Remaining before submission
Run qa/start-stack.sh and all 11 Playwright tests on a Docker host. Retain reports,
review any failures, update this file with actual results, and complete candidate
sign-off in the AI journal. The project is prepared for that validation; it is
not represented as fully submission-verified until that run is done.
