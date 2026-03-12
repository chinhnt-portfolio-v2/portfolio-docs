# Story 2.4.0: Oracle A1 Server Provisioning

Status: ready-for-dev

## Story

As the developer,
I want a fully provisioned Oracle A1 server with Java 21, PostgreSQL, and Nginx installed and configured,
so that Stories 2.4.1 (Nginx config) and 2.4.2 (CI/CD pipeline) can deploy and run successfully.

## Acceptance Criteria

1. Given a new Oracle Cloud free-tier account, when an ARM A1 Compute instance is created, then the instance uses shape `VM.Standard.A1.Flex` (4 OCPU, 24GB RAM), runs Ubuntu 22.04 LTS, and is reachable via `ssh ubuntu@<ORACLE_HOST>` using the deploy key.

2. Given the server is provisioned, when software is installed, then `java -version` returns Java 21 (Temurin distribution), `psql --version` returns PostgreSQL 14+, and `nginx -v` returns Nginx 1.18+.

3. Given PostgreSQL is installed, when the database is configured, then a `portfolio_v2` database and `portfolio_user` role exist, `portfolio_user` can connect via `localhost` with password auth, and the user owns the `portfolio_v2` database.

4. Given the OCI Console and OS firewall, when both layers are configured, then inbound connections on ports 22 (SSH), 80 (HTTP), and 443 (HTTPS) succeed from the internet; port 8080 is blocked from internet but accessible on `localhost`.

5. Given the deploy directory is bootstrapped, when complete, then `/opt/portfolio-platform/` exists, is owned by `ubuntu`, contains `/opt/portfolio-platform/.env` with all required env vars, and the `portfolio-platform` systemd service is enabled (`sudo systemctl is-enabled portfolio-platform` → `enabled`).

6. Given GitHub secrets are configured at: **repo → Settings → Secrets → Actions**, then secrets `ORACLE_HOST`, `ORACLE_USER`, `ORACLE_SSH_KEY`, and `ORACLE_HOST_KEY` all exist and the CI/CD pipeline from Story 2.4.2 can SSH to Oracle A1 successfully.

7. Given the initial manual JAR deployment, when the service starts, then `curl http://localhost:8080/actuator/health` returns HTTP 200 from inside the server, confirming the Spring Boot app and PostgreSQL connection are both working.

## Tasks / Subtasks

- [ ] Task 1: Provision Oracle Cloud account and ARM A1 instance (AC: #1)
  - [ ] 1.1: Create Oracle Cloud free-tier account at `cloud.oracle.com` (requires credit card — will NOT be charged for always-free resources)
  - [ ] 1.2: Navigate to Compute → Instances → Create Instance
  - [ ] 1.3: Select shape `VM.Standard.A1.Flex` → set OCPU=4, RAM=24GB (always-free allowance)
  - [ ] 1.4: Select image: Canonical Ubuntu 22.04 (aarch64/ARM64)
  - [ ] 1.5: Upload your SSH public key (or generate one; download private key — this is your access key)
  - [ ] 1.6: Wait for instance to reach Running state; note the public IP address
  - [ ] 1.7: Verify SSH: `ssh ubuntu@<PUBLIC_IP>` works with the key from 1.5

- [ ] Task 2: Configure OCI Console Security List (firewall layer 1) (AC: #4)
  - [ ] 2.1: Navigate to Networking → VCNs → your VCN → Security Lists → Default Security List
  - [ ] 2.2: Add Ingress Rule: Source `0.0.0.0/0`, Protocol TCP, Destination Port `80`
  - [ ] 2.3: Add Ingress Rule: Source `0.0.0.0/0`, Protocol TCP, Destination Port `443`
  - [ ] 2.4: Verify port 22 ingress rule already exists (added by default during instance creation)
  - [ ] 2.5: Do NOT add port 8080 to Security List — Spring Boot port is internal only

- [ ] Task 3: Configure OS-level firewall (iptables) (AC: #4)
  - [ ] 3.1: SSH into Oracle A1; check current rules: `sudo iptables -L INPUT -n --line-numbers`
  - [ ] 3.2: Allow HTTP: `sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT`
  - [ ] 3.3: Allow HTTPS: `sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT`
  - [ ] 3.4: Persist rules: `sudo netfilter-persistent save` (install if missing: `sudo apt install iptables-persistent -y`)
  - [ ] 3.5: Verify port 8080 is NOT open externally (`nmap -p 8080 <PUBLIC_IP>` from local machine → filtered)

- [ ] Task 4: Install base software (AC: #2)
  - [ ] 4.1: Update packages: `sudo apt update && sudo apt upgrade -y`
  - [ ] 4.2: Install Java 21 Temurin (ARM64):
    ```bash
    sudo apt install -y wget apt-transport-https gnupg
    wget -qO - https://packages.adoptium.net/artifactory/api/gpg/key/public | sudo tee /etc/apt/trusted.gpg.d/adoptium.asc
    echo "deb https://packages.adoptium.net/artifactory/deb $(lsb_release -cs) main" | sudo tee /etc/apt/sources.list.d/adoptium.list
    sudo apt update
    sudo apt install -y temurin-21-jdk
    ```
  - [ ] 4.3: Verify: `java -version` → shows `openjdk 21.x.x ... Temurin`
  - [ ] 4.4: Install PostgreSQL: `sudo apt install -y postgresql postgresql-contrib`
  - [ ] 4.5: Start and enable PostgreSQL: `sudo systemctl enable --now postgresql`
  - [ ] 4.6: Install Nginx: `sudo apt install -y nginx`
  - [ ] 4.7: Enable Nginx (do NOT start yet — config from Story 2.4.1 must be applied first): `sudo systemctl enable nginx`

- [ ] Task 5: Configure PostgreSQL (AC: #3)
  - [ ] 5.1: Connect as postgres superuser: `sudo -u postgres psql`
  - [ ] 5.2: Create database user:
    ```sql
    CREATE USER portfolio_user WITH PASSWORD '<secure-password>';
    ```
  - [ ] 5.3: Create database:
    ```sql
    CREATE DATABASE portfolio_v2 OWNER portfolio_user;
    GRANT ALL PRIVILEGES ON DATABASE portfolio_v2 TO portfolio_user;
    \q
    ```
  - [ ] 5.4: Verify connection: `psql -h localhost -U portfolio_user -d portfolio_v2 -c "SELECT 1;"` → returns `1`
  - [ ] 5.5: Confirm PostgreSQL listens on localhost only (NOT 0.0.0.0): `sudo -u postgres psql -c "SHOW listen_addresses;"` → `localhost`

- [ ] Task 6: Bootstrap deploy directory and systemd service (AC: #5)
  - [ ] 6.1: Create deploy directory: `sudo mkdir -p /opt/portfolio-platform && sudo chown ubuntu:ubuntu /opt/portfolio-platform`
  - [ ] 6.2: From local machine, SCP the systemd service file:
    ```bash
    scp portfolio-platform/deploy/portfolio-platform.service ubuntu@<ORACLE_HOST>:/tmp/portfolio-platform.service
    ```
  - [ ] 6.3: On Oracle A1, install the service:
    ```bash
    sudo cp /tmp/portfolio-platform.service /etc/systemd/system/portfolio-platform.service
    sudo systemctl daemon-reload
    sudo systemctl enable portfolio-platform
    ```
  - [ ] 6.4: Create `/opt/portfolio-platform/.env` on Oracle A1 with real values (see template below — NEVER commit this file):
    ```bash
    nano /opt/portfolio-platform/.env
    ```
  - [ ] 6.5: Verify service is enabled: `sudo systemctl is-enabled portfolio-platform` → `enabled`

- [ ] Task 7: Generate and configure GitHub deploy keys (AC: #6)
  - [ ] 7.1: On local machine, generate dedicated deploy key:
    ```bash
    ssh-keygen -t ed25519 -C "github-actions-deploy" -f ~/.ssh/oracle_deploy_key -N ""
    ```
  - [ ] 7.2: Add public key to Oracle A1:
    ```bash
    cat ~/.ssh/oracle_deploy_key.pub | ssh ubuntu@<ORACLE_HOST> "cat >> ~/.ssh/authorized_keys"
    ```
  - [ ] 7.3: Test deploy key works: `ssh -i ~/.ssh/oracle_deploy_key ubuntu@<ORACLE_HOST> "echo OK"` → `OK`
  - [ ] 7.4: Get Oracle A1 host key for MITM prevention:
    ```bash
    ssh-keyscan <ORACLE_HOST>
    # Copy the ssh-ed25519 line (e.g.: "152.xx.xx.xx ssh-ed25519 AAAA...")
    ```
  - [ ] 7.5: Add GitHub secrets at: **repo → Settings → Secrets and variables → Actions → New repository secret**:
    - `ORACLE_HOST` = Public IP of Oracle A1 instance
    - `ORACLE_USER` = `ubuntu`
    - `ORACLE_SSH_KEY` = Full content of `~/.ssh/oracle_deploy_key` (private key, including header/footer)
    - `ORACLE_HOST_KEY` = Full output line from step 7.4 (e.g., `152.xx.xx.xx ssh-ed25519 AAAA...`)

- [ ] Task 8: Initial manual deployment to verify full stack (AC: #7)
  - [ ] 8.1: On local machine, build the JAR:
    ```bash
    cd portfolio-platform
    ./mvnw clean package -DskipTests
    ```
  - [ ] 8.2: SCP JAR and smoke tests to Oracle A1:
    ```bash
    scp target/portfolio-platform-0.0.1-SNAPSHOT.jar ubuntu@<ORACLE_HOST>:/opt/portfolio-platform/portfolio-platform.jar
    scp deploy/smoke-tests.sh ubuntu@<ORACLE_HOST>:/opt/portfolio-platform/smoke-tests.sh
    ssh ubuntu@<ORACLE_HOST> "chmod +x /opt/portfolio-platform/smoke-tests.sh"
    ```
  - [ ] 8.3: Start the service: `ssh ubuntu@<ORACLE_HOST> "sudo systemctl start portfolio-platform"`
  - [ ] 8.4: Check logs for startup errors: `ssh ubuntu@<ORACLE_HOST> "sudo journalctl -u portfolio-platform -n 50 --no-pager"`
  - [ ] 8.5: Run smoke tests: `ssh ubuntu@<ORACLE_HOST> "bash /opt/portfolio-platform/smoke-tests.sh"` → "All smoke tests passed."
  - [ ] 8.6: Verify service stays running after 30s: `ssh ubuntu@<ORACLE_HOST> "sudo systemctl status portfolio-platform"` → `active (running)`

## Dev Notes

### This Story is Manual Operations — Not Code

This story has **no source code changes**. All tasks are manual shell commands and OCI Console clicks. The "dev agent" executing this story is the human operator (Chính) via SSH and browser.

- No JUnit tests required
- No Java source changes
- No CI/CD changes (those are in Story 2.4.2)
- Completion verified by smoke tests passing (AC #7)

### Oracle A1 Always-Free Tier Specs

```
Shape:   VM.Standard.A1.Flex
OCPUs:   4 (ARM Cortex-A72 cores)
RAM:     24 GB
Storage: 200 GB block storage (always-free)
Network: 10 Gbps, 10 TB/month outbound free
Cost:    $0/month (permanent always-free)
Region:  Choose closest to Vietnam (e.g., ap-singapore-1, ap-tokyo-1, ap-sydney-1)
```

**Important:** Always-free resources are region-specific. Once a region is selected for your account, you cannot move the instance. Choose wisely.

### Software Versions to Install

| Software | Version | Method |
|---|---|---|
| Java | 21 LTS (Temurin) | Eclipse Adoptium apt repo |
| PostgreSQL | 14+ (Ubuntu 22.04 ships 14) | Ubuntu apt |
| Nginx | 1.18+ | Ubuntu apt |
| OS | Ubuntu 22.04 LTS (Jammy) | OCI image |

**Do NOT use Oracle JDK** — requires license agreement. Temurin (Eclipse Adoptium) is the community distribution of OpenJDK and is free/production-grade.

### Environment File Template

Create at `/opt/portfolio-platform/.env` — **NEVER commit this file to git**:

```bash
# /opt/portfolio-platform/.env
# Loaded by systemd EnvironmentFile directive
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/portfolio_v2
SPRING_DATASOURCE_USERNAME=portfolio_user
SPRING_DATASOURCE_PASSWORD=<choose-a-strong-password>
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_ID=<google-oauth-client-id>
SPRING_SECURITY_OAUTH2_CLIENT_REGISTRATION_GOOGLE_CLIENT_SECRET=<google-oauth-secret>
JWT_SECRET=<base64-encoded-256-bit-random-string>
```

Generate JWT_SECRET:
```bash
openssl rand -base64 32
```

Google OAuth2 credentials: create at `console.cloud.google.com` → APIs & Services → Credentials → OAuth 2.0 Client IDs. Required for Stories 5.2–5.4 but the placeholder must exist now to prevent Spring Boot startup failure.

### OCI Dual-Layer Firewall Architecture

Oracle A1 has two independent firewalls — **both** must allow a port for traffic to flow:

```
Internet → OCI VCN Security List (Layer 1, cloud-level)
         → OS iptables (Layer 2, OS-level)
         → Service (Nginx :80/:443 or Spring Boot :8080)
```

Story 2.4.1 learnings confirmed: missing either layer causes silent connection timeout. Port 8080 stays internal (only iptables localhost allows it via Nginx proxy).

### Spring Boot Profile

The systemd service launches with `-Dspring.profiles.active=prod`. This maps to `application-prod.yml` in the JAR. Profile names are:
- **`local`** — local development (NOT `dev`)
- **`prod`** — Oracle A1 production (NOT `production`)

[Source: Story 2.3 and 2.4.1 learnings — architecture.md has outdated profile names `dev`/`prod`; actual implementation confirmed `local`/`prod` only]

### PostgreSQL Connection

Spring Boot connects via `localhost:5432` — PostgreSQL is co-located, never exposed to internet. Flyway runs database migrations automatically on startup. After Task 8, Flyway will create the schema from `src/main/resources/db/migration/`.

### Project Structure After This Story

No new files in the codebase. After this story, the following server-side state exists:

```
Oracle A1 (Ubuntu 22.04 ARM64):
├── /opt/portfolio-platform/
│   ├── portfolio-platform.jar      ← manually deployed JAR (will be CI/CD after 2.4.2)
│   ├── smoke-tests.sh              ← from deploy/smoke-tests.sh
│   └── .env                        ← secrets (NEVER in git)
├── /etc/systemd/system/
│   └── portfolio-platform.service  ← from deploy/portfolio-platform.service
├── /etc/nginx/                     ← base install; config from Story 2.4.1
└── PostgreSQL:
    ├── database: portfolio_v2
    └── user: portfolio_user
```

### Dependency Map

```
Story 2.4.0 (this) ─── must be done first ──────────────────────────────────────┐
                                                                                  │
Story 2.4.1 (Nginx config) ── requires: server exists, Nginx installed ──────────┤
Story 2.4.2 (CI/CD) ─────── requires: GitHub secrets, deploy dir bootstrapped ──┘
Story 2.4.3 (DB backup) ─── requires: PostgreSQL running, cron available
```

### Cross-Story Coordination

- **Story 2.4.1:** Nginx config (`deploy/nginx/portfolio-v2.nginx.conf`) is committed to repo but not yet active on server. After this story, run Story 2.4.1 tasks manually to apply Nginx config and get SSL cert.
- **Story 2.4.2:** CI/CD pipeline is committed to repo. After GitHub secrets are added (Task 7), pushing to `main` will trigger auto-deploy.
- **Story 2.4.3:** Database backup cron job — needs PostgreSQL running and Oracle Object Storage bucket.

### References

- Oracle A1 always-free: [Source: architecture.md#Hosting Strategy]
- Java 21, Spring Boot profiles: [Source: architecture.md#Selected Starter: Platform BE]
- Dual-layer firewall: [Source: Story 2.4.1 learnings — "Oracle A1 dual-layer firewall: OCI Console Security List + OS iptables"]
- systemd unit template: [Source: architecture.md#deploy/portfolio-platform.service template (~line 1443)]
- Deploy bootstrap steps: [Source: portfolio-platform/deploy/README.md#One-Time Oracle A1 Bootstrap]
- GitHub Secrets required: [Source: portfolio-platform/deploy/README.md#GitHub Secrets Setup]
- NFR-O5: Infrastructure cost $0/month: [Source: epics.md#NFR-O5]

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

### Completion Notes List

### File List
