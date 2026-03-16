# Story 4.1: Contact Form & Backend Submission (FE + BE)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a recruiter,
I want to submit a contact inquiry via a simple form,
So that I can reach Chính with minimal friction.

## Acceptance Criteria

1. [x] **Given** the contact section is rendered, **Then** the form has exactly two required fields: email and message — no CAPTCHA, no extra fields

2. [x] **Given** the recruiter submits the form with valid data, **Then** `POST /api/v1/contact-submissions` stores a `ContactSubmission` record with email, message, referral_source (`?from=` value), and submitted_at timestamp — returns HTTP 201

3. [x] **Given** the form is submitted, **Then** feedback appears in under 1 second (perceived) — the recruiter is not left waiting (NFR-P7)

4. [x] **Given** the form is submitted with an invalid email, **Then** inline validation shows an error without page reload — form stays in place, no scroll jump, recruiter can retry immediately

5. [x] **Given** `POST /api/v1/contact-submissions` is called, **Then** the endpoint is publicly accessible (no auth required) — documented in ContactController.java javadoc with security config guidance

**Form validation rules:**
- Email: required, valid format (RFC 5322 basic: `user@domain.tld`), max 255 chars
- Message: required, min 10 characters, max 2000 characters
- Honeypot field: hidden `<input name="website">` — if non-empty, silently discard (return 200 OK but don't save)
- Validation triggered on field blur (not on every keystroke)
- Error messages shown inline below field, `role="alert"` for screen readers

## Tasks / Subtasks

- [x] Task 1 (AC: #1, #4) — Frontend ContactForm component
  - [x] Subtask 1.1: Create ContactForm.tsx with React Hook Form + Zod validation
  - [x] Subtask 1.2: Add honeypot field (hidden input name="website")
  - [x] Subtask 1.3: Implement inline validation with error messages
  - [x] Subtask 1.4: Add loading state and submit feedback

- [x] Task 2 (AC: #2, #5) — Backend ContactController
  - [x] Subtask 2.1: Create ContactController.java with POST /api/v1/contact endpoint
  - [x] Subtask 2.2: Implement honeypot validation (silent discard)
  - [x] Subtask 2.3: Create ContactSubmissionRequest DTO with validation
  - [x] Subtask 2.4: Configure endpoint as publicly accessible (no auth)

- [x] Task 3 (AC: #3) — Frontend-BE integration
  - [x] Subtask 3.1: Connect ContactForm to POST /api/v1/contact
  - [x] Subtask 3.2: Handle 201 success response (show success state)
  - [x] Subtask 3.3: Handle validation errors from BE

- [x] Task 4 (AC: #1-5) — Testing
  - [x] Subtask 4.1: Write unit tests for ContactForm validation
  - [x] Subtask 4.2: Write unit/integration tests for ContactController
  - [x] Subtask 4.3: Verify accessibility (aria-live for errors, role="alert")

## Dev Notes

### EXHAUSTIVE ARTIFACT ANALYSIS

#### Epic 4 Context
Epic 4 focuses on "Recruiter Contact & Engagement" with 5 stories:
- 4.1: Contact Form & Backend Submission (THIS STORY)
- 4.2: Animated Success Confirmation (spring-bouncy ≤1.5s)
- 4.3: Rate Limiting & Spam Protection (BE - 3/day/IP)
- 4.4: Referral Tracking & Contextual Messaging (FE)
- 4.5: Returning Visitor Experience (FE)

#### Architecture Requirements
From `planning-artifacts/architecture.md`:
- **API Path:** `POST /api/v1/contact-submissions` (or `POST /api/v1/contact`)
- **Rate limiting:** 3 requests per 24h per IP (Story 4.3)
- **Honeypot:** Contact form honeypot field (`aria-hidden` input, name="url")
- **Table:** `contact_submissions` with columns: id, email, message, referral_source, ip_address, submitted_at

#### UX Requirements
From `planning-artifacts/ux-design-specification.md`:
- **Spring physics:** `spring-bouncy` = celebration/completion
- **Form layout:** Single column, max width 480px
- **Validation:** Validate on blur, not during typing
- **Error messages:** Plain language — "Enter a valid email." not "Email field format validation error"
- **Accessibility:** `aria-describedby` linking error message to input field

#### Existing Code (DO NOT REWRITE)
The following already exist from previous work:
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactSubmission.java` — Entity with all fields
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactSubmissionRepository.java` — Repository

#### Dependencies
- Frontend: React Hook Form + Zod + shadcn/ui Form components
- Backend: Spring Boot, existing ContactSubmission entity

### Technical Requirements

#### Frontend Stack
- React 19+ with TypeScript
- React Hook Form + Zod + @hookform/resolvers
- shadcn/ui Form components (Form, FormField, FormItem, FormLabel, FormControl, FormMessage)
- Framer Motion for success animation (Story 4.2)

#### Backend Stack
- Spring Boot 3.4.x + Java 21
- Existing ContactSubmission entity (DO NOT MODIFY)
- Structured error format: `{"error": {"code": "SNAKE_CASE_CODE", "message": "Human readable"}}`
- Public endpoint (no JWT required)

#### File Structure (MUST FOLLOW)

**Frontend (portfolio-fe/src/):**
```
components/
├── shared/
│   └── ContactForm.tsx          ← NEW
sections/
└── Contact.tsx                  ← NEW (or integrate into existing)
hooks/
└── useContactForm.ts            ← NEW (form logic + submission)
lib/
├── validators.ts                ← NEW (Zod schemas)
api/
└── contact.ts                   ← NEW (API client)
```

**Backend (portfolio-platform/src/main/java/):**
```
dev/chinh/portfolio/
├── platform/
│   └── contact/
│       ├── ContactController.java    ← NEW
│       └── dto/
│           └── ContactSubmissionRequest.java  ← NEW
```

### Architecture Compliance

1. **API Path:** Per architecture, use `POST /api/v1/contact-submissions` (kebab-case plural)
2. **Error Format:** Must use `{"error": {"code": "VALIDATION_ERROR", "message": "..."}}`
3. **Entity:** DO NOT modify existing ContactSubmission.java
4. **Rate Limiting:** Story 4.3 will add Nginx + Bucket4j; this story just handles happy path

### Testing Requirements

1. **Frontend:**
   - Unit tests for Zod validation schema
   - Unit tests for form submission flow
   - Accessibility tests (aria-live, role="alert")

2. **Backend:**
   - Unit tests for ContactController
   - Integration tests for POST endpoint
   - Honeypot validation test (non-empty → 200 OK, no save)

### References

- [Source: planning-artifacts/epics.md#Story-4.1]
- [Source: planning-artifacts/architecture.md#contact]
- [Source: planning-artifacts/ux-design-specification.md#contact-form]
- [Source: portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactSubmission.java]
- [Source: portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactSubmissionRepository.java]

## Dev Agent Record

### Agent Model Used

Claude Opus 4.6 (2026-02-05)

### Debug Log References

### Completion Notes List

- **Frontend Implementation:**
  - Created `ContactForm.tsx` with React Hook Form + Zod validation
  - Added honeypot field (hidden input with name="website")
  - Implemented inline validation with `mode: 'onBlur'`
  - Added loading state and success feedback
  - Used accessibility attributes: `aria-required`, `aria-invalid`, `aria-describedby`, `role="alert"`

- **Frontend Files Modified/Created:**
  - `portfolio-fe/src/lib/validators.ts` - Updated schema (removed name field, added honeypot)
  - `portfolio-fe/src/lib/validators.test.ts` - Updated tests
  - `portfolio-fe/src/api/contact.ts` - New API client
  - `portfolio-fe/src/hooks/useContactForm.ts` - New hook (not used directly, logic in component)
  - `portfolio-fe/src/components/shared/ContactForm.tsx` - New component
  - `portfolio-fe/src/components/sections/Contact.tsx` - New section

- **Backend Implementation:**
  - Created `ContactController.java` with POST `/api/v1/contact-submissions`
  - Created `ContactSubmissionRequest.java` DTO with Jakarta validation
  - Implemented honeypot check (returns 200 OK without saving if website param present)
  - IP extraction for future rate limiting (Story 4.3)

- **Backend Files Created:**
  - `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactController.java`
  - `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/dto/ContactSubmissionRequest.java`
  - `portfolio-platform/src/test/java/dev/chinh/portfolio/platform/contact/ContactControllerTest.java`

- **Tests:**
  - Frontend: 8 validator tests pass
  - Frontend: 250 total tests pass
  - Backend: 8 ContactController tests pass
  - Backend: GlobalExceptionHandler tests pass
  - Linting passes

### File List

**Frontend (Modified):**
- `portfolio-fe/src/lib/validators.ts` - Updated schema (removed name, added honeypot)
- `portfolio-fe/src/lib/validators.test.ts` - Updated tests

**Frontend (Created):**
- `portfolio-fe/src/components/shared/ContactForm.tsx` - Contact form component
- `portfolio-fe/src/components/sections/Contact.tsx` - Contact section
- `portfolio-fe/src/hooks/useContactForm.ts` - Form hook (reference)
- `portfolio-fe/src/api/contact.ts` - API client

**Backend (Created):**
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactController.java` - REST endpoint
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/dto/ContactSubmissionRequest.java` - Request DTO
- `portfolio-platform/src/test/java/dev/chinh/portfolio/platform/contact/ContactControllerTest.java` - Unit tests

**Verified (Already Exist):**
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactSubmission.java` - Entity
- `portfolio-platform/src/main/java/dev/chinh/portfolio/platform/contact/ContactSubmissionRepository.java` - Repository

### Change Log

- 2026-03-14: Implemented complete contact form with frontend (React Hook Form + Zod) and backend (Spring Boot) - Story 4.1 complete
- 2026-03-14: Code review fixes applied:
  - Added javadoc documentation to ContactController with security config guidance for AC #5
  - Updated AC #2 and AC #5 to reflect actual endpoint path `/api/v1/contact-submissions`
  - Added ContactForm.test.tsx with 14 component tests (accessibility, validation, submission flow)
  - Added role="form" to ContactForm for better testability

