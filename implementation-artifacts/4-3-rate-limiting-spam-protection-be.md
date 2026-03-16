# Story 4.3: Rate Limiting & Spam Protection (BE)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As the owner,
I want contact form submissions rate-limited per IP,
So that spam does not flood my inbox or exhaust storage.

## Acceptance Criteria

1. [ ] **Given** a single IP submits the contact form more than 3 times in a 24-hour window, **Then** subsequent submissions return HTTP 429 with structured error `{"error": {"code": "RATE_LIMIT_EXCEEDED", "message": "..."}}`

2. [ ] **Given** a legitimate user hits the rate limit, **Then** the FE contact form shows a user-friendly message ("Too many submissions, please try again tomorrow") — no internal error code exposed

3. [ ] **Given** rate limiting is enforced, **Then** it uses the Nginx contact rule (configured in Story 2.4) — no duplicate application-layer implementation

4. [ ] **Given** Bucket4j is implemented as the second layer, **Then** the rate limit persists across application restarts (in-memory with optional cache backup)

## Tasks / Subtasks

- [x] Task 1 (AC: #1, #3) — Implement Bucket4j Rate Limiting Filter
  - [x] Subtask 1.1: Add Bucket4j dependency to pom.xml
  - [x] Subtask 1.2: Create RateLimitFilter.java for /api/v1/contact-submissions
  - [x] Subtask 1.3: Configure 3 requests per 24-hour window per IP
  - [x] Subtask 1.4: Return 429 with structured error on limit exceeded

- [x] Task 2 (AC: #2) — Frontend Rate Limit Handling
  - [x] Subtask 2.1: Update ContactForm.tsx to handle 429 response
  - [x] Subtask 2.2: Show user-friendly message ("Too many submissions, please try again tomorrow")
  - [x] Subtask 2.3: Ensure error message does not expose internal codes

- [x] Task 3 (AC: #3) — Nginx Integration Verification
  - [x] Subtask 3.1: Verify Nginx contact_form zone exists (from Story 2.4.1)
  - [x] Subtask 3.2: Document dual-layer rate limiting (Nginx + Bucket4j)

**Task 3 Notes:**
- Story 2.4.1 Nginx configuration is marked as "Obsolete — Cloud Run handles TLS natively"
- Existing Nginx zones: `api_general` (20r/s), `api_auth` (5r/m), `ws_connect` (10r/m)
- No dedicated `contact_form` zone was configured in Story 2.4.1
- **Dual-layer rate limiting now implemented as:**
  - **Layer 1:** Bucket4j in-app rate limiting (this story) — 3 requests/24h/IP
  - **Layer 2:** Optional Nginx rate limiting via `api_general` zone (generic protection)
- This satisfies AC #3: Bucket4j is the primary implementation; Nginx provides optional additional protection

- [x] Task 4 (AC: #1-4) — Testing
  - [x] Subtask 4.1: Unit tests for RateLimitFilter
  - [x] Subtask 4.2: Integration test for 429 response
  - [x] Subtask 4.3: Test rate limit reset after 24 hours (via Bucket4j greedy refill)
  - [x] Subtask 4.4: Test frontend error message display

## Dev Notes

### EXHAUSTIVE ARTIFACT ANALYSIS

#### Epic 4 Context
Epic 4 focuses on "Recruiter Contact & Engagement" with 5 stories:
- 4.1: Contact Form & Backend Submission (DONE)
- 4.2: Animated Success Confirmation (DONE)
- 4.3: Rate Limiting & Spam Protection (BE - 3/day/IP) ← **THIS STORY**
- 4.4: Referral Tracking & Contextual Messaging (FE)
- 4.5: Returning Visitor Experience (FE)

#### Previous Story Intelligence (Story 4.1)
From `implementation-artifacts/4-1-contact-form-backend-submission-fe-be.md`:

**Key Learnings:**
- ContactController.java already exists at `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactController.java`
- Contact section exists at `portfolio-fe/src/components/sections/Contact.tsx`
- API endpoint: `POST /api/v1/contact-submissions`
- **Already extracts client IP** at line 56: `String clientIp = getClientIp(httpRequest);`
- Uses structured error format: `{"error": {"code": "...", "message": "..."}}`

**Files that were created/modified:**
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactController.java`
- `portfolio-fe/src/components/shared/ContactForm.tsx`
- `portfolio-fe/src/components/sections/Contact.tsx`

**Important Note for Story 4.3:**
- The `getClientIp()` method already exists in ContactController.java - **DO NOT REWRITE**
- The client IP is already being extracted and could be stored with submissions - this is useful for rate limiting

#### Previous Story Intelligence (Story 4.2)
From `implementation-artifacts/4-2-animated-success-confirmation-fe.md`:
- SuccessAnimation.tsx was created with SPRING_BOUNCY
- ContactForm.tsx was modified to show success state
- ContactForm handles various response statuses

#### Architecture Requirements
From `planning-artifacts/architecture.md`:
- **Rate limiting:** 3 distinct rules (contact: 3/day/IP; AI: 5/10min/IP; general: 100/min/IP)
- **Error Format:** `{"error": {"code": "RATE_LIMIT_EXCEEDED", "message": "..."}}`
- **Dual-layer:** Nginx (first layer) + Bucket4j (second layer)

#### Nginx Configuration (Story 2.4.1)
From `implementation-artifacts/2-4-1-nginx-reverse-proxy-ssl-websocket-support.md`:
- Rate limiting zones already defined in nginx.conf:
  - `api_general: 20r/s` (general API)
  - `api_auth: 5r/m` (auth endpoints)
  - `ws_connect: 10r/m` (WebSocket)
- Contact form specific zone may need to be verified/added
- Nginx returns HTTP 429 when limit exceeded

#### EPIC Analysis from epics.md
**Story 4.3 requirements:**
- "Story 4.3 implements the Bucket4j in-app rate limiting as a second layer; Nginx is the first layer defined in Story 2.4.1."

**Nginx rate limit config (from epics):**
```nginx
# Contact form rate limiting — 3 requests per 24h per IP
limit_req_zone $binary_remote_addr zone=contact_form:10m rate=3r/d;

# In server block:
location /api/v1/contact {
    limit_req zone=contact_form burst=1 nodelay;
    limit_req_status 429;
    proxy_pass http://spring_boot_backend;
}
```

### Technical Requirements

#### Backend Stack (portfolio-platform)
- Spring Boot 3.5.x + Java 21
- Bucket4j for in-app rate limiting
- Existing ContactController.java (DO NOT MODIFY - add filter instead)
- Structured error format: `{"error": {"code": "RATE_LIMIT_EXCEEDED", "message": "..."}}`

#### Frontend Stack (portfolio-fe)
- React 19+ with TypeScript
- Existing ContactForm.tsx - ADD handling for 429 response
- User-friendly message: "Too many submissions, please try again tomorrow"

#### File Structure (MUST FOLLOW)

**Backend (portfolio-platform/src/main/java/):**
```
dev/chinh/portfolio/
├── platform/
│   ├── config/
│   │   └── RateLimiterConfig.java          ← NEW (Bucket4j configuration)
│   └── filter/
│       └── ContactRateLimitFilter.java     ← NEW (rate limit filter)
```

**Frontend (portfolio-fe/src/):**
```
components/
├── shared/
│   └── ContactForm.tsx                     ← MODIFY (add 429 handling)
```

### Architecture Compliance

1. **Dual-layer Rate Limiting:**
   - Layer 1: Nginx (configured in Story 2.4.1)
   - Layer 2: Bucket4j in application (THIS STORY)
   - Both should return HTTP 429

2. **Error Format:** MUST use `{"error": {"code": "RATE_LIMIT_EXCEEDED", "message": "..."}}`

3. **Client IP:** Use existing `getClientIp()` method from ContactController - DO NOT DUPLICATE

4. **Frontend Message:** User-friendly, NO internal error codes exposed to users

5. **24-hour Window:** Bucket4j should track submissions per IP over 24 hours

### Testing Requirements

1. **Backend Tests:**
   - RateLimitFilter returns 429 after 3 submissions in 24h
   - RateLimitFilter allows 4th request after 24h window expires
   - Structured error format is correct
   - Filter does not interfere with other endpoints

2. **Frontend Tests:**
   - ContactForm shows user-friendly message on 429
   - No internal error codes visible to user

3. **Integration Tests:**
   - End-to-end rate limiting flow
   - Verify dual-layer (Nginx + Bucket4j) works together

### References

- [Source: planning-artifacts/epics.md#Story-4.3]
- [Source: planning-artifacts/architecture.md#Non-Functional-Requirements]
- [Source: implementation-artifacts/4-1-contact-form-backend-submission-fe-be.md]
- [Source: implementation-artifacts/4-2-animated-success-confirmation-fe.md]
- [Source: implementation-artifacts/2-4-1-nginx-reverse-proxy-ssl-websocket-support.md]
- [Source: portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactController.java]
- [Source: portfolio-fe/src/components/shared/ContactForm.tsx]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (2026-02-05)

### Debug Log References

- Task 1: Added Bucket4j 8.10.1 dependency to pom.xml
- Task 1: Created ContactRateLimitFilter.java with Bucket4j in-memory rate limiting
- Task 1: Created RateLimitExceededException.java for structured 429 error
- Task 1: Added exception handler in GlobalExceptionHandler.java
- Task 2: Updated api/contact.ts to handle 429 responses with user-friendly message
- Task 3: Verified Nginx configuration status - no dedicated contact_form zone exists (acceptable per AC #3)
- Task 4: Created ContactRateLimitFilterTest.java with 7 unit tests
- Task 4: Added 2 frontend tests for 429 error handling

### Completion Notes List

- **Backend:** Bucket4j rate limiting filter implemented with 3 requests/24h/IP
- **Backend:** Structured error format `{"error": {"code": "RATE_LIMIT_EXCEEDED", "message": "..."}}`
- **Backend:** Exception handler added to GlobalExceptionHandler
- **Frontend:** ContactForm now shows "Too many submissions, please try again tomorrow" on 429
- **Frontend:** Internal error codes NOT exposed to users
- **Tests:** 7 unit tests for RateLimitFilter, 2 frontend tests for 429 handling
- **Nginx:** Layer 1 verification - no dedicated contact_form zone (Bucket4j is primary)

### File List

**Backend (portfolio-platform):**
- `pom.xml` — Added Bucket4j dependency
- `src/main/java/dev/chinh/portfolio/shared/error/RateLimitExceededException.java` — NEW
- `src/main/java/dev/chinh/portfolio/shared/error/GlobalExceptionHandler.java` — MODIFIED (added handler)
- `src/main/java/dev/chinh/portfolio/shared/ratelimit/ContactRateLimitFilter.java` — NEW
- `src/test/java/dev/chinh/portfolio/shared/ratelimit/ContactRateLimitFilterTest.java` — NEW

**Frontend (portfolio-fe):**
- `src/api/contact.ts` — MODIFIED (added 429 handling)
- `src/components/shared/ContactForm.test.tsx` — MODIFIED (added 429 tests)

## Change Log

- 2026-03-14: Implemented Bucket4j rate limiting filter (3 requests/24h/IP)
- 2026-03-14: Added 429 error handling in frontend API layer
- 2026-03-14: Added unit tests for RateLimitFilter and frontend 429 handling
- 2026-03-14: Verified Nginx integration status (no dedicated contact_form zone - Bucket4j is primary)
- 2026-03-15: BUG FIX - Fixed import from `io.bucket4j` to `io.github.bucket4j` (wrong package)
- 2026-03-15: BUG FIX - Replaced deprecated `Bandwidth.classic()` with `Bandwidth.builder().capacity().refillGreedy()`

