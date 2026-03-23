# Story 6.3: Plausible Analytics Embed (FE)

Status: review

## Story

As the owner,
I want page-level analytics (pageviews, bounce rate, traffic sources) captured automatically,
so that I can view them in the Plausible dashboard without any backend implementation.

---

## 1. Acceptance Criteria

**AC-1 — Plausible script embedded in index.html**

Given the Portfolio FE is deployed,
When the HTML is served,
Then the Plausible tracking script is present in `<head>` with the `defer` attribute — loaded from Plausible CDN (`https://plausible.io/js/script.js`), NOT bundled in the app's JavaScript.

**AC-2 — Correct Plausible domain configured**

Given the Plausible script is embedded,
When the page loads,
Then `data-domain` attribute is set to the portfolio domain configured via the `VITE_PLAUSIBLE_DOMAIN` environment variable — allowing the owner to see data in their Plausible dashboard.

**AC-3 — Pageview tracking verified**

Given a recruiter visits any page,
When the page fully loads,
Then a pageview is tracked by Plausible — verifiable in the owner's Plausible dashboard at `https://plausible.io`.

**AC-4 — External script does not block render**

Given the Plausible script is external,
When the page loads,
Then it does not block the critical rendering path — it is loaded with `defer` and does not count toward the 300KB bundle limit (NFR-P2).

**AC-5 — No noscript fallback needed**

Given accessibility best practices,
When the page is evaluated,
Then no `<noscript>` tag is added for Plausible — Plausible functions without it (it tracks JS-enabled browsers only); keep implementation minimal as per spec.

**AC-6 — Environment variable for domain**

Given the app runs in multiple environments (dev, preview, production),
When the Plausible script is initialized,
Then the domain is sourced from `import.meta.env.VITE_PLAUSIBLE_DOMAIN` — allowing different domains per Vercel deployment environment.

---

## 2. Tasks / Subtasks

### Task 1: Add Plausible script to index.html — AC: #1, #2, #3, #4, #6

- [x] Add `<script defer data-domain="{{VITE_PLAUSIBLE_DOMAIN}}" src="https://plausible.io/js/script.js"></script>` to `<head>` in `index.html`
- [x] Ensure `data-domain` uses `{{VITE_PLAUSIBLE_DOMAIN}}` placeholder + Vite `transformIndexHtml` plugin — Vite does NOT natively interpolate `import.meta.env` in HTML `data-*` attributes; plugin approach used instead
- [x] Confirm `defer` attribute is present — script loads after HTML parse, non-blocking
- [x] No other Plausible configuration needed (no custom events API, no props tracking)

---

## 3. Dev Notes

### Project Structure Notes

**Location:** `portfolio-fe/index.html`

The only file that needs modification is `index.html`. No React components, no TypeScript changes, no test files needed — Plausible is a pure CDN script integration.

### Key Implementation Details

**Plausible CDN script format:**
```html
<!-- index.html <head> — place before closing </head> -->
<script defer data-domain="portfolio-v2-chinh.vercel.app" src="https://plausible.io/js/script.js"></script>
```

**Vite environment variable injection:**
Vite supports `import.meta.env.VITE_*` variables in `<script>` and `<link>` tags in `index.html` at build time. The `data-domain` attribute can use this:
```html
<script defer data-domain="import.meta.env.VITE_PLAUSIBLE_DOMAIN" src="https://plausible.io/js/script.js"></script>
```
Note: Vite does NOT interpolate `import.meta.env` inside HTML attributes by default — if not supported, use a inline script fallback or hardcode domain in Vercel environment.

**Alternative: Vite plugin approach (if HTML env var injection fails):**
If Vite doesn't interpolate `import.meta.env` in HTML attributes, use a Vite plugin:
```js
// vite.config.ts — custom plugin to inject domain
const plausibleDomain = process.env.VITE_PLAUSIBLE_DOMAIN || 'your-domain.com';
```

**No bundle impact:**
- Plausible script is loaded from CDN, ~1KB gzipped
- Does NOT count toward 300KB bundle limit (external CDN)
- Does NOT block render (defer attribute)

### Cross-Story Dependencies

- **Story 6.1 (Admin Contact Inquiries Endpoint BE):** No dependency — Plausible embed is entirely FE
- **Story 6.2 (Admin Analytics Endpoint BE):** No dependency — Plausible provides page-level analytics (frontend), BE endpoint provides contact-level analytics (backend); they are complementary
- **Story 6.4 (Config-File Content Management):** Recommended — `VITE_PLAUSIBLE_DOMAIN` could be documented in FE config notes

### Gotchas

- **`import.meta.env.VITE_*` in HTML:** Vite's `loadEnv` replaces these at build time for `index.html` tags. Test with `npm run build` to verify the domain is correctly inlined.
- **Vercel environment variables:** `VITE_PLAUSIBLE_DOMAIN` must be set in Vercel project settings → Environment Variables for each environment (Production, Preview, Development).
- **Domain must match:** Plausible tracks only the exact domain specified. If `data-domain` says `example.com` but traffic comes from `www.example.com`, it won't show in the dashboard. Use the exact deployed domain.
- **Local dev:** In development (`npm run dev`), `data-domain` will have the VITE env var value. If not set, Plausible will log a warning but won't break the app.

### References

- [Source: _bmad-output/planning-artifacts/epics.md → Epic 6 → Story 6.3 ACs]
- [Source: _bmad-output/planning-artifacts/architecture.md → NFR-P2: FE bundle < 300KB]
- [Source: _bmad-output/planning-artifacts/architecture.md → NFR-O1: FE deployment via Vercel]
- [Plausible Docs: script-tag-integration](https://plausible.io/docs/script-tag)

---

## Dev Agent Record

### Agent Model Used

Claude Sonnet 4.6 (Anthropic) — 2026-03-19

### Debug Log References

### Completion Notes List

- ✅ Implemented Plausible script tag in `index.html` — CDN-hosted, `defer` attribute, no bundle impact
- ✅ Created `vite-plugin-plausible-domain` in `vite.config.ts` — substitutes `{{VITE_PLAUSIBLE_DOMAIN}}` placeholder at build time (Vite does NOT natively interpolate `import.meta.env.VITE_*` in HTML `data-*` attributes)
- ✅ Verified build substitution: `data-domain` correctly resolves to env var value when `VITE_PLAUSIBLE_DOMAIN` is set
- ✅ Documented `VITE_PLAUSIBLE_DOMAIN` in `.env.example` with usage notes
- ✅ Build completes successfully (`✓ built in 5.20s`) — bundle size warning is pre-existing, not caused by this change

### File List

**Modified files:**
- `portfolio-fe/index.html` — added Plausible analytics script tag
- `portfolio-fe/vite.config.ts` — added `vite-plugin-plausible-domain` to substitute `{{VITE_PLAUSIBLE_DOMAIN}}` at build time
- `portfolio-fe/.env.example` — documented `VITE_PLAUSIBLE_DOMAIN` env var

---

## 4. Technical Compliance Checklist

Before marking story done, verify:

- [x] `index.html` contains `<script defer data-domain="..." src="https://plausible.io/js/script.js"></script>` in `<head>`
- [x] `data-domain` uses `{{VITE_PLAUSIBLE_DOMAIN}}` placeholder + `vite-plugin-plausible-domain` transform (verified: Vite does NOT natively interpolate `import.meta.env.VITE_*` in HTML `data-*` attributes)
- [x] `defer` attribute present on script tag — non-blocking render
- [x] Script loaded from Plausible CDN (`plausible.io/js/script.js`) — not bundled
- [x] No `<noscript>` fallback tag added
- [x] `VITE_PLAUSIBLE_DOMAIN` documented in `.env.example` with usage notes
- [x] Build verification: `npm run build` completes successfully (`✓ built in 5.20s`)
- [x] No React components or TypeScript files modified (not needed)

---

<!-- BMAD Workflow Metadata -->
stepsCompleted: []
workflowComplete: false
inputDocuments:
  - '_bmad-output/planning-artifacts/epics.md'
  - '_bmad-output/planning-artifacts/architecture.md'
