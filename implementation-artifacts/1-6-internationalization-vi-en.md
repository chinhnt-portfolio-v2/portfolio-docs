# Story 1.6: Internationalization (vi/en)

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a recruiter,
I want to switch the portfolio between Vietnamese and English,
so that I can read personal content in my preferred language.

## Acceptance Criteria

**AC1: Language Toggle Switches Content Instantly**
```
Given the recruiter clicks the language toggle in the nav bar,
When the toggle fires,
Then all translated UI labels and personal content (Hero, About, Projects, nav links)
  switch between vi and en instantly — no page reload.
```

**AC2: Language Persists Across Sessions**
```
Given the language is set to Vietnamese and the browser is closed,
When the recruiter returns,
Then Vietnamese is restored via zustand/middleware persist
  (Zustand key: 'language-preference').
```

**AC3: `?from=linkedin` Defaults to English**
```
Given the page loads with `?from=linkedin` in the URL,
AND no `?lang=` param is present,
AND no prior language preference is stored in localStorage,
Then English is the default language for that session.
```

**AC4: `?from=cv-vn` Defaults to Vietnamese**
```
Given the page loads with `?from=cv-vn` in the URL,
AND no `?lang=` param is present,
AND no prior language preference is stored in localStorage,
Then Vietnamese is the default language for that session.
```

**AC5: `?lang=` Param Overrides All Other Sources**
```
Given `?lang=vi` or `?lang=en` is in the URL,
Then it overrides both the ?from= default AND any stored preference.
The override language is also stored to localStorage immediately.
```

**AC6: Browser Language Fallback**
```
Given no URL params exist,
AND no preference is stored in localStorage,
Then browser language (navigator.language) is used as fallback:
  vi-* → 'vi', all others → 'en'.
```

**AC7: Technical Content Stays English**
```
Given technical content is displayed (code snippets, metrics,
  build timelines, GitHub data, tech badge labels, tag chips),
Then it stays in English regardless of selected language.
```

**AC8: HTML lang Attribute Updated**
```
Given the language is switched,
Then document.documentElement.lang is updated to 'vi' or 'en' accordingly.
```

**AC9: Language Toggle Accessibility**
```
Given the LanguageToggle button in the nav,
Then it has aria-label="Switch language",
  displays the active language prominently (EN or VI underlined/active),
  and is reachable via keyboard Tab with visible focus ring.
```

**AC10: No Regressions & Full Build**
```
Given Story 1.6 implementation is complete,
When validation runs,
Then:
  - All existing 157 tests from Stories 1.1–1.5 still pass
  - New tests written for: referral language mapping, language toggle click,
    i18n detection logic, and translated component rendering
  - pnpm run lint: 0 errors
  - pnpm run build: succeeds (Vite + tsc both pass)
```

## Tasks / Subtasks

- [x] **Task 1: Extend `lib/referral.ts` — Add language mapping utilities** (AC: 3, 4, 5, 6)
  - [x] Add `getLanguageFromReferral(source: string | null): 'en' | 'vi' | null`
  - [x] Add `getLangFromUrlParam(search: string): 'en' | 'vi' | null`
  - [x] Add `detectInitialLanguage(search: string): 'en' | 'vi'` with full priority chain
  - [x] Update `referral.test.ts` — 18 new tests (23 total), all pass

- [x] **Task 2: Update `i18n/index.ts` — Remove hardcoded language, fix export** (AC: 1, 2)
  - [x] Removed `lng: 'en'` hardcoded default
  - [x] Added `initImmediate: false`
  - [x] Fixed export: `export default i18n` (was pointing to wrong variable)

- [x] **Task 3: Update `App.tsx` — Wire i18n initialization & language sync** (AC: 1, 2, 5, 8)
  - [x] Side-effect import of `@/i18n`
  - [x] `detectInitialLanguage` on mount → `setLanguage` + `i18n.changeLanguage` + `document.documentElement.lang`
  - [x] `useLanguageStore.subscribe()` in useEffect for ongoing sync + cleanup

- [x] **Task 4: Populate translation files (`en.json`, `vi.json`)** (AC: 1, 7)
  - [x] `en.json` — full structure: nav, hero, projects, about, projectDetail
  - [x] `vi.json` — complete Vietnamese translations for all keys
  - [x] Technical content stays English (tech tags, URLs, metrics)

- [x] **Task 5: Fix `Nav.tsx` — Wire LanguageToggle onClick** (AC: 1, 9)
  - [x] Added `setLanguage` + `onClick` handler (toggles en↔vi)
  - [x] `aria-label="Switch language"` per AC9
  - [x] Active language highlighted (font-semibold text-foreground), inactive muted
  - [x] `useTranslation()` for nav link labels via `NAV_LINK_DEFS` + `t('nav.${key}')`

- [x] **Task 6: Update `Hero.tsx` — Add i18n** (AC: 1, 7)
  - [x] `useTranslation()` added
  - [x] All UI strings use `t()`: eyebrow, heading prefix+highlight, tagline, CTA buttons

- [x] **Task 7: Update `About.tsx` — Add i18n** (AC: 1, 7)
  - [x] Add `useTranslation()` hook
  - [x] Replace heading with `t('about.heading')` ("Why Hire Me" / "Tại sao chọn tôi")
  - [x] Replace `aboutContent.whyHireMe` with `t('about.whyHireMe')`
  - [x] Replace "Verified Skills" heading with `t('about.skills.heading')`
  - [x] Replace "Availability" heading with `t('about.availability.heading')`
  - [x] Replaced `STATUS_LABEL[availability.status]` with `t(\`about.availability.${availability.status}\`)` — removed STATUS_LABEL import from About.tsx
  - [x] Replace location string `availability.location` with `t('about.availability.location')`

- [x] **Task 8: Update `Projects.tsx` — Add i18n** (AC: 1, 7)
  - [x] `useTranslation()` added
  - [x] Heading: `t('projects.heading')`, filter labels: `t('projects.filter.all')`, `t('projects.filter.featured')`
  - [x] Empty state: `t('projects.emptyState.message')`, `t('projects.emptyState.reset')`
  - [x] Tech tag filter chips stay untranslated (English-only)

- [x] **Task 9: Update `ProjectDetailPage.tsx` — Add i18n** (AC: 1, 7)
  - [x] `useTranslation()` added
  - [x] Translated: not-found message, back link, view live/github links, "Live soon" badge, build timeline heading, "What I'd do differently" heading
  - [x] Did NOT translate: project title, tech tags, milestone labels, dates

- [x] **Task 10: Add mock setup for react-i18next in tests** (AC: 10)
  - [x] Updated `src/test/setup.ts` with global `vi.mock('react-i18next', ...)`
  - [x] Mock: `t(key)` returns actual English translation from `en.json` (flattened lookup)
  - [x] All 157 existing tests continue to pass after mock added
  - [x] Fixed `tsconfig.app.json` to exclude `src/test/**/*` from production build

- [x] **Task 11: Write new tests** (AC: 10)
  - [x] `lib/referral.test.ts` — 18 new tests (23 total) for all new functions
  - [x] `Nav.test.tsx` — 3 new tests: LanguageToggle aria-label, click→vi, click→en
  - [x] `i18n/i18n.test.ts` — 13 tests: key structure parity, non-empty values

- [x] **Task 12: Full validation** (AC: 10)
  - [x] `pnpm run test` — 194/194 tests pass (157 existing + 37 new), 0 regressions
  - [x] `pnpm run lint` — 0 errors
  - [x] `pnpm run build` — succeeds (Vite + tsc, 496KB bundle)

## Dev Notes

### CRITICAL ARCHITECTURE CONSTRAINTS (Carry-forward from Stories 1.3–1.5)

1. **NO `tailwind.config.js`** — Tailwind v4 CSS-first. All new design tokens in `src/index.css`.
2. **NO manual `localStorage.setItem/getItem`** for language state — Zustand persist middleware manages it. Exception: reading the raw value in `detectInitialLanguage()` is allowed since it's outside React.
3. **NO `../../` relative imports** — Use `@/` alias exclusively.
4. **Spring tokens from `@/constants/motion`** — Never inline spring values.
5. **`<MotionConfig reducedMotion="user">` is already in `App.tsx`** — Do NOT add another MotionConfig.
6. **shadcn components are owned code** — No new shadcn components needed for this story.
7. **TypeScript strict mode** — No `any`. Use explicit types.
8. **pnpm** — Always `pnpm`, never `npm install` or `yarn`.
9. **ESLint `import/no-relative-parent-imports`** — Use `@/` always. Never `../../`.

### What Already Exists (DO NOT RECREATE)

| File | Status | Notes |
|------|--------|-------|
| `src/stores/languageStore.ts` | ✅ Done | Zustand + persist, key `'language-preference'`, type `'en' \| 'vi'`, default `'en'` |
| `src/i18n/index.ts` | ⚠️ Incomplete | Configured but: hardcoded `lng: 'en'`, broken export name (`i18next` var → exported as `i18n`), no store sync |
| `src/i18n/en.json` | ⚠️ Empty `{}` | Needs full translation content |
| `src/i18n/vi.json` | ⚠️ Empty `{}` | Needs full translation content |
| `src/lib/referral.ts` | ⚠️ Partial | Only `parseReferralSource()` — needs language mapping added |
| `src/components/layout/Nav.tsx` | ⚠️ Bug | `LanguageToggle` renders correctly but **missing `onClick` handler** — cannot toggle |
| `src/stores/languageStore.test.ts` | ✅ Done | 3 tests covering init, set vi, set en |

### Persist Key Discrepancy

**Epics spec says:** `pf_language` (localStorage key)
**Existing implementation uses:** `'language-preference'` (set in Story 1.2 scaffold)

**Decision:** Keep `'language-preference'` — changing it would silently lose any user's saved preference. This is a minor naming inconsistency vs. the epics; the behavior is identical. Document in Change Log.

### i18n Architecture Pattern

```
App.tsx (mount)
  ↓
detectInitialLanguage() in lib/referral.ts
  Priority: ?lang= URL param > localStorage('language-preference') > ?from= → language mapping > navigator.language
  ↓
i18n.changeLanguage(lang) + setLanguage(lang) in languageStore
  ↓
languageStore.subscribe() in App.tsx
  → on change: i18n.changeLanguage() + document.documentElement.lang = lang
  ↓
useTranslation() in each component → reactive re-render on language change
```

### Translation Key Structure (en.json / vi.json)

```json
// en.json
{
  "nav": {
    "projects": "Projects",
    "about": "About",
    "contact": "Contact"
  },
  "hero": {
    "eyebrow": "⚡ Backend · Fullstack",
    "heading": "Building systems that stay live.",
    "tagline": "I ship production-grade software with real observability — not just demos.",
    "cta": {
      "projects": "View Projects",
      "about": "About Me"
    }
  },
  "projects": {
    "heading": "Projects",
    "filter": {
      "all": "All",
      "featured": "Featured",
      "archived": "Archived"
    },
    "card": {
      "viewLive": "Live",
      "viewGitHub": "GitHub",
      "status": {
        "live": "Live",
        "building": "Building",
        "archived": "Archived"
      }
    }
  },
  "about": {
    "heading": "Why Hire Me",
    "whyHireMe": "I build systems that are meant to stay running — not just demos that survive a presentation. My engineering philosophy is to make the invisible visible: instrumentation, CI/CD guardrails, and honest documentation of every architectural decision, including the wrong ones. I treat each project as a production system from day one, which means you get a developer who ships, monitors, and improves — not one who hands off and moves on.",
    "skills": {
      "heading": "Verified Skills"
    },
    "availability": {
      "heading": "Availability",
      "open": "Open to Work",
      "selective": "Selectively Looking",
      "unavailable": "Not Available",
      "location": "Remote · Ho Chi Minh City"
    }
  },
  "projectDetail": {
    "artistStatement": "About This Project",
    "lessonsLearned": "What I'd Do Differently",
    "buildTimeline": "Build Timeline",
    "cta": {
      "viewLive": "View Live",
      "viewGitHub": "View on GitHub"
    }
  }
}
```

```json
// vi.json
{
  "nav": {
    "projects": "Dự án",
    "about": "Giới thiệu",
    "contact": "Liên hệ"
  },
  "hero": {
    "eyebrow": "⚡ Backend · Fullstack",
    "heading": "Xây dựng hệ thống vận hành ổn định.",
    "tagline": "Tôi phát triển phần mềm production-grade với observability thực sự — không chỉ là demo.",
    "cta": {
      "projects": "Xem Dự Án",
      "about": "Về Tôi"
    }
  },
  "projects": {
    "heading": "Dự Án",
    "filter": {
      "all": "Tất cả",
      "featured": "Nổi bật",
      "archived": "Đã lưu trữ"
    },
    "card": {
      "viewLive": "Xem Live",
      "viewGitHub": "GitHub",
      "status": {
        "live": "Đang hoạt động",
        "building": "Đang xây dựng",
        "archived": "Đã lưu trữ"
      }
    }
  },
  "about": {
    "heading": "Tại Sao Chọn Tôi",
    "whyHireMe": "Tôi xây dựng hệ thống để chạy bền vững — không chỉ là demo sống sót qua một buổi thuyết trình. Triết lý kỹ thuật của tôi là làm cho điều vô hình trở nên hiển thị: instrumentation, CI/CD guardrails, và tài liệu trung thực về mọi quyết định kiến trúc, kể cả những quyết định sai. Tôi coi mỗi dự án là hệ thống production ngay từ ngày đầu tiên — bạn có được một developer vừa ship, vừa monitor, vừa cải thiện liên tục.",
    "skills": {
      "heading": "Kỹ Năng Đã Được Chứng Minh"
    },
    "availability": {
      "heading": "Tình Trạng Tuyển Dụng",
      "open": "Sẵn Sàng Làm Việc",
      "selective": "Đang Xem Xét",
      "unavailable": "Không Nhận",
      "location": "Remote · Hồ Chí Minh"
    }
  },
  "projectDetail": {
    "artistStatement": "Về Dự Án Này",
    "lessonsLearned": "Nếu Làm Lại, Tôi Sẽ...",
    "buildTimeline": "Lịch Sử Phát Triển",
    "cta": {
      "viewLive": "Xem Live",
      "viewGitHub": "Xem trên GitHub"
    }
  }
}
```

> **Technical content that stays English regardless of language:** tech tag names (React, TypeScript, Spring Boot, etc.), metric numbers, GitHub URLs, commit dates, build timelines dates, version numbers on skill badges.

### `lib/referral.ts` Full Implementation

```typescript
// Current (existing):
export function parseReferralSource(search: string): string | null {
  const params = new URLSearchParams(search)
  return params.get('from')
}

// Add:
type Language = 'en' | 'vi'

/**
 * Maps a ?from= referral source to a default language.
 * Returns null if the source doesn't have a language mapping.
 */
export function getLanguageFromReferral(source: string | null): Language | null {
  if (source === 'linkedin') return 'en'
  if (source === 'cv-vn') return 'vi'
  return null
}

/**
 * Parses ?lang= URL param to a Language.
 * Returns null if not present or invalid value.
 */
export function getLangFromUrlParam(search: string): Language | null {
  const params = new URLSearchParams(search)
  const lang = params.get('lang')
  if (lang === 'vi' || lang === 'en') return lang
  return null
}

/**
 * Determines the initial language using priority order:
 * 1. ?lang= URL param (explicit override)
 * 2. Persisted Zustand value in localStorage
 * 3. ?from= referral source → language mapping
 * 4. navigator.language fallback (vi-* → 'vi', else 'en')
 */
export function detectInitialLanguage(search: string): Language {
  // Priority 1: ?lang= explicit override
  const urlLang = getLangFromUrlParam(search)
  if (urlLang) return urlLang

  // Priority 2: Persisted Zustand value
  try {
    const raw = localStorage.getItem('language-preference')
    if (raw) {
      const parsed = JSON.parse(raw) as { state?: { language?: Language } }
      const stored = parsed?.state?.language
      if (stored === 'en' || stored === 'vi') return stored
    }
  } catch {
    // localStorage unavailable (SSR / private mode) — continue to fallback
  }

  // Priority 3: ?from= referral mapping
  const from = parseReferralSource(search)
  const referralLang = getLanguageFromReferral(from)
  if (referralLang) return referralLang

  // Priority 4: navigator.language fallback
  if (typeof navigator !== 'undefined' && navigator.language.startsWith('vi')) return 'vi'
  return 'en'
}
```

### `App.tsx` Updated Pattern

```typescript
import '@/i18n' // Side-effect: initializes i18next

import { lazy, Suspense, useEffect } from 'react'
import i18n from '@/i18n'

import { MotionConfig } from 'framer-motion'
import { BrowserRouter, Navigate, Route, Routes } from 'react-router-dom'

import { detectInitialLanguage } from '@/lib/referral'
import HomePage from '@/pages/HomePage'
import { useLanguageStore } from '@/stores/languageStore'
import { useThemeStore } from '@/stores/themeStore'

const ProjectDetailPage = lazy(() => import('@/pages/ProjectDetailPage'))

export default function App() {
  const { theme } = useThemeStore()
  const { setLanguage } = useLanguageStore()

  // Sync theme store → HTML root class
  useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark')
  }, [theme])

  // Initialize language on mount (runs once)
  useEffect(() => {
    const lang = detectInitialLanguage(window.location.search)
    setLanguage(lang)
    void i18n.changeLanguage(lang)
    document.documentElement.lang = lang

    // Subscribe to future store changes
    const unsub = useLanguageStore.subscribe((state) => {
      void i18n.changeLanguage(state.language)
      document.documentElement.lang = state.language
    })
    return unsub
  }, [setLanguage])

  return (
    <MotionConfig reducedMotion="user">
      <BrowserRouter>
        <Routes>
          <Route path="/" element={<HomePage />} />
          <Route
            path="/projects/:slug"
            element={
              <Suspense fallback={<div>Loading...</div>}>
                <ProjectDetailPage />
              </Suspense>
            }
          />
          <Route path="*" element={<Navigate to="/" replace />} />
        </Routes>
      </BrowserRouter>
    </MotionConfig>
  )
}
```

> ⚠️ **`i18n.changeLanguage()` returns a Promise** — use `void` to suppress the floating promise lint warning, or `await` inside an async function. Do not `.then()` without a `.catch()`.

### `Nav.tsx` LanguageToggle Fix

```typescript
function LanguageToggle() {
  const { language, setLanguage } = useLanguageStore()
  const { t } = useTranslation()

  return (
    <Button
      variant="ghost"
      size="sm"
      onClick={() => setLanguage(language === 'en' ? 'vi' : 'en')}
      className="focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2 text-muted-foreground"
      aria-label="Switch language"
    >
      <span className={language === 'en' ? 'font-semibold text-foreground' : ''}>EN</span>
      <span className="mx-1 text-muted-foreground/50">|</span>
      <span className={language === 'vi' ? 'font-semibold text-foreground' : ''}>VI</span>
    </Button>
  )
}

// In NAV_LINKS — render translated labels in JSX, not in the const array:
// Change the map render:
// {NAV_LINKS.map((link) => (
//   <li key={link.href}>
//     <a href={link.href}>
//       {t(`nav.${link.labelKey}`)}   ← use labelKey, not label
//     </a>
//   </li>
// ))}

// Update NAV_LINKS type:
const NAV_LINKS = [
  { href: '#projects', labelKey: 'projects' },
  { href: '#about', labelKey: 'about' },
  { href: '#contact', labelKey: 'contact' },
]
```

### Test Setup Pattern for react-i18next

Create `src/test/setup.ts` (or update if exists):

```typescript
import '@testing-library/jest-dom'

// Mock react-i18next so t(key) returns the key
// This way all existing tests still find their English text via real en.json values,
// OR you can use the actual translation strings in assertions (both work with this mock)
vi.mock('react-i18next', () => ({
  useTranslation: () => ({
    t: (key: string) => key,  // Returns key as-is for predictable test assertions
    i18n: {
      changeLanguage: vi.fn(),
      language: 'en',
    },
  }),
  initReactI18next: {
    type: '3rdParty' as const,
    init: vi.fn(),
  },
  Trans: ({ children }: { children: React.ReactNode }) => children,
}))
```

> ⚠️ **CRITICAL:** If using `t: (key) => key` mock, existing tests that assert on translated text (e.g. `screen.getByText(/open to work/i)`) will FAIL because `t('about.availability.open')` now returns `'about.availability.open'` not `'Open to Work'`.
>
> **Two options:**
> 1. ✅ **Recommended:** Change `t: (key) => key` to `t: (key: string) => translationsEn[key] ?? key` — pass actual EN translations. Update `About.test.tsx` assertions to use translated strings from en.json.
> 2. Use the real i18next in tests by adding `import '@/i18n'` to the setup file and using `i18n.changeLanguage('en')` before tests.
>
> **If going with option 1**, import `en.json` in the test setup file:
> ```typescript
> import en from '../i18n/en.json'
> const flattenTranslations = (obj: Record<string, unknown>, prefix = ''): Record<string, string> => {
>   return Object.entries(obj).reduce((acc, [k, v]) => {
>     const key = prefix ? `${prefix}.${k}` : k
>     if (typeof v === 'object' && v !== null) return { ...acc, ...flattenTranslations(v as Record<string, unknown>, key) }
>     return { ...acc, [key]: v as string }
>   }, {} as Record<string, string>)
> }
> const translations = flattenTranslations(en)
> // Then in mock: t: (key: string) => translations[key] ?? key
> ```

### Reading Components Before Modifying

**Before updating each component, read it first:**
- `src/components/layout/Nav.tsx` — has LanguageToggle (currently no onClick)
- `src/components/sections/Hero.tsx` — read to find all hardcoded English strings
- `src/components/sections/Projects.tsx` — read for section heading + filter label strings
- `src/components/sections/About.tsx` — already reviewed; know which strings to translate
- `src/pages/ProjectDetailPage.tsx` — read for detail page labels

### Files to Create / Modify

```
src/
├── i18n/
│   ├── index.ts              ← MODIFY: fix export, remove hardcoded lng: 'en', add initImmediate: false
│   ├── en.json               ← MODIFY: populate with full translation content
│   └── vi.json               ← MODIFY: populate with Vietnamese translations
├── lib/
│   ├── referral.ts           ← MODIFY: add getLanguageFromReferral, getLangFromUrlParam, detectInitialLanguage
│   └── referral.test.ts      ← MODIFY: add tests for new functions
├── stores/
│   └── languageStore.ts      ← READ ONLY: already correct, no changes needed
├── App.tsx                   ← MODIFY: import i18n, wire language init + subscription
├── test/
│   └── setup.ts              ← CREATE or MODIFY: add react-i18next mock
├── components/layout/
│   └── Nav.tsx               ← MODIFY: fix LanguageToggle onClick, add useTranslation
├── components/sections/
│   ├── Hero.tsx              ← MODIFY: add useTranslation, replace hardcoded strings
│   ├── Projects.tsx          ← MODIFY: add useTranslation for heading + filter labels
│   └── About.tsx             ← MODIFY: add useTranslation, replace STATUS_LABEL values
└── pages/
    └── ProjectDetailPage.tsx ← MODIFY: add useTranslation for labels
```

### Dependencies Already Installed (Do NOT Re-Install)

From Story 1.1 scaffold (confirmed in package.json via architecture):
- `react-i18next` — installed
- `i18next` — installed

Run `pnpm list react-i18next i18next` to confirm before starting.

### Testing Standards (Same as 1.3–1.5)

- Co-located test files (`.test.tsx` alongside component)
- `@testing-library/react` + `vitest`
- framer-motion mock: `vi.mock('framer-motion', ...)` if component uses motion
- **react-i18next mock required** — add to global test setup (see test setup section above)
- Run `pnpm run test` after EVERY task, not just at the end

### Previous Story Learnings (Critical from 1.3–1.5)

1. **`role="listitem"` on `<a>` overrides link role** — always use `<ul>/<li>/<a>` structure (fixed in 1.5)
2. **`react-refresh/only-export-components`** — do NOT export non-component constants from component files (fixed in 1.5: STATUS_LABEL moved to about.ts)
3. **Safari+VoiceOver: `list-none` removes list semantics** — add `role="list"` explicitly on `<ul>` when using `list-style: none` (fixed in 1.5)
4. **`Record<AvailabilityStatus, string>` not `Record<string, string>`** — use specific union types for lookup maps (fixed in code review for 1.5)
5. **`pnpm run lint` before submitting** — `import/order` violations most common with new files. Run `npx eslint --fix` on new files immediately
6. **157 tests currently pass** — run `pnpm run test` before each task to ensure no regressions
7. **`typeof import()` forbidden** — use `import type * as Mod from '...'` if needed for mocking

### Cross-Story Dependency Notes

- **Story 1.7.x (Accessibility)** — `document.documentElement.lang` update (AC8) is a prerequisite for proper screen reader language announcement. Story 1.6 lays this foundation.
- **Story 1.8 (Animations)** — No i18n impact; animation tokens are in `constants/motion.ts`.
- **Story 4.4 (Referral tracking)** — `referralStore.ts` already scaffolded. Story 1.6 extends `lib/referral.ts` for language mapping; Story 4.4 will extend it further for contextual messaging. Do NOT remove `parseReferralSource` — it's needed by Story 4.4.
- **Stories 1.7.1 / 1.7.2** — ARIA labels and `lang` attribute from this story feed directly into accessibility audit requirements.

### Project Structure Notes

- `i18n/` lives at `src/i18n/` — matches architecture diagram exactly [Source: architecture.md line 1260]
- `lib/referral.ts` already at `src/lib/referral.ts` — extend in place [Source: architecture.md line 1253]
- No new stores, pages, or shadcn components needed — pure wiring + content story

### References

- Epics: [_bmad-output/planning-artifacts/epics.md](_bmad-output/planning-artifacts/epics.md) — Story 1.6 definition (lines 436–477)
- Architecture: [_bmad-output/planning-artifacts/architecture.md](_bmad-output/planning-artifacts/architecture.md) — i18n cross-cutting concern (lines 46–48, 159–161), Zustand state (lines 568–608), FR41-42 mapping (line 1613)
- PRD: [_bmad-output/planning-artifacts/prd.md](_bmad-output/planning-artifacts/prd.md) — FR41–FR42 (selective bilingual scope)
- Previous story: [_bmad-output/implementation-artifacts/1-5-about-skills-availability.md](_bmad-output/implementation-artifacts/1-5-about-skills-availability.md) — Dev learnings (code review fixes, eslint patterns)

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

### Completion Notes List

- All 12 tasks completed. 194/194 tests pass (157 existing + 37 new), 0 lint errors, build succeeds (496KB bundle).
- Persist key kept as `'language-preference'` (not `pf_language` from epics) to avoid losing existing user preferences. Documented in Dev Notes.
- `STATUS_LABEL` import removed from `About.tsx` — replaced with `t(\`about.availability.${status}\`)` inline.
- `tsconfig.app.json` required an `src/test/**/*` exclude to prevent `vi` global from leaking into the production tsc compilation (setup.ts is not a `*.test.ts` file so it wasn't previously excluded).
- react-i18next global mock in `setup.ts` uses `flattenTranslations(en.json)` so `t(key)` returns actual English values — all 157 pre-existing tests pass without modification.
- `detectInitialLanguage()` priority: `?lang=` > localStorage > `?from=` referral > `navigator.language`. Aligns with AC3–AC6.

### File List

**Modified:**
- `src/lib/referral.ts` — Added `getLanguageFromReferral`, `getLangFromUrlParam`, `detectInitialLanguage`
- `src/lib/referral.test.ts` — 18 new tests (23 total)
- `src/i18n/index.ts` — Fixed export, removed hardcoded `lng: 'en'`, added `initImmediate: false`
- `src/i18n/en.json` — Populated: nav, hero, projects, about, projectDetail namespaces
- `src/i18n/vi.json` — Populated: full Vietnamese translations for all keys
- `src/App.tsx` — Added i18n init, `detectInitialLanguage` on mount, `useLanguageStore.subscribe()` sync; consolidated `import i18n from '@/i18n'` (code review fix)
- `src/components/layout/Nav.tsx` — Fixed `LanguageToggle` onClick, added `useTranslation()` for nav links via `NAV_LINK_DEFS`
- `src/components/layout/Nav.test.tsx` — 3 new tests: LanguageToggle aria-label, click→vi, click→en; added explicit `beforeEach` import (code review fix)
- `src/components/sections/Hero.tsx` — Added `useTranslation()`, all strings via `t()`
- `src/components/sections/About.tsx` — Added `useTranslation()`, removed STATUS_LABEL import, all UI strings via `t()`
- `src/components/sections/Projects.tsx` — Added `useTranslation()`, heading + filter labels + empty state via `t()`; fixed variable shadowing `t` → `tag` in `.some()` callback (code review fix)
- `src/pages/ProjectDetailPage.tsx` — Added `useTranslation()`, all UI labels via `t()`
- `src/test/setup.ts` — Added global `vi.mock('react-i18next', ...)` with `flattenTranslations(en.json)` lookup; added explicit `vi` import from vitest (code review fix)
- `tsconfig.app.json` — Added `src/test/**/*` to exclude list

**Created:**
- `src/i18n/i18n.test.ts` — 13 tests: key parity between en.json and vi.json, non-empty value guards; added `viewLiveAriaLabel` assertions to en/vi projectDetail tests (code review fix)

## Change Log

- 2026-03-05: Story 1.6 created by SM agent (create-story workflow). Story key: `1-6-internationalization-vi-en`. Persist key discrepancy documented: epics spec says `pf_language`, existing impl uses `'language-preference'` — kept existing key to avoid regression.
- 2026-03-06: Story 1.6 implemented by dev agent (claude-sonnet-4-6). All 12 tasks complete. 194/194 tests pass, 0 lint, build green. Status: review.
- 2026-03-06: Code review by claude-sonnet-4-6. 5 issues found and fixed (2 Medium, 3 Low): Projects.tsx variable shadowing, App.tsx i18n import consolidation, i18n.test.ts missing assertions, Nav.test.tsx + setup.ts explicit vitest imports. 194/194 tests, 0 lint, build green. Status: done.
