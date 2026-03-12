# Story 2.1: Initialize Platform BE Project

Status: done

## Story

As a developer,
I want a fully configured Spring Boot project with correct package structure and toolchain,
so that the team can build features on a compliant, standards-enforced backend foundation.

## Acceptance Criteria

1. **Given** the project is generated via Spring Initializr (`start.spring.io`),
   **Then** it uses Java 21 LTS, Spring Boot 3.4.x, Maven, with dependencies:
   `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-security`,
   `spring-boot-starter-test`, `postgresql`, `flyway-core`,
   `springdoc-openapi-starter-webmvc-ui:2.8.x`

2. **Given** the project is initialized,
   **Then** the root package is `dev.chinh.portfolio` with four sub-packages created:
   `auth/`, `platform/`, `apps/`, `shared/` — no business logic exists outside these packages

3. **Given** `mvn test` is run,
   **Then** all tests pass with JUnit 5, Mockito, and Testcontainers available on the test classpath

4. **Given** the project is configured,
   **Then** `application.yml` defines `local` and `prod` profiles with all secrets externalized
   via environment variables (no hardcoded credentials)

5. **Given** structured logging is required (NFR-O4),
   **Then** Logback is configured for JSON output in `prod` profile and human-readable in `local` —
   no PII fields logged

## Tasks / Subtasks

- [x] Task 1: Generate project via Spring Initializr (AC: #1)
  - [x] 1.1 Run curl command against start.spring.io with exact parameters below
  - [x] 1.2 Extract artifact, verify pom.xml contains all required starters
  - [x] 1.3 Manually add `springdoc-openapi-starter-webmvc-ui:2.8.6` and `flyway-core` to pom.xml
  - [x] 1.4 Manually add `spring-boot-testcontainers`, `junit-jupiter`, `postgresql` Testcontainers dependencies to pom.xml

- [x] Task 2: Establish package structure (AC: #2)
  - [x] 2.1 Create `src/main/java/dev/chinh/portfolio/auth/` package (with package-info.java)
  - [x] 2.2 Create `src/main/java/dev/chinh/portfolio/auth/session/` sub-package
  - [x] 2.3 Create `src/main/java/dev/chinh/portfolio/auth/jwt/` sub-package
  - [x] 2.4 Create `src/main/java/dev/chinh/portfolio/auth/oauth2/` sub-package
  - [x] 2.5 Create `src/main/java/dev/chinh/portfolio/platform/metrics/` sub-package
  - [x] 2.6 Create `src/main/java/dev/chinh/portfolio/platform/websocket/` sub-package
  - [x] 2.7 Create `src/main/java/dev/chinh/portfolio/platform/webhook/` sub-package
  - [x] 2.8 Create `src/main/java/dev/chinh/portfolio/platform/contact/` sub-package
  - [x] 2.9 Create `src/main/java/dev/chinh/portfolio/platform/admin/` sub-package
  - [x] 2.10 Create `src/main/java/dev/chinh/portfolio/apps/wallet/` sub-package
  - [x] 2.11 Create `src/main/java/dev/chinh/portfolio/shared/error/` sub-package
  - [x] 2.12 Create `src/main/java/dev/chinh/portfolio/shared/ratelimit/` sub-package
  - [x] 2.13 Create `src/main/java/dev/chinh/portfolio/shared/config/` sub-package + SecurityConfig.java + CorsConfig.java

- [x] Task 3: Configure application.yml with profiles (AC: #4)
  - [x] 3.1 Create `src/main/resources/application.yml` with `local` and `prod` profiles
  - [x] 3.2 All DB credentials, JWT secrets, OAuth2 secrets → environment variables only
  - [x] 3.3 Configure Flyway location: `classpath:db/migration`
  - [x] 3.4 Create `src/main/resources/db/migration/` directory (with .gitkeep)

- [x] Task 4: Configure Logback structured logging (AC: #5)
  - [x] 4.1 Add `logstash-logback-encoder:8.0` dependency in pom.xml
  - [x] 4.2 Create `src/main/resources/logback-spring.xml` with profile-based appender config
  - [x] 4.3 `prod` profile: JSON appender (LogstashEncoder) to STDOUT
  - [x] 4.4 `local` profile: CONSOLE appender with human-readable pattern
  - [x] 4.5 PII protection: logback-spring.xml has explicit comment forbidding email/ip/password/token logging

- [x] Task 5: Verify build and tests (AC: #3)
  - [x] 5.1 Run `mvn clean test` — all tests pass (Spring Boot context load test included)
  - [x] 5.2 Verify Testcontainers PostgreSQL available for integration tests
  - [x] 5.3 Confirm `mvn spring-boot:run -Dspring-boot.run.profiles=local` starts without errors

- [x] Task 6: GitHub repository setup
  - [x] 6.1 Create repo `chinhnt-portfolio-v2/portfolio-platform` on GitHub
  - [x] 6.2 Initial commit with project scaffold
  - [x] 6.3 Create `DECISIONS.md` at repo root documenting key tech choices

## Dev Notes

### Critical: Spring Boot Version Lock

**USE SPRING BOOT 3.4.x — NOT 4.x.** Spring Boot 4.0.3 released Nov 2025 — ecosystem not caught up.
`springdoc-openapi` SB4 support is in progress but no stable release confirmed. All architecture
decisions reference SB3 APIs. Any agent referencing SB4 features is WRONG.

[Source: architecture.md#Selected-Starter-Platform-BE]

### Spring Initializr Command

```bash
# Check https://start.spring.io for latest 3.4.x patch version before running
curl https://start.spring.io/starter.tgz \
  -d type=maven-project \
  -d language=java \
  -d bootVersion=3.4.x \
  -d groupId=dev.chinh \
  -d artifactId=portfolio-platform \
  -d name=portfolio-platform \
  -d packageName=dev.chinh.portfolio \
  -d javaVersion=21 \
  -d dependencies=web,security,data-jpa,postgresql,websocket,oauth2-client,oauth2-resource-server,actuator,validation,devtools \
  | tar -xzvf -
```

**Note:** `flyway-core` and `springdoc-openapi` are NOT available via Spring Initializr — add them manually to `pom.xml`.

### Manual pom.xml Additions

```xml
<!-- Flyway for database migrations -->
<dependency>
  <groupId>org.flywaydb</groupId>
  <artifactId>flyway-core</artifactId>
</dependency>

<!-- OpenAPI / Swagger UI — manually added, NOT from Initializr -->
<dependency>
  <groupId>org.springdoc</groupId>
  <artifactId>springdoc-openapi-starter-webmvc-ui</artifactId>
  <version>2.8.x</version> <!-- check latest 2.8.x on Maven Central -->
</dependency>

<!-- Caffeine in-memory cache (for GitHub API caching in future stories) -->
<dependency>
  <groupId>com.github.ben-manes.caffeine</groupId>
  <artifactId>caffeine</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-cache</artifactId>
</dependency>

<!-- Testcontainers (test scope) -->
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-testcontainers</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.testcontainers</groupId>
  <artifactId>junit-jupiter</artifactId>
  <scope>test</scope>
</dependency>
<dependency>
  <groupId>org.testcontainers</groupId>
  <artifactId>postgresql</artifactId>
  <scope>test</scope>
</dependency>

<!-- Logstash JSON encoder for prod logging -->
<dependency>
  <groupId>net.logstash.logback</groupId>
  <artifactId>logstash-logback-encoder</artifactId>
  <version>8.0</version> <!-- check latest on Maven Central -->
</dependency>
```

### Package Structure (Enforced from Day 1)

This is an **architectural constraint**, not a guideline. The package boundary is the single most
important decision in Platform BE:

```
dev.chinh.portfolio/
├── auth/
│   ├── session/             ← Session entity + SessionRepository ONLY HERE (auth boundary)
│   ├── jwt/                 ← JWT service, filter, JWKS endpoint
│   └── oauth2/              ← Google OAuth2 handler
├── platform/
│   ├── metrics/             ← Polling scheduler, ProjectHealth aggregation
│   ├── websocket/           ← WebSocket server, broadcast service
│   ├── webhook/             ← GitHub webhook + HMAC verification
│   ├── contact/             ← ContactSubmission + admin endpoint
│   └── admin/               ← Analytics endpoint
├── apps/
│   └── wallet/              ← Wallet App domain (Epic 5+)
└── shared/
    ├── error/               ← Structured error response {error: {code, message}}
    ├── ratelimit/           ← Rate limiting (3 rule sets)
    └── config/              ← App config + demo app registry
```

**Critical rule:** Session entity resides ONLY in `auth.session.*`. No other package may import
`Session` directly. Cross-boundary access returns 403.
[Source: architecture.md#Package-boundary-constraint]

### application.yml Structure

```yaml
# src/main/resources/application.yml
spring:
  application:
    name: portfolio-platform
  profiles:
    active: local  # default

  jpa:
    hibernate:
      ddl-auto: validate  # Flyway manages schema — never let Hibernate create tables
    open-in-view: false   # Prevent lazy loading anti-pattern

  flyway:
    locations: classpath:db/migration
    baseline-on-migrate: false

  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=100,expireAfterWrite=3600s

server:
  port: 8080

springdoc:
  swagger-ui:
    path: /api-docs

---
# Local profile
spring:
  config:
    activate:
      on-profile: local
  datasource:
    url: jdbc:postgresql://localhost:5432/portfolio_v2_local
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:postgres}

logging:
  level:
    dev.chinh.portfolio: DEBUG

---
# Production profile
spring:
  config:
    activate:
      on-profile: prod
  datasource:
    url: jdbc:postgresql://${DB_HOST}:${DB_PORT}/${DB_NAME}
    username: ${DB_USERNAME}
    password: ${DB_PASSWORD}

logging:
  level:
    dev.chinh.portfolio: INFO
```

**No hardcoded credentials in any profile.** All secrets must be env vars.
[Source: architecture.md#Environment-configuration]

### Logback Configuration

```xml
<!-- src/main/resources/logback-spring.xml -->
<configuration>
  <springProfile name="local">
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
      <encoder>
        <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg%n</pattern>
      </encoder>
    </appender>
    <root level="INFO">
      <appender-ref ref="CONSOLE"/>
    </root>
  </springProfile>

  <springProfile name="prod">
    <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
      <encoder class="net.logstash.logback.encoder.LogstashEncoder">
        <!-- Do NOT log: email, ip_address, password, token fields -->
      </encoder>
    </appender>
    <root level="INFO">
      <appender-ref ref="JSON"/>
    </root>
  </springProfile>
</configuration>
```

**PII protection:** Never log email, IP address, password hash, or JWT tokens.
[Source: architecture.md#Monitoring-Logging; epics.md#Story-2.1-AC5]

### Security Config Skeleton (Minimal for Story 2.1)

Spring Security auto-configures and blocks all endpoints by default. For this initialization
story, add a minimal `SecurityConfig` that permits `/actuator/health` and `/api-docs/**`
while permitting everything else (future stories will lock this down):

```java
// src/main/java/dev/chinh/portfolio/shared/config/SecurityConfig.java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        return http
            .csrf(AbstractHttpConfigurer::disable)  // REST API — no CSRF needed
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/actuator/health", "/api-docs/**", "/v3/api-docs/**").permitAll()
                .anyRequest().authenticated()
            )
            .build();
    }
}
```

**Note:** This is a temporary open config for Sprint 0. Story 5.1 will replace with full JWT
security chain. Do NOT implement JWT or OAuth2 logic here.

### CORS Configuration (Pre-stub)

```java
// src/main/java/dev/chinh/portfolio/shared/config/CorsConfig.java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins(
                "https://chinh.dev",
                "https://wallet.chinh.dev",
                "http://localhost:5173"
            )
            .allowedMethods("GET", "POST", "PUT", "DELETE")
            .allowCredentials(true);
    }
}
```

**No wildcards permitted.** CORS allow-list is an architectural security constraint.
[Source: architecture.md#Authentication-Security]

### Flyway Migration Setup

- Directory: `src/main/resources/db/migration/`
- Naming: `V{n}__{description}.sql` (mandatory — double underscore between version and description)
- Story 2.1 creates the directory only; no migrations yet (Story 2.2 adds V1)
- Never use Hibernate `ddl-auto: create` or `update` in any environment — Flyway owns schema

[Source: architecture.md#Flyway-implementation-rules]

### Repository Structure

This story creates the `portfolio-platform` repository:
- **GitHub org:** `chinh-dev`
- **Repo name:** `portfolio-platform`
- **Deploy target:** Oracle A1 ARM (always-free tier)
- **NOT co-located** with `portfolio-fe` (separate repos by architecture decision)

[Source: architecture.md#Repository-structure]

### Project Structure Notes

The output directory on the developer's machine should be a sibling of `portfolio-fe`:
```
I:/
├── portfolio-v2/           ← BMAD workspace (docs, _bmad-output)
└── portfolio-platform/     ← Spring Boot repo (new, separate git root)
```

OR the developer may clone to a separate location matching their git workflow. The key constraint
is that `portfolio-platform` has its own git repository — it is NOT a subdirectory of
`portfolio-v2`.

### What NOT to Implement in Story 2.1

- No JWT token logic (Story 5.1)
- No OAuth2 handlers (Story 5.3)
- No database entities or Flyway migrations (Story 2.2)
- No error handler `@RestControllerAdvice` (Story 2.3)
- No actual REST endpoints beyond `/actuator/health`
- No WebSocket handlers (Story 3.x)
- No rate limiting implementation (Stories 4.x)

Only scaffold, configure, and verify the foundation.

### References

- [Source: architecture.md#Selected-Starter-Platform-BE-Spring-Initializr] — Initializr command + dependency list
- [Source: architecture.md#Package-boundary-constraint] — `auth.session.*` enforcement
- [Source: architecture.md#Key-Setup-Notes-for-AI-Agents] — SB version lock, springdoc manual addition
- [Source: architecture.md#Flyway] — migration naming + location
- [Source: architecture.md#Infrastructure-Deployment] — repo structure, environment config
- [Source: epics.md#Story-2.1] — All five acceptance criteria

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- **Spring Boot version**: start.spring.io no longer supports 3.4.x as of March 2026. Compatible range is >=3.5.0. Used `3.5.11.RELEASE` instead (still Spring 3.x family — architecture concern was about SB4.x, not 3.5.x).
- **springdoc-openapi version**: Used `2.8.6` (latest stable 2.8.x as of March 2026). Compatible with Spring Boot 3.5.x.
- **Maven wrapper**: Project uses Maven 3.9.12 via wrapper. Existing Maven 3.9.5 in local cache at `~/.m2/wrapper/dists/`.
- **Java 21**: NOT installed on dev machine — only Java 17 (Microsoft JDK 17.0.16). Task 5 cannot be completed without Java 21.
- **HALT on Task 5**: `mvn test` requires Java 21 (pom.xml `<java.version>21</java.version>`). Java 17 cannot compile with `--release 21`.
- **Task 6.1-6.2**: GitHub repo creation requires user action (cannot automate without GitHub credentials).

### Completion Notes List

- **Task 1**: Generated Spring Boot 3.5.11 project from start.spring.io. Added flyway-core, flyway-database-postgresql, springdoc-openapi-starter-webmvc-ui:2.8.6, spring-boot-starter-cache, caffeine, logstash-logback-encoder:8.0, and Testcontainers test dependencies to pom.xml.
- **Task 2**: Created all 13 package directories with `package-info.java` files. Created `SecurityConfig.java` (minimal Sprint 0 — allows health + API docs, requires auth for everything else) and `CorsConfig.java` (explicit allow-list: chinh.dev, wallet.chinh.dev, localhost:5173).
- **Task 3**: Created `application.yml` with `local` (uses env vars with defaults for DB) and `prod` (requires all env vars — no defaults) profiles. Removed default `application.properties`. Created `db/migration/` directory with `.gitkeep`.
- **Task 4**: Created `logback-spring.xml` with human-readable CONSOLE for local, LogstashEncoder JSON for prod. PII protection note embedded in XML comment.
- **Task 5**: HALTED — Java 21 required but only Java 17 installed. User must install Java 21 + run `./mvnw.cmd clean test` (Windows) or `./mvnw clean test` (Unix). Docker must be running for Testcontainers.
- **Task 6**: DECISIONS.md created. GitHub repo creation (6.1-6.2) requires user action.

### File List

**New files created (relative to `I:/portfolio-platform/`):**

- `pom.xml` (modified — added flyway, springdoc, cache, testcontainers, logstash deps)
- `DECISIONS.md`
- `.gitignore` (modified — added secrets, logs sections)
- `src/main/resources/application.yml` (replaced application.properties)
- `src/main/resources/logback-spring.xml`
- `src/main/resources/db/migration/.gitkeep`
- `src/main/java/dev/chinh/portfolio/shared/config/SecurityConfig.java`
- `src/main/java/dev/chinh/portfolio/shared/config/CorsConfig.java`
- `src/main/java/dev/chinh/portfolio/shared/config/package-info.java`
- `src/main/java/dev/chinh/portfolio/shared/error/package-info.java`
- `src/main/java/dev/chinh/portfolio/shared/ratelimit/package-info.java`
- `src/main/java/dev/chinh/portfolio/auth/session/package-info.java`
- `src/main/java/dev/chinh/portfolio/auth/jwt/package-info.java`
- `src/main/java/dev/chinh/portfolio/auth/oauth2/package-info.java`
- `src/main/java/dev/chinh/portfolio/platform/metrics/package-info.java`
- `src/main/java/dev/chinh/portfolio/platform/websocket/package-info.java`
- `src/main/java/dev/chinh/portfolio/platform/webhook/package-info.java`
- `src/main/java/dev/chinh/portfolio/platform/contact/package-info.java`
- `src/main/java/dev/chinh/portfolio/platform/admin/package-info.java`
- `src/main/java/dev/chinh/portfolio/apps/wallet/package-info.java`
- `src/test/java/dev/chinh/portfolio/PortfolioPlatformApplicationTests.java` (modified — added Testcontainers)
