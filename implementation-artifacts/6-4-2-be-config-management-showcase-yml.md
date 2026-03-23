# Story 6.4.2: BE Config Management (showcase.yml)

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

**As the** platform owner,
**I want** a YAML config file to register demo apps for health polling,
**so that** I can add new demo apps by editing config and redeploying with no code changes.

---

## 1. Acceptance Criteria

**AC-1 — showcase.yml drives demo app polling automatically**

Given `src/main/resources/showcase.yml` exists,
When I add a new entry with `slug`, `name`, `healthEndpoint`, `demoUrl`,
Then Platform BE automatically starts polling that app's health endpoint on next restart.

**AC-2 — Fail-fast on malformed YAML**

Given the config file is malformed (invalid YAML),
When Platform BE starts,
Then it logs a clear error with the specific parse failure and refuses to start (fail-fast).

**AC-3 — Graceful degradation for unreachable endpoints**

Given the config file has a valid entry but the health endpoint is unreachable,
When Platform BE polls it,
Then status is recorded as `DOWN` in `project_health` table; no exception is thrown; polling continues for other apps.

**AC-4 — Remove polling by removing config entry**

Given a demo app entry is removed from `showcase.yml`,
When BE restarts,
Then polling stops for that app and no `project_health` record is created for it.

---

## 2. Tasks / Subtasks

### Task 1: Define showcase.yml schema and create config file — AC: #1, #4

- [ ] Create `src/main/resources/showcase.yml` with initial placeholder entries
- [ ] Define schema: `apps[].slug`, `apps[].name`, `apps[].healthEndpoint`, `apps[].demoUrl`
- [ ] Document the schema with comments in the YAML file
- [ ] Add at least one placeholder entry (e.g. `wallet-app` pointing to placeholder URL) so the schema is self-verifying
- [ ] Ensure file passes YAML linting with no trailing errors

**showcase.yml schema (exact field names):**
```yaml
apps:
  - slug: wallet-app                # unique identifier — must match project slug in FE config
    name: "Wallet App"               # display name for logging/admin
    healthEndpoint: "https://wallet.chinh.dev/health"  # BE polls this URL every 60s
    demoUrl: "https://wallet.chinh.dev"               # optional — used for admin display only
  # Add new app here → restart BE to activate polling
```

### Task 2: Create DemoAppRegistry config loader — AC: #1, #2

- [ ] Create `shared/config/DemoAppRegistry.java` in `portfolio-platform`
- [ ] Load `showcase.yml` on application startup using Spring's `YamlPropertiesFactoryBean` or `@ConfigurationProperties`
- [ ] Validate required fields (`slug`, `healthEndpoint`) on load — throw `BeanInitializationException` with clear message if missing
- [ ] Expose `List<DemoAppEntry> getApps()` — called by `MetricsAggregationService`
- [ ] Handle YAML parse failure gracefully: catch parse exception, log full error with file location, and throw `ApplicationContextInitializationException` to fail-fast
- [ ] Add unit tests: valid config loads, missing slug field throws, malformed YAML throws

### Task 3: Wire DemoAppRegistry into MetricsAggregationService — AC: #1, #3, #4

- [ ] Modify `MetricsAggregationService` to receive `DemoAppRegistry` as a dependency
- [ ] Replace hardcoded poll target list with `demoAppRegistry.getApps()` iteration
- [ ] For each app: call `healthEndpoint` via RestClient, record status
- [ ] On connection timeout or HTTP error: record status `DOWN`, log at WARN, continue to next app — no exception propagation
- [ ] On success: record status `UP`, record `uptime_percent`, `response_time_ms`, `last_online_at`
- [ ] If app removed from config → no record created on next poll cycle (orphan records handled by Story 3.5 staleness logic)
- [ ] Add unit tests: 2-app config polls both, unreachable app recorded as DOWN, valid app recorded as UP

### Task 4: Admin endpoint exposes registered apps — AC: #1

- [ ] Create `GET /api/v1/admin/showcase/apps` (Owner-only) returning all registered apps from `DemoAppRegistry`
- [ ] Response: `{ "apps": [{ "slug", "name", "healthEndpoint", "demoUrl" }] }`
- [ ] Owner can view which apps are registered for polling without checking config file

### Task 5: Integration test — AC: #1, #3

- [ ] Write integration test using WireMock: stub `/health` for 2 apps (1 UP, 1 DOWN)
- [ ] Verify: `ProjectHealth` records created for both apps with correct status
- [ ] Verify: DOWN app has `status = DOWN`, UP app has `status = UP`
- [ ] Use Testcontainers + WireMock for full integration context

### Task 6: Documentation — AC: all

- [ ] Add `showcase.yml` entry to `.env.example` with comment explaining the file
- [ ] Document adding a new app in `DECISIONS.md` (or `README.md`): "Edit `showcase.yml`, restart BE"
- [ ] Verify `spring-boot:run -Dspring-boot.run.profiles=dev` starts cleanly with default `showcase.yml`

---

## 3. Dev Notes

### Project Structure Notes

**Location:** `portfolio-platform/src/main/resources/showcase.yml` and `portfolio-platform/src/main/java/dev/chinh/portfolio/shared/config/DemoAppRegistry.java`

```
src/main/resources/
├── showcase.yml                  ← NEW: Demo app registry (edit to register apps)
└── db/migration/
    └── V4__create_project_health_table.sql   ← Already exists from Story 2.2

src/main/java/dev/chinh/portfolio/
├── shared/
│   ├── config/
│   │   ├── DemoAppRegistry.java  ← NEW: Config loader + validator
│   │   └── AppConfig.java        ← EXISTING: RestClient bean (already configured)
│   └── error/
│       └── GlobalExceptionHandler.java  ← EXISTING: single @ControllerAdvice
└── platform/
    └── metrics/
        ├── MetricsAggregationService.java  ← MODIFY: use DemoAppRegistry
        └── ProjectHealth.java               ← EXISTING: JPA entity

src/test/
└── java/dev/chinh/portfolio/
    └── shared/
        └── config/
            └── DemoAppRegistryTest.java     ← NEW: unit tests
```

### Key Implementation Details

**showcase.yml schema (exact, no deviation):**

```yaml
# Platform BE Demo App Registry
# Edit this file to register/remove demo apps for health polling.
# Restart BE after editing to apply changes.
# Malformed YAML → application refuses to start (fail-fast)
apps:
  - slug: wallet-app
    name: "Wallet App"
    healthEndpoint: "https://wallet.chinh.dev/health"
    demoUrl: "https://wallet.chinh.dev"
  # Uncomment and fill to add more apps:
  # - slug: new-app
  #   name: "New App"
  #   healthEndpoint: "https://newapp.chinh.dev/health"
  #   demoUrl: "https://newapp.chinh.dev"
```

**DemoAppRegistry.java:**
```java
// Uses Spring's @ConfigurationProperties for type-safe YAML binding
// Path: src/main/resources/showcase.yml (auto-loaded via classpath)

@Component
@ConfigurationProperties(prefix = "showcase")
@Validated
public class DemoAppRegistry {

    private List<DemoAppEntry> apps = new ArrayList<>();

    public List<DemoAppEntry> getApps() {
        return Collections.unmodifiableList(apps);
    }

    // Called by MetricsAggregationService — never null, may be empty

    public static class DemoAppEntry {
        @NotBlank(message = "slug is required")
        private String slug;

        @NotBlank(message = "name is required")
        private String name;

        @NotBlank(message = "healthEndpoint is required")
        private String healthEndpoint;

        private String demoUrl; // optional

        // getters, setters (or record if using Java 21)
    }
}
```

**application.yml additions:**
```yaml
# Load showcase.yml from classpath
spring:
  config:
    import: classpath:showcase.yml

showcase:
  apps:
    - slug: wallet-app
      name: "Wallet App"
      healthEndpoint: "https://wallet.chinh.dev/health"
      demoUrl: "https://wallet.chinh.dev"
```

**MetricsAggregationService wiring:**
```java
@Service
public class MetricsAggregationService {

    private final ProjectHealthRepository repository;
    private final DemoAppRegistry demoAppRegistry;   // ← NEW dependency
    private final RestClient restClient;              // ← from Story 2.1

    @Scheduled(fixedRate = 60_000)
    public void pollAllApps() {
        demoAppRegistry.getApps().forEach(app -> pollApp(app));  // ← replaced hardcoded list
    }

    private void pollApp(DemoAppRegistry.DemoAppEntry app) {
        try {
            HealthResponse health = restClient.get()
                .uri(app.getHealthEndpoint())
                .retrieve()
                .body(HealthResponse.class);
            // record UP
        } catch (Exception e) {
            // record DOWN — no exception propagation, log at WARN
            log.warn("Health check failed for app '{}': {}", app.getSlug(), e.getMessage());
        }
    }
}
```

**Admin endpoint response:**
```json
// GET /api/v1/admin/showcase/apps
{
  "apps": [
    {
      "slug": "wallet-app",
      "name": "Wallet App",
      "healthEndpoint": "https://wallet.chinh.dev/health",
      "demoUrl": "https://wallet.chinh.dev"
    }
  ]
}
```

### Cross-Story Dependencies

- **Story 2.1 (Initialize BE project):** REQUIRED — `AppConfig.java` with `RestClient` bean must exist before this story. RestClient is the HTTP client used for health endpoint polling.
- **Story 2.2 (Database schema):** REQUIRED — `ProjectHealth` JPA entity and `ProjectHealthRepository` must exist. This story writes to those entities.
- **Story 2.3 (OpenAPI/error handler):** REQUIRED — `GlobalExceptionHandler` must be in `shared/error/`. Structured error format (`{error: {code, message}}`) must be used if any API errors occur.
- **Story 3.1 (Health polling pipeline):** REQUIRED — `MetricsAggregationService` with `@Scheduled` annotation must exist. This story extends it to use `DemoAppRegistry` instead of a hardcoded list.
- **Story 6.4.1 (FE projects.ts):** PARALLEL — FE `projects.ts` slug values must match BE `showcase.yml` slug values. Coordinate with FE dev agent to ensure slugs align (e.g., `wallet-app` in both).
- **Story 6.4.3 (Smoke tests):** FUTURE — smoke test will verify that registered apps appear in `project_health` table and are broadcast via WebSocket.

### Architecture Compliance

- **RestClient (not WebClient):** Health polling uses `RestClient` bean from `shared/config/AppConfig.java` — do NOT use `WebClient`.
- **Package boundary:** `DemoAppRegistry` lives in `shared/config/` — NOT in `platform/metrics/`. MetricsAggregationService imports and calls it, but the registry itself is shared config.
- **Spring profiles:** `dev` and `prod` only — no other profile names.
- **One GlobalExceptionHandler:** Any exceptions thrown during config loading surface through Spring Boot's default behavior. Ensure fail-fast is triggered via `BeanDefinitionValidationException` or `BeanCreationException` wrapping — not a custom runtime exception.
- **No new database migration:** This story adds no new tables or columns. All state is written to existing `project_health` table (Story 2.2).

### Gotchas

- **Config file path:** `src/main/resources/showcase.yml` — loaded via Spring's config import mechanism. Do NOT hardcode a `File` path or use `new FileInputStream()`.
- **YAML fail-fast:** Spring Boot auto-configuration for YAML (`spring-boot-configuration-processor`) handles the parse. Ensure `showcase.yml` is referenced in `application.yml` via `spring.config.import: classpath:showcase.yml`. If the YAML is malformed, Spring throws `BeanCreationException` during startup — this is the desired fail-fast behavior.
- **slug vs projectId naming:** The `project_health` table uses `project_slug` as the column name. Ensure `DemoAppEntry.slug` maps to `ProjectHealth.projectSlug` when upserting. Use `projectSlug` (camelCase Java field → `project_slug` DB column via JPA `@Column(name = "project_slug")`).
- **Health endpoint response shape:** Story 3.1's AC specifies the health endpoint returns `{status, uptimePercent, responseTimeMs, lastDeploy}`. RestClient must deserialize this correctly. Define a `HealthEndpointResponse` record matching the expected JSON shape.
- **Polling isolation:** If one app's health endpoint returns a non-2xx status, do NOT throw an exception — handle as `DOWN`. One app failure must not affect polling of other apps.
- **DemoAppRegistry caching:** No caching needed — config is reloaded on BE restart. If hot-reload is desired in future, reload via `@RefreshScope`, but that is out of scope for this story.
- **WireMock for integration tests:** Use WireMock standalone (`wiremock:wiremock-jetty12`) via JUnit 5 `@TempDir` or Testcontainers. Do NOT use embedded WireMock JAR conflicts with Spring Boot version.
- **Test isolation:** `DemoAppRegistryTest` must not depend on Spring context — use plain unit test with Jackson `ObjectMapper` to parse YAML directly. Spring context tests go in `platform/metrics/` integration suite.

### References

- [Source: _bmad-output/planning-artifacts/epics.md → Epic 6 → Story 6.4.2 ACs]
- [Source: _bmad-output/planning-artifacts/epics.md → Epic 3 → Story 3.1 Health Polling Pipeline — RestClient pattern, @Scheduled 60s]
- [Source: _bmad-output/planning-artifacts/architecture.md → Backend Architecture → Code Organization → shared/config/ DemoAppRegistry placeholder]
- [Source: _bmad-output/planning-artifacts/architecture.md → Backend Architecture → Spring Boot 3.4.x — RestClient usage pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md → Project Structure → portfolio-platform → demo-apps.yml schema]
- [Source: _bmad-output/planning-artifacts/architecture.md → Implementation Patterns → Spring Boot Validation — @Validated, BeanValidation]
- [Source: _bmad-output/implementation-artifacts/2-2-database-schema-core-entities.md → project_health table schema]
- [Source: _bmad-output/implementation-artifacts/6-4-1-fe-config-management-projects-ts.md → slug alignment note with FE config]
- [Source: _bmad-output/planning-artifacts/architecture.md → Integration Points → Metrics Pipeline → data flow]

---

## Dev Agent Record

### Agent Model Used

Claude Sonnet 4.6 (Anthropic) — 2026-03-19

### Debug Log References

### Completion Notes List

### File List

**New files:**
- `portfolio-platform/src/main/resources/showcase.yml` — demo app registry config
- `portfolio-platform/src/main/java/dev/chinh/portfolio/shared/config/DemoAppRegistry.java` — config loader + validator
- `portfolio-platform/src/test/java/dev/chinh/portfolio/shared/config/DemoAppRegistryTest.java` — unit tests

**Modified files:**
- `portfolio-platform/src/main/resources/application.yml` — add `spring.config.import: classpath:showcase.yml`
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/metrics/MetricsAggregationService.java` — use DemoAppRegistry
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/admin/AdminShowcaseController.java` — GET /api/v1/admin/showcase/apps

**Test files:**
- `portfolio-platform/src/test/java/dev/chinh/portfolio/platform/metrics/MetricsAggregationServiceTest.java` — update for DemoAppRegistry
- `portfolio-platform/src/test/java/dev/chinh/portfolio/platform/metrics/MetricsAggregationIntegrationTest.java` — WireMock integration test

---

## 5. Technical Compliance Checklist

Before marking story done, verify:

- [ ] `showcase.yml` exists at `src/main/resources/showcase.yml` with valid YAML and at least one entry
- [ ] `DemoAppRegistry.java` loads config on startup — `getApps()` returns all registered apps
- [ ] Malformed `showcase.yml` causes application startup failure with clear error message
- [ ] Missing required field (`slug` or `healthEndpoint`) throws validation error on startup
- [ ] `MetricsAggregationService` iterates over `demoAppRegistry.getApps()` — no hardcoded list
- [ ] Unreachable health endpoint → status recorded as `DOWN`, no exception thrown, polling continues for other apps
- [ ] Reachables health endpoint → status recorded as `UP` with `uptime_percent`, `response_time_ms`
- [ ] `GET /api/v1/admin/showcase/apps` returns all registered apps (Owner-only auth)
- [ ] `DemoAppRegistryTest` covers: valid config, missing slug, missing healthEndpoint, malformed YAML
- [ ] WireMock integration test: 2-app config → 1 UP, 1 DOWN → correct `project_health` records
- [ ] Adding new entry to `showcase.yml` + restart → polling starts for new app
- [ ] Removing entry from `showcase.yml` + restart → polling stops (no orphan records created)
- [ ] `slug` values in `showcase.yml` match `slug` values in `portfolio-fe/src/config/projects.ts` for the same apps
- [ ] Spring profiles: `dev` and `prod` only — no other profile names in any modified config
- [ ] `mvn test` passes for all modified/added classes
- [ ] OpenAPI spec (`/api-docs`) includes new admin endpoint
- [ ] Structured error format used if any HTTP errors occur in new endpoints
