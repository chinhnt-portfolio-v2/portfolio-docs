---
stepsCompleted: ['step-01-init', 'step-02-discovery', 'step-02b-vision', 'step-02c-executive-summary', 'step-03-success', 'step-04-journeys', 'step-05-domain', 'step-01b-continue', 'step-06-innovation', 'step-01b-continue', 'step-07-project-type', 'step-08-scoping', 'step-09-functional', 'step-10-nonfunctional', 'step-11-polish', 'step-12-complete']
workflowStatus: 'complete'
completedAt: '2026-02-28'
lastEdited: '2026-03-01'
editHistory:
  - date: '2026-03-01'
    workflow: 'validate-prd → post-fix'
    changes: 'L1-L3 language consistency (NFR-P2/P3/P4b Vietnamese fragments); IL1-IL2 implementation leakage (NFR-I2 Spring Security ref, NFR-A5 Framer Motion ref); FR47 added (spring physics animations); FR49-FR51 added (Platform Data & API Contract — data entities, error contract, OpenAPI)'
  - date: '2026-03-01'
    workflow: 'edit-prd (step-e-03) + party-mode review'
    changes: 'SMART measurability improvements: FR2 (breathing room → ≥ 16px padding), FR15 (micro-celebration → animated confirmation ≤ 1.5s), FR17 (serves differently → streamlined returning-visitor experience with concrete example), FR34 (honest walkthrough → structured narrative min 300 words, 2 override decisions)'
inputDocuments:
  - '_bmad-output/brainstorming/brainstorming-session-2026-02-25.md'
workflowType: 'prd'
classification:
  projectType: 'web_app + api_backend'
  domain: 'general'
  complexity: 'medium'
  projectContext: 'greenfield'
  scope: 'Portfolio FE + Platform BE + Extensibility Spec. Demo apps = separate PRDs'
documentCounts:
  briefCount: 0
  researchCount: 0
  brainstormingCount: 1
  projectDocsCount: 0
---

# Product Requirements Document - portfolio-v2

**Author:** Chính
**Date:** 2026-02-25

## Executive Summary

Technical recruiters evaluating developers face a core verification problem: portfolio claims — "fast", "quality-driven", "production-ready" — are unverifiable without actually working with the candidate. portfolio-v2 giải quyết vấn đề này bằng cách thay thế claims bằng evidence có thể kiểm chứng trực tiếp.

portfolio-v2 là developer portfolio và application platform được rebuild hoàn toàn từ đầu, phục vụ technical recruiters và hiring managers đang evaluate backend/fullstack developers. Target user chính cần đánh giá năng lực thực tế của một developer trong ít hơn 5 phút — không phải đọc CV, mà xem proof.

Sản phẩm bao gồm hai components: **Portfolio FE** (Vite SPA, Vercel CDN) là marketing/showcase surface; **Platform BE** (Oracle A1 ARM) là shared infrastructure serving portfolio và tất cả demo apps — shared auth, cross-app data access, metrics APIs. Any new demo app can plug into Platform BE by implementing the auth contract and metrics API.

### What Makes This Special

portfolio-v2 hoạt động như một **evidence portfolio** — mỗi claim được back bởi một loại proof cụ thể, được deliver qua ba vectors đồng thời:

1. **Live output:** Demo apps chạy thật trên production infrastructure, metrics hiển thị real-time trên showcase (uptime, response time, last deploy). Recruiter không đọc về năng lực — họ quan sát nó.

2. **Documented process:** Bản thân portfolio-v2 được build với AI agents, đầy đủ PRD, architecture docs, và implementation specs từ ngày 1. Process là bằng chứng về cách làm việc, không chỉ kết quả.

3. **Speed signal:** Git commit history của mỗi project surface build timeline từ first commit đến production deploy — verifiable execution speed data, không thể fake, ngay lập tức readable trong developer culture.

Core insight: Trong thời AI, "build nhanh và tốt" là claim phổ biến và rẻ. Verifiable evidence — live products + documented methodology + real timelines — là scarce và không thể fake. Scarcity of proof là competitive moat.

**Anti-goal:** Bị bỏ qua. Bất kỳ hành động nào của recruiter sau khi visit đều tính là success — không có hành động nào là failure.

## Project Classification

| Field | Value |
|---|---|
| **Project Type** | `web_app` (Portfolio FE — Vite SPA, Vercel CDN) + `api_backend` (Platform BE — shared infrastructure, Oracle A1 ARM) |
| **Domain** | General — developer portfolio + multi-app platform |
| **Complexity** | Medium — shared auth, cross-app data access, AI layer, extensibility contract |
| **Project Context** | Greenfield — complete rebuild, zero code reuse from v1 |
| **Scope** | Portfolio FE + Platform BE + Extensibility Spec. Demo apps scoped separately per PRD khi có idea. |

## Success Criteria

### Technical Success

- **Zero cold start:** Platform BE runs on Oracle A1 (always-free, always-on) — no sleep, no cold start delay.
- **Fast first load:** Portfolio FE static SPA on Vercel CDN — LCP < 1.5s on first visit.
- **Graceful degradation:** If Platform BE metrics unavailable, showcase cards render correctly — only live data missing, not broken.
- **Free-tier sustainable:** Entire system runs on free tier indefinitely (Oracle A1 + Vercel free).
- **Extensibility contract:** Adding a new demo app requires zero changes to Portfolio FE core or Platform BE core.

### Measurable Outcomes

| Metric | Target | Notes |
|---|---|---|
| Frontend Lighthouse Performance | ≥ 90 (CI gate) / ≥ 95 (aspirational) | Enforced via Lighthouse CI on deploy |
| Platform BE uptime | ≥ 99.9% | Monitored via UptimeRobot |
| New demo app onboarding | ≤ 1 day | From dev complete → showcase live |
| Recruiter engagement | ≥ 1 genuine inquiry / 3-month cycle | Lagging indicator — market-dependent |
| Frontend load time (LCP) | < 1.5s | Vercel CDN, Vite SPA |
| Bounce rate | < 60% | Requires analytics setup (e.g. Plausible) |

## Product Scope

*See **Project Scoping & Phased Development** section for complete MVP feature set, sprint sequence, phased roadmap, and risk mitigation. Summary below.*

**MVP (Sprints 0-3):** Infrastructure validation → Platform BE (auth + WebSocket + metrics APIs) → Portfolio FE (8 FE features) → Wallet App Demo App #1. Two-phase launch: soft launch (FE + BE live, no CV distribution) → full launch (Wallet App live, CV distribution begins).

**Phase 2:** AI hover touchpoint, command palette, video walkthroughs, Lighthouse badge, README-driven pages, multiple project views.

**Phase 3:** Build Story visualization, Recruiter Wrapped, backlink constellation, open source template, public API, demo app ecosystem.

## User Journeys

### Journey 1 — Linh, Recruiter Fast Scanner

**Persona:** Linh, technical recruiter tại một công ty fintech 200 người. Mỗi ngày review 15-20 hồ sơ. Có một JD mở cho Java backend engineer.

**Opening Scene:** Linh đang review stack CV buổi chiều. Mở CV của Chính — format clean, experience relevant. Thấy dòng portfolio link. Nhìn đồng hồ — còn 30 giây trước meeting.

**Rising Action:** Click link. Portfolio load ngay lập tức — không có splash screen, không có loading spinner. Hero section: tên, role, availability widget hiển thị "Available · Remote / HCM City". Linh scroll xuống — projects section hiện ra đầu tiên. Cards clean, mỗi card có live metrics: *"Wallet App — 99.9% uptime · 187ms · deployed 3h ago"*. Không cần đọc gì cả — data đang nói thay.

**Climax:** Linh thấy GitHub contribution graph — 300+ day streak. Không phải screenshot. Live widget. Trong 20 giây, Linh đã thấy: người này available, người này có apps đang chạy thật, người này code consistently. Không một chữ về "passionate developer."

**Resolution:** Linh bookmark tab, ghi chú "strong signal — follow up." Meeting bắt đầu. Chính không bị bỏ qua — dù Linh chỉ dành 25 giây.

*Capabilities revealed: Instant load (SSG/CDN), projects-first layout, live metrics on showcase cards, availability widget, GitHub contribution graph widget.*

---

### Journey 2 — Minh, Recruiter Deep Explorer

**Persona:** Minh, senior recruiter tại một product company. Thích đánh giá kỹ trước khi reach out — không muốn waste engineer's time hay mình's time.

**Opening Scene:** Minh tìm được profile Chính trên LinkedIn — portfolio link trong bio. JD cần fullstack với Spring Boot experience. Click vào.

**Rising Action:** Hover vào Wallet App card 3 giây — một AI insight xuất hiện nhẹ nhàng: *"Built for personal finance tracking. Core challenge: real-time sync across devices without websockets."* *(AI hover touchpoint — Phase 2)* Minh bị hook. Click vào project gallery. Breathing room, không phải screenshot dump. "Artist statement": *"Tôi build cái này để giải quyết vấn đề tôi đang gặp — bank app của tôi không sync real-time."* Scroll xuống — thấy git timeline: first commit đến production deploy: **4 ngày**.

**Climax:** Minh click "What I'd do differently": *"Auth flow này tôi sẽ dùng refresh token rotation thay vì simple JWT — đây là lý do..."* Minh dừng lại. Đây không phải junior dev. Người này biết limitation của code mình đã viết và biết cách improve. Sau đó scroll đến About — "Why Hire Me" section: không có generic paragraph, chỉ có 3 claims mỗi claim kèm link proof. Dưới Wallet App card: uptime 99.9%, deployed 3h ago — không phải số tĩnh, timestamp đang đếm realtime.

*(AI hover touchpoint — Phase 2 — sẽ surface thêm context khi hover 3s vào project card)*

**Resolution:** Minh fill contact form. Submit — confetti nhỏ, message: *"Chính sẽ reply trong 24h — trong khi đó, bạn có muốn xem Music App không?"* Minh cười. Điền note vào ATS: "Strong candidate — portfolio unusually self-aware."

*Capabilities revealed: project gallery with artist statement, git timeline visualization, "What I'd do differently" section, "Why Hire Me" About section, social proof on claims, contact form with micro-celebration. AI hover touchpoint (Phase 2) shown as future state.*

---

### Journey 3 — David, Engineering Manager

**Persona:** David, Engineering Manager tại một startup Series A. Đang tìm Senior Backend Engineer. Nhận application của Chính qua email — portfolio link trong email signature.

**Opening Scene:** David đọc email trong lúc uống cà phê sáng. Click portfolio link — không phải để bị impressed bởi design, mà để tìm evidence of technical depth.

**Rising Action:** Scroll qua hero nhanh — thấy availability và contribution graph, gật đầu. Đi thẳng vào Wallet App. Không quan tâm screenshot — click "AI Build Story" *(Phase 2)*: *"Tôi dùng Claude để design database schema ban đầu, nhưng tôi override suggestion về indexing strategy vì traffic pattern của app này khác..."* David đọc chậm lại. Đây là điều David tìm kiếm — không phải "tôi dùng AI", mà "tôi biết khi nào tin AI và khi nào không."

**Climax:** David mở tab mới, search GitHub username từ contribution graph — thấy commit history public. Mở một commit random: message rõ ràng, diff clean. Quay lại portfolio — đọc Platform BE description trong About section: *"Backend là shared platform serving tất cả demo apps — shared auth, cross-app metrics."* David pause. Junior dev không nghĩ như vậy. Nhìn Wallet App card: uptime badge 99.9%, last deployed 2h ago. Một app đang chạy thật. Đây là evidence, không phải claim.

**Resolution:** David forward portfolio link cho CTO với note: "Worth talking to." Portfolio đã convert từ cold email thành internal recommendation — không cần cover letter.

*Capabilities revealed: public GitHub integration, Platform BE concept surfaced in portfolio, contribution graph with profile link, live uptime metrics on showcase cards. AI Build Story (Phase 2) sẽ surface thêm AI/human decision boundary detail.*

---

### Journey 4 — Chính, Owner/Admin

**Persona:** Chính — đã build và deploy Music App mới sau 2 tuần. App đang chạy trên Oracle A1. Muốn add vào portfolio showcase.

**Opening Scene:** Music App production-ready. Chính mở Platform BE admin (hoặc config file) — không phải CMS phức tạp.

**Rising Action:** Register Music App với Platform BE: implement auth contract (shared token validation) + metrics API endpoint (`/health`, `/metrics`). Platform BE nhận diện app mới. Chính update showcase config trong Portfolio FE repo: add entry cho Music App với metadata (name, description, URL, tech stack, artist statement). Push to main — Vercel auto-deploy.

**Climax:** Portfolio FE reload — Music App card xuất hiện trong showcase với live metrics tự động pull từ Platform BE. Uptime: 100% (vừa deploy). Response time: 89ms. Chính hover — AI insight hiển thị đúng như config.

**Resolution:** Total time từ "app ready" đến "showcase live": **4 giờ** (first time), **< 1 giờ** cho apps tiếp theo khi đã quen flow. Extensibility contract hoạt động đúng như designed — zero changes to Portfolio FE core hay Platform BE core.

*Capabilities revealed: Platform BE auth contract + metrics API, showcase config-driven (no CMS), Vercel auto-deploy on push, extensibility contract working end-to-end.*

---

### Journey 5 — Demo App End User

**Persona:** Thao, junior developer đang học personal finance management. Tìm trên Google "personal finance app Vietnam" — thấy link dẫn đến Wallet App của Chính.

**Opening Scene:** Thao land trên Wallet App — URL riêng, không phải portfolio. App chạy ngay, không có cold start delay (Oracle A1 always-on).

**Rising Action:** Thao sign up — auth flow smooth (Platform BE shared auth). Add vài transactions. Real-time sync hoạt động. Thao share app cho bạn. Vài ngày sau, quay lại — data vẫn còn, app vẫn hoạt động. Thao notice footer: *"Built by Chính — see how this was made"* — link về portfolio.

**Climax:** Thao click link, land trên portfolio — thấy Wallet App trong showcase với metrics đang chạy live. Đọc AI Build Story. Thao không phải recruiter — nhưng Thao là developer. Bookmark portfolio để học từ build approach.

**Resolution:** Wallet App có real user. Portfolio có organic traffic source ngoài job search. "Build less, build perfectly" — một app với real users là stronger proof than ten apps that nobody uses.

*Capabilities revealed: Platform BE always-on (zero cold start for demo apps), shared auth works across apps, portfolio backlink from demo apps, real usage validates "live output" vector.*

---

### Journey Requirements Summary

| Capability | Journeys |
|---|---|
| SSG frontend, instant load, zero splash screen | J1, J5 |
| Projects-first layout | J1, J2 |
| Live metrics on showcase cards (uptime, response time, last deploy) | J1, J2, J4 |
| Availability signal widget (config-driven) | J1, J3 |
| GitHub contribution graph (live widget) | J1, J3 |
| AI hover touchpoint on project cards *(Phase 2)* | J2 |
| Project gallery view (breathing room, artist statement) | J2, J3 |
| Git timeline visualization (first commit → production) | J2 |
| "What I'd do differently" collapsible section | J2 |
| "Why Hire Me" About section (claims + proof links) | J2, J3 |
| Contact form with micro-celebration on submit | J2 |
| AI Build Story *(Phase 2)* | J3 |
| Platform BE auth contract + metrics API | J4, J5 |
| Showcase config-driven (no CMS, push to deploy) | J4 |
| `?from=` URL param personalization | J1, J2 |
| Demo app backlink → portfolio | J5 |
| Platform BE always-on, zero cold start | J5 |

## Innovation & Novel Patterns

### Innovation Areas Identified

**Innovation 1: Kiến Trúc Platform-First**

portfolio-v2 đảo ngược hoàn toàn kiến trúc portfolio truyền thống. Mô hình thông thường: xây một portfolio site, tùy chọn thêm backend để phục vụ nó. portfolio-v2 đảo ngược điều này — backend là một **shared developer platform** (Oracle A1 ARM), và Portfolio FE chỉ đơn giản là một tenant bình đẳng trong hệ thống đó. Bất kỳ demo app nào đều có thể kết nối vào platform bằng cách implement shared auth contract và metrics API.

Đây không phải là tối ưu kỹ thuật — đây là product philosophy. Platform tồn tại độc lập với portfolio; portfolio là marketing surface của platform.

**Innovation 2: Evidence Portfolio**

Portfolio truyền thống hoạt động dựa trên claims: "nhanh", "chất lượng", "production-ready". Những claims này không thể kiểm chứng nếu không thực sự làm việc cùng candidate. portfolio-v2 thay thế claims bằng một kiến trúc evidence có cấu trúc, hoạt động đồng thời qua ba vectors:

- **Live output:** Demo apps đang chạy thật trên production infrastructure, với real-time metrics (uptime, response time, last deploy) hiển thị trực tiếp trên portfolio. Recruiter không đọc về năng lực — họ quan sát nó.
- **Documented process:** Bản thân việc build portfolio-v2 được document đầy đủ với PRD, architecture specs, và AI agent workflow từ ngày đầu. Process là bằng chứng về methodology, không chỉ là bằng chứng về output.
- **Speed signal:** Git commit history surface verifiable build timeline — từ first commit đến production deploy — theo format mà technical recruiter đọc được ngay lập tức. Không thể fake.

**Core insight:** Trong thời AI, "build nhanh và tốt" là claim phổ biến và rẻ. Verifiable evidence — live products + documented methodology + real timelines — khan hiếm và không thể làm giả. Sự khan hiếm của proof chính là competitive moat.

### Market Context & Competitive Landscape

Developer portfolio hiện tại đều vận hành theo mô hình claim-based. Dù được thiết kế tốt đến đâu (case study format, UI đẹp), recruiter vẫn phải có một bước nhảy cuối cùng dựa trên niềm tin — họ phải tin vào claims. Không có portfolio nào hiện tại giải quyết vấn đề verification như một design constraint cốt lõi.

Các analogues gần nhất là open-source projects (có thể verify qua commit history) và SaaS products (có thể verify qua live usage) — nhưng chúng không được định vị như portfolio artifacts. portfolio-v2 mượn verification logic từ cả hai và áp dụng vào hiring context.

### Validation Approach

- **Platform Innovation:** Được validate khi demo app thứ hai kết nối thành công vào Platform BE thông qua shared auth contract. Platform được chứng minh ngay khi có nhiều hơn một tenant.
- **Evidence Portfolio:** Được validate khi recruiter tương tác với live metric (ví dụ: click vào uptime badge, kiểm tra last deploy time) thay vì chỉ đọc claim. Proxy metric: time-on-site trên project cards có live status so với static cards.

### Risk Mitigation

| Risk | Mitigation |
|------|-----------|
| Platform BE cold start impacts portfolio | Portfolio FE là pure static — render đầy đủ không cần backend. AI/live features degrade gracefully. |
| Evidence architecture requires ongoing maintenance | Metrics được tự động hóa (CI/CD pipelines, health checks). Không cần update thủ công sau khi setup. |
| Git timeline có thể bị gian lận (bulk commits) | Commit granularity và PR history tự hiển thị với technical reviewer. Signal trung thực theo bản chất. |

## Project Scoping & Phased Development

### MVP Strategy & Philosophy

**MVP Approach:** Experience MVP + Platform MVP hybrid

portfolio-v2 không phải typical product MVP — không có "minimum viable" theo nghĩa chức năng tối thiểu. Thay vào đó, MVP phải deliver **evidence portfolio promise**: recruiter visit và thấy proof, không phải claims. Đồng thời, Platform BE phải đủ solid để Demo App #1 plug in ngay sau Portfolio FE launch.

**Core principle:** Ship ít, ship đúng. 8 FE features thực thi hoàn chỉnh còn hơn 15 features bị cắt góc.

**Priority order:** Platform thinking trước, recruiter experience sau — vì Platform đúng → FE có live data thật → showcase có evidence thật. Platform extensibility không thể retrofit; FE features có thể defer.

**Team:** Solo developer. Zero external team dependencies. Infrastructure: Oracle A1 (always-free) + Vercel (free tier).

**Launch strategy:** Hai-phase launch trong cùng MVP:
- **Phase 1a — Soft launch:** Portfolio FE + Platform BE live. Showcase hiển thị placeholder "Demo apps coming soon." URL live nhưng chưa distribute CV.
- **Phase 1b — Full launch:** Wallet App live, showcase hiển thị live metrics. CV bắt đầu được distribute. Recruiter chỉ thấy portfolio khi fully complete.

---

### Sprint Sequence

**Sprint 0 — Infrastructure Validation (1-2 ngày)**

Setup và validate toàn bộ infrastructure stack trước khi write business code.

| Task | Done Criteria |
|------|---------------|
| Oracle A1 provisioned, SSH accessible | SSH login thành công |
| Nginx running, HTTPS configured (Let's Encrypt) | `https://api.domain.com` trả về 200 |
| WebSocket endpoint test (`/ws/test` echo) | Browser console confirm WS connection open |
| Vercel project created, custom domain pointed | `https://portfolio.domain.com` live |
| GitHub Actions CI pipeline: push → test → deploy | First automated deploy thành công |

**Exit criteria Sprint 0:** Browser tab có thể open WebSocket connection đến Oracle A1 thành công. Không cần business logic — chỉ cần infra stack validated.

---

**Sprint 1 — Platform BE Core**

Full focus vào backend. FE không được touch trong sprint này.

| Feature | Notes |
|---------|-------|
| JWT auth (access 15min + refresh 7 days) | Foundation cho tất cả clients |
| Google OAuth2 social login | Demo app users |
| JWKS endpoint (`/api/v1/auth/jwks`) | JWT validation shared |
| Metrics aggregation pipeline | Poll demo app `/health` endpoints |
| WebSocket stream (`/ws/metrics`) | Push aggregated metrics tới FE |
| Portfolio APIs: contact, availability, GitHub contributions proxy | Support FE workstream 1 |
| Package structure: `auth.*`, `platform.*`, `apps.wallet.*` | Clean separation từ đầu |

**Exit criteria Sprint 1:** Demo App có thể plug in bằng cách implement `/health` + `/metrics` → Platform BE aggregates và pushes qua WebSocket thành công.

---

**Sprint 2 — Portfolio FE**

Build toàn bộ Portfolio FE. Dùng mock data cho metrics trong local dev, switch sang live WebSocket khi integration test.

| Feature | Journey |
|---------|---------|
| Projects-first layout | J1, J2 |
| Gallery principle per project (breathing room, artist statement) | J2, J3 |
| Live GitHub contribution graph widget | J1, J3 |
| Availability signal widget (config-driven) | J1, J3 |
| "Why Hire Me" reframe (claims + proof links) | J2, J3 |
| Spring physics animations throughout (Framer Motion) | J1, J2 |
| Contact form với micro-celebration on submit | J2 |
| `?from=` URL param personalization (localStorage) | J1, J2 |
| WebSocket client + "Last updated X ago" + graceful degradation | J1, J2 |

**Exit criteria Sprint 2 (Phase 1a):** Portfolio FE deployed to Vercel, connects to Platform BE WebSocket, soft launch ready.

---

**Sprint 3 — Wallet App (Demo App #1)**

Build Wallet App trên nền Platform BE đã stable. Focus vào real business domain — không phải toy app.

| Feature | Notes |
|---------|-------|
| Google login via Platform BE auth | Shared auth, zero re-implement |
| Transaction CRUD (add, edit, delete, categorize) | Core domain |
| Balance dashboard (by category, by period) | Recruiter-visible UX |
| `/health` endpoint → trả về uptime + response time | Platform contract |
| `/metrics` endpoint → trả về app-specific stats | Platform contract |
| User-scoped data isolation (proper repository queries) | Security — critical từ đầu |

**Exit criteria Sprint 3 (Phase 1b — Full Launch):** User login bằng Google, add transaction, xem balance. Portfolio showcase card hiển thị Wallet App live metrics. CV distribution bắt đầu.

---

### MVP Feature Set Summary

**Must-Have (Phase 1):**

| # | Feature | Workstream | Sprint |
|---|---------|-----------|--------|
| 1 | Projects-first layout | Portfolio FE | 2 |
| 2 | Gallery principle (breathing room, artist statement) | Portfolio FE | 2 |
| 3 | Live GitHub contribution graph widget | Portfolio FE | 2 |
| 4 | Availability signal widget (config-driven) | Portfolio FE | 2 |
| 5 | "Why Hire Me" reframe (claims + proof links) | Portfolio FE | 2 |
| 6 | Spring physics animations (Framer Motion) | Portfolio FE | 2 |
| 7 | Contact form với micro-celebration on submit | Portfolio FE | 2 |
| 8 | `?from=` URL param personalization | Portfolio FE | 2 |
| 9 | JWT auth + Google OAuth2 | Platform BE | 1 |
| 10 | Metrics aggregation + WebSocket stream | Platform BE | 1 |
| 11 | Contact / availability / GitHub proxy APIs | Platform BE | 1 |
| 12 | Wallet App (transactions CRUD + dashboard + platform contract) | Demo App #1 | 3 |

---

### Post-MVP Features

**Phase 2 — Growth (sau Full Launch):**

| Priority | Feature | Rationale |
|----------|---------|-----------|
| 🥇 First | AI hover touchpoint — hover 3s on project card → surface insight (3rd-party AI API) | Highest recruiter "wow" potential — differentiates từ 99% portfolios |
| 2 | Command palette (Cmd+K) với one-time discovery hint | Power user signal — technical recruiter delight |
| 3 | Video walkthroughs per project (90s hard limit, hook in 5s, transcript) | Personality + communication proof |
| 4 | Live Lighthouse performance badge (self-display score) | Meta-proof — portfolio proves its own quality |
| 5 | README-driven project pages (live fetch từ GitHub API) | Zero-maintenance sync |
| 6 | Multiple project views (Grid / Table / Timeline) | Enhanced exploration |

**Phase 3 — Expansion (future):**

| Feature | Rationale |
|---------|-----------|
| Build Story as Deployment Timeline visualization | High-effort, high-impact — significant FE work |
| Recruiter Wrapped — session summary on exit | Viral potential, post-solid-foundation |
| Backlink constellation — skill cross-linking | Advanced navigation, requires data modeling |
| Portfolio codebase as open source template | Community play — after portfolio polished |
| Public Portfolio API (`curl` easter egg) | Developer culture signal |
| Full ecosystem of demo apps với domain riêng | Ongoing — mỗi demo app là separate initiative |

---

### Risk Mitigation Strategy

**Technical Risks:**

| Risk | Mitigation |
|------|-----------|
| WebSocket pipeline complexity (BE polls apps → aggregates → pushes FE) | Build và validate end-to-end trong Sprint 0/1 — integration test của toàn bộ live metrics pipeline |
| Oracle A1 Nginx WebSocket config non-trivial | Sprint 0 dedicated — validate trước khi write business code |
| Wallet App data isolation per user | User-scoped repository queries từ ngày đầu Sprint 3 — không patch sau |

**Resource Risks (Solo Developer):**

| Risk | Mitigation |
|------|-----------|
| Context switching giữa 3 workstreams | Sequential sprint order: BE → FE → Demo App. Parallel chỉ khi integration testing |
| Scope creep từ brainstorming ideas | Phase 2/3 list là parking lot, không backlog — không add vào sprint mà không conscious decision |
| Demo App #1 blocking full launch | Wallet App scope giới hạn: login + CRUD + dashboard + platform contract. Feature-complete sau launch |

**Launch Risks:**

| Risk | Mitigation |
|------|-----------|
| Recruiter visits trong Phase 1a (no live metrics) | Soft launch — URL live nhưng không appear trong CV/LinkedIn cho đến Phase 1b |
| Live metrics offline khi recruiter visit | "Last updated X ago" timestamp → ẩn data nếu > 24h offline — không broken UI, không mislead |

## Technical Architecture Decisions

### Project-Type Overview

portfolio-v2 gồm hai hệ thống độc lập với concerns riêng biệt:

- **Portfolio FE**: Vite SPA, deployed to Vercel. Static single-page app phục vụ recruiter experience. Không cần SEO. Load từ CDN — không phụ thuộc Platform BE để render. **Portfolio FE không yêu cầu recruiter login** — frictionless experience là nguyên tắc tối thượng.
- **Platform BE**: Spring Boot monolith, deployed to Oracle A1 ARM. Một backend duy nhất phục vụ tất cả clients: Portfolio FE và tất cả demo app frontends. Provides shared auth, business APIs cho từng demo app, metrics aggregation, và platform APIs. Demo apps là separate frontends — chúng gọi về cùng một Platform BE.

### Portfolio FE — Technical Specifications

**Framework & Build:**
- Vite + React với `vite-plugin-ssg` — hybrid SSG + client hydration approach
- Build time: tất cả routes được pre-rendered thành static HTML
- Runtime: React hydrate client-side sau khi HTML đã hiển thị cho user
- Deployed as static assets to Vercel CDN — instant first paint từ CDN edge, không chờ JS execution

**Browser & Device Support:**
- Target: All modern evergreen browsers (Chrome, Firefox, Safari, Edge — latest 2 versions)
- Responsive design: Mobile-first, all screen sizes từ 320px đến 4K
- No IE11 support required

**Accessibility:** WCAG 2.1 AA compliance
- Keyboard navigability cho tất cả interactive elements
- Screen reader compatibility
- Sufficient color contrast (4.5:1 minimum)
- Focus indicators visible

**Real-Time Integration (WebSocket):**
- Portfolio FE kết nối tới Platform BE qua WebSocket để nhận **live project health metrics**
- Project cards hiển thị real-time: uptime %, response time (ms), last updated timestamp
- **Degradation strategy:** Khi WebSocket unavailable, FE chuyển sang polling fallback. Luôn hiển thị **"Updated X min ago"** timestamp — recruiter biết data mới đến mức nào, không bao giờ bị mislead bởi stale numbers. Khi metrics offline > 24h, ẩn số liệu hoàn toàn thay vì hiển thị stale data.
- WebSocket connection chỉ mở khi recruiter đang ở Projects section — không mở ngay khi page load (bandwidth optimization)

**Performance Targets:**
- Lighthouse Performance: ≥ 90 (CI gate) / ≥ 95 (aspirational) — see NFR-P3
- First Contentful Paint: < 1.5s
- Total Bundle Size: < 300KB gzipped

### Platform BE — Technical Specifications

**Architecture:**
- Single Spring Boot monolith on Oracle A1 ARM
- Package structure must maintain strict separation between `auth`, `platform`, and `apps` concerns from day one — enables future service extraction without refactoring
- Nginx reverse proxy in front of Spring Boot: SSL termination, WebSocket upgrade pass-through, rate limiting

**WebSocket Constraint:**
Reverse proxy must be explicitly configured to support WebSocket protocol upgrade — see NFR-I5.

**Authentication Architecture:**
- JWT-based auth: access token 15 min TTL, refresh token 7 ngày
- Google OAuth2 social login (Spring Security OAuth2 Client)
- **Auth chỉ dành cho demo app users** — recruiter visit Portfolio FE không cần login
- Demo app frontends authenticate users qua Platform BE; JWT validation shared across all apps qua JWKS public key endpoint
- Portfolio FE owner (Chính) có admin account riêng để access private admin endpoints

**API Design:**
- REST + JSON (primary interface)
- Base path: `/api/v1/` — single version, không versioning complexity
- Webhooks: Incoming webhooks hỗ trợ cho GitHub push events (triggers health check refresh cho affected projects)

**Rate Limiting:**
- IP-based rate limiting via Bucket4j (Spring Boot)
- General endpoints: 100 req/min per IP
- Contact form: **3 submissions/day per IP + honeypot field** — less restrictive với legitimate users (kể cả shared corporate IPs), effective hơn với spammers
- AI-powered endpoints: 10 req/min per authenticated user — áp dụng cho các features gọi 3rd-party AI API (ví dụ: generate project insight on hover). Rate limit per authenticated session để tránh abuse cost
- CORS: Allow-list cho Vercel production domain + localhost dev

### Platform BE — Endpoint Specification

**Auth Endpoints:**
```
POST /api/v1/auth/login           → Email/password login → JWT pair
POST /api/v1/auth/refresh         → Refresh access token
POST /api/v1/auth/logout          → Invalidate refresh token
GET  /api/v1/auth/oauth2/google   → Initiate Google OAuth2 flow (demo app users)
GET  /api/v1/auth/jwks            → Public key set (JWKS) for JWT validation
```

**Portfolio FE Endpoints:**
```
GET  /api/v1/availability              → Current job availability status (from config)
GET  /api/v1/github/contributions      → Proxy: GitHub contribution graph data
POST /api/v1/contact                   → Submit contact form (rate limited: 3/day/IP + honeypot)
GET  /api/v1/projects/{id}/health      → Static health snapshot for a project
WS   /ws/metrics                       → WebSocket: live health metrics stream
```

**Platform Admin Endpoints (private — Chính only):**
```
GET  /api/v1/admin/contacts      → List contact form submissions
GET  /api/v1/admin/analytics     → Visitor analytics summary
```

**Platform Health:**
```
GET  /actuator/health            → Spring Boot Actuator health check
```

**Incoming Webhooks:**
```
POST /api/v1/webhooks/github     → GitHub push event → trigger health check refresh
```

**Demo App APIs (example pattern):**
```
POST /api/v1/apps/wallet/...     → Wallet App business logic
POST /api/v1/apps/<name>/...     → Future demo app business logic
```

### Demo App Frontend Deployment

Mỗi demo app frontend được deploy như một **Vercel project riêng biệt** trên cùng Vercel account:

```
Vercel account (Chính):
├── portfolio-v2          → portfolio.chinh.dev   (Portfolio FE)
├── wallet-app            → wallet.chinh.dev       (Demo App #1)
└── [future-app]          → [app].chinh.dev        (Future demo apps)

Oracle A1 ARM:
└── api.chinh.dev         → Platform BE (serve tất cả frontends)
```

**Rationale:** Static assets trên Vercel CDN (không serve từ Oracle A1 để tránh resource contention). Vercel free tier không limit số projects trên cùng account — chỉ limit bandwidth. Custom subdomain mỗi app qua Vercel custom domain settings.

---

### Implementation Considerations

- **FE/BE Independence:** Portfolio FE renders hoàn toàn từ static assets — Platform BE chỉ cần thiết cho dynamic features (live metrics, contact, AI). FE vẫn load và useful khi BE offline.
- **WebSocket + Reverse Proxy:** Reverse proxy must pass WebSocket upgrade headers — does not work with default proxy config. See NFR-I5.
- **Monolith package discipline:** Single Spring Boot app with clear package structure from day one (`auth`, `platform`, `apps.*`) — ensures clean separation and easy service extraction if needed.
- **AI rate limiting precision:** AI endpoint rate limit (10/min per user) áp dụng riêng cho các routes gọi 3rd-party AI API. Features như hover insight sẽ khó hit limit này trong normal usage — limit chủ yếu protect against abuse.
- **Oracle A1 networking:** Firewall rules trên Oracle A1 phải open ports cho WebSocket connections (TCP upgrade), không chỉ standard HTTP/HTTPS.

## Functional Requirements

> **Capability Contract:** UX designers only design, architects only support, and epic breakdown only implements what is listed here. Missing FR = missing feature. Phase 2 items annotated — they are roadmap contracts, not Sprint 1-3.

### 1. Showcase & Project Discovery

- **FR1:** Recruiter can view featured projects as the primary section on portfolio entry (projects-first layout)
- **FR2:** Recruiter can view each project in a visually distinct gallery card with generous whitespace (≥ 16px padding) and contextual description
- **FR3:** Recruiter can read the WHY behind each project build (artist statement), not just the WHAT
- **FR4:** Recruiter can view Chính's background through recruiter-centric framing ("Why Hire Me")
- **FR5:** Recruiter can view Chính's skills with each claim linked to a verifiable proof artifact
- **FR6:** Recruiter can view "What I'd do differently" per project (collapsible, honest lessons learned)
- **FR7:** Recruiter can view a verifiable build timeline for each project (first commit → production deploy date)
- **FR8:** Recruiter can view Chính's current job availability status (including location preference)
- **FR43:** Recruiter can view a full project detail page with its own shareable URL (`/projects/{slug}`) — includes gallery, build timeline, lessons learned. AI Build Story section appears in Phase 2.

### 2. Live Evidence Layer

- **FR9:** Recruiter can view real-time health metrics for each demo app directly on the project card (uptime %, response time ms, last deploy)
- **FR10:** Recruiter can see the timestamp of the last metrics update ("Updated X ago") when real-time data is unavailable
- **FR11:** System automatically hides metrics when data has been offline for more than 24 hours — never displays stale or misleading numbers
- **FR12:** Recruiter can view Chính's live GitHub contribution graph (not a screenshot)
- **FR13:** Portfolio self-displays its own Lighthouse performance score as a trust signal *(Phase 2)*

### 3. Recruiter Engagement & Personalization

- **FR14:** Recruiter can submit a contact inquiry via form
- **FR15:** Recruiter receives an animated success confirmation (e.g., confetti burst or checkmark animation, ≤ 1.5s duration) immediately after successfully submitting the contact form
- **FR16:** Portfolio reads referral source from URL parameter (`?from=`) and saves to local storage
- **FR17:** Portfolio delivers a streamlined returning-visitor experience — e.g., reduced onboarding elements and quick access to last-viewed content — based on locally persisted browsing context
- **FR46:** Portfolio displays contextual messaging appropriate to referral source — e.g.: `?from=linkedin` (global platform) surfaces English-first professional framing; `?from=cv-vn` surfaces Vietnamese-friendly context

### 4. Portfolio Navigation, Accessibility & Appearance

- **FR18:** Recruiter can navigate the entire portfolio using keyboard only
- **FR19:** Portfolio content is accessible via screen readers (WCAG 2.1 AA)
- **FR20:** Portfolio renders correctly on all modern evergreen browsers and all screen sizes from 320px to 4K
- **FR21:** Recruiter can use command palette to jump to any section or project *(Phase 2)*
- **FR22:** Recruiter can view a video walkthrough (≤90 seconds) per project *(Phase 2)*
- **FR23:** Portfolio auto-syncs project page content from the corresponding GitHub repository README *(Phase 2)*
- **FR44:** Recruiter can switch appearance between dark mode and light mode
- **FR45:** Portfolio remembers recruiter's appearance preference across return visits (locally persisted)
- **FR47:** Recruiter experiences spring-physics micro-animations on portfolio interactions (hover effects, scroll reveals, page transitions); all animation is disabled when the `prefers-reduced-motion` OS setting is active

### 5. Internationalization (i18n)

- **FR41:** Recruiter can switch portfolio language between Vietnamese and English
- **FR42:** Portfolio remembers recruiter's language preference across return visits (locally persisted)
- **Scope:** Selective bilingual — UI labels + personal content sections (About, Why Hire Me, artist statements, project descriptions) have both `vi` and `en`. Technical content (code snippets, metrics, build timelines, GitHub data) English-only.
- **Default logic:** `?from=` param driven — global platform referrals (`?from=linkedin`) default English; Vietnamese referrals (`?from=cv-vn`) default Vietnamese. Explicit `?lang=vi` or `?lang=en` param override. Fallback: browser language detection.

### 6. Platform Auth & Identity

- **FR24:** Platform BE supports email/password authentication (JWT-based). Wallet App (Demo App #1) exposes only Google OAuth login on UI — email/password available for future demo apps with appropriate use cases.
- **FR25:** Demo app user can log in with a Google account
- **FR26:** Demo app user's session automatically refreshes without requiring re-login within the session window
- **FR27:** Demo app user can log out and invalidate their session
- **FR28:** Platform ensures each user can only access their own data (user-scoped isolation)
- **FR29:** Platform exposes a public JWKS endpoint for demo apps to validate JWT tokens independently

### 7. Metrics Platform & Extensibility Contract

- **FR30:** Platform BE aggregates health metrics from all registered demo apps by polling each app's `/health` endpoint every **60 seconds**. Detection latency ≤ 60s; WebSocket delivery latency after detection < 3s (see NFR-P5).
- **FR31:** Portfolio FE receives live health metric updates via WebSocket connection from Platform BE
- **FR32:** Platform BE automatically refreshes health metrics for a project upon receiving a GitHub push event webhook
- **FR33:** Owner can register a new demo app to the platform without changing application code anywhere. Process: (1) add a config entry to Platform BE config file to register the app endpoint — Platform BE auto-polls; (2) add a showcase entry to Portfolio FE config file with metadata (name, description, URL, tech stack, artist statement). Both are config file changes, not code changes — deployable via normal git push to each repo.
- **FR34:** Platform exposes AI Build Story per project — a structured written narrative of AI/human decision boundaries, minimum 300 words, highlighting at least 2 decisions where developer overrode AI suggestion *(Phase 2)*
- **FR48:** System verifies webhook signature (HMAC-SHA256 with GitHub secret) before processing any incoming webhook

### 8. Owner Administration

- **FR35:** Owner can update showcase configuration (project list, metadata, artist statements, build timelines) by editing config file and pushing to deploy
- **FR36:** Owner can update job availability status by editing config file
- **FR37:** Owner can view submitted contact form inquiries via private admin endpoint
- **FR38a:** Owner can view page-level analytics (pageviews, bounce rate, traffic sources) via **Plausible** dashboard — external tool, no BE implementation required. Plausible script is embedded in Portfolio FE.
- **FR38b:** Owner can view portfolio-specific engagement data (contact form submission count, `?from=` referral source breakdown) via private admin endpoint `/api/v1/admin/analytics`
- **FR39:** System enforces rate limits on contact form submissions to prevent spam
- **FR40:** System enforces rate limits on AI-powered endpoints to control third-party API costs *(Phase 2 — when AI features are built)*

### 9. Platform Data & API Contract

- **FR49:** Platform BE manages four core data entities: User (identity, auth state), Session (refresh token store), ContactSubmission (form inquiry data), and ProjectHealth (aggregated metrics snapshot); all user-owned entities are scoped to the authenticated user ID — cross-user data access returns 403
- **FR50:** Platform BE returns structured JSON error responses with a consistent `{"error": {"code": "...", "message": "..."}}` format for all 4xx and 5xx responses — no raw stack traces or unstructured error strings are exposed to clients
- **FR51:** Platform BE API is self-documented via OpenAPI specification accessible at `/api-docs` — spec covers all `/api/v1/` endpoints including request/response schemas and authentication requirements

## Non-Functional Requirements

### Performance

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| NFR-P1 | Portfolio FE Largest Contentful Paint | < 1.5s | Vercel CDN, static assets — first impression speed |
| NFR-P2 | Portfolio FE total bundle size (gzipped) | < 300KB | Fast load on mobile connections |
| NFR-P3 | Portfolio FE Lighthouse Performance score | ≥ 90 (CI gate) / ≥ 95 (aspirational) | CI gate enforced via Lighthouse CI; aspirational target for local dev. Portfolio self-displays score (Phase 2). |
| NFR-P4 | Platform BE internal API response time (p95) | ≤ 200ms | CRUD, auth, metrics aggregation endpoints. Excludes proxy/external calls. |
| NFR-P4b | Platform BE proxy/external API response time | ≤ 2s with hard timeout | GitHub API proxy, future AI endpoints. Timeout enforced; failure graceful. |
| NFR-P5 | WebSocket health metric delivery latency | < 3s after Platform BE detects change | Measured from when Platform BE receives poll result to FE card update. Polling detection window (60s) not included in this latency — see FR30. |
| NFR-P6 | Portfolio FE Time to Interactive | < 2s | Recruiter can scroll/click immediately without waiting for JS hydration |
| NFR-P7 | Contact form submission feedback | < 1s | Micro-celebration must fire immediately — perceived performance |

### Security

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| NFR-S1 | All client-server communication | HTTPS only (TLS 1.2+) | Encrypt in transit — Nginx SSL termination |
| NFR-S2 | JWT access token TTL | 15 minutes | Short-lived tokens, refresh flow required |
| NFR-S3 | JWT refresh token TTL | 7 days | Convenience vs security balance |
| NFR-S4 | Password storage | bcrypt (cost factor ≥ 12) | No plaintext or reversible encryption |
| NFR-S5 | User data isolation | Query-level enforcement | Every DB query scoped to authenticated user ID — no cross-user data leakage |
| NFR-S6 | Incoming webhook verification | HMAC-SHA256 signature check | Reject unverified payloads before processing |
| NFR-S7 | Admin endpoints | Authenticated + role-based | Only Owner account can access `/api/v1/admin/*` |
| NFR-S8 | CORS policy | Explicit allow-list only | Vercel production domain + localhost dev. No wildcard. |
| NFR-S9 | Rate limiting | Per-IP + per-user where applicable | Contact: 3/day/IP. AI endpoints: 5 req/10min/IP — unauthenticated recruiter experience, cannot rate limit by user (Phase 2). Limit is generous for a normal browsing session (5 project cards), primarily blocks abuse. General: 100/min/IP. |
| NFR-S10 | Wallet App financial data | No at-rest encryption required | Personal tracking data, not banking-grade. HTTPS sufficient. |

### Reliability

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| NFR-R1 | Platform BE uptime | 99.9% (≤ 8.7h downtime/year) | Oracle A1 always-on. Monitored via **UptimeRobot** (free, 5-min checks). Nginx + Spring Boot auto-restart on failure. |
| NFR-R2 | Portfolio FE availability | 99.99% (Vercel CDN SLA) | Static assets — practically always available |
| NFR-R3 | WebSocket reconnection | Automatic with exponential backoff | Max 3 retry attempts, then fallback to polling |
| NFR-R4 | Graceful degradation (BE offline) | Portfolio FE fully renders without BE | Dynamic features degrade; static content unaffected |
| NFR-R5 | Metrics staleness handling | Hide after 24h offline | "Updated X ago" visible; stale data never displayed as current |
| NFR-R6 | Database backup | Daily automated backup, RPO 24h | **Implementation:** cron job → PostgreSQL dump → Oracle Object Storage (free 20GB). Wallet App user data recoverable within 24h. |
| NFR-R7 | Critical happy path smoke tests | Pass on every deploy | Minimum coverage: (1) contact form submit → admin endpoint shows entry; (2) WebSocket connection → metrics update → FE card refreshes. Catch integration breakage automatically. |

### Accessibility

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| NFR-A1 | WCAG compliance level | 2.1 AA | All Portfolio FE pages and interactive elements |
| NFR-A2 | Color contrast ratio | ≥ 4.5:1 (text) / ≥ 3:1 (large text & UI components) | Both dark and light mode must independently meet contrast targets |
| NFR-A3 | Keyboard navigation | 100% interactive elements reachable | Tab order logical, focus indicators visible |
| NFR-A4 | Screen reader compatibility | All content accessible via ARIA | Images with alt text, form labels, landmark regions |
| NFR-A5 | Motion sensitivity | Respect `prefers-reduced-motion` | **Convention:** all animated components must check `prefers-reduced-motion` media query and disable animation when set. No exceptions. |

### Maintainability

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| NFR-M1 | Platform BE — Auth module unit test coverage | ≥ 80% | Security-critical; must catch regressions |
| NFR-M2 | Platform BE — Metrics aggregation unit test coverage | ≥ 80% | Core evidence layer — breakage visible to recruiter |
| NFR-M3 | Wallet App — Business logic unit test coverage | ≥ 70% | Demo app integrity; user financial data |
| NFR-M4 | Portfolio FE — Utility functions unit test coverage | ≥ 60% | Sufficient for SPA; focus on shared logic |
| NFR-M5 | Package structure discipline | Enforced from Sprint 1 | `auth.*`, `platform.*`, `apps.*` separation maintained. No cross-package direct dependencies outside defined contracts. |

### Integration

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| NFR-I1 | GitHub API dependency | Cached TTL 1h, in-memory | Contribution graph data. In-session fallback: last cached data if API unavailable. After server restart (empty cache): if API unavailable → hide contribution graph section entirely. Cache does not persist to disk — intentional, rare edge case. |
| NFR-I2 | Google OAuth2 | Standard OAuth2 authorization code flow | No custom token exchange |
| NFR-I3 | Future AI API integration (Phase 2) | Per-user rate limited, hard timeout 10s | AI calls must not block page render. Timeout → graceful failure, no spinner freeze. |
| NFR-I4 | GitHub Webhooks | Signature verified + idempotent processing | Duplicate delivery must not cause duplicate health refreshes |
| NFR-I5 | Reverse proxy WebSocket support | Must support HTTP → WS protocol upgrade | Reverse proxy must pass WebSocket upgrade headers without application-layer configuration. |

### Operational

| ID | Requirement | Target | Context |
|----|-------------|--------|---------|
| NFR-O1 | Portfolio FE deployment | Push to main → auto-deploy (Vercel) | Zero manual deployment steps |
| NFR-O2 | Platform BE deployment | GitHub Actions CI/CD → Oracle A1 | Automated deploy on merge to main |
| NFR-O3 | Health monitoring | UptimeRobot (free tier, 5-min checks) | Alerts to email on downtime. Satisfies NFR-R1 measurement. |
| NFR-O4 | Logging | Structured JSON logs, configurable log level | Debuggable without exposing sensitive data (no PII in logs) |
| NFR-O5 | Infrastructure cost | $0/month for infrastructure | Oracle A1 always-free + Vercel free tier. AI API costs (Phase 2) excluded — controlled via rate limiting (NFR-S9). |
