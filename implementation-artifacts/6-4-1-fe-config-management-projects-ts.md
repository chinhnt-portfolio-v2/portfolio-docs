# Story 6.4.1: FE Config Management (projects.ts)

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

**As the** portfolio owner,
**I want** a typed config file to manage all project showcase data,
**so that** I can update projects, metadata, and artist statements via a git push with no code changes.

---

## 1. Acceptance Criteria

**AC-1 — projects.ts config drives gallery automatically**

Given `src/config/projects.ts` exists with TypeScript interfaces,
When I add a new project entry to the config,
Then the portfolio gallery automatically includes it on next deploy with no component code changes.

**AC-2 — Config schema completeness**

Config schema includes: `slug`, `name`, `description`, `techStack[]`, `demoUrl`, `repoUrl`, `artistStatement`, `timeline.milestones[]`, `hasBuildStory`, `availability` (for About section).

**AC-3 — about.ts availability config**

Given `src/config/about.ts` exists,
When I update `availability.status` or `availability.location`,
Then the job availability badge on the portfolio reflects the change on next deploy.

---

## 2. Tasks / Subtasks

### Task 1: Define TypeScript interfaces for project config — AC: #1, #2

- [ ] Create `src/config/projects.ts` file
- [ ] Define `Project` interface with all required fields: `slug`, `name`, `description`, `techStack[]`, `demoUrl`, `repoUrl`, `artistStatement`, `timeline.milestones[]`, `hasBuildStory`, `availability`
- [ ] Define `TimelineMilestone` interface: `date`, `title`, `description`
- [ ] Define `ProjectAvailability` enum or type: `available`, `not-available`, `open-to-roles`
- [ ] Export typed `projects` array constant (initially empty or with placeholder entries)
- [ ] Export `getProjectBySlug(slug: string): Project | undefined` helper
- [ ] Export `getAllProjects(): Project[]` helper
- [ ] Export `getAvailableProjects(): Project[]` helper (filters by availability)

### Task 2: Integrate config into Gallery component — AC: #1

- [ ] Read existing gallery component (`ProjectGallery` or similar in `src/components/`)
- [ ] Replace hardcoded project data with import from `src/config/projects.ts`
- [ ] Verify gallery renders all projects from config without component changes for new entries
- [ ] Ensure slug-based routing (`/projects/:slug`) still works with config slugs

### Task 3: Define TypeScript interfaces for about/availability config — AC: #3

- [ ] Create `src/config/about.ts` file
- [ ] Define `Availability` type with `status` and `location` fields
- [ ] Define `AboutConfig` interface composing availability and other about-section fields
- [ ] Export typed `aboutConfig` constant
- [ ] Export `getAvailability(): Availability` helper

### Task 4: Integrate about config into About component — AC: #3

- [ ] Read existing About component (`AboutSection` or similar)
- [ ] Replace hardcoded availability data with import from `src/config/about.ts`
- [ ] Verify availability badge updates automatically when config changes

### Task 5: Unit Tests — AC: all

- [ ] Test `getProjectBySlug` returns correct project or undefined
- [ ] Test `getAllProjects` returns all projects
- [ ] Test `getAvailableProjects` filters correctly
- [ ] Test `getAvailability` returns correct status and location
- [ ] Test gallery component renders projects from config (shallow mount with config mock)

---

## 3. Dev Notes

### Project Structure Notes

**Location:** `portfolio-fe/src/config/`

```
src/config/
├── projects.ts      ← NEW: Typed project showcase config
├── about.ts         ← NEW: Typed about/availability config
└── index.ts         ← OPTIONAL: barrel export

src/components/
├── gallery/         ← Existing: update to import from config
│   ├── ProjectGallery.tsx
│   └── ProjectCard.tsx
└── about/
    ├── AboutSection.tsx   ← Existing: update to import from config
    └── AvailabilityBadge.tsx  ← Existing: refactor to use config
```

**Config file path:** `portfolio-fe/src/config/projects.ts`
**About config path:** `portfolio-fe/src/config/about.ts`

### Key Implementation Details

**TypeScript interfaces:**
```typescript
// projects.ts
export interface TimelineMilestone {
  date: string; // ISO 8601 date
  title: string;
  description: string;
}

export type ProjectAvailability = 'available' | 'not-available' | 'open-to-roles';

export interface Project {
  slug: string;
  name: string;
  description: string;
  techStack: string[];
  demoUrl: string | null;
  repoUrl: string | null;
  artistStatement: string;
  timeline: {
    milestones: TimelineMilestone[];
  };
  hasBuildStory: boolean;
  availability: ProjectAvailability;
}

// Config entries example (placeholder — owner fills in real data)
export const projects: Project[] = [
  // Add/remove entries here — no component code changes needed
];

export const getProjectBySlug = (slug: string): Project | undefined =>
  projects.find(p => p.slug === slug);

export const getAllProjects = (): Project[] => projects;

export const getAvailableProjects = (): Project[] =>
  projects.filter(p => p.availability === 'available');
```

```typescript
// about.ts
export interface Availability {
  status: 'available' | 'not-available' | 'open-to-roles';
  location: string; // e.g., "Ho Chi Minh City, Vietnam"
}

export interface AboutConfig {
  availability: Availability;
  // Extend with other about-section fields as needed
}

export const aboutConfig: AboutConfig = {
  availability: {
    status: 'open-to-roles',
    location: 'Ho Chi Minh City, Vietnam',
  },
};

export const getAvailability = (): Availability => aboutConfig.availability;
```

**Gallery integration pattern:**
```tsx
// In ProjectGallery.tsx
import { getAllProjects } from '@/config/projects';

// Replace: const projects = [...hardcodedData];
const projects = getAllProjects();
```

**About integration pattern:**
```tsx
// In AvailabilityBadge.tsx
import { getAvailability } from '@/config/about';

const availability = getAvailability();
// Render based on availability.status
```

### Cross-Story Dependencies

- **Story 1.4 (Project Detail Page):** Required — project detail page uses `slug` for routing. Ensure config `slug` field matches existing URL patterns (`/projects/:slug`).
- **Story 1.5 (About/Skills/Availability):** Required — existing About section must be refactored to use `src/config/about.ts` instead of hardcoded values.
- **Story 1.6 (i18n):** Required — project names, descriptions, artist statements should be i18n-compatible if using Vietnamese/English. Consider using i18n keys in config or wrapping config exports with translation functions.

### Gotchas

- **`slug` uniqueness** — enforce unique slugs in the `projects` array. Add runtime validation or TypeScript assertion for duplicate slugs.
- **`null` for demoUrl/repoUrl** — not all projects have live demos or public repos. Handle `null` gracefully in `ProjectCard` (hide links or show disabled state).
- **`hasBuildStory` flag** — projects marked `hasBuildStory: true` may need a "View Build Story" link. Gallery/detail pages should respect this flag.
- **`techStack` array** — should render as tag badges. Keep strings simple (e.g., `"React"`, `"Spring Boot"`).
- **i18n consideration** — if project content needs Vietnamese/English variants, consider a two-letter structure or defer to i18n key pattern. Check with Story 1.6 implementation before assuming English-only config values.
- **Timeline milestones** — date format should be ISO 8601 (`YYYY-MM-DD`) for consistency. Milestones should be sorted by date automatically in `getProjectBySlug` or rendered sorted in the UI.
- **No component code changes for new projects** — the goal is zero-touch gallery expansion. Verify by adding a test project entry to config and confirming it renders without touching any component files.

### References

- [Source: _bmad-output/planning-artifacts/epics.md → Epic 6 → Story 6.4.1 ACs]
- [Source: _bmad-output/planning-artifacts/architecture.md → Frontend Architecture → Starter Template → TypeScript strict mode]
- [Source: _bmad-output/planning-artifacts/architecture.md → Starter Template → shadcn/ui, Tailwind v4 CSS-first config]
- [Source: _bmad-output/planning-artifacts/architecture.md → Project Structure → Portfolio FE → src/components/, src/config/ convention]
- [Source: _bmad-output/implementation-artifacts/1-4-project-detail-page.md → slug-based routing for project detail]
- [Source: _bmad-output/implementation-artifacts/1-5-about-skills-availability.md → existing About section implementation to refactor]
- [Source: _bmad-output/planning-artifacts/architecture.md → Implementation Patterns → Frontend → Static Config Files → Config-driven content]

---

## Dev Agent Record

### Agent Model Used

Claude Sonnet 4.6 (Anthropic) — 2026-03-19

### Debug Log References

### Completion Notes List

### File List

**New files:**
- `portfolio-fe/src/config/projects.ts` — typed project showcase config
- `portfolio-fe/src/config/about.ts` — typed about/availability config

**Modified files:**
- `portfolio-fe/src/components/gallery/ProjectGallery.tsx` — import from config instead of hardcoded data
- `portfolio-fe/src/components/about/AboutSection.tsx` — import availability from config
- `portfolio-fe/src/components/about/AvailabilityBadge.tsx` — use config-driven availability

**Test files:**
- `portfolio-fe/src/config/__tests__/projects.test.ts` — config helper unit tests
- `portfolio-fe/src/config/__tests__/about.test.ts` — about config helper unit tests

---

## 5. Technical Compliance Checklist

Before marking story done, verify:

- [ ] `src/config/projects.ts` exists with full `Project`, `TimelineMilestone`, `ProjectAvailability` interfaces
- [ ] `projects` array is typed as `Project[]`
- [ ] `getProjectBySlug(slug)` helper exported and returns `Project | undefined`
- [ ] `getAllProjects()` helper exported
- [ ] `getAvailableProjects()` helper exported (filters by `availability === 'available'`)
- [ ] `src/config/about.ts` exists with `Availability` and `AboutConfig` interfaces
- [ ] `aboutConfig` exports typed `Availability` with `status` and `location`
- [ ] `getAvailability()` helper exported
- [ ] Gallery component imports from `src/config/projects.ts` — no hardcoded data
- [ ] About/AvailabilityBadge component imports from `src/config/about.ts`
- [ ] Adding a new entry to `projects` array renders in gallery without component code changes
- [ ] Updating `availability.status` in `aboutConfig` updates badge without component changes
- [ ] `slug` uniqueness is validated (no duplicate slugs in config)
- [ ] `null` demoUrl/repoUrl handled gracefully (no broken links in UI)
- [ ] Timeline milestones sorted by date in UI
- [ ] TypeScript strict mode compiles without errors
- [ ] Unit tests pass: `getProjectBySlug`, `getAllProjects`, `getAvailableProjects`, `getAvailability`

