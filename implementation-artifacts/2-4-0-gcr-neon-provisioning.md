# Story 2.4.0: Google Cloud Run + Neon Provisioning & Initial Deployment

Status: ready-for-dev

## Story

As the developer,
I want the Spring Boot backend deployed on Google Cloud Run with Neon PostgreSQL,
so that Story 2.4.2 (CI/CD pipeline) can deploy automatically on every push to `main`.

## Context / Decision

Original story targeted Oracle A1 (always-free ARM VM). Oracle A1 and E2.1.Micro capacity
unavailable in all regions. Fly.io and Koyeb no longer have free tiers for new accounts.

**Final stack:**
- **Compute**: Google Cloud Run (scale-to-zero, 2M requests/month free, credit card required but $0 for portfolio traffic)
- **Database**: Neon PostgreSQL (free forever, no credit card, auto-suspend, 0.5GB storage)
- **TLS/HTTPS**: Built-in via Cloud Run (automatic)
- **CI/CD**: GitHub Actions → `gcloud run deploy`

**Impact on other stories:**
- **Story 2.4.1 (Nginx)** → Obsolete. Cloud Run handles TLS natively.
- **Story 2.4.2 (CI/CD)** → Must be rewritten: use `gcloud run deploy` instead of SSH/SCP.

## Acceptance Criteria

1. Given gcloud CLI is installed and authenticated, when `gcloud run services describe portfolio-platform --region asia-southeast1` is run, then output shows the service exists and is serving traffic.

2. Given the service is deployed, when `curl https://<cloud-run-url>/actuator/health` is called, then response is HTTP 200 with `{"status":"UP"}` — confirming Spring Boot and Neon PostgreSQL are both healthy.

3. Given Neon database is configured, when Spring Boot starts, then Flyway runs migrations successfully (log shows `Successfully applied N migration(s)` with no errors).

4. Given Cloud Run env vars are set, when `gcloud run services describe portfolio-platform --region asia-southeast1 --format json` is run, then `SPRING_DATASOURCE_URL`, `JWT_SECRET`, and Google OAuth placeholders are present (values hidden).

5. Given `Dockerfile` is committed to `portfolio-platform/Dockerfile`, when Cloud Build builds it, then image uses Temurin 21 JRE with JVM memory flags.

6. Given GitHub secrets `GCP_PROJECT_ID`, `GCP_SA_KEY`, and `NEON_DATABASE_URL` are set, then Story 2.4.2 CI/CD pipeline can deploy without interactive login.

## Tasks / Subtasks

- [ ] Task 1: Create Google Cloud account and project (AC: #1) **[MANUAL — browser]**
  - [ ] 1.1: Go to `console.cloud.google.com` → sign in with Google account
  - [ ] 1.2: Create new project: click dropdown top-left → "New Project" → name: `portfolio-v2`
  - [ ] 1.3: Add billing account (credit card required — will NOT be charged for free tier usage)
  - [ ] 1.4: Note the **Project ID** (e.g. `portfolio-v2-123456`) — needed later
  - [ ] 1.5: Enable required APIs (can do via Console or CLI in Task 2):
    - Cloud Run API
    - Artifact Registry API
    - Cloud Build API

- [ ] Task 2: Install gcloud CLI and authenticate (AC: #1) **[MANUAL — local machine]**
  - [ ] 2.1: Download and install Google Cloud SDK:
    - Windows: download installer from `cloud.google.com/sdk/docs/install`
    - Or via winget: `winget install Google.CloudSDK`
  - [ ] 2.2: Initialize and authenticate:
    ```bash
    gcloud init
    # Follow prompts: sign in → select project portfolio-v2
    ```
  - [ ] 2.3: Enable APIs:
    ```bash
    gcloud services enable run.googleapis.com \
      artifactregistry.googleapis.com \
      cloudbuild.googleapis.com
    ```
  - [ ] 2.4: Set default region:
    ```bash
    gcloud config set run/region asia-southeast1
    ```
  - [ ] 2.5: Verify: `gcloud auth list` and `gcloud config list`

- [ ] Task 3: Create Artifact Registry repository (AC: #5) **[MANUAL — CLI]**
  - [ ] 3.1: Create Docker registry:
    ```bash
    gcloud artifacts repositories create portfolio-repo \
      --repository-format=docker \
      --location=asia-southeast1 \
      --description="Portfolio platform Docker images"
    ```
  - [ ] 3.2: Configure Docker auth:
    ```bash
    gcloud auth configure-docker asia-southeast1-docker.pkg.dev
    ```

- [ ] Task 4: Create Neon PostgreSQL database (AC: #3) **[MANUAL — browser]**
  - [ ] 4.1: Go to `console.neon.tech` → sign up (no credit card needed)
  - [ ] 4.2: Create new project → name: `portfolio-v2` → region: **AWS Singapore (ap-southeast-1)**
    > Note: Neon runs on AWS, Singapore region is closest to Vietnam
  - [ ] 4.3: After project created, go to **Dashboard → Connection Details**
  - [ ] 4.4: Copy the connection string format:
    ```
    postgresql://neondb_owner:<password>@ep-xxx.ap-southeast-1.aws.neon.tech/neondb?sslmode=require
    ```
    > **Save this securely** — needed for Task 5 env vars
  - [ ] 4.5: Convert to JDBC format for Spring Boot:
    ```
    jdbc:postgresql://ep-xxx.ap-southeast-1.aws.neon.tech/neondb?sslmode=require
    ```
  - [ ] 4.6: Verify connection string works:
    ```bash
    psql "postgresql://neondb_owner:<password>@ep-xxx.ap-southeast-1.aws.neon.tech/neondb?sslmode=require" -c "SELECT 1;"
    ```

- [ ] Task 5: Update application-prod.yml for Neon (AC: #3, #4)
  - [ ] 5.1: Update `src/main/resources/application-prod.yml`:
    ```yaml
    spring:
      datasource:
        url: ${SPRING_DATASOURCE_URL}
        username: ${SPRING_DATASOURCE_USERNAME}
        password: ${SPRING_DATASOURCE_PASSWORD}
        hikari:
          connection-timeout: 30000
          maximum-pool-size: 3
          minimum-idle: 1
          idle-timeout: 600000
          keepalive-time: 30000
      flyway:
        enabled: true
        url: ${SPRING_DATASOURCE_URL}
        user: ${SPRING_DATASOURCE_USERNAME}
        password: ${SPRING_DATASOURCE_PASSWORD}
    ```
    > `maximum-pool-size: 3` — Neon free tier has connection limits; keep pool small.
    > `keepalive-time: 30000` — prevents Neon from closing idle connections.

- [ ] Task 6: Build Docker image and push to Artifact Registry (AC: #5) **[MANUAL — CLI]**
  - [ ] 6.1: Build JAR:
    ```bash
    cd I:/portfolio-v2/portfolio-platform
    ./mvnw clean package -DskipTests
    ```
  - [ ] 6.2: Build and tag Docker image:
    ```bash
    IMAGE=asia-southeast1-docker.pkg.dev/<PROJECT_ID>/portfolio-repo/portfolio-platform:latest
    docker build -t $IMAGE .
    ```
  - [ ] 6.3: Push to Artifact Registry:
    ```bash
    docker push $IMAGE
    ```

- [ ] Task 7: Deploy to Cloud Run (AC: #1, #2, #4) **[MANUAL — CLI]**
  - [ ] 7.1: Deploy with env vars:
    ```bash
    gcloud run deploy portfolio-platform \
      --image asia-southeast1-docker.pkg.dev/<PROJECT_ID>/portfolio-repo/portfolio-platform:latest \
      --region asia-southeast1 \
      --platform managed \
      --allow-unauthenticated \
      --memory 512Mi \
      --cpu 1 \
      --min-instances 0 \
      --max-instances 2 \
      --port 8080 \
      --set-env-vars="SPRING_DATASOURCE_URL=jdbc:postgresql://ep-xxx.ap-southeast-1.aws.neon.tech/neondb?sslmode=require" \
      --set-env-vars="SPRING_DATASOURCE_USERNAME=neondb_owner" \
      --set-env-vars="SPRING_DATASOURCE_PASSWORD=<neon-password>" \
      --set-env-vars="JWT_SECRET=$(openssl rand -base64 32)" \
      --set-env-vars="SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID=placeholder" \
      --set-env-vars="SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET=placeholder"
    ```
  - [ ] 7.2: Note the Cloud Run URL from output (e.g. `https://portfolio-platform-xxx-as.a.run.app`)
  - [ ] 7.3: Verify health:
    ```bash
    curl https://<cloud-run-url>/actuator/health
    # Expected: {"status":"UP"}
    ```
  - [ ] 7.4: Check logs for Flyway migrations:
    ```bash
    gcloud run services logs read portfolio-platform --region asia-southeast1 --limit 50
    ```

- [ ] Task 8: Create Service Account for CI/CD (AC: #6) **[MANUAL — CLI]**
  - [ ] 8.1: Create service account:
    ```bash
    gcloud iam service-accounts create portfolio-deployer \
      --display-name "Portfolio CI/CD Deployer"
    ```
  - [ ] 8.2: Grant required roles:
    ```bash
    PROJECT_ID=$(gcloud config get-value project)
    SA="portfolio-deployer@${PROJECT_ID}.iam.gserviceaccount.com"

    gcloud projects add-iam-policy-binding $PROJECT_ID \
      --member="serviceAccount:$SA" \
      --role="roles/run.admin"

    gcloud projects add-iam-policy-binding $PROJECT_ID \
      --member="serviceAccount:$SA" \
      --role="roles/artifactregistry.writer"

    gcloud projects add-iam-policy-binding $PROJECT_ID \
      --member="serviceAccount:$SA" \
      --role="roles/iam.serviceAccountUser"
    ```
  - [ ] 8.3: Create and download JSON key:
    ```bash
    gcloud iam service-accounts keys create sa-key.json \
      --iam-account=$SA
    ```
    > **NEVER commit sa-key.json** — add to .gitignore immediately
  - [ ] 8.4: Add GitHub secrets at repo → Settings → Secrets → Actions:
    - `GCP_PROJECT_ID` = output of `gcloud config get-value project`
    - `GCP_SA_KEY` = full content of `sa-key.json`
    - `NEON_DATABASE_URL` = JDBC connection string from Task 4.5

- [ ] Task 9: Clean up Oracle A1 artifacts (housekeeping)
  - [ ] 9.1: Delete obsolete Oracle deploy files:
    ```bash
    rm portfolio-platform/deploy/portfolio-platform.service
    rm portfolio-platform/deploy/deploy.sh
    rm -rf portfolio-platform/deploy/nginx/
    ```
  - [ ] 9.2: Add to `.gitignore`:
    ```
    sa-key.json
    ```
  - [ ] 9.3: Update smoke-tests.sh `BASE_URL` to Cloud Run URL
  - [ ] 9.4: Commit cleanup

## Dev Notes

### Why GCR + Neon

| | Oracle A1 | Fly.io | GCR + Neon |
|---|---|---|---|
| Cost | $0 | ~$5-6/mo | **$0** (free tier) |
| DB | Self-managed | Managed ($2-3) | **Neon free forever** |
| TLS | Nginx manual | Built-in | **Built-in** |
| Provisioning | Manual VM | `fly launch` | `gcloud run deploy` |
| Availability | ❌ No capacity | ✅ | ✅ |
| Credit card | ❌ Not needed | ✅ Required | ✅ Required (but $0) |

### GCR Free Tier (per month)

- **Requests**: 2,000,000 free
- **Compute**: 360,000 GB-seconds (512MB × ~700,000 seconds)
- **vCPU**: 180,000 vCPU-seconds
- **Outbound**: 1GB free (additional: $0.12/GB)

Portfolio traffic estimate: ~1,000 requests/month → **well within free tier**.

### Neon Free Tier

- **Storage**: 0.5GB (enough for portfolio data)
- **Compute**: 100 CU-hrs/month (0.25 CU default → 400 active hours)
- **Auto-suspend**: after 5 min idle → ~2-5 active hours/month for portfolio
- **No credit card**, no expiry, no manual unpausing needed

### Cold Start Consideration

GCR with `--min-instances 0` scales to zero → cold start ~8-12s for Spring Boot.
For portfolio this is acceptable. If needed, set `--min-instances 1` (~$4/month for 512MB always-on).

To minimize cold start time, add to `application-prod.yml`:
```yaml
spring:
  main:
    lazy-initialization: true
```

### Neon Connection Pooling

Neon suspends compute after 5 min idle. Spring Boot HikariCP will get connection errors
if it tries to use a stale connection after Neon wakes. The `keepalive-time: 30000` setting
in Task 5 prevents this by sending keepalive probes every 30s during active sessions.

### Story 2.4.1 Status: OBSOLETE

Cloud Run provides automatic TLS termination. No Nginx needed.

### Story 2.4.2 CI/CD — Must Be Rewritten

Replace Oracle SSH/SCP steps with:
```yaml
- name: Authenticate to GCP
  uses: google-github-actions/auth@v2
  with:
    credentials_json: ${{ secrets.GCP_SA_KEY }}

- name: Deploy to Cloud Run
  uses: google-github-actions/deploy-cloudrun@v2
  with:
    service: portfolio-platform
    region: asia-southeast1
    image: asia-southeast1-docker.pkg.dev/${{ secrets.GCP_PROJECT_ID }}/portfolio-repo/portfolio-platform:latest
```

### Dependency Map

```
Story 2.4.0 (this) ── GCR deployed, Neon attached, secrets set ──┐
                                                                    │
Story 2.4.2 (CI/CD) ── requires: GCP_PROJECT_ID, GCP_SA_KEY ─────┘
Story 2.4.1 (Nginx) ── OBSOLETE, skip
Story 2.4.3 (DB backup) ── requires: Neon running (pg_dump via Neon connection string)
```

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

### Completion Notes List

### File List

- `portfolio-platform/Dockerfile` ← already created in Fly.io attempt
- `portfolio-platform/.dockerignore` ← already created
- `portfolio-platform/src/main/resources/application-prod.yml` ← update datasource + hikari config
- `portfolio-platform/deploy/smoke-tests.sh` ← update BASE_URL to Cloud Run URL
- `portfolio-platform/deploy/portfolio-platform.service` ← DELETE
- `portfolio-platform/deploy/deploy.sh` ← DELETE
- `portfolio-platform/deploy/nginx/` ← DELETE
