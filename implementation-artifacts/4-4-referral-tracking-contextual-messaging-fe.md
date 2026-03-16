# Story 4.4: Referral Tracking & Contextual Messaging (FE)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a recruiter arriving from a specific source,
I want the portfolio to display messaging appropriate to my context,
So that the content feels relevant to where I came from.

## Acceptance Criteria

1. **Given** the page loads with `?from=linkedin`, **Then** the portfolio defaults to English and surfaces professional English-first framing in the hero (FR46)

2. **Given** the page loads with `?from=cv-vn`, **Then** the portfolio defaults to Vietnamese and surfaces Vietnamese-friendly context (FR46)

3. **Given** any `?from=` value is present, **Then** it is persisted to Zustand store via `zustand/middleware` persist — NOT manual `localStorage` calls

4. **Given** `?from=` is absent on a return visit but was previously persisted, **Then** the persisted referral source is used for contextual messaging

5. **Given** localStorage access fails (private browsing), **Then** the app falls back to default behavior gracefully — no uncaught exception, no broken layout

## Tasks / Subtasks

- [x] Task 1 (AC: #3, #4) — Implement Referral Source Persistence
  - [x] Subtask 1.1: Enhance existing referralStore.ts to handle URL param parsing
  - [x] Subtask 1.2: Add initialization logic to read `?from=` from URL on app load
  - [x] Subtask 1.3: Ensure store persists using existing zustand/middleware persist
  - [x] Subtask 1.4: Handle localStorage failure gracefully with try-catch

- [x] Task 2 (AC: #1, #2) — Implement Contextual Language & Messaging
  - [x] Subtask 2.1: Add translation keys for contextual messaging (hero.context.linkedin, hero.context.cv-vn)
  - [x] Subtask 2.2: Modify Hero.tsx to display contextual messaging based on referral source
  - [x] Subtask 2.3: Implement language auto-switching for linkedin (→ en) and cv-vn (→ vi)

- [x] Task 3 (AC: #5) — Add Graceful Fallback
  - [x] Subtask 3.1: Add error boundary or try-catch around store initialization
  - [x] Subtask 3.2: Test with localStorage disabled (private browsing simulation)
  - [x] Subtask 3.3: Verify no console errors on storage failure

- [x] Task 4 (AC: #1-5) — Testing
  - [x] Subtask 4.1: Unit tests for referralStore with URL param parsing
  - [x] Subtask 4.2: Integration test for contextual messaging display
  - [x] Subtask 4.3: Test localStorage failure fallback behavior
  - [x] Subtask 4.4: Accessibility test for contextual messaging (screen reader)

## Dev Notes

### EXHAUSTIVE ARTIFACT ANALYSIS

#### Epic 4 Context
Epic 4 focuses on "Recruiter Contact & Engagement" with 5 stories:
- 4.1: Contact Form & Backend Submission (DONE)
- 4.2: Animated Success Confirmation (DONE)
- 4.3: Rate Limiting & Spam Protection (BE - 3/day/IP) (DONE)
- 4.4: Referral Tracking & Contextual Messaging (FE) ← **THIS STORY**
- 4.5: Returning Visitor Experience (FE - backlog)

#### Previous Story Intelligence (Story 4.3)
From `implementation-artifacts/4-3-rate-limiting-spam-protection-be.md`:
- ContactController.java already extracts client IP at line 56: `String clientIp = getClientIp(httpRequest);`
- ContactForm.tsx handles various response statuses
- Frontend API layer already handles error responses gracefully
- **This story is FE-only; no BE changes required**

#### Previous Story Intelligence (Story 4.1)
From `implementation-artifacts/4-1-contact-form-backend-submission-fe-be.md`:
- Contact section exists at `portfolio-fe/src/components/sections/Contact.tsx`
- ContactForm.tsx is at `portfolio-fe/src/components/shared/ContactForm.tsx`

#### Architecture Requirements
From `planning-artifacts/architecture.md`:
- **FR46:** Contextual messaging based on referral source
- **i18n:** Smart `?from=`-based default detection per FR41-FR42
- **State Management:** localStorage state machine via zustand/middleware persist

### Technical Requirements

#### Frontend Stack (portfolio-fe)
- React 19+ with TypeScript
- Zustand with persist middleware (ALREADY EXISTS - see below)
- Existing Hero.tsx component
- react-i18next for translations

#### EXISTING STORES - DO NOT REINVENT

**1. referralStore.ts ALREADY EXISTS** (portfolio-fe/src/stores/referralStore.ts):
```typescript
interface ReferralStore {
  referralSource: string | null
  setReferralSource: (source: string) => void
}
export const useReferralStore = create<ReferralStore>()(
  persist(
    (set) => ({
      referralSource: null,
      setReferralSource: (source) => set({ referralSource: source }),
    }),
    { name: 'referral-source' }
  )
)
```
**THIS STORE ALREADY USES zustand/middleware persist - DO NOT REWRITE**

**2. languageStore.ts EXISTS** (portfolio-fe/src/stores/languageStore.ts):
```typescript
type Language = 'vi' | 'en'
interface LanguageStore {
  language: Language
  setLanguage: (language: Language) => void
}
export const useLanguageStore = create<LanguageStore>()(
  persist(
    (set) => ({
      language: 'en',
      setLanguage: (language) => set({ language }),
    }),
    { name: 'language-preference' }
  )
)
```
**LANGUAGE STORE ALREADY HAS persist - DO NOT REWRITE**

### File Structure (MUST FOLLOW)

**Frontend (portfolio-fe/src/):**
```
stores/
├── referralStore.ts            ← EXISTS (MODIFY - add URL param parsing on init)
├── languageStore.ts            ← EXISTS (MODIFY - add auto-switch based on referral)
├── visitorStore.ts             ← EXISTS (Story 4.5 will use)
├── themeStore.ts               ← EXISTS
├── galleryStore.ts             ← EXISTS
└── metricsStore.ts             ← EXISTS

components/
└── sections/
    └── Hero.tsx                ← EXISTS (MODIFY - add contextual messaging)

i18n/
└── (translation files)        ← EXISTS (ADD - new translation keys)
```

### Implementation Approach

**Step 1: Enhance referralStore.ts**
- Add initialization logic that reads `?from=` from URL search params
- Add try-catch for localStorage failure
- The store should auto-initialize on app load

**Step 2: Enhance languageStore.ts**
- Add logic to auto-set language based on referral source:
  - `?from=linkedin` → language = 'en'
  - `?from=cv-vn` → language = 'vi'
- Should only auto-set if user hasn't manually set language before

**Step 3: Modify Hero.tsx**
- Add contextual messaging display based on `useReferralStore().referralSource`
- Add translation keys for contextual hero content
- Ensure messaging is accessible (aria-live or similar)

**Step 4: Test localStorage failure**
- Wrap store initialization in try-catch
- Verify app works when localStorage is disabled

### Architecture Compliance

1. **MUST use zustand/middleware persist** - Already implemented in stores, do NOT use manual localStorage
2. **Error Handling** - Graceful fallback when localStorage fails (private browsing)
3. **i18n Integration** - Use react-i18next for all contextual text
4. **Accessibility** - Contextual messaging must be accessible

### Testing Requirements

1. **Unit Tests:**
   - referralStore URL param parsing
   - language auto-switch logic
   - localStorage failure handling

2. **Integration Tests:**
   - Hero displays correct contextual message for each referral source
   - Language switches correctly on page load with `?from=`

3. **Accessibility Tests:**
   - Screen reader announces contextual messaging
   - Keyboard navigation works

### References

- [Source: planning-artifacts/epics.md#Story-4.4]
- [Source: planning-artifacts/architecture.md#Recruiter-Engagement]
- [Source: implementation-artifacts/4-3-rate-limiting-spam-protection-be.md]
- [Source: implementation-artifacts/4-1-contact-form-backend-submission-fe-be.md]
- [Source: portfolio-fe/src/stores/referralStore.ts] ← EXISTS
- [Source: portfolio-fe/src/stores/languageStore.ts] ← EXISTS
- [Source: portfolio-fe/src/components/sections/Hero.tsx] ← EXISTS

## Dev Agent Record

### Agent Model Used
Claude Opus 4.6 (2026-03-15)

### Debug Log References

### Completion Notes List

- Implemented referralStore.ts with URL param parsing, hydration tracking, and safe storage wrapper
- Implemented languageStore.ts with auto-switch based on referral source and safe storage wrapper
- Added contextual translation keys for hero.context.linkedin and hero.context.cv-vn in both en.json and vi.json
- Modified Hero.tsx to display contextual messaging based on referralSource from store
- Added aria-live attributes for accessibility when contextual messaging is displayed
- Created comprehensive unit tests for referralStore (URL parsing, hydration, error handling)
- Created comprehensive unit tests for languageStore (auto-switch logic, hydration)
- Created integration tests for Hero contextual messaging display

### Code Review Fixes Applied (2026-03-15)

- Added localStorage failure handling tests to referralStore.test.ts (3 new tests)
- Added localStorage failure handling tests to languageStore.test.ts (3 new tests)
- Added autoSetFromReferral unit tests to languageStore.test.ts (6 tests - already existed, verified)
- All 301 tests pass, 0 lint errors

### File List

**Frontend (portfolio-fe):**
- `src/stores/referralStore.ts` — MODIFIED (add URL param parsing, error handling)
- `src/stores/languageStore.ts` — MODIFIED (add auto-switch based on referral)
- `src/components/sections/Hero.tsx` — MODIFIED (add contextual messaging)
- `src/i18n/en.json` — MODIFIED (add hero.context.linkedin)
- `src/i18n/vi.json` — MODIFIED (add hero.context.cv-vn)
- `src/stores/referralStore.test.ts` — MODIFIED (add unit tests)
- `src/stores/languageStore.test.ts` — MODIFIED (add auto-switch tests)
- `src/components/sections/Hero.test.tsx` — MODIFIED (add contextual tests)

## Change Log

- 2026-03-15: Implemented referral source persistence with URL param parsing and safe storage wrapper (Task 1)
- 2026-03-15: Implemented contextual language auto-switching and messaging (Task 2)
- 2026-03-15: Added graceful localStorage fallback with try-catch wrappers (Task 3)
- 2026-03-15: Added comprehensive unit and integration tests (Task 4)
- 2026-03-15: All 295 tests pass, 0 lint errors
- 2026-03-15: Code review fixes - added localStorage failure handling tests (6 new tests)

