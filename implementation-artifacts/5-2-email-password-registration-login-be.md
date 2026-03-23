# Story 5.2: Email/Password Registration & Login (BE)

Status: done

## Story

As a demo app user,
I want to register and log in with email and password,
So that I can access my personal data in demo apps that support this flow.

## Acceptance Criteria

1. **Given** `POST /api/v1/auth/register` is called with email and password,
   **Then** a new user record is created with `provider = LOCAL` and the password stored as bcrypt hash with cost factor ≥ 12 (NFR-S4)

2. **Given** `POST /api/v1/auth/login` is called with valid credentials,
   **Then** the response contains an access token (JWT, 15m TTL) and a refresh token (opaque, 7d TTL stored in `sessions` table)

3. **Given** `POST /api/v1/auth/login` is called with invalid credentials,
   **Then** HTTP 401 is returned — no information about whether the email exists is leaked

4. **Given** a user registers with an already-used email,
   **Then** HTTP 409 is returned with structured error `{"error": {"code": "EMAIL_ALREADY_EXISTS", "message": "..."}}`

## Tasks / Subtasks

- [x] Task 1: Create User entity and repository (AC: #1)
  - [x] 1.1 Create `src/main/java/dev/chinh/portfolio/auth/user/User.java` — @Entity mapped to `users` table
  - [x] 1.2 Create `src/main/java/dev/chinh/portfolio/auth/user/UserRepository.java` — extends JpaRepository with `findByEmail()` method
  - [x] 1.3 Add `@EntityGraph` for lazy-loading if needed

- [x] Task 2: Create UserService with registration logic (AC: #1, #4)
  - [x] 2.1 Create `src/main/java/dev/chinh/portfolio/auth/user/UserService.java`
  - [x] 2.2 Method `register(RegisterRequest)` → validates email format, checks existing user, creates User with bcrypt hash (cost factor 12+)
  - [x] 2.3 Method `findByEmail(String)` → for login validation
  - [x] 2.4 Method `existsByEmail(String)` → for duplicate check

- [x] Task 3: Create DTOs for auth requests/responses (AC: #2, #4)
  - [x] 3.1 Create `RegisterRequest.java` — email, password fields with validation (@Email, @Size)
  - [x] 3.2 Create `LoginRequest.java` — email, password fields
  - [x] 3.3 Create `AuthResponse.java` — accessToken, refreshToken, tokenType fields
  - [x] 3.4 Create `UserDto.java` — user profile for potential /me endpoint

- [x] Task 4: Create AuthController endpoints (AC: #1, #2, #3, #4)
  - [x] 4.1 Create `src/main/java/dev/chinh/portfolio/auth/AuthController.java`
  - [x] 4.2 POST `/api/v1/auth/register` → calls UserService.register(), returns 201 with user info or 409
  - [x] 4.3 POST `/api/v1/auth/login` → validates credentials, generates tokens via JwtService, stores refresh token in sessions table, returns AuthResponse
  - [x] 4.4 Ensure /api/v1/auth/** is already public (from Story 5.1 SecurityConfig)

- [x] Task 5: Create SessionService for refresh token management (AC: #2)
  - [x] 5.1 Create `src/main/java/dev/chinh/portfolio/auth/session/SessionService.java`
  - [x] 5.2 Method `createSession(User)` → generates opaque refresh token, stores in sessions table with 7d expiry
  - [x] 5.3 Method `validateRefreshToken(String)` → checks token exists, not revoked, not expired
  - [x] 5.4 Method `revokeSession(String)` → for logout (Story 5.4)
  - [x] 5.5 Create Session entity if needed (mapping to existing `sessions` table)

- [x] Task 6: Update JwtService for user-based token generation (AC: #2)
  - [x] 6.1 Verify `JwtService.generateAccessToken(User)` exists from Story 5.1
  - [x] 6.2 Verify `JwtService.generateRefreshToken(User)` exists from Story 5.1
  - [x] 6.3 Ensure tokens include user ID as subject

- [x] Task 7: Password validation (AC: #1)
  - [x] 7.1 Add password requirements: min 8 chars, at least 1 uppercase, 1 lowercase, 1 number (per NFR-S4)
  - [x] 7.2 Add validation message in structured error format

- [x] Task 8: Write unit tests (AC: #all)
  - [x] 8.1 Create `src/test/java/dev/chinh/portfolio/auth/user/UserServiceTest.java`
  - [x] 8.2 Test: register creates user with bcrypt hash (cost factor 12+)
  - [x] 8.3 Test: register with duplicate email throws EMAIL_ALREADY_EXISTS
  - [x] 8.4 Test: login with valid credentials returns tokens
  - [x] 8.5 Test: login with invalid password returns 401 (timing-safe comparison)
  - [x] 8.6 Test: login with non-existent email returns 401 (same timing as invalid password)
  - [x] 8.7 Create `src/test/java/dev/chinh/portfolio/auth/AuthControllerTest.java`
  - [x] 8.8 Test: register endpoint returns 201 and user data
  - [x] 8.9 Test: register with duplicate email returns 409
  - [x] 8.10 Test: login returns 200 with tokens
  - [x] 8.11 Test: login with wrong password returns 401

## Dev Notes

### Architecture Overview

```
portfolio-platform/
├── src/main/java/dev/chinh/portfolio/
│   ├── auth/
│   │   ├── user/
│   │   │   ├── User.java                 ← @Entity: id, email, passwordHash, provider, role
│   │   │   ├── UserRepository.java       ← findByEmail(), existsByEmail()
│   │   │   └── UserService.java          ← register(), findByEmail()
│   │   ├── session/
│   │   │   ├── Session.java              ← @Entity: id, userId, refreshToken, expiresAt
│   │   │   ├── SessionRepository.java    ← findByRefreshToken(), findByUserId()
│   │   │   └── SessionService.java       ← createSession(), validateRefreshToken()
│   │   ├── dto/
│   │   │   ├── RegisterRequest.java      ← email, password
│   │   │   ├── LoginRequest.java         ← email, password
│   │   │   ├── AuthResponse.java         ← accessToken, refreshToken, tokenType
│   │   │   └── UserDto.java              ← id, email, provider, role
│   │   └── AuthController.java           ← POST /register, POST /login
│   └── shared/
│       └── config/
│           └── SecurityConfig.java       ← Already configured: /api/v1/auth/** is public
```

### CRITICAL: Database Schema

The `users` and `sessions` tables already exist from Story 2.2 (V1__create_core_schema.sql):

```sql
-- users table (already exists)
CREATE TABLE users (
    id            UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    email         VARCHAR(255) NOT NULL UNIQUE,
    password_hash VARCHAR(255),                              -- NULL for OAuth-only users
    provider      VARCHAR(20)  NOT NULL DEFAULT 'LOCAL',     -- 'LOCAL' | 'GOOGLE'
    role          VARCHAR(20)  NOT NULL DEFAULT 'USER',      -- 'USER' | 'OWNER'
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW(),
    updated_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);

-- sessions table (already exists)
CREATE TABLE sessions (
    id            UUID         PRIMARY KEY DEFAULT gen_random_uuid(),
    user_id       UUID         NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    refresh_token VARCHAR(512) NOT NULL UNIQUE,
    expires_at    TIMESTAMPTZ  NOT NULL,
    revoked       BOOLEAN      NOT NULL DEFAULT FALSE,
    created_at    TIMESTAMPTZ  NOT NULL DEFAULT NOW()
);
```

### CRITICAL: Security Requirements

1. **Password Hashing:**
   - Use bcrypt with cost factor ≥ 12 (NFR-S4)
   - Spring Security's `BCryptPasswordEncoder` defaults to strength 10, must override:
   ```java
   @Bean
   public PasswordEncoder passwordEncoder() {
       return new BCryptPasswordEncoder(12);
   }
   ```

2. **Timing-Safe Comparison:**
   - For login, compare password hash using timing-safe comparison to prevent timing attacks
   - Always return 401 for both "user not found" and "wrong password" (no info leak)

3. **Error Response Format:**
   - Use existing `ErrorResponse` and `ErrorDetail` from Story 2.3
   - EMAIL_ALREADY_EXISTS: `{"error": {"code": "EMAIL_ALREADY_EXISTS", "message": "Email already registered"}}`
   - INVALID_CREDENTIALS: `{"error": {"code": "INVALID_CREDENTIALS", "message": "Invalid email or password"}}`

4. **JWT Token Generation:**
   - Reuse `JwtService` from Story 5.1
   - Access token: 15 minute TTL (already configured in JwtConfig)
   - Refresh token: opaque random string, stored in sessions table, 7 day TTL

### CRITICAL: Login Flow

```
POST /api/v1/auth/login
├── 1. Find user by email (UserRepository.findByEmail)
├── 2. If user not found → return 401 (same as wrong password for timing safety)
├── 3. If user found → timing-safe password comparison
├── 4. If password invalid → return 401
├── 5. If password valid:
│   ├── Generate JWT access token via JwtService.generateAccessToken(user)
│   ├── Generate opaque refresh token (UUID.randomUUID().toString())
│   ├── Store refresh token in sessions table with 7d expiry via SessionService
│   └── Return { accessToken, refreshToken, tokenType: "Bearer" }
```

### CRITICAL: Registration Flow

```
POST /api/v1/auth/register
├── 1. Validate email format (@Email)
├── 2. Validate password requirements (min 8 chars, upper, lower, number)
├── 3. Check if email exists (UserRepository.existsByEmail)
├── 4. If exists → return 409 EMAIL_ALREADY_EXISTS
├── 5. If not exists:
│   ├── Generate bcrypt hash with cost factor 12+
│   ├── Create User with provider = 'LOCAL'
│   ├── Save to database
│   └── Return 201 with user info (no password in response)
```

### API Endpoints Summary

| Endpoint | Method | Auth | Request | Response |
|----------|--------|------|---------|----------|
| `/api/v1/auth/register` | POST | Public | `{"email": "...", "password": "..."}` | 201: `{"user": {...}}` or 409: error |
| `/api/v1/auth/login` | POST | Public | `{"email": "...", "password": "..."}` | 200: `{"accessToken": "...", "refreshToken": "...", "tokenType": "Bearer"}` or 401: error |

### Testing Standards

- Use `@SpringBootTest` with Testcontainers for integration tests
- Use `@WebMvcTest` with mocks for controller unit tests
- Test timing-safe comparison (mock BCrypt to test path)
- Test password validation messages
- Test duplicate email detection
- Test token generation and session storage

---

### Project Structure Notes

- **Repository root:** `I:/portfolio-v2/portfolio-platform/` (separate git root)
- **Module path:** Backend is `portfolio-platform`, NOT `portfolio-v2/portfolio-platform`
- This is BE only — no FE changes in this story
- JWT keys already exist at `src/main/resources/keys/`
- SecurityConfig already permits `/api/v1/auth/**`

### References

- [Source: epics.md#Story-5.2] — Acceptance criteria and user story
- [Source: epics.md#Epic-5] — Epic context (JWT auth with 15m/7d TTL)
- [Source: architecture.md#Authentication-Security] — JWT RS256, Spring Security config
- [Source: architecture.md#Project-Structure-portfolio-platform] — `auth/user/`, `auth/session/` package structure
- [Source: 5-1-jwt-infrastructure-spring-security-config-be.md] — JWT infrastructure, JwtService, SecurityConfig
- [Source: V1__create_core_schema.sql] — users and sessions tables already exist
- [Source: 2-3-structured-error-handler-openapi.md] — ErrorResponse, ErrorDetail classes to reuse

---

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- User entity already exists from Story 2.2 - no changes needed
- Session entity already exists from Story 2.2 - no changes needed
- UserRepository.findByEmail() already exists - added existsByEmail() method
- SessionRepository already exists - no changes needed
- JwtService already supports generateAccessToken(User) and generateRefreshToken(User) from Story 5.1
- SecurityConfig already permits /api/v1/auth/** - no changes needed
- Created new SecurityConfig in auth/config for PasswordEncoder bean with bcrypt strength 12

### Completion Notes List

- Implemented email/password registration with bcrypt cost factor 12
- Implemented login with JWT access token (15m TTL) and opaque refresh token (7d TTL stored in sessions table)
- Implemented timing-safe credential validation to prevent timing attacks
- Implemented proper error responses: 401 for invalid credentials, 409 for duplicate email
- Added password validation: min 8 chars, uppercase, lowercase, number
- Created comprehensive unit tests (51 tests, all passing)
- Code compiles successfully

### File List

**New Files:**
- `portfolio-platform/src/main/java/dev/chinh/portfolio/auth/dto/RegisterRequest.java`
- `portfolio-platform/src/main/java/dev/chinh/portfolio/auth/dto/LoginRequest.java`
- `portfolio-platform/src/main/java/dev/chinh/portfolio/auth/dto/AuthResponse.java`
- `portfolio-platform/src/main/java/dev/chinh/portfolio/auth/config/SecurityConfig.java` (PasswordEncoder bean)
- `portfolio-platform/src/main/java/dev/chinh/portfolio/auth/user/UserService.java`
- `portfolio-platform/src/main/java/dev/chinh/portfolio/auth/session/SessionService.java`
- `portfolio-platform/src/test/java/dev/chinh/portfolio/auth/user/UserServiceTest.java`
- `portfolio-platform/src/test/java/dev/chinh/portfolio/auth/session/SessionServiceTest.java`
- `portfolio-platform/src/test/java/dev/chinh/portfolio/auth/AuthControllerTest.java`

**Modified Files:**
- `portfolio-platform/src/main/java/dev/chinh/portfolio/auth/user/UserRepository.java` (added existsByEmail)
- `portfolio-platform/src/main/java/dev/chinh/portfolio/auth/AuthController.java` (implemented endpoints)
- `portfolio-platform/src/main/java/dev/chinh/portfolio/shared/error/GlobalExceptionHandler.java` (added exception handlers)

**Existing Files Verified:**
- User entity - already exists
- Session entity - already exists
- JwtService - already supports token generation
- SecurityConfig (shared) - already permits /api/v1/auth/**

---
