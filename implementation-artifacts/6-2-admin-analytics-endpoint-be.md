# Story 6.2: Admin Analytics Endpoint (BE)

Status: ready-for-dev

## Story

As the owner,
I want to view portfolio engagement data (contact form counts, referral source breakdown) via a private endpoint,
so that I can understand which channels are driving recruiter contact.

---

## 1. Acceptance Criteria

**AC-1 — Analytics endpoint returns correct data shape**

Given `GET /api/v1/admin/analytics` is called with a valid Owner JWT,
When the request is processed,
Then the response body matches this exact JSON schema:

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

**AC-2 — Owner role enforced**

Given `GET /api/v1/admin/analytics` is called without authentication or with a non-Owner JWT,
When the request is processed,
Then HTTP 403 is returned with structured error: `{"error": {"code": "FORBIDDEN", "message": "..."}}`

**AC-3 — Empty data returns zero counts**

Given no contact submissions exist in the database,
When `GET /api/v1/admin/analytics` is called,
Then HTTP 200 is returned with all counts at 0, empty `recentSubmissions` array, null dateRange values.

**AC-4 — byReferralSource sorting and rounding**

Given submissions exist with referral source data,
When the response is built,
Then entries in `byReferralSource` are sorted by `count` descending; `percentage` values are rounded to 1 decimal place; `null` source represents direct/unknown visits.

**AC-5 — recentSubmissions privacy**

Given `recentSubmissions` is populated,
When the response is built,
Then each entry contains only `id`, `email`, `submittedAt`, `referralSource` — the `message` body is NEVER returned (privacy).

**AC-6 — Timestamps in ISO 8601 UTC**

Given any timestamp in the response,
When the response is built,
Then all timestamps use ISO 8601 UTC format with `Z` suffix (e.g., `"2026-03-03T14:00:00Z"`).

---

## 2. User Story

**As** the owner,
**I want** to view portfolio engagement data (contact form counts, referral source breakdown) via a private endpoint,
**so that** I can understand which channels are driving recruiter contact.

---

## 3. Tasks / Subtasks

### Task 1: Analytics Aggregation Query (ContactSubmissionRepository) — AC: #1

- [ ] Add `countAll()`, `countBySubmittedAtAfter(LocalDateTime)`, `countByReferralSource(String)`, `findAllOrderBySubmittedAtDesc()` methods to `ContactSubmissionRepository`
- [ ] Add `findBySubmittedAtBetweenOrderBySubmittedAtDesc(LocalDateTime from, LocalDateTime to)` for date range queries
- [ ] Add `count()` aggregate by referral source — handle null via `COALESCE` or native query

### Task 2: Analytics Service (AdminAnalyticsService) — AC: #1, #3, #4, #5, #6

- [ ] `AnalyticsSummaryDto` record with `totalSubmissions`, `last30DaysCount`, `last7DaysCount`
- [ ] `ReferralSourceStatDto` record with `source`, `count`, `percentage`
- [ ] `RecentSubmissionDto` record with `id`, `email`, `submittedAt`, `referralSource` — NO message field
- [ ] `DateRangeDto` record with `earliest`, `latest` (nullable for empty data)
- [ ] `AdminAnalyticsDto` record composing all of the above
- [ ] Build `byReferralSource` list: count per source, compute percentage (total > 0 ? count/total*100 : 0), round to 1dp, sort descending
- [ ] Handle `null` referral source as `source: null` entry
- [ ] Limit `recentSubmissions` to last 20 entries
- [ ] Format all timestamps as ISO 8601 UTC `Z`-suffix strings

### Task 3: Analytics Controller (AdminAnalyticsController) — AC: #1, #2, #6

- [ ] `GET /api/v1/admin/analytics` mapping with `@PreAuthorize("hasRole('OWNER')")`
- [ ] Delegate to `AdminAnalyticsService.getAnalytics()`
- [ ] Return `ResponseEntity<AdminAnalyticsDto>` with HTTP 200
- [ ] No auth → Spring Security returns 401/403 before reaching controller

### Task 4: Authorization Guard (NFR-S7) — AC: #2

- [ ] Verify Spring Security config (`SecurityConfig.java`) permits only OWNER role on `/api/v1/admin/**`
- [ ] Add `@PreAuthorize("hasRole('OWNER')")` on controller or method level
- [ ] Verify 403 is returned for non-Owner JWT via integration test

### Task 5: Unit Tests — AC: all

- [ ] `AdminAnalyticsServiceTest`: test all ACs — zero submissions, mixed referral sources, null sources, percentage rounding, date range
- [ ] `AdminAnalyticsControllerTest`: test 200 for Owner, 403 for non-Owner, 401 for unauthenticated

---

## 4. Dev Notes

### Project Structure Notes

**Location:** `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/admin/`

```
platform/
└── admin/
    ├── AdminAnalyticsController.java   ← GET /api/v1/admin/analytics (Owner only)
    ├── AdminAnalyticsService.java    ← aggregation logic
    ├── dto/
    │   ├── AnalyticsSummaryDto.java     ← record: totalSubmissions, last30Days, last7Days
    │   ├── ReferralSourceStatDto.java   ← record: source, count, percentage
    │   ├── RecentSubmissionDto.java     ← record: id, email, submittedAt, referralSource (NO message)
    │   ├── DateRangeDto.java            ← record: earliest, latest (nullable)
    │   └── AdminAnalyticsDto.java      ← record: summary, byReferralSource[], recentSubmissions[], dateRange
    └── AdminAnalyticsMapper.java     ← Entity → DTO mapping
```

**ContactSubmissionRepository location:** `platform/contact/ContactSubmissionRepository.java`

### Key Implementation Details

**Repository query for referral source breakdown:**
```java
// Group by referral_source including NULL
@Query("SELECT c.referralSource, COUNT(c) FROM ContactSubmission c GROUP BY c.referralSource")
List<Object[]> countGroupByReferralSource();

// Handle null in result — returns [null, count] for direct/unknown
```

**Percentage calculation:**
```java
double percentage = totalCount > 0 ? (count * 100.0 / totalCount) : 0.0;
BigDecimal rounded = BigDecimal.valueOf(percentage).setScale(1, RoundingMode.HALF_UP).doubleValue();
```

**Date range boundary:**
```java
LocalDateTime now = LocalDateTime.now(ZoneOffset.UTC);
LocalDateTime last30Days = now.minusDays(30);
LocalDateTime last7Days = now.minusDays(7);
```

**ISO 8601 UTC timestamp formatting:**
```java
// Jackson ObjectMapper configured globally — LocalDateTime serialized as ISO 8601 UTC
// Verify: ensure application.yml has jackson serialization: WRITE_DATES_AS_TIMESTAMPS = false
// Or use ZonedDateTime/Instant for explicit UTC
Instant.now().atZone(ZoneOffset.UTC).toString(); // → "2026-03-03T14:00:00Z"
```

### Cross-Story Dependencies

- **Story 5.1 (JWT Infrastructure):** Required — `SecurityConfig` with `/api/v1/admin/**` role guard must already be in place. Check `SecurityConfig.java` before assuming the role guard is configured.
- **Story 4.1 (Contact Form BE):** Required — `ContactSubmission` entity, `ContactSubmissionRepository`, and `contact_submissions` table must exist before this story.
- **Story 6.1 (Admin Contact Inquiries Endpoint):** Recommended — `AdminContactController` pattern is identical. Use as reference for `AdminAnalyticsController` structure and `SecurityConfig` admin role guard setup.
- **Story 5.5 (User Data Isolation & JWKS):** Required — `SecurityConfig` with OWner role must be configured.

### Gotchas

- **`recentSubmissions` must NEVER include message body** — this is a privacy requirement. Use `AdminAnalyticsMapper` to explicitly map only the 4 permitted fields.
- **`source: null` in JSON** — Jackson serializes Java `null` as JSON `null` by default. Do NOT replace with the string `"direct"` — the spec explicitly says `null` for direct/unknown.
- **Percentage rounding** — use `RoundingMode.HALF_UP` (standard banker's rounding). Verify at exactly x.05 cases (e.g., 1/3 ≈ 33.3, not 33.4 vs 33.3 depending on rounding direction).
- **`/api/v1/admin/analytics` vs `/api/v1/admin/analytics/`** — trailing slash behavior should be consistent. Prefer no trailing slash. Configure accordingly in `SecurityConfig` and controller mapping.
- **Test isolation** — use `@SpringBootTest` with Testcontainers. Do NOT assume a pre-populated database.

### References

- [Source: _bmad-output/planning-artifacts/epics.md → Epic 6 → Story 6.2 ACs]
- [Source: _bmad-output/planning-artifacts/architecture.md → API Boundary Map → Admin endpoints]
- [Source: _bmad-output/planning-artifacts/architecture.md → Communication Patterns → API Response Formats]
- [Source: _bmad-output/planning-artifacts/architecture.md → Data Boundaries → contact_submissions table ownership]
- [Source: _bmad-output/implementation-artifacts/4-1-contact-form-backend-submission-fe-be.md → ContactSubmissionRepository schema]
- [Source: _bmad-output/planning-artifacts/architecture.md → Naming Patterns → API Endpoint Naming → Admin endpoints use `/admin/` prefix]
- [Source: _bmad-output/planning-artifacts/architecture.md → Format Patterns → Error Format: {"error": {"code", "message"}}]
- [Source: _bmad-output/planning-artifacts/architecture.md → Implementation Patterns → Backend Service Layer Pattern]
- [Source: _bmad-output/planning-artifacts/architecture.md → Enforcement Summary → Rule: No MapStruct]

---

## Dev Agent Record

### Agent Model Used

Claude Sonnet 4.6 (Anthropic) — 2026-03-19

### Debug Log References

### Completion Notes List

### File List

**New files:**
- `src/main/java/dev/chinh/portfolio/platform/admin/dto/AnalyticsSummaryDto.java`
- `src/main/java/dev/chinh/portfolio/platform/admin/dto/ReferralSourceStatDto.java`
- `src/main/java/dev/chinh/portfolio/platform/admin/dto/RecentSubmissionDto.java`
- `src/main/java/dev/chinh/portfolio/platform/admin/dto/DateRangeDto.java`
- `src/main/java/dev/chinh/portfolio/platform/admin/dto/AdminAnalyticsDto.java`
- `src/main/java/dev/chinh/portfolio/platform/admin/AdminAnalyticsMapper.java`
- `src/main/java/dev/chinh/portfolio/platform/admin/AdminAnalyticsService.java`
- `src/main/java/dev/chinh/portfolio/platform/admin/AdminAnalyticsController.java`
- `src/test/java/dev/chinh/portfolio/platform/admin/AdminAnalyticsServiceTest.java`
- `src/test/java/dev/chinh/portfolio/platform/admin/AdminAnalyticsControllerTest.java`

**Modified files:**
- `src/main/java/dev/chinh/portfolio/platform/contact/ContactSubmissionRepository.java` — add aggregation query methods

---

## 5. Technical Compliance Checklist

Before marking story done, verify:

- [ ] `AdminAnalyticsController` has `GET /api/v1/admin/analytics` mapping
- [ ] `@PreAuthorize("hasRole('OWNER')")` present on controller or method
- [ ] `SecurityConfig` has rule: `.requestMatchers("/api/v1/admin/**").hasRole("OWNER")`
- [ ] `AdminAnalyticsDto` has all 4 fields: `summary`, `byReferralSource`, `recentSubmissions`, `dateRange`
- [ ] `recentSubmissions` entries NEVER include `message` field
- [ ] `byReferralSource` entries include `source: null` for direct/unknown visits
- [ ] All percentages rounded to 1 decimal place with `RoundingMode.HALF_UP`
- [ ] All timestamps formatted as ISO 8601 UTC with `Z` suffix
- [ ] Empty data returns HTTP 200 with zero counts (not 404 or error)
- [ ] Unit tests cover: zero submissions, mixed referral sources, null source, rounding edge cases, date range
- [ ] Controller tests cover: 200 for Owner JWT, 403 for non-Owner, 401 for unauthenticated
- [ ] `GlobalExceptionHandler` in `shared/error/` handles all error cases consistently
- [ ] No `@ControllerAdvice` anywhere else in the codebase
- [ ] All timestamps in `application.yml` use UTC zone configuration
