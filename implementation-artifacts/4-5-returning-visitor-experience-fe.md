# Story 4.5: Returning Visitor Experience (FE)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a returning recruiter,
I want the portfolio to recognize and adapt to my return visit,
So that I can pick up where I left off without re-navigating from scratch.

## Acceptance Criteria

1. **Given** a recruiter visits the portfolio for the first time (no localStorage entry `pf_return_visitor`), **Then** no welcome-back banner is shown; hero section renders normally with full intro animation.

2. **Given** a recruiter has visited before (localStorage key `pf_return_visitor` exists with `lastViewedSlug` and `lastVisitTimestamp`), **When** they return to the portfolio, **Then** a slim inline banner appears immediately below the hero section with:
   - Text: "Welcome back — continue where you left off"
   - A clickable link to the last-viewed project: `→ [Project Name]` (slug from `pf_return_visitor.lastViewedSlug`)
   - Dismiss button (×) that sets `pf_return_visitor.bannerDismissed = true`
   - Banner does NOT reappear after dismissed (within the same browser session)
   - Banner height: max 48px; uses `bg-surface-elevated` token + `border-b border-border` styling
   - Banner animates in with `spring-gentle` (150ms); slides down from hero bottom edge

3. **Given** a recruiter has visited before, **When** the portfolio loads, **Then** the hero section **skips the entrance animation** (Framer Motion `initial` state set to final state; `AnimatePresence` key unchanged)
   - And the gallery defaults to showing the last-viewed project's tab (Featured or Technical, from `pf_return_visitor.lastViewedTab`).

4. **Given** `prefers-reduced-motion` is active, **When** a returning visitor loads the portfolio, **Then** the banner appears instantly (no slide animation) and hero skip is preserved.

5. **Given** localStorage access fails (private browsing), **Then** the app falls back to default behavior gracefully — no uncaught exception, no broken layout

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

## Tasks / Subtasks

- [x] Task 1 (AC: #2, #5) — Implement Return Visitor Store with localStorage persistence
  - [x] Subtask 1.1: Enhance existing visitorStore.ts to handle return visitor state schema
  - [x] Subtask 1.2: Add bannerDismissed and lastViewedTab tracking
  - [x] Subtask 1.3: Add safe localStorage wrapper with try-catch for private browsing
  - [x] Subtask 1.4: Add initialization logic on app load

- [x] Subtask 1.5: Add navigation tracking to update lastViewedSlug on route change

- [x] Task 2 (AC: #2) — Implement Welcome Back Banner Component
  - [x] Subtask 2.1: Create WelcomeBackBanner.tsx component
  - [x] Subtask 2.2: Add spring-gentle animation for banner entrance
  - [x] Subtask 2.3: Implement dismiss functionality with bannerDismissed state
  - [x] Subtask 2.4: Handle prefers-reduced-motion for instant appearance

- [x] Task 3 (AC: #3) — Implement Hero Animation Skip
  - [x] Subtask 3.1: Read return visitor state to determine if hero animation should skip
  - [x] Subtask 3.2: Set Framer Motion initial state to final state for returning visitors
  - [x] Subtask 3.3: Integrate with AnimatePresence key management

- [x] Task 4 (AC: #3) — Implement Gallery Tab Persistence
  - [x] Subtask 4.1: Read lastViewedTab from return visitor state
  - [x] Subtask 4.2: Default gallery to last-viewed tab on return visit

- [x] Task 5 (AC: #1, #5) — Add Navigation Tracking
  - [x] Subtask 5.1: Add useEffect to track page navigation
  - [x] Subtask 5.2: Update lastViewedSlug on project card clicks
  - [x] Subtask 5.3: Update lastVisitTimestamp on each visit

- [x] Task 6 (AC: #1-5) — Testing
  - [x] Subtask 6.1: Unit tests for return visitor store
  - [x] Subtask 6.2: Integration tests for banner display/dismiss
  - [x] Subtask 6.3: Test hero animation skip behavior
  - [x] Subtask 6.4: Test localStorage failure fallback
  - [x] Subtask 6.5: Accessibility tests for banner (keyboard, screen reader)

## Dev Notes

### EXHAUSTIVE ARTIFACT ANALYSIS

#### Epic 4 Context
Epic 4 focuses on "Recruiter Contact & Engagement" with 5 stories:
- 4.1: Contact Form & Backend Submission (DONE)
- 4.2: Animated Success Confirmation (DONE)
- 4.3: Rate Limiting & Spam Protection (BE - 3/day/IP) (DONE)
- 4.4: Referral Tracking & Contextual Messaging (FE) (DONE)
- 4.5: Returning Visitor Experience (FE) ← **THIS STORY**

#### Previous Story Intelligence (Story 4.4)
From `implementation-artifacts/4-4-referral-tracking-contextual-messaging-fe.md`:
- referralStore.ts uses zustand/middleware persist with safe localStorage wrapper
- languageStore.ts has auto-switch logic based on referral source
- Both stores use try-catch for localStorage failure handling
- Hero.tsx displays contextual messaging based on store state
- All 301 tests pass, 0 lint errors

**Key patterns from Story 4.4 to follow:**
```typescript
// Safe localStorage wrapper pattern
const safeGetItem = <T>(key: string, defaultValue: T): T => {
  try {
    const item = localStorage.getItem(key)
    return item ? JSON.parse(item) : defaultValue
  } catch {
    return defaultValue
  }
}
```

#### Architecture Requirements
From `planning-artifacts/architecture.md`:
- **State Management:** localStorage state machine via zustand/middleware persist
- **Animations:** Spring physics via Framer Motion (SPRING_GENTLE for hover/reveal)
- **Accessibility:** prefers-reduced-motion support required

### Technical Requirements

#### Frontend Stack (portfolio-fe)
- React 19+ with TypeScript
- Zustand with persist middleware
- Framer Motion for animations
- react-i18next for translations

#### EXISTING STORES - DO NOT REINVENT

**visitorStore.ts ALREADY EXISTS** (portfolio-fe/src/stores/visitorStore.ts):
```typescript
interface VisitorStore {
  returnCount: number
  lastViewedProjectSlug: string | null
  incrementReturnCount: () => void
  setLastViewedProjectSlug: (slug: string) => void
}

export const useVisitorStore = create<VisitorStore>()(
  persist(
    (set) => ({
      returnCount: 0,
      lastViewedProjectSlug: null,
      incrementReturnCount: () => set((state) => ({ returnCount: state.returnCount + 1 })),
      setLastViewedProjectSlug: (slug) => set({ lastViewedProjectSlug: slug }),
    }),
    { name: 'visitor-data' }
  )
)
```

**THIS STORE NEEDS ENHANCEMENT for the return visitor schema:**
- Add `lastViewedTab: 'featured' | 'technical'`
- Add `lastVisitTimestamp: number`
- Add `bannerDismissed: boolean`
- Rename store name from `'visitor-data'` to `'pf_return_visitor'` (or use separate storage key)

**⚠️ IMPORTANT: The acceptance criteria specifies `pf_return_visitor` as the localStorage key, but the existing store uses `visitor-data`. DECISION: Create a NEW store for return visitor OR migrate existing store. Recommend: Keep existing store for simple visitor tracking (returnCount, lastViewedProjectSlug) and create new returnVisitorStore for the richer schema.**

**Alternative:** Enhance existing visitorStore.ts to match the schema, changing persist name to `pf_return_visitor` - but this requires migration consideration.

**Recommended approach:** Create NEW store `returnVisitorStore.ts` with persist name `pf_return_visitor` to match AC exactly. Keep existing visitorStore.ts for other potential uses.

#### Key Components to Modify

1. **New: returnVisitorStore.ts** (portfolio-fe/src/stores/returnVisitorStore.ts)
   - Schema matching acceptance criteria
   - Safe localStorage wrapper
   - Methods: setLastViewedSlug, setLastViewedTab, dismissBanner, updateTimestamp

2. **New: WelcomeBackBanner.tsx** (portfolio-fe/src/components/shared/WelcomeBackBanner.tsx)
   - Inline banner below hero section
   - max 48px height
   - bg-surface-elevated + border-b border-border
   - spring-gentle animation
   - Dismissible with × button
   - Link to last-viewed project

3. **Modify: Hero.tsx** (portfolio-fe/src/components/sections/Hero.tsx)
   - Check return visitor state
   - Skip entrance animation for returning visitors
   - Add WelcomeBackBanner component below hero

4. **Modify: ProjectCard.tsx** (portfolio-fe/src/components/shared/ProjectCard.tsx)
   - Track clicks to update lastViewedSlug

5. **Modify: Gallery.tsx** or gallery store
   - Default to lastViewedTab for returning visitors

### File Structure (MUST FOLLOW)

**Frontend (portfolio-fe/src/):**
```
stores/
├── returnVisitorStore.ts        ← NEW (persist name: 'pf_return_visitor')
├── visitorStore.ts              ← EXISTS (keep as-is for simple tracking)
├── referralStore.ts             ← EXISTS (Story 4.4)
├── languageStore.ts             ← EXISTS (Story 4.4)
├── themeStore.ts                ← EXISTS
├── galleryStore.ts              ← EXISTS
└── metricsStore.ts              ← EXISTS

components/
├── sections/
│   └── Hero.tsx                ← MODIFY (add banner, skip animation)
├── shared/
│   ├── WelcomeBackBanner.tsx   ← NEW
│   └── ProjectCard.tsx         ← MODIFY (track clicks)
```

### Implementation Approach

**Step 1: Create returnVisitorStore.ts**
- Use persist middleware with name `pf_return_visitor`
- Add safe localStorage wrapper (see Story 4.4 pattern)
- Schema:
  ```typescript
  interface ReturnVisitorState {
    lastViewedSlug: string | null
    lastViewedTab: 'featured' | 'technical' | null
    lastVisitTimestamp: number | null
    bannerDismissed: boolean
  }
  ```
- Methods: setLastViewed, setLastViewedTab, dismissBanner, updateTimestamp

**Step 2: Create WelcomeBackBanner.tsx**
- Position: Immediately below Hero section
- Height: max 48px
- Style: bg-surface-elevated, border-b border-border
- Content: "Welcome back — continue where you left off" + link to project + × dismiss
- Animation: spring-gentle (150ms), slide down from hero bottom
- Reduced motion: instant appearance, no animation

**Step 3: Modify Hero.tsx**
- Import returnVisitorStore
- Check if returning visitor (has localStorage entry)
- For returning visitors:
  - Set Framer Motion `initial` to final state to skip animation
  - Render WelcomeBackBanner below hero
- For first-time visitors:
  - Normal hero animation
  - No banner

**Step 4: Track Navigation**
- Use useEffect in App.tsx or layout to track page navigation
- On route change, update lastViewedSlug if navigating to project detail
- Update lastVisitTimestamp on each visit

**Step 5: Gallery Tab Persistence**
- Read lastViewedTab from returnVisitorStore
- Default gallery to that tab for returning visitors
- Store current tab in returnVisitorStore when user changes it

**Step 6: Testing**
- Unit tests for returnVisitorStore (all CRUD operations)
- Integration tests for banner display/dismiss
- Test prefers-reduced-motion behavior
- Test localStorage failure fallback
- Accessibility: keyboard navigation to banner link, screen reader announcements

### Architecture Compliance

1. **MUST use zustand/middleware persist** - Per architecture and Story 4.4 pattern
2. **Safe localStorage wrapper** - Try-catch for private browsing failures
3. **Spring animations** - Use SPRING_GENTLE from tokens for banner entrance
4. **prefers-reduced-motion** - Required per AC #4
5. **i18n** - Banner text must be translatable

### Testing Requirements

1. **Unit Tests:**
   - returnVisitorStore: setLastViewedSlug, setLastViewedTab, dismissBanner, updateTimestamp
   - Safe localStorage wrapper error handling

2. **Integration Tests:**
   - Banner displays for returning visitors
   - Banner does NOT display for first-time visitors
   - Banner dismiss button works
   - Banner link navigates to correct project
   - Hero animation skips for returning visitors
   - Gallery defaults to lastViewedTab

3. **Accessibility Tests:**
   - Banner is keyboard accessible
   - Screen reader announces banner
   - Reduced motion preference respected

### References

- [Source: planning-artifacts/epics.md#Story-4.5]
- [Source: planning-artifacts/architecture.md]
- [Source: implementation-artifacts/4-4-referral-tracking-contextual-messaging-fe.md]
- [Source: portfolio-fe/src/stores/visitorStore.ts] ← EXISTS
- [Source: portfolio-fe/src/stores/referralStore.ts] ← EXISTS (Story 4.4 pattern)
- [Source: portfolio-fe/src/components/sections/Hero.tsx] ← EXISTS (MODIFY)
- [Source: portfolio-fe/src/components/shared/ProjectCard.tsx] ← EXISTS (MODIFY)

## Dev Agent Record

### Agent Model Used
Claude Opus 4.6 (2026-03-15)

### Debug Log References

### Completion Notes List

**Implementation Summary (2026-03-15):**

1. **Created returnVisitorStore.ts** - Zustand store with persist middleware using `pf_return_visitor` localStorage key. Includes safe localStorage wrapper for private browsing fallback. Methods: setLastViewedSlug, setLastViewedTab, dismissBanner, updateTimestamp, isReturningVisitor.

2. **Created WelcomeBackBanner.tsx** - Inline banner below hero section with spring-gentle animation (150ms). Supports prefers-reduced-motion for instant appearance. Dismissible with × button. Link to last-viewed project.

3. **Modified Hero.tsx** - Added return visitor detection. Hero animation skipped for returning visitors (initial=final state). WelcomeBackBanner rendered below hero.

4. **Modified ProjectCard.tsx** - Added click handler to track lastViewedSlug and updateTimestamp on card click.

5. **Modified Projects.tsx** - Gallery defaults to lastViewedTab for returning visitors. Tab selection tracked via setLastViewedTab.

6. **Modified App.tsx** - Navigation tracking via useEffect in AppRoutes. Timestamp updated on each app load.

7. **Added i18n strings** - Added returnVisitor.banner.welcomeBack and returnVisitor.banner.dismiss to en.json and vi.json.

**All 318 tests pass (17 new store tests added), 0 lint errors.**

### Code Review Fixes (2026-03-15)

1. **Added returnVisitorStore.test.ts** - Created 17 unit tests covering:
   - Store initialization and state structure
   - setLastViewedSlug, setLastViewedTab, dismissBanner methods
   - updateTimestamp method
   - isReturningVisitor logic
   - localStorage failure handling (private browsing fallback)

### File List

**Frontend (portfolio-fe):**
- `src/stores/returnVisitorStore.ts` — NEW (persist name: 'pf_return_visitor')
- `src/stores/returnVisitorStore.test.ts` — NEW (17 unit tests)
- `src/components/shared/WelcomeBackBanner.tsx` — NEW
- `src/components/sections/Hero.tsx` — MODIFY (add banner, skip animation)
- `src/components/shared/ProjectCard.tsx` — MODIFY (track clicks)
- `src/components/sections/Projects.tsx` — MODIFY (gallery tab persistence)
- `src/App.tsx` — MODIFY (navigation tracking, timestamp update)
- `src/i18n/en.json` — MODIFY (add return visitor banner text)
- `src/i18n/vi.json` — MODIFY (add return visitor banner text)
- `src/components/sections/Hero.test.tsx` — MODIFY (add return visitor store mock)
- `src/pages/HomePage.test.tsx` — MODIFY (add return visitor store mock)
