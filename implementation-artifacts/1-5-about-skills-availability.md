# Story 1.5: About, Skills & Availability

Status: done

## Story

As a recruiter,
I want to understand Chính's background, verified skills, and current availability,
so that I can assess fit before deciding to reach out.

## Acceptance Criteria

**AC1: "Why Hire Me" Paragraph**
```
Given I am a recruiter viewing the portfolio,
When I scroll to the About / Skills section,
Then I see a concise "Why Hire Me" paragraph (3–4 sentences) written in
  recruiter-centric framing — emphasizing engineering philosophy and judgment,
  not CV bullet points.
```

**AC2: Versioned Tech Badges as Proof**
```
Given I am a recruiter viewing any project card or project detail page,
When I look at the technology stack on the About section,
Then I see versioned tech badges (e.g. "React 18", "Spring Boot 3.4")
  that serve as live proof of skill — each badge links to the corresponding
  live project or GitHub repo as the primary proof artifact.
```

**AC3: Skills in Project Context (No Isolated Skill List)**
```
Given I am a recruiter viewing the About section,
When I look at the skills displayed,
Then skills are presented in context of real projects — no isolated skill list
  without proof; every technology mentioned is linked to at least one project
  that demonstrates it (via slug reference to projects.ts).
```

**AC4: Job Availability Badge**
```
Given I am a recruiter viewing the portfolio,
When I check the job availability section,
Then I see a Job Availability Badge displaying:
  - Current status: one of "Open to Work" | "Selectively Looking" | "Not Available"
  - Location preference (e.g. "Remote · Ho Chi Minh City")
  - Badge color per status:
      "Open to Work"       → Green  (#22C55E)
      "Selectively Looking"→ Yellow (#EAB308)
      "Not Available"      → Gray   (#6B7280)
  - Data source: read from src/constants/about.ts → availability: { status, location }
```

**AC5: Mobile Responsive**
```
Given I am a recruiter viewing the portfolio on mobile,
When the About/Skills section renders at 320px viewport width,
Then the section is fully readable and badges are legible with no horizontal scroll.
```

**AC6: No Regressions & Full Build**
```
Given Story 1.5 implementation is complete,
When validation runs,
Then:
  - All existing 138 tests from Stories 1.1–1.4 still pass
  - New component tests written for About.tsx
  - pnpm run lint: 0 errors
  - pnpm run build: succeeds (Vite + tsc both pass)
```

## Tasks / Subtasks

- [x] **Task 1: Create `src/constants/about.ts` config** (AC: 1, 3, 4)
  - [x] Define and export `AboutConfig` interface with fields: `whyHireMe: string`, `skills: SkillWithProof[]`, `availability: { status: 'open' | 'selective' | 'unavailable'; location: string }`
  - [x] Define `SkillWithProof` interface: `{ tech: string; version: string; projectSlug: string }` (links to projects.ts by slug)
  - [x] Populate `aboutContent` export with real data (3–4 sentence "Why Hire Me" text + skills list referencing existing project slugs)
  - [x] Write `about.test.ts` — verify shape of exported object (status is valid enum, skills have projectSlug, whyHireMe is non-empty string)

- [x] **Task 2: Create `src/components/sections/About.tsx`** (AC: 1, 2, 3, 4, 5)
  - [x] Import `aboutContent` from `@/constants/about`; import `projects` from `@/constants/projects` for slug → liveUrl/githubUrl resolution
  - [x] Render section with `id="about"` and `section-container py-24` classes
  - [x] Render H2 section heading (e.g. "Why Hire Me")
  - [x] Render "Why Hire Me" paragraph: `<p>{aboutContent.whyHireMe}</p>`
  - [x] Render Skills-with-proof: For each skill in `aboutContent.skills`, render a `<Badge>` (shadcn) showing `tech version` (e.g. "React 18"), wrapped in an `<a>` linking to `project.liveUrl ?? project.githubUrl` (resolved from projects by slug)
  - [x] Render Job Availability Badge:
    - Map `availability.status` to: label ("Open to Work" / "Selectively Looking" / "Not Available") and color class
    - Render badge using inline style for data-driven hex colors (NOT Tailwind color utilities)
    - Render `availability.location` below/next to the badge
  - [x] Layout: centered single-column, `max-w-2xl mx-auto` (per UX spec for About section)
  - [x] Mobile-first: all styles base (mobile), override with `md:` / `lg:` where needed
  - [x] Add `aria-label` on skills group (e.g. `aria-label="Verified skills with proof"`)

- [x] **Task 3: Write `src/components/sections/About.test.tsx`** (AC: 1–5)
  - [x] Renders "Why Hire Me" heading and paragraph text
  - [x] Renders correct number of skill badges from `aboutContent.skills`
  - [x] Each skill badge renders as a link (`<a>`) with href pointing to project URL
  - [x] Renders job availability status label correctly (e.g. "Open to Work")
  - [x] Renders location string
  - [x] Section has `id="about"` for anchor navigation

- [x] **Task 4: Wire About section into `HomePage.tsx`** (AC: 1–5)
  - [x] Replace placeholder `<section id="about" ...>` with `<About />`
  - [x] Add import: `import { About } from '@/components/sections/About'`
  - [x] Do NOT modify Nav.tsx, Hero.tsx, Projects.tsx, App.tsx

- [x] **Task 5: Full validation** (AC: 6)
  - [x] `pnpm run test` — 154/154 tests pass (138 existing + 16 new, 0 regressions)
  - [x] `pnpm run lint` — 0 errors
  - [x] `pnpm run build` — succeeds (Vite + tsc)

## Dev Notes

### CRITICAL ARCHITECTURE CONSTRAINTS (Same as Stories 1.3 & 1.4 — Do Not Violate)

1. **NO `tailwind.config.js`** — Tailwind v4 CSS-first. All new design tokens in `src/index.css`.
2. **NO manual `localStorage`** — Zustand persist middleware only.
3. **NO `../../` relative imports** — Use `@/` alias exclusively.
4. **Spring tokens from `@/constants/motion`** — Never inline spring values.
5. **`<MotionConfig reducedMotion="user">` is already in App.tsx** — Do NOT add another MotionConfig.
6. **shadcn components are owned code** — Use `pnpm dlx shadcn@latest add [component]` to copy source. Always run `eslint --fix` on generated files immediately.
7. **TypeScript strict mode** — No `any`. Use explicit types.
8. **pnpm** — Always `pnpm`, never `npm install` or `yarn`.
9. **No Framer Motion in this story** — About section is static content, no animations required. Do NOT add motion wrappers unless the component becomes genuinely interactive.

### Config File: `src/constants/about.ts`

> **Note:** Epics spec says `src/config/about.ts`, but the architecture doesn't define a `config/` folder. Use `src/constants/about.ts` to align with existing `constants/projects.ts` pattern. This keeps all static data in one place.

```typescript
// src/constants/about.ts

export type AvailabilityStatus = 'open' | 'selective' | 'unavailable'

export interface SkillWithProof {
  tech: string         // e.g. "React"
  version: string      // e.g. "18" → renders as "React 18"
  projectSlug: string  // must match a slug in projects.ts
}

export interface AboutConfig {
  whyHireMe: string              // 3–4 sentences, recruiter-centric
  skills: SkillWithProof[]
  availability: {
    status: AvailabilityStatus
    location: string             // e.g. "Remote · Ho Chi Minh City"
  }
}

export const aboutContent: AboutConfig = {
  whyHireMe:
    'I build systems that are meant to stay running — not just demos that survive a presentation. ' +
    'My engineering philosophy is to make the invisible visible: instrumentation, CI/CD guardrails, and honest documentation of every architectural decision, including the wrong ones. ' +
    'I treat each project as a production system from day one, which means you get a developer who ships, monitors, and improves — not one who hands off and moves on.',
  skills: [
    { tech: 'React', version: '18', projectSlug: 'portfolio-v2' },
    { tech: 'TypeScript', version: '5', projectSlug: 'portfolio-v2' },
    { tech: 'Spring Boot', version: '3.4', projectSlug: 'portfolio-v2' },
    { tech: 'PostgreSQL', version: '16', projectSlug: 'portfolio-v2' },
    { tech: 'Java', version: '21', projectSlug: 'portfolio-v2' },
  ],
  availability: {
    status: 'open',
    location: 'Remote · Ho Chi Minh City',
  },
}
```

> **Adjust `whyHireMe` text and skills list as appropriate.** The slug `'portfolio-v2'` references the existing project in `projects.ts`. Add `wallet-app` slug references when that project has a `liveUrl` or `githubUrl` defined.

### About.tsx Component Spec

```tsx
// src/components/sections/About.tsx

import { aboutContent } from '@/constants/about'
import { projects } from '@/constants/projects'
import { Badge } from '@/components/ui/badge'

const STATUS_LABEL: Record<string, string> = {
  open: 'Open to Work',
  selective: 'Selectively Looking',
  unavailable: 'Not Available',
}

// Use inline style for semantic status colors (not Tailwind design tokens):
const STATUS_COLOR: Record<string, string> = {
  open: '#22C55E',      // Green — per epics spec
  selective: '#EAB308', // Yellow
  unavailable: '#6B7280', // Gray
}

export function About() {
  const { whyHireMe, skills, availability } = aboutContent

  return (
    <section id="about" className="section-container py-24">
      <div className="max-w-2xl mx-auto space-y-8">
        {/* Why Hire Me */}
        <div>
          <h2 className="text-2xl font-semibold mb-4">Why Hire Me</h2>
          <p className="text-muted-foreground leading-relaxed">{whyHireMe}</p>
        </div>

        {/* Skills with proof */}
        <div>
          <h3 className="text-lg font-medium mb-3">Verified Skills</h3>
          <div
            className="flex flex-wrap gap-2"
            role="list"
            aria-label="Verified skills with proof"
          >
            {skills.map((skill) => {
              const project = projects.find((p) => p.slug === skill.projectSlug)
              const href = project?.liveUrl ?? project?.githubUrl ?? '#'
              return (
                <a
                  key={`${skill.tech}-${skill.version}`}
                  href={href}
                  target="_blank"
                  rel="noopener noreferrer"
                  role="listitem"
                  aria-label={`${skill.tech} ${skill.version} — view proof project`}
                >
                  <Badge variant="secondary">
                    {skill.tech} {skill.version}
                  </Badge>
                </a>
              )
            })}
          </div>
        </div>

        {/* Availability Badge */}
        <div>
          <h3 className="text-lg font-medium mb-3">Availability</h3>
          <div className="flex items-center gap-3">
            <span
              className="inline-flex items-center gap-1.5 rounded-full px-3 py-1 text-sm font-medium text-white"
              style={{ backgroundColor: STATUS_COLOR[availability.status] }}
              aria-label={`Job availability: ${STATUS_LABEL[availability.status]}`}
            >
              <span className="h-2 w-2 rounded-full bg-white opacity-80" aria-hidden="true" />
              {STATUS_LABEL[availability.status]}
            </span>
            <span className="text-muted-foreground text-sm">{availability.location}</span>
          </div>
        </div>
      </div>
    </section>
  )
}
```

> **Do NOT use Tailwind color utilities for the status badge** (e.g. `bg-green-500`) because:
> 1. The exact hex colors are specified in the epic (#22C55E, #EAB308, #6B7280)
> 2. Tailwind v4 CSS-first — custom semantic colors would require adding to `index.css`
> 3. Inline `style` is acceptable for dynamic, data-driven colors

### About.test.tsx Pattern

```tsx
// src/components/sections/About.test.tsx
import { render, screen } from '@testing-library/react'
import { describe, it, expect } from 'vitest'
import { About } from '@/components/sections/About'
import { aboutContent } from '@/constants/about'
import { projects } from '@/constants/projects'

describe('About', () => {
  it('renders the Why Hire Me heading', () => {
    render(<About />)
    expect(screen.getByRole('heading', { name: /why hire me/i })).toBeInTheDocument()
  })

  it('renders whyHireMe paragraph text', () => {
    render(<About />)
    // Check a substring of the text (not full string — avoids brittle exact matches)
    expect(screen.getByText(/build systems/i)).toBeInTheDocument()
  })

  it('renders correct number of skill badges', () => {
    render(<About />)
    const skillLinks = screen.getAllByRole('listitem')
    expect(skillLinks).toHaveLength(aboutContent.skills.length)
  })

  it('each skill badge is a link to the proof project', () => {
    render(<About />)
    const links = screen.getAllByRole('link')
    // At least one link per skill
    expect(links.length).toBeGreaterThanOrEqual(aboutContent.skills.length)
    links.forEach((link) => {
      expect(link).toHaveAttribute('href')
      expect(link).toHaveAttribute('target', '_blank')
    })
  })

  it('renders availability status label', () => {
    render(<About />)
    // 'open' → 'Open to Work'
    expect(screen.getByText(/open to work/i)).toBeInTheDocument()
  })

  it('renders location string', () => {
    render(<About />)
    expect(screen.getByText(aboutContent.availability.location)).toBeInTheDocument()
  })

  it('section has id="about" for anchor navigation', () => {
    const { container } = render(<About />)
    expect(container.querySelector('#about')).not.toBeNull()
  })
})
```

### File Structure for Story 1.5

```
src/
├── constants/
│   ├── about.ts               ← NEW (config: whyHireMe, skills, availability)
│   └── about.test.ts          ← NEW (shape validation)
├── components/
│   └── sections/
│       ├── About.tsx          ← NEW (section component)
│       └── About.test.tsx     ← NEW (component tests)
└── pages/
    └── HomePage.tsx           ← MODIFY (replace placeholder with <About />)
```

### Existing Files to Read Before Starting

- `src/pages/HomePage.tsx` — contains About placeholder at line 14–17
- `src/constants/projects.ts` — `ProjectConfig` interface and slug names (use slugs in `aboutContent.skills`)
- `src/components/ui/badge.tsx` — shadcn Badge component (already installed from Story 1.3)
- `src/index.css` — existing design tokens (`section-container`, `text-muted-foreground`, etc.)
- `src/components/sections/Hero.tsx` — reference for section structure pattern
- `src/components/sections/Projects.tsx` — reference for section structure pattern

### Dependencies Already Installed (Do NOT Re-Install)

All required packages exist in `package.json`:
- `@/components/ui/badge` — shadcn Badge already copied from Story 1.3
- No new packages required for this story

### Testing Standards

Same patterns as Stories 1.3 & 1.4:
- Co-located test files
- `@testing-library/react` + `vitest`
- **No framer-motion mock needed** — About.tsx has no animations

**Mock `@/constants/about` if needed** for isolation, but generally importing the real module is fine since it's pure static data with no side effects.

### Previous Story Learnings (From Stories 1.3 & 1.4 — CRITICAL)

1. **framer-motion mock** — Not needed for this story (no animations). If you add any `motion.*` for future reasons, mock `motion.div` and ALL used motion elements.
2. **`typeof import()` forbidden** — `@typescript-eslint/consistent-type-imports` rule. Use `import type * as Mod from '...'` pattern. Not needed here unless mocking.
3. **ESLint `import/no-relative-parent-imports`** — Use `@/` always. Never `../../`.
4. **shadcn files** — `badge.tsx` is already installed. Do NOT run shadcn CLI again for Badge. Check `src/components/ui/` first.
5. **`aria-label` overrides accessible name** — If `getByRole('link', { name: /view live/i })` fails in tests, the element has an `aria-label` that overrides text content. Match the full aria-label or design labels consistently.
6. **138 tests currently pass** — Run `pnpm run test` before finalizing. Zero regressions.
7. **`pnpm run lint` before submitting** — `import/order` violations are the most common issue with new files.

### Cross-Story Dependency Notes

- **Story 1.6 (i18n)** — "Why Hire Me" text and availability labels use hardcoded English strings. Story 1.6 will add translation keys. Do NOT add i18n in this story.
- **Story 1.7.x (Accessibility)** — Ensure ARIA labels on skills group and availability badge are correct from the start. This story lays the groundwork.
- **Story 3.4 / 3.6 (Live Metrics)** — GitHub contribution graph will be added to About or a new section later. Reserve no space for it here — the About section is self-contained.
- **`src/config/` folder** — Epics says `src/config/about.ts`; architecture has no `config/` folder. **Use `src/constants/about.ts`** to match existing constants pattern. If Story 1.6 or later creates a `config/` directory, this file can be moved at that time.

### Project Structure Notes

- `About.tsx` lives in `src/components/sections/` — matches architecture diagram exactly (line 1217 of architecture.md)
- `about.ts` lives in `src/constants/` — extends existing pattern (alongside `projects.ts`, `motion.ts`, `config.ts`)
- No new stores, hooks, or pages needed for this story — pure static content component

### References

- Epics: [_bmad-output/planning-artifacts/epics.md](_bmad-output/planning-artifacts/epics.md) — Story 1.5 definition (lines 402–433)
- Architecture: [_bmad-output/planning-artifacts/architecture.md](_bmad-output/planning-artifacts/architecture.md) — FE file structure (lines 800–817, 1214–1218), About section layout (line 1217)
- UX Spec: [_bmad-output/planning-artifacts/ux-design-specification.md](_bmad-output/planning-artifacts/ux-design-specification.md) — About/Contact layout (line 1609: centered single-column `max-w-2xl mx-auto`)
- Previous story: [_bmad-output/implementation-artifacts/1-4-project-detail-page.md](_bmad-output/implementation-artifacts/1-4-project-detail-page.md) — Dev learnings, ESLint patterns, test patterns

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- **ESLint `import/order`**: `About.tsx` had Badge import after constants imports. `About.test.tsx` and `about.test.ts` were missing empty lines between import groups. Fixed via `npx eslint --fix`.
- **`role="listitem"` on `<a>` overrides link role**: Initial implementation used `role="listitem"` on `<a>` elements, which made `getAllByRole('link')` return 0 results. Fixed by switching to `<ul>/<li>` structure — `<li>` carries listitem role, `<a>` retains implicit link role.
- **[Code Review] `react-refresh/only-export-components`**: Exporting `STATUS_LABEL` from `About.tsx` triggered fast-refresh lint error (non-component export from component file). Fixed by moving `STATUS_LABEL` and `STATUS_COLOR` to `about.ts` constants file — cleaner architecture, both testable and importable by the component.

### Completion Notes List

- **Task 1**: Created `src/constants/about.ts` with `AvailabilityStatus` type, `SkillWithProof` and `AboutConfig` interfaces, and populated `aboutContent` (3-sentence "Why Hire Me" + 5 skills linked to `wallet-app`/`portfolio-v2` slugs + `open` availability status). 5 shape-validation tests written.
- **Task 2**: Created `src/components/sections/About.tsx` — "Why Hire Me" H2 + paragraph, verified skills `<ul>/<li>/<a>/<Badge>` list, availability badge with inline hex color (data-driven, not Tailwind utilities). No framer-motion used. Layout: `max-w-2xl mx-auto`, mobile-first.
- **Task 3**: 11 tests cover all ACs: heading renders, paragraph text, correct badge count, link attributes (target/rel/href), availability label + location, section anchor ID, and slug integrity check against projects.ts.
- **Task 4**: Replaced `HomePage.tsx` placeholder `<section>` with `<About />` import. No other files modified.
- **Task 5**: 154/154 tests pass (138 existing + 16 new), 0 lint errors, Vite+tsc build succeeds.

### Senior Developer Review (AI)

**Review Date:** 2026-03-05
**Outcome:** Changes Requested → All Fixed

**Issues Found & Fixed (7 total):**
- 🔴 [HIGH] `Record<string, string>` instead of `Record<AvailabilityStatus, string>` for STATUS_LABEL/STATUS_COLOR — TypeScript couldn't catch invalid status keys. Fixed: typed with `AvailabilityStatus`.
- 🟡 [MEDIUM] No runtime fallback for undefined status color — `backgroundColor: undefined` rendered transparent badge silently. Fixed: `?? '#6B7280'` fallback added.
- 🟡 [MEDIUM] `list-none` CSS removes list semantics in Safari+VoiceOver without explicit `role="list"`. Fixed: added `role="list"` on `<ul>`.
- 🟢 [LOW] No test for `selective`/`unavailable` status labels. Fixed: added `STATUS_LABEL maps all availability statuses correctly` test.
- 🟢 [LOW] `whyHireMe` tested with only a short substring — no minimum length guard. Fixed: added `whyHireMe text meets minimum length requirement` assertion.
- 🟢 [LOW] `about.test.ts` didn't validate slugs against `projects.ts`. Fixed: added `each skill projectSlug resolves to an existing project` test.
- 🟢 [LOW] `STATUS_LABEL`/`STATUS_COLOR` not accessible for tests. Fixed: moved to `about.ts` (also avoids `react-refresh/only-export-components` violation).

**Post-fix result:** 157/157 tests pass, 0 lint errors, build succeeds.

### File List

New files:
- portfolio-fe/src/constants/about.ts
- portfolio-fe/src/constants/about.test.ts
- portfolio-fe/src/components/sections/About.tsx
- portfolio-fe/src/components/sections/About.test.tsx

Modified files:
- portfolio-fe/src/pages/HomePage.tsx (replace About placeholder with `<About />` component)

## Change Log

- 2026-03-05: Story 1.5 created by SM agent (create-story workflow)
- 2026-03-05: Story 1.5 implemented by dev agent (claude-sonnet-4-6). 154 tests pass, 0 lint errors, build succeeds. Status → review.
- 2026-03-05: Code review by AI (claude-sonnet-4-6). 7 issues found and fixed (1 HIGH, 2 MEDIUM, 4 LOW). STATUS_LABEL/STATUS_COLOR moved to about.ts, role="list" added, fallback color added, 3 new tests added. 157 tests pass. Status → done.
