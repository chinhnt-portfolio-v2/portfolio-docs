# Story 1.1: Initialize Portfolio FE Project

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a developer,
I want a fully configured Portfolio FE project with all required tooling and folder structure,
so that the team can build all Epic 1–6 features on a stable, standards-compliant foundation.

## Acceptance Criteria

**AC1: Project Bootstrap**
```
Given no project exists,
When `npm create vite@latest portfolio-fe -- --template react-ts` is run,
Then the project starts with `npm run dev` showing a working React app in the browser
```

**AC2: All Dependencies Installed**
```
Given all dependencies are installed,
Then Tailwind v4 (CSS-first, no `tailwind.config.js`), shadcn/ui CLI, Zustand, Framer Motion,
React Router v7, react-i18next, Lucide React, and Vitest are all available and importable
```

**AC3: Test Environment Configured**
```
Given Vitest is configured,
When `npm run test` is run,
Then Vitest runs with jsdom environment and reports 0 failing tests
```

**AC4: Folder Structure Established**
```
Given the folder structure is set up,
Then source code follows:
  src/components/, src/pages/, src/stores/, src/i18n/, src/lib/,
  src/hooks/, src/constants/, src/types/
```

**AC5: Vercel Deployment Configuration**
```
Given Vercel deployment is needed,
Then a `vercel.json` exists with SPA rewrite rules so all routes serve `index.html`
```

**AC6: ESLint Configured**
```
Given ESLint is configured,
When `npm run lint` is run (or equivalent),
Then no lint errors occur and import/order + no-relative-parent-imports rules are enforced
```

**AC7: Environment Variables Wired**
```
Given .env.example exists in repo,
Then it documents VITE_API_URL and VITE_WS_URL with placeholder values (no secrets committed)
```

## Tasks / Subtasks

- [x] **Task 1: Scaffold Vite + React + TypeScript project** (AC: 1)
  - [x] Run `npm create vite@latest portfolio-fe -- --template react-ts`
  - [x] Verify `npm run dev` produces a working React app in browser
  - [x] Remove all Vite default template boilerplate (App.tsx placeholder content, App.css, logo, counter)

- [x] **Task 2: Install and configure Tailwind CSS v4** (AC: 2)
  - [x] Run `pnpm add tailwindcss @tailwindcss/vite`
  - [x] Run `pnpm add -D @types/node`
  - [x] Add `@tailwindcss/vite` plugin to `vite.config.ts`
  - [x] Replace `src/index.css` content with `@import "tailwindcss";` + `@theme {}` block containing design tokens (Electric Purple `#A855F7`, base spacing unit 8px)
  - [x] **CRITICAL: Do NOT create `tailwind.config.js`** — all config is CSS-first in `src/index.css`
  - [x] Add `tw-animate-css` package (replaces deprecated `tailwindcss-animate`)

- [x] **Task 3: Initialize shadcn/ui** (AC: 2)
  - [x] Run `pnpm dlx shadcn@latest init` (auto-detects Vite + Tailwind v4)
  - [x] Verify `components.json` is generated
  - [x] Verify `src/components/ui/` directory is created (owned source code, not package import)
  - [x] **NOTE:** shadcn components are copied as owned source code — never import from a `shadcn` package

- [x] **Task 4: Install remaining core libraries** (AC: 2)
  - [x] Run `pnpm add framer-motion lucide-react`
  - [x] Run `pnpm add react-i18next i18next`
  - [x] Run `pnpm add react-router-dom` (v7)
  - [x] Run `pnpm add zustand`
  - [x] Run `pnpm add react-hook-form @hookform/resolvers zod`

- [x] **Task 5: Configure Vitest** (AC: 3)
  - [x] Run `pnpm add -D vitest @testing-library/react @testing-library/user-event jsdom`
  - [x] Create `vitest.config.ts` with jsdom environment and coverage thresholds (>=60% for functions/lines/branches/statements in `src/lib/**`, `src/hooks/**`, `src/stores/**`)
  - [x] Add `"test": "vitest"` and `"test:coverage": "vitest run --coverage"` scripts to `package.json`
  - [x] Verify `npm run test` passes with 0 failures

- [x] **Task 6: Set up full project folder structure** (AC: 4)
  - [x] Create `src/pages/` — route-level page components
  - [x] Create `src/components/layout/` — Nav, MobileSheet, SkipLinks
  - [x] Create `src/components/sections/` — Hero, Projects, About, Contact
  - [x] Create `src/components/shared/` — reusable cross-section components
  - [x] Ensure `src/components/ui/` exists (from shadcn init)
  - [x] Create `src/hooks/` — custom React hooks
  - [x] Create `src/stores/` — Zustand stores
  - [x] Create `src/lib/` — utility functions
  - [x] Create `src/i18n/` — translation files
  - [x] Create `src/constants/` — app-wide constants
  - [x] Create `src/types/` — TypeScript type definitions
  - [x] Add `.gitkeep` files to empty directories to track in git

- [x] **Task 7: Create placeholder/stub files for key modules** (AC: 4)
  - [x] Create `src/constants/motion.ts` with `SPRING_GENTLE`, `SPRING_SNAPPY`, `SPRING_BOUNCY` exports
  - [x] Create `src/constants/config.ts` with `WS_RECONNECT_MAX_RETRIES=3`, `WS_BACKOFF_BASE_MS=1000`
  - [x] Create `src/constants/projects.ts` with empty project config array and type definition
  - [x] Create `src/stores/themeStore.ts` — dark/light with Zustand persist middleware
  - [x] Create `src/stores/languageStore.ts` — vi/en with Zustand persist middleware
  - [x] Create `src/stores/visitorStore.ts` — returnCount, lastViewedProjectSlug with persist
  - [x] Create `src/stores/referralStore.ts` — referralSource with persist
  - [x] Create `src/stores/galleryStore.ts` — activeFilter (URL param, NOT persisted)
  - [x] Create `src/stores/metricsStore.ts` — WS metrics + connectionState (session-only, no persist)
  - [x] Create `src/lib/utils.ts` with `cn()` helper (Tailwind class merge utility)
  - [x] Create `src/lib/formatDate.ts` — ISO → relative time (Intl.RelativeTimeFormat stub)
  - [x] Create `src/lib/validators.ts` — Zod schema exports (contactFormSchema stub)
  - [x] Create `src/lib/referral.ts` — parse `?from=` param stub
  - [x] Create `src/i18n/index.ts` — i18next init
  - [x] Create `src/i18n/en.json` — empty English translations `{}`
  - [x] Create `src/i18n/vi.json` — empty Vietnamese translations `{}`
  - [x] Create `src/types/motion.types.ts` — Framer Motion spring token types

- [x] **Task 8: Configure TypeScript, path alias and ESLint** (AC: 6)
  - [x] Ensure `tsconfig.app.json` has `"strict": true` enabled
  - [x] Add `paths: { "@/*": ["./src/*"] }` in `tsconfig.app.json`
  - [x] Add path alias in `vite.config.ts`: `resolve: { alias: { '@': path.resolve(__dirname, './src') } }`
  - [x] Install ESLint plugins: `pnpm add -D eslint eslint-plugin-import eslint-import-resolver-typescript @typescript-eslint/eslint-plugin`
  - [x] Configure `eslint.config.js` with `import/order`, `import/no-relative-parent-imports`, `@typescript-eslint/consistent-type-imports` rules
  - [x] Verify `import/no-relative-parent-imports` prevents `../../` imports; `@/` alias works

- [x] **Task 9: Set up Vite dev proxy for BE integration** (AC: 5)
  - [x] Add server proxy config to `vite.config.ts`:
    - `/api` → `http://localhost:8080` with `changeOrigin: true`
    - `/ws` → `ws://localhost:8080` with `ws: true`, `changeOrigin: true`, `rewriteWsOrigin: true`

- [x] **Task 10: Create App.tsx skeleton with providers** (AC: 1, 2)
  - [x] Wrap entire app with `<MotionConfig reducedMotion="user">` (Framer Motion built-in, reads `prefers-reduced-motion`)
  - [x] Set up `BrowserRouter` from React Router v7
  - [x] Wire up basic route structure: `/` → HomePage, `/projects/:slug` → ProjectDetailPage (React.lazy + Suspense)
  - [x] Create stub `src/pages/HomePage.tsx` and `src/pages/ProjectDetailPage.tsx`

- [x] **Task 11: Configure Vercel deployment** (AC: 5)
  - [x] Create `vercel.json` in project root with SPA rewrite rules

- [x] **Task 12: Create environment variable files** (AC: 7)
  - [x] Create `.env.example` (committed) with VITE_API_URL and VITE_WS_URL placeholders
  - [x] Create `.env.local` (gitignored, actual dev values)
  - [x] Verify `.gitignore` includes `.env.local` and `.env.production`

- [x] **Task 13: Create GitHub Actions CI workflow** (AC: 3, 6)
  - [x] Create `.github/workflows/ci.yml` that runs Vitest + ESLint on PR

- [x] **Task 14: Create DECISIONS.md** (AC: 1)
  - [x] Create `DECISIONS.md` in project root documenting key FE architectural decisions

- [x] **Task 15: Final verification** (AC: 1-7)
  - [x] `npm run dev` starts without errors
  - [x] `npm run build` succeeds
  - [x] `npm run test` shows 0 failures
  - [x] No `tailwind.config.js` exists in project
  - [x] No manual `localStorage.setItem/getItem` anywhere in codebase
  - [x] All imports use `@/` alias (no `../../`)
  - [x] `vercel.json` exists with SPA rewrite rules

## Dev Notes

### Critical Architecture Constraints (MUST FOLLOW — No Exceptions)

1. **NO `tailwind.config.js`** — Tailwind v4 is CSS-first. ALL config (theme, colors, tokens) goes in `src/index.css` using `@theme {}`. Any agent creating `tailwind.config.js` is making a critical error.

2. **NO manual `localStorage.setItem/getItem`** — Every localStorage operation MUST use `zustand/middleware persist`. This includes theme, language, visitor data, referral source.

3. **NO `../../` relative imports** — Always use `@/` path alias. ESLint `import/no-relative-parent-imports` enforces this.

4. **NO `new WebSocket()` in components** — only allowed inside `src/hooks/useWebSocket.ts` (future story, but architecture hook must be established).

5. **`<MotionConfig reducedMotion="user">` at App.tsx root** — this is the single source of prefers-reduced-motion compliance. No custom motion context needed.

6. **Spring tokens from `constants/motion.ts` only** — never inline `stiffness`/`damping` values in components.

7. **shadcn/ui components are owned code** — `pnpm dlx shadcn@latest add [component]` copies source into `src/components/ui/`. Never import from a `shadcn` package.

8. **React Router v7** — not v6 or earlier.

9. **Vite 7 + `vite-plugin-ssg` compatibility is UNVERIFIED** — Do NOT add `vite-plugin-ssg` in this story.

10. **TypeScript strict mode** — no `any` bypasses allowed.

### Package Manager

Use **`pnpm`** as package manager. npm only for `npm create vite@latest` scaffolding.

### References

- [Source: _bmad-output/planning-artifacts/architecture.md#Frontend Technology Stack]
- [Source: _bmad-output/planning-artifacts/epics.md#Story 1.1]

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- shadcn/ui init required `paths` to be added to both `tsconfig.app.json` AND `tsconfig.json` (root) before it could validate the import alias
- Test files caused `tsc -b` build failure — fixed by adding `"vitest/globals"` to `types` in `tsconfig.app.json` and excluding test files from the app tsconfig
- ESLint flat config (v9) used instead of `.eslintrc.json` — project scaffolded with Vite which defaults to `eslint.config.js`

### Completion Notes List

All 15 tasks completed and verified:
- Vite 7 + React 19 + TypeScript project scaffolded at `portfolio-fe/`
- Tailwind CSS v4 configured CSS-first (no tailwind.config.js)
- shadcn/ui initialized with components.json generated
- All dependencies installed: framer-motion, zustand, react-router-dom v7, react-i18next, lucide-react, react-hook-form, zod
- Vitest with jsdom, coverage thresholds >=60%
- Full folder structure with .gitkeep files
- All stub files: 6 Zustand stores, 4 lib utilities, 3 constants, i18n setup, motion types
- ESLint: import/order, import/no-relative-parent-imports, consistent-type-imports
- App.tsx: MotionConfig + BrowserRouter + lazy routing
- vercel.json SPA rewrites, .env.example, GitHub Actions CI
- Build: 88KB gzip (target <300KB)
- Tests: 1/1 pass
- Lint: 0 errors

### File List

portfolio-fe/package.json
portfolio-fe/pnpm-lock.yaml
portfolio-fe/vite.config.ts
portfolio-fe/vitest.config.ts
portfolio-fe/tsconfig.json
portfolio-fe/tsconfig.app.json
portfolio-fe/tsconfig.node.json
portfolio-fe/eslint.config.js
portfolio-fe/components.json
portfolio-fe/vercel.json
portfolio-fe/.env.example
portfolio-fe/.env.local
portfolio-fe/.gitignore
portfolio-fe/DECISIONS.md
portfolio-fe/index.html
portfolio-fe/src/App.tsx
portfolio-fe/src/main.tsx
portfolio-fe/src/index.css
portfolio-fe/src/vite-env.d.ts
portfolio-fe/src/pages/HomePage.tsx
portfolio-fe/src/pages/ProjectDetailPage.tsx
portfolio-fe/src/components/layout/.gitkeep
portfolio-fe/src/components/sections/.gitkeep
portfolio-fe/src/components/shared/.gitkeep
portfolio-fe/src/components/ui/.gitkeep
portfolio-fe/src/hooks/.gitkeep
portfolio-fe/src/lib/utils.ts
portfolio-fe/src/lib/formatDate.ts
portfolio-fe/src/lib/validators.ts
portfolio-fe/src/lib/referral.ts
portfolio-fe/src/stores/themeStore.ts
portfolio-fe/src/stores/languageStore.ts
portfolio-fe/src/stores/visitorStore.ts
portfolio-fe/src/stores/referralStore.ts
portfolio-fe/src/stores/galleryStore.ts
portfolio-fe/src/stores/metricsStore.ts
portfolio-fe/src/constants/motion.ts
portfolio-fe/src/constants/config.ts
portfolio-fe/src/constants/projects.ts
portfolio-fe/src/i18n/index.ts
portfolio-fe/src/i18n/en.json
portfolio-fe/src/i18n/vi.json
portfolio-fe/src/types/motion.types.ts
portfolio-fe/src/test/setup.ts
portfolio-fe/src/test/setup.test.ts
portfolio-fe/.github/workflows/ci.yml
portfolio-fe/src/lib/formatDate.test.ts
portfolio-fe/src/lib/validators.test.ts
portfolio-fe/src/lib/referral.test.ts
portfolio-fe/src/lib/utils.test.ts
portfolio-fe/src/stores/themeStore.test.ts
portfolio-fe/src/stores/galleryStore.test.ts
portfolio-fe/src/stores/languageStore.test.ts
portfolio-fe/src/stores/visitorStore.test.ts
portfolio-fe/src/stores/referralStore.test.ts
portfolio-fe/src/stores/metricsStore.test.ts
portfolio-fe/tsconfig.test.json

## Senior Developer Review (AI)

**Review Date:** 2026-03-04
**Reviewer:** claude-sonnet-4-6 (adversarial code review)
**Outcome:** Changes Requested → Fixed in same session

### Action Items (all resolved)

- [x] [High] `--color-accent` CSS conflict — shadcn `@theme inline` was overriding Electric Purple brand token
- [x] [High] No CI coverage step — `pnpm test:coverage` was never called; 60% threshold never enforced
- [x] [Med] Dark mode class toggle missing — `useThemeStore` stored preference but never applied `.dark` class to HTML root
- [x] [Med] `tw-animate-css` in devDependencies instead of dependencies
- [x] [Med] `vitest/globals` polluting production app TypeScript namespace
- [x] [Med] `public/vite.svg` (Vite template default) not removed despite task claiming it was

### Remaining Known Issues (Low severity, deferred)

- `src/i18n/index.ts` naming diverges from architecture's `src/i18n/config.ts` — acceptable for story 1.1, refactor in story 1.6 when i18n is fully built out
- No `public/og-image.png` — not in story 1.1 scope; add before first production deploy
- ESLint import groups not fully granular vs 5-group architecture spec — functional, improve in later story

## Change Log

- 2026-03-04: Story implemented — Portfolio FE project initialized with full tooling stack (Vite 7, React 19, TypeScript, Tailwind v4 CSS-first, shadcn/ui, Zustand, Framer Motion, React Router v7, Vitest, ESLint)
- 2026-03-04: Code review fixes — renamed `--color-accent` to `--color-brand`, added 42 unit tests (100% coverage), CI coverage gate, dark mode toggle, moved `tw-animate-css` to deps, isolated vitest types, removed `vite.svg`
