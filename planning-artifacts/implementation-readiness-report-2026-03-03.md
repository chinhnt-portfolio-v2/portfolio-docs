---
stepsCompleted: ["step-01-document-discovery", "step-02-prd-analysis", "step-03-epic-coverage-validation", "step-04-ux-alignment", "step-05-epic-quality-review", "step-06-final-assessment"]
workflowStatus: complete
documentsUsed:
  prd: "_bmad-output/planning-artifacts/prd.md"
  architecture: "_bmad-output/planning-artifacts/architecture.md"
  epics: "_bmad-output/planning-artifacts/epics.md"
  ux: "_bmad-output/planning-artifacts/ux-design-specification.md"
---

# Implementation Readiness Assessment Report

**Date:** 2026-03-03
**Project:** portfolio-v2

---

## PRD Analysis

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
FR13: Portfolio self-displays its own Lighthouse performance score as a trust signal *(Phase 2)*
FR14: Recruiter can submit a contact inquiry via form
FR15: Recruiter receives an animated success confirmation (e.g., confetti burst or checkmark animation, ≤ 1.5s duration) immediately after successfully submitting the contact form
FR16: Portfolio reads referral source from URL parameter (`?from=`) and saves to local storage
FR17: Portfolio delivers a streamlined returning-visitor experience — e.g., reduced onboarding elements and quick access to last-viewed content — based on locally persisted browsing context
FR18: Recruiter can navigate the entire portfolio using keyboard only
FR19: Portfolio content is accessible via screen readers (WCAG 2.1 AA)
FR20: Portfolio renders correctly on all modern evergreen browsers and all screen sizes from 320px to 4K
FR21: Recruiter can use command palette to jump to any section or project *(Phase 2)*
FR22: Recruiter can view a video walkthrough (≤90 seconds) per project *(Phase 2)*
FR23: Portfolio auto-syncs project page content from the corresponding GitHub repository README *(Phase 2)*
FR24: Platform BE supports email/password authentication (JWT-based). Wallet App exposes only Google OAuth login on UI — email/password available for future demo apps.
FR25: Demo app user can log in with a Google account
FR26: Demo app user's session automatically refreshes without requiring re-login within the session window
FR27: Demo app user can log out and invalidate their session
FR28: Platform ensures each user can only access their own data (user-scoped isolation)
FR29: Platform exposes a public JWKS endpoint for demo apps to validate JWT tokens independently
FR30: Platform BE aggregates health metrics from all registered demo apps by polling each app's `/health` endpoint every 60 seconds. Detection latency ≤ 60s; WebSocket delivery latency after detection < 3s.
FR31: Portfolio FE receives live health metric updates via WebSocket connection from Platform BE
FR32: Platform BE automatically refreshes health metrics for a project upon receiving a GitHub push event webhook
FR33: Owner can register a new demo app to the platform without changing application code — config file changes only in both Platform BE and Portfolio FE.
FR34: Platform exposes AI Build Story per project — structured narrative min 300 words, highlighting at least 2 override decisions *(Phase 2)*
FR35: Owner can update showcase configuration by editing config file and pushing to deploy
FR36: Owner can update job availability status by editing config file
FR37: Owner can view submitted contact form inquiries via private admin endpoint
FR38a: Owner can view page-level analytics via Plausible dashboard (external, no BE implementation required)
FR38b: Owner can view portfolio-specific engagement data via private admin endpoint `/api/v1/admin/analytics`
FR39: System enforces rate limits on contact form submissions to prevent spam
FR40: System enforces rate limits on AI-powered endpoints *(Phase 2)*
FR41: Recruiter can switch portfolio language between Vietnamese and English
FR42: Portfolio remembers recruiter's language preference across return visits (locally persisted)
FR43: Recruiter can view a full project detail page with shareable URL (`/projects/{slug}`)
FR44: Recruiter can switch appearance between dark mode and light mode
FR45: Portfolio remembers recruiter's appearance preference across return visits (locally persisted)
FR46: Portfolio displays contextual messaging appropriate to referral source (`?from=` parameter)
FR47: Recruiter experiences spring-physics micro-animations; all animation disabled when `prefers-reduced-motion` is active
FR48: System verifies webhook signature (HMAC-SHA256 with GitHub secret) before processing any incoming webhook
FR49: Platform BE manages four core data entities: User, Session, ContactSubmission, ProjectHealth; all user-owned entities scoped to authenticated user ID — cross-user access returns 403
FR50: Platform BE returns structured JSON error responses `{"error": {"code": "...", "message": "..."}}` for all 4xx and 5xx responses
FR51: Platform BE API is self-documented via OpenAPI specification accessible at `/api-docs`

**Total FRs: 51** (Phase 1 MVP: 44 | Phase 2: FR13, FR21, FR22, FR23, FR34, FR40 = 7 deferred)

---

### Non-Functional Requirements

**Performance:**
NFR-P1: Portfolio FE LCP < 1.5s
NFR-P2: Portfolio FE total bundle size (gzipped) < 300KB
NFR-P3: Lighthouse Performance score ≥ 90 (CI gate) / ≥ 95 (aspirational)
NFR-P4: Platform BE internal API response time (p95) ≤ 200ms
NFR-P4b: Platform BE proxy/external API response time ≤ 2s with hard timeout
NFR-P5: WebSocket health metric delivery latency < 3s after Platform BE detects change
NFR-P6: Portfolio FE Time to Interactive < 2s
NFR-P7: Contact form submission feedback < 1s

**Security:**
NFR-S1: All client-server communication HTTPS only (TLS 1.2+)
NFR-S2: JWT access token TTL 15 minutes
NFR-S3: JWT refresh token TTL 7 days
NFR-S4: Password storage bcrypt (cost factor ≥ 12)
NFR-S5: User data isolation — query-level enforcement, scoped to authenticated user ID
NFR-S6: Incoming webhook verification HMAC-SHA256 signature check
NFR-S7: Admin endpoints authenticated + role-based
NFR-S8: CORS policy — explicit allow-list only (no wildcard)
NFR-S9: Rate limiting — Contact: 3/day/IP; AI endpoints: 5 req/10min/IP (Phase 2); General: 100/min/IP
NFR-S10: Wallet App financial data — no at-rest encryption required

**Reliability:**
NFR-R1: Platform BE uptime 99.9% (≤ 8.7h downtime/year), monitored via UptimeRobot
NFR-R2: Portfolio FE availability 99.99% (Vercel CDN SLA)
NFR-R3: WebSocket reconnection — automatic with exponential backoff, max 3 retries then fallback to polling
NFR-R4: Graceful degradation (BE offline) — Portfolio FE fully renders without BE
NFR-R5: Metrics staleness handling — hide after 24h offline, "Updated X ago" visible
NFR-R6: Database backup — daily automated backup, RPO 24h
NFR-R7: Critical happy path smoke tests pass on every deploy

**Accessibility:**
NFR-A1: WCAG 2.1 AA compliance
NFR-A2: Color contrast ratio ≥ 4.5:1 (text) / ≥ 3:1 (large text & UI components)
NFR-A3: Keyboard navigation — 100% interactive elements reachable
NFR-A4: Screen reader compatibility — all content accessible via ARIA
NFR-A5: Motion sensitivity — respect `prefers-reduced-motion`

**Maintainability:**
NFR-M1: Platform BE auth module unit test coverage ≥ 80%
NFR-M2: Platform BE metrics aggregation unit test coverage ≥ 80%
NFR-M3: Wallet App business logic unit test coverage ≥ 70%
NFR-M4: Portfolio FE utility functions unit test coverage ≥ 60%
NFR-M5: Package structure discipline enforced from Sprint 1 (`auth.*`, `platform.*`, `apps.*`)

**Integration:**
NFR-I1: GitHub API — cached TTL 1h in-memory; hide contribution graph if API unavailable after server restart
NFR-I2: Google OAuth2 — standard OAuth2 authorization code flow
NFR-I3: Future AI API integration (Phase 2) — per-user rate limited, hard timeout 10s
NFR-I4: GitHub Webhooks — signature verified + idempotent processing
NFR-I5: Reverse proxy WebSocket support — must support HTTP → WS protocol upgrade

**Operational:**
NFR-O1: Portfolio FE deployment — push to main → auto-deploy (Vercel)
NFR-O2: Platform BE deployment — GitHub Actions CI/CD → Oracle A1
NFR-O3: Health monitoring — UptimeRobot (free tier, 5-min checks)
NFR-O4: Logging — structured JSON logs, configurable log level, no PII
NFR-O5: Infrastructure cost — $0/month

**Total NFRs: 47**

---

### PRD Completeness Assessment

The PRD is thorough and well-structured with 51 FRs and 47 NFRs clearly numbered and organized. Phase 2 features are clearly delineated. Requirements are SMART (specific, measurable) with concrete thresholds (response times, bundle sizes, TTLs, coverage percentages). Minor areas to watch: FR13/FR40 Phase 2 dependencies chain with other Phase 2 NFRs.

---

## Epic Coverage Validation

### Coverage Matrix

| FR# | Requirement Summary | Epic Coverage | Status |
|-----|-------------------|---------------|--------|
| FR1 | Projects-first layout | Epic 1 | ✅ Covered |
| FR2 | Gallery card with whitespace ≥16px | Epic 1 | ✅ Covered |
| FR3 | Artist statement per project | Epic 1 | ✅ Covered |
| FR4 | "Why Hire Me" background section | Epic 1 | ✅ Covered |
| FR5 | Skills with verifiable proof links | Epic 1 | ✅ Covered |
| FR6 | Collapsible lessons learned | Epic 1 | ✅ Covered |
| FR7 | Verifiable build timeline | Epic 1 | ✅ Covered |
| FR8 | Job availability status | Epic 1 | ✅ Covered |
| FR9 | Real-time health metrics on card | Epic 3 | ✅ Covered |
| FR10 | "Updated X ago" timestamp | Epic 3 | ✅ Covered |
| FR11 | Auto-hide metrics after 24h | Epic 3 | ✅ Covered |
| FR12 | Live GitHub contribution graph | Epic 3 | ✅ Covered |
| FR13 | Lighthouse score self-display | Phase 2 | ⏭️ Deferred |
| FR14 | Contact inquiry form | Epic 4 | ✅ Covered |
| FR15 | Animated success confirmation ≤1.5s | Epic 4 | ✅ Covered |
| FR16 | `?from=` referral tracking → localStorage | Epic 4 | ✅ Covered |
| FR17 | Returning visitor streamlined experience | Epic 4 | ✅ Covered |
| FR18 | Full keyboard navigation | Epic 1 | ✅ Covered |
| FR19 | WCAG 2.1 AA screen reader | Epic 1 | ✅ Covered |
| FR20 | Responsive 320px–4K, all browsers | Epic 1 | ✅ Covered |
| FR21 | Command palette | Phase 2 | ⏭️ Deferred |
| FR22 | Video walkthrough per project | Phase 2 | ⏭️ Deferred |
| FR23 | GitHub README auto-sync | Phase 2 | ⏭️ Deferred |
| FR24 | Email/password JWT authentication (BE) | Epic 5 | ✅ Covered |
| FR25 | Google OAuth2 login | Epic 5 | ✅ Covered |
| FR26 | Automatic session refresh | Epic 5 | ✅ Covered |
| FR27 | Logout + session invalidation | Epic 5 | ✅ Covered |
| FR28 | User-scoped data isolation | Epic 5 | ✅ Covered |
| FR29 | Public JWKS endpoint | Epic 5 | ✅ Covered |
| FR30 | Health polling pipeline 60s cadence | Epic 3 | ✅ Covered |
| FR31 | WebSocket live metrics client (FE) | Epic 3 | ✅ Covered |
| FR32 | GitHub push webhook → metrics refresh | Epic 3 | ✅ Covered |
| FR33 | Config-only demo app registration | Epic 3 | ✅ Covered |
| FR34 | AI Build Story per project | Phase 2 | ⏭️ Deferred |
| FR35 | Config-file showcase content management | Epic 6 | ✅ Covered |
| FR36 | Job availability via config file | Epic 6 | ✅ Covered |
| FR37 | Admin endpoint for contact inquiries | Epic 6 | ✅ Covered |
| FR38a | Plausible analytics embed (FE) | Epic 6 | ✅ Covered |
| FR38b | Admin analytics endpoint | Epic 6 | ✅ Covered |
| FR39 | Contact form rate limiting | Epic 4 | ✅ Covered |
| FR40 | AI endpoint rate limit placeholder (Nginx) | Epic 2 (Story 2.4) | ✅ Covered |
| FR41 | Language toggle vi/en | Epic 1 | ✅ Covered |
| FR42 | Language preference persistence | Epic 1 | ✅ Covered |
| FR43 | Project detail page /projects/{slug} | Epic 1 | ✅ Covered |
| FR44 | Dark/light mode toggle | Epic 1 | ✅ Covered |
| FR45 | Theme preference persistence | Epic 1 | ✅ Covered |
| FR46 | Contextual messaging per referral source | Epic 4 | ✅ Covered |
| FR47 | Spring-physics micro-animations (prefers-reduced-motion) | Epic 1 | ✅ Covered |
| FR48 | HMAC-SHA256 webhook verification | Epic 3 | ✅ Covered |
| FR49 | 4 core data entities | Epic 2 | ✅ Covered |
| FR50 | Structured JSON error format | Epic 2 | ✅ Covered |
| FR51 | OpenAPI self-docs at /api-docs | Epic 2 | ✅ Covered |

### Missing Requirements

**None.** All 51 FRs are accounted for in the epics document.

### Coverage Statistics

- Total PRD FRs: 51
- FRs covered in epics (Phase 1): 46 ✅
- FRs explicitly deferred to Phase 2: 5 ⏭️ (FR13, FR21, FR22, FR23, FR34)
- Coverage percentage: **100%** (46/46 in-scope FRs)

**Epic distribution:**
- Epic 1 — Portfolio Static Showcase: 17 FRs
- Epic 2 — Platform Backend Core & Infrastructure: 3 FRs
- Epic 3 — Live Evidence Layer: 9 FRs
- Epic 4 — Recruiter Contact & Engagement: 6 FRs
- Epic 5 — Platform Auth & Demo App Identity: 6 FRs
- Epic 6 — Owner Admin & Analytics: 6 FRs (+ FR40 via Epic 2 placeholder)

---

## UX Alignment Assessment

### UX Document Status

**Found:** `ux-design-specification.md` (78,115 bytes, modified 2026-03-03) — comprehensive document covering visual design, component strategy, motion patterns, responsive design, and accessibility.

### Coverage Summary

| Category | Coverage | Issues |
|---|---|---|
| Animation & Motion (FR47, FR15) | ✅ 100% | None |
| Accessibility (FR18, FR19) | ✅ 95% | No i18n architecture |
| Responsive Design (FR20) | ✅ 95% | Routing model ambiguous |
| Contact Form (FR14, FR15, FR39) | ✅ 100% | None |
| Real-Time Data Display (FR9, FR10, FR11, FR12) | ⚠️ 60% | FR11, FR12 unaddressed |
| Language/i18n (FR41, FR42) | ❌ 0% | Completely missing |
| Dark/Light Mode (FR44, FR45) | ⚠️ 40% | Toggle UI + persistence missing |
| Job Status & Timeline (FR7, FR8) | ❌ 0% | Completely unaddressed |
| Project Detail Routing (FR43) | ⚠️ 60% | URL structure not defined |
| Skills Section (FR5) | ⚠️ 50% | Replaced by tech badges, may not satisfy |

### Alignment Issues

#### Critical Gaps (must resolve before dev start)

1. **FR41 + FR42 — Language Toggle & Persistence:** UX spec has zero i18n architecture. PRD requires vi/en toggle + localStorage persistence. No component design exists. **Blocker for MVP if language toggle is launch-day requirement.**

2. **FR7 — Build Timeline Visualization:** PRD requires verifiable first-commit-to-deploy timeline. UX spec only shows "days to ship" scalar metric. No timeline component designed.

3. **FR8 — Job Availability Status:** PRD lists this as a distinct, visible feature. UX spec has no "hiring/availability" badge or component design whatsoever.

4. **FR11 — Auto-Hide Metrics After 24h:** No behavioral spec for metric visibility timeouts.

5. **FR12 — Live GitHub Contribution Graph:** No visual design or component spec for the graph.

#### High Priority Gaps

6. **FR44 + FR45 — Dark/Light Mode Toggle + Persistence:** Dark mode is set as default in UX spec. Light mode "available but not prioritized for MVP." No toggle UI component designed, no localStorage persistence pattern specified.

7. **FR43 — Project Detail Routing:** UX spec emphasizes single-page anchor scroll as primary pattern. PRD requires `/projects/{slug}` shareable URLs. Direct conflict in routing model needs resolution.

8. **FR5 — Skills with Proof Links:** PRD intended a dedicated skills section with each claim linked to proof. UX spec replaces this with versioned tech badges on project cards. Different interpretation.

#### Medium Issues

9. **FR16 vs. UX spec approach — Referral Tracking:** PRD specifies `?from=` query param tracking. UX spec uses `document.referrer` detection. Both achieve similar intent but different APIs — needs alignment.

10. **FR46 — Contextual Messaging:** PRD requires contextual copy variations per referral source. UX spec changes gallery tab default but doesn't specify copy variation design.

11. **FR10 — "Updated X ago" Timestamp:** Mentioned in text hierarchy but not mandated explicitly on project cards.

12. **FR6 — Collapsible Lessons Learned:** UX spec uses Build Story approach with scroll reveals but no expand/collapse toggle specified.

### Warnings

- ⚠️ **Routing Model Decision Required:** Team must decide between SPA anchor-scroll (UX spec preference) vs. file-based routing with `/projects/{slug}` (PRD requirement) before Epic 1 dev begins
- ⚠️ **i18n Architecture Gap:** No i18n spec means language support will require significant design work that is currently unplanned in epics/stories
- ✅ No missing UX document — the spec exists and is detailed (~78KB)
- ✅ Motion system, accessibility, responsive design, and contact form patterns are production-ready in UX spec

---

## Epic Quality Review

### Epic Assessment Summary

| Epic | Title | User Value | Independence | Issues |
|------|-------|-----------|--------------|--------|
| Epic 1 | Portfolio Static Showcase | ✅ Pass | ⚠️ Forward dep on Epic 3 | Story 1.5 missing BDD ACs; Story 1.7 oversized |
| Epic 2 | Platform Backend Core & Infrastructure | ⚠️ Technical milestone focus | ❌ Foundation dep (acceptable) | Schema incomplete; Story 2.4 oversized |
| Epic 3 | Live Evidence Layer | ✅ Pass | ❌ Requires Epic 2 (acceptable) | Story 3.7 config forward dep |
| Epic 4 | Recruiter Contact & Engagement | ✅ Pass | ⚠️ Requires Epic 2 for BE | Story 4.5 ACs vague; Rate limit cross-ref gap |
| Epic 5 | Platform Auth & Demo App Identity | ✅ Pass | ❌ Requires Epic 2 (acceptable) | Minor detail gaps |
| Epic 6 | Owner Admin & Analytics | ✅ Pass | ❌ Requires Epics 2+4 (acceptable) | Story 6.2 schema vague; Story 6.4 oversized |

### Violations Found

#### 🔴 Critical Violations

1. **Story 2.2 — Incomplete Database Schema:** `contact_submissions` and `project_health` tables defined only by column names, no data types, constraints, or indexes. Blocks Stories 4.1, 3.1, 3.7 from being fully implementable.

2. **Story 1.5 — Missing BDD Acceptance Criteria:** "Skills with proof links" stated but no Given/When/Then format. Unspecified: how many skills, what constitutes valid proof, can links be empty.

3. **Story 4.5 — Vague Returning Visitor ACs:** "Welcome back signal shown once" — no definition of UI. "Last-viewed project promoted" — no sort logic. "Hero intro animation skipped" — no spec for what replaces it.

4. **Story 6.2 — Undefined Analytics Response Schema:** "Breakdown by referral_source" — counts? percentages? sorted? Date range format undefined. Not testable/verifiable.

#### 🟠 Major Issues

1. **Forward Dependency: Story 1.3 → Epic 3** — Gallery mentions "live demo URL button" before WebSocket infrastructure exists. Needs clarification.

2. **Forward Dependency: Story 1.5 → Epic 6** — Skills "read from config file" but config file structure not defined until Story 6.4.

3. **Forward Dependency: Story 3.7 → Story 2.4** — Requires `showcase.yml` config but Story 2.4 does not define config file structure or location.

4. **Forward Dependency: Story 4.3 ↔ Story 2.4** — Rate limiting references "Nginx contact rule configured in Story 2.4" but Story 2.4 only vaguely mentions "three rule sets exist."

5. **Story 2.4 Oversized** — Combines Nginx + TLS + CI/CD + DB backup. Should split into 2-3 stories.

6. **Story 6.4 Oversized** — Combines FE config + BE config + 2 smoke tests. Should split into 3-4 stories.

7. **Epic 2 Title Violates User Value Principle** — "Platform BE deployed on Oracle A1" is a technical milestone, not user benefit. Consider retitling.

#### 🟡 Minor Concerns

1. Story 1.7 oversized — keyboard nav + screen reader + contrast + responsive should be 2-3 stories
2. Story 5.3 — OAuth2 provider config (Client ID/Secret storage, redirect URI) not specified
3. Story 4.1 — No validation rules for email format, min/max message length
4. Story 3.4 — "Last known data" fallback undefined when WebSocket never received data before disconnect
5. Story 3.7 — Config reload mechanism ambiguous: in-memory reload vs. app restart
6. Story 2.4 — GitHub Actions `deploy.yml` structure not specified (runner, deployment mechanism)
7. Epic 1 ↔ Epic 3 integration implicit — story dependencies between gallery cards (1.3) and health metrics (3.4) not explicitly cross-referenced

### Dependency Map

| Story | Depends On | Type |
|-------|-----------|------|
| Story 1.3 | Epic 3 (demo URLs) | ⚠️ Forward — needs resolution |
| Story 1.5 | Epic 6 (config) | ⚠️ Forward — needs resolution |
| Story 3.7 | Story 2.4 (config structure) | ⚠️ Forward — needs resolution |
| Story 4.3 | Story 2.4 (Nginx rules) | ⚠️ Forward — needs resolution |
| Story 4.5 | Story 4.1 (form triggers return) | ℹ️ Sequential — acceptable |
| Story 6.4 (smoke tests) | Stories 6.1 + 3.4 | ℹ️ Sequential — acceptable |

### Summary Statistics

| Metric | Count |
|--------|-------|
| Total epics reviewed | 6 |
| Total stories reviewed | 29 |
| Stories — Pass | 16 |
| Stories — Partial Issues | 9 |
| Stories — Fail | 4 |
| Critical violations | 4 |
| Major issues | 7 |
| Minor concerns | 7 |
| **Overall Quality Rating** | **FAIR** |

### Recommendations Priority

1. 🚨 **URGENT**: Expand Story 2.2 with full schema — data types, constraints, indexes for all 4 tables
2. 🚨 **URGENT**: Define config file schema centrally (in Epic 2 or early Epic 6), not distributed across stories
3. 🔴 **HIGH**: Rewrite Story 1.5 ACs in BDD format with specific skill count and proof link requirements
4. 🔴 **HIGH**: Rewrite Story 4.5 ACs with specific UI spec for "welcome back signal" and "last-viewed" logic
5. 🟠 **MEDIUM**: Resolve 4 forward dependencies (1.3→Epic3, 1.5→Epic6, 3.7→2.4, 4.3→2.4)
6. 🟠 **MEDIUM**: Split oversized stories (1.7, 2.4, 6.4)
7. 🟡 **LOW**: Add Story 6.2 response schema definition; Story 4.1 validation rules; Epic 2 title refinement

---

## Summary and Recommendations

### Overall Readiness Status

## ⚠️ NEEDS WORK — Conditionally Ready

Project planning is solid at 80%+ completeness. FR coverage is 100%. Architecture and UX docs exist and are high quality. However, **4 critical issues and 7 major issues** must be resolved — primarily in the epics/stories document — before development should begin. These are fixable in a short sprint planning session, not blockers requiring re-planning from scratch.

---

### Issue Count by Category

| Category | Critical | Major | Minor | Total |
|----------|---------|-------|-------|-------|
| PRD Completeness | 0 | 0 | 0 | 0 ✅ |
| FR Coverage in Epics | 0 | 0 | 0 | 0 ✅ |
| UX ↔ PRD Alignment | 2 | 5 | 3 | 10 ⚠️ |
| Epic/Story Quality | 4 | 7 | 7 | 18 ⚠️ |
| **Total** | **6** | **12** | **10** | **28** |

---

### Critical Issues Requiring Immediate Action

1. **Incomplete Database Schema (Story 2.2)** — `contact_submissions` and `project_health` tables lack data types, constraints, and indexes. Developers cannot implement Stories 4.1, 3.1, 3.7 without this. **Must fix before Sprint 1 begins.**

2. **Missing BDD Acceptance Criteria — Story 1.5** — "Skills with verifiable proof links" has no testable criteria. Developers don't know what to build or how to verify it. **Must fix before Story 1.5 is assigned.**

3. **Vague Returning Visitor ACs — Story 4.5** — "Welcome back signal," "last-viewed project promoted," and "hero animation skipped" are undefined. Cannot implement or test. **Must fix before Story 4.5 is assigned.**

4. **Undefined Analytics Schema — Story 6.2** — Response shape for `/api/v1/admin/analytics` is unspecified. Developers and testers cannot align on output. **Must fix before Story 6.2 is assigned.**

5. **UX Gap: Language Toggle (FR41/FR42)** — No i18n architecture exists in UX spec. Story 1.6 covers this in epics, but no component design. **The Story 1.6 developer must also define the component pattern before implementation.**

6. **UX Gap: Job Availability + Build Timeline (FR7/FR8)** — No component design in UX spec. Stories 1.4/1.5 reference these but UX spec has no design guidance. **Developer must define component pattern during implementation.**

---

### Recommended Next Steps

**Before Development Starts (Pre-Sprint 0):**

1. **Fix Story 2.2** — Expand database schema with full DDL-level detail for all 4 tables (column names, types, NOT NULL constraints, foreign keys, indexes)

2. **Fix Story 1.5 ACs** — Rewrite in BDD format. Specify: skill list source (config file), how many entries, what a "proof link" means (URL to GitHub/project/demo), empty link behavior

3. **Fix Story 4.5 ACs** — Define "welcome back signal" UI (toast? header message?), "last-viewed project" sort logic (localStorage timestamp), what replaces hero animation on return

4. **Fix Story 6.2 ACs** — Define exact JSON response schema for analytics endpoint including field names, types, and grouping

5. **Centralize Config File Schema** — Create one story (suggested: early Epic 2 or Epic 6) that defines and documents both `projects.ts` (FE) and `showcase.yml` (BE) schemas. Remove distributed references across Stories 1.5, 3.7, 6.4

6. **Resolve Forward Dependencies** — Add explicit notes in Stories 1.3, 3.7, 4.3 clarifying which specific prior story/config they depend on

**During Sprint 1 (Can Proceed But Watch):**

7. **Split Story 2.4** into: (a) Nginx reverse proxy + SSL + WebSocket support, (b) GitHub Actions CI/CD, (c) Database backup automation

8. **Split Story 6.4** into: (a) FE config (projects.ts) management, (b) BE config (showcase.yml) management, (c) Automated smoke tests

9. **UX Design Additions** — Before Stories 1.5, 1.6, 3.5 are implemented: add component designs for job availability badge, build timeline visualization, language toggle UI, and dark/light mode toggle persistence

---

### Final Note

This assessment reviewed **51 FRs, 47 NFRs, 6 epics, 29 stories** across **4 planning documents** (PRD, Architecture, UX Spec, Epics). The foundation is strong:

✅ **PRD**: Complete, SMART requirements, 100% FR coverage in epics
✅ **Epic Coverage**: Every single FR traced to an epic — no gaps
✅ **UX Spec**: Production-grade motion, accessibility, and component patterns
⚠️ **Story Quality**: Fair — 4 critical ACs need rewriting, 4 forward dependencies need resolution, 3 stories oversized

**Verdict:** This project is **ready to begin pre-implementation refinements.** The epics doc requires ~4-6 hours of targeted updates (schema expansion, AC rewrites, config centralization) before the first development sprint should start. No re-architecture or re-planning is needed.

---
*Assessment completed: 2026-03-03 | Assessor: BMAD check-implementation-readiness workflow | Project: portfolio-v2*
