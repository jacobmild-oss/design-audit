# Batch D — Close-out

**Date:** 2026-04-19
**Status:** Substantively closed.
**Related:** `design-system.md §5 Batch D`, `d8-accessibility-report.md`, `d8-performance-baseline.md`

---

## What closed

Batch D set out to bring color, type, motion, and craft discipline up to the standard design-system v3.2 calls for. Every item in the sprint plan is on `main`:

| ID | Title | Commits |
|---|---|---|
| D-1 | Color discipline sweep — strip amber from non-CTA surfaces | `4ae6343`, `5762d23` (retro) |
| D-2 | Financial numbers to white, status pills | `19ba518` |
| D-3 | Status palette unification + Day Of hue | `ff5af95`, `5762d23` (retro) |
| D-4 | Role family compression + UpgradeBanner + neutral entity-type cleanup | `c0fd42c`, `5762d23` (retro) |
| D-5 | (bundled with D-4) UpgradeBanner primitive | `c0fd42c`, `5762d23` (retro) |
| D-6 | Brand color default + preset picker + marketing pricing | `64fa5f6`, `5762d23` (retro) |
| D-7 | Calendar color-by-artist architecture | `f56bf92` |
| D-8a | Static accessibility + performance verification | `f208e7b` |
| D-8b | Dynamic verification — axe-core + Lighthouse + Playwright | `85c7cb5` |
| D-8c-1 | BLOCKER accessibility remediation | `635c8a9`, `c43da02`, `74a9fae`, `9fa6361` + this close-out |
| D-9 | CopyableValue + hyperlink neutral migration | `07b6437` |
| D-10a | Vercel Analytics + PostHog with GDPR/DNT respect | `2bcfb62` |
| D-11 | Motion language foundation + prefers-reduced-motion | `2c58e0f` |
| D-12 | `<Kbd>` inline keyboard-shortcut hint component | `54ae0b9` |
| D-13 | Ambient gradient on landing + dashboard | `93338ec` |

D-10b (recurring analytics review) was scoped as ongoing practice, not a one-shot commit — no sprint-closing work required.

---

## D-8c-1 — BLOCKER accessibility remediation summary

D-8c split into three sub-slices after D-8b surfaced more than was budgeted:

- **D-8c-1 (this slice) — BLOCKER only.** Automated axe critical violations + WCAG 2.5.8 AA touch targets.
- **D-8c-2 — HIGH, structural.** `page-has-heading-one` on 18 routes, `region` landmarks on 10 mobile routes, `heading-order` on `/dashboard`. Deferred to **F-15** (detail work pass).
- **D-8c-3 — HIGH, sensory/contrast.** `color-contrast` serious findings (35 of 40 runs, 246 element nodes), focus-ring gaps from D-8a §1F (calendar nav, ErrorFallback, UpgradeBanner), colorblind full simulation. Deferred to **F-15**.

### What D-8c-1 remediated

Four category-scoped commits, each preceded by `npm run build` (0 TS errors) and `npm run lint` (288 problems — pre-existing baseline maintained):

| SHA | Category | Files | axe nodes cleared |
|---|---|---:|---:|
| `635c8a9` | `button-name` — icon-only buttons get `aria-label` | 9 | 78 |
| `c43da02` | `link-name` — icon/image-only links get `aria-label` | 2 | 11 |
| `74a9fae` | `label` / `label-title-only` — native `htmlFor`/`id`, `role="switch"` + `aria-checked`, or `aria-label` when no visible label exists | 5 | 14 |
| `9fa6361` | touch targets — icon buttons bumped to `h-6 w-6`, sidebar nav `min-h-[44px] md:min-h-0` | 3 | 94 |

### Verification

Production build (`npm run build && npm run start` on port 3000) re-ran `axe-detail.mjs` across all 40 route-viewport combinations from D-8b.

**Result:** every route reports `hits=0`. The by-rule tally is empty (`{}`). The four critical rules that D-8b surfaced are fully cleared on `main`.

Raw axe output before:
```
button-name: 78 nodes across 21 routes
link-name:   11 nodes across 2 routes
label:       13 nodes across 5 routes
label-title-only: 1 node on /settings?tab=account
```

Raw axe output after:
```
(empty)
```

### What D-8c-1 deliberately did NOT do

Per the execution spec:

- **No HIGH / MEDIUM remediation.** `color-contrast`, `page-has-heading-one`, `region`, `heading-order`, focus-ring — all stand open.
- **No bundle analysis.** The 3.7 MB total / 222 KB top-chunk numbers from D-8a §2C are unchanged. Deferred to **F-14**.
- **No colorblind simulation pass.** Still requires human-eye review across 4 deficiency modes × key screens. Deferred to **F-15**.
- **No component refactors beyond the minimum.** Inputs got `id` attributes; `Label` in `gear/new` grew an optional `htmlFor` prop; `MerchVariantRow` refactored from `.map(...=>(...))` to `.map(...=>{...; return (...);})` to compute an `aria-label` from the variant label. Nothing structural.
- **No design changes.** The touch-target bumps (20→24 on sidebar favorites ×, 20→24 icon buttons) are the only visible deltas; they are within the §1.4 / §3.6 interaction budget and read as "slightly heavier chrome," not a new look.

### F-15 / F-14 follow-up catalog

Deferred from D-8c and logged here so nothing falls on the floor:

| Finding | Severity (axe) | Severity (Jacob) | Landing |
|---|---|---|---|
| `color-contrast` on 35 routes (246 nodes) | serious | HIGH (subset touching prose) / MEDIUM (chip/decorative) | F-15 |
| `page-has-heading-one` on 18 routes | moderate | MEDIUM | F-15 |
| `region` landmark on 10 mobile routes | moderate | MEDIUM | F-15 |
| `heading-order` on `/dashboard` | moderate | MEDIUM | F-15 |
| Calendar nav / ErrorFallback / UpgradeBanner missing `:focus-visible` | — (D-8a static) | HIGH | F-15 |
| Deuteranopia / protanopia / tritanopia / achromatopsia screen walk | — (not automatable) | HIGH | F-15 |
| Shows view-toggle uses `title=` (not aria-label) | — (D-8a static) | MEDIUM | F-15 |
| MiniMonth dots lack glyph fallback for colorblind users | — (D-8a static) | MEDIUM | F-15 |
| `chart-expense` on raised / float surfaces (§1B) | serious | HIGH | F-15 |
| Bundle size — 222 KB top chunk + 3.7 MB total + Turbopack no per-route first-load | (static) | HIGH | **F-14** |
| Mobile LCP 4.7–6.2 s across all dashboard routes (prod build) | (CWV "poor") | HIGH | **F-14** |
| Advancing tab 34 ms max render (React.Profiler) | (single outlier) | MEDIUM | F-14 |
| Calendar agenda scroll 43% frame-drop rate (headless) | (suggestive, verify on device) | MEDIUM | F-14 |
| OAuth first-login double-click bug (apex→www 307) | (functional) | HIGH (pre-beta) | follow-up before launch |

---

## Batch D — what shipped, in aggregate

- **Color:** amber is now CTA-only; status, role, chart, and chip families have their own discipline. Calendar encodes artist via `ENTITY_TYPE_COLORS`, not event type.
- **Typography:** financial numbers render white with adjacent status pills; prose uses NeueMontreal, display uses BasementGrotesque-Black, data uses the system mono stack. Settings tabs no longer wrap to two lines.
- **Motion:** `--duration-*` + `--ease-*` tokens, `animate-lift` + `animate-pulse-soft` utilities, `prefers-reduced-motion` override.
- **Craft:** `<Kbd>`, `<CopyableValue>`, `<UpgradeBanner>`, `<BrandColorPicker>` primitives. Ambient gradient on landing + dashboard.
- **Analytics:** Vercel Analytics + PostHog with GDPR / DNT respect.
- **Accessibility:** axe critical rules clean on production build; WCAG 2.5.8 AA touch targets met across the dashboard.
- **Performance baseline:** Lighthouse desktop 89–93 perf (zero CLS across the matrix); mobile 75–83 perf on prod, LCP still "poor" — documented as F-14 work.

---

## What's next

- **Batch E** — landing-page rebuild, right-panel extension, Files IA reconciliation, settings tab overflow, Money Income single-currency. See `design-system.md §5 Batch E`.
- **F-14** — 60fps performance discipline. Bundle analyzer, per-route budgets, Lighthouse CI, image/font/bundle optimization.
- **F-15** — detail work pass. Recurring bi-weekly practice. Absorbs the `color-contrast` / structural / focus-ring / colorblind catalog above plus the "thousand small cuts" that accrue during Batch E.

Batch D is closed. Everything the design-system v3.2 planned for this batch has landed on `main`, and the accessibility BLOCKER queue that D-8b surfaced is zero.
