# Story 1.3: Project Gallery & Hero Section

Status: done

## Story

As a recruiter,
I want to see featured projects immediately upon landing in a visually compelling hero and scannable gallery,
so that I can quickly identify Chính's relevant work and live proof signals without scrolling through bio content.

## Acceptance Criteria

**AC1: Hero 2-Column Layout on Desktop**
```
Given the portfolio homepage loads on a viewport ≥ 1024px,
When the hero section renders,
Then it uses a CSS grid layout: 2 columns (1.2fr : 1fr), gap 48px,
  left column: EyebrowChip + display headline + subtitle + CTA row,
  right column: HeroCardStack (stacked project mini-cards with metrics),
  hero background: #050508 (deeper than bg-base),
  ambient glow: top-right radial-gradient(circle, rgba(168,85,247,.12), transparent 70%),
                bottom-left radial-gradient(circle, rgba(168,85,247,.07), transparent 70%),
  glows are CSS-only (pointer-events: none), no JavaScript required.
```

**AC2: Hero Copy Elements**
```
Given the hero section renders,
When left column displays,
Then:
  - EyebrowChip shows "⚡ Backend · Fullstack" (purple-border pill, purple-300 text)
  - Display headline uses: font-size clamp(36px, 6vw, 64px), font-weight 800, letter-spacing -0.045em
  - Key phrase in headline has gradient text: linear-gradient(135deg, #C084FC, #D8B4FE)
  - Light mode fallback for gradient text: color: #9333EA (purple-600, NOT gradient — unreadable on light)
  - Subtitle: 13px, text-secondary color, max 2 lines, max-width 380px
  - Primary CTA: "See the evidence" — purple-500 bg, glow box-shadow, scrolls to #projects
  - Secondary CTA: "Get in touch" — ghost button, rgba(255,255,255,.06) bg, scrolls to #contact
```

**AC3: HeroCardStack — Static Proof Cluster**
```
Given the hero section renders,
When right column displays the HeroCardStack,
Then:
  - Displays 3 aggregate metrics from projects.ts data (all static, no API calls):
      "N projects shipped" (count of projects with status 'live' or 'archived')
      "N-day avg. time to ship" (average of metrics.shipDays across projects with values)
      "99%+ uptime (30d)" (derived from highest uptimeDays across live projects)
  - Each metric displayed as a MetricPair (large number + small label)
  - StatusIndicator showing 'live' (green pulse dot + "Live" label)
  - Glow border default: box-shadow 0 0 18px rgba(168,85,247,.1)
  - Glow border hover: box-shadow 0 0 28px rgba(168,85,247,.25)
  - role="region" aria-label="Portfolio overview"
  - All MetricPair values have aria-label providing context
```

**AC4: Mobile Hero Layout**
```
Given viewport is < 768px,
When hero renders,
Then it collapses to single column: copy block above, HeroCardStack below (stacked).
  HeroCardStack shows a bottom fade (linear-gradient overlay) hinting at scroll.
```

**AC5: Projects Grid Responsive Layout**
```
Given the projects section renders,
When the viewport changes,
Then:
  - Desktop (≥ 1024px): 3-column grid (lg:grid-cols-3), gap 20px
  - Tablet (768px – 1023px): 2-column grid (md:grid-cols-2), gap 20px
  - Mobile (< 768px): 1-column list, gap 20px
```

**AC6: FilterChips — URL State & Referrer Init**
```
Given the projects section renders,
When FilterChips are displayed,
Then:
  - "All" chip is always first and visible; clicking clears filter (activeFilter = null)
  - Remaining chips are unique tech tags derived from projects.ts data (tags[] array union)
  - "Featured" chip is always second (shows only projects with featured: true)
  - Active chip: purple-500 bg + white text
  - Inactive chip: ghost border + muted text + hover purple tint

Given document.referrer contains "github.com",
When the gallery initializes,
Then activeFilter defaults to null ("All" tab — shows all Technical work)

Given document.referrer does NOT contain "github.com" (LinkedIn, direct, unknown),
When the gallery initializes,
Then activeFilter defaults to "featured" (Featured tab)

Given a FilterChip is clicked,
When activeFilter changes,
Then URL updates via history.pushState (format: /?filter=react or /?filter=featured or /?filter= for All)
  and the URL is shareable (paste URL → same filter state)

Given the page loads with ?filter=react in URL,
When FilterChips render,
Then "react" chip is pre-selected (read from window.location.search on mount)
```

**AC7: ProjectCard Anatomy & Interaction**
```
Given a project is rendered as a ProjectCard,
Then it shows:
  - Project title (H2, 28px weight-700)
  - StatusIndicator badge (live/building/archived)
  - Description text (≤ 80 chars displayed, truncated after 2 lines)
  - Tech stack chips: up to 5 visible (Geist Mono, 13px), overflow hidden
  - MetricPair(s): only rendered if metric value exists (e.g., shipDays, uptimeDays)
  - "View project →" link to /projects/:slug
  - Card padding: 28px internal

Given a ProjectCard is hovered (desktop),
Then:
  - Card lifts: translateY(-2px)
  - Glow intensifies: box-shadow 0 0 28px rgba(168,85,247,.25)
  - Transition: spring-snappy (stiffness 400, damping 25)

Given a ProjectCard is clicked anywhere (not just the link),
Then navigates to /projects/:slug (React Router)

Given a ProjectCard is rendered,
Then: role="article", aria-label={title}, keyboard-focusable
```

**AC8: Gallery Filter Animation & Empty State**
```
Given the activeFilter changes,
When AnimatePresence transitions the ProjectCard grid,
Then filtered-out cards animate out: opacity 0 + scale(0.95)
  filtered-in cards animate in: opacity 1 + scale(1)
  AnimatePresence mode="popLayout" wraps the grid (NOT the entire section)
  Every motion child has a stable key={project.slug}

Given the activeFilter matches zero projects,
When the empty state renders,
Then:
  - Text: "No projects match this filter."
  - Link: "Reset filter →" that clears activeFilter
  - Empty state fades in via spring-gentle after last card exits
  - Tone: direct, no apology wording
```

**AC9: Scroll Reveal Animations**
```
Given a ProjectCard scrolls into viewport,
When useInView triggers (once: true, amount: 0.15),
Then:
  - Card fades in + translateY(16px) → translateY(0) via SPRING_GENTLE
  - Cards in same row stagger with 50ms delay per card index
  - Animation runs ONCE (once: true) — no re-trigger on scroll back
  - prefers-reduced-motion: animations skip instantly (MotionConfig in App.tsx handles this)
```

**AC10: StatusIndicator Component**
```
Given a StatusIndicator is rendered with status prop,
Then:
  - 'live': 8px green (#22C55E) circle + CSS pulse animation + "Live" label
  - 'building': 8px amber (#F59E0B) circle static + "Building" label
  - 'archived': 8px gray circle static + "Archived" label
  - aria-label="Project status: live" (or building/archived)
  - Color is NEVER the sole indicator (label always present alongside color)
  - CSS pulse animation: @keyframes statusPulse in index.css
  - @media (prefers-reduced-motion: reduce) { animation: none } on pulse
```

**AC11: MetricPair Component**
```
Given a MetricPair is rendered with value and label props,
Then:
  - value displayed in large bold text with font-variant-numeric: tabular-nums
  - label displayed in small (13px) muted color text
  - "47" not "47.0" — no unnecessary decimal places
  - Component is NOT rendered if value is undefined or null (caller conditionally renders)
```

**AC12: EyebrowChip Component**
```
Given an EyebrowChip is rendered,
Then:
  - Small pill shape: border + text
  - variant="role" (hero usage): purple border + purple-300 text
  - Content ≤ 4 words (e.g., "⚡ Backend · Fullstack")
```

**AC13: projects.ts Populated with Real Data**
```
Given projects.ts is the single source of truth for static project showcase,
When the file is read,
Then:
  - ProjectConfig interface is expanded (see Dev Notes for full interface)
  - At least 3 real projects are defined with complete data
  - Each project has: slug, title, description, status, tags, featured flag, metrics
  - projects array is exported as const for type inference
```

**AC14: No Regressions & Full Build**
```
Given Story 1.3 implementation is complete,
When validation runs,
Then:
  - All existing 58 tests from Stories 1.1–1.2 still pass
  - New component tests written for ALL new components
  - pnpm run lint: 0 errors (run eslint --fix on any shadcn-generated files)
  - pnpm run build: succeeds (Vite + tsc both pass)
```

## Tasks / Subtasks

- [x] **Task 1: Install required shadcn/ui components** (AC: 7, 10)
  - [x] `pnpm dlx shadcn@latest add badge`
  - [x] `pnpm dlx shadcn@latest add skeleton`
  - [x] Run `npx eslint --fix src/components/ui/badge.tsx src/components/ui/skeleton.tsx`
  - [x] Check for `react-refresh/only-export-components` lint warnings — add `// eslint-disable-next-line` if needed

- [x] **Task 2: Expand ProjectConfig interface and populate projects.ts** (AC: 13)
  - [x] Expand `ProjectConfig` interface with: `status`, `featured`, `metrics`, `artistStatement`, `buildTimelineStart`, `buildTimelineEnd`
  - [x] Add 3 real projects with complete data (see Dev Notes for data guidance)
  - [x] Export `projects` as `const` (enables TypeScript literal inference)
  - [x] Write test `constants/projects.test.ts` — verify data shape and required fields

- [x] **Task 3: Add CSS animations for StatusIndicator to index.css** (AC: 10)
  - [x] Add `@keyframes statusPulse` in `@layer base` or raw CSS block
  - [x] Add `.status-pulse` utility or use Tailwind `animate-*` with custom keyframe
  - [x] Add `@media (prefers-reduced-motion: reduce)` rule to disable animation

- [x] **Task 4: Create EyebrowChip component** (AC: 12)
  - [x] Create `src/components/shared/EyebrowChip.tsx`
  - [x] Props: `children: React.ReactNode`, `variant?: 'role' | 'tag'`
  - [x] variant="role": purple border pill, purple-300 text
  - [x] Write `EyebrowChip.test.tsx` — renders content, applies correct styles

- [x] **Task 5: Create StatusIndicator component** (AC: 10)
  - [x] Create `src/components/shared/StatusIndicator.tsx`
  - [x] Props: `status: 'live' | 'building' | 'archived'`
  - [x] CSS pulse class on live dot (NOT Framer Motion — CSS @keyframes only)
  - [x] aria-label="Project status: {status}"
  - [x] Write `StatusIndicator.test.tsx` — renders correct label, aria-label, classes per status

- [x] **Task 6: Create MetricPair component** (AC: 11)
  - [x] Create `src/components/shared/MetricPair.tsx`
  - [x] Props: `value: number`, `label: string`
  - [x] tabular-nums on value, muted small label
  - [x] Write `MetricPair.test.tsx` — renders value and label correctly

- [x] **Task 7: Create FilterChip component** (AC: 6)
  - [x] Create `src/components/shared/FilterChip.tsx`
  - [x] Props: `label: string`, `isActive: boolean`, `onClick: () => void`
  - [x] Active: purple-500 bg + white text; Inactive: ghost border + muted
  - [x] role="button" (or use `<button>`), aria-pressed={isActive}
  - [x] Write `FilterChip.test.tsx` — active styles, click handler, aria-pressed

- [x] **Task 8: Create HeroCardStack component** (AC: 3, 4)
  - [x] Create `src/components/shared/HeroCardStack.tsx`
  - [x] Import from `@/constants/projects` — compute aggregates:
        - count: `projects.filter(p => p.status !== 'building').length`
        - avgShipDays: average of `metrics.shipDays` across projects with values
        - Use hardcoded "99%+" for uptime (real metrics come in Epic 3)
  - [x] 3 MetricPair instances (count, avg ship days, uptime placeholder)
  - [x] StatusIndicator 'live'
  - [x] Glow border + hover intensify (CSS transition, NOT Framer Motion)
  - [x] role="region" aria-label="Portfolio overview"
  - [x] Mobile: shows inside hero below copy
  - [x] Write `HeroCardStack.test.tsx` — renders metrics, accessibility

- [x] **Task 9: Create ProjectCard component** (AC: 7, 8, 9)
  - [x] Create `src/components/shared/ProjectCard.tsx`
  - [x] Props per ProjectCardProps interface (see Dev Notes)
  - [x] Use `motion.article` from framer-motion for scroll reveal (useInView from framer-motion)
  - [x] Hover: `whileHover={{ y: -2 }}` + CSS glow transition (box-shadow via className on hover)
  - [x] `initial={{ opacity: 0, y: 16 }}` → `animate={{ opacity: 1, y: 0 }}` with SPRING_GENTLE
  - [x] `transition={{ delay: index * 0.05 }}` for stagger (accept `index` prop)
  - [x] Full card clickable via `<Link to={/projects/${slug}}>` wrapping card
  - [x] "View project →" as visible link within card
  - [x] Write `ProjectCard.test.tsx` — renders all fields, accessible, link works

- [x] **Task 10: Create Hero section component** (AC: 1, 2, 4)
  - [x] Create `src/components/sections/Hero.tsx`
  - [x] 2-column CSS grid (lg only): `lg:grid-cols-[1.2fr_1fr] gap-12` (gap-12 = 48px)
  - [x] Hero bg color: add `bg-[#050508]` or use CSS variable
  - [x] Ambient glow: two `<div>` with `radial-gradient` background, `pointer-events-none absolute`
  - [x] Left: `<EyebrowChip>`, headline `<h1>` with gradient span, subtitle `<p>`, CTA `<div>` with 2 buttons
  - [x] CTAs use `<a href="#projects">` and `<a href="#contact">` (scroll anchors, no React Router)
  - [x] Right: `<HeroCardStack />`
  - [x] Mobile: `grid-cols-1` with HeroCardStack below (order via flex/grid ordering)
  - [x] Write `Hero.test.tsx` — renders eyebrow, headline, CTAs, HeroCardStack

- [x] **Task 11: Create Projects section component** (AC: 5, 6, 7, 8, 9)
  - [x] Create `src/components/sections/Projects.tsx`
  - [x] On mount:
        1. Read `window.location.search` for `?filter=` param
        2. If URL has filter → use it; else check `document.referrer`
        3. If referrer contains "github.com" → setActiveFilter(null) [All]
        4. Else → setActiveFilter('featured')
  - [x] Filter chip labels: ['All', 'Featured', ...unique tags from projects.ts sorted]
  - [x] Project filtering logic: null → all; 'featured' → p.featured; other → p.tags.includes(filter)
  - [x] On FilterChip click: setActiveFilter + history.pushState
  - [x] `<AnimatePresence mode="popLayout">` wraps card grid (children have stable key={p.slug})
  - [x] Empty state: renders inside AnimatePresence with spring-gentle fade
  - [x] Write `Projects.test.tsx` — filtering works, URL updates, empty state, referrer logic

- [x] **Task 12: Update HomePage.tsx** (AC: 1, 5)
  - [x] Replace projects placeholder with `<Hero />`
  - [x] Replace projects section placeholder with `<Projects />`
  - [x] Keep about and contact section placeholders (Story 1.5, Story 4.1)
  - [x] Update `HomePage.test.tsx` — tests still pass, hero renders, projects section renders

- [x] **Task 13: Run full validation** (AC: 14)
  - [x] `pnpm run test` — 115/115 tests pass (57 existing + 58 new, 0 regressions)
  - [x] `pnpm run lint` — 0 errors
  - [x] `pnpm run build` — succeeds (Vite + tsc, bundle 483KB/156KB gzipped)

## Dev Notes

### CRITICAL ARCHITECTURE CONSTRAINTS (Violations = Blocked PR)

1. **NO `tailwind.config.js`** — Tailwind v4 CSS-first. All new tokens (statusPulse keyframes, hero bg) go in `src/index.css` using `@layer base` or raw CSS.
2. **NO manual `localStorage`** — All persistent state via Zustand persist middleware only. galleryStore is NOT persisted (URL-based).
3. **NO `../../` relative imports** — Use `@/` alias exclusively (ESLint enforces this).
4. **Spring tokens from `@/constants/motion`** — Never inline spring values. Import `SPRING_GENTLE`, `SPRING_SNAPPY`, `SPRING_BOUNCY`.
5. **NO `tailwind.config.js` `tailwind-animate`** — Use `tw-animate-css` (Tailwind v4 compatible). StatusIndicator pulse is custom `@keyframes` in `index.css`.
6. **`<MotionConfig reducedMotion="user">` is already in App.tsx** — Do NOT add another MotionConfig. Do NOT add a custom MOTION_ENABLED context.
7. **shadcn components are owned code** — Use `pnpm dlx shadcn@latest add [component]` to copy source. Never `import from 'shadcn'`. Always run `eslint --fix` on generated files.
8. **TypeScript strict mode** — No `any`. Use proper interfaces.
9. **pnpm** — Always `pnpm`, never `npm install` or `yarn`.
10. **Gallery filter = URL param, NOT localStorage** — `galleryStore` is intentionally not persisted. Referrer detection runs fresh on each page load.
11. **Gradient text in light mode** — gradient-text on `<h1>` is invisible on light backgrounds. Use `color: #9333EA` (purple-600) as the light mode fallback. Pattern:
    ```css
    /* in component via conditional className */
    /* dark: gradient on key phrase span */
    /* light: solid purple-600 */
    ```
    Or use CSS variable approach: set background-clip text in dark, override in light with solid color.
12. **Electric Purple (#A855F7) is decorative only** — Do NOT use purple for body text on white backgrounds (3.8:1 contrast, fails WCAG AA). Text must use `text-primary` (#FAFAFA dark / #09090B light).

### Expanded ProjectConfig Interface

Update `src/constants/projects.ts` to this complete interface:

```typescript
export interface ProjectConfig {
  slug: string
  title: string
  description: string        // ≤ 80 chars for gallery card truncation
  tags: string[]             // Tech stack (used for FilterChip generation + card display)
  status: 'live' | 'building' | 'archived'
  featured?: boolean         // true = appears in "Featured" filter tab
  githubUrl?: string
  liveUrl?: string
  metrics?: {
    shipDays?: number        // Days from first commit to first production deploy
    uptimeDays?: number      // Consecutive days of uptime (for display)
  }
  artistStatement?: string   // The WHY behind the project (FR3) — used in ProjectDetailPage
  buildTimelineStart?: string // "YYYY-MM" first commit month
  buildTimelineEnd?: string   // "YYYY-MM" production deploy month
}

export const projects: ProjectConfig[] = [
  {
    slug: 'portfolio-v2',
    title: 'Portfolio v2',
    description: 'Production-ready portfolio with live metrics, i18n, and WebSocket integration.',
    tags: ['React', 'TypeScript', 'Spring Boot', 'PostgreSQL', 'WebSocket'],
    status: 'building',
    featured: true,
    githubUrl: 'https://github.com/chinh-dev11/portfolio-v2',
    metrics: {
      shipDays: 30,
    },
    artistStatement: 'Built to demonstrate production-grade engineering — not just polished UI, but live infrastructure, CI/CD, and real observability.',
    buildTimelineStart: '2026-02',
    buildTimelineEnd: undefined, // still in progress
  },
  {
    slug: 'wallet-app',
    title: 'Wallet App',
    description: 'Fintech demo app with Google OAuth2, transaction tracking, and JWT auth.',
    tags: ['React', 'Spring Boot', 'PostgreSQL', 'JWT', 'OAuth2'],
    status: 'live',
    featured: true,
    githubUrl: 'https://github.com/chinh-dev11/wallet-app',
    liveUrl: 'https://wallet.chinh.dev',
    metrics: {
      shipDays: 47,
      uptimeDays: 90,
    },
    artistStatement: 'Built to prove I can ship a real auth system with proper security — not just a demo that breaks in prod.',
    buildTimelineStart: '2025-10',
    buildTimelineEnd: '2025-12',
  },
  // Add a third project if available; minimum 2 required for story completion
]
```

> **Note to dev agent:** Replace slugs/URLs/dates with accurate real data. The `description` field must be ≤80 chars as it truncates after 2 lines in the card. Fill in `artistStatement` with honest, first-person language (no marketing fluff — see UX tone guidelines).

### ProjectCardProps Interface

```typescript
interface ProjectCardProps {
  slug: string
  title: string
  description: string
  status: 'live' | 'building' | 'archived'
  tags: string[]
  metrics?: {
    shipDays?: number
    uptimeDays?: number
  }
  featured?: boolean
  liveUrl?: string
  githubUrl?: string
  index: number              // For stagger animation delay (index * 50ms)
}
```

### Animation Patterns for This Story

**ProjectCard scroll reveal** (useInView from framer-motion):
```tsx
import { motion, useInView } from 'framer-motion'
import { useRef } from 'react'
import { SPRING_GENTLE } from '@/constants/motion'

function ProjectCard({ index, ...props }: ProjectCardProps) {
  const ref = useRef(null)
  const isInView = useInView(ref, { once: true, amount: 0.15 })

  return (
    <motion.article
      ref={ref}
      role="article"
      aria-label={props.title}
      initial={{ opacity: 0, y: 16 }}
      animate={isInView ? { opacity: 1, y: 0 } : { opacity: 0, y: 16 }}
      transition={{ ...SPRING_GENTLE, delay: index * 0.05 }}
    >
      {/* card content */}
    </motion.article>
  )
}
```

**AnimatePresence for gallery filter** — wrap only the grid items:
```tsx
import { AnimatePresence, motion } from 'framer-motion'
import { SPRING_GENTLE } from '@/constants/motion'

// In Projects.tsx:
<div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-5">
  <AnimatePresence mode="popLayout">
    {filteredProjects.map((project, i) => (
      <motion.div
        key={project.slug}
        exit={{ opacity: 0, scale: 0.95 }}
        transition={SPRING_GENTLE}
      >
        <ProjectCard {...project} index={i} />
      </motion.div>
    ))}
    {filteredProjects.length === 0 && (
      <motion.div
        key="empty-state"
        initial={{ opacity: 0 }}
        animate={{ opacity: 1 }}
        exit={{ opacity: 0 }}
        transition={SPRING_GENTLE}
        className="col-span-full text-center py-16"
      >
        <p className="text-secondary mb-2">No projects match this filter.</p>
        <button onClick={() => setFilter(null)} className="text-brand hover:underline">
          Reset filter →
        </button>
      </motion.div>
    )}
  </AnimatePresence>
</div>
```

**Hero ambient glow** (CSS only, no JS):
```tsx
// In Hero.tsx — relative container, absolute glow divs:
<section className="relative overflow-hidden bg-[#050508]">
  {/* Ambient glows — CSS only, pointer-events: none */}
  <div className="pointer-events-none absolute -top-40 right-0 h-96 w-96
                  bg-[radial-gradient(circle,rgba(168,85,247,.12),transparent_70%)]" />
  <div className="pointer-events-none absolute -bottom-40 left-0 h-96 w-96
                  bg-[radial-gradient(circle,rgba(168,85,247,.07),transparent_70%)]" />

  <div className="section-container grid grid-cols-1 lg:grid-cols-[1.2fr_1fr] gap-12 py-24">
    {/* ... */}
  </div>
</section>
```

**Gradient headline text** (dark: gradient, light: solid purple):
```tsx
// Option A: Tailwind dark: variant
<span className="
  bg-gradient-to-br from-[#C084FC] to-[#D8B4FE] bg-clip-text text-transparent
  dark:bg-gradient-to-br dark:from-[#C084FC] dark:to-[#D8B4FE] dark:text-transparent
  not-dark:text-purple-600 not-dark:bg-none not-dark:text-clip
">
  Building systems that stay live.
</span>

// Option B: CSS class in index.css (preferred for clarity):
// .gradient-text { ... } with dark mode override
```
> **Preferred approach:** Add `.gradient-headline` utility in `@layer utilities` in `index.css`:
```css
@layer utilities {
  .gradient-headline {
    background: linear-gradient(135deg, #C084FC, #D8B4FE);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    background-clip: text;
  }
}
/* Light mode override — gradient text is invisible on light backgrounds */
html:not(.dark) .gradient-headline {
  background: none;
  -webkit-background-clip: unset;
  -webkit-text-fill-color: unset;
  color: #9333EA; /* purple-600 */
}
```

### StatusIndicator CSS Pulse Animation

Add to `src/index.css` (in `@layer base` or outside layer blocks):

```css
@keyframes statusPulse {
  0%, 100% { opacity: 1; transform: scale(1); }
  50% { opacity: 0.5; transform: scale(1.4); }
}

.status-pulse {
  animation: statusPulse 2s ease-in-out infinite;
}

@media (prefers-reduced-motion: reduce) {
  .status-pulse {
    animation: none;
  }
}
```

### URL Filter State Pattern (no React Router)

```typescript
// In Projects.tsx — on mount, read URL and set initial filter:
useEffect(() => {
  const params = new URLSearchParams(window.location.search)
  const urlFilter = params.get('filter')

  if (urlFilter !== null) {
    // URL param takes priority
    setActiveFilter(urlFilter || null)
  } else {
    // No URL param — use referrer heuristic
    const referrer = document.referrer
    if (referrer.includes('github.com')) {
      setActiveFilter(null) // Technical/All default
    } else {
      setActiveFilter('featured') // Featured default (LinkedIn, direct, unknown)
    }
  }
}, []) // Run once on mount

// On FilterChip click:
const handleFilterChange = (filter: string | null) => {
  setActiveFilter(filter)
  const params = new URLSearchParams()
  if (filter) params.set('filter', filter)
  const newUrl = filter ? `?${params.toString()}` : window.location.pathname
  history.pushState(null, '', newUrl)
}
```

### HeroCardStack Metric Derivation

```typescript
// In HeroCardStack.tsx — compute from imported projects array:
import { projects } from '@/constants/projects'

const shippedProjects = projects.filter(p => p.status !== 'building')
const projectCount = shippedProjects.length

const shipDayValues = projects
  .filter(p => p.metrics?.shipDays !== undefined)
  .map(p => p.metrics!.shipDays!)
const avgShipDays = shipDayValues.length > 0
  ? Math.round(shipDayValues.reduce((a, b) => a + b, 0) / shipDayValues.length)
  : null

// uptime: use static "99%+" string (real metrics come in Epic 3, Story 3.4)
```

### File Structure for New Files

```
src/
├── components/
│   ├── sections/
│   │   ├── Hero.tsx                   ← NEW (AC 1, 2, 4)
│   │   ├── Hero.test.tsx              ← NEW
│   │   ├── Projects.tsx               ← NEW (AC 5, 6, 7, 8, 9)
│   │   └── Projects.test.tsx          ← NEW
│   └── shared/
│       ├── EyebrowChip.tsx            ← NEW (AC 12)
│       ├── EyebrowChip.test.tsx       ← NEW
│       ├── StatusIndicator.tsx        ← NEW (AC 10)
│       ├── StatusIndicator.test.tsx   ← NEW
│       ├── MetricPair.tsx             ← NEW (AC 11)
│       ├── MetricPair.test.tsx        ← NEW
│       ├── FilterChip.tsx             ← NEW (AC 6)
│       ├── FilterChip.test.tsx        ← NEW
│       ├── HeroCardStack.tsx          ← NEW (AC 3)
│       ├── HeroCardStack.test.tsx     ← NEW
│       ├── ProjectCard.tsx            ← NEW (AC 7)
│       └── ProjectCard.test.tsx       ← NEW
├── constants/
│   └── projects.ts                    ← MODIFY (expand interface + populate data)
├── pages/
│   ├── HomePage.tsx                   ← MODIFY (replace placeholders)
│   └── HomePage.test.tsx              ← MODIFY (update tests)
└── index.css                          ← MODIFY (statusPulse keyframes, gradient-headline)
```

> **New `components/shared/` directory** — does not exist yet. Create it.
> **New `components/sections/` directory** — does not exist yet. Create it.

### Previous Story Learnings (CRITICAL — From Story 1.2)

1. **vitest.config.ts already has `plugins: [react()]`** — fixed in Story 1.2. Do NOT re-add it.
2. **ESLint `import/no-relative-parent-imports`** — `ignore: ['^@/']` already configured in `eslint.config.js`. All `@/` imports work. Do NOT use `../../` relative parent imports.
3. **shadcn-generated files have `import/order` errors** — Run `npx eslint --fix src/components/ui/badge.tsx src/components/ui/skeleton.tsx` immediately after generating.
4. **`react-refresh/only-export-components`** — shadcn may export non-component symbols. Add `// eslint-disable-next-line react-refresh/only-export-components` above offending exports in generated files.
5. **Fontsource type declarations** — `src/types/fontsource.d.ts` already exists and covers both geist packages. No new declarations needed for this story.
6. **`@tailwindcss/vite` plugin in vitest.config.ts** — already added in Story 1.2 code review. Present in vitest.config.ts.

### Dependencies Already Installed (Do NOT Re-Install)

All required packages are already in package.json:
- `framer-motion ^12.35.0` — for motion.article, AnimatePresence, useInView
- `zustand ^5.0.11` — for galleryStore, referralStore
- `react-router-dom ^7.13.1` — for `<Link>` in ProjectCard
- `lucide-react ^0.577.0` — for any icons (if needed in components)
- `tailwindcss ^4.2.1` — CSS-first, all tokens in index.css

### Existing Files to Read Before Starting

Read these files before implementing to avoid duplicate work / regressions:
- `src/index.css` — existing design tokens (do NOT duplicate, build on existing `@theme {}`)
- `src/constants/motion.ts` — SPRING_GENTLE/SNAPPY/BOUNCY definitions
- `src/stores/galleryStore.ts` — existing activeFilter store (just needs URL sync in component)
- `src/stores/referralStore.ts` — existing referral source store (read-only in this story)
- `src/pages/HomePage.tsx` — current placeholder structure to replace
- `src/App.tsx` — MotionConfig already in place, Router already configured

### Testing Standards for New Components

- All test files co-located with their component (`ComponentName.test.tsx` next to `ComponentName.tsx`)
- Use `@testing-library/react` + `@testing-library/user-event`
- Coverage threshold ≥60% enforced for `src/lib/**`, `src/hooks/**`, `src/stores/**` — NOT for components (but write meaningful tests)
- Recommended tests per component:
  - `EyebrowChip`: renders children, applies purple border class
  - `StatusIndicator`: correct label per status, aria-label, pulse class on 'live'
  - `MetricPair`: renders value and label
  - `FilterChip`: active/inactive styles, onClick fires, aria-pressed={isActive}
  - `ProjectCard`: renders title, description, status, tags, link to /projects/:slug
  - `HeroCardStack`: renders metrics, role="region", aria-label
  - `Hero`: renders eyebrow, h1, CTAs with correct href
  - `Projects`: filters projects correctly, renders empty state, updates URL

### Cross-Story Dependency Notes

- **Story 3.4** will overlay real-time metrics onto the same ProjectCard components built here. Do NOT add WebSocket logic in this story — keep ProjectCard's `metrics` prop as optional static data.
- **Story 1.4** (Project Detail Page) will use the `slug` from projects.ts to render `/projects/:slug`. The `ProjectDetailPage.tsx` stub already exists.
- **Story 1.5** (About/Skills) will replace the About section placeholder. Do NOT build About in this story.
- **Story 1.6** (i18n) will add translation keys. Text content in `en.json`/`vi.json` are currently empty `{}`. Do NOT add i18n in this story — use hardcoded English strings (Story 1.6 will refactor).

### Project Structure Notes

- **Alignment with architecture:** `components/sections/` and `components/shared/` match the architecture spec exactly. These directories are new — create them.
- **galleryStore** (`src/stores/galleryStore.ts`) is already implemented with `activeFilter` + `setActiveFilter`. Extend functionality via component-level URL sync (see URL Filter State Pattern above) — do NOT modify the store's persistence config.
- **referralStore** is NOT used in this story directly. Referrer detection is done via `document.referrer` inline in the Projects component on mount.
- **metricsStore** — do NOT integrate WebSocket in this story. HeroCardStack and ProjectCard use static data from projects.ts only.

### References

- Architecture: `_bmad-output/planning-artifacts/architecture.md` — Component structure, animation patterns, Zustand patterns, Tailwind v4 CSS-first, testing standards
- UX Spec: `_bmad-output/planning-artifacts/ux-design-specification.md` — Hero layout, ProjectCard anatomy, FilterChip behavior, color tokens, typography scale, motion tokens
- Epics: `_bmad-output/planning-artifacts/epics.md` — Story 1.3 user story, FR coverage (FR1, FR2, FR3, FR7)
- Previous story: `_bmad-output/implementation-artifacts/1-2-app-shell-design-system-theme-toggle.md` — Dev learnings

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- `badge.tsx` shadcn-generated: `react-refresh/only-export-components` on `badgeVariants` export → added `// eslint-disable-next-line` comment
- `Projects.test.tsx`: framer-motion mock missing `motion.article` → all 10 tests failed → added `motion.article` to mock
- ESLint `@typescript-eslint/no-unused-vars` flagging `_`-prefixed destructured params → added `argsIgnorePattern: '^_'` and `varsIgnorePattern: '^_'` to eslint.config.js
- `Projects.test.tsx` unused `user` variable in async test → refactored test to sync (no userEvent.setup() needed)

### Completion Notes List

All 13 tasks + code review fixes completed. 117/117 tests pass (57 pre-existing + 60 new). 0 lint errors. Build succeeds.

Key implementation decisions:
- `gradient-headline` CSS utility added to `index.css @layer utilities` with light mode override in `html:not(.dark)` selector
- `statusPulse` CSS `@keyframes` added as raw CSS (outside layer blocks) with `@media prefers-reduced-motion` disable
- `HeroCardStack` metrics computed from `projects.ts` at runtime (static, no API) — avg shipDays from all projects with values, uptime hardcoded "99%+" until Epic 3 Story 3.4
- `Projects` referrer detection and URL sync handled in component `useEffect` on mount — galleryStore unchanged (still in-memory only)
- `FilterChip` uses `<button>` with `aria-pressed` (not `role="group"` on the chip itself — `role="group"` is on the container in Projects.tsx)
- ESLint config updated with `argsIgnorePattern: '^_'` — standard TypeScript convention, should have been there from start
- framer-motion mocked in test files with typed `stripMotionProps` helper for ProjectCard/Projects/HomePage tests

Code review fixes applied (2026-03-05):
- ProjectCard: Added full-card click navigation via `useNavigate` (AC7 fix)
- ProjectCard: Added `SPRING_SNAPPY` to `whileHover` transition (AC7 fix)
- Hero h1: Replaced inline `style` with `text-[var(--text-display)]` CSS token classes (design system consistency)
- MetricPair: Added `aria-label={value + label}` to container (AC3 fix)
- projects.ts: Removed redundant `buildTimelineEnd: undefined`
- Projects.tsx: Removed dead `typeof document !== 'undefined'` SSR guard

### File List

New files created:
- portfolio-fe/src/components/shared/EyebrowChip.tsx
- portfolio-fe/src/components/shared/EyebrowChip.test.tsx
- portfolio-fe/src/components/shared/StatusIndicator.tsx
- portfolio-fe/src/components/shared/StatusIndicator.test.tsx
- portfolio-fe/src/components/shared/MetricPair.tsx
- portfolio-fe/src/components/shared/MetricPair.test.tsx
- portfolio-fe/src/components/shared/FilterChip.tsx
- portfolio-fe/src/components/shared/FilterChip.test.tsx
- portfolio-fe/src/components/shared/HeroCardStack.tsx
- portfolio-fe/src/components/shared/HeroCardStack.test.tsx
- portfolio-fe/src/components/shared/ProjectCard.tsx
- portfolio-fe/src/components/shared/ProjectCard.test.tsx
- portfolio-fe/src/components/sections/Hero.tsx
- portfolio-fe/src/components/sections/Hero.test.tsx
- portfolio-fe/src/components/sections/Projects.tsx
- portfolio-fe/src/components/sections/Projects.test.tsx
- portfolio-fe/src/components/ui/badge.tsx
- portfolio-fe/src/components/ui/skeleton.tsx
- portfolio-fe/src/constants/projects.test.ts

Files modified:
- portfolio-fe/src/constants/projects.ts (expanded interface + 3 real projects)
- portfolio-fe/src/index.css (gradient-headline utility + statusPulse keyframes)
- portfolio-fe/src/pages/HomePage.tsx (Hero + Projects sections wired)
- portfolio-fe/src/pages/HomePage.test.tsx (updated for new sections)
- portfolio-fe/eslint.config.js (added @typescript-eslint/no-unused-vars argsIgnorePattern)

## Change Log

- 2026-03-05: Story 1.3 created by SM agent (create-story workflow)
- 2026-03-05: Story 1.3 implemented by claude-sonnet-4-6 — all 13 tasks complete, 115/115 tests pass, 0 lint errors, build succeeds → status: review
- 2026-03-05: Code review by claude-sonnet-4-6 — 1 HIGH + 3 MEDIUM + 2 LOW issues found and auto-fixed. 117/117 tests pass, 0 lint errors → status: done
