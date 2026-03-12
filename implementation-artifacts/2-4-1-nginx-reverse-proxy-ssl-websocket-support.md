# Story 2.4.1: Nginx Reverse Proxy, SSL & WebSocket Support

Status: done

## Story

As the Platform BE owner,
I want Nginx properly configured as a reverse proxy with SSL termination and WebSocket support on Oracle A1,
so that the Spring Boot backend is securely accessible over HTTPS and WebSocket connections work end-to-end for the live metrics layer.

## Acceptance Criteria

1. Given Nginx is installed on Oracle A1, when a request arrives at port 443, then Nginx terminates TLS (certificate via Let's Encrypt/certbot) and proxies to Spring Boot on `localhost:8080`.

2. Given a WebSocket upgrade request arrives at `/ws/`, when Nginx processes it, then Nginx passes `Upgrade` and `Connection` headers correctly — WebSocket connections succeed without application-layer changes.

3. Given HTTP requests arrive on port 80, when processed, then Nginx returns 301 redirect to HTTPS for all paths.

4. Given the config is applied, when `sudo nginx -t` is run, then validation passes with no errors; `proxy_http_version 1.1`, `proxy_set_header Upgrade $http_upgrade`, and `proxy_set_header Connection` directives are present in the WebSocket location block.

5. Given Oracle A1 has both OCI Security List and OS iptables configured, when ports 80 and 443 are tested from external, then both ports are reachable from the internet.

6. Given Nginx is running, when `curl -I https://<domain>/actuator/health` is executed, then response is HTTP 200 (or 401 if auth active) with valid TLS and no certificate errors.

7. Given rate limiting is configured, when an IP exceeds the zone limit, then Nginx returns HTTP 429 (not 503).

## Tasks / Subtasks

- [ ] Task 1: Configure Oracle Cloud A1 firewall for ports 80/443 (AC: #5) **[MANUAL — run on Oracle A1 server]**
  - [ ] 1.1: Add ingress rules in OCI Console Security List for TCP 80 and TCP 443 from 0.0.0.0/0
  - [ ] 1.2: Add OS-level iptables rules (insert before REJECT rule at position 6)
  - [ ] 1.3: Persist iptables rules via `iptables-persistent` / `netfilter-persistent save`

- [ ] Task 2: Install Nginx and Certbot on Oracle A1 (AC: #1, #4) **[MANUAL — run on Oracle A1 server]**
  - [ ] 2.1: Install Nginx via apt: `sudo apt install nginx`
  - [ ] 2.2: Install Certbot via snap: `sudo snap install --classic certbot && sudo ln -s /snap/bin/certbot /usr/local/bin/certbot`
  - [ ] 2.3: Obtain Let's Encrypt certificate for domain: `sudo certbot certonly --nginx -d <domain>`
  - [ ] 2.4: Verify systemd timer for auto-renewal: `sudo systemctl status snap.certbot.renew.timer`
  - [ ] 2.5: Test renewal: `sudo certbot renew --dry-run`
  - [ ] 2.6: Add post-renewal hook to reload Nginx: `/etc/letsencrypt/renewal-hooks/post/reload-nginx.sh`

- [x] Task 3: Create Nginx site configuration (AC: #1, #2, #3, #4, #7)
  - [x] 3.1: Add `map $http_upgrade $connection_upgrade` block and `limit_req_zone` definitions in `/etc/nginx/nginx.conf` http block
  - [x] 3.2: Create `/etc/nginx/sites-available/portfolio-v2` with HTTP→HTTPS redirect server block
  - [x] 3.3: Add HTTPS server block with SSL cert/key paths, TLS 1.2+1.3 protocols, ciphers, security headers
  - [x] 3.4: Add `/api/` location block with `limit_req` (api_general zone) and `proxy_pass http://127.0.0.1:8080`
  - [x] 3.5: Add `/ws/` location block with WebSocket-critical directives (`proxy_http_version 1.1`, `Upgrade`, `Connection`, `proxy_buffering off`, `proxy_read_timeout 3600s`)
  - [x] 3.6: Symlink config: `sudo ln -s /etc/nginx/sites-available/portfolio-v2 /etc/nginx/sites-enabled/portfolio-v2` (documented in config header)

- [x] Task 4: Update Spring Boot application config for reverse proxy (AC: #1, #6)
  - [x] 4.1: Add `server.forward-headers-strategy=FRAMEWORK` to prod profile in `application.yml`
  - [x] 4.2: Add `server.tomcat.remoteip.internal-proxies=127\\.0\\.0\\.1` to restrict trusted proxy to localhost Nginx
  - [x] 4.3: Verify `server.port=8080` is present in `application.yml` — confirmed present

- [ ] Task 5: Validate and test (AC: #4, #5, #6) **[MANUAL — run on Oracle A1 server after Tasks 1 & 2]**
  - [ ] 5.1: Run `sudo nginx -t` — must pass with no errors
  - [ ] 5.2: Run `sudo systemctl reload nginx`
  - [ ] 5.3: Test HTTP redirect: `curl -I http://<domain>/` → HTTP 301 Location: https://
  - [ ] 5.4: Test HTTPS proxy: `curl -I https://<domain>/actuator/health` → HTTP 200 or 401
  - [ ] 5.5: Test WebSocket upgrade manually (Browser DevTools → Network → WS tab, connect to `wss://<domain>/ws/metrics`)
  - [ ] 5.6: Test rate limiting: rapid curl loop to `/api/` → HTTP 429 after limit exceeded

- [x] Task 6: Store Nginx config in repository (AC: #4)
  - [x] 6.1: Create `portfolio-platform/deploy/nginx/portfolio-v2.nginx.conf` in the repo as the canonical reference copy
  - [x] 6.2: Add inline comments documenting every non-obvious directive

## Dev Notes

### Critical Architecture Contract

**This story is a hard blocker for all of Epic 3 (Live Evidence Layer).**
- Story 3.3 (WebSocket Server BE) and 3.4 (WebSocket Client FE) cannot function without the Nginx WebSocket upgrade configuration in this story.
- The FE will connect to `wss://<domain>/ws/metrics` — Nginx MUST upgrade HTTP/1.1 to WebSocket protocol at the `/ws/` location.
- Missing `proxy_http_version 1.1` + `Upgrade`/`Connection` headers = WebSocket silently fails with 502/400.

### Oracle Cloud A1 Dual-Layer Firewall (MANDATORY)

Oracle Cloud has **two independent firewall layers** — both must be open or traffic is blocked:

**Layer 1 — OCI Console Security List (Ingress Rules):**

Navigate: OCI Console → Networking → Virtual Cloud Networks → your VCN → Security Lists → Add Ingress Rules:

| Source CIDR | Protocol | Dest Port | Description |
|---|---|---|---|
| 0.0.0.0/0 | TCP | 22 | SSH |
| 0.0.0.0/0 | TCP | 80 | HTTP |
| 0.0.0.0/0 | TCP | 443 | HTTPS |

**Layer 2 — OS iptables (Ubuntu default blocks everything except SSH):**

Ubuntu images on OCI ship with a `REJECT` rule at position 6+. New rules must be inserted *before* it:

```bash
# Open HTTP and HTTPS
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
sudo iptables -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT

# Persist across reboots
sudo apt install -y iptables-persistent
sudo netfilter-persistent save
```

> **Do NOT use UFW on Oracle Cloud** — it conflicts with OCI's iptables chain management.
>
> If `Couldn't load match 'state'` error appears (nf_tables conflict), use `iptables-legacy` instead:
> ```bash
> sudo iptables-legacy -I INPUT 6 -m state --state NEW -p tcp --dport 80 -j ACCEPT
> sudo iptables-legacy -I INPUT 6 -m state --state NEW -p tcp --dport 443 -j ACCEPT
> sudo iptables-legacy-save | sudo tee /etc/iptables/rules.v4
> ```

### Nginx Configuration — Complete Reference

**Step 1: Add to `/etc/nginx/nginx.conf` inside the `http {}` block:**

```nginx
# WebSocket upgrade map — must be in http block, NOT server block
map $http_upgrade $connection_upgrade {
    default upgrade;
    ''      close;
}

# Rate limiting zones (defined once, applied per location)
limit_req_zone $binary_remote_addr zone=api_general:10m rate=20r/s;
limit_req_zone $binary_remote_addr zone=api_auth:10m    rate=5r/m;
limit_req_zone $binary_remote_addr zone=ws_connect:10m  rate=10r/m;

limit_req_status 429;  # RFC-correct: rate limited = 429, not 503
server_tokens off;     # Hide Nginx version from response headers
```

**Step 2: Create `/etc/nginx/sites-available/portfolio-v2`:**

```nginx
# HTTP -> HTTPS redirect (all paths, permanent)
server {
    listen 80;
    listen [::]:80;
    server_name <domain>;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl;
    listen [::]:443 ssl;
    http2 on;   # Nginx 1.25.1+ syntax; use 'listen 443 ssl http2;' if older
    server_name <domain>;

    # SSL — Let's Encrypt certificate paths
    ssl_certificate     /etc/letsencrypt/live/<domain>/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/<domain>/privkey.pem;

    # TLS 1.2 + 1.3 (intermediate compatibility profile — supports all modern browsers)
    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers "ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:DHE-RSA-AES128-GCM-SHA256";
    ssl_prefer_server_ciphers off;   # Let client pick best TLS 1.3 cipher
    ssl_ecdh_curve X25519:prime256v1:secp384r1;

    ssl_session_cache   shared:SSL:10m;
    ssl_session_timeout 1d;
    ssl_session_tickets off;

    # NOTE: Let's Encrypt DISCONTINUED OCSP stapling in 2025 — do NOT add ssl_stapling

    # Security headers
    add_header Strict-Transport-Security "max-age=63072000; includeSubDomains; preload" always;
    add_header X-Content-Type-Options    "nosniff" always;
    add_header X-Frame-Options           "DENY" always;
    add_header Referrer-Policy           "strict-origin-when-cross-origin" always;
    add_header Permissions-Policy        "geolocation=(), microphone=(), camera=()" always;

    # Shared proxy headers (re-declared in WS block too since location inherits but WS needs explicit)
    proxy_set_header Host              $host;
    proxy_set_header X-Real-IP         $remote_addr;
    proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;

    # REST API — general endpoints
    location /api/ {
        limit_req zone=api_general burst=40 nodelay;

        proxy_pass http://127.0.0.1:8080;
        proxy_read_timeout    60s;
        proxy_connect_timeout 10s;
        proxy_send_timeout    60s;
    }

    # REST API — auth endpoints (stricter rate limit)
    location /api/auth/ {
        limit_req zone=api_auth burst=10 nodelay;

        proxy_pass http://127.0.0.1:8080;
        proxy_read_timeout    30s;
        proxy_connect_timeout 10s;
    }

    # Phase 2 AI endpoint placeholder (commented out until Epic 7)
    # location /api/ai/ {
    #     limit_req zone=api_ai burst=2 nodelay;
    #     proxy_pass http://127.0.0.1:8080;
    # }

    # WebSocket endpoint — CRITICAL: all sub-paths under /ws/ must be covered
    # SockJS uses paths like: /ws/info, /ws/<server>/<session>/websocket
    location /ws/ {
        limit_req zone=ws_connect burst=20 nodelay;

        proxy_pass http://127.0.0.1:8080;

        # CRITICAL: HTTP/1.0 does NOT support Connection: Upgrade — must use 1.1
        proxy_http_version 1.1;

        # CRITICAL: Pass WebSocket upgrade handshake headers to backend
        proxy_set_header Upgrade    $http_upgrade;
        proxy_set_header Connection $connection_upgrade;  # Uses map above

        # Re-declare forwarding headers (location block resets inherited values)
        proxy_set_header Host              $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;

        # Extended timeout for long-lived WebSocket connections
        # Default 60s causes drop of idle connections — use 1h
        proxy_read_timeout    3600s;
        proxy_send_timeout    3600s;
        proxy_connect_timeout 10s;

        # Disable buffering — WebSocket frames must flow immediately
        proxy_buffering off;
    }

    # Fallback — proxy all other paths to Spring Boot
    location / {
        proxy_pass http://127.0.0.1:8080;
        proxy_read_timeout    60s;
        proxy_connect_timeout 10s;
    }

    # Certbot ACME challenge (for renewal without downtime)
    location /.well-known/acme-challenge/ {
        root /var/www/certbot;
    }
}
```

**Step 3: Enable the site:**

```bash
sudo ln -s /etc/nginx/sites-available/portfolio-v2 /etc/nginx/sites-enabled/portfolio-v2

# Test config syntax
sudo nginx -t

# Apply config without downtime
sudo systemctl reload nginx
```

### Certbot / Let's Encrypt Setup

```bash
# Install Certbot via snap (always current, self-updating)
sudo snap install --classic certbot
sudo ln -s /snap/bin/certbot /usr/local/bin/certbot

# Obtain certificate (certonly = don't auto-edit nginx config)
sudo certbot certonly --nginx -d <domain>

# Verify auto-renewal timer (snap installs this automatically)
sudo systemctl status snap.certbot.renew.timer

# Test renewal dry-run
sudo certbot renew --dry-run

# Add post-renewal hook to reload Nginx after cert rotation
sudo tee /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh << 'EOF'
#!/bin/bash
nginx -t && systemctl reload nginx
EOF
sudo chmod +x /etc/letsencrypt/renewal-hooks/post/reload-nginx.sh
```

> Certificates renew automatically when within 30 days of expiry. The snap systemd timer runs twice daily. No cron entry needed.

### Spring Boot Config for Reverse Proxy

Add to `portfolio-platform/src/main/resources/application-prod.yml`:

```yaml
server:
  port: 8080
  forward-headers-strategy: FRAMEWORK  # Installs ForwardedHeaderFilter to handle X-Forwarded-* headers
  tomcat:
    remoteip:
      internal-proxies: 127\\.0\\.0\\.1  # Trust ONLY localhost Nginx — prevents X-Forwarded-For spoofing
      remote-ip-header: x-forwarded-for
      protocol-header: x-forwarded-proto
    redirect-context-root: false
```

> `FRAMEWORK` strategy: installs Spring's `ForwardedHeaderFilter` as a servlet filter. Correctly rewrites scheme, host, port from `X-Forwarded-*` headers so generated redirect URLs, security headers, and links use `https://` not `http://`.
>
> `internal-proxies` restriction is a security requirement: without it, malicious clients can spoof `X-Forwarded-For` to impersonate trusted IPs.

### Repository Layout for Nginx Config

Store the canonical Nginx config in the repo for version control and CI/CD use (Story 2.4.2 will reference it):

```
portfolio-platform/
└── deploy/
    └── nginx/
        └── portfolio-v2.nginx.conf   ← canonical config, deployed via CI/CD SCP
```

The CI/CD pipeline (Story 2.4.2) will SCP this file to `/etc/nginx/sites-available/portfolio-v2` on the A1 instance during deployment.

### WebSocket Directive Reference (Why Each Is Required)

| Directive | Required Because |
|---|---|
| `proxy_http_version 1.1` | HTTP/1.0 cannot carry `Connection: Upgrade` — WebSocket upgrade fails silently |
| `proxy_set_header Upgrade $http_upgrade` | Without this, the `Upgrade: websocket` header is stripped by Nginx |
| `proxy_set_header Connection $connection_upgrade` | The `map` block outputs `"upgrade"` for WS or `"close"` for regular HTTP |
| `proxy_buffering off` | Nginx buffering breaks WebSocket frame streaming — frames would only arrive at buffer flush |
| `proxy_read_timeout 3600s` | Default 60s drops idle-but-alive WebSocket connections (e.g., user with no metric updates) |

### Rate Limiting Zones (NFR-S9)

Three zones required per PRD NFR-S9:

| Zone | Rate | Burst | Applied To |
|---|---|---|---|
| api_general | 20r/s (~100 req/5s) | 40 | `/api/` general endpoints |
| api_auth | 5r/m | 10 | `/api/auth/` login/register |
| ws_connect | 10r/m | 20 | `/ws/` WebSocket upgrade handshakes |

> Phase 2 AI zone (5 req/10min/IP) is commented-out as a placeholder — will be activated in Epic 7.

### Spring Boot Dependencies Already Present

No new dependencies needed for Story 2.4.1. `spring-boot-starter-websocket` is already in `pom.xml` from Story 2.1.

### Paths to Modify

| File | Change |
|---|---|
| `/etc/nginx/nginx.conf` (on A1 server) | Add `map` block and `limit_req_zone` directives to `http {}` block |
| `/etc/nginx/sites-available/portfolio-v2` (on A1 server) | Create — full config per template above |
| `portfolio-platform/src/main/resources/application-prod.yml` | Add `server.forward-headers-strategy`, `server.tomcat.remoteip.*` |
| `portfolio-platform/deploy/nginx/portfolio-v2.nginx.conf` | Create — canonical copy for version control |

### Key Constraint: Port is Fixed

Per architecture decision log: Spring Boot BE listens on **port 8080** — not configurable. All Nginx `proxy_pass` directives MUST target `http://127.0.0.1:8080`.

### Cross-Story Impact

- **Story 2.4.2 (CI/CD):** Will SCP `deploy/nginx/portfolio-v2.nginx.conf` and `sudo systemctl reload nginx` as part of deployment pipeline. Coordinate config file path.
- **Story 2.4.3 (DB Backup):** No Nginx interaction — independent cron job.
- **Story 3.3 (WebSocket Server BE):** Registers STOMP/SockJS endpoint at `/ws`. Nginx location `/ws/` must cover all SockJS sub-paths (`/ws/info`, `/ws/<server>/<session>/websocket`).
- **Story 3.4 (WebSocket Client FE):** FE connects to `wss://<domain>/ws/metrics`. TLS certificate must be valid (not self-signed) for `wss://` to work in browsers.

### Previous Story Learnings (Story 2.3)

- **Spring Boot version is 3.5.11.RELEASE** (important: use 3.5.x docs, not 3.4.x)
- **Spring profiles are `local` / `prod`** — NOT `dev` / `prod`. Always edit `application-prod.yml`, not `application-dev.yml`
- **`SecurityConfig`** already has `permitAll()` for `/actuator/health`, `/api-docs/**`, `/swagger-ui/**` — no changes to SecurityConfig needed for Nginx passthrough

### Project Structure Notes

- Backend root: `I:/portfolio-v2/portfolio-platform/`
- Application config: `src/main/resources/application.yml`, `application-local.yml`, `application-prod.yml`
- SecurityConfig: `src/main/java/dev/chinh/portfolio/shared/config/SecurityConfig.java`
- pom.xml: Spring Boot 3.5.11, Java 21, `spring-boot-starter-websocket` already present
- Deploy directory (create): `portfolio-platform/deploy/nginx/`

### References

- Architecture WebSocket-Nginx contract: [Source: _bmad-output/planning-artifacts/architecture.md#WebSocket Infrastructure]
- NFR-S9 rate limiting requirements: [Source: _bmad-output/planning-artifacts/prd.md#Non-Functional Requirements]
- Epic 2 Story 2.4.1 AC: [Source: _bmad-output/planning-artifacts/epics.md#Epic 2 Story 2.4.1]
- Oracle A1 dual-layer firewall: [Source: https://blogs.oracle.com/developers/enabling-network-traffic-to-ubuntu-images-in-oracle-cloud-infrastructure]
- Nginx WebSocket proxy: [Source: https://nginx.org/en/docs/http/websocket.html]
- Let's Encrypt OCSP discontinuation 2025: [Source: https://letsencrypt.org/docs/]
- Spring Boot forward headers: [Source: Spring Boot 3.5.x Docs#Reverse Proxy]

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- `ForwardedHeaderFilter` in Spring Boot 3.x is registered as `FilterRegistrationBean<ForwardedHeaderFilter>`, not directly as `ForwardedHeaderFilter` bean. Test must use `getBeansOfType(FilterRegistrationBean.class)` and check `.getFilter() instanceof ForwardedHeaderFilter`.

### Completion Notes List

- Tasks 3, 4, 6 implemented (code artifacts): Nginx config created in repo, Spring Boot prod profile updated with `server.forward-headers-strategy=FRAMEWORK` + `server.tomcat.remoteip.*`.
- Tasks 1, 2, 5 are manual ops tasks requiring SSH access to Oracle A1 server — documented with `[MANUAL]` markers. To be completed by owner when server is provisioned.
- 5 tests total (`ReverseProxyConfigTest`): verifies `FilterRegistrationBean<ForwardedHeaderFilter>` present, bean name stable, `/actuator/health` responds with and without `X-Forwarded-*` headers, `internal-proxies` is non-wildcard.
- Full suite: 20/20 pass, 0 regressions.
- Code review fixes applied: `http2 on;` → `listen 443 ssl http2;` (compatibility), `api_general` rate → `100r/m` (NFR-S9), CSP header added, `X-Frame-Options` → SAMEORIGIN, `proxy_send_timeout` added to `/api/auth/` and `/`, misleading prefix-ordering comment fixed, `internalProxies` security test added.

### File List

- `portfolio-platform/deploy/nginx/portfolio-v2.nginx.conf` (created)
- `portfolio-platform/src/main/resources/application.yml` (modified — prod profile)
- `portfolio-platform/src/test/java/dev/chinh/portfolio/ReverseProxyConfigTest.java` (created)
