---
stepsCompleted: ['step-01-validate-prerequisites', 'step-02-design-epics', 'step-03-create-stories', 'step-04-final-validation']
workflowComplete: true
inputDocuments:
  - '_bmad-output/planning-artifacts/prd.md'
  - '_bmad-output/planning-artifacts/architecture.md'
  - '_bmad-output/planning-artifacts/ux-design-specification.md'
---

# portfolio-v2 - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for portfolio-v2, decomposing the requirements from the PRD, UX Design, and Architecture requirements into implementable stories.

## Requirements Inventory

### Functional Requirements

FR1: Recruiter can view featured projects as the primary section on portfolio entry (projects-first layout)
FR2: Recruiter can view each project in a visually distinct gallery card with generous whitespace (≥ 16px padding) and contextual description
FR3: Recruiter can read the WHY behind each project build (artist statement), not just the WHAT
FR4: Recruiter can view Chính's background through recruiter-centric framing ("Why Hire Me")
FR5: Recruiter can view Chính's skills with each claim linked to a verifiable proof artifact
FR6: Recruiter can view "What I'd do differently" per project (collapsible, honest lessons learned)
FR7: Recruiter can view a verifiable build timeline for each project (first commit → production deploy date)
FR8: Recruiter can view Chính's current job availability status (including location preference)
FR9: Recruiter can view real-time health metrics for each demo app directly on the project card (uptime %, response time ms, last deploy)
FR10: Recruiter can see the timestamp of the last metrics update ("Updated X ago") when real-time data is unavailable
FR11: System automatically hides metrics when data has been offline for more than 24 hours — never displays stale or misleading numbers
FR12: Recruiter can view Chính's live GitHub contribution graph (not a screenshot)
FR13: Portfolio self-displays its own Lighthouse performance score as a trust signal (Phase 2)
FR14: Recruiter can submit a contact inquiry via form
FR15: Recruiter receives an animated success confirmation (confetti burst or checkmark animation, ≤ 1.5s duration) immediately after successfully submitting the contact form
FR16: Portfolio reads referral source from URL parameter (?from=) and saves to local storage
FR17: Portfolio delivers a streamlined returning-visitor experience — e.g., reduced onboarding elements and quick access to last-viewed content — based on locally persisted browsing context
FR18: Recruiter can navigate the entire portfolio using keyboard only
FR19: Portfolio content is accessible via screen readers (WCAG 2.1 AA)
FR20: Portfolio renders correctly on all modern evergreen browsers and all screen sizes from 320px to 4K
FR21: Recruiter can use command palette to jump to any section or project (Phase 2)
FR22: Recruiter can view a video walkthrough (≤90 seconds) per project (Phase 2)
FR23: Portfolio auto-syncs project page content from the corresponding GitHub repository README (Phase 2)
FR24: Platform BE supports email/password authentication (JWT-based). Wallet App exposes only Google OAuth login — email/password available for future demo apps.
FR25: Demo app user can log in with a Google account
FR26: Demo app user's session automatically refreshes without requiring re-login within the session window
FR27: Demo app user can log out and invalidate their session
FR28: Platform ensures each user can only access their own data (user-scoped isolation)
FR29: Platform exposes a public JWKS endpoint for demo apps to validate JWT tokens independently
FR30: Platform BE aggregates health metrics from all registered demo apps by polling each app's /health endpoint every 60 seconds. Detection latency ≤ 60s; WebSocket delivery latency after detection < 3s.
FR31: Portfolio FE receives live health metric updates via WebSocket connection from Platform BE
FR32: Platform BE automatically refreshes health metrics for a project upon receiving a GitHub push event webhook
FR33: Owner can register a new demo app to the platform without changing application code anywhere (config-only registration)
FR34: Platform exposes AI Build Story per project — structured narrative min 300 words, highlighting at least 2 decisions where developer overrode AI suggestion (Phase 2)
FR35: Owner can update showcase configuration (project list, metadata, artist statements, build timelines) by editing config file and pushing to deploy
FR36: Owner can update job availability status by editing config file
FR37: Owner can view submitted contact form inquiries via private admin endpoint
FR38a: Owner can view page-level analytics (pageviews, bounce rate, traffic sources) via Plausible dashboard — external tool, no BE implementation required
FR38b: Owner can view portfolio-specific engagement data (contact form submission count, ?from= referral source breakdown) via private admin endpoint /api/v1/admin/analytics
FR39: System enforces rate limits on contact form submissions to prevent spam
FR40: System enforces rate limits on AI-powered endpoints to control third-party API costs (Phase 2)
FR41: Recruiter can switch portfolio language between Vietnamese and English
FR42: Portfolio remembers recruiter's language preference across return visits (locally persisted)
FR43: Recruiter can view a full project detail page with its own shareable URL (/projects/{slug}) — includes gallery, build timeline, lessons learned. AI Build Story section appears in Phase 2.
FR44: Recruiter can switch appearance between dark mode and light mode
FR45: Portfolio remembers recruiter's appearance preference across return visits (locally persisted)
FR46: Portfolio displays contextual messaging appropriate to referral source — e.g., ?from=linkedin surfaces English-first professional framing; ?from=cv-vn surfaces Vietnamese-friendly context
FR47: Recruiter experiences spring-physics micro-animations on portfolio interactions (hover effects, scroll reveals, page transitions); all animation is disabled when prefers-reduced-motion OS setting is active
FR48: System verifies webhook signature (HMAC-SHA256 with GitHub secret) before processing any incoming webhook
FR49: Platform BE manages four core data entities: User (identity, auth state), Session (refresh token store), ContactSubmission (form inquiry data), and ProjectHealth (aggregated metrics snapshot); all user-owned entities are scoped to the authenticated user ID — cross-user data access returns 403
FR50: Platform BE returns structured JSON error responses with a consistent {"error": {"code": "...", "message": "..."}} format for all 4xx and 5xx responses — no raw stack traces or unstructured error strings
FR51: Platform BE API is self-documented via OpenAPI specification accessible at /api-docs — spec covers all /api/v1/ endpoints including request/response schemas and authentication requirements

### NonFunctional Requirements

**Performance:**
NFR-P1: Portfolio FE Largest Contentful Paint < 1.5s (Vercel CDN, static assets)
NFR-P2: Portfolio FE total bundle size (gzipped) < 300KB
NFR-P3: Portfolio FE Lighthouse Performance score ≥ 90 (CI gate) / ≥ 95 (aspirational)
NFR-P4: Platform BE internal API response time (p95) ≤ 200ms (CRUD, auth, metrics aggregation)
NFR-P4b: Platform BE proxy/external API response time ≤ 2s with hard timeout (GitHub API proxy, future AI endpoints)
NFR-P5: WebSocket health metric delivery latency < 3s after Platform BE detects change
NFR-P6: Portfolio FE Time to Interactive < 2s
NFR-P7: Contact form submission feedback < 1s (perceived performance)

**Security:**
NFR-S1: All client-server communication HTTPS only (TLS 1.2+)
NFR-S2: JWT access token TTL = 15 minutes
NFR-S3: JWT refresh token TTL = 7 days
NFR-S4: Password storage bcrypt (cost factor ≥ 12)
NFR-S5: User data isolation — query-level enforcement, every DB query scoped to authenticated user ID
NFR-S6: Incoming webhook verification — HMAC-SHA256 signature check
NFR-S7: Admin endpoints — authenticated + role-based, only Owner can access /api/v1/admin/*
NFR-S8: CORS policy — explicit allow-list only (Vercel production domain + localhost dev, no wildcard)
NFR-S9: Rate limiting — contact 3/day/IP; AI endpoints 5 req/10min/IP; general 100/min/IP
NFR-S10: Wallet App financial data — HTTPS sufficient (no at-rest encryption required)

**Reliability:**
NFR-R1: Platform BE uptime ≥ 99.9% (≤ 8.7h downtime/year), monitored via UptimeRobot
NFR-R2: Portfolio FE availability 99.99% (Vercel CDN SLA)
NFR-R3: WebSocket reconnection — automatic with exponential backoff, max 3 retry attempts, then fallback to polling
NFR-R4: Graceful degradation — Portfolio FE fully renders without BE; dynamic features degrade gracefully
NFR-R5: Metrics staleness handling — hide after 24h offline, "Updated X ago" visible
NFR-R6: Database backup — daily automated backup, RPO 24h (cron → PostgreSQL dump → Oracle Object Storage)
NFR-R7: Critical happy path smoke tests pass on every deploy

**Accessibility:**
NFR-A1: WCAG compliance level 2.1 AA (all Portfolio FE pages and interactive elements)
NFR-A2: Color contrast ratio ≥ 4.5:1 (text) / ≥ 3:1 (large text & UI components) — both dark and light mode
NFR-A3: Keyboard navigation — 100% interactive elements reachable, tab order logical, focus indicators visible
NFR-A4: Screen reader compatibility — all content accessible via ARIA (alt text, form labels, landmark regions)
NFR-A5: Motion sensitivity — respect prefers-reduced-motion; all animated components must check media query (NO EXCEPTIONS)

**Maintainability:**
NFR-M1: Platform BE — Auth module unit test coverage ≥ 80%
NFR-M2: Platform BE — Metrics aggregation unit test coverage ≥ 80%
NFR-M3: Wallet App — Business logic unit test coverage ≥ 70%
NFR-M4: Portfolio FE — Utility functions unit test coverage ≥ 60%
NFR-M5: Package structure discipline enforced from Sprint 1 (auth.*, platform.*, apps.* separation; no cross-package direct dependencies outside defined contracts)

**Integration:**
NFR-I1: GitHub API dependency — cached TTL 1h, in-memory; fallback to last cached data; if API unavailable after BE restart → hide contribution graph section
NFR-I2: Google OAuth2 — standard OAuth2 authorization code flow (no custom token exchange)
NFR-I3: Future AI API integration (Phase 2) — per-user rate limited, hard timeout 10s
NFR-I4: GitHub Webhooks — signature verified + idempotent processing (duplicate delivery must not cause duplicate health refreshes)
NFR-I5: Reverse proxy WebSocket support — Nginx must pass WebSocket upgrade headers without application-layer configuration

**Operational:**
NFR-O1: Portfolio FE deployment — push to main → auto-deploy (Vercel), zero manual steps
NFR-O2: Platform BE deployment — GitHub Actions CI/CD → Oracle A1, automated deploy on merge to main
NFR-O3: Health monitoring — UptimeRobot (free tier, 5-min checks), email alerts on downtime
NFR-O4: Logging — structured JSON logs, configurable log level, no PII in logs
NFR-O5: Infrastructure cost $0/month (Oracle A1 always-free + Vercel free tier)

### Additional Requirements

**From Architecture (Technical Implementation Requirements):**
- Starter templates: Portfolio FE uses `create-vite@latest --template react-ts`; Platform BE uses Spring Initializr (start.spring.io) — NO community boilerplates
- Tech stack is fixed (see Architecture doc): React 18+ / Vite / TypeScript strict / Tailwind v4 CSS-first / shadcn/ui 2.4.x / Framer Motion / Zustand / React Router v7 / react-i18next (FE); Java 21 / Spring Boot 3.4.x / PostgreSQL / Hibernate JPA / Maven / Flyway / Native WebSocket / Spring RestClient / Caffeine cache / springdoc-openapi (BE)
- BE package structure enforced from Sprint 0: auth/, platform/, apps/, shared/ packages — no cross-boundary direct dependencies
- All BE database migrations use Flyway SQL files in `src/main/resources/db/migration/` with `V{n}__{description}.sql` naming
- FE TypeScript types generated from BE OpenAPI spec (single source of truth for API contracts)
- All FE localStorage-persisted state MUST use zustand/middleware persist — no manual localStorage calls
- Spring Boot version MUST be 3.4.x (not 4.0.x — ecosystem not ready)
- WebSocket implementation MUST be native (not STOMP/SockJS — avoids +25KB bundle)
- HTTP client on BE MUST be Spring RestClient (not RestTemplate maintenance mode, not WebClient/reactive overhead)
- Tailwind v4 CSS-first config — NO `tailwind.config.js` file
- shadcn/ui components copied as owned source code (not imported from package)
- PostgreSQL on Oracle A1 (co-located with BE); initial schema: 4 core entities (User, Session, ContactSubmission, ProjectHealth)
- CI/CD: Vercel auto-deploy (FE) + GitHub Actions (BE) → Oracle A1
- Nginx as reverse proxy + SSL termination (Let's Encrypt TLS 1.2+) — must support WebSocket upgrade headers
- Session entity CRITICAL CONSTRAINT: belongs ONLY to auth.session.* package — cross-boundary access returns 403

**From UX Design (UX Implementation Requirements):**
- Responsive breakpoints: Mobile 320px–767px (20px padding), Tablet 768px–1023px (40px padding), Desktop 1024px–1200px (64px padding)
- 12-column grid system with 20px gap; gallery: 3-col desktop → 2-col tablet → 1-col mobile
- Hero layout: desktop/tablet 2-column (1.2fr:1fr), mobile single column
- Three semantic spring physics tokens: spring-gentle (hover/reveals), spring-snappy (navigation), spring-bouncy (celebrations/confirmations)
- Global MOTION_ENABLED context at app root controls all animations (Framer Motion conditional + CSS media query)
- All localStorage access wrapped in try/catch for private browsing graceful degradation
- Gallery smart default driven by document.referrer (GitHub → Technical tab, else Featured tab) — maximum 2 variants only
- Contact form: zero CAPTCHA, only email + message required, confirmation toast auto-dismiss after 2s
- Design system: Geist font family, Electric Purple accent (#A855F7), base 8px spacing unit
- Focus ring: 2px solid purple-400 (#C084FC) with 2px offset (WCAG AA compliant)
- Phase 2 architectural hooks in Phase 1 layout: command palette keyboard event registry, AI rate limit tier in Nginx (commented block), video walkthrough space reservation in project detail layout

### FR Coverage Map

FR1: Epic 1 — Projects-first layout (hero section)
FR2: Epic 1 — Gallery card design with whitespace
FR3: Epic 1 — Artist statement per project
FR4: Epic 1 — "Why Hire Me" / background section
FR5: Epic 1 — Skills with verifiable proof links
FR6: Epic 1 — Collapsible lessons learned per project
FR7: Epic 1 — Verifiable build timeline per project
FR8: Epic 1 — Job availability status display
FR9: Epic 3 — Real-time health metrics on project card (FE display)
FR10: Epic 3 — "Updated X ago" staleness timestamp display
FR11: Epic 3 — Auto-hide metrics after 24h offline
FR12: Epic 3 — Live GitHub contribution graph (not screenshot)
FR13: Phase 2 — Lighthouse score self-display (out of scope)
FR14: Epic 4 — Contact inquiry form
FR15: Epic 4 — Animated success confirmation (≤ 1.5s, spring-bouncy)
FR16: Epic 4 — ?from= referral source tracking → localStorage
FR17: Epic 4 — Returning visitor streamlined experience
FR18: Epic 1 — Full keyboard navigation
FR19: Epic 1 — WCAG 2.1 AA screen reader accessibility
FR20: Epic 1 — Responsive 320px to 4K, all evergreen browsers
FR21: Phase 2 — Command palette (out of scope)
FR22: Phase 2 — Video walkthroughs per project (out of scope)
FR23: Phase 2 — GitHub README auto-sync (out of scope)
FR24: Epic 5 — Email/password JWT authentication (BE)
FR25: Epic 5 — Google OAuth2 login
FR26: Epic 5 — Automatic session refresh
FR27: Epic 5 — Logout + session invalidation
FR28: Epic 5 — User-scoped data isolation (query-level)
FR29: Epic 5 — Public JWKS endpoint for JWT validation
FR30: Epic 3 — Polling pipeline (60s cadence, BE scheduler)
FR31: Epic 3 — WebSocket live metrics client (FE)
FR32: Epic 3 — GitHub push webhook → metrics refresh
FR33: Epic 3 — Config-only demo app registration (zero code change)
FR34: Phase 2 — AI Build Story per project (out of scope)
FR35: Epic 6 — Config-file-driven showcase content management
FR36: Epic 6 — Job availability via config file
FR37: Epic 6 — Admin endpoint for contact inquiries
FR38a: Epic 6 — Plausible analytics script embed (FE)
FR38b: Epic 6 — Admin analytics endpoint /api/v1/admin/analytics
FR39: Epic 4 — Contact form rate limiting (3/day/IP)
FR40: Epic 2 (Story 2.4) — Nginx commented-out AI endpoint rate limit block as Phase 2 placeholder hook
FR41: Epic 1 — Language toggle vi/en
FR42: Epic 1 — Language preference persistence (localStorage)
FR43: Epic 1 — Project detail page /projects/{slug}
FR44: Epic 1 — Dark/light mode toggle
FR45: Epic 1 — Theme preference persistence (localStorage)
FR46: Epic 4 — Contextual messaging per ?from= referral source
FR47: Epic 1 — Spring-physics micro-animations (prefers-reduced-motion aware)
FR48: Epic 3 — HMAC-SHA256 webhook signature verification
FR49: Epic 2 — 4 core data entities (User, Session, ContactSubmission, ProjectHealth)
FR50: Epic 2 — Structured JSON error format {"error": {"code", "message"}}
FR51: Epic 2 — OpenAPI self-documentation at /api-docs

## Epic List

### Epic 1: Portfolio Static Showcase
Recruiter can visit the portfolio, discover and explore projects, read context, navigate fully from any device, in Vietnamese or English, with full accessibility and complete design system.
**FRs covered:** FR1, FR2, FR3, FR4, FR5, FR6, FR7, FR8, FR18, FR19, FR20, FR41, FR42, FR43, FR44, FR45, FR47
**Est:** ~10 stories

### Epic 2: Platform Backend Core & Infrastructure
Platform BE is deployed on Oracle A1, Nginx running, PostgreSQL schema correct, CI/CD active — Owner can hit /api-docs and see the full API contract.
**FRs covered:** FR49, FR50, FR51 + infrastructure (Oracle A1, Nginx, Let's Encrypt, GitHub Actions CI/CD, PostgreSQL)
**Est:** ~5 stories

### Epic 3: Live Evidence Layer
Recruiter sees real-time health metrics on project cards — proving demo apps are actually running in production — with graceful degradation when data is offline.
**FRs covered:** FR9, FR10, FR11, FR12, FR30, FR31, FR32, FR33, FR48
**Est:** ~8 stories

### Epic 4: Recruiter Contact & Engagement
Recruiter contacts Chính via frictionless form; portfolio intelligently adapts to referral context and returning visit behavior.
**FRs covered:** FR14, FR15, FR16, FR17, FR39, FR46
**Est:** ~5 stories

### Epic 5: Platform Auth & Demo App Identity
Demo app users authenticate securely (Google OAuth2), sessions are managed, users access only their own data — enabling Wallet App launch.
**FRs covered:** FR24, FR25, FR26, FR27, FR28, FR29
**Est:** ~6 stories

### Epic 6: Owner Admin & Analytics
Owner manages portfolio content, reviews contact inquiries, tracks engagement — no code changes needed, only config file + git push.
**FRs covered:** FR35, FR36, FR37, FR38a, FR38b, FR40
**Est:** ~5 stories

**Phase 2 (out of current scope):** FR13, FR21, FR22, FR23, FR34

---

## Epic 1: Portfolio Static Showcase

Recruiter can visit the portfolio, discover and explore projects, read context, navigate fully from any device, in Vietnamese or English, with full accessibility and complete design system.

### Story 1.1: Initialize Portfolio FE Project

As a developer,
I want a fully configured Portfolio FE project with all required tooling,
So that the team can build features on a stable, standards-compliant foundation.

**Acceptance Criteria:**

**Given** no project exists,
**When** `npm create vite@latest portfolio-fe -- --template react-ts` is run,
**Then** the project starts with `npm run dev` showing a working React app in the browser

**Given** all dependencies are installed,
**Then** Tailwind v4 (CSS-first, no `tailwind.config.js`), shadcn/ui CLI, Zustand, Framer Motion, React Router v7, react-i18next, Lucide React, and Vitest are all available and importable

**Given** Vitest is configured,
**When** `npm run test` is run,
**Then** Vitest runs with jsdom environment and reports 0 failing tests

**Given** the folder structure is set up,
**Then** source code follows: `src/components/`, `src/pages/`, `src/store/`, `src/i18n/`, `src/lib/`

**Given** Vercel deployment is needed,
**Then** a `vercel.json` exists with SPA rewrite rules so all routes serve `index.html`

---

### Story 1.2: App Shell, Design System & Theme Toggle

As a recruiter,
I want a visually polished app with dark/light mode that remembers my preference,
So that I can view the portfolio in my preferred appearance on every visit.

**Acceptance Criteria:**

**Given** the app loads,
**When** the page renders,
**Then** Geist font is applied with correct typography scale (display `clamp(36px,6vw,64px)`, H1 40px, body 16px, mono 13px) and Electric Purple (#A855F7) accent

**Given** the recruiter clicks the theme toggle,
**When** switching between modes,
**Then** dark/light mode changes without page reload and color contrast meets ≥ 4.5:1 in both modes

**Given** the recruiter switches to light mode and closes the browser,
**When** they return,
**Then** light mode is restored — state persisted via `zustand/middleware` persist, NOT manual `localStorage` calls

**Given** the app initializes,
**Then** a global `MOTION_ENABLED` context is created by checking `window.matchMedia('(prefers-reduced-motion: reduce)')` — all animated components must consume this context

**Given** the app shell is rendered,
**Then** a sticky navigation header exists with portfolio branding, nav links, theme toggle, and language toggle — keyboard-accessible on all breakpoints

**Given** any section container,
**Then** horizontal padding is: 20px (mobile <768px), 40px (tablet 768–1023px), 64px (desktop ≥1024px)

**Theme toggle component spec (FR44/FR45):**
- Position: top navigation bar, far right
- Appearance: icon button — sun icon (light mode) / moon icon (dark mode)
- Default: dark mode (matches UX spec design direction)
- Behavior: clicking toggles `data-theme` attribute on `<html>` element between 'dark' and 'light'
- Persistence: theme saved to `localStorage` key `pf_theme` ('dark' | 'light')
- Initial load: read `pf_theme` from localStorage; if not set, default to 'dark'
- No system preference override — explicit user choice always wins over `prefers-color-scheme`
- Transition: CSS `transition: background-color 200ms ease, color 200ms ease` on `<html>`
- ARIA: `aria-label="Switch to light mode"` / `"Switch to dark mode"` (dynamic)

---

### Story 1.3: Project Gallery & Hero Section

**Note on live demo URL buttons:** Project cards in this story display static demo URLs from `projects.ts` config as regular `<a href>` links — NOT WebSocket-connected metrics. Real-time uptime/response metrics are added in Epic 3 (Story 3.4) which overlays onto these same cards. Story 1.3 can be fully completed and deployed without Epic 3.

As a recruiter,
I want to see featured projects immediately upon landing in a scannable gallery,
So that I can quickly identify Chính's relevant work without scrolling through bio content.

**Acceptance Criteria:**

**Given** the portfolio homepage loads,
**Then** projects are the primary hero section — bio/about content appears below the fold on desktop

**Given** the hero layout on desktop (≥1024px),
**Then** it uses a 2-column layout (1.2fr : 1fr, gap 48px): copy + CTA on left, stacked project cards on right

**Given** the gallery is rendered,
**When** viewport is ≥1024px,
**Then** projects display in a 3-column grid; at 768–1023px in 2-column; below 768px in 1-column

**Given** a project card is rendered,
**Then** it shows: project name, contextual description, tech stack chips (versioned e.g. "React 18.3"), build timeline summary, and live demo URL button — with ≥ 16px card padding

**Given** `document.referrer` contains "github.com",
**When** the gallery loads,
**Then** the "Technical" tab is selected by default; all other referrers default to "Featured" tab

---

### Story 1.4: Project Detail Page

As a recruiter,
I want a full project page with its own shareable URL,
So that I can bookmark and share specific projects and read the complete decision story.

**Acceptance Criteria:**

**Given** the recruiter navigates to `/projects/{slug}`,
**Then** a project detail page renders with correct project data (gallery, build timeline, tech stack, live URL, GitHub URL)

**Given** the lessons learned section exists,
**When** the recruiter clicks the expand trigger,
**Then** it expands/collapses showing "What I'd do differently" content

**Given** a detail page URL is shared,
**When** a new user opens it directly,
**Then** the correct project detail loads (SPA deep linking via React Router + Vercel rewrites)

**Given** an invalid slug (e.g. `/projects/nonexistent`),
**Then** a "project not found" message is shown with a navigation link back to the gallery

**Given** the detail page layout,
**Then** a reserved placeholder space exists for the Phase 2 "AI Build Story" section — renders cleanly in its absence with no layout shift

**Build Timeline component spec (FR7):**
Display as a numbered vertical milestones list in the project detail page:

```
① First Commit    Jan 15, 2026
② MVP Complete    Feb 10, 2026
③ Production Deploy  Mar 3, 2026
```

- Each milestone has: number badge (circle), label (text), date
- Data source: `projects.ts` config field `timeline.milestones: [{label, date}]`
- Minimum 2 milestones required: First Commit + Production Deploy
- Additional milestones optional (MVP, Beta, etc.)
- Styling: vertical connector line between milestones using CSS border-left
- Mobile: same layout, connector line maintained

---

### Story 1.5: About, Skills & Availability

As a recruiter,
I want to understand Chính's background, verified skills, and current availability,
So that I can assess fit before deciding to reach out.

**Acceptance Criteria:**

Given I am a recruiter viewing the portfolio,
When I scroll to the About / Skills section,
Then I see a concise "Why Hire Me" paragraph (3-4 sentences) written in recruiter-centric framing — emphasizing engineering philosophy, not CV bullet points.

Given I am a recruiter viewing any project card or project detail page,
When I look at the technology stack,
Then I see versioned tech badges (e.g. "React 18", "Spring Boot 3.2") that serve as live proof of skill — each badge links to the corresponding live project or GitHub repo as the primary proof artifact.

Given I am a recruiter viewing the About section,
When I look at the skills displayed,
Then skills are presented in context of real projects — no isolated skill list without proof; every technology mentioned is linked to at least one project that demonstrates it.

Given I am a recruiter viewing the portfolio,
When I check the job availability section,
Then I see a **Job Availability Badge** displaying:
- Current status: one of "Open to Work", "Selectively Looking", "Not Available"
- Location preference (e.g. "Remote · Ho Chi Minh City")
- Badge color: Green (#22C55E) for Open, Yellow (#EAB308) for Selective, Gray (#6B7280) for Unavailable
- Data source: read from `src/config/about.ts` config field `availability: { status, location }`

Given I am a recruiter viewing the portfolio on mobile,
When the About/Skills section renders,
Then the section is fully readable and badges are legible at 320px width.

---

### Story 1.6: Internationalization (vi/en)

As a recruiter,
I want to switch the portfolio between Vietnamese and English,
So that I can read personal content in my preferred language.

**Acceptance Criteria:**

**Given** the recruiter clicks the language toggle,
**Then** UI labels and personal content (About, Why Hire Me, artist statements, project descriptions) switch between vi and en without page reload

**Given** the language is set to Vietnamese and the browser is closed,
**When** the recruiter returns,
**Then** Vietnamese is restored via `zustand/middleware` persist

**Given** the page loads with `?from=linkedin`,
**Then** English is the default language

**Given** the page loads with `?from=cv-vn`,
**Then** Vietnamese is the default language

**Given** `?lang=vi` or `?lang=en` is in the URL,
**Then** it overrides the `?from=` default regardless of referral source

**Given** no URL params exist,
**Then** browser language (`navigator.language`) is used as fallback

**Given** technical content is displayed (code snippets, metrics, build timelines, GitHub data),
**Then** it stays in English regardless of selected language

**Language toggle component spec (FR41/FR42):**
- Position: top navigation bar, right side (next to theme toggle)
- Appearance: pill toggle or text button: `EN | VI` — active language underlined or highlighted
- Behavior: clicking switches all translated content instantly (no page reload)
- Persistence: chosen language saved to `localStorage` key `pf_language` ('en' | 'vi')
- Initial load logic:
  1. Check `pf_language` in localStorage → use if present
  2. Else check `?lang=` URL param → use if present
  3. Else check `?from=` param: 'linkedin' → 'en', 'cv-vn' → 'vi'
  4. Else use browser `navigator.language` (vi-* → 'vi', else → 'en')
- ARIA: `aria-label="Switch language"`, `lang` attribute on `<html>` updated on switch

---

### Story 1.7: Accessibility & Responsive Completion

> **Note: Story 1.7 was identified as oversized. Split into 1.7.1 (Keyboard & Screen Reader) and 1.7.2 (Visual & Responsive) for better tracking.**

As a recruiter with accessibility needs,
I want to navigate the entire portfolio via keyboard and screen reader,
So that no assistive technology barrier prevents me from evaluating the work.

**Acceptance Criteria:**

**Given** any interactive element,
**When** navigating via Tab key,
**Then** all interactive elements are reachable in logical order with a visible 2px solid purple-400 (#C084FC) focus ring with 2px offset

**Given** a screen reader is active,
**When** navigating the portfolio,
**Then** all images have descriptive alt text, all form inputs have labels, and landmark regions (header, main, nav, footer) are correctly marked

**Given** any text or UI color,
**Then** contrast ratio meets ≥ 4.5:1 for text and ≥ 3:1 for large text/UI components in BOTH dark and light mode independently

**Given** a 320px wide viewport,
**Then** no horizontal scroll occurs and all content is readable without zooming

**Given** an automated accessibility audit (e.g. axe-core),
**Then** 0 critical or serious WCAG 2.1 AA violations are reported on any page

---

### Story 1.7.1: Keyboard Navigation & Screen Reader Accessibility

**As a** recruiter with motor or visual impairments,
**I want** the portfolio fully navigable by keyboard and screen reader,
**so that** I can access all content without a mouse.

**Acceptance Criteria:**

Given keyboard-only navigation,
When I press Tab through all interactive elements,
Then every button, link, card, and form field is reachable in a logical order; none are skipped.

Given I use a screen reader (VoiceOver / NVDA),
When I navigate the portfolio,
Then all images have descriptive `alt` text; all form inputs have `<label>` elements; landmark regions (`<nav>`, `<main>`, `<section>`) use `aria-labelledby`.

Given keyboard focus moves between elements,
When an element receives focus,
Then a visible focus ring is shown (outline: 2px solid var(--color-primary), offset 2px) — no element has `outline: none` without replacement.

Given a modal or drawer opens,
When it opens,
Then focus is trapped inside until closed; Escape key closes it; focus returns to the trigger element.

A "Skip to main content" link exists as the first focusable element; visible on focus.

---

### Story 1.7.2: Visual Accessibility, Color Contrast & Responsive Verification

**As a** recruiter on any device,
**I want** the portfolio to render correctly and meet visual accessibility standards,
**so that** I can read and use it on any screen size.

**Acceptance Criteria:**

Given dark mode is active,
When color contrast is measured,
Then all text/background pairs meet ≥ 4.5:1 (normal text) and ≥ 3:1 (large text ≥ 18px or bold ≥ 14px) — verified with axe-core CI check.

Given light mode is active,
When color contrast is measured,
Then same contrast requirements met independently.

Given the viewport is 320px wide (minimum),
When the portfolio renders,
Then all content is visible without horizontal scroll; no text truncation; touch targets ≥ 44px.

Given the viewport is 2560px (4K),
When the portfolio renders,
Then layout uses max-width constraints; content does not stretch to full 4K width; readability maintained.

Lighthouse CI accessibility score ≥ 90 passes on every PR (enforced as CI gate alongside performance score).

---

### Story 1.8: Spring Physics Animations

As a recruiter,
I want portfolio interactions to feel responsive and polished with subtle spring animations,
So that micro-interactions signal intentional craft and attention to detail.

**Acceptance Criteria:**

**Given** the recruiter hovers over a project card,
**Then** a spring-gentle animation (stiffness 300, damping 30) triggers a subtle hover reveal effect

**Given** the recruiter navigates between pages,
**Then** a spring-snappy transition (stiffness 400, damping 25) animates the page change

**Given** first visit hero load,
**Then** hero elements animate in sequentially using spring-gentle tokens

**Given** the OS `prefers-reduced-motion` is enabled,
**Then** ALL Framer Motion animations are completely disabled — all state changes are instant with no transitions

**Given** any animated component in the codebase,
**Then** it reads from the `MOTION_ENABLED` context before applying any animation — no component may bypass this check

---

## Epic 2: Platform Backend Core & Infrastructure

Platform BE is deployed on Oracle A1, Nginx running, PostgreSQL schema correct, CI/CD active — Owner can hit /api-docs and see the full API contract.

**Epic Goal (user value framing):** Upon completing this epic, the Portfolio FE can connect to a live, secure, and documented backend API — enabling all dynamic features (live metrics, contact form, auth) to function. Owner can verify the API via Swagger UI at /api-docs.

### Story 2.1: Initialize Platform BE Project

As a developer,
I want a fully configured Spring Boot project with correct package structure and toolchain,
So that the team can build features on a compliant, standards-enforced backend foundation.

**Acceptance Criteria:**

**Given** the project is generated via Spring Initializr (`start.spring.io`),
**Then** it uses Java 21 LTS, Spring Boot 3.4.x, Maven, with dependencies: `spring-boot-starter-web`, `spring-boot-starter-data-jpa`, `spring-boot-starter-security`, `spring-boot-starter-test`, `postgresql`, `flyway-core`, `springdoc-openapi-starter-webmvc-ui:2.8.x`

**Given** the project is initialized,
**Then** the root package is `dev.chinh.portfolio` with four sub-packages created: `auth/`, `platform/`, `apps/`, `shared/` — no business logic exists outside these packages

**Given** `mvn test` is run,
**Then** all tests pass with JUnit 5, Mockito, and Testcontainers available on the test classpath

**Given** the project is configured,
**Then** `application.yml` defines `local` and `prod` profiles with all secrets externalized via environment variables (no hardcoded credentials)

**Given** structured logging is required (NFR-O4),
**Then** Logback is configured for JSON output in `prod` profile and human-readable in `local` — no PII fields logged

---

### Story 2.2: Database Schema & Core Entities

As a developer,
I want the four core data entities defined and migrated via Flyway,
So that all subsequent epics have a correct, versioned database schema to build on.

**Acceptance Criteria:**

**Given** the application starts with a PostgreSQL connection,
**Then** Flyway auto-runs migrations from `src/main/resources/db/migration/` on startup with naming `V{n}__{description}.sql`

**Given** migration `V1__create_core_schema.sql` runs,
**Then** four tables are created: `users`, `sessions`, `contact_submissions`, `project_health` — each with correct columns, types, and constraints

**Given** the `Session` entity exists,
**Then** it resides ONLY in the `auth.session` package — no other package references it directly

**Given** a user record is created,
**Then** `users` stores: id (UUID PK), email (unique), password_hash (nullable for OAuth users), provider (LOCAL/GOOGLE), role (OWNER/USER — default USER), created_at, updated_at

**Given** any query on user-owned entities (`sessions`, `contact_submissions`),
**Then** the query is scoped to the authenticated user ID at the repository layer — no cross-user access is structurally possible

#### Full Table Schemas (DDL-level)

**users**
```sql
CREATE TABLE users (
  id          UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email       VARCHAR(255) NOT NULL UNIQUE,
  password_hash VARCHAR(255),              -- NULL for OAuth-only users
  role        VARCHAR(20) NOT NULL DEFAULT 'USER',  -- 'USER' | 'OWNER'
  created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at  TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_users_email ON users(email);
```

**sessions**
```sql
CREATE TABLE sessions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id         UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  refresh_token   VARCHAR(512) NOT NULL UNIQUE,
  expires_at      TIMESTAMPTZ NOT NULL,
  revoked         BOOLEAN NOT NULL DEFAULT FALSE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_sessions_user_id ON sessions(user_id);
CREATE INDEX idx_sessions_refresh_token ON sessions(refresh_token);
```

**contact_submissions**
```sql
CREATE TABLE contact_submissions (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  email           VARCHAR(255) NOT NULL,
  message         TEXT NOT NULL,           -- min 10 chars, max 2000 chars
  referral_source VARCHAR(100),            -- value of ?from= param; NULL if not present
  ip_address      INET NOT NULL,           -- for rate limiting; never exposed in responses
  submitted_at    TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_contact_submitted_at ON contact_submissions(submitted_at DESC);
CREATE INDEX idx_contact_referral_source ON contact_submissions(referral_source);
```

**project_health**
```sql
CREATE TABLE project_health (
  id              UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  project_slug    VARCHAR(100) NOT NULL UNIQUE,  -- matches projects.ts config slug
  status          VARCHAR(20) NOT NULL DEFAULT 'UNKNOWN',  -- 'UP' | 'DOWN' | 'DEGRADED' | 'UNKNOWN'
  uptime_percent  DECIMAL(5,2),            -- e.g. 99.87; NULL if no data
  response_time_ms INTEGER,               -- last measured p95 ms; NULL if no data
  last_deploy_at  TIMESTAMPTZ,            -- from GitHub webhook; NULL if no webhook received
  last_polled_at  TIMESTAMPTZ,            -- timestamp of last poll attempt
  last_online_at  TIMESTAMPTZ,            -- last time status was UP; NULL if never
  consecutive_failures INTEGER NOT NULL DEFAULT 0,
  updated_at      TIMESTAMPTZ NOT NULL DEFAULT NOW()
);
CREATE INDEX idx_project_health_slug ON project_health(project_slug);
CREATE INDEX idx_project_health_last_polled ON project_health(last_polled_at DESC);
```

**Constraints summary:**
- All UUIDs generated server-side (PostgreSQL gen_random_uuid())
- Referential integrity enforced at DB level (CASCADE deletes)
- No at-rest encryption needed (per NFR-S10 for Wallet App; contact data is not banking-grade)
- ip_address stored as PostgreSQL INET type; never returned in any API response

---

### Story 2.3: Structured Error Handler & OpenAPI

As a developer consuming the Platform BE API,
I want all error responses in a consistent format and a live API specification,
So that frontend integration is predictable and the API is self-documenting.

**Acceptance Criteria:**

**Given** any `4xx` or `5xx` response from the API,
**Then** the response body is always `{"error": {"code": "SNAKE_CASE_CODE", "message": "Human readable"}}` — no raw stack traces or unstructured strings ever exposed

**Given** an unhandled exception occurs in any controller,
**Then** a `GlobalExceptionHandler` (`@RestControllerAdvice` in `shared.error`) catches it and returns a structured error with correct HTTP status

**Given** the application is running,
**When** a browser hits `/api-docs`,
**Then** Swagger UI renders with all `/api/v1/` endpoints documented including request/response schemas and auth requirements (FR51)

**Given** a request is made to a non-existent endpoint,
**Then** the response follows the structured error format with code `NOT_FOUND` and HTTP 404

---

### Story 2.4: Oracle A1 Infrastructure & CI/CD

> **Note: Story 2.4 was identified as oversized during implementation readiness review. It has been split into sub-stories 2.4.1, 2.4.2, 2.4.3 for better tracking. Original story kept for reference.**

As the owner,
I want Platform BE automatically deployed to Oracle A1 on every merge to main,
So that production is always up-to-date without manual steps.

**Acceptance Criteria:**

**Given** Oracle A1 is provisioned,
**Then** Nginx is configured as reverse proxy: routes `/api/*` to Spring Boot on port 8080, terminates SSL via Let's Encrypt (TLS 1.2+), and passes WebSocket upgrade headers transparently (NFR-I5)

**Given** a push lands on the `main` branch,
**When** GitHub Actions `deploy.yml` runs,
**Then** it builds the Maven artifact, runs all tests, and deploys the JAR to Oracle A1 via SSH — failing tests block deployment (NFR-O2)

**Given** the BE is deployed,
**When** `GET /actuator/health` is called,
**Then** it returns `{"status": "UP"}` with HTTP 200

**Given** the production environment is running,
**Then** PostgreSQL is co-located on Oracle A1, accessible only via localhost, and a daily cron job dumps the database to Oracle Object Storage (NFR-R6)

**Given** Nginx rate limiting is configured,
**Then** three rule sets exist: general 100 req/min/IP, contact 3 req/day/IP, and a commented-out AI endpoint block as Phase 2 placeholder (NFR-S9)

---

### Story 2.4.1: Nginx Reverse Proxy, SSL & WebSocket Support

**As the** Platform BE owner,
**I want** Nginx properly configured as a reverse proxy with SSL termination and WebSocket support,
**so that** the Spring Boot backend is securely accessible and WebSocket connections work end-to-end.

**Acceptance Criteria:**

Given Nginx is installed on Oracle A1,
When a request arrives at port 443,
Then Nginx terminates TLS (certificate via Let's Encrypt / certbot) and proxies to Spring Boot on port 8080.

Given a WebSocket upgrade request arrives,
When Nginx processes it,
Then Nginx passes `Upgrade` and `Connection` headers correctly — WebSocket connections succeed without application-layer changes.

Given HTTP requests arrive on port 80,
When processed,
Then Nginx redirects all HTTP → HTTPS (301).

Given the server is configured,
When Nginx config is validated (`nginx -t`),
Then no errors; `proxy_pass`, `proxy_http_version 1.1`, and `proxy_set_header Upgrade $http_upgrade` are set in location block.

**Config file location:** `/etc/nginx/sites-available/portfolio-v2`

---

### Story 2.4.2: GitHub Actions CI/CD Pipeline

**As the** developer,
**I want** automated CI/CD so every push to `main` deploys Platform BE to Oracle A1,
**so that** deployment is zero-manual-steps.

**Acceptance Criteria:**

Given a push to `main` branch,
When GitHub Actions `deploy.yml` workflow triggers,
Then it runs: (1) unit tests, (2) build JAR, (3) SCP JAR to Oracle A1, (4) SSH restart Spring Boot service.

Given the deploy workflow runs,
When all steps pass,
Then deployment completes within 5 minutes; app is reachable at the production URL.

Given a test step fails,
When the workflow runs,
Then deployment step is skipped; a GitHub Actions failure notification is sent.

**Workflow file:** `.github/workflows/deploy.yml`
**Secrets required in GitHub repo:** `ORACLE_HOST`, `ORACLE_USER`, `ORACLE_SSH_KEY`

---

### Story 2.4.3: Database Backup Automation

**As the** platform owner,
**I want** daily automated PostgreSQL backups stored to Oracle Object Storage,
**so that** data is recoverable within 24h (RPO) in case of failure.

**Acceptance Criteria:**

Given a cron job runs at 02:00 UTC daily,
When it executes,
Then it runs `pg_dump portfolio_v2 | gzip > backup_$(date +%Y%m%d).sql.gz` and uploads to Oracle Object Storage bucket `portfolio-backups`.

Given a backup succeeds,
When uploaded to Object Storage,
Then the previous backup file older than 7 days is deleted (retain last 7 daily backups).

Given a backup fails,
When the cron job encounters an error,
Then it sends an alert email to the owner (configured via `ALERT_EMAIL` env var).

**Cron entry:** `0 2 * * * /opt/scripts/backup.sh >> /var/log/backup.log 2>&1`

---

## Epic 3: Live Evidence Layer

Recruiter sees real-time health metrics on project cards — proving demo apps are actually running in production — with graceful degradation when data is offline.

### Story 3.1: Health Metrics Polling Pipeline (BE)

As the owner,
I want Platform BE to continuously poll registered demo app health endpoints,
So that real-time uptime data is always available for display.

**Acceptance Criteria:**

**Given** a demo app is registered in the Platform BE config file,
**Then** the `@Scheduled` polling job in `platform.metrics` calls that app's `/health` endpoint every 60 seconds

**Given** a poll succeeds,
**Then** a `ProjectHealth` record is upserted with: app_id, uptime_percent, response_time_ms, last_deploy, checked_at timestamp

**Given** a poll fails (timeout or connection error),
**Then** the failure is logged at WARN level, the previous `ProjectHealth` record is preserved with its timestamp, and no exception propagates to crash the scheduler

**Given** the config file lists multiple demo apps,
**Then** all are polled independently — one app failing does not block others

**Given** a duplicate GitHub webhook delivery arrives for the same event,
**Then** processing is idempotent — no duplicate `ProjectHealth` record is created (NFR-I4)

---

### Story 3.2: GitHub Webhook → Metrics Refresh (BE)

As the owner,
I want a GitHub push event to immediately trigger a health metrics refresh,
So that after a deploy the portfolio shows updated data within seconds.

**Acceptance Criteria:**

**Given** `POST /api/v1/webhooks/github` is called with a valid GitHub push event,
**Then** the platform immediately re-polls the affected project's `/health` endpoint outside the normal 60s cycle

**Given** a webhook arrives,
**Then** the HMAC-SHA256 signature is verified against the GitHub secret before any processing — unverified payloads return HTTP 401 and are not processed (FR48)

**Given** a webhook arrives for a project not in the config registry,
**Then** it is silently ignored with HTTP 200 — no error is exposed to GitHub

**Given** two webhook deliveries arrive within 5 seconds for the same project,
**Then** only one re-poll is triggered (debounce — NFR-I4)

---

### Story 3.3: WebSocket Server — Metrics Broadcast (BE)

As the Portfolio FE,
I want to receive live health metric updates via WebSocket,
So that project cards update in real time without browser-side polling.

**Acceptance Criteria:**

**Given** a WebSocket client connects to `ws://{host}/ws/metrics`,
**Then** `MetricsWebSocketHandler` in `platform.websocket` accepts the connection and adds it to the active sessions list

**Given** a new `ProjectHealth` snapshot is produced (by scheduler or webhook trigger),
**Then** the updated metrics are broadcast to ALL connected WebSocket sessions within 3 seconds of detection (NFR-P5)

**Given** a WebSocket client disconnects,
**Then** the session is removed from the broadcast list — no further messages are sent to it

**Given** the WebSocket implementation,
**Then** it uses native WebSocket only — NOT STOMP, NOT SockJS — no `sockjs-client` or `@stomp/stompjs` dependencies anywhere

---

### Story 3.4: WebSocket Client & Metrics Cards (FE)

As a recruiter,
I want to see live health metrics on project cards update in real time,
So that I can verify demo apps are genuinely running during my visit.

**Acceptance Criteria:**

**Given** the Portfolio FE loads,
**Then** a WebSocket connection is established to Platform BE `ws://{host}/ws/metrics`

**Given** a metrics update message arrives via WebSocket,
**Then** the affected project card's uptime %, response time ms, and last deploy update without a page refresh

**Given** the WebSocket connection drops,
**Then** automatic reconnection is attempted with exponential backoff — max 3 retries — after which the FE displays last known data (NFR-R3)

**Given** Platform BE is completely offline,
**Then** the portfolio FE renders all static content correctly — only the live metrics area shows a degraded state, no layout break (NFR-R4)

**Given** metrics data is temporarily unavailable,
**Then** each affected card shows "Updated X ago" timestamp instead of live values — card layout does not shift

Given the WebSocket connection is established but then disconnects before any data is received,
When the connection drops,
Then project cards show "Status unavailable" (not "Updated X ago") — the staleness timer only starts after first data receipt.

---

### Story 3.5: Metrics Staleness & Auto-Hide (FE)

As a recruiter,
I want the portfolio to never show misleading or stale metrics,
So that I can trust that data I see is genuinely current.

**Acceptance Criteria:**

**Given** a project's health data has not been updated for more than 24 hours,
**Then** the metrics section on that card is hidden entirely — no stale numbers are displayed (FR11)

**Given** metrics data is temporarily unavailable (< 24h since last update),
**Then** "Updated X ago" is shown with elapsed time — e.g. "Updated 2h ago" (FR10)

**Given** metrics data becomes available again after being hidden,
**Then** the metrics section re-appears automatically when a live update arrives via WebSocket

---

### Story 3.6: Live GitHub Contribution Graph (FE + BE)

As a recruiter,
I want to see Chính's live GitHub contribution graph,
So that I can verify recent activity is genuine and not a screenshot.

**Acceptance Criteria:**

**Given** the portfolio loads,
**Then** the GitHub contribution graph is fetched from Platform BE which proxies the GitHub API — not a screenshot or static image (FR12)

**Given** the GitHub API response is received,
**Then** it is cached using Spring `@Cacheable` + Caffeine with `maximumSize=100, expireAfterWrite=3600s` — no other caching mechanism is used (NFR-I1)

**Given** the GitHub API is unavailable but the in-session cache has data,
**Then** the cached graph is displayed without error

**Given** the GitHub API is unavailable AND the cache is empty (e.g. after BE restart),
**Then** the contribution graph section is hidden entirely — no broken image or error message shown (NFR-I1)

**Given** the GitHub API proxy call exceeds 2 seconds,
**Then** an `AbortController` timeout fires, the section is hidden gracefully, and no spinner freeze occurs

---

### Story 3.7: Config-Only Demo App Registration (BE)

**Config file dependency:** The `showcase.yml` config file format is defined in Story 2.4.1 (Platform BE Infrastructure). Story 3.7 EXTENDS that config — do not implement Story 3.7 before Story 2.4.1 is complete and the config schema is established.

As the owner,
I want to register a new demo app by editing only a config file,
So that onboarding a new app requires zero application code changes.

**Acceptance Criteria:**

**Given** a new demo app entry is added to the Platform BE config file (app_id, health endpoint URL, display name),
**Then** after a git push and redeploy, the scheduler automatically begins polling that app's `/health` endpoint — no code changes required (FR33)

**Given** a demo app entry is removed from the config,
**Then** after redeploy, polling stops for that app and its metrics are no longer broadcast

**Given** the config file is malformed (invalid YAML),
**Then** the application fails to start with a clear error message identifying the config problem

---

## Epic 4: Recruiter Contact & Engagement

Recruiter contacts Chính via frictionless form; portfolio intelligently adapts to referral context and returning visit behavior.

### Story 4.1: Contact Form & Backend Submission (FE + BE)

As a recruiter,
I want to submit a contact inquiry via a simple form,
So that I can reach Chính with minimal friction.

**Acceptance Criteria:**

**Given** the contact section is rendered,
**Then** the form has exactly two required fields: email and message — no CAPTCHA, no extra fields

**Given** the recruiter submits the form with valid data,
**Then** `POST /api/v1/contact` stores a `ContactSubmission` record with email, message, referral_source (`?from=` value), and submitted_at timestamp — returns HTTP 201

**Given** the form is submitted,
**Then** feedback appears in under 1 second (perceived) — the recruiter is not left waiting (NFR-P7)

**Given** the form is submitted with an invalid email,
**Then** inline validation shows an error without page reload — form stays in place, no scroll jump, recruiter can retry immediately

**Given** `POST /api/v1/contact` is called,
**Then** the endpoint is publicly accessible (no auth required)

**Form validation rules:**
- Email: required, valid format (RFC 5322 basic: `user@domain.tld`), max 255 chars
- Message: required, min 10 characters, max 2000 characters
- Honeypot field: hidden `<input name="website">` — if non-empty, silently discard (return 200 OK but don't save)
- Validation triggered on field blur (not on every keystroke)
- Error messages shown inline below field, `role="alert"` for screen readers

---

### Story 4.2: Animated Success Confirmation (FE)

As a recruiter,
I want to see a satisfying animated confirmation after submitting the contact form,
So that I know my message was received and the interaction feels polished.

**Acceptance Criteria:**

**Given** the contact form submits successfully,
**Then** a spring-bouncy animation (stiffness 200, damping 15) plays — confetti burst or checkmark animation — completing within ≤ 1.5 seconds (FR15)

**Given** `prefers-reduced-motion` is enabled,
**Then** the animation is skipped entirely — a static success message is shown instead

**Given** the success state is shown,
**Then** a confirmation toast auto-dismisses after 2 seconds

**Given** the success animation plays,
**Then** it reads from the `MOTION_ENABLED` context — no animation bypass allowed

---

### Story 4.3: Rate Limiting & Spam Protection (BE)

As the owner,
I want contact form submissions rate-limited per IP,
So that spam does not flood my inbox or exhaust storage.

**Acceptance Criteria:**

**Given** a single IP submits the contact form more than 3 times in a 24-hour window,
**Then** subsequent submissions return HTTP 429 with structured error `{"error": {"code": "RATE_LIMIT_EXCEEDED", "message": "..."}}`

**Given** a legitimate user hits the rate limit,
**Then** the FE contact form shows a user-friendly message ("Too many submissions, please try again tomorrow") — no internal error code exposed

**Given** rate limiting is enforced,
**Then** it uses the Nginx contact rule (configured in Story 2.4) — no duplicate application-layer implementation

**Nginx rate limit config (to be set up in Story 2.4.1):**
```nginx
# Contact form rate limiting — 3 requests per 24h per IP
limit_req_zone $binary_remote_addr zone=contact_form:10m rate=3r/d;

# In server block:
location /api/v1/contact {
    limit_req zone=contact_form burst=1 nodelay;
    limit_req_status 429;
    proxy_pass http://spring_boot_backend;
}

# General API rate limiting — 100 req/min per IP
limit_req_zone $binary_remote_addr zone=api_general:10m rate=100r/m;

# Phase 2 placeholder (commented out):
# limit_req_zone $binary_remote_addr zone=ai_endpoints:10m rate=5r/m;
```
Story 4.3 implements the Bucket4j in-app rate limiting as a second layer; Nginx is the first layer defined in Story 2.4.1.

---

### Story 4.4: Referral Tracking & Contextual Messaging (FE)

As a recruiter arriving from a specific source,
I want the portfolio to display messaging appropriate to my context,
So that the content feels relevant to where I came from.

**Acceptance Criteria:**

**Given** the page loads with `?from=linkedin`,
**Then** the portfolio defaults to English and surfaces professional English-first framing in the hero (FR46)

**Given** the page loads with `?from=cv-vn`,
**Then** the portfolio defaults to Vietnamese and surfaces Vietnamese-friendly context (FR46)

**Given** any `?from=` value is present,
**Then** it is persisted to Zustand store via `zustand/middleware` persist — NOT manual `localStorage` calls

**Given** `?from=` is absent on a return visit but was previously persisted,
**Then** the persisted referral source is used for contextual messaging

**Given** localStorage access fails (private browsing),
**Then** the app falls back to default behavior gracefully — no uncaught exception, no broken layout

---

### Story 4.5: Returning Visitor Experience (FE)

As a returning recruiter,
I want the portfolio to recognize and adapt to my return visit,
So that I can pick up where I left off without re-navigating from scratch.

**Acceptance Criteria:**

Given a recruiter visits the portfolio for the first time (no localStorage entry `pf_return_visitor`),
When the portfolio loads,
Then no welcome-back banner is shown; hero section renders normally with full intro animation.

Given a recruiter has visited before (localStorage key `pf_return_visitor` exists with `lastViewedSlug` and `lastVisitTimestamp`),
When they return to the portfolio,
Then a slim inline banner appears **immediately below the hero section** with:
  - Text: "Welcome back — continue where you left off"
  - A clickable link to the last-viewed project: `→ [Project Name]` (slug from `pf_return_visitor.lastViewedSlug`)
  - Dismiss button (×) that sets `pf_return_visitor.bannerDismissed = true`
  - Banner does NOT reappear after dismissed (within the same browser session)
  - Banner height: max 48px; uses `bg-surface-elevated` token + `border-b border-border` styling
  - Banner animates in with `spring-gentle` (150ms); slides down from hero bottom edge

Given a recruiter has visited before,
When the portfolio loads,
Then the hero section **skips the entrance animation** (Framer Motion `initial` state set to final state; `AnimatePresence` key unchanged)
And the gallery defaults to showing the last-viewed project's tab (Featured or Technical, from `pf_return_visitor.lastViewedTab`).

Given `prefers-reduced-motion` is active,
When a returning visitor loads the portfolio,
Then the banner appears instantly (no slide animation) and hero skip is preserved.

**localStorage schema for return visitor state:**
```json
{
  "pf_return_visitor": {
    "lastViewedSlug": "wallet-app",
    "lastViewedTab": "featured",
    "lastVisitTimestamp": 1741008000000,
    "bannerDismissed": false
  }
}
```

localStorage is set/updated on every page navigation and project card click.

---

## Epic 5: Platform Auth & Demo App Identity

Demo app users authenticate securely (Google OAuth2), sessions are managed, users access only their own data — enabling Wallet App launch.

### Story 5.1: JWT Infrastructure & Spring Security Config (BE)

As a developer,
I want JWT-based authentication infrastructure set up with correct Spring Security configuration,
So that all subsequent auth flows share a single, consistent security foundation.

**Acceptance Criteria:**

**Given** the Spring Security config is in place,
**Then** all `/api/v1/` endpoints require a valid JWT Bearer token by default — except explicitly permitted public paths (`/api/v1/contact`, `/api/v1/webhooks/**`, `/api/v1/auth/**`, `/api-docs/**`, `/actuator/health`)

**Given** a valid JWT is issued,
**Then** it uses RS256 (asymmetric signing) with access token TTL of 15 minutes and refresh token TTL of 7 days (NFR-S2, NFR-S3)

**Given** an expired or invalid JWT is sent,
**Then** the API returns HTTP 401 with structured error `{"error": {"code": "UNAUTHORIZED", "message": "..."}}`

**Given** CORS is configured,
**Then** only the Vercel production domain and localhost are in the allow-list — no wildcard (NFR-S8)

**Given** the JWT infrastructure is complete,
**Then** unit test coverage for the `auth.jwt` package is ≥ 80% (NFR-M1)

---

### Story 5.2: Email/Password Registration & Login (BE)

As a demo app user,
I want to register and log in with email and password,
So that I can access my personal data in demo apps that support this flow.

**Acceptance Criteria:**

**Given** `POST /api/v1/auth/register` is called with email and password,
**Then** a new user record is created with `provider = LOCAL` and the password stored as bcrypt hash with cost factor ≥ 12 (NFR-S4)

**Given** `POST /api/v1/auth/login` is called with valid credentials,
**Then** the response contains an access token (JWT, 15m TTL) and a refresh token (opaque, 7d TTL stored in `sessions` table)

**Given** `POST /api/v1/auth/login` is called with invalid credentials,
**Then** HTTP 401 is returned — no information about whether the email exists is leaked

**Given** a user registers with an already-used email,
**Then** HTTP 409 is returned with structured error `{"error": {"code": "EMAIL_ALREADY_EXISTS", "message": "..."}}`

---

### Story 5.3: Google OAuth2 Login (BE)

As a demo app user,
I want to log in with my Google account,
So that I can access the Wallet App without creating a separate password.

**Acceptance Criteria:**

**Given** the OAuth2 flow is initiated,
**Then** it uses the standard OAuth2 authorization code flow — no custom token exchange (NFR-I2)

**Given** a user completes Google OAuth2 successfully,
**Then** a user record is created or retrieved with `provider = GOOGLE`, and an access token + refresh token pair is returned in the same format as password login

**Given** the same Google account logs in a second time,
**Then** no duplicate user record is created — the existing user is matched by email

**Given** Google OAuth2 fails (user cancels or Google returns error),
**Then** the user is redirected to the login page with a user-friendly error — no internal OAuth details exposed

**Google OAuth2 provider configuration required:**
- Create Google Cloud project → OAuth 2.0 credentials (Web application type)
- Authorized redirect URIs: `https://api.portfolio-v2.com/api/v1/auth/oauth2/callback/google` (production) + `http://localhost:8080/api/v1/auth/oauth2/callback/google` (dev)
- Store `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET` as environment variables / GitHub Secrets
- Spring Boot config: `spring.security.oauth2.client.registration.google.*`

---

### Story 5.4: Session Refresh & Logout (BE)

As a demo app user,
I want my session to refresh automatically and be fully invalidated on logout,
So that I stay logged in during active use and my session is clean when I leave.

**Acceptance Criteria:**

**Given** `POST /api/v1/auth/refresh` is called with a valid refresh token,
**Then** a new access token (15m TTL) is returned and the refresh token is rotated — old one invalidated, new one issued (FR26)

**Given** `POST /api/v1/auth/refresh` is called with an expired or invalid refresh token,
**Then** HTTP 401 is returned — user must re-authenticate

**Given** `POST /api/v1/auth/logout` is called with a valid access token,
**Then** the associated refresh token is deleted from the `sessions` table — the session is fully invalidated (FR27)

**Given** the same refresh token is used after logout,
**Then** HTTP 401 is returned — replay attack is not possible

---

### Story 5.5: User Data Isolation & JWKS Endpoint (BE)

As a demo app user,
I want my data completely isolated from other users,
And as a demo app developer, I want to validate JWTs independently without calling Platform BE on every request.

**Acceptance Criteria:**

**Given** an authenticated user calls any user-scoped endpoint,
**Then** the response contains only data belonging to that user — cross-user access returns HTTP 403 (FR28, NFR-S5)

**Given** `GET /api/v1/.well-known/jwks.json` is called,
**Then** the public key set is returned in JWKS format — publicly accessible, no auth required (FR29)

**Given** a demo app uses the JWKS endpoint to validate a JWT,
**Then** the JWT signature can be verified against the published public key with no call back to Platform BE

**Given** the auth package is complete,
**Then** overall `auth.*` unit test coverage is ≥ 80% — including boundary tests that assert HTTP 403 on cross-user access attempts (NFR-M1)

---

## Epic 6: Owner Admin & Analytics

Owner manages portfolio content, reviews contact inquiries, tracks engagement — no code changes needed, only config file + git push.

### Story 6.1: Admin Contact Inquiries Endpoint (BE)

As the owner,
I want to view all submitted contact inquiries via a private API endpoint,
So that I can review recruiter messages without needing a database client.

**Acceptance Criteria:**

**Given** `GET /api/v1/admin/contact` is called with a valid Owner JWT,
**Then** the response lists all `ContactSubmission` records sorted by submitted_at descending — including email, message, referral_source, and submitted_at

**Given** the endpoint is called without auth or with a non-Owner JWT,
**Then** HTTP 403 is returned with structured error — no submissions data is exposed (NFR-S7)

**Given** there are no submissions,
**Then** the endpoint returns an empty array with HTTP 200 — not a 404

**Given** an Owner role is required,
**Then** only the account with `role = OWNER` can access `/api/v1/admin/**` — no other role bypasses this

---

### Story 6.2: Admin Analytics Endpoint (BE)

As the owner,
I want to view portfolio engagement data (contact form counts, referral source breakdown) via a private endpoint,
So that I can understand which channels are driving recruiter contact.

**Acceptance Criteria:**

**Given** `GET /api/v1/admin/analytics` is called with a valid Owner JWT,
**Then** the response includes: total contact submission count, breakdown of submissions by referral_source, and date range of submissions

**Given** the endpoint is called without Owner auth,
**Then** HTTP 403 is returned — no analytics data exposed (NFR-S7)

**Given** no submissions exist,
**Then** the response returns zero counts with correct structure — no error

**Analytics endpoint response schema — GET /api/v1/admin/analytics:**

```json
{
  "summary": {
    "totalSubmissions": 42,
    "last30DaysCount": 17,
    "last7DaysCount": 5
  },
  "byReferralSource": [
    { "source": "linkedin",  "count": 18, "percentage": 42.9 },
    { "source": "cv-vn",     "count": 12, "percentage": 28.6 },
    { "source": "direct",    "count": 8,  "percentage": 19.0 },
    { "source": null,        "count": 4,  "percentage": 9.5  }
  ],
  "recentSubmissions": [
    {
      "id": "uuid",
      "email": "recruiter@company.com",
      "submittedAt": "2026-03-03T14:00:00Z",
      "referralSource": "linkedin"
    }
  ],
  "dateRange": {
    "earliest": "2026-01-15T08:23:00Z",
    "latest": "2026-03-03T14:00:00Z"
  }
}
```

Notes:
- `byReferralSource` sorted by count descending; null source = direct/unknown
- `recentSubmissions` limited to last 20 entries; message body NOT returned (privacy)
- `percentage` rounded to 1 decimal place
- All timestamps in ISO 8601 UTC format

---

### Story 6.3: Plausible Analytics Embed (FE)

As the owner,
I want page-level analytics (pageviews, bounce rate, traffic sources) captured automatically,
So that I can view them in the Plausible dashboard without any backend implementation.

**Acceptance Criteria:**

**Given** the Portfolio FE is deployed,
**Then** the Plausible tracking script is embedded in `index.html` with `defer` attribute — loaded from Plausible CDN, not bundled (FR38a)

**Given** a recruiter visits any page,
**Then** the pageview is tracked — verifiable in the Plausible dashboard

**Given** the Plausible script is external,
**Then** it does not count toward the 300KB bundle limit (NFR-P2) and does not block page render

**Given** no noscript fallback is needed,
**Then** no `<noscript>` tag is added — Plausible functions without it, keep implementation minimal

---

### Story 6.4: Config-File Content Management & Smoke Tests (FE + BE)

> **Note: Story 6.4 was identified as oversized during implementation readiness review. Split into sub-stories 6.4.1, 6.4.2, 6.4.3 for better tracking.**

As the owner,
I want to update portfolio content and verify the system works end-to-end after every deploy,
So that content stays current and integration regressions are caught automatically without manual testing.

**Acceptance Criteria:**

**Config Management — FE (separate repo, separate deploy):**

**Given** `src/config/projects.ts` exists in the Portfolio FE repo,
**When** the owner edits project metadata (name, description, artist statement, build timeline, tech stack, availability status) and pushes to main,
**Then** Vercel auto-deploys the FE only — no BE changes, no code changes required (FR35, FR36)

**Config Management — BE (separate repo, separate deploy):**

**Given** `src/main/resources/config/showcase.yml` exists in the Platform BE repo,
**When** the owner edits demo app registry entries and pushes to main,
**Then** GitHub Actions auto-deploys the BE only — no FE changes, no code changes required (FR33, FR35)

**Smoke Tests — automated in GitHub Actions CI:**

**Given** every BE deploy completes via GitHub Actions,
**Then** the following smoke tests run automatically as a post-deploy step (NFR-R7):
1. Submit contact form → `GET /api/v1/admin/contact` returns the new entry
2. WebSocket connection establishes → a metrics broadcast is received within 3s → the FE project card updates

**Given** any smoke test fails,
**Then** the GitHub Actions workflow step is marked as failed and the owner is notified — the deploy is flagged as requiring investigation

---

### Story 6.4.1: FE Config Management (projects.ts)

**As the** portfolio owner,
**I want** a typed config file to manage all project showcase data,
**so that** I can update projects, metadata, and artist statements via a git push with no code changes.

**Acceptance Criteria:**

Given `src/config/projects.ts` exists with TypeScript interfaces,
When I add a new project entry to the config,
Then the portfolio gallery automatically includes it on next deploy with no component code changes.

Config schema includes: `slug`, `name`, `description`, `techStack[]`, `demoUrl`, `repoUrl`, `artistStatement`, `timeline.milestones[]`, `hasBuildStory`, `availability` (for About section).

Given `src/config/about.ts` exists,
When I update `availability.status` or `availability.location`,
Then the job availability badge on the portfolio reflects the change on next deploy.

---

### Story 6.4.2: BE Config Management (showcase.yml)

**As the** platform owner,
**I want** a YAML config file to register demo apps for health polling,
**so that** I can add new demo apps by editing config and redeploying with no code changes.

**Acceptance Criteria:**

Given `src/main/resources/showcase.yml` exists,
When I add a new entry with `slug`, `name`, `healthEndpoint`, `demoUrl`,
Then Platform BE automatically starts polling that app's health endpoint on next restart.

Given the config file is malformed (invalid YAML),
When Platform BE starts,
Then it logs a clear error with the specific parse failure and refuses to start (fail-fast).

Given the config file has a valid entry but the health endpoint is unreachable,
When Platform BE polls it,
Then status is recorded as `DOWN` in `project_health` table; no exception is thrown; polling continues for other apps.

---

### Story 6.4.3: Automated Smoke Tests (CI Gate)

**As the** development team,
**I want** automated smoke tests that run on every deploy,
**so that** critical integration paths are verified before production traffic hits.

**Acceptance Criteria:**

Given the deploy pipeline completes,
When smoke tests run,
Then Test 1 — Contact Form → Admin: POST `/api/v1/contact` with valid payload → GET `/api/v1/admin/analytics` → verify submission count incremented.

Given the deploy pipeline completes,
When smoke tests run,
Then Test 2 — WebSocket Metrics: Open WS connection to Platform BE → wait ≤ 5s → verify at least one `project_health` message received with valid `status` field.

Given either smoke test fails,
When the CI pipeline checks,
Then the deploy is flagged as failed; alert sent; previous version remains live (no automatic rollback needed — manual intervention required).

**Test implementation:** Jest + Supertest for REST; `ws` npm package for WebSocket test. Run as part of GitHub Actions `smoke.yml` workflow triggered on successful `deploy.yml` completion.

---
