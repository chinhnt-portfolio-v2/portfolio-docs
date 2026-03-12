# Story 1.4: Project Detail Page

Status: done

## Story

As a recruiter,
I want a full project page with its own shareable URL,
so that I can bookmark and share specific projects and read the complete decision story.

## Acceptance Criteria

**AC1: Page Load from Slug**
```
Given the recruiter navigates to /projects/{slug},
When the page loads,
Then a project detail page renders with the correct project data matching the slug
  (title, artistStatement, tech stack tags, liveUrl, githubUrl, build timeline milestones).
```

**AC2: Build Timeline Component (FR7)**
```
Given a project has timeline.milestones defined,
When the detail page renders,
Then a vertical milestone list is displayed with:
  - Numbered circle badge (1-indexed: ①, ②, ③...)
  - Milestone label (e.g. "First Commit", "MVP Complete", "Production Deploy")
  - Milestone date formatted as "MMM DD, YYYY" (e.g. "Jan 15, 2026")
  - A vertical connector line between milestones (CSS border-left)
  - Minimum 2 milestones required for a project to show the timeline
  - Mobile: same vertical layout, connector line maintained
```

**AC3: "What I'd Do Differently" Collapsible Section (FR6)**
```
Given a project has a lessonsLearned field,
When the recruiter views the detail page,
Then a collapsible section renders with:
  - Expand/collapse trigger label: "What I'd do differently"
  - Content hidden by default (collapsed state)
  - Clicking the trigger toggles expanded/collapsed
  - Animation: spring-gentle expand/collapse (height transition via framer-motion layout)
  - Accessible: trigger is a <button> with aria-expanded attribute
```

**AC4: "AI Build Story" Placeholder Section**
```
Given the detail page layout,
Then a reserved placeholder space exists for the Phase 2 "AI Build Story" section.
  - Renders cleanly in its absence with no layout shift
  - Placeholder renders a visually subtle empty container or omits entirely if no content
  - MUST NOT break the layout when BuildStoryPreview returns null/error
```

**AC5: Not Found State**
```
Given the recruiter navigates to /projects/nonexistent (a slug not in projects.ts),
When the page attempts to load,
Then a "project not found" state renders with:
  - Message: "Project not found."
  - A link/button: "Back to gallery" that navigates to / (home)
  - No broken layout or console errors
```

**AC6: Deep Linking (SPA)**
```
Given a detail page URL is shared (e.g. https://portfolio.chinh.dev/projects/wallet-app),
When a new user opens it directly (not via React Router internal navigation),
Then the correct project detail page loads — Vercel rewrites handle SPA routing,
  React Router v7 handles the client-side match to /projects/:slug.
```

**AC7: "Back to Gallery" Navigation**
```
Given the recruiter is on a project detail page,
When they look for a way back,
Then a "← Back to gallery" link/button renders at the top of the page,
  navigating to "/" using React Router <Link> (not window.location.href).
```

**AC8: External Links**
```
Given the detail page renders liveUrl and/or githubUrl for a project,
Then each link:
  - Opens in a new tab (target="_blank")
  - Has rel="noopener noreferrer"
  - Shows a "↗" icon appended
  - "Live" link labeled "View live ↗"
  - "GitHub" link labeled "View on GitHub ↗"

Given a project has no liveUrl (status "building"),
Then the "View live" link is disabled:
  - opacity: 0.4, cursor: default, no hover effect
  - Replaced by a "Live soon" badge next to it
```

**AC9: ProjectConfig Interface Extension**
```
Given Story 1.4 implementation is complete,
When projects.ts is read,
Then ProjectConfig interface has been extended with:
  - lessonsLearned?: string   // "What I'd do differently" content
  - timeline?: {
      milestones: Array<{ label: string; date: string }> // date: "YYYY-MM-DD"
    }
  And at least 2 of the 3 existing projects have valid timeline.milestones populated.
```

**AC10: No Regressions & Full Build**
```
Given Story 1.4 implementation is complete,
When validation runs,
Then:
  - All existing 117 tests from Stories 1.1–1.3 still pass
  - New component tests written for ProjectDetailPage and BuildTimeline
  - pnpm run lint: 0 errors
  - pnpm run build: succeeds (Vite + tsc both pass)
  - Coverage: ProjectDetailPage.test.tsx covers not-found state, back navigation, and data rendering
```

## Tasks / Subtasks

- [x] **Task 1: Extend ProjectConfig interface & populate data** (AC: 2, 9)
  - [x] Add `lessonsLearned?: string` to `ProjectConfig` in `src/constants/projects.ts`
  - [x] Add `timeline?: { milestones: Array<{ label: string; date: string }> }` to `ProjectConfig`
  - [x] Populate `timeline.milestones` for `portfolio-v2`, `wallet-app`, `portfolio-v1` (min 2 milestones each)
  - [x] Populate `lessonsLearned` for all 3 projects
  - [x] Update `constants/projects.test.ts` — verify new fields pass shape check

- [x] **Task 2: Create BuildTimeline component** (AC: 2)
  - [x] Create `src/components/shared/BuildTimeline.tsx`
  - [x] Props: `milestones: Array<{ label: string; date: string }>`
  - [x] Render numbered badge (1-indexed) + label + formatted date ("MMM DD, YYYY" via `formatDate.ts`)
  - [x] CSS `border-left` vertical connector line between milestones
  - [x] Mobile responsive: same vertical layout
  - [x] Write `BuildTimeline.test.tsx` — renders milestones with correct badge numbers, labels, dates

- [x] **Task 3: Implement ProjectDetailPage** (AC: 1, 3, 4, 5, 6, 7, 8)
  - [x] Read `useParams()` from react-router-dom to get `slug`
  - [x] Find project by slug: `const project = projects.find(p => p.slug === slug)`
  - [x] If not found → render not-found state with "Project not found." + "← Back to gallery" `<Link to="/">`
  - [x] If found → render full page layout:
    - `<Link to="/">← Back to gallery</Link>` at top (React Router Link, NOT <a href>)
    - Hero: project title (H1) + StatusIndicator badge
    - Artist Statement block: `project.artistStatement` (the WHY)
    - Tech stack: row of `<Badge>` tags (shadcn/ui badge)
    - External links: "View live ↗" + "View on GitHub ↗" (disabled pattern for no liveUrl)
    - BuildTimeline: rendered if `project.timeline?.milestones.length >= 2`
    - Collapsible "What I'd do differently" section (if `project.lessonsLearned` exists)
    - BuildStoryPreview placeholder: reserved `<div>` space (empty, no layout shift)
  - [x] Lazy loading already configured in App.tsx — do NOT add another Suspense wrapper
  - [x] Write `ProjectDetailPage.test.tsx`:
    - Test: renders correct project data for valid slug
    - Test: renders not-found state for invalid slug
    - Test: "Back to gallery" link renders with correct href
    - Test: collapsible section toggles on click
    - Test: BuildTimeline renders when milestones ≥ 2

- [x] **Task 4: Implement collapsible lessonsLearned section** (AC: 3)
  - [x] Use framer-motion `motion.div` with `layout` for height animation
  - [x] Trigger: `<button aria-expanded={isOpen}>What I'd do differently</button>`
  - [x] Transition: `SPRING_GENTLE` (from `@/constants/motion`)
  - [x] Initial state: collapsed (hidden)
  - [x] CSS: `overflow: hidden` on the animated container

- [x] **Task 5: Run full validation** (AC: 10)
  - [x] `pnpm run test` — all tests pass (117 existing + new tests, 0 regressions)
  - [x] `pnpm run lint` — 0 errors
  - [x] `pnpm run build` — succeeds

## Dev Notes

### CRITICAL ARCHITECTURE CONSTRAINTS (Same as Story 1.3 — Do Not Violate)

1. **NO `tailwind.config.js`** — Tailwind v4 CSS-first. All new tokens in `src/index.css`.
2. **NO manual `localStorage`** — Zustand persist middleware only.
3. **NO `../../` relative imports** — Use `@/` alias exclusively.
4. **Spring tokens from `@/constants/motion`** — Never inline spring values.
5. **`<MotionConfig reducedMotion="user">` is already in App.tsx** — Do NOT add another MotionConfig.
6. **shadcn components are owned code** — Use `pnpm dlx shadcn@latest add [component]` to copy source. Always run `eslint --fix` on generated files.
7. **TypeScript strict mode** — No `any`.
8. **pnpm** — Always `pnpm`, never `npm install` or `yarn`.
9. **Lazy loading already configured** — `ProjectDetailPage` is already `React.lazy()`-loaded in `App.tsx` inside a `<Suspense>`. Do NOT add another Suspense inside the page component.

### ProjectConfig Interface Extension (Required Changes to projects.ts)

```typescript
// Extend the existing interface — add these two fields:
export interface ProjectConfig {
  slug: string
  title: string
  description: string
  tags: string[]
  status: 'live' | 'building' | 'archived'
  featured?: boolean
  githubUrl?: string
  liveUrl?: string
  metrics?: {
    shipDays?: number
    uptimeDays?: number
  }
  artistStatement?: string
  buildTimelineStart?: string   // legacy — kept for backward compat
  buildTimelineEnd?: string     // legacy — kept for backward compat
  // NEW in Story 1.4:
  lessonsLearned?: string       // "What I'd do differently" content (collapsible)
  timeline?: {
    milestones: Array<{
      label: string  // e.g. "First Commit", "MVP Complete", "Production Deploy"
      date: string   // ISO date: "YYYY-MM-DD"
    }>
  }
}
```

**Sample populated data for portfolio-v2:**
```typescript
{
  slug: 'portfolio-v2',
  // ... existing fields unchanged ...
  lessonsLearned: 'I would have invested in the design system upfront instead of retrofitting tokens. Building the component library first would have prevented 3 rounds of refactoring.',
  timeline: {
    milestones: [
      { label: 'First Commit', date: '2026-02-01' },
      { label: 'App Shell + Design System', date: '2026-02-15' },
      { label: 'Production Deploy', date: '2026-03-15' },  // update with real date
    ]
  }
}
```

> **Note to dev agent:** Update `date` values to reflect actual real dates. The `formatDate.ts` utility already exists — use it. Check `src/lib/formatDate.ts` to see the existing format function signature before writing new date logic.

### BuildTimeline Component Spec

```tsx
// src/components/shared/BuildTimeline.tsx
interface BuildTimelineProps {
  milestones: Array<{ label: string; date: string }>
}

// Visual layout:
// ①  First Commit         Jan 15, 2026
// │
// ②  MVP Complete         Feb 10, 2026
// │
// ③  Production Deploy    Mar 3, 2026

// Implementation:
// - Outer container: relative, with left-border connector (border-left: 2px solid var(--color-border))
// - Each milestone: flex row, number badge (circle) + text block
// - Number badge: small circle with milestone index (1-indexed), border, no fill background
// - Date formatting: use existing `formatDate.ts` OR `new Intl.DateTimeFormat('en-US', { month: 'short', day: 'numeric', year: 'numeric' }).format(new Date(date))`
// - CSS connector: `::before` pseudo-element or actual div with border-left on container
```

**Accessible BuildTimeline:**
```tsx
<ol aria-label="Build timeline">
  {milestones.map((milestone, i) => (
    <li key={i} className="...">
      <span aria-hidden="true" className="milestone-badge">{i + 1}</span>
      <div>
        <span className="milestone-label">{milestone.label}</span>
        <time dateTime={milestone.date} className="milestone-date">
          {formatMilestoneDate(milestone.date)}
        </time>
      </div>
    </li>
  ))}
</ol>
```

### ProjectDetailPage Layout Structure

```tsx
// Rough structure — implement with proper Tailwind classes
export default function ProjectDetailPage() {
  const { slug } = useParams<{ slug: string }>()
  const project = projects.find(p => p.slug === slug)

  if (!project) {
    return (
      <main className="section-container py-24 text-center">
        <p className="text-secondary mb-4">Project not found.</p>
        <Link to="/" className="...">Back to gallery</Link>
      </main>
    )
  }

  return (
    <main id="main-content">
      {/* Back nav */}
      <Link to="/">← Back to gallery</Link>

      {/* Hero block */}
      <h1>{project.title}</h1>
      <StatusIndicator status={project.status} />

      {/* Artist Statement (the WHY) */}
      {project.artistStatement && <p>{project.artistStatement}</p>}

      {/* Tech stack */}
      <div role="list" aria-label="Tech stack">
        {project.tags.map(tag => <Badge key={tag}>{tag}</Badge>)}
      </div>

      {/* External links */}
      {project.liveUrl ? (
        <a href={project.liveUrl} target="_blank" rel="noopener noreferrer">View live ↗</a>
      ) : (
        <span className="opacity-40 cursor-default">View live</span>
        // + "Live soon" badge
      )}
      {project.githubUrl && (
        <a href={project.githubUrl} target="_blank" rel="noopener noreferrer">View on GitHub ↗</a>
      )}

      {/* Build Timeline */}
      {project.timeline && project.timeline.milestones.length >= 2 && (
        <BuildTimeline milestones={project.timeline.milestones} />
      )}

      {/* Collapsible: What I'd do differently */}
      {project.lessonsLearned && (
        <CollapsibleLessons content={project.lessonsLearned} />
      )}

      {/* BuildStoryPreview placeholder — Phase 2 */}
      {/* Reserved slot: renders nothing now, no layout shift */}
      <div aria-hidden="true" />
    </main>
  )
}
```

### Collapsible Lessons Learned Pattern

```tsx
// Inline in ProjectDetailPage (not a separate file unless it gets complex)
const [lessonsOpen, setLessonsOpen] = useState(false)

<div>
  <button
    onClick={() => setLessonsOpen(o => !o)}
    aria-expanded={lessonsOpen}
    className="..."
  >
    What I'd do differently {lessonsOpen ? '▲' : '▼'}
  </button>
  <AnimatePresence>
    {lessonsOpen && (
      <motion.div
        key="lessons"
        initial={{ height: 0, opacity: 0 }}
        animate={{ height: 'auto', opacity: 1 }}
        exit={{ height: 0, opacity: 0 }}
        transition={SPRING_GENTLE}
        style={{ overflow: 'hidden' }}
      >
        <p>{project.lessonsLearned}</p>
      </motion.div>
    )}
  </AnimatePresence>
</div>
```

> **Note:** `AnimatePresence` wrapping a single toggling item is correct here (it exits/enters). This is the `skeleton → content` swap pattern from architecture doc.

### Not Found State Details

```tsx
// Simple, no drama:
if (!project) {
  return (
    <main className="max-w-2xl mx-auto py-24 px-4 text-center">
      <p className="text-secondary mb-6">Project not found.</p>
      <Link
        to="/"
        className="text-brand hover:underline"
      >
        ← Back to gallery
      </Link>
    </main>
  )
}
```

### External Links Pattern (Disable for "building" / no liveUrl)

```tsx
// From UX spec: "Not yet available" links display as disabled
// opacity: 0.4, cursor: default, no hover — with "Live soon" badge
{project.liveUrl ? (
  <a
    href={project.liveUrl}
    target="_blank"
    rel="noopener noreferrer"
    aria-label={`View ${project.title} live (opens in new tab)`}
  >
    View live ↗
  </a>
) : (
  <span className="flex items-center gap-2 opacity-40 cursor-default" aria-label="Live deployment not yet available">
    View live
    <Badge variant="secondary">Live soon</Badge>
  </span>
)}
```

### Existing Files to Read Before Starting

Read these before implementing to avoid regressions:
- `src/pages/ProjectDetailPage.tsx` — current stub (lines 1-9, minimal placeholder)
- `src/constants/projects.ts` — existing interface and 3 project definitions
- `src/lib/formatDate.ts` — existing date formatting utility (use it for milestone dates)
- `src/App.tsx` — confirms lazy loading setup (do NOT modify App.tsx)
- `src/components/shared/StatusIndicator.tsx` — existing component to reuse in detail page
- `src/components/ui/badge.tsx` — shadcn Badge for tech stack tags
- `src/constants/motion.ts` — SPRING_GENTLE for collapsible animation

### Dependencies Already Installed (Do NOT Re-Install)

All required packages are in `package.json`:
- `framer-motion ^12.35.0` — AnimatePresence + motion.div for collapsible
- `react-router-dom ^7.13.1` — useParams, Link
- `lucide-react ^0.577.0` — optional icons for ↗ external links

### File Structure for Story 1.4

```
src/
├── components/
│   └── shared/
│       ├── BuildTimeline.tsx          ← NEW (AC 2)
│       └── BuildTimeline.test.tsx     ← NEW
├── constants/
│   └── projects.ts                    ← MODIFY (add timeline + lessonsLearned to interface + data)
└── pages/
    ├── ProjectDetailPage.tsx          ← MODIFY (replace stub with full implementation)
    └── ProjectDetailPage.test.tsx     ← NEW
```

### Testing Standards

Use same patterns as Story 1.3 — co-located tests, `@testing-library/react`, mock framer-motion.

**Mandatory framer-motion mock** (same pattern as Projects.test.tsx in Story 1.3):
```tsx
vi.mock('framer-motion', async (importOriginal) => {
  import type * as FramerMotion from 'framer-motion'
  const actual = await importOriginal<typeof FramerMotion>()
  return {
    ...actual,
    motion: {
      div: ({ children, ...props }: React.HTMLAttributes<HTMLDivElement>) => <div {...props}>{children}</div>,
      // add others if needed
    },
    AnimatePresence: ({ children }: { children: React.ReactNode }) => <>{children}</>,
  }
})
```

**Required tests for ProjectDetailPage.test.tsx:**
1. Renders project title for valid slug
2. Renders "Project not found." for invalid slug
3. "Back to gallery" `<Link>` renders with `to="/"` (not href)
4. BuildTimeline renders when milestones ≥ 2
5. Collapsible section is hidden by default, visible after button click

**Required tests for BuildTimeline.test.tsx:**
1. Renders correct number of milestones
2. Each milestone shows numbered badge (1-indexed), label, formatted date
3. Does not render if milestones array is empty

### Previous Story Learnings (From Story 1.3 — CRITICAL)

1. **framer-motion mock pattern** — Mock `motion.div` AND any `motion.article` used. Missing entries cause "Element type is invalid" failures. Check which `motion.*` elements are used in the component and mock ALL of them.
2. **`typeof import()` forbidden** — `@typescript-eslint/consistent-type-imports` disallows `importOriginal<typeof import('...')>()`. Use `import type * as Mod from '...'` then `importOriginal<typeof Mod>()`.
3. **ESLint `import/no-relative-parent-imports`** — all `@/` imports work. Do NOT use `../../`.
4. **shadcn generated files** — Run `npx eslint --fix` immediately after `pnpm dlx shadcn@latest add`.
5. **`react-refresh/only-export-components`** — If shadcn generates a file with non-component exports, add `// eslint-disable-next-line react-refresh/only-export-components`.
6. **No unused `_`-prefixed vars** — ESLint `argsIgnorePattern: '^_'` is already in `eslint.config.js` from Story 1.3 fix.
7. **117 tests currently pass** — Do not break them. Run `pnpm run test` before finalizing.

### Cross-Story Dependency Notes

- **Story 1.6 (i18n)** — Text in ProjectDetailPage uses hardcoded English strings. Story 1.6 will add translation keys. Do NOT add i18n in this story.
- **Story 1.8 (Spring Physics)** — Page transition animation may be added. Do NOT add page-level AnimatePresence in this story.
- **Story 3.4 (WebSocket metrics)** — `BuildStoryPreview` will be implemented in a later story (Epic 3 or standalone). Reserve the space only — do NOT fetch from GitHub API in this story.
- **BuildStoryPreview placeholder** — Architecture spec defines `src/components/shared/BuildStoryPreview.tsx` as a future component. Do NOT create this file in Story 1.4. Just reserve a `<div>` in the layout.

### Project Structure Notes

- `ProjectDetailPage.tsx` is in `src/pages/` — this is correct per architecture. It stays there.
- `BuildTimeline.tsx` goes in `src/components/shared/` — matches architecture component structure.
- No new stores or hooks needed for this story — all data comes from `projects.ts` static config + React Router's `useParams`.

### References

- Epics: [_bmad-output/planning-artifacts/epics.md](_bmad-output/planning-artifacts/epics.md) — Story 1.4 definition (lines 359–400)
- Architecture: [_bmad-output/planning-artifacts/architecture.md](_bmad-output/planning-artifacts/architecture.md) — React Router routing (lines 610–627), FE file structure (lines 1170–1282), lazy loading pattern (line 626)
- UX Spec: [_bmad-output/planning-artifacts/ux-design-specification.md](_bmad-output/planning-artifacts/ux-design-specification.md) — External link patterns (lines 1566–1580), Button hierarchy (lines 1362–1386)
- Previous story: [_bmad-output/implementation-artifacts/1-3-project-gallery-hero-section.md](_bmad-output/implementation-artifacts/1-3-project-gallery-hero-section.md) — Dev learnings, framer-motion mock pattern, ESLint fixes

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- ESLint import/order: `import ProjectDetailPage` initially placed after `vi.mock()`. Fixed by moving import to top and using simple factory mock (no `importOriginal`), matching the pattern from Projects.test.tsx (Story 1.3).
- External link `aria-label` override: Custom `aria-label` with project name broke `getByRole('link', { name: /view live/i })`. Fixed by removing redundant `aria-label` — text content "View live ↗" is the accessible name directly.

### Completion Notes List

- **Task 1**: Extended `ProjectConfig` with `lessonsLearned?: string` and `timeline?: { milestones: Array<{label, date}> }`. Populated all 3 projects with 3 milestones each (ISO dates). Added 3 shape validation tests to `projects.test.ts`.
- **Task 2**: Created `BuildTimeline.tsx` as named export. Uses `Intl.DateTimeFormat` for "MMM DD, YYYY" formatting. Accessible `<ol aria-label="Build timeline">` with numbered badges. Returns `null` when empty. 6 tests written.
- **Task 3 & 4**: Replaced `ProjectDetailPage.tsx` stub with full page — not-found state, back nav, H1 + StatusIndicator, artist statement, tech stack badges, external links (disabled pattern for building), BuildTimeline, collapsible lessonsLearned (framer-motion AnimatePresence + SPRING_GENTLE), BuildStoryPreview placeholder slot. 11 tests written.
- **Task 5**: 137 tests pass (26 test files), 0 lint errors, Vite+tsc build succeeds. `ProjectDetailPage-BNHkrb6c.js` lazy chunk: 4.90 kB (1.79 kB gzip).

### Senior Developer Review (AI)

**Review Date:** 2026-03-05
**Outcome:** Changes Requested → All Fixed

**Issues Found & Fixed (5 total):**
- 🔴 [HIGH] `formatMilestoneDate` UTC timezone bug: `new Date('YYYY-MM-DD')` + local timezone = off-by-one in UTC-N zones. Fixed: added `timeZone: 'UTC'` to `Intl.DateTimeFormat`.
- 🟡 [MEDIUM] "Live soon" badge shown for `archived` projects (portfolio-v1 has no liveUrl). Fixed: badge only shown when `project.status === 'building'`; archived projects render nothing.
- 🟢 [LOW] Collapsible `<section>` lacked accessible name. Fixed: added `aria-labelledby="lessons-heading"` + `id="lessons-heading"` on button.
- 🟢 [LOW] `key={i}` index key in BuildTimeline. Fixed: use `milestone.label` as stable key.
- 🟢 [LOW] No test for archived + no-liveUrl case. Fixed: added test `does not render "Live soon" badge for archived project without liveUrl`.

**Post-fix result:** 138/138 tests pass, 0 lint errors, build succeeds.

### File List

New files:
- portfolio-fe/src/components/shared/BuildTimeline.tsx
- portfolio-fe/src/components/shared/BuildTimeline.test.tsx
- portfolio-fe/src/pages/ProjectDetailPage.test.tsx
- portfolio-fe/src/constants/projects.test.ts (extended with 3 new shape tests)

Modified files:
- portfolio-fe/src/constants/projects.ts (extend interface + populate new fields)
- portfolio-fe/src/pages/ProjectDetailPage.tsx (replace stub with full implementation; fixed archived live-link logic; added section aria-labelledby)
- portfolio-fe/src/components/shared/BuildTimeline.tsx (added timeZone: UTC fix; key fix)

## Change Log

- 2026-03-05: Story 1.4 created by SM agent (create-story workflow)
- 2026-03-05: Story 1.4 implemented by dev agent (claude-sonnet-4-6). 137 tests pass, 0 lint errors, build succeeds. Status → review.
- 2026-03-05: Code review by AI (claude-sonnet-4-6). 5 issues found and fixed (1 HIGH, 1 MEDIUM, 3 LOW). 138 tests pass. Status → done.
