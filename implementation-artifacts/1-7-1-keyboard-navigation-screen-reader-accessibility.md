# Story 1.7.1: Keyboard Navigation & Screen Reader Accessibility

Status: done

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a recruiter with motor or visual impairments,
I want the portfolio fully navigable by keyboard and screen reader,
so that I can access all content without a mouse.

## Acceptance Criteria

**AC1: All Interactive Elements Reachable via Tab**
```
Given keyboard-only navigation,
When I press Tab through all interactive elements,
Then every button, link, card, and form field is reachable
  in a logical order; none are skipped.
```

**AC2: Screen Reader Landmark & Alt Text Coverage**
```
Given I use a screen reader (VoiceOver / NVDA),
When I navigate the portfolio,
Then all images have descriptive `alt` text;
  all form inputs have `<label>` elements;
  landmark regions (<nav>, <main>, <section>) have accessible names
  via aria-label or aria-labelledby.
```

**AC3: Visible Focus Ring on All Focused Elements**
```
Given keyboard focus moves between elements,
When an element receives focus,
Then a visible focus ring is shown
  (2px solid ring, offset 2px, brand-light / ring token color)
  — no element has outline:none without a replacement focus indicator.
```

**AC4: Modal/Drawer Focus Trap**
```
Given the mobile navigation drawer opens,
When it opens,
Then focus is trapped inside until closed;
  Escape key closes it;
  focus returns to the hamburger trigger element.
```

**AC5: Skip to Main Content Link**
```
Given the portfolio page loads,
When a keyboard user presses Tab once,
Then a "Skip to main content" link is the first focusable element,
  becomes visible on focus,
  and navigates to #main-content on activation.
```

**AC6: No Regressions & Full Build**
```
Given Story 1.7.1 implementation is complete,
When validation runs,
Then:
  - All existing 194 tests from Stories 1.1–1.6 still pass
  - New tests written for: focus ring presence, ARIA landmark labeling,
    skip link behavior, and screen reader attributes
  - pnpm run lint: 0 errors
  - pnpm run build: succeeds (Vite + tsc both pass)
```

## Tasks / Subtasks

- [x] **Task 1: Fix ARIA landmark labeling on unlabeled sections** (AC: 2)
  - [x] `About.tsx`: Add `id="about-heading"` to the `<h2>` inside About section; add `aria-labelledby="about-heading"` to `<section id="about">`
  - [x] `HomePage.tsx`: Add `aria-label="Contact"` to the placeholder `<section id="contact">` (Story 4.1 target)
  - [x] Verify Hero, Projects sections already have `aria-label` (they do — no change needed)

- [x] **Task 2: Fix missing focus rings on interactive elements** (AC: 3)
  - [x] `Hero.tsx`: Add `focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2 focus-visible:outline-none rounded-md` to both CTA `<a>` tags (`#projects` and `#contact` links)
  - [x] `ProjectCard.tsx`: Add `focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2 focus-visible:outline-none rounded-sm` to "View project" `<Link>` and both icon `<a>` links (ExternalLink, Github)
  - [x] `ProjectDetailPage.tsx`: Add focus ring classes to: back-to-gallery `<Link>`, "View Live" `<a>`, "View on GitHub" `<a>`, and the lessons-learned `<button>`

- [x] **Task 3: Verify MobileSheet focus trap & Escape behavior** (AC: 4)
  - [x] Confirm Radix UI Sheet (used in MobileSheet.tsx) already handles: focus trap on open, Escape key closes, focus returns to trigger
  - [x] Read `src/components/ui/sheet.tsx` to confirm Radix `DialogContent` is the underlying primitive
  - [x] Write a `MobileSheet.test.tsx` test verifying: sheet renders with correct ARIA attributes (role="dialog")
  - [x] No code changes needed if Radix handles it — only tests

- [x] **Task 4: Write new accessibility tests** (AC: 1, 2, 3, 5, 6)
  - [x] `SkipLinks.test.tsx`: Add tests for focus-visible:not-sr-only and single anchor
  - [x] `Hero.test.tsx`: Add test checking both CTA links have `focus-visible:ring` in className
  - [x] `ProjectCard.test.tsx`: Add test verifying "View project" link and icon links have focus ring classes
  - [x] `About.test.tsx`: Add test checking `<section>` has `aria-labelledby` matching the h2 `id`
  - [x] `Nav.test.tsx`: Add tests checking nav aria-label, links are anchor elements, logo has focus ring
  - [x] `ProjectDetailPage.test.tsx`: Add tests for focus ring presence on back link, external links, and lessons button

- [x] **Task 5: Full validation** (AC: 6)
  - [x] Run `pnpm run test` — 211/211 tests pass (194 existing + 17 new), 0 regressions
  - [x] Run `pnpm run lint` — 0 errors
  - [x] Run `pnpm run build` — succeeds (497KB bundle)

## Dev Notes

### CRITICAL ARCHITECTURE CONSTRAINTS (Carry-forward from Stories 1.1–1.6)

1. **NO `tailwind.config.js`** — Tailwind v4 CSS-first. All new design tokens in `src/index.css`.
2. **NO `../../` relative imports** — Use `@/` alias exclusively.
3. **Spring tokens from `@/constants/motion`** — Never inline spring values.
4. **`<MotionConfig reducedMotion="user">` is already in `App.tsx`** — Do NOT add another.
5. **shadcn components are owned code** — `src/components/ui/` files may be edited; do NOT re-run the shadcn CLI.
6. **TypeScript strict mode** — No `any`. Use explicit types.
7. **pnpm** — Always `pnpm`, never `npm install` or `yarn`.
8. **ESLint `import/no-relative-parent-imports`** — Use `@/` always. Never `../../`.
9. **react-i18next mock** — Global mock in `src/test/setup.ts` uses `flattenTranslations(en.json)`, so `t(key)` returns actual English values in tests.

### What Already Exists (DO NOT RECREATE)

| File | Status | Notes |
|------|--------|-------|
| `src/components/layout/SkipLinks.tsx` | ✅ Done | "Skip to main content" sr-only link, href="#main-content", visible on focus. **DO NOT re-implement.** |
| `src/components/layout/SkipLinks.test.tsx` | ✅ Done | 2 tests: renders with correct href, has sr-only class. Extend, don't replace. |
| `src/components/layout/Nav.tsx` | ✅ Done | `aria-label="Main navigation"`, focus rings on all interactive elements, `role="list"` on ul, hamburger `aria-expanded` + `aria-controls="mobile-nav"` |
| `src/components/layout/MobileSheet.tsx` | ✅ Done | Radix UI Sheet handles focus trap + Escape. `aria-label="Mobile navigation"`, `SheetTitle`, `SheetDescription`. |
| `src/components/sections/Hero.tsx` | ⚠️ Partial | `aria-label="Hero"` on section ✓; `aria-hidden="true"` on decorative divs ✓; CTA links **missing focus rings** ✗ |
| `src/components/sections/Projects.tsx` | ✅ Done | `aria-label="Projects"` ✓, `role="group" aria-label="Filter projects"` ✓ |
| `src/components/sections/About.tsx` | ⚠️ Partial | Skill list `role="list"` + `aria-label` ✓; availability `aria-label` ✓; **section itself missing `aria-labelledby`** ✗ |
| `src/components/shared/ProjectCard.tsx` | ⚠️ Partial | `aria-label={title}` on article ✓; icon links have `aria-label` ✓; **footer links missing focus rings** ✗ |
| `src/components/shared/HeroCardStack.tsx` | ✅ Done | `role="region" aria-label="Portfolio overview"` |
| `src/components/shared/StatusIndicator.tsx` | ✅ Done | `aria-label="Project status: {status}"` |
| `src/components/shared/FilterChip.tsx` | ✅ Done | `focus-visible:ring-2 focus-visible:ring-brand-light` ✓ |
| `src/pages/ProjectDetailPage.tsx` | ⚠️ Partial | `aria-labelledby` on timeline/lessons sections ✓; `aria-expanded` on collapse button ✓; **links + button missing focus rings** ✗ |
| `src/pages/HomePage.tsx` | ⚠️ Partial | `<main id="main-content">` ✓; SkipLinks rendered first ✓; **contact section missing `aria-label`** ✗ |

### Current ARIA Landmark Audit

```
<body>
  <a href="#main-content">Skip to main content</a>   ← SkipLinks ✅
  <header>                                             ← implicit landmark ✅
    <nav aria-label="Main navigation">                 ← labeled ✅
  <main id="main-content">                             ← implicit landmark ✅
    <section aria-label="Hero">                        ← labeled ✅
    <section aria-label="Projects">                    ← labeled ✅
    <section id="about">                               ← MISSING label ✗ → fix: aria-labelledby="about-heading"
    <section id="contact">                             ← MISSING label ✗ → fix: aria-label="Contact"
```

### Focus Ring Gaps — Exact Elements to Fix

**Hero.tsx** (both CTA `<a>` tags, currently have NO focus ring):
```tsx
// Add to both <a> className:
"focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2 focus-visible:outline-none rounded-md"
```

**ProjectCard.tsx** — three elements missing focus rings:
```tsx
// "View project" Link:
<Link
  to={`/projects/${slug}`}
  className="text-sm font-medium text-brand hover:text-brand-light transition-colors
    focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2
    focus-visible:outline-none rounded-sm"
  aria-label={`View project ${title}`}
>

// External link icon:
<a
  ...
  className="text-muted-foreground hover:text-foreground transition-colors
    focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2
    focus-visible:outline-none rounded-sm"
  aria-label={`Open live demo for ${title}`}
>

// GitHub link icon:
<a
  ...
  className="text-muted-foreground hover:text-foreground transition-colors
    focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2
    focus-visible:outline-none rounded-sm"
  aria-label={`View source code for ${title} on GitHub`}
>
```

**ProjectDetailPage.tsx** — four elements missing focus rings:
```tsx
// Back link:
<Link to="/" className="mb-8 inline-flex items-center gap-1 text-sm text-muted-foreground
  hover:text-foreground focus-visible:ring-2 focus-visible:ring-brand-light
  focus-visible:ring-offset-2 focus-visible:outline-none rounded-sm">

// "View Live" link:
<a ... className="inline-flex items-center gap-1 text-sm font-medium text-brand
  hover:underline focus-visible:ring-2 focus-visible:ring-brand-light
  focus-visible:ring-offset-2 focus-visible:outline-none rounded-sm">

// "View on GitHub" link:
<a ... className="inline-flex items-center gap-1 text-sm font-medium text-muted-foreground
  hover:text-foreground hover:underline focus-visible:ring-2 focus-visible:ring-brand-light
  focus-visible:ring-offset-2 focus-visible:outline-none rounded-sm">

// Lessons toggle button (raw <button>, NOT shadcn Button):
<button ... className="flex w-full items-center justify-between text-left text-base
  font-semibold text-foreground hover:text-brand
  focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2
  focus-visible:outline-none rounded-sm">
```

### ARIA Fix for About Section

```tsx
// About.tsx — section gets aria-labelledby:
<section id="about" aria-labelledby="about-heading" className="section-container py-24">
  <div className="max-w-2xl mx-auto space-y-10">
    <div>
      <h2 id="about-heading" className="text-2xl font-semibold mb-4">{t('about.heading')}</h2>
```

### ARIA Fix for Contact Placeholder (HomePage.tsx)

```tsx
// HomePage.tsx — contact section gets aria-label:
<section id="contact" aria-label="Contact" className="section-container py-24">
```

### No Images in Current Codebase

There are **no `<img>` elements** in the current codebase. All images are:
- Lucide React SVG icons (inline SVG, rendered as `<svg>`) — already have `aria-hidden` or `aria-label` where appropriate
- CSS background gradients (`aria-hidden="true"` on container divs) ✓

No alt-text work needed for Story 1.7.1. This will be relevant in future stories (Story 3.x: live metrics with possible screenshots).

### No Form Inputs in Current Codebase

The contact form (`<form>`, `<input>`, `<label>`) is Story 4.1 scope. Current interactive elements are: links, buttons, filter chips. All are correctly labeled.

### MobileSheet Focus Trap — Already Handled

`MobileSheet.tsx` uses shadcn `Sheet` → Radix UI `DialogContent`. Radix handles:
- Focus trap when open (via `@radix-ui/react-dialog`)
- Escape key closes (built-in Radix behavior)
- Focus return to trigger: Radix returns focus to the element that opened the dialog on close

Verify by reading `src/components/ui/sheet.tsx` — it wraps `@radix-ui/react-dialog`. **No code changes needed.**

### `motion.article` in ProjectCard — Not a Keyboard Target

`ProjectCard` uses `<motion.article onClick={() => navigate(...)}>`. This `onClick` on the article is a *supplementary* click affordance for mouse users. Keyboard users navigate the card via the **"View project" Link** inside the card footer. The article itself does NOT need `tabIndex={0}` or a keyboard handler — the inner links provide all the keyboard accessibility needed. Adding `tabIndex` to the card would duplicate tab stops and harm keyboard UX.

### Radix UI Sheet Accessibility Attributes

When `MobileSheet` opens, Radix renders a `div` with:
- `role="dialog"`
- `aria-modal="true"`
- `aria-labelledby` pointing to `SheetTitle`

These are auto-generated by Radix — **do not manually add** them to `MobileSheet.tsx`.

### Testing Standards (Same as 1.3–1.6)

- Co-located test files (`.test.tsx` alongside component)
- `@testing-library/react` + `vitest`
- framer-motion mock: `vi.mock('framer-motion', ...)` required in components using motion
- react-i18next mock: already global in `src/test/setup.ts` (returns actual en.json values)
- Run `pnpm run test` after EVERY task, not just at the end
- **DO NOT** add `@axe-core/react` or `jest-axe` in this story — axe-core automated CI gate is Story 1.7.2 scope

### Test Patterns for This Story

**Testing focus ring presence:**
```tsx
// Check className contains focus-visible ring
const link = screen.getByRole('link', { name: /view project/i })
expect(link.className).toContain('focus-visible:ring-2')
```

**Testing ARIA labelledby:**
```tsx
// About section test
const section = document.getElementById('about')
expect(section).toHaveAttribute('aria-labelledby', 'about-heading')
const heading = document.getElementById('about-heading')
expect(heading).toBeInTheDocument()
```

**Testing SkipLinks position:**
```tsx
// Verify SkipLinks renders before Nav in DOM
const { container } = render(<HomePage />)
const skipLink = container.querySelector('a[href="#main-content"]')
const nav = container.querySelector('nav')
// skipLink should appear before nav in DOM order
expect(container.firstChild).toBe(skipLink) // or check DOM order
```

### Previous Story Learnings (Critical from 1.3–1.6)

1. **`role="listitem"` on `<a>` overrides link role** — always use `<ul>/<li>/<a>` structure (already applied in Nav and About) — do NOT add `role` to `<a>` tags
2. **`react-refresh/only-export-components`** — do NOT export non-component constants from component files
3. **Safari+VoiceOver: `list-none` removes list semantics** — always add `role="list"` explicitly on `<ul>` when using `list-style: none` (Nav and About already have this)
4. **`pnpm run lint` before submitting** — `import/order` violations most common with new files
5. **194 tests currently pass** — run `pnpm run test` before each task to ensure no regressions
6. **`aria-label` overrides accessible name in queries** — `getByRole('link', { name: /view project/i })` matches either the text content or the `aria-label`, whichever is the accessible name
7. **`focus-visible:outline-none` must pair with a ring** — never add `outline-none` without `focus-visible:ring-*` alongside it

### Files to Create / Modify

```
src/
├── components/layout/
│   ├── SkipLinks.test.tsx     ← MODIFY: add DOM-order test
│   └── MobileSheet.test.tsx   ← MODIFY: add dialog ARIA role test
├── components/sections/
│   ├── Hero.tsx               ← MODIFY: add focus rings to CTA links
│   ├── Hero.test.tsx          ← MODIFY: add focus ring assertion tests
│   └── About.tsx              ← MODIFY: add id to h2, aria-labelledby to section
│   └── About.test.tsx         ← MODIFY: add aria-labelledby test
├── components/shared/
│   ├── ProjectCard.tsx        ← MODIFY: add focus rings to footer links
│   └── ProjectCard.test.tsx   ← MODIFY: add focus ring tests
├── pages/
│   ├── HomePage.tsx           ← MODIFY: add aria-label to contact section
│   ├── HomePage.test.tsx      ← MODIFY: add contact section aria-label test
│   ├── ProjectDetailPage.tsx  ← MODIFY: add focus rings to links + lessons button
│   └── ProjectDetailPage.test.tsx ← MODIFY: add focus ring tests
```

**No new files need to be created.**

### Dependencies Already Installed (Do NOT Re-Install)

All needed packages are already in `package.json`:
- `@testing-library/react` — installed (Stories 1.1+)
- `@testing-library/jest-dom` — installed (Stories 1.1+)
- `vitest` — installed (Stories 1.1+)
- `framer-motion` — installed (Stories 1.3+)
- `react-i18next` — installed (Story 1.6)

**Do NOT install `@axe-core/react`, `jest-axe`, or `vitest-axe`** — out of scope for 1.7.1.

### Cross-Story Context

- **Story 1.6 (done):** `document.documentElement.lang` is updated on language switch → screen readers announce language correctly. This was a prerequisite.
- **Story 1.7.2 (next):** Visual accessibility, color contrast, axe-core CI gate, responsive verification. Do NOT do contrast or axe-core work in this story.
- **Story 1.8:** Spring physics animations. `<MotionConfig reducedMotion="user">` already in App.tsx handles `prefers-reduced-motion` globally — no additional work needed in 1.7.1.
- **Story 4.1:** Contact form — form inputs + labels will need proper accessibility in that story, not here.

### Project Structure Notes

- All components live at `src/components/` — co-locate tests
- Section components at `src/components/sections/`
- Layout components at `src/components/layout/`
- Pages at `src/pages/`
- No new directories needed

### References

- Epics: [_bmad-output/planning-artifacts/epics.md](_bmad-output/planning-artifacts/epics.md) — Stories 1.7, 1.7.1 definition (lines 480–535)
- Architecture: [_bmad-output/planning-artifacts/architecture.md](_bmad-output/planning-artifacts/architecture.md) — Accessibility NFRs (lines 92–94), FR18–FR19 (keyboard nav, WCAG 2.1 AA), Phase 2 command palette hook (line 184–186)
- Previous story: [_bmad-output/implementation-artifacts/1-6-internationalization-vi-en.md](_bmad-output/implementation-artifacts/1-6-internationalization-vi-en.md) — Dev learnings, test setup pattern, component modifications
- WCAG 2.1 AA success criteria: SC 2.1.1 (keyboard), SC 2.4.3 (focus order), SC 2.4.7 (focus visible), SC 4.1.2 (name/role/value), SC 1.3.1 (info & relationships)

## Dev Agent Record

### Agent Model Used

claude-sonnet-4-6

### Debug Log References

- MobileSheet: Radix Dialog in jsdom does NOT render `aria-modal="true"` attribute in test environment (it handles focus trapping natively in browser without the explicit attribute in jsdom). Test adjusted to assert `role="dialog"` only.

### Completion Notes List

- All 5 tasks completed. 213/213 tests pass (19 new tests added), 0 lint errors, build succeeds.
- Task 1: Added `aria-labelledby="about-heading"` + `id="about-heading"` to About section; added `aria-label="Contact"` to HomePage contact placeholder.
- Task 2: Added `focus-visible:ring-2 focus-visible:ring-brand-light focus-visible:ring-offset-2 focus-visible:outline-none` focus ring classes to 9 interactive elements across 4 files: Hero CTA links (2), ProjectCard footer links (3), ProjectDetailPage links + button (4 happy path + 1 not-found path). Also fixed About.tsx skill badge links missing focus rings.
- Task 3: Confirmed Radix UI Dialog (used by MobileSheet via Sheet primitive) handles focus trap, Escape key, and focus return natively. No MobileSheet.tsx code changes needed. 2 new tests added to MobileSheet.test.tsx.
- Task 4: 19 new tests added across 8 test files, all passing. Code review caught 2 AC3 violations fixed post-review: ProjectDetailPage not-found back link and About.tsx skill links.
- No new dependencies installed. All changes are CSS classes and ARIA attributes.

### File List

**Modified:**
- `src/components/sections/About.tsx` — Added `id="about-heading"` to h2, `aria-labelledby="about-heading"` to section; added focus ring classes to skill badge links
- `src/pages/HomePage.tsx` — Added `aria-label="Contact"` to contact section
- `src/components/sections/Hero.tsx` — Added focus ring classes to both CTA links
- `src/components/shared/ProjectCard.tsx` — Added focus ring classes to "View project" Link, ExternalLink a, Github a
- `src/pages/ProjectDetailPage.tsx` — Added focus ring classes to back link (found + not-found states), "View Live" a, "View on GitHub" a, lessons button
- `src/components/sections/About.test.tsx` — Added aria-labelledby test
- `src/pages/HomePage.test.tsx` — Added contact section aria-label test
- `src/components/sections/Hero.test.tsx` — Added CTA links focus ring test
- `src/components/shared/ProjectCard.test.tsx` — Added 3 focus ring tests (View project, live demo, GitHub)
- `src/pages/ProjectDetailPage.test.tsx` — Added 5 tests (back link found+not-found states, View Live, View on GitHub, lessons button focus rings)
- `src/components/layout/Nav.test.tsx` — Added 3 tests (aria-label, anchor elements, logo focus ring)
- `src/components/layout/SkipLinks.test.tsx` — Added 2 tests (focus-visible:not-sr-only, single anchor)
- `src/components/layout/MobileSheet.test.tsx` — Added 2 tests (role="dialog", keyboard-reachable links)

## Change Log

- 2026-03-06: Story 1.7.1 created by SM agent (create-story workflow). Story key: `1-7-1-keyboard-navigation-screen-reader-accessibility`.
- 2026-03-06: Story 1.7.1 implemented by dev agent (claude-sonnet-4-6). All 5 tasks complete. 211/211 tests pass (17 new), 0 lint, build green. Status: review.
- 2026-03-06: Code review by claude-sonnet-4-6. Fixed 2 AC3 violations: ProjectDetailPage not-found back link missing focus ring, About.tsx skill links missing focus rings. 2 tests added. 213/213 tests pass, 0 lint. Status: done.
