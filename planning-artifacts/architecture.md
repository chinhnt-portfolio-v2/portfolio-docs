---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
lastStep: 8
status: 'complete'
completedAt: '2026-03-03'
inputDocuments:
  - "_bmad-output/planning-artifacts/prd.md"
  - "_bmad-output/planning-artifacts/ux-design-specification.md"
  - "_bmad-output/brainstorming/brainstorming-session-2026-02-25.md"
workflowType: 'architecture'
project_name: 'portfolio-v2'
user_name: 'Chính'
date: '2026-03-03'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**

51 FRs tổ chức thành 9 nhóm chức năng:

1. **Showcase & Discovery (FR1–FR8, FR43):** Projects-first layout, gallery cards, artist
   statements, skills-with-proof, lessons learned, build timeline, job availability, shareable
   project detail pages. Architectural implication: component-heavy SPA với route `/projects/{slug}`,
   lazy-loaded detail pages.

2. **Live Evidence Layer (FR9–FR13):** Real-time health metrics, staleness indicators, auto-hide
   logic, live GitHub graph, Lighthouse badge. Architectural implication: WebSocket subscriber trên
   FE, graceful degradation logic bắt buộc.

3. **Recruiter Engagement & Personalization (FR14–FR17, FR46):** Contact form, success animation,
   URL referral tracking (`?from=`), returning visitor experience, contextual messaging.
   Architectural implication: localStorage state machine, URL param persistence.

4. **Navigation, Accessibility & Appearance (FR18–FR23, FR44–FR47):** Keyboard nav, WCAG 2.1 AA,
   multi-device, command palette (Phase 2), video walkthroughs (Phase 2), dark/light mode, spring
   physics animations. Architectural implication: global motion context, accessible component
   primitives từ shadcn/ui.

5. **Internationalization (FR41–FR42):** vi/en toggle, language persistence, smart `?from=`-based
   default detection. Architectural implication: i18n context at app root, URL-param driven language
   selection.

6. **Platform Auth & Identity (FR24–FR29):** JWT auth (15m access/7d refresh), Google OAuth2,
   auto-refresh, logout, user data isolation, public JWKS endpoint. Architectural implication:
   3 auth flows (password, OAuth2, JWKS) trong một Spring Security config.

7. **Metrics Platform & Extensibility (FR30–FR34, FR48):** Polling pipeline (60s), WebSocket
   delivery (<3s), GitHub webhook integration, config-driven app registration (zero code change),
   HMAC-SHA256 webhook verification. Architectural implication: scheduler + WebSocket server +
   webhook endpoint designed độc lập; config file là single source of truth.

8. **Owner Administration (FR35–FR40):** Config-file driven showcase, contact form admin endpoint,
   Plausible analytics, rate limiting (3 tiers). Architectural implication: no CMS, git-based
   content pipeline, role-based API authorization.

9. **Platform Data & API Contract (FR49–FR51):** 4 core entities (User, Session,
   ContactSubmission, ProjectHealth), structured error format `{error: {code, message}}`, OpenAPI
   at `/api-docs`. Architectural implication: strict API contract design before implementation.

**Non-Functional Requirements:**

NFRs trực tiếp drive architectural decisions:

- **Performance:**
  - LCP < 1.5s → Static SPA on Vercel CDN (no SSR); SSG prerender at build time
  - Bundle < 300KB gzipped → Code splitting; lazy-load project detail pages
  - Lighthouse CI ≥90 gate → Build pipeline enforcement, not runtime check
  - Platform BE ≤200ms p95 → CRUD stays simple; no N+1 queries; connection pooling
  - WebSocket delivery < 3s → Async event-driven architecture in Platform BE

- **Security:**
  - JWT 15m/7d TTL → Token refresh logic mandatory on FE
  - CORS explicit allow-list → No wildcard; Vercel domain + localhost only
  - Rate limiting: 3 distinct rules (contact: 3/day/IP; AI: 5/10min/IP; general: 100/min/IP)
  - HMAC-SHA256 webhook verification → GitHub webhook security gate

- **Reliability:**
  - Platform BE uptime ≥99.9% → Google Cloud Run + Neon always-on; zero cold start
  - WebSocket reconnect with exponential backoff (max 3 retries → polling) → FE-owned decision
  - Graceful degradation: Portfolio FE fully functional without BE
  - Database backup: Daily cron → PostgreSQL dump → Oracle Object Storage
  - BE restart recovery: FE WebSocket reconnection must handle BE JVM warmup gracefully,
    not just network disconnect

- **Accessibility:**
  - WCAG 2.1 AA mandatory → shadcn/ui accessible primitives; skip links; ARIA attributes
  - `prefers-reduced-motion` no exceptions → Global MOTION_ENABLED context at app root

**Scale & Complexity:**

- **Primary domain:** Full-stack Web App + Platform API (Greenfield)
- **Complexity level:** Medium (PRD classification confirmed)
- **Architectural components estimated:** 12–15 distinct modules
  (Hero, Projects Gallery, Project Detail, About, Contact, Nav, Admin API, Auth, Metrics Pipeline,
  WebSocket Server, GitHub Proxy, Webhook Handler, Config Registry, i18n, Motion System)

### Technical Constraints & Dependencies

**Hosting constraints (hard):**
- Portfolio FE → Vercel (free tier) — static SPA only, no server-side execution
- Platform BE → Google Cloud Run (always-on, containerized) — Spring Boot JAR in Docker container
- Database → PostgreSQL on Neon (free tier, serverless) — connection pooling via HikariCP
- SSL → Cloud Run terminates TLS natively — no Nginx, no Let's Encrypt
- Rate Limiting → Spring Boot `RateLimitService` (Bucket4j) in `shared/ratelimit/*` — not in Nginx

**Technology choices (committed):**
- Frontend: React + TypeScript + Vite + Tailwind CSS + shadcn/ui + Framer Motion
- Backend: Spring Boot (Java) + PostgreSQL + Bucket4j rate limiting (no Nginx)
- Auth: JWT stateless + Google OAuth2 + JWKS public endpoint
- Monitoring: UptimeRobot (5-min checks); Plausible analytics (external)
- CI/CD: GitHub Actions → Google Cloud Run; Vercel auto-deploy on push

**External service dependencies:**
- GitHub API (cached 1h in-memory; fallback to last cached; graceful hide if cache empty)
- Google OAuth2 (auth code flow for demo apps only)
- Plausible (analytics script embed — no backend dependency)
- UptimeRobot (outbound HTTP monitor — no code dependency)

**Package boundary constraint (enforced Sprint 1):**

```
Platform BE (Spring Boot):
├── auth.*              # JWT, OAuth2, JWKS endpoints
│   └── auth.session.*  # Session entity + SessionRepository (auth boundary, NOT platform)
├── platform.*          # Metrics aggregation, WebSocket, admin APIs
├── apps.wallet.*       # Wallet App domain logic
└── shared.*            # Utilities, error handling, rate limiting
```

**Session entity belongs to `auth.session.*`** — not `platform.*`. Cross-boundary access
returns 403. This boundary is an architectural constraint, not a guideline.

### Cross-Cutting Concerns Identified

1. **Authentication & Authorization** — JWT lifecycle (issue/refresh/revoke) + OAuth2 + role-based
   admin endpoints + JWKS public endpoint; affects auth module, all API endpoints, FE token
   management, demo app integration.

2. **Real-Time Data Pipeline** — WebSocket server (BE) + WebSocket subscriber with reconnect logic
   (FE) + polling scheduler (BE) + staleness state machine (FE); affects metrics display,
   connection health, graceful degradation path.
   - **WS fallback ownership:** FE-owned decision. FE initiates reconnection attempts (exponential
     backoff, max 3 retries), then self-switches to polling. BE does NOT signal fallback.

3. **Graceful Degradation** — FE renders 100% without BE; WS falls back to polling (FE-owned);
   GitHub API timeout hides section (AbortController 2s, no error shown); metrics hidden after 24h;
   BE JVM restart handled by FE WS reconnect logic.

4. **Motion System** — Global `MOTION_ENABLED` context reads `prefers-reduced-motion` at app root;
   3 semantic spring tokens (gentle/snappy/bouncy); CSS animation disable via media query;
   affects every animated component — no exceptions allowed.

5. **Internationalization** — vi/en at app root; smart referral-based default (`?from=`);
   localStorage persistence; selective bilingual (UI + personal content only; technical = English);
   affects content structure, URL params, display logic.

6. **Rate Limiting** — 3 distinct rule sets (contact: 3/day/IP + honeypot; AI: 5/10min/IP;
   general: 100/min/IP); implemented via `RateLimitService` (Bucket4j) + `RateLimitFilter`
   in `shared/ratelimit/*`; general tier enforced per-request, contact tier enforced at
   submission, AI tier is a no-op placeholder (Phase 2).

7a. **BE Config Registry (Runtime)** — Platform BE config file drives polling target registration
    for demo apps; new app = add config entry + restart; zero code change to core; affects
    scheduler, WebSocket broadcast scope, health aggregation.

7b. **FE Build Config (Static Build Time)** — Portfolio FE config file drives showcase content
    (project metadata, artist statements, build timelines); new project = add config entry + git
    push → Vercel auto-rebuild; separate lifecycle from BE config.

8. **API Contract** — Structured error format `{error: {code, message}}`; OpenAPI at `/api-docs`;
   `/api/v1/` base path; 4 core entities with user-scoped isolation; affects every endpoint design.

9. **WebSocket-Cloud Run Infrastructure Contract** — Cloud Run natively handles WebSocket
   upgrade when `forward-headers-strategy: FRAMEWORK` is set in `application-prod.yml`. The
   `WebSocketConfig.java` registers `/ws/**` directly — no Nginx required. WebSocketConfig
   is the sole authority for WS endpoint registration.

10. **Phase 2 Architectural Hooks** — The following Phase 2 features require architectural
    accommodation in Phase 1 code to avoid future refactoring:
    - **Command Palette (FR21):** Requires global keyboard event registry at app root (Phase 1
      must not block keyboard events at section level).
    - **AI Endpoint Rate Limiting (FR40):** `RateLimitService.tryAi()` currently returns
      true (no-op). When Phase 2 AI endpoints are added, wire `tryAi()` to the AI
      endpoint handler and set the Bucket4j tier to active.
    - **Video Walkthroughs (FR22):** Project detail page layout must reserve space for video embed
      without layout shift when feature is added.

## Starter Template Evaluation

### Primary Technology Domain

Two independent projects with separate deployment lifecycles:
- **Portfolio FE** → Static SPA (Vite + React + TypeScript) on Vercel CDN
- **Platform BE** → Spring Boot monolith on Google Cloud Run + Neon ARM

### Starter Options Considered

**Frontend:**
- `create-vite@latest --template react-ts` ✅ Selected — official, minimal, no hidden opinions
- T3 Stack (`create-t3-app`) — rejected: Next.js SSR not needed; SEO explicitly out of scope
- `create-react-app` — rejected: deprecated, unmaintained

**Backend:**
- `start.spring.io` (Spring Initializr) ✅ Selected — official, always current, granular deps
- `Bootify.io` — rejected: CRUD scaffolding conflicts with our package boundary discipline
- Community boilerplates (Genc, Tzesh) — rejected: add Redis/Kafka complexity not in scope

### Selected Starter: Portfolio FE — `create-vite` + shadcn/ui

**Rationale:** Official Vite starter is intentionally minimal — clean base to add exactly what
we need without hidden opinions. Tailwind v4 chosen (CSS-first, no `tailwind.config.js`).

**Initialization Sequence:**

```bash
# Step 1: Scaffold Vite + React + TypeScript
npm create vite@latest portfolio-fe -- --template react-ts
cd portfolio-fe

# Step 2: Install Tailwind CSS v4 (CSS-first, no tailwind.config.js)
pnpm add tailwindcss @tailwindcss/vite
pnpm add -D @types/node

# Step 3: Initialize shadcn/ui (detects Vite + Tailwind v4 automatically)
pnpm dlx shadcn@latest init

# Step 4: Install Framer Motion + Lucide
pnpm add framer-motion lucide-react

# Step 5: Install i18n
pnpm add react-i18next i18next

# Note: vite-plugin-ssg added in Sprint 0 AFTER Vite 7 compatibility is verified
# See: vite-plugin-ssg Sprint 0 verification note below
```

**Architectural Decisions Made by This Stack:**

**Language & Runtime:**
- TypeScript (strict mode via `tsconfig.app.json`)
- Vite 7.3.1 — Node.js ≥20.19 required
- Split TypeScript config: `tsconfig.json` + `tsconfig.app.json` + `tsconfig.node.json`
- Path alias `@/*` → `./src/*` configured in both tsconfig files + `vite.config.ts`

**Styling Solution:**
- Tailwind CSS v4 — **CSS-first configuration (no `tailwind.config.js` — this file must NOT
  be created)**. All config (theme, colors, tokens) lives in `src/index.css` via `@theme {}`
- Design tokens (Electric Purple `#A855F7`, spring animation variables) in `src/index.css`
- shadcn/ui 2.4.x — components copied into `src/components/ui/` (owned code, no lock-in)
- `tw-animate-css` (replaces deprecated `tailwindcss-animate`)

**Build Tooling:**
- Vite 7 with `@tailwindcss/vite` plugin
- Default browser target: `baseline-widely-available`
- ⚠️ `vite-plugin-ssg`: **Verify Vite 7 compatibility in Sprint 0 before committing.**
  Community-maintained plugin (last release v0.24.x, supports Vite 5+). Vite 7 compatibility
  unconfirmed at time of writing.
  - **Fallback A:** Pure SPA + `<link rel="preload">` + skeleton states (may achieve LCP 1.5s
    on Vercel CDN without SSG — must be measured in Sprint 0)
  - **Fallback B:** Migrate FE to **Astro** (`@astrojs/react` island architecture) if SSG is
    required and `vite-plugin-ssg` incompatible. Astro has native Vite 7 + React support.

**Testing Framework:**
- Not included in Vite starter → add in Sprint 0:
  `pnpm add -D vitest @testing-library/react @testing-library/user-event jsdom`

**Code Organization:**
```
portfolio-fe/
├── index.html
├── vite.config.ts
├── components.json          ← shadcn/ui config
├── src/
│   ├── main.tsx
│   ├── App.tsx              ← route root + providers (Motion, i18n, Theme, WS)
│   ├── index.css            ← Tailwind v4 @theme{} tokens + base styles
│   ├── components/
│   │   └── ui/              ← shadcn/ui components (owned)
│   ├── lib/
│   │   └── utils.ts         ← cn() helper
│   └── vite-env.d.ts
```

---

### Selected Starter: Platform BE — Spring Initializr (Spring Boot 3.5.x)

**Rationale:** Spring Boot 3.5.x chosen over 4.0.3. SB4 released November 2025 (4 months old
at time of writing) — ecosystem not yet fully caught up: `springdoc-openapi` SB4 support in
progress but no stable release confirmed, Stack Overflow answers still SB3.x-centric. SB3.4.x
is battle-tested, full ecosystem support, satisfies all PRD requirements. No feature in scope
requires SB4. Defer SB4 until ecosystem matures.

**Initialization Command:**

```bash
# Get latest 3.4.x version from https://start.spring.io before running
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

**Architectural Decisions Made by This Stack:**

**Language & Runtime:**
- Java 21 LTS — stable virtual threads, wide ecosystem, proven in production
  (Java 25 available but 21 preferred: more mature community support, all required libraries
  tested against 21)
- Spring Boot 3.5.x — Spring Framework 6.x + Jakarta EE 10
- Maven — predictable CI/CD debugging, safer for solo dev vs Gradle edge cases

**Dependencies Included:**

| Starter | Purpose |
|---|---|
| `spring-boot-starter-web` | REST API (Jackson JSON, embedded Tomcat) |
| `spring-boot-starter-security` | Auth foundation (JWT filter chain configured manually) |
| `spring-boot-starter-data-jpa` | ORM + repository pattern (Hibernate) |
| `postgresql` (runtime) | PostgreSQL JDBC driver |
| `spring-boot-starter-websocket` | WebSocket server (STOMP or native WS) |
| `spring-boot-starter-oauth2-client` | Google OAuth2 authorization code flow |
| `spring-boot-starter-oauth2-resource-server` | JWT validation + JWKS endpoint |
| `spring-boot-starter-actuator` | `/actuator/health` for UptimeRobot |
| `spring-boot-starter-validation` | Bean validation for request payloads |
| `spring-boot-devtools` | Hot reload in development |

**Added manually in Sprint 0 (not in Initializr):**
- `springdoc-openapi-starter-webmvc-ui:2.8.x` → `/api-docs` (OpenAPI, FR51)

**Code Organization (package boundary enforced from Sprint 0):**
```
dev.chinh.portfolio/
├── auth/
│   ├── session/             ← Session entity + SessionRepository (auth boundary)
│   ├── jwt/                 ← JWT service, filter, JWKS endpoint
│   └── oauth2/              ← Google OAuth2 handler
├── platform/
│   ├── metrics/             ← Polling scheduler, ProjectHealth aggregation
│   ├── websocket/           ← WebSocket server, broadcast service
│   ├── webhook/             ← GitHub webhook + HMAC verification
│   ├── contact/             ← ContactSubmission + admin endpoint
│   └── admin/               ← Analytics endpoint
├── apps/
│   └── wallet/              ← Wallet App domain (Sprint 2+)
└── shared/
    ├── error/               ← Structured error response {error: {code, message}}
    ├── ratelimit/           ← Rate limiting (3 rule sets)
    └── config/              ← App config + demo app registry
```

**Testing Framework:**
- Spring Boot Test + JUnit 5 + Mockito (included)
- Add Testcontainers for PostgreSQL integration tests in Sprint 0
- Coverage targets: auth ≥80%, metrics ≥80% (NFR-M1, NFR-M2)

---

### Key Setup Notes for AI Agents

1. **Tailwind v4 is CSS-first — NO `tailwind.config.js`:** All design tokens, theme config, and
   custom utilities go in `src/index.css` using `@theme {}`. Any agent that creates a
   `tailwind.config.js` is making an error.

2. **shadcn/ui components are owned code:** `pnpm dlx shadcn@latest add [component]` copies
   source into `src/components/ui/`. Never import from a `shadcn` package. Modify components
   directly in `src/components/ui/`.

3. **Sprint 0 must verify `vite-plugin-ssg` before use:** Do not assume it works with Vite 7.
   Measure LCP with pure SPA first; if ≥1.5s, evaluate fallback options (pure SPA or Astro).

4. **Session entity in `auth.session.*` only:** Never place Session in `platform.*`. Package
   boundary is enforced from the first line of code.

5. **Spring Boot version is 3.4.x, NOT 4.x:** Any agent referencing SB4 features or APIs is
   using the wrong version. Verify against SB3.4 docs.

6. **`springdoc-openapi` is manually added to `pom.xml`:** It does not come from Spring
   Initializr. Sprint 0 setup story must include adding this dependency.

**Note:** Project initialization using these commands should be the first two implementation
stories in Sprint 0 (one story per project: `portfolio-fe` and `portfolio-platform`).

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Database migration tool selected → determines Sprint 0 schema setup
- WebSocket protocol selected → determines BE WebSocket handler + FE hook architecture
- FE state management selected → determines App.tsx provider structure from first component
- Repository structure selected → determines CI/CD pipeline setup

**Important Decisions (Shape Architecture):**
- FE routing library → determines route definitions + lazy loading pattern
- BE HTTP client → determines GitHub API proxy + caching implementation
- OpenAPI-to-TypeScript type generation → determines FE/BE contract maintenance workflow

**Deferred Decisions (Post-MVP):**
- Redis caching (if in-memory Caffeine cache proves insufficient at scale)
- Spring Boot 4.x migration (when ecosystem matures, no timeline set)
- Astro migration (only if vite-plugin-ssg incompatible AND LCP target unachievable as SPA)

---

### Data Architecture

#### Database Migrations: Flyway

**Decision:** Flyway (not Liquibase)

**Rationale:** SQL-native migration files, zero format learning curve, Spring Boot auto-runs
on startup, perfect fit for greenfield 4-entity schema. Liquibase XML/YAML format is overkill
for project scope.

**Implementation rules:**
- Migration files location: `src/main/resources/db/migration/` (Spring Boot Flyway default,
  zero configuration required)
- Naming convention (mandatory): `V{n}__{description}.sql`
  - Examples: `V1__create_users_table.sql`, `V2__create_sessions_table.sql`,
    `V3__create_contact_submissions_table.sql`, `V4__create_project_health_table.sql`
- Add to `pom.xml`: `spring-boot-starter-data-jpa` includes Flyway auto-detection when
  `flyway-core` is on classpath: `<dependency><groupId>org.flywaydb</groupId>
  <artifactId>flyway-core</artifactId></dependency>`

#### GitHub API Caching: Spring Cache + Caffeine

**Decision:** Spring `@Cacheable` abstraction + Caffeine cache implementation

**Rationale:** PRD requires in-memory cache with 1h TTL. Manual `ConcurrentHashMap` risks memory
leaks and lacks TTL support. Spring Cache abstraction (`@EnableCaching` + `@Cacheable`) keeps
service code clean. Caffeine is the de-facto standard in-memory cache for Spring Boot — fast,
TTL-native, no additional infrastructure.

**Implementation:**
```xml
<!-- pom.xml additions -->
<dependency>
  <groupId>com.github.ben-manes.caffeine</groupId>
  <artifactId>caffeine</artifactId>
</dependency>
<dependency>
  <groupId>org.springframework.boot</groupId>
  <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

```yaml
# application.yml
spring:
  cache:
    type: caffeine
    caffeine:
      spec: maximumSize=100,expireAfterWrite=3600s
```

```java
// GitHubProxyService.java
@Cacheable("github-contributions")
public ContributionGraphDto getContributionGraph() { ... }
```

**No other caching mechanisms permitted** — do not create manual cache Maps.

---

### Authentication & Security

*(Decisions already committed in PRD — documented here for agent reference)*

**JWT Implementation:**
- Library: `spring-boot-starter-oauth2-resource-server` (Spring Security's native JWT support)
- Access token TTL: 15 minutes | Refresh token TTL: 7 days
- Token signing: RS256 (asymmetric) — BE holds private key, JWKS endpoint exposes public key
- JWKS endpoint: `GET /api/v1/auth/jwks.json` — public, no auth required

**Google OAuth2:**
- Library: `spring-boot-starter-oauth2-client` (already in starter)
- Flow: Authorization code (server-side) — BE exchanges code for tokens with Google
- Scope: email, profile — sufficient for demo app user identity

**Password hashing:** bcrypt cost factor 12 (`BCryptPasswordEncoder(12)`)

**CORS:**
```java
// Explicit allow-list — no wildcards
allowedOrigins: ["https://chinh.dev", "https://wallet.chinh.dev", "http://localhost:5173"]
```

---

### API & Communication Patterns

#### WebSocket Protocol: Native WebSocket

**Decision:** Native WebSocket (not STOMP over SockJS)

**Rationale:** Portfolio-v2 WebSocket use case is simple one-way broadcast (BE → FE, metrics
updates). STOMP/SockJS adds pub/sub broker complexity, requires `sockjs-client` + `@stomp/stompjs`
on FE (+~25KB bundle). Native WebSocket sufficient: zero additional FE libraries, simpler BE
handler, browser-native API.

**BE Implementation pattern:**
```java
// One WebSocketHandler, broadcast to all connected sessions
@Component
public class MetricsWebSocketHandler extends TextWebSocketHandler {
    private final Set<WebSocketSession> sessions = ConcurrentHashMap.newKeySet();
    // onOpen: add session | onClose: remove session | broadcast(): send to all
}
```

**FE Implementation rule — CRITICAL:**
- `useWebSocket` hook is the **single point of WebSocket lifecycle management**
- **No component may call `new WebSocket()` directly** — always use the hook
- Hook responsibilities: connect, reconnect (exponential backoff, max 3 retries → polling),
  broadcast metrics state updates to Zustand store

```typescript
// Pattern (not implementation — exact code in story)
const useWebSocket = () => {
  // manages: connect, reconnect backoff, fallback to polling, cleanup on unmount
  // writes to: metricsStore (Zustand)
}
// Usage: call once in App.tsx, never in individual components
```

#### BE HTTP Client for External APIs: RestClient

**Decision:** Spring `RestClient` (not WebClient, not RestTemplate)

**Rationale:** BE is servlet-based (not reactive). `RestClient` is the modern synchronous HTTP
client introduced in Spring Boot 3.2 — fluent API, no WebFlux dependency. `RestTemplate` is in
maintenance mode. `WebClient` requires `spring-boot-starter-webflux` (full reactive stack)
unnecessarily.

```java
// RestClient usage pattern
RestClient restClient = RestClient.builder()
    .baseUrl("https://api.github.com")
    .defaultHeader("Authorization", "Bearer " + token)
    .build();
```

Apply to: GitHub API proxy, future external service calls.

#### REST API Conventions

*(From PRD — documented for agent consistency)*
- Base path: `/api/v1/`
- Error format: `{"error": {"code": "SNAKE_CASE_CODE", "message": "Human readable"}}`
- No stack traces in any response
- OpenAPI: `springdoc-openapi-starter-webmvc-ui:2.8.x` → `/api-docs`

---

### Frontend Architecture

#### State Management: Zustand

**Decision:** Zustand (not React Context API, not Jotai, not Redux)

**Rationale:** 8 shared state items identified (theme, language, return visitor count, last viewed
project, motion enabled, gallery filter, referral source, WebSocket connection state). Context API
creates Provider nesting hell for this many independent state concerns. Zustand: ~1KB, no
boilerplate, excellent TypeScript support, DevTools available.

**Required pattern — Zustand persist middleware (MANDATORY):**

All state that persists to localStorage **must** use `zustand/middleware` `persist` — no manual
`localStorage.setItem()` / `localStorage.getItem()` calls anywhere in the codebase.

```typescript
// Pattern for persistent stores
import { create } from 'zustand'
import { persist } from 'zustand/middleware'

export const useThemeStore = create(persist(
  (set) => ({ theme: 'dark', setTheme: (t) => set({ theme: t }) }),
  { name: 'theme-preference' }  // localStorage key
))
```

**localStorage resilience:** All persist middleware calls must handle storage unavailability
gracefully (private browsing). Zustand persist handles this natively — do not add manual
try/catch around store reads.

**Store organization:**
```
src/stores/
├── themeStore.ts          ← dark/light (persistent)
├── languageStore.ts       ← vi/en (persistent)
├── visitorStore.ts        ← returnCount, lastViewedProject (persistent)
├── galleryStore.ts        ← activeFilter (URL param, not localStorage)
├── referralStore.ts       ← referralSource (persistent)
└── metricsStore.ts        ← WebSocket data, connectionState (session-only, no persist)
```

#### Routing: React Router v7

**Decision:** React Router v7 (not TanStack Router)

**Rationale:** ~5 routes (SPA anchor scroll + `/projects/{slug}` detail). RRv7's superior
type safety over TanStack Router not justified for this route count. Established ecosystem,
familiar patterns, SSG-compatible.

**Route structure:**
```typescript
// main routes
/                    → Home (SPA with anchor sections: #projects, #about, #contact)
/projects/:slug      → Project detail page (lazy loaded)
*                    → 404 redirect to /
```

**Lazy loading:** Project detail pages use `React.lazy()` + `Suspense` for code splitting
(bundle size constraint: <300KB gzipped total).

#### FE/BE Type Safety: OpenAPI-Generated Types

**Decision:** FE TypeScript types generated from BE OpenAPI spec

**Rationale:** With separate repos, BE API response shapes and FE types will drift if maintained
separately. OpenAPI spec (`/api-docs`) is the single source of truth. `openapi-typescript`
generates TypeScript interfaces automatically.

**Workflow:**
```bash
# Run in portfolio-fe when BE API changes
npx openapi-typescript http://localhost:8080/api-docs -o src/types/api.d.ts
```

**Rules:**
- `src/types/api.d.ts` is **generated — never manually edited**
- All API response types imported from `src/types/api.d.ts`
- Regenerate when BE adds/changes endpoints (Sprint 0 story includes initial generation)

---

### Infrastructure & Deployment

*(Decisions committed in PRD — documented for agent reference)*

**Repository structure: GitHub Organization + 3 Separate Repos**

**Organization:** `github.com/chinhnt-portfolio-v2/`

**Rationale:** No shared code between FE (TypeScript/React) and BE (Java/Spring Boot). Separate
deployment targets (Vercel vs Google Cloud Run + Neon). Monorepo adds CI/CD complexity with zero benefit.
Type sharing handled via OpenAPI-generated types (not shared code packages). AI planning
artifacts (PRD, architecture, UX spec) live in a dedicated public docs repo — versioned on git,
visible as a portfolio artifact, and decoupled from code deployments.

- `chinhnt-portfolio-v2/portfolio-fe` → Vite SPA code + `DECISIONS.md` → Vercel (auto-deploy on push to main)
- `chinhnt-portfolio-v2/portfolio-platform` → Spring Boot code + `DECISIONS.md` → Google Cloud Run + Neon (GitHub Actions CI/CD → SSH deploy)
- `chinhnt-portfolio-v2/portfolio-docs` → Planning artifacts (`brainstorming/`, `planning-artifacts/`) → GitHub (public, manual push)

**Cloud Run configuration responsibilities:**
- TLS termination handled natively by Cloud Run (no Nginx, no Let's Encrypt)
- `server.forward-headers-strategy: FRAMEWORK` in `application-prod.yml` — required for WebSocket upgrade and `X-Forwarded-*` header passthrough
- Rate limiting: handled by `RateLimitService` (Bucket4j) + `RateLimitFilter` in `shared/ratelimit/*`
- Reverse proxy to Spring Boot (localhost:8080)

**Environment configuration:**
- Spring profiles: `dev` (H2 or local Postgres) / `prod` (Google Cloud Run + Neon Postgres)
- Secrets: environment variables (never committed to repo)
- FE: Vite `.env` files (`VITE_API_URL`, `VITE_WS_URL`)

**CI/CD:**
- Portfolio FE: Vercel GitHub integration (push to main → auto-build → CDN deploy)
- Platform BE: GitHub Actions → `mvn clean package -DskipTests` → SCP JAR to Google Cloud Run + Neon
  → restart service

**Monitoring & Logging:**
- UptimeRobot: 5-min HTTP checks to `/actuator/health`
- Logback (Spring Boot default): structured JSON output, configurable level via env var
- No PII in logs (NFR-O4)

---

### Decision Impact Analysis

**Implementation Sequence (Sprint 0 must complete in order):**

1. Create GitHub Organization `chinhnt-portfolio-v2` + 3 repos (`portfolio-fe`, `portfolio-platform`, `portfolio-docs`)
2. Initialize `portfolio-fe` (Vite + Tailwind v4 + shadcn/ui + Zustand + RRv7)
3. Initialize `portfolio-platform` (Spring Initializr + manual additions)
4. Establish package structure in `portfolio-platform` (auth/platform/apps/shared)
5. Write `V1__create_users_table.sql` through `V4__create_project_health_table.sql`
6. Generate initial OpenAPI spec → run `openapi-typescript` in `portfolio-fe`
7. Verify `vite-plugin-ssg` Vite 7 compatibility; measure LCP baseline
8. Configure `server.forward-headers-strategy: FRAMEWORK` in `application-prod.yml` for WebSocket support on Cloud Run

**Cross-Component Dependencies:**

| Decision | Depends On | Affects |
|---|---|---|
| Flyway migrations | Package structure established | All JPA entities |
| `useWebSocket` hook | Zustand `metricsStore` exists | `ProjectCard`, `HeroCardStack` |
| OpenAPI type generation | BE running + `/api-docs` accessible | All FE API calls |
| Zustand persist middleware | Store structure finalized | All persistent state |
| forward-headers-strategy config | Cloud Run provisioned | All real-time features |

## Implementation Patterns & Consistency Rules

**Critical Conflict Points Identified:** 23 areas across naming, structure, format,
communication, and process patterns.

---

### Naming Patterns

#### Database Naming Conventions

| Element | Convention | Examples |
|---|---|---|
| Table names | `snake_case`, **plural** | `users`, `sessions`, `contact_submissions`, `project_health` |
| Column names | `snake_case` | `user_id`, `created_at`, `is_read`, `refresh_token` |
| Foreign keys | `{singular_table}_id` | `user_id` (not `fk_user`, not `userId`) |
| Primary keys | `id` (UUID) | `id UUID PRIMARY KEY DEFAULT gen_random_uuid()` |
| Index names | `idx_{table}_{column(s)}` | `idx_sessions_user_id`, `idx_contact_submissions_created_at` |
| Timestamp columns | `created_at`, `updated_at` | Always `_at` suffix, always UTC |

**Anti-patterns:**
- ❌ `Users` (PascalCase table) → ✅ `users`
- ❌ `userId` (camelCase column) → ✅ `user_id`
- ❌ `fk_users_id` (FK prefix) → ✅ `user_id`

#### API Endpoint Naming Conventions

| Element | Convention | Examples |
|---|---|---|
| Resource paths | plural nouns, `kebab-case` | `/api/v1/contact-submissions`, `/api/v1/project-health` |
| Path parameters | `camelCase` in `{braces}` | `/api/v1/projects/{projectId}` |
| Query parameters | `camelCase` | `?pageSize=10&sortBy=createdAt` |
| Action endpoints | verb after resource | `/api/v1/auth/logout`, `/api/v1/auth/refresh` |
| Admin endpoints | `/api/v1/admin/` prefix | `/api/v1/admin/contacts`, `/api/v1/admin/analytics` |

**HTTP method conventions:**
- `GET` — read (idempotent)
- `POST` — create or action
- `PUT` — full replace
- `PATCH` — partial update
- `DELETE` — remove

**Anti-patterns:**
- ❌ `/api/v1/getContacts` → ✅ `GET /api/v1/admin/contacts`
- ❌ `/api/v1/contact_submissions` → ✅ `/api/v1/contact-submissions`
- ❌ `/api/v1/project/{project_id}` → ✅ `/api/v1/projects/{projectId}`

#### Java Code Naming Conventions

| Element | Convention | Examples |
|---|---|---|
| Classes | `PascalCase` | `JwtAuthenticationFilter`, `MetricsWebSocketHandler` |
| Methods | `camelCase`, verb-first | `getUserById()`, `broadcastMetrics()`, `validateToken()` |
| Variables | `camelCase` | `userId`, `accessToken`, `projectHealthDto` |
| Constants | `UPPER_SNAKE_CASE` | `JWT_EXPIRY_MINUTES`, `MAX_RATE_LIMIT_ATTEMPTS` |
| Packages | all lowercase | `dev.chinh.portfolio.auth.jwt` |
| DTOs | `{Entity}Dto` suffix | `ProjectHealthDto`, `ContactSubmissionDto` |
| Repositories | `{Entity}Repository` | `UserRepository`, `SessionRepository` |
| Services | `{Domain}Service` | `JwtService`, `MetricsAggregationService` |

#### TypeScript/React Naming Conventions

| Element | Convention | Examples |
|---|---|---|
| Component files | `PascalCase.tsx` | `ProjectCard.tsx`, `HeroCardStack.tsx` |
| Hook files | `use{Name}.ts(x)` | `useWebSocket.ts`, `useMotion.ts` |
| Store files | `{name}Store.ts` | `themeStore.ts`, `metricsStore.ts` |
| Utility files | `camelCase.ts` | `formatDate.ts` |
| Test files | `{Name}.test.tsx` | `ProjectCard.test.tsx`, `useWebSocket.test.ts` |
| Constants | `UPPER_SNAKE_CASE` | `WS_RECONNECT_MAX_RETRIES` |
| Type aliases | `PascalCase` | `ProjectHealthData`, `FilterOption` |
| Zustand stores | `use{Name}Store` export | `export const useThemeStore = create(...)` |
| Motion constants | `SPRING_{NAME}` | `SPRING_GENTLE`, `SPRING_SNAPPY`, `SPRING_BOUNCY` |

**Anti-patterns:**
- ❌ `project-card.tsx` → ✅ `ProjectCard.tsx`
- ❌ `WebsocketHook.ts` → ✅ `useWebSocket.ts`

---

### Structure Patterns

#### Frontend File Organization

```
src/
├── components/
│   ├── ui/                   ← shadcn/ui owned components (never modify manually)
│   ├── layout/               ← Nav, Footer, Layout wrappers
│   ├── sections/             ← Hero, Projects, About, Contact (page sections)
│   └── shared/               ← ProjectCard, StatusIndicator, MetricPair, FilterChip
├── hooks/                    ← all custom hooks (useWebSocket, etc.)
├── stores/                   ← all Zustand stores
├── lib/                      ← utilities (cn, formatDate, validators)
├── types/                    ← api.d.ts (generated) + manual types
├── i18n/                     ← en.json, vi.json + i18n config
├── pages/                    ← route-level components (lazy loaded)
│   ├── HomePage.tsx
│   └── ProjectDetailPage.tsx
└── constants/
    ├── motion.ts             ← SPRING_GENTLE, SPRING_SNAPPY, SPRING_BOUNCY
    └── config.ts             ← WS_RECONNECT_MAX_RETRIES, etc.
```

**Rules:**
- `components/ui/` — only `pnpm dlx shadcn@latest add` modifies it, never manual edits
- `types/api.d.ts` — generated only, never manually edited
- Hooks always in `hooks/` — never inline in components
- Stores always in `stores/` — never co-located with components

#### Test File Location

- **FE:** Co-located (`ProjectCard.test.tsx` next to `ProjectCard.tsx`)
- **BE:** Mirror `src/main/` in `src/test/` (`JwtServiceTest.java` mirrors `JwtService.java`)

#### Backend Service Layer Pattern

```
{domain}/
├── {Domain}Controller.java     ← @RestController, HTTP mapping only
├── {Domain}Service.java        ← business logic, @Transactional
├── {Domain}Repository.java     ← @Repository, JPA queries
├── {Entity}.java               ← @Entity, DB mapping
├── {Entity}Dto.java            ← request/response shape (Java record preferred)
└── {Domain}Mapper.java         ← Entity ↔ DTO (manual mapping, no MapStruct)
```

**Rules:**
- Controllers never contain business logic — always delegate to service
- Services never access other domains' repositories — go through that domain's service
- No MapStruct — manual mapping in Mapper class (explicit, debuggable)

#### Global Exception Handler (BE)

Single `GlobalExceptionHandler.java` in `shared/error/` — the **only** `@ControllerAdvice`:

```java
@ControllerAdvice
public class GlobalExceptionHandler {
    // MethodArgumentNotValidException  → 400 + structured error
    // AccessDeniedException           → 403 + structured error
    // EntityNotFoundException (custom) → 404 + structured error
    // Exception (catch-all)           → 500 + structured error (no stack trace)
}
```

No other `@ControllerAdvice` classes permitted in any package.

---

### Format Patterns

#### API Response Formats

```json
// Single resource (direct object — no wrapper)
{ "id": "uuid", "email": "user@example.com", "createdAt": "2026-03-03T10:30:00Z" }

// List (direct array)
[ { "id": "uuid", ... }, { "id": "uuid", ... } ]

// Paginated list (Spring Page convention)
{ "content": [...], "totalElements": 42, "page": 0, "size": 20 }

// Error (always this exact shape — FR50)
{ "error": { "code": "SNAKE_CASE_CODE", "message": "Human readable." } }
```

No `{data: ..., status: ...}` wrappers. No other shapes.

#### JSON Field Naming

- API JSON: **camelCase** (`userId`, `createdAt`, `isRead`)
- DB columns stay `snake_case` (`user_id`, `created_at`, `is_read`) — Jackson maps automatically
- Exception: `api.d.ts` auto-generated from OpenAPI — follows BE output

#### Date/Time Format

- Always ISO 8601 UTC: `"2026-03-03T10:30:00Z"`
- Never epoch milliseconds in API
- FE localizes from ISO string using `Intl.RelativeTimeFormat`

#### HTTP Status Codes

| Scenario | Status |
|---|---|
| Success with body | `200 OK` |
| Created | `201 Created` |
| Success no body | `204 No Content` |
| Validation error | `400 Bad Request` |
| Unauthenticated | `401 Unauthorized` |
| Forbidden | `403 Forbidden` |
| Not found | `404 Not Found` |
| Rate limited | `429 Too Many Requests` |
| Server error | `500 Internal Server Error` |

Never return `200` with an error body.

---

### Communication Patterns

#### WebSocket Message Format

```json
{
  "type": "METRICS_UPDATE",
  "payload": {
    "projectId": "wallet-app",
    "uptime": 99.87,
    "responseTime": 187,
    "lastDeploy": "2026-03-03T10:30:00Z",
    "lastUpdated": "2026-03-03T14:22:00Z"
  }
}
```

**Message types (exhaustive):** `METRICS_UPDATE`, `METRICS_OFFLINE`, `PING`

- `type` always UPPER_SNAKE_CASE; `payload` always an object
- FE ignores unknown `type` values (forward-compatible)

#### Motion System Pattern

**`<MotionConfig reducedMotion="user">` at App.tsx root** — Framer Motion's built-in
`reducedMotion` prop reads `prefers-reduced-motion` media query globally. No custom context
needed.

```tsx
// App.tsx root — single location
import { MotionConfig } from 'framer-motion'

export default function App() {
  return (
    <MotionConfig reducedMotion="user">
      {/* all other providers and routes */}
    </MotionConfig>
  )
}
```

**Spring tokens — single source of truth in `constants/motion.ts`:**

```typescript
import type { Transition } from 'framer-motion'

export const SPRING_GENTLE: Transition = { type: 'spring', stiffness: 300, damping: 30 }
export const SPRING_SNAPPY: Transition = { type: 'spring', stiffness: 400, damping: 25 }
export const SPRING_BOUNCY: Transition = { type: 'spring', stiffness: 200, damping: 15 }
```

Never inline spring values in components — always import from `constants/motion.ts`.

#### AnimatePresence Pattern

Wrap **animated lists only** — never pages, never single items that don't exit:

```tsx
// ✅ Correct — gallery filter transitions (list that changes)
<AnimatePresence mode="popLayout">
  {filteredProjects.map(p => <motion.div key={p.id} ...>{...}</motion.div>)}
</AnimatePresence>

// ✅ Correct — skeleton → content swap
<AnimatePresence mode="wait">
  {isLoading ? <Skeleton key="skeleton" /> : <Content key="content" />}
</AnimatePresence>

// ❌ Wrong — wrapping page or static container
<AnimatePresence><div>{children}</div></AnimatePresence>
```

**Rule:** Every `motion.*` child inside `AnimatePresence` must have a stable `key` prop.

#### Zustand State Updates

```typescript
// ✅ Functional update for derived state
set((state) => ({ count: state.count + 1 }))

// ✅ Direct set for independent state
set({ theme: 'dark' })

// ❌ Never — direct mutation
state.theme = 'dark'
```

---

### Process Patterns

#### Frontend Error Handling

Three tiers — choose based on criticality:

1. **Silent failure** (non-critical): `console.error()`, hide section, no user message.
   Applies to: GitHub API timeout, WebSocket offline, metrics stale >24h.
2. **Toast notification** (non-fatal user action): shadcn `Toast` (`role="status"`).
   Applies to: clipboard copy result, non-critical form feedback.
3. **Inline error** (blocking user action): error message within the section.
   Applies to: contact form submission failure.

```typescript
// ✅ Pattern — error in hook, component reacts cleanly
const { data, isLoading, error } = useProjectHealth(projectId)
if (error) return null  // silent — section disappears

// ❌ Wrong — error handling in component render
const [data, setData] = useState(null)
try { const res = await fetch('...'); setData(res) } catch { ... }
```

#### Loading State Naming

```typescript
const [isLoading, setIsLoading] = useState(true)      // data loading
const [isSubmitting, setIsSubmitting] = useState(false) // form submission
const [data, setData] = useState<T | null>(null)        // null = not loaded
const [error, setError] = useState<string | null>(null) // null = no error
```

Always show `<Skeleton>` (shadcn/ui) when `isLoading === true`. Never empty state during load.

#### Form Handling Pattern

All forms: **React Hook Form + Zod + shadcn/ui Form** (mandatory stack):

```bash
# Add to portfolio-fe Sprint 0
pnpm add react-hook-form @hookform/resolvers zod
```

```typescript
// ✅ Required pattern
const schema = z.object({ email: z.string().email(), message: z.string().min(10) })
// schema defined at module level, outside component

const form = useForm({ resolver: zodResolver(schema) })
// validation triggers onSubmit (not onChange)
// errors via <FormMessage> — never manual <p> tags
```

#### Backend Validation

```java
// Controller — only @Valid, no manual validation
@PostMapping public ResponseEntity<Void> submit(@Valid @RequestBody RequestDto req) { }

// DTO — annotations only
public record RequestDto(
  @Email @NotBlank String email,
  @Size(min = 10, max = 2000) @NotBlank String message
) {}
// GlobalExceptionHandler converts MethodArgumentNotValidException → 400 structured error
```

#### Environment Variable Conventions

**Frontend (Vite):**
- All client-accessible vars **must** have `VITE_` prefix — without it, value is `undefined` at runtime with no error
- File structure:
  - `.env.example` — committed to repo, placeholder values, documents all required vars
  - `.env.local` — gitignored, actual dev values
  - `.env.production` — gitignored or set in Vercel dashboard

```bash
# .env.example (committed)
VITE_API_URL=http://localhost:8080
VITE_WS_URL=ws://localhost:8080/ws/metrics
```

**Backend (Spring Boot):**
- Spring profiles: **`dev` and `prod` only** — no `local`, `development`, `production`, `staging`
- Config files: `application.yml` (shared) + `application-dev.yml` + `application-prod.yml`
- Secrets via environment variables injected at runtime — never hardcoded in any `.yml`

#### TypeScript Import Order (ESLint enforced)

```typescript
// 1. React
import React, { useState } from 'react'
// 2. Third-party (alphabetical)
import { motion } from 'framer-motion'
// 3. Internal components (@/ alias — never relative paths)
import { ProjectCard } from '@/components/shared/ProjectCard'
// 4. Internal hooks, stores, utils
import { useWebSocket } from '@/hooks/useWebSocket'
// 5. Types
import type { ProjectHealthData } from '@/types/api'
```

**ESLint enforcement stack:**
```bash
pnpm add -D eslint eslint-plugin-import eslint-import-resolver-typescript @typescript-eslint/eslint-plugin
```

Key rules in `.eslintrc`:
- `import/order` — enforces the 5-group import order above
- `import/no-relative-parent-imports` — blocks `../../` imports, forces `@/` alias
- `@typescript-eslint/consistent-type-imports` — enforces `import type` for types

---

### Enforcement Summary

**All AI Agents MUST:**

1. `snake_case` plural for DB tables, `snake_case` for columns — always
2. `camelCase` for JSON API fields — always
3. `VITE_` prefix for all Vite env vars — without it, value is `undefined` at runtime
4. Spring profiles: `dev` / `prod` only — no other profile names
5. Use `@/` alias for all FE internal imports — never `../../`
6. No `tailwind.config.js` — Tailwind v4 config in `src/index.css` `@theme {}` only
7. No `new WebSocket()` outside `useWebSocket` hook
8. No `localStorage.setItem()` directly — Zustand `persist` middleware only
9. No manual `try/catch` in component render path — error state in hooks
10. One `GlobalExceptionHandler.java` in `shared/error/` — no other `@ControllerAdvice`
11. Spring tokens from `constants/motion.ts` — never inline stiffness/damping values
12. `<MotionConfig reducedMotion="user">` at App.tsx root — no custom motion context
13. `AnimatePresence` wraps animated lists only — never pages or static containers
14. ISO 8601 UTC for all dates in API — never epoch ms, never localized strings
15. `React Hook Form + Zod` for all forms — never manual controlled form state

**Anti-Pattern Quick Reference:**

| ❌ Anti-Pattern | ✅ Correct Pattern |
|---|---|
| `localStorage.setItem('theme', v)` | `useThemeStore.setState({ theme: v })` |
| `new WebSocket('ws://...')` in component | `useWebSocket()` in App.tsx only |
| `tailwind.config.js` | `@theme {}` in `src/index.css` |
| `fetch('/api/...')` in component | Custom hook or service function |
| `{ data: {...}, status: 200 }` | Direct `{...}` object |
| `snake_case` in JSON response | `camelCase` in JSON |
| `import X from '../../components/X'` | `import X from '@/components/X'` |
| `API_URL=...` in `.env` | `VITE_API_URL=...` in `.env` |
| `application-local.yml` | `application-dev.yml` |
| Multiple `@ControllerAdvice` classes | One `GlobalExceptionHandler` in `shared/error/` |
| Inline `{ type: 'spring', stiffness: 300 }` | `SPRING_GENTLE` from `constants/motion.ts` |
| `<AnimatePresence><Page /></AnimatePresence>` | `<AnimatePresence>{list.map(...)}</AnimatePresence>` |
| Manual form state with `useState` | `useForm({ resolver: zodResolver(schema) })` |

## Project Structure & Boundaries

### Repository Overview

Three repositories under GitHub Organization `chinhnt-portfolio-v2`, deployed independently:

| Repo | Stack | Deployment | Branch strategy |
|---|---|---|---|
| `portfolio-fe` | Vite + React + TypeScript | Vercel | `main` → auto-deploy |
| `portfolio-platform` | Spring Boot 3.5.x + Java 21 | Google Cloud Run | `main` → GitHub Actions CI/CD |
| `portfolio-docs` | Markdown docs + BMAD tooling | GitHub (public) | `main` → manual push |

---

### Complete Project Tree: `portfolio-fe`

```
portfolio-fe/
│
├── .github/
│   └── workflows/
│       └── ci.yml                    ← Vitest + ESLint on PR (Vercel handles deploy)
│
├── public/
│   ├── favicon.ico
│   └── og-image.png                  ← Social preview (Layer 0 trust signal)
│
├── src/
│   ├── main.tsx                      ← ReactDOM.createRoot entry
│   ├── App.tsx                       ← Root: MotionConfig + Providers + Router
│   ├── index.css                     ← Tailwind v4 @import + @theme{} design tokens
│   ├── vite-env.d.ts
│   │
│   ├── pages/
│   │   ├── HomePage.tsx              ← All SPA sections (Hero → Projects → About → Contact)
│   │   └── ProjectDetailPage.tsx     ← /projects/:slug (React.lazy + Suspense)
│   │
│   ├── components/
│   │   ├── ui/                       ← shadcn/ui owned — only shadcn CLI modifies
│   │   │   ├── button.tsx
│   │   │   ├── badge.tsx
│   │   │   ├── card.tsx
│   │   │   ├── dialog.tsx
│   │   │   ├── input.tsx
│   │   │   ├── textarea.tsx
│   │   │   ├── label.tsx
│   │   │   ├── skeleton.tsx
│   │   │   ├── toast.tsx
│   │   │   ├── navigation-menu.tsx
│   │   │   ├── sheet.tsx
│   │   │   ├── separator.tsx
│   │   │   └── form.tsx
│   │   │
│   │   ├── layout/
│   │   │   ├── Nav.tsx               ← Sticky nav (desktop inline + mobile hamburger)
│   │   │   ├── MobileSheet.tsx       ← Mobile nav drawer (focus trap)
│   │   │   └── SkipLinks.tsx         ← "Skip to main content" + "Skip to navigation"
│   │   │
│   │   ├── sections/
│   │   │   ├── Hero.tsx              ← Eyebrow + headline + HeroCardStack + CTAs
│   │   │   ├── Projects.tsx          ← FilterChips + ProjectCard grid + AnimatePresence
│   │   │   ├── About.tsx             ← POV bio + claims-with-proof + Why Hire Me
│   │   │   └── Contact.tsx           ← Email copy-to-clipboard + ContactForm
│   │   │
│   │   └── shared/
│   │       ├── ProjectCard.tsx        ← Gallery card (metrics + tech stack + glow border)
│   │       ├── ProjectCard.test.tsx
│   │       ├── HeroCardStack.tsx      ← 3-metric proof cluster in hero
│   │       ├── HeroCardStack.test.tsx
│   │       ├── BuildStoryPreview.tsx  ← GitHub-fetched decision log + skeleton
│   │       ├── BuildStoryPreview.test.tsx
│   │       ├── FilterChip.tsx         ← Tech stack single-select filter (URL state)
│   │       ├── StatusIndicator.tsx    ← Live/Building/Archived dot + label + aria-label
│   │       ├── MetricPair.tsx         ← Number + label display
│   │       ├── EyebrowChip.tsx        ← Role identity pill above hero headline
│   │       └── ContactForm.tsx        ← RHF + Zod + shadcn Form + success animation
│   │
│   ├── hooks/
│   │   ├── useWebSocket.ts           ← WS lifecycle: connect, reconnect backoff → metricsStore
│   │   ├── useWebSocket.test.ts
│   │   ├── useProjectHealth.ts       ← Reads metricsStore, returns per-project data
│   │   ├── useGitHubBuildStory.ts    ← fetch + AbortController 2s timeout
│   │   ├── useReturnVisitor.ts       ← Reads visitorStore → { isReturn, count }
│   │   └── useContactForm.ts         ← RHF form logic + submission + success state
│   │
│   ├── stores/
│   │   ├── themeStore.ts             ← 'dark'|'light' (persist: 'theme-preference')
│   │   ├── languageStore.ts          ← 'vi'|'en' (persist: 'language-preference')
│   │   ├── visitorStore.ts           ← returnCount, lastViewedProjectSlug (persist)
│   │   ├── referralStore.ts          ← referralSource from ?from= (persist)
│   │   ├── galleryStore.ts           ← activeFilter (URL param — NOT persisted)
│   │   └── metricsStore.ts           ← WS metrics data + connectionState (session only)
│   │
│   ├── lib/
│   │   ├── utils.ts                  ← cn() helper
│   │   ├── formatDate.ts             ← ISO → relative time (Intl.RelativeTimeFormat)
│   │   ├── validators.ts             ← Zod schemas (contactFormSchema, etc.)
│   │   └── referral.ts               ← Parse ?from= param, map to language default
│   │
│   ├── constants/
│   │   ├── motion.ts                 ← SPRING_GENTLE, SPRING_SNAPPY, SPRING_BOUNCY
│   │   ├── config.ts                 ← WS_RECONNECT_MAX_RETRIES=3, WS_BACKOFF_BASE_MS=1000
│   │   └── projects.ts               ← Static showcase config (FR35 — edit to update gallery)
│   │
│   ├── i18n/
│   │   ├── index.ts                  ← i18next init + language detection logic
│   │   ├── en.json                   ← English translations
│   │   └── vi.json                   ← Vietnamese translations
│   │
│   └── types/
│       ├── api.d.ts                  ← GENERATED from OpenAPI — never manually edited
│       └── motion.types.ts           ← Spring token types, animation variant types
│
├── index.html
├── vite.config.ts                    ← Tailwind v4 plugin + @/ alias + dev proxy
├── tsconfig.json
├── tsconfig.app.json                 ← strict, paths: @/ → ./src/
├── tsconfig.node.json
├── components.json                   ← shadcn/ui config
├── .eslintrc.json                    ← import/order + no-relative-parent + type-imports
├── vitest.config.ts                  ← jsdom + coverage thresholds (see below)
├── .env.example                      ← VITE_API_URL, VITE_WS_URL (committed, no secrets)
├── .env.local                        ← Actual dev values (gitignored)
├── .gitignore
├── DECISIONS.md                      ← FE architectural decisions log (fetched by BuildStoryPreview)
└── package.json
```

**`vite.config.ts` dev proxy (exact config — WS requires `rewriteWsOrigin`):**
```typescript
server: {
  proxy: {
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true
    },
    '/ws': {
      target: 'ws://localhost:8080',
      ws: true,
      changeOrigin: true,
      rewriteWsOrigin: true   ← required in Vite 7 for WS upgrade
    }
  }
}
```

**`vitest.config.ts` coverage thresholds (NFR-M4: FE utilities ≥60%):**
```typescript
coverage: {
  provider: 'v8',
  thresholds: {
    functions: 60,
    lines: 60,
    branches: 60,
    statements: 60
  },
  include: ['src/lib/**', 'src/hooks/**', 'src/stores/**']
}
```

---

### Complete Project Tree: `portfolio-platform`

```
portfolio-platform/
│
├── .github/
│   └── workflows/
│       └── ci-cd.yml                 ← Build + test → Cloud Run deploy via gcloud CLI
│
├── deploy/
│   ├── deploy.sh                     ← gcloud run deploy (Cloud Run — no systemd on container)
│   └── backup.sh                     ← pg_dump → gzip → Neon backup or Object Storage (cron daily)
│
├── src/
│   ├── main/
│   │   ├── java/dev/chinh/portfolio/
│   │   │   │
│   │   │   ├── PortfolioApplication.java
│   │   │   │
│   │   │   ├── auth/
│   │   │   │   ├── session/
│   │   │   │   │   ├── Session.java              ← @Entity: id, userId, refreshToken, expiresAt
│   │   │   │   │   ├── SessionRepository.java
│   │   │   │   │   └── SessionService.java
│   │   │   │   ├── jwt/
│   │   │   │   │   ├── JwtService.java           ← issue/validate RS256 tokens
│   │   │   │   │   ├── JwtAuthenticationFilter.java
│   │   │   │   │   └── JwksController.java       ← GET /api/v1/auth/jwks.json (public)
│   │   │   │   ├── oauth2/
│   │   │   │   │   └── GoogleOAuth2Handler.java
│   │   │   │   ├── user/
│   │   │   │   │   ├── User.java                 ← @Entity: id, email, passwordHash, role
│   │   │   │   │   ├── UserRepository.java
│   │   │   │   │   ├── UserService.java
│   │   │   │   │   ├── UserDto.java
│   │   │   │   │   └── UserMapper.java
│   │   │   │   └── AuthController.java           ← /auth/login, /register, /refresh, /logout
│   │   │   │
│   │   │   ├── platform/
│   │   │   │   ├── metrics/
│   │   │   │   │   ├── ProjectHealth.java        ← @Entity: projectId, uptime, responseTime,
│   │   │   │   │   │                                 lastDeploy, lastUpdated
│   │   │   │   │   ├── ProjectHealthRepository.java
│   │   │   │   │   ├── ProjectHealthDto.java
│   │   │   │   │   ├── MetricsAggregationService.java  ← @Scheduled 60s + manual trigger
│   │   │   │   │   └── MetricsMapper.java
│   │   │   │   ├── websocket/
│   │   │   │   │   ├── MetricsWebSocketHandler.java   ← TextWebSocketHandler: sessions + broadcast
│   │   │   │   │   └── WebSocketConfig.java           ← @EnableWebSocket, /ws/metrics
│   │   │   │   ├── webhook/
│   │   │   │   │   ├── GitHubWebhookController.java   ← POST /api/v1/webhooks/github
│   │   │   │   │   └── HmacVerificationService.java   ← HMAC-SHA256 check
│   │   │   │   ├── contact/
│   │   │   │   │   ├── ContactSubmission.java    ← @Entity: id, email, message, referralSource,
│   │   │   │   │   │                                 createdAt, isRead
│   │   │   │   │   ├── ContactSubmissionRepository.java
│   │   │   │   │   ├── ContactSubmissionDto.java
│   │   │   │   │   ├── ContactService.java       ← submit (rate-check → save)
│   │   │   │   │   ├── ContactMapper.java
│   │   │   │   │   └── ContactController.java    ← POST /api/v1/contact-submissions
│   │   │   │   ├── github/
│   │   │   │   │   └── GitHubProxyService.java   ← RestClient + @Cacheable TTL 1h
│   │   │   │   └── admin/
│   │   │   │       ├── AdminContactController.java    ← GET /api/v1/admin/contacts (Owner)
│   │   │   │       └── AdminAnalyticsController.java  ← GET /api/v1/admin/analytics
│   │   │   │
│   │   │   ├── apps/
│   │   │   │   └── wallet/                       ← Sprint 2+ (empty placeholder)
│   │   │   │       └── .gitkeep
│   │   │   │
│   │   │   └── shared/
│   │   │       ├── error/
│   │   │       │   ├── GlobalExceptionHandler.java   ← @ControllerAdvice (only one in codebase)
│   │   │       │   ├── ErrorResponse.java            ← record: ErrorDetail error
│   │   │       │   ├── ErrorDetail.java              ← record: String code, String message
│   │   │       │   └── EntityNotFoundException.java  ← custom 404
│   │   │       ├── ratelimit/
│   │   │       │   └── RateLimitService.java         ← IP-based check (3 rule sets)
│   │   │       ├── security/
│   │   │       │   └── SecurityConfig.java           ← filter chain, CORS, OAuth2, JWT RS
│   │   │       └── config/
│   │   │           ├── AppConfig.java               ← RestClient bean
│   │   │           ├── CacheConfig.java             ← @EnableCaching + Caffeine spec
│   │   │           └── DemoAppRegistry.java         ← loads demo-apps.yml at startup
│   │   │
│   │   └── resources/
│   │       ├── application.yml               ← Shared config
│   │       ├── application-dev.yml           ← Local PostgreSQL, debug logging
│   │       ├── application-prod.yml          ← Cloud Run config, Neon PostgreSQL, info logging
│   │       ├── demo-apps.yml                 ← Config-driven demo app registry (FR33)
│   │       └── db/
│   │           └── migration/
│   │               ├── V1__create_users_table.sql
│   │               ├── V2__create_sessions_table.sql
│   │               ├── V3__create_contact_submissions_table.sql
│   │               └── V4__create_project_health_table.sql
│   │
│   └── test/
│       └── java/dev/chinh/portfolio/
│       │   ├── TestcontainersConfiguration.java  ← @TestConfiguration, PostgreSQL container
│       │   ├── auth/
│       │   │   ├── session/SessionServiceTest.java
│       │   │   ├── jwt/JwtServiceTest.java
│       │   │   └── AuthControllerTest.java       ← @SpringBootTest + Testcontainers
│       │   ├── platform/
│       │   │   ├── metrics/MetricsAggregationServiceTest.java
│       │   │   ├── contact/ContactServiceTest.java
│       │   │   └── webhook/HmacVerificationServiceTest.java
│       │   └── shared/
│       │       └── error/GlobalExceptionHandlerTest.java
│       └── resources/
│           └── application-test.yml          ← Testcontainers DB override (no real DB needed)
│
├── pom.xml
├── .env.example                      ← DB_URL, DB_USER, DB_PASSWORD, JWT_PRIVATE_KEY,
│                                         GITHUB_WEBHOOK_SECRET, GOOGLE_CLIENT_ID/SECRET
├── .gitignore
├── DECISIONS.md                      ← BE architectural decisions log
└── README.md
```

**`deploy/portfolio-platform.service` (systemd template):**
```ini
[Unit]
Description=Portfolio Platform API
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/opt/portfolio-platform
ExecStart=/usr/bin/java -jar -Dspring.profiles.active=prod portfolio-platform.jar
Restart=always
RestartSec=10
EnvironmentFile=/opt/portfolio-platform/.env

[Install]
WantedBy=multi-user.target
```

**`resources/demo-apps.yml` schema (FR33 — zero code change to register new app):**
```yaml
apps:
  - id: wallet-app
    name: "Wallet App"
    healthEndpoint: "https://wallet.chinh.dev/health"
    metricsEndpoint: "https://wallet.chinh.dev/metrics"
    pollIntervalSeconds: 60
  # Add new demo app here — restart BE to activate
  # - id: new-app
  #   name: "New App"
  #   healthEndpoint: "https://newapp.chinh.dev/health"
  #   metricsEndpoint: "https://newapp.chinh.dev/metrics"
  #   pollIntervalSeconds: 60
```

**`src/test/resources/application-test.yml`:**
```yaml
spring:
  datasource:
    url: ${TEST_DB_URL}      # Provided by Testcontainers at runtime
  flyway:
    enabled: true            # Run migrations on test DB
  jpa:
    hibernate:
      ddl-auto: validate     # Validate schema against migrations
```

---

### Complete Project Tree: `portfolio-docs`

*Public repository — AI planning artifacts only. Decoupled from code deployments.*

```
portfolio-docs/
│
├── brainstorming/
│   └── brainstorming-session-2026-02-25.md
│
├── planning-artifacts/
│   ├── prd.md                    ← Product Requirements Document
│   ├── ux-design-specification.md  ← UX patterns & component specs
│   └── architecture.md           ← This document
│
├── .gitignore
└── README.md                     ← "How this portfolio was built" narrative
```

**Workflow:**
- `portfolio-docs` is the single source of truth for all AI-generated planning artifacts
- Updated manually after each BMAD session: `git add . && git commit -m "..." && git push`
- Public visibility: demonstrates the end-to-end AI-assisted planning process as a portfolio artifact
- `BuildStoryPreview` in `portfolio-fe` reads `DECISIONS.md` from `portfolio-fe` repo (same repo, via GitHub raw API)

---

### Architectural Boundaries

#### API Boundary Map

```
Public endpoints (no auth):
  POST /api/v1/auth/login
  POST /api/v1/auth/register
  POST /api/v1/auth/refresh
  POST /api/v1/auth/logout
  GET  /api/v1/auth/jwks.json
  POST /api/v1/contact-submissions          ← rate limited 3/day/IP + honeypot
  GET  /api/v1/project-health               ← live metrics for Portfolio FE
  GET  /api/v1/github/contributions         ← GitHub API proxy (cached 1h)
  POST /api/v1/webhooks/github              ← HMAC-SHA256 verified
  GET  /api-docs                            ← OpenAPI spec

Admin endpoints (Owner role only):
  GET   /api/v1/admin/contacts
  GET   /api/v1/admin/contacts/{id}
  PATCH /api/v1/admin/contacts/{id}/read
  GET   /api/v1/admin/analytics

WebSocket (public — metrics broadcast):
  WS /ws/metrics                            ← BE → FE one-way broadcast
```

#### Component Communication Patterns

```
FE internal data flow:
  App.tsx init:
    useWebSocket() → connects /ws/metrics → writes to metricsStore
    MotionConfig(reducedMotion="user") → all Framer Motion respects prefers-reduced-motion

  Section renders:
    Hero → HeroCardStack → reads metricsStore (top 1 project live metrics)
    Projects → ProjectCard[] → each reads metricsStore[projectId]
    Contact → ContactForm → useContactForm → POST /api/v1/contact-submissions

  Project detail (lazy):
    BuildStoryPreview → useGitHubBuildStory → GET /api/v1/github/contributions
                      → AbortController 2s timeout → silent hide on failure

State persistence layer:
  URL params → referralStore (on mount), galleryStore (reactive)
  localStorage (Zustand persist) → themeStore, languageStore, visitorStore, referralStore
  Session only → metricsStore, galleryStore
```

#### Data Boundaries

```
PostgreSQL — 4 tables, package ownership:

  Table               Owned By Package     Access Rule
  ─────────────────────────────────────────────────────
  users               auth.user.*          auth.* only
  sessions            auth.session.*       auth.* only
  contact_submissions platform.contact.*   write: public API; read: admin API
  project_health      platform.metrics.*   write: scheduler; read: public API

Cross-package rule: services never access another package's repository.
  ✅ MetricsAggregationService → ProjectHealthRepository
  ❌ MetricsAggregationService → UserRepository (use UserService interface instead)

In-memory cache (Caffeine, Caffeine spec: maximumSize=100, expireAfterWrite=3600s):
  Cache name             Service                    TTL
  ──────────────────────────────────────────────────────
  "github-contributions" GitHubProxyService         1h
```

---

### Requirements to Structure Mapping

| FR Category | FE Files | BE Files |
|---|---|---|
| Showcase & Discovery (FR1–8, FR43) | `sections/Projects`, `shared/ProjectCard`, `pages/ProjectDetailPage`, `constants/projects.ts` | `platform/metrics/ProjectHealth*` |
| Live Evidence (FR9–13) | `hooks/useWebSocket`, `stores/metricsStore`, `shared/StatusIndicator`, `shared/MetricPair` | `platform/metrics/*`, `platform/websocket/*` |
| Recruiter Engagement (FR14–17, FR46) | `sections/Contact`, `shared/ContactForm`, `hooks/useContactForm`, `stores/referralStore`, `stores/visitorStore` | `platform/contact/*`, `shared/ratelimit/*` |
| Nav & Appearance (FR18–23, FR44–47) | `components/layout/*`, `stores/themeStore`, `constants/motion.ts` | — |
| i18n (FR41–42) | `stores/languageStore`, `i18n/*`, `lib/referral.ts` | — |
| Platform Auth (FR24–29) | token handling in fetch hooks | `auth/*`, `shared/security/SecurityConfig` |
| Metrics Platform (FR30–34, FR48) | `hooks/useWebSocket`, `stores/metricsStore` | `platform/metrics/*`, `platform/websocket/*`, `platform/webhook/*`, `platform/github/*` |
| Admin (FR35–40) | — | `platform/admin/*`, `shared/ratelimit/*`, `RateLimitService` |
| API Contract (FR49–51) | `types/api.d.ts` (generated) | `shared/error/*`, all Controller + Dto classes |

---

### Integration Points

#### Data Flow: Metrics Pipeline

```
[GitHub push event]
  → POST /api/v1/webhooks/github
  → HmacVerificationService (HMAC-SHA256)
  → MetricsAggregationService.triggerRefresh()

[@Scheduled every 60s] OR [webhook trigger]
  → MetricsAggregationService
  → GET {app.healthEndpoint} for each entry in demo-apps.yml
  → Save ProjectHealth entity
  → MetricsWebSocketHandler.broadcast(ProjectHealthDto as JSON)
  → All connected /ws/metrics sessions receive METRICS_UPDATE

[Portfolio FE — useWebSocket in App.tsx]
  → onMessage: parse {type, payload}
  → type=METRICS_UPDATE → metricsStore.setMetrics(payload.projectId, payload)
  → ProjectCard + HeroCardStack re-render reactively
```

#### FE → BE Type Contract Maintenance

```
When BE API changes:
  1. BE developer updates Controller + Dto
  2. Run BE locally → /api-docs updates
  3. In portfolio-fe: npx openapi-typescript http://localhost:8080/api-docs -o src/types/api.d.ts
  4. TypeScript compilation catches FE type mismatches
  5. Fix FE usages → commit both repos
```

---

### Development Workflow

**Local setup:**
```bash
# Terminal 1: Spring Boot (dev profile, local PostgreSQL)
cd portfolio-platform && ./mvnw spring-boot:run -Dspring-boot.run.profiles=dev

# Terminal 2: Vite dev server (proxy → :8080)
cd portfolio-fe && pnpm dev
# http://localhost:5173 — /api/* and /ws/* proxied to :8080
```

**Deploy:**
```bash
# FE — automatic on push to main (Vercel GitHub integration)
git push origin main

# BE — GitHub Actions ci-cd.yml on push to main:
#   1. mvn clean package -DskipTests
#   2. SCP target/portfolio-platform.jar to Google Cloud Run + Neon
#   3. SSH: sudo systemctl restart portfolio-platform
```

---

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:**
All technology choices are compatible: Vite 7.3.1 + React 18 + TypeScript + Tailwind CSS v4 +
shadcn/ui 2.4.x are validated together. Spring Boot 3.5.x + Java 21 LTS + PostgreSQL + Flyway +
Caffeine are a proven, stable stack. OpenAPI-generated types (`openapi-typescript`) bridge FE/BE
type safety without shared packages. No version conflicts identified.

**Pattern Consistency:**
Implementation patterns (23 conflict points resolved) fully support architectural decisions.
Naming conventions consistent: snake_case DB, camelCase JSON, PascalCase React, camelCase Java
variables. `useWebSocket` hook → `metricsStore` → reactive UI is a coherent data flow. Zustand
`persist` middleware aligns with localStorage strategy.

**Structure Alignment:**
GitHub Org + 3 repos cleanly separates concerns: code deploys independently, `portfolio-docs`
holds AI planning artifacts publicly. `DECISIONS.md` in each code repo enables `BuildStoryPreview`
to fetch from the correct location. Package boundaries (`auth.*`, `platform.*`, `apps.*`,
`shared.*`) respected in both directory structure and access rules.

### Requirements Coverage Validation ✅

**Functional Requirements Coverage:**
All 51 FRs verified architecturally supported across 9 FR categories:
- FR1–8, FR43: Showcase & Discovery → `sections/Projects`, `shared/ProjectCard`, `constants/projects.ts`
- FR9–13: Live Evidence → `useWebSocket`, `metricsStore`, `platform/websocket/*`
- FR14–17, FR46: Recruiter Engagement → `shared/ContactForm`, `platform/contact/*`, `shared/ratelimit/*`
- FR18–23, FR44–47: Nav & Appearance → `components/layout/*`, `constants/motion.ts`
- FR24–29: Auth → `auth/*`, `shared/security/SecurityConfig`
- FR30–34, FR48: Metrics Platform → `platform/metrics/*`, `platform/webhook/*`, `platform/github/*`
- FR35–40: Admin → `platform/admin/*`, `shared/ratelimit/*`
- FR41–42: i18n → `stores/languageStore`, `i18n/*`, `lib/referral.ts`
- FR49–51: API Contract → `shared/error/*`, all Controller + Dto classes

**Non-Functional Requirements Coverage:**
43 NFRs reviewed. Two gaps identified and resolved during validation:

- **NFR-R7 (smoke tests post-deploy):** Added `deploy/smoke-tests.sh` to `portfolio-platform`
  structure — script pings `/actuator/health` and `/api/v1/project-health` after systemd restart.
- **NFR-S (honeypot pattern):** Contact form honeypot field (`aria-hidden` input, name="url")
  added to `ContactForm.tsx` spec — BE validates field is empty on POST `/api/v1/contact-submissions`.

All remaining NFRs (performance, security, accessibility, observability, compliance) fully covered
by Bucket4j rate limiting, Spring Security, WCAG-2.1 AA patterns, UptimeRobot + Logback, NFR-O4
no-PII-in-logs rule.

### Implementation Readiness Validation ✅

**Decision Completeness:**
All critical decisions documented with exact versions: Vite 7.3.1, React 18, shadcn/ui 2.4.x,
Tailwind CSS v4, Spring Boot 3.5.x, Java 21, Flyway, Caffeine. Starter commands documented for
both repos. Sprint 0 initialization sequence is unambiguous.

**Structure Completeness:**
Complete file trees defined for all 3 repositories. All components, hooks, stores, services,
repositories, controllers, and config classes named and located. `DECISIONS.md` placed in each
code repo root with minimum schema (markdown table: `# | Decision | Rationale | Date`).
Sprint 0 must initialize `DECISIONS.md` in both `portfolio-fe` and `portfolio-platform` root.
`BuildStoryPreview` fetches this file via GitHub raw API — empty/malformed file = silent hide
(acceptable per UX spec AbortController 2s timeout behavior).

**Pattern Completeness:**
23 conflict points resolved across naming, structure, format, communication, and process patterns.
15 mandatory enforcement rules documented. Anti-pattern reference table provided. All pattern
categories have concrete examples.

**BE Test Coverage:**
No explicit % target — intentional decision. Testcontainers integration tests with full
application context load typically achieve >70% coverage organically. JaCoCo optional (can be
added to `pom.xml` in Sprint 0 if enforcement desired). FE coverage threshold ≥60% enforced via
`vitest.config.ts` (NFR-M4).

### Gap Analysis Results

**Critical Gaps:** None identified.

**Important Gaps (resolved):**
- NFR-R7: `deploy/smoke-tests.sh` added to `portfolio-platform` structure
- Honeypot: `aria-hidden` input pattern specified for `ContactForm.tsx`
- `DECISIONS.md` schema: Minimum format defined (markdown table). Sprint 0 must initialize file
  in both code repos. `BuildStoryPreview` fetches via GitHub raw API; silent hide on failure is
  acceptable fallback.

**Nice-to-Have Gaps (deferred):**
- `docker-compose.yml` for local PostgreSQL (devs can use system PostgreSQL or manual setup)
- npm script `openapi:gen` for one-command type regeneration (flow documented in Integration Points)
- JaCoCo BE coverage enforcement (organic coverage via Testcontainers expected to be adequate)

### Validation Issues Addressed

No blocking issues found. Two important gaps resolved during validation (smoke tests + honeypot).
Advanced Elicitation (Tree of Thoughts) applied to repository structure — adopted GitHub
Organization + 3-repo structure (Path D, score 79/80) with `portfolio-docs` as public AI
planning artifact repo. Party Mode (Winston, Amelia, Quinn) confirmed implementation readiness
and added `DECISIONS.md` schema spec + BE coverage rationale.

### Architecture Completeness Checklist

**✅ Requirements Analysis**

- [x] Project context thoroughly analyzed (10 cross-cutting concerns mapped)
- [x] Scale and complexity assessed (personal portfolio, solo dev, free-tier)
- [x] Technical constraints identified (Google Cloud Run, Vercel, WebSocket-Cloud Run contract)
- [x] Cross-cutting concerns mapped (auth, real-time, motion, i18n, rate limiting, config registry)

**✅ Architectural Decisions**

- [x] Critical decisions documented with exact versions
- [x] Technology stack fully specified (FE + BE + deployment)
- [x] Integration patterns defined (OpenAPI types, WS protocol, GitHub API cache)
- [x] Performance considerations addressed (Caffeine TTL, WS broadcast, Vite build)

**✅ Implementation Patterns**

- [x] Naming conventions established (23 conflict points, 15 mandatory rules)
- [x] Structure patterns defined (co-located tests, feature-by-type FE, package-by-feature BE)
- [x] Communication patterns specified (WS message format, API response format, error structure)
- [x] Process patterns documented (error handling 3-tier, loading state `isLoading`, form pattern)

**✅ Project Structure**

- [x] Complete directory structure defined (all 3 repos)
- [x] Component boundaries established (shadcn/ui boundary, package boundaries)
- [x] Integration points mapped (API boundary map, data flow diagrams)
- [x] Requirements to structure mapping complete (FR category → file mapping table)

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** HIGH — comprehensive validation across coherence, coverage, and readiness.
All 51 FRs and 43 NFRs architecturally supported. No blocking gaps. All important gaps resolved.

**Key Strengths:**
- Type safety bridge via OpenAPI-generated types eliminates FE/BE contract drift
- `useWebSocket` single-point hook prevents WS lifecycle conflicts across agents
- `DECISIONS.md` per code repo (minimum schema defined) enables `BuildStoryPreview` from Sprint 1
- GitHub Org + 3 repos cleanly separates code, deployment, and AI planning artifacts
- 23 conflict points pre-resolved prevents agent inconsistency across sprints
- Party Mode (5 sessions) + Advanced Elicitation contributed 23 total improvements to architecture

**Areas for Future Enhancement:**
- Sprint 2+: `apps/wallet/` package (placeholder ready, entry point defined)
- Phase 2 hooks (Concern #10): testimonials, blog, case studies — architectural hooks pre-defined
- `docker-compose.yml` for local dev PostgreSQL (optional convenience)
- JaCoCo BE coverage enforcement (optional Sprint 0 addition)

### Implementation Handoff

**AI Agent Guidelines:**
- Follow all architectural decisions exactly as documented in this file
- Use implementation patterns (Section 5) consistently across all components
- Respect project structure and package boundaries — cross-package repository access is forbidden
- Initialize `DECISIONS.md` in both `portfolio-fe` and `portfolio-platform` root in Sprint 0
- Refer to this document for all architectural questions before making implementation decisions

**First Implementation Priority:**
```bash
# Sprint 0 — Step 1: Create GitHub Organization and 3 repos
# github.com/chinhnt-portfolio-v2/ → portfolio-fe, portfolio-platform, portfolio-docs

# Initialize portfolio-fe:
npm create vite@latest portfolio-fe -- --template react-ts

# Initialize portfolio-platform (Spring Boot 3.5.x):
curl https://start.spring.io/starter.tgz \
  -d bootVersion=3.4.x -d javaVersion=21 \
  -d dependencies=web,security,data-jpa,postgresql,websocket,oauth2-client,oauth2-resource-server,actuator,validation,devtools \
  | tar -xzvf -
```