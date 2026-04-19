# D-8a — Performance Baseline (Static Slice)

**Date:** 2026-04-19
**Scope:** `apps/web/` production build artifacts
**Method:** filesystem inspection of `.next/static/` after `npm run build`. Dynamic sections (Lighthouse, 60fps scrolling, re-render profiling) are scaffolded as checklists for D-8b manual verification.
**Related:** design-system.md §5 Batch D — D-8 Accessibility + performance pass.

---

## 2A — Lighthouse scores (desktop + mobile)

**Status:** **REQUIRES LIVE BROWSER — see D-8b checklist below.**

Cannot be produced from source: Lighthouse needs a rendered page, real network timing, and CPU profiling.

### D-8b checklist — Jacob, run this manually

Set up:
1. Run `cd apps/web && npm run build && npm start` so you're measuring the production build, not dev.
2. Open each route in Chrome incognito (no extensions).
3. DevTools → Lighthouse → Generate report (once for **desktop**, once for **mobile**).

**Routes to measure (8 total):**
- `/` (landing)
- `/login`
- `/dashboard`
- `/calendar`
- `/shows`
- `/money`
- `/settings`
- `/releases`

**Metrics to record per route × device:**

| Metric | Threshold (aspirational) |
|---|---|
| Performance | ≥ 90 desktop / ≥ 80 mobile |
| Accessibility | ≥ 95 both |
| Best Practices | ≥ 95 both |
| FCP (First Contentful Paint) | ≤ 1.0s desktop / ≤ 1.8s mobile |
| LCP (Largest Contentful Paint) | ≤ 2.0s desktop / ≤ 2.5s mobile |
| TTI (Time to Interactive) | ≤ 3.0s desktop / ≤ 3.8s mobile |
| CLS (Cumulative Layout Shift) | ≤ 0.05 |
| TBT (Total Blocking Time) | ≤ 200ms desktop / ≤ 300ms mobile |

**Recording template:**

```
## /calendar — desktop
- Performance: __
- Accessibility: __
- Best Practices: __
- FCP: __ s
- LCP: __ s
- TTI: __ s
- CLS: __
- TBT: __ ms
- Opportunities (top 3):
  1.
  2.
  3.
```

**Record at:** `design-audit/d8b-lighthouse-findings.md`.

**Severity scale:**
- **BLOCKER** — Performance score < 50 or LCP > 4s on any priority route
- **HIGH** — Performance < 80 mobile, or LCP > 2.5s mobile
- **MEDIUM** — Best Practices < 90 or missing a low-effort opportunity
- **LOW** — Polish only

---

## 2B — 60fps interaction recordings

**Status:** **REQUIRES LIVE BROWSER — see D-8b checklist below.**

Cannot be measured from source: frame timing only exists in the composited output.

### D-8b checklist — Jacob, run this manually

Use Chrome DevTools → Performance tab. For each scenario:
1. Reload page with recording on.
2. Perform the interaction.
3. Stop recording.
4. Check the frames rail — red bars are dropped frames.

**Scenarios (3 total):**

1. **Dashboard scroll** — from top to bottom of `/dashboard` with ~10 shows/releases in data.
   - Target: 0 dropped frames at 60fps on mid-range laptop (throttled CPU × 4).
2. **Calendar month switching** — on `/calendar` (month view), click prev/next 6 times in a row.
   - Target: each transition < 100ms to paint.
3. **Agenda scroll** — on `/calendar` (agenda view) with 60+ events, scroll fast from top to bottom.
   - Target: 0 dropped frames; no flicker.

**Recording template:**

```
## Dashboard scroll
- Frames: __ dropped / __ total
- Longest task: __ ms
- Scripting: __ ms / Rendering: __ ms / Painting: __ ms
- Layout shifts visible: yes/no
- Findings:
```

**Record at:** `design-audit/d8b-frame-findings.md`.

**Severity scale:**
- **BLOCKER** — > 10% dropped frames on any scenario on default CPU
- **HIGH** — > 5% dropped frames with 4× CPU throttle
- **MEDIUM** — Occasional dropped frame during animation
- **LOW** — Visual stutter only under 6× throttle

---

## 2C — Bundle size analysis

### 2C.1 Build artifacts (post-`npm run build`)

- **Total `.next/static/`:** 3.7 MB
  - `chunks/`: 3.5 MB across **85 JS files**
  - `media/`: 228 KB (fonts + static images)
  - `quYMIQL5HXzJhakDPYWlT/` (build-id dir): ~0 KB
- **CSS bundle:** not emitted as a separate `css/` dir — Tailwind CSS is inlined or bundled into the route chunks under Turbopack.

### 2C.2 Largest chunks (top 10 by size)

| Rank | Size | Filename |
|---:|---:|---|
| 1 | 227,535 B (≈ **222 KB**) | `0vbh5zezkk_s-.js` |
| 2 | 224,552 B (≈ **219 KB**) | `17g56cnx01qv2.js` |
| 3 | 183,744 B (≈ **179 KB**) | `14q9xpakxt~0t.js` |
| 4 | 159,142 B (≈ **155 KB**) | `0euuo2adp-nk_.js` |
| 5 | 135,177 B (≈ **132 KB**) | `11dfyg3qe2vlz.js` |
| 6 | 135,177 B (≈ **132 KB**) | `150nqj-ifia1u.js` |
| 7 | 112,594 B (≈ **110 KB**) | `03~yq9q893hmn.js` |
| 8 | 109,536 B (≈ **107 KB**) | `100ea6h8vmkr..js` |
| 9 | 99,649 B (≈ **97 KB**) | `0d5_0-ih6tlot.js` |
| 10 | 78,389 B (≈ **77 KB**) | `0_lysbo~.1fx6.js` |

**Top 10 total:** ≈ 1.43 MB (41% of the 3.5 MB JS bundle).

### 2C.3 Turbopack limitation — NO per-route First Load JS table

Next.js 16.2.1 under **Turbopack** does **not** emit the per-route First Load JS size table that webpack builds print. The build output contains only the route tree with static/dynamic flags; shared chunks and per-route sizes are not reported.

Chunk filenames under Turbopack are content-hashed (e.g., `0vbh5zezkk_s-.js`) and do not carry the route name the way webpack's `pages/[route]-[hash].js` did. This means mapping a chunk back to the module that produced it requires one of:

- **Next.js bundle analyzer** (`@next/bundle-analyzer`) — would require re-running the build through webpack or adding an analyzer plugin.
- **`source-map-explorer`** on emitted `.js.map` files — Turbopack emits source maps; feasible but time-boxed to D-8b/D-8c.
- **Chrome DevTools → Network → Coverage** — tells you which chunks are loaded on a given route, but still needs in-browser verification.

### 2C.4 Recommendations for D-8c

1. Audit the top 3 chunks (≈ 222 KB, 219 KB, 179 KB). Likely suspects:
   - Recharts (used in Money views) — known to be ~100 KB gzipped.
   - Lucide icons (often full-bundle if imported from barrel) — check whether we use `lucide-react/icons/*` direct imports vs the top-level barrel.
   - Supabase client JS — expected ≥ 60 KB.
2. Enable `@next/bundle-analyzer` in a one-off branch to confirm which modules live in the largest chunks.
3. Target: no single chunk > 170 KB; total JS < 3 MB.

### 2C.5 Severity of current baseline

- **Current:** 3.7 MB total static (3.5 MB JS). This is **acceptable for a feature-rich dashboard SaaS** but on the heavy side for mobile-first use.
- **No BLOCKER.** Flag as **MEDIUM** for D-8c follow-up analysis.

---

## 2D — React DevTools re-render profile

**Status:** **REQUIRES LIVE BROWSER — see D-8b checklist below.**

Cannot be observed from source: re-render counts only exist at runtime.

### D-8b checklist — Jacob, run this manually

Use React DevTools → Profiler tab. For each scenario:
1. Open React DevTools → Profiler.
2. Start recording.
3. Perform the interaction below.
4. Stop recording.
5. Note component re-render counts, render durations, and any components rendering that shouldn't be.

**Scenarios (3 total):**

1. **Calendar event selection** — on `/calendar` (month view), click a day, then click an event.
   - Measure: how many `MonthGrid` / `DayCell` components re-render?
   - Expected: only the popover + the clicked cell. If the entire grid re-renders, flag.

2. **Agenda filter change** — on `/calendar` (agenda view), toggle the financials filter.
   - Measure: does the full agenda re-render, or only the affected rows?
   - Expected: only the rows with `amount != null` should visibly change.

3. **Dashboard refresh via route remount** — navigate away from `/dashboard` and back.
   - Measure: first render vs subsequent renders; any long tasks.

**Recording template:**

```
## Calendar event selection
- Components that re-rendered: __
- Total render time: __ ms
- Longest individual render: __ component, __ ms
- Unexpected re-renders:
- Recommendations:
```

**Record at:** `design-audit/d8b-rerender-findings.md`.

**Severity scale:**
- **BLOCKER** — single interaction re-renders > 50 components or takes > 200ms
- **HIGH** — full-grid re-render on a localized change
- **MEDIUM** — avoidable re-render chain (e.g., whole list re-renders when one row changes)
- **LOW** — negligible re-render; profiling opportunity only

---

## D-8a performance summary

- **Build:** green (npm run build exits 0).
- **Total bundle:** 3.7 MB static / 3.5 MB JS across 85 chunks.
- **Largest chunk:** 222 KB.
- **Per-route First Load JS:** **not measurable under Turbopack** without an analyzer plugin. Flagged as D-8b follow-up.
- **No BLOCKER findings from static analysis.**
- **MEDIUM follow-ups:** (1) investigate top 3 chunks, (2) wire up bundle analyzer in a branch, (3) all live-browser sections below.

### Scaffolded for D-8b (live-browser verification)

- 2A Lighthouse (8 routes × 2 devices × 8 metrics)
- 2B Frame / 60fps recordings (3 scenarios)
- 2D React DevTools re-render profile (3 scenarios)

---
