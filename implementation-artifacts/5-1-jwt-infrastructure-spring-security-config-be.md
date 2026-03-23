# Story 5.1: JWT Infrastructure & Spring Security Config (BE)

Status: done

## Story

As a developer,
I want JWT-based authentication infrastructure set up with,
So correct Spring Security configuration that all subsequent auth flows share a single, consistent security foundation.

## Acceptance Criteria

1. **Given** the Spring Security config is in place,
   **Then** all `/api/v1/` endpoints require a valid JWT Bearer token by default — except explicitly permitted public paths (`/api/v1/contact`, `/api/v1/webhooks/**`, `/api/v1/auth/**`, `/api-docs/**`, `/actuator/health`)

2. **Given** a valid JWT is issued,
   **Then** it uses RS256 (asymmetric signing) with access token TTL of 15 minutes and refresh token TTL of 7 days (NFR-S2, NFR-S3)

3. **Given** an expired or invalid JWT is sent,
   **Then** the API returns HTTP 401 with structured error `{"error": {"code": "UNAUTHORIZED", "message": "..."}}`

4. **Given** CORS is configured,
   **Then** only the Vercel production domain and localhost are in the allow-list — no wildcard (NFR-S8)

5. **Given** the JWT infrastructure is complete,
   **Then** unit test coverage for the `auth.jwt` package is ≥ 80% (NFR-M1)

## Tasks / Subtasks

- [x] Task 1: Generate RSA key pair for RS256 (AC: #2)
  - [x] 1.1 Generate RSA private key (2048-bit) and store in `src/main/resources/keys/private.pem`
  - [x] 1.2 Extract public key to `src/main/resources/keys/public.pem`
  - [x] 1.3 Add keys to `.gitignore` — NEVER commit private keys
  - [x] 1.4 Add key generation script: `scripts/generate-keys.sh` for local dev

- [x] Task 2: Create JWT configuration properties (AC: #2)
  - [x] 2.1 Create `src/main/java/dev/chinh/portfolio/auth/config/JwtConfig.java` — `@ConfigurationProperties(prefix = "jwt")`
  - [x] 2.2 Add to `application.yml`:
    ```yaml
    jwt:
      issuer: https://api.portfolio-v2.com
      access-token-ttl-minutes: 15
      refresh-token-ttl-days: 7
      key-path: classpath:keys/private.pem
    ```

- [x] Task 3: Create JwtService (AC: #2)
  - [x] 3.1 Create `src/main/java/dev/chinh/portfolio/auth/jwt/JwtService.java`
  - [x] 3.2 Methods: `generateAccessToken(User)`, `generateRefreshToken(User)`, `validateToken(String)`, `extractUsername(String)`, `extractClaims(String)`
  - [x] 3.3 Use RS256 with loaded RSA private key from config
  - [x] 3.4 Use Spring's `JwtDecoder` for validation with public key

- [x] Task 4: Create JwtAuthenticationFilter (AC: #1, #3)
  - [x] 4.1 Create `src/main/java/dev/chinh/portfolio/auth/jwt/JwtAuthenticationFilter.java`
  - [x] 4.2 Extend `OncePerRequestFilter` — extracts JWT from `Authorization: Bearer <token>` header
  - [x] 4.3 Validates token via `JwtService`, sets `SecurityContextHolder` authentication
  - [x] 4.4 Chain continues if valid; if invalid → let entry point handle (returns 401)

- [x] Task 5: Create JwtAuthenticationEntryPoint (AC: #3)
  - [x] 5.1 Create `src/main/java/dev/chinh/portfolio/auth/jwt/JwtAuthenticationEntryPoint.java`
  - [x] 5.2 Implements `AuthenticationEntryPoint` — returns HTTP 401 with structured error `{"error": {"code": "UNAUTHORIZED", "message": "..."}}`
  - [x] 5.3 Use existing `ErrorResponse` and `ErrorDetail` from Story 2.3

- [x] Task 6: Update SecurityConfig (AC: #1, #3, #4)
  - [x] 6.1 Load `SecurityConfig.java` from Story 2.1
  - [x] 6.2 Add `@Bean` for `SecurityFilterChain` with JWT filter chain:
    - Default: all `/api/v1/**` requires authentication
    - Public paths permitted: `/api/v1/contact`, `/api/v1/webhooks/**`, `/api/v1/auth/**`, `/api-docs/**`, `/actuator/health`
  - [x] 6.3 Add `JwtAuthenticationFilter` before `UsernamePasswordAuthenticationFilter` (or disable that filter)
  - [x] 6.4 Set `JwtAuthenticationEntryPoint` as authentication entry point
  - [x] 6.5 Configure CORS: allowedOrigins = Vercel domain + localhost (no wildcard)

- [x] Task 7: Create JWKS endpoint (AC: #future)
  - [x] 7.1 Create `src/main/java/dev/chinh/portfolio/auth/jwt/JwksController.java` at `GET /api/v1/.well-known/jwks.json`
  - [x] 7.2 Returns public key set in standard JWKS format — this is for Story 5.5, but key must be loadable now
  - [x] 7.3 Make public (no auth required) — added to permitAll

- [x] Task 8: Write unit tests (AC: #5)
  - [x] 8.1 Create `src/test/java/dev/chinh/portfolio/auth/jwt/JwtServiceTest.java`
  - [x] 8.2 Test: `generateAccessToken` creates valid RS256 JWT with correct claims
  - [x] 8.3 Test: `generateRefreshToken` creates valid JWT with 7-day expiry
  - [x] 8.4 Test: `validateToken` returns true for valid token, false for expired/invalid
  - [x] 8.5 Test: `extractUsername` returns correct subject
  - [x] 8.6 Test: Invalid/expired token throws appropriate exception or returns false
  - [x] 8.7 Create `src/test/java/dev/chinh/portfolio/auth/jwt/JwtAuthenticationFilterTest.java`
  - [x] 8.8 Test: Valid Bearer token → authentication set in security context
  - [x] 8.9 Test: Missing/invalid token → chain continues (entry point handles)
  - [x] 8.10 Coverage target: ≥ 80% for `auth.jwt` package

- [ ] Task 9: Integration test (optional, AC: #1, #3)
  - [ ] 9.1 Create `src/test/java/dev/chinh/portfolio/auth/jwt/JwtIntegrationTest.java`
  - 9.2 Test: Request to protected endpoint without token → 401
  - 9.3 Test: Request with valid JWT → 200
  - 9.4 Test: Request with expired JWT → 401

## Dev Notes

### Architecture Overview

```
portfolio-platform/
├── src/main/java/dev/chinh/portfolio/
│   ├── auth/
│   │   ├── config/
│   │   │   └── JwtConfig.java              ← @ConfigurationProperties
│   │   ├── jwt/
│   │   │   ├── JwtService.java             ← Token generation/validation (RS256)
│   │   │   ├── JwtAuthenticationFilter.java ← Request filter (OncePerRequestFilter)
│   │   │   ├── JwtAuthenticationEntryPoint.java ← 401 handler
│   │   │   └── JwksController.java         ← /api/v1/.well-known/jwks.json
│   │   └── AuthController.java             ← Placeholder for /api/v1/auth/**
│   └── shared/
│       └── config/
│           └── SecurityConfig.java         ← Modified: add JWT filter chain
├── src/main/resources/
│   ├── keys/
│   │   ├── private.pem                     ← RSA private key (gitignored)
│   │   └── public.pem                      ← RSA public key
│   └── config/
│       └── showcase.yml                    ← Add JWT config section
```

### CRITICAL: RSA Key Generation

RS256 requires asymmetric keys. Generate on first startup:

```bash
# Generate 2048-bit RSA private key
openssl genrsa -out src/main/resources/keys/private.pem 2048

# Extract public key
openssl rsa -in src/main/resources/keys/private.pem -pubout -out src/main/resources/keys/public.pem

# Ensure gitignored
echo "**/keys/*.pem" >> .gitignore
```

**NEVER commit `private.pem`** — add to `.gitignore` before first commit.

### CRITICAL: SecurityConfig Update

The existing `SecurityConfig.java` from Story 2.1 must be MODIFIED (not replaced). Key changes:

```java
// Add to existing SecurityConfig.java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfig {

    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    private final JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint;

    public SecurityConfig(JwtAuthenticationFilter jwtAuthenticationFilter,
                          JwtAuthenticationEntryPoint jwtAuthenticationEntryPoint) {
        this.jwtAuthenticationFilter = jwtAuthenticationFilter;
        this.jwtAuthenticationEntryPoint = jwtAuthenticationEntryPoint;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable()) // Stateless API
            .cors(cors -> cors.configurationSource(corsConfigurationSource()))
            .authorizeHttpRequests(auth -> auth
                .requestMatchers(
                    "/api/v1/contact",
                    "/api/v1/webhooks/**",
                    "/api/v1/auth/**",
                    "/api-docs/**",
                    "/v3/api-docs/**",
                    "/swagger-ui/**",
                    "/swagger-ui.html",
                    "/actuator/health"
                ).permitAll()
                .requestMatchers("/api/v1/**").authenticated()
                .anyRequest().permitAll()
            )
            .exceptionHandling(ex -> ex
                .authenticationEntryPoint(jwtAuthenticationEntryPoint)
            )
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }

    @Bean
    public CorsConfigurationSource corsConfigurationSource() {
        CorsConfiguration configuration = new CorsConfiguration();
        configuration.setAllowedOrigins(Arrays.asList(
            "https://portfolio-v2.vercel.app",  // Vercel production
            "http://localhost:5173",             // Vite dev
            "http://localhost:3000"              // Alternative dev
        ));
        configuration.setAllowedMethods(Arrays.asList("GET", "POST", "PUT", "DELETE", "OPTIONS"));
        configuration.setAllowedHeaders(Arrays.asList("Authorization", "Content-Type"));
        configuration.setAllowCredentials(true);
        configuration.setMaxAge(3600L);

        UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
        source.registerCorsConfiguration("/**", configuration);
        return source;
    }
}
```

**Key points:**
- CSRF disabled (stateless JWT)
- CORS: NO wildcard — specific origins only
- JWT filter runs BEFORE `UsernamePasswordAuthenticationFilter` (or filter is disabled since we're using JWT only)
- Entry point handles 401 with structured error format

### CRITICAL: Error Response Format

Use existing classes from Story 2.3:
```java
// JwtAuthenticationEntryPoint.java
@RestController
public class JwtAuthenticationEntryPoint implements AuthenticationEntryPoint {
    @Override
    public void commence(HttpServletRequest request,
                         HttpServletResponse response,
                         AuthenticationException authException) throws IOException {
        ErrorResponse errorResponse = new ErrorResponse(
            new ErrorDetail("UNAUTHORIZED", "Authentication required")
        );
        response.setStatus(HttpServletResponse.SC_UNAUTHORIZED);
        response.setContentType("application/json");
        new ObjectMapper().writeValue(response.getOutputStream(), errorResponse);
    }
}
```

### JWT Token Claims

Standard JWT claims for access token:
```json
{
  "sub": "user-uuid",
  "iss": "https://api.portfolio-v2.com",
  "iat": 1234567890,
  "exp": 1234567890 + 900,  // 15 minutes
  "type": "access"
}
```

Refresh token: same structure but `exp` = 7 days, `type`: "refresh"

### JWKS Endpoint (Story 5.5 pre-requisite)

For Story 5.5 (JWKS endpoint), the public key must be loadable:

```java
@RestController
@RequestMapping("/api/v1/.well-known")
public class JwksController {

    private final JwtConfig jwtConfig;

    @GetMapping("/jwks.json")
    public Map<String, Object> getJwks() throws Exception {
        String publicKeyPEM = new String(Files.readAllBytes(Paths.get(
            jwtConfig.getKeyPath().replace("classpath:", ""))));

        // Parse PEM to get RSA key parameters
        // Return in standard JWKS format:
        // { "keys": [ { "kty": "RSA", "n": "...", "e": "AQAB", ... } ] }
        // This is implemented fully in Story 5.5, but key must be accessible now
        return Map.of("keys", List.of()); // Placeholder for Story 5.5
    }
}
```

**Important:** Make this endpoint public (no auth required) — add to `permitAll()`.

### Testing Standards

- Use `@SpringBootTest` with Testcontainers for integration tests
- Use `@WebMvcTest` with mocks for unit tests
- Coverage target: ≥ 80% for `auth.jwt` package
- Test both positive (valid token) and negative (expired/invalid/missing) scenarios

---

### Project Structure Notes

- **Repository root:** `I:/portfolio-v2/portfolio-platform/` (separate git root)
- **Module path:** Backend is `portfolio-platform`, NOT `portfolio-v2/portfolio-platform`
- This is BE only — no FE changes in this story

### References

- [Source: epics.md#Story-5.1] — Acceptance criteria and user story
- [Source: architecture.md#Authentication-Security] — JWT RS256, Spring Security config
- [Source: architecture.md#Project-Structure-portfolio-platform] — `auth/jwt/` package structure
- [Source: architecture.md#Enforcement-Summary] — Security patterns, CORS rules
- [Source: 2-3-structured-error-handler-openapi.md] — ErrorResponse, ErrorDetail classes to reuse
- [Source: 2-1-initialize-platform-be-project.md] — Existing SecurityConfig to modify

---

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- RSA key pair generated successfully using OpenSSL 3.5.4
- JJWT library 0.12.5 added to pom.xml for token handling
- Spring Security configured with JWT filter chain
- CORS configured with specific origins (no wildcard)
- All unit tests pass (21 tests for JWT package)

### Completion Notes List

- [x] Task 1: RSA key pair generated and gitignored
- [x] Task 2: JwtConfig with @ConfigurationProperties and application.yml updated
- [x] Task 3: JwtService with RS256 token generation/validation
- [x] Task 4: JwtAuthenticationFilter extends OncePerRequestFilter
- [x] Task 5: JwtAuthenticationEntryPoint returns structured 401 error
- [x] Task 6: SecurityConfig with JWT filter chain and CORS
- [x] Task 7: JwksController at /api/v1/.well-known/jwks.json
- [x] Task 8: Unit tests created and passing (21 tests)

### File List

**New files created:**
```
src/main/java/dev/chinh/portfolio/auth/config/JwtConfig.java
src/main/java/dev/chinh/portfolio/auth/jwt/JwtService.java
src/main/java/dev/chinh/portfolio/auth/jwt/JwtAuthenticationFilter.java
src/main/java/dev/chinh/portfolio/auth/jwt/JwtAuthenticationEntryPoint.java
src/main/java/dev/chinh/portfolio/auth/jwt/JwksController.java
src/main/java/dev/chinh/portfolio/auth/AuthController.java
src/main/resources/keys/private.pem
src/main/resources/keys/public.pem
scripts/generate-keys.sh
src/test/java/dev/chinh/portfolio/auth/jwt/JwtServiceTest.java
src/test/java/dev/chinh/portfolio/auth/jwt/JwtAuthenticationFilterTest.java
```

**Files modified:**
- `src/main/java/dev/chinh/portfolio/shared/config/SecurityConfig.java` — NEW file with JWT filter chain, CORS, entry point
- `src/main/resources/application.yml` — added JWT config section
- `.gitignore` — added `**/keys/*.pem`
- `pom.xml` — added spring-boot-starter-security, jjwt dependencies

**No migrations needed** — auth tables come in Story 5.2/5.4

---

## Change Log

- 2026-03-16 (Code Review Fixes):
  - Added `@EnableMethodSecurity` to SecurityConfig.java for method-level security
  - Fixed JwtAuthenticationEntryPoint to include auth exception message in error response
  - Fixed JwksController to load public key from classpath (works in JAR)
  - Confirmed `scripts/generate-keys.sh` exists

- 2026-03-16: Implemented JWT infrastructure with RS256 asymmetric signing
  - Added Spring Security with JWT filter chain
  - Added JwtService with token generation/validation
  - Added JwtAuthenticationFilter for request authentication
  - Added JwtAuthenticationEntryPoint for 401 errors
  - Added JwksController for JWKS endpoint
  - Added unit tests (21 tests passing)
  - Configured CORS with specific origins (no wildcard)
