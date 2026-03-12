# Story 2.4.2: GitHub Actions CI/CD Pipeline

Status: done

## Story

As the developer,
I want automated CI/CD so every push to `main` deploys Platform BE to Google Cloud Run,
so that deployment is zero-manual-steps.

> **Updated (2026-03-09):** Oracle A1 abandoned (no capacity). Target is now Google Cloud Run + Neon PostgreSQL per Story 2.4.0.

## Acceptance Criteria

1. Given a push to `main` branch, when GitHub Actions `deploy.yml` workflow triggers, then it runs in order: (1) unit tests via `mvn test`, (2) build JAR via `mvn clean package -DskipTests`, (3) SCP JAR to Oracle A1 at `/opt/portfolio-platform/`, (4) SSH restart Spring Boot service via `sudo systemctl restart portfolio-platform`.

2. Given the deploy workflow runs and all steps pass, when deployment completes, then the app is reachable at the production URL within 5 minutes; `curl https://<domain>/actuator/health` returns HTTP 200.

3. Given a test step fails, when the workflow runs, then the deployment step is skipped; GitHub Actions marks the workflow as failed and sends a failure notification.

4. Given the workflow file exists at `.github/workflows/deploy.yml`, when the repository is cloned, then no secrets are hardcoded — all sensitive values (`ORACLE_HOST`, `ORACLE_USER`, `ORACLE_SSH_KEY`) are referenced as `${{ secrets.* }}`.

5. Given the `deploy/portfolio-platform.service` systemd unit file is committed to the repo, when deployed to Oracle A1, then it configures Spring Boot to run as user `ubuntu`, loads env vars from `/opt/portfolio-platform/.env`, and restarts automatically on failure.

6. Given `deploy/deploy.sh` is committed, when executed on Oracle A1 via SSH, then it: stops the service, copies the new JAR, starts the service, and exits non-zero on any failure.

7. Given the Nginx config at `deploy/nginx/portfolio-v2.nginx.conf` (created in Story 2.4.1), when the deploy workflow runs, then it SCPs the file to `/etc/nginx/sites-available/portfolio-v2` on Oracle A1 and runs `sudo nginx -t && sudo systemctl reload nginx`.

## Tasks / Subtasks

- [x] Task 1: Create GitHub Actions deploy workflow (AC: #1, #2, #3, #4)
  - [x] 1.1: Create `.github/workflows/deploy.yml` with trigger on push to `main`
  - [x] 1.2: Add `test` job: checkout → set up JDK 21 → `mvn test`
  - [x] 1.3: Add `deploy` job with `needs: test`: checkout → set up JDK 21 → `mvn clean package -DskipTests`
  - [x] 1.4: Add SSH key setup step using `webfactory/ssh-agent@v0.9.0` action with `ORACLE_SSH_KEY` secret
  - [x] 1.5: Add SCP step for JAR file to `ubuntu@${{ secrets.ORACLE_HOST }}:/opt/portfolio-platform/portfolio-platform.jar`
  - [x] 1.6: Add SCP step for Nginx config to `ubuntu@${{ secrets.ORACLE_HOST }}:/tmp/portfolio-v2.nginx.conf`
  - [x] 1.7: Add SSH step to run deploy: move Nginx config, reload Nginx, restart service

- [x] Task 2: Create systemd unit file (AC: #5)
  - [x] 2.1: Create `deploy/portfolio-platform.service` with proper `[Unit]`, `[Service]`, `[Install]` sections
  - [x] 2.2: Configure `User=ubuntu`, `WorkingDirectory=/opt/portfolio-platform`, `EnvironmentFile=/opt/portfolio-platform/.env`
  - [x] 2.3: Set `ExecStart=/usr/bin/java -jar -Dspring.profiles.active=prod portfolio-platform.jar`
  - [x] 2.4: Set `Restart=always`, `RestartSec=10`

- [x] Task 3: Create deploy script (AC: #6)
  - [x] 3.1: Create `deploy/deploy.sh` with `set -e` (fail fast)
  - [x] 3.2: Add `sudo systemctl stop portfolio-platform` step
  - [x] 3.3: Add JAR copy step: `cp /tmp/portfolio-platform.jar /opt/portfolio-platform/portfolio-platform.jar`
  - [x] 3.4: Add `sudo systemctl start portfolio-platform` step
  - [x] 3.5: Add post-start health check: poll `/actuator/health` until 200 or timeout 60s

- [x] Task 4: Smoke test script (AC: #2)
  - [x] 4.1: Create `deploy/smoke-tests.sh` that pings `/actuator/health` and `/api/v1/project-health`
  - [x] 4.2: Exit non-zero if either endpoint returns non-200 after 3 retries
  - [x] 4.3: Add smoke test call as final step in `deploy.yml` workflow after service restart

- [x] Task 5: Document manual Oracle A1 setup (AC: #5)
  - [x] 5.1: Add README section in `deploy/README.md` documenting one-time Oracle A1 bootstrap steps
  - [x] 5.2: Document: install service file (`sudo cp deploy/portfolio-platform.service /etc/systemd/system/`), `sudo systemctl daemon-reload`, `sudo systemctl enable portfolio-platform`
  - [x] 5.3: Document: create `/opt/portfolio-platform/` directory, set owner to `ubuntu`
  - [x] 5.4: Document: create `/opt/portfolio-platform/.env` with required env vars (see template below)

## Dev Notes

### Architecture Contract: CI/CD Flow

The complete deployment pipeline per architecture is:

```
Push to main
  └─> GitHub Actions deploy.yml
        ├─> Job: test          → mvn test (unit tests only)
        └─> Job: deploy        → needs: [test]
              ├─> mvn clean package -DskipTests
              ├─> SCP target/portfolio-platform-0.0.1-SNAPSHOT.jar → /opt/portfolio-platform/portfolio-platform.jar
              ├─> SCP deploy/nginx/portfolio-v2.nginx.conf → /tmp/portfolio-v2.nginx.conf
              └─> SSH deploy:
                    sudo mv /tmp/portfolio-v2.nginx.conf /etc/nginx/sites-available/portfolio-v2
                    sudo nginx -t && sudo systemctl reload nginx
                    sudo systemctl restart portfolio-platform
                    ./deploy/smoke-tests.sh
```

[Source: architecture.md#CI/CD section, line ~680]

### JAR Artifact Name

Maven artifact produces: `target/portfolio-platform-0.0.1-SNAPSHOT.jar`

- `artifactId`: `portfolio-platform`
- `version`: `0.0.1-SNAPSHOT`
- No `<finalName>` override in pom.xml — use default name

Production path: `/opt/portfolio-platform/portfolio-platform.jar`

### GitHub Actions Workflow Structure

**File:** `.github/workflows/deploy.yml`

Architecture specifies this file as `ci-cd.yml` in one place and `deploy.yml` in another. Story 2.4.2 epics specifies `deploy.yml` — use `deploy.yml`.

```yaml
name: Deploy Platform BE

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'
      - name: Run tests
        run: mvn test

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Set up JDK 21
        uses: actions/setup-java@v4
        with:
          java-version: '21'
          distribution: 'temurin'
          cache: 'maven'
      - name: Build JAR
        run: mvn clean package -DskipTests
      - name: Set up SSH agent
        uses: webfactory/ssh-agent@v0.9.0
        with:
          ssh-private-key: ${{ secrets.ORACLE_SSH_KEY }}
      - name: Add Oracle A1 to known hosts
        run: |
          ssh-keyscan -H ${{ secrets.ORACLE_HOST }} >> ~/.ssh/known_hosts
      - name: SCP JAR to Oracle A1
        run: |
          scp target/portfolio-platform-0.0.1-SNAPSHOT.jar \
            ${{ secrets.ORACLE_USER }}@${{ secrets.ORACLE_HOST }}:/tmp/portfolio-platform.jar
      - name: SCP Nginx config to Oracle A1
        run: |
          scp deploy/nginx/portfolio-v2.nginx.conf \
            ${{ secrets.ORACLE_USER }}@${{ secrets.ORACLE_HOST }}:/tmp/portfolio-v2.nginx.conf
      - name: Deploy and restart services
        run: |
          ssh ${{ secrets.ORACLE_USER }}@${{ secrets.ORACLE_HOST }} << 'EOF'
            set -e
            # Update Nginx config
            sudo mv /tmp/portfolio-v2.nginx.conf /etc/nginx/sites-available/portfolio-v2
            sudo nginx -t
            sudo systemctl reload nginx
            # Deploy JAR
            sudo systemctl stop portfolio-platform || true
            cp /tmp/portfolio-platform.jar /opt/portfolio-platform/portfolio-platform.jar
            sudo systemctl start portfolio-platform
          EOF
      - name: Run smoke tests
        run: |
          ssh ${{ secrets.ORACLE_USER }}@${{ secrets.ORACLE_HOST }} 'bash /opt/portfolio-platform/smoke-tests.sh'
```

### GitHub Secrets Required

Configure these in: GitHub repo → Settings → Secrets and variables → Actions → New repository secret

| Secret | Value | Notes |
|---|---|---|
| `ORACLE_HOST` | IP or domain of Oracle A1 instance | e.g. `152.xx.xx.xx` or `api.yourdomain.com` |
| `ORACLE_USER` | SSH username | `ubuntu` (default for Oracle Ubuntu images) |
| `ORACLE_SSH_KEY` | Private SSH key (PEM format) | Generated with `ssh-keygen -t ed25519 -C "github-actions"` |

**How to set up SSH key for GitHub Actions:**
```bash
# On local machine — generate dedicated key for CI
ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/oracle_deploy_key -N ""

# Add public key to Oracle A1 authorized_keys
cat ~/.ssh/oracle_deploy_key.pub | ssh ubuntu@<ORACLE_HOST> "cat >> ~/.ssh/authorized_keys"

# Add private key content to GitHub secret ORACLE_SSH_KEY
cat ~/.ssh/oracle_deploy_key
# Copy entire output including -----BEGIN/END OPENSSH PRIVATE KEY----- lines
```

### systemd Unit File

**File:** `deploy/portfolio-platform.service`

```ini
[Unit]
Description=Portfolio Platform API
After=network.target postgresql.service

[Service]
User=ubuntu
WorkingDirectory=/opt/portfolio-platform
ExecStart=/usr/bin/java -jar -Dspring.profiles.active=prod portfolio-platform.jar
Restart=always
RestartSec=10
EnvironmentFile=/opt/portfolio-platform/.env
StandardOutput=journal
StandardError=journal
SyslogIdentifier=portfolio-platform

[Install]
WantedBy=multi-user.target
```

[Source: architecture.md#deploy/portfolio-platform.service template]

### Oracle A1 Environment File Template

**Path on server:** `/opt/portfolio-platform/.env`

This file is NEVER committed to the repo. It must be created manually on Oracle A1:

```bash
# /opt/portfolio-platform/.env
# Spring Boot prod profile picks these up via EnvironmentFile
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/portfolio_v2
SPRING_DATASOURCE_USERNAME=portfolio_user
SPRING_DATASOURCE_PASSWORD=<secure-password>
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID=<google-client-id>
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET=<google-secret>
JWT_SECRET=<base64-encoded-256bit-secret>
```

[Source: architecture.md#Environment configuration — "Secrets: environment variables (never committed to repo)"]

### Deploy Script

**File:** `deploy/deploy.sh`

This script is documented for reference but the CI/CD pipeline runs deploy commands inline via SSH heredoc. The script serves as a manual fallback.

```bash
#!/bin/bash
set -e  # Exit immediately on any error

DEPLOY_DIR=/opt/portfolio-platform
SERVICE_NAME=portfolio-platform
JAR_NAME=portfolio-platform.jar
HEALTH_URL=http://localhost:8080/actuator/health

echo "=== Stopping service ==="
sudo systemctl stop $SERVICE_NAME || true

echo "=== Copying JAR ==="
cp /tmp/$JAR_NAME $DEPLOY_DIR/$JAR_NAME

echo "=== Starting service ==="
sudo systemctl start $SERVICE_NAME

echo "=== Waiting for health check ==="
for i in $(seq 1 30); do
    if curl -sf $HEALTH_URL > /dev/null 2>&1; then
        echo "=== Service healthy after ${i} attempts ==="
        exit 0
    fi
    sleep 2
done

echo "=== ERROR: Health check failed after 60s ==="
exit 1
```

### Smoke Test Script

**File:** `deploy/smoke-tests.sh`

Per NFR-R7: smoke tests run automatically post-deploy.

```bash
#!/bin/bash
set -e

HOST=http://localhost:8080
RETRIES=3
SLEEP=5

check_endpoint() {
    local url=$1
    local name=$2
    for i in $(seq 1 $RETRIES); do
        if curl -sf "$url" > /dev/null 2>&1; then
            echo "OK: $name"
            return 0
        fi
        echo "Attempt $i/$RETRIES failed for $name — retrying in ${SLEEP}s..."
        sleep $SLEEP
    done
    echo "FAIL: $name did not respond after $RETRIES attempts"
    exit 1
}

check_endpoint "$HOST/actuator/health" "actuator/health"
check_endpoint "$HOST/api/v1/project-health" "api/v1/project-health"

echo "All smoke tests passed."
```

### One-Time Oracle A1 Bootstrap (MANUAL — SSH to server)

These steps must be done ONCE before CI/CD can work:

```bash
# 1. Create deployment directory
sudo mkdir -p /opt/portfolio-platform
sudo chown ubuntu:ubuntu /opt/portfolio-platform

# 2. Copy and enable systemd service
sudo cp /tmp/portfolio-platform.service /etc/systemd/system/portfolio-platform.service
sudo systemctl daemon-reload
sudo systemctl enable portfolio-platform

# 3. Create .env file (fill in real values)
nano /opt/portfolio-platform/.env

# 4. Copy smoke test script
cp /tmp/smoke-tests.sh /opt/portfolio-platform/smoke-tests.sh
chmod +x /opt/portfolio-platform/smoke-tests.sh

# 5. Initial deploy: manually SCP JAR and start
# scp target/portfolio-platform-0.0.1-SNAPSHOT.jar ubuntu@<host>:/opt/portfolio-platform/portfolio-platform.jar
sudo systemctl start portfolio-platform
sudo systemctl status portfolio-platform
```

### Spring Boot Profile Reminder

- Profiles are `local` (local dev) and `prod` (Oracle A1) — confirmed in Story 2.3 learnings
- Do NOT use `dev`, `development`, `staging`, or `production` — only `local` / `prod`
- `application-prod.yml` already contains `server.forward-headers-strategy: FRAMEWORK` from Story 2.4.1
- Service is launched with `-Dspring.profiles.active=prod`

[Source: architecture.md#Spring Boot section "Spring profiles: dev and prod only"]

### Project Structure After This Story

```
portfolio-platform/
├── .github/
│   └── workflows/
│       └── deploy.yml          ← NEW: CI/CD workflow
├── deploy/
│   ├── nginx/
│   │   └── portfolio-v2.nginx.conf   ← EXISTS from Story 2.4.1
│   ├── portfolio-platform.service    ← NEW: systemd unit file
│   ├── deploy.sh                     ← NEW: manual deploy fallback script
│   ├── smoke-tests.sh                ← NEW: post-deploy health checks (NFR-R7)
│   └── README.md                     ← NEW: bootstrap instructions
└── src/
    └── ...                           ← no src changes needed
```

### Cross-Story Coordination

- **Story 2.4.1 (done):** Nginx config is already at `deploy/nginx/portfolio-v2.nginx.conf`. This story SCPs it during deployment. File path is fixed — do not rename.
- **Story 2.4.3 (next):** Database backup is a separate cron job on Oracle A1 — no interaction with this story's CI/CD workflow.
- **Story 3.1–3.7 (Epic 3):** After this story, BE deployment is automated. Epic 3 stories can assume that pushing to `main` triggers auto-deploy.

### Testing Notes

This story is primarily infrastructure — no unit tests required beyond what already exists. The deploy workflow itself is the "test":

- The `test` job in `deploy.yml` runs `mvn test` — this covers the existing test suite
- Smoke tests (`smoke-tests.sh`) serve as integration verification post-deploy
- No new JUnit/Mockito tests needed for this story — the artifacts are scripts and YAML, not Java code
- Existing test count: 20 tests in `ReverseProxyConfigTest` + prior stories — all must still pass in CI

### Maven Build Notes

```bash
# Full build with tests (used in test job)
mvn test

# Package JAR without running tests (used in deploy job — tests already passed)
mvn clean package -DskipTests

# Output: target/portfolio-platform-0.0.1-SNAPSHOT.jar
```

Maven wrapper (`mvnw`) can be used instead of `mvn` if available. Using bare `mvn` in CI is fine since `actions/setup-java` with `cache: maven` installs Maven.

### Previous Story Learnings (Story 2.4.1)

- **Spring Boot version:** 3.5.11.RELEASE — use 3.5.x docs
- **Spring profiles:** `local` / `prod` only — NEVER `dev` or `production`
- **Port is fixed at 8080** — all proxy/firewall rules target `localhost:8080`
- **Oracle A1 dual-layer firewall:** OCI Console Security List + OS iptables — both must allow ports (ports 80/443 already opened in 2.4.1)
- **Nginx config location:** `deploy/nginx/portfolio-v2.nginx.conf` — canonical repo file, deployed via SCP in this story
- **`application.yml` already has** `server.forward-headers-strategy: FRAMEWORK` and `server.tomcat.remoteip.internal-proxies` — no changes to application config needed

### Project Structure Notes

- Backend root: `I:/portfolio-v2/portfolio-platform/`
- Deploy dir (already partially exists): `portfolio-platform/deploy/nginx/portfolio-v2.nginx.conf`
- GitHub workflows dir to create: `portfolio-platform/.github/workflows/`
- No `src/` changes required for this story

### References

- CI/CD architecture decision: [Source: architecture.md#CI/CD section (~line 680)]
- Repository structure and deploy dir: [Source: architecture.md#portfolio-platform directory tree (~line 1320)]
- systemd unit file template: [Source: architecture.md#deploy/portfolio-platform.service (~line 1443)]
- NFR-R7 smoke tests requirement: [Source: architecture.md#Validation section (~line 1721)]
- Story 2.4.2 acceptance criteria: [Source: epics.md#Story 2.4.2 (~line 793)]
- Story 2.4.1 cross-story note: [Source: implementation-artifacts/2-4-1-nginx-reverse-proxy-ssl-websocket-support.md#Cross-Story Impact]

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

_No issues encountered._

### Completion Notes List

- All 5 tasks completed in one session. Story is pure infrastructure — no Java source changes.
- Created `.github/workflows/deploy.yml`: triggers on push to `main`, `test` job runs `mvn test`, `deploy` job (needs: test) builds JAR, SCPs JAR + Nginx config to Oracle A1 via SSH, runs inline deploy commands, then smoke tests.
- Created `deploy/portfolio-platform.service`: systemd unit configuring Spring Boot with `User=ubuntu`, `EnvironmentFile=/opt/portfolio-platform/.env`, `-Dspring.profiles.active=prod`, `Restart=always`.
- Created `deploy/deploy.sh`: manual fallback script with `set -e`, stop/copy/start pattern, 60s health poll on `/actuator/health`.
- Created `deploy/smoke-tests.sh`: checks `/actuator/health` and `/api/v1/project-health` with 3 retries each.
- Created `deploy/README.md`: documents Oracle A1 bootstrap, GitHub Secrets setup, and CI/CD pipeline overview.
- Existing test suite: **20/20 tests pass** (no regressions). `mvnw test` → BUILD SUCCESS.
- No new JUnit tests added (infrastructure artifacts only — per story Dev Notes).

**Code Review Fixes (2026-03-07):**
- Added `ORACLE_HOST_KEY` secret-based known_hosts verification (replaces `ssh-keyscan` TOFU pattern)
- Added SCP step for `smoke-tests.sh` to ensure CI/CD updates the script on server (was only manually bootstrapped)
- Added `cp + chmod` for smoke-tests in SSH deploy step to install the updated script
- Added `environment: production` to deploy job for GitHub Actions deployment protection gates
- Added `timeout-minutes: 15` to deploy job to prevent runaway workflows
- Fixed `smoke-tests.sh` FAIL message to output to stderr (`>&2`)
- Documented `ORACLE_HOST_KEY` secret generation in `deploy/README.md`
- Documented git repository placement requirement (must be repo root) in `deploy/README.md`

### File List

- `.github/workflows/deploy.yml` (new)
- `deploy/portfolio-platform.service` (new)
- `deploy/deploy.sh` (new)
- `deploy/smoke-tests.sh` (modified)
- `deploy/README.md` (modified)

### Change Log

- 2026-03-07: Initial implementation — created all CI/CD infrastructure artifacts; 20/20 tests pass; story moved to review.
- 2026-03-07: Code review fixes — added ORACLE_HOST_KEY fingerprint verification, smoke-tests SCP in CI/CD, environment protection, timeout, stderr fix; story moved to done.
