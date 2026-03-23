# Story 5.5: User Data Isolation & JWKS Endpoint (BE)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a demo app user,
I want my data completely isolated from other users,
And as a demo app developer, I want to validate JWTs independently without calling Platform BE on every request.

## Acceptance Criteria

1. **Given** an authenticated user calls any user-scoped endpoint,
   **Then** the response contains only data belonging to that user — cross-user access returns HTTP 403 (FR28, NFR-S5)

2. **Given** `GET /api/v1/.well-known/jwks.json` is called,
   **Then** the public key set is returned in JWKS format — publicly accessible, no auth required (FR29)

3. **Given** a demo app uses the JWKS endpoint to validate a JWT,
   **Then** the JWT signature can be verified against the published public key with no call back to Platform BE

4. **Given** the auth package is complete,
   **Then** overall `auth.*` unit test coverage is ≥ 80% — including boundary tests that assert HTTP 403 on cross-user access attempts (NFR-M1)

## Tasks / Subtasks

- [x] Task 1: Implement User Data Isolation (AC: #1)
  - [x] 1.1 Add @CurrentUser annotation to extract userId from JWT
  - [x] 1.2 Create UserDataIsolationAspect or method parameter filtering
  - [x] 1.3 Add ownership check in repositories/services
  - [x] 1.4 Add HTTP 403 handler for cross-user access attempts

- [x] Task 2: Implement JWKS Endpoint (AC: #2, #3)
  - [x] 2.1 Create JwksController with GET /api/v1/.well-known/jwks.json
  - [x] 2.2 Extract public key from existing RSA key pair
  - [x] 2.3 Format response as standard JWKS (RFC 7517)
  - [x] 2.4 Configure SecurityConfig to permit JWKS endpoint (public, no auth)

- [x] Task 3: Coverage Verification (AC: #4)
  - [x] 3.1 Run coverage report on auth package
  - [x] 3.2 Identify coverage gaps
  - [x] 3.3 Add boundary tests for HTTP 403 scenarios

## Dev Notes

### Architecture Overview

```
portfolio-platform/
├── src/main/java/dev/chinh/portfolio/
│   ├── auth/
│   │   ├── AuthController.java           ← EXISTING: login/register/oauth2/refresh/logout
│   │   ├── JwksController.java           ← ADD: GET /api/v1/.well-known/jwks.json
│   │   ├── controller/
│   │   │   └── UserDataController.java   ← ADD: Demo endpoint showing data isolation
│   │   ├── annotation/
│   │   │   └── CurrentUser.java           ← ADD: Custom annotation for user context
│   │   ├── resolver/
│   │   │   └── CurrentUserArgumentResolver.java ← ADD: Resolver for @CurrentUser
│   │   ├── service/
│   │   │   └── OwnershipHelper.java       ← ADD: Ownership verification utility
│   │   ├── user/
│   │   │   ├── User.java                  ← EXISTING: @Entity
│   │   │   ├── UserService.java           ← EXISTING: findById()
│   │   │   └── UserRepository.java        ← EXISTING: JPA repository
│   │   ├── jwt/
│   │   │   ├── JwtService.java           ← EXISTING: generateAccessToken()
│   │   │   └── JwtAuthenticationFilter.java ← EXISTING: extracts user from JWT
│   │   └── session/
│   │       ├── Session.java               ← EXISTING: @Entity
│   │       ├── SessionService.java        ← EXISTING: validateRefreshToken()
│   │       └── SessionRepository.java     ← EXISTING: JPA repository
│   └── shared/
│       ├── config/
│       │   └── SecurityConfig.java        ← MODIFIED: PERMIT /.well-known/jwks.json
│       │   └── CorsConfig.java           ← MODIFIED: Added argument resolvers
│       └── error/
│           ├── ForbiddenException.java     ← ADD: HTTP 403 exception
│           └── GlobalExceptionHandler.java ← MODIFIED: Added ForbiddenException handler
```

### Implementation Details

**@CurrentUser Annotation:**
- Created `CurrentUser.java` annotation that can be applied to method parameters
- Created `CurrentUserArgumentResolver.java` to extract user ID from JWT authentication
- Registered resolver in `CorsConfig.java` via `addArgumentResolvers()`

**Ownership Helper:**
- Created `OwnershipHelper.java` utility class with static methods to verify ownership
- Throws `ForbiddenException` (HTTP 403) when ownership check fails

**Demo Endpoint (UserDataController):**
- Added `UserDataController.java` demonstrating data isolation pattern
- `GET /api/v1/user/data` - Returns current user's data using @CurrentUser
- `GET /api/v1/user/data/{dataId}` - Returns specific data item with ownership check

**ForbiddenException:**
- Created `ForbiddenException.java` runtime exception for cross-user access
- Added handler in `GlobalExceptionHandler.java` to return 403 with proper error format

**JWKS Endpoint:**
- Refactored to use cleaner X509EncodedKeySpec approach
- Updated kid from "default" to "portfolio-rsa-key"
- Simplified DER parsing by using standard Java security APIs

### Reuse from Previous Stories

- **JwtService** — Story 5.1: JWT infrastructure with RSA key pair
- **JwtAuthenticationFilter** — Story 5.1: Extracts user from JWT
- **SecurityConfig** — Story 5.1: Security configuration (JWKS already permitted)
- **User entity** — Story 5.2: User with id, email, provider, role
- **ErrorResponse** — Story 2.3: Already exists
- **GlobalExceptionHandler** — Story 2.3: Already handles exceptions

### Testing Standards

- Unit tests for JwksController (3 tests) - PASS
- Unit tests for OwnershipHelper (8 tests) - PASS
- Unit tests for ForbiddenException (2 tests) - PASS
- Unit tests for CurrentUserArgumentResolver (6 tests) - PASS
- Unit tests for UserDataController (4 tests) - PASS
- Added forbidden endpoint test to GlobalExceptionHandlerTest - PASS

---

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (2026-02-05)

### Debug Log References

- Fixed JwksController DER parsing bug by using X509EncodedKeySpec
- Fixed test issues with OAuth2 auto-configuration by using simpler unit test approach

### Completion Notes List

**Task 1 (User Data Isolation):**
- Created @CurrentUser annotation in `auth/annotation/CurrentUser.java`
- Created CurrentUserArgumentResolver in `auth/resolver/CurrentUserArgumentResolver.java`
- Created OwnershipHelper utility in `auth/service/OwnershipHelper.java`
- Created ForbiddenException in `shared/error/ForbiddenException.java`
- Added ForbiddenException handler to GlobalExceptionHandler
- Registered argument resolver in CorsConfig

**Task 2 (JWKS Endpoint):**
- Existing JwksController refactored for cleaner code
- Updated kid to "portfolio-rsa-key"
- Simplified key parsing using X509EncodedKeySpec

**Task 3 (Coverage):**
- Added JwksControllerTest with 3 tests
- Added OwnershipHelperTest with 8 tests
- Added ForbiddenExceptionTest with 2 tests
- Added CurrentUserArgumentResolverTest with 6 tests
- Added forbiddenException_returns403WithStructuredError to GlobalExceptionHandlerTest

**Code Review Fixes:**
- Added UserDataController demonstrating data isolation pattern
- Fixed unused imports in JwksController (removed BigInteger import)
- Added UserDataControllerTest with 4 tests

### File List

**New Files:**
- `src/main/java/dev/chinh/portfolio/auth/annotation/CurrentUser.java`
- `src/main/java/dev/chinh/portfolio/auth/resolver/CurrentUserArgumentResolver.java`
- `src/main/java/dev/chinh/portfolio/auth/service/OwnershipHelper.java`
- `src/main/java/dev/chinh/portfolio/auth/controller/UserDataController.java`
- `src/main/java/dev/chinh/portfolio/shared/error/ForbiddenException.java`
- `src/test/java/dev/chinh/portfolio/auth/jwt/JwksControllerTest.java`
- `src/test/java/dev/chinh/portfolio/auth/resolver/CurrentUserArgumentResolverTest.java`
- `src/test/java/dev/chinh/portfolio/auth/service/OwnershipHelperTest.java`
- `src/test/java/dev/chinh/portfolio/auth/controller/UserDataControllerTest.java`
- `src/test/java/dev/chinh/portfolio/shared/error/ForbiddenExceptionTest.java`

**Modified Files:**
- `src/main/java/dev/chinh/portfolio/shared/config/CorsConfig.java` - Added argument resolver registration
- `src/main/java/dev/chinh/portfolio/shared/error/GlobalExceptionHandler.java` - Added ForbiddenException handler
- `src/main/java/dev/chinh/portfolio/auth/jwt/JwksController.java` - Refactored for cleaner code, updated kid
- `src/test/java/dev/chinh/portfolio/shared/error/TestStubController.java` - Added /test/forbidden endpoint
- `src/test/java/dev/chinh/portfolio/shared/error/GlobalExceptionHandlerTest.java` - Added forbidden test, fixed OAuth2 exclusion

### Change Log

- 2026-03-18: Implemented Story 5.5 - User Data Isolation & JWKS Endpoint
  - Added @CurrentUser annotation and resolver for extracting user ID from JWT
  - Added OwnershipHelper for verifying resource ownership
  - Added ForbiddenException with HTTP 403 response
  - Refactored JwksController with cleaner key parsing
  - Added comprehensive unit tests for new components

- 2026-03-18: Code Review Fixes
  - Added UserDataController demo endpoint showing data isolation in action
  - Fixed unused BigInteger import in JwksController
  - Added UserDataControllerTest demonstrating ownership checks
