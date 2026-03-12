# Story 2.4.0: Fly.io App Provisioning & Initial Deployment

Status: ready-for-dev

## Story

As the developer,
I want the Spring Boot backend provisioned and running on Fly.io with PostgreSQL,
so that Stories 2.4.2 (CI/CD pipeline) can deploy automatically on every push to `main`.

## Context / Decision

Original story targeted Oracle A1 (always-free ARM VM). Oracle A1 capacity is unavailable in all
Asia-Pacific regions. Fly.io is the replacement: PaaS platform with genuine free tier, no VM
provisioning, TLS handled natively (Story 2.4.1 Nginx is obsolete — see note below).

**Impact on other stories:**
- **Story 2.4.1 (Nginx)** → Obsolete. Fly.io terminates TLS and proxies internally. Nginx config
  files in `deploy/nginx/` can be deleted. Story marked obsolete.
- **Story 2.4.2 (CI/CD)** → Must be rewritten: replace SSH/SCP steps with `flyctl deploy`.

## Acceptance Criteria

1. Given flyctl is installed and authenticated, when `fly status --app portfolio-platform` is run,
   then output shows the app exists in region `sin` (Singapore) with at least 1 machine running.

2. Given the app is deployed, when `curl https://portfolio-platform.fly.dev/actuator/health` is
   called, then response is HTTP 200 with `{"status":"UP"}` — confirming Spring Boot and
   PostgreSQL connection are both healthy.

3. Given Fly Postgres is attached, when Spring Boot starts, then Flyway runs migrations
   successfully (log shows `Successfully applied N migration(s)` with no errors).

4. Given `fly secrets list --app portfolio-platform` is run, then secrets
   `SPRING_DATASOURCE_PASSWORD`, `JWT_SECRET`, `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID`,
   and `SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET` all exist (values hidden).

5. Given the `Dockerfile` is committed to `portfolio-platform/Dockerfile`, when `fly deploy` runs,
   then the build succeeds using Temurin 21 JRE base image with JVM memory flags set.

6. Given `fly.toml` is committed to `portfolio-platform/fly.toml`, when inspected, then
   `primary_region = "sin"`, `internal_port = 8080`, `force_https = true`, and health check path
   is `/actuator/health`.

7. Given GitHub secret `FLY_API_TOKEN` is set at repo → Settings → Secrets → Actions, then the
   CI/CD pipeline from Story 2.4.2 can deploy to Fly.io without interactive login.

## Tasks / Subtasks

- [ ] Task 1: Install flyctl and create Fly.io account (AC: #1) **[MANUAL]**
  - [ ] 1.1: Install flyctl on local machine:
    ```bash
    # Windows (PowerShell)
    pwsh -Command "iwr https://fly.io/install.ps1 -useb | iex"
    # Or via scoop: scoop install flyctl
    ```
  - [ ] 1.2: Create account and authenticate:
    ```bash
    fly auth signup
    # or if already have account:
    fly auth login
    ```
  - [ ] 1.3: Verify: `fly version` and `fly auth whoami`

- [ ] Task 2: Create Dockerfile (AC: #5)
  - [ ] 2.1: Create `portfolio-platform/Dockerfile`:
    ```dockerfile
    FROM eclipse-temurin:21-jre-jammy AS runtime
    WORKDIR /app
    COPY target/portfolio-platform-*.jar app.jar
    EXPOSE 8080
    ENTRYPOINT ["java", \
      "-XX:MaxRAMPercentage=75", \
      "-XX:+UseG1GC", \
      "-Dspring.profiles.active=prod", \
      "-jar", "app.jar"]
    ```
  - [ ] 2.2: Create `portfolio-platform/.dockerignore`:
    ```
    .git
    .github
    target/
    !target/portfolio-platform-*.jar
    src/
    deploy/
    *.md
    ```
    > Note: `.dockerignore` excludes `target/` then re-includes the JAR so Docker COPY works
    > after local `mvn package` builds the JAR first.

- [ ] Task 3: Initialize Fly.io app and create fly.toml (AC: #1, #6) **[MANUAL CLI]**
  - [ ] 3.1: Build JAR first (required before `fly launch` detects Spring Boot):
    ```bash
    cd portfolio-platform
    ./mvnw clean package -DskipTests
    ```
  - [ ] 3.2: Launch app (no deploy yet):
    ```bash
    fly launch \
      --name portfolio-platform \
      --region sin \
      --no-deploy \
      --dockerfile Dockerfile
    ```
    When prompted: accept suggested config, say NO to Postgres (create separately in Task 4)
  - [ ] 3.3: Edit generated `fly.toml` to match:
    ```toml
    app = "portfolio-platform"
    primary_region = "sin"

    [build]

    [env]
      SERVER_PORT = "8080"

    [http_service]
      internal_port = 8080
      force_https = true
      auto_stop_machines = "stop"
      auto_start_machines = true
      min_machines_running = 0
      processes = ["app"]

    [[http_service.checks]]
      grace_period = "30s"
      interval = "15s"
      method = "GET"
      path = "/actuator/health"
      timeout = "5s"

    [[vm]]
      memory = "512mb"
      cpu_kind = "shared"
      cpus = 1
    ```
    > `auto_stop_machines = "stop"` + `min_machines_running = 0`: machine sleeps when no traffic
    > (cold start ~5s). For portfolio this is acceptable and keeps cost at $0/month.
    > If always-on is needed: set `min_machines_running = 1` (~$3.19/month for 512MB).

- [ ] Task 4: Create and attach Fly Postgres (AC: #3) **[MANUAL CLI]**
  - [ ] 4.1: Create Postgres cluster (free tier: shared-cpu-1x, 256MB, 1GB volume):
    ```bash
    fly postgres create \
      --name portfolio-db \
      --region sin \
      --initial-cluster-size 1 \
      --vm-size shared-cpu-1x \
      --volume-size 1
    ```
    > Save the output — it shows the `DATABASE_URL`, `OPERATOR_PASSWORD`, etc. Save securely.
  - [ ] 4.2: Attach Postgres to the app (auto-sets `DATABASE_URL` secret):
    ```bash
    fly postgres attach portfolio-db --app portfolio-platform
    ```
    > This sets `DATABASE_URL` secret automatically. Spring Boot prod profile must map this to
    > `SPRING_DATASOURCE_URL` — see Dev Notes for `application-prod.yml` config required.
  - [ ] 4.3: Verify attachment: `fly secrets list --app portfolio-platform` → shows `DATABASE_URL`

- [ ] Task 5: Set remaining app secrets (AC: #4) **[MANUAL CLI]**
  - [ ] 5.1: Set secrets (replace placeholder values with real ones):
    ```bash
    fly secrets set \
      JWT_SECRET=$(openssl rand -base64 32) \
      SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID=placeholder \
      SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET=placeholder \
      --app portfolio-platform
    ```
    > Google OAuth2 credentials: `console.cloud.google.com` → APIs & Services → Credentials →
    > OAuth 2.0 Client IDs. Use `placeholder` now to prevent Spring Boot startup failure —
    > replace with real values before Stories 5.2–5.4.
  - [ ] 5.2: Verify all secrets exist: `fly secrets list --app portfolio-platform`

- [ ] Task 6: Initial deploy and smoke test (AC: #1, #2, #3) **[MANUAL CLI]**
  - [ ] 6.1: Deploy:
    ```bash
    cd portfolio-platform
    ./mvnw clean package -DskipTests
    fly deploy --app portfolio-platform
    ```
  - [ ] 6.2: Watch logs for startup (Flyway migrations, datasource):
    ```bash
    fly logs --app portfolio-platform
    ```
  - [ ] 6.3: Verify health endpoint:
    ```bash
    curl https://portfolio-platform.fly.dev/actuator/health
    # Expected: {"status":"UP"}
    ```
  - [ ] 6.4: Check machine is running: `fly status --app portfolio-platform`

- [ ] Task 7: Add GitHub secret for CI/CD (AC: #7) **[MANUAL]**
  - [ ] 7.1: Get Fly.io API token:
    ```bash
    fly tokens create deploy --app portfolio-platform
    ```
  - [ ] 7.2: Add to GitHub: repo → Settings → Secrets and variables → Actions → New repository secret:
    - `FLY_API_TOKEN` = token from step 7.1

- [ ] Task 8: Clean up Oracle A1 artifacts (housekeeping)
  - [ ] 8.1: Delete obsolete Oracle deploy files:
    ```bash
    rm portfolio-platform/deploy/portfolio-platform.service
    rm portfolio-platform/deploy/deploy.sh
    rm -rf portfolio-platform/deploy/nginx/
    ```
  - [ ] 8.2: Keep `deploy/smoke-tests.sh` — adapt to call Fly.io URL instead of localhost:
    Update `BASE_URL` in smoke-tests.sh from `http://localhost:8080` to
    `https://portfolio-platform.fly.dev`
  - [ ] 8.3: Update `deploy/README.md` to reflect Fly.io deployment
  - [ ] 8.4: Commit: `git add -A && git commit -m "chore: replace Oracle A1 deploy artifacts with Fly.io"`

## Dev Notes

### Why Fly.io over Oracle A1

| | Oracle A1 | Fly.io |
|---|---|---|
| Cost | $0 (always-free) | $0 (free allowances) |
| Provisioning | Manual VM setup | `fly launch` |
| TLS/HTTPS | Nginx + Certbot | Built-in, automatic |
| PostgreSQL | Self-managed | `fly postgres` managed |
| CI/CD | SSH + SCP | `flyctl deploy` |
| Cold start | N/A (always on) | ~5s if auto-stopped |
| Availability | ARM capacity often unavailable | Available immediately |

### application-prod.yml — DATABASE_URL Mapping Required

Fly.io sets a `DATABASE_URL` environment variable in format:
`postgres://user:password@host:5432/dbname`

Spring Boot's `SPRING_DATASOURCE_URL` expects JDBC format:
`jdbc:postgresql://host:5432/dbname`

Add to `src/main/resources/application-prod.yml`:
```yaml
spring:
  datasource:
    url: ${SPRING_DATASOURCE_URL:${DATABASE_URL}}
    username: ${SPRING_DATASOURCE_USERNAME:}
    password: ${SPRING_DATASOURCE_PASSWORD:}
  flyway:
    url: ${spring.datasource.url}
```

And add to `fly.toml` env section:
```toml
[env]
  SERVER_PORT = "8080"
  SPRING_DATASOURCE_URL = "jdbc:postgresql://${DATABASE_HOST}:5432/${DATABASE_NAME}"
```

> Actually simpler: use `fly postgres attach` which sets individual env vars
> (`DATABASE_URL`, `DATABASE_HOST`, `DATABASE_PORT`, `DATABASE_NAME`, `DATABASE_USER`,
> `DATABASE_PASSWORD`). Map in `application-prod.yml`:
> ```yaml
> spring:
>   datasource:
>     url: jdbc:postgresql://${DATABASE_HOST:localhost}:${DATABASE_PORT:5432}/${DATABASE_NAME:portfolio_v2}
>     username: ${DATABASE_USER:portfolio_user}
>     password: ${DATABASE_PASSWORD:}
> ```

### Fly.io Free Tier Limits (as of 2025)

- **Compute**: 3 shared-cpu-1x 256MB machines free (160GB outbound/month)
- **Postgres**: 1 shared-cpu-1x 256MB + 1GB volume free
- **With auto_stop**: machine stops when idle → well within free limits
- **Custom domain**: `fly certs add yourdomain.com` → free TLS cert

### Story 2.4.1 Status: OBSOLETE

Fly.io provides:
- Automatic TLS termination on port 443
- HTTP → HTTPS redirect (enforced by `force_https = true`)
- WebSocket support natively (no Nginx config needed)

The Nginx config at `deploy/nginx/portfolio-v2.nginx.conf` can be deleted.
Story 2.4.1 should be marked **obsolete** — do not implement.

### Story 2.4.2 CI/CD — Must Be Rewritten

Replace Oracle SSH/SCP deploy steps with:
```yaml
- name: Set up flyctl
  uses: superfly/flyctl-actions/setup-flyctl@master

- name: Deploy to Fly.io
  run: fly deploy --remote-only --app portfolio-platform
  env:
    FLY_API_TOKEN: ${{ secrets.FLY_API_TOKEN }}
```
`--remote-only` means Fly.io builds the Docker image on their infrastructure (no local Docker needed in CI).

### Dependency Map

```
Story 2.4.0 (this) ── Fly.io provisioned, DB attached, secrets set ──┐
                                                                       │
Story 2.4.2 (CI/CD) ── requires: FLY_API_TOKEN secret set ───────────┘
Story 2.4.1 (Nginx) ── OBSOLETE, skip
Story 2.4.3 (DB backup) ── requires: Fly Postgres running
```

### References

- Fly.io Spring Boot guide: https://fly.io/docs/languages-and-frameworks/java/
- Fly Postgres docs: https://fly.io/docs/postgres/
- flyctl install: https://fly.io/docs/flyctl/install/

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

### Completion Notes List

### File List

- `portfolio-platform/Dockerfile` ← new
- `portfolio-platform/.dockerignore` ← new
- `portfolio-platform/fly.toml` ← new
- `portfolio-platform/src/main/resources/application-prod.yml` ← update datasource config
- `portfolio-platform/deploy/smoke-tests.sh` ← update BASE_URL to Fly.io URL
- `portfolio-platform/deploy/README.md` ← update deployment docs
- `portfolio-platform/deploy/portfolio-platform.service` ← DELETE
- `portfolio-platform/deploy/deploy.sh` ← DELETE
- `portfolio-platform/deploy/nginx/` ← DELETE
