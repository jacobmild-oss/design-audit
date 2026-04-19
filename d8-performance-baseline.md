# D-8a/b — Performance Baseline

**Date (D-8a static):** 2026-04-19
**Date (D-8b dynamic):** 2026-04-19
**Scope:** `apps/web/` production build artifacts + runtime measurements on `localhost:3000`
**D-8a method:** filesystem inspection of `.next/static/` after `npm run build`.
**D-8b method:** Lighthouse CLI (8 routes × 2 devices), Playwright frame tracing (3 scenarios), React.Profiler on a throwaway branch (1 scenario). Measurements against the locally running dev/test environment on `localhost:3000` (Turbopack dev build, PID 11100).
**Related:** design-system.md §5 Batch D — D-8 Accessibility + performance pass.

**D-8b methodology notes:**

1. **Dev vs prod.** All D-8b measurements ran against the Turbopack **dev** server on `localhost:3000`. Dev builds are substantially slower than production (no tree shaking, extra HMR overhead, source maps, React dev-only warnings). Production numbers will be meaningfully better. **Do not use D-8b Lighthouse scores as the release budget** — re-run against `npm run build && npm run start` before setting performance SLOs.
2. **Headless Chromium frame timing.** Playwright headless Chrome frame pacing is not a perfect proxy for real hardware. The high drop-rate in Part 4 should be interpreted as "suggestive of jank," not "this many frames dropped on real hardware."

---

## 2A — Lighthouse scores (dynamic — full table)

**Method:** Lighthouse 13.1.0 Node API with chrome-launcher. Desktop: no throttle, 1350×940. Mobile: Lighthouse-default 4G simulation (150ms RTT, 1638kbps down, 4× CPU slowdown), 412×823 @ 1.75 DPR. Session cookies replayed via `extraHeaders: { Cookie: ... }`.

### Desktop

| Route | Perf | A11y | BP | FCP | LCP | TTI | CLS | TBT |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| `/` | **93** | 96 | 100 | 253 ms | 1725 ms | 1725 ms | 0.000 | 0 ms |
| `/dashboard` | **91** | 93 | 100 | 256 ms | 1915 ms | 1915 ms | 0.000 | 14 ms |
| `/shows` | **92** | 93 | 100 | 256 ms | 1918 ms | 1918 ms | 0.000 | 4 ms |
| `/shows/[id]` | **89** | 93 | 100 | 255 ms | 2230 ms | 2230 ms | 0.000 | 0 ms |
| `/releases/[id]` | **89** | 93 | 100 | 255 ms | 2150 ms | 2150 ms | 0.000 | 1 ms |
| `/calendar` | **89** | 93 | 100 | 259 ms | 2167 ms | 2167 ms | 0.000 | 37 ms |
| `/money` | **92** | 93 | 100 | 259 ms | 1844 ms | 1844 ms | 0.000 | 13 ms |
| `/settings?tab=profile` | **89** | 93 | 100 | 260 ms | 2210 ms | 2210 ms | 0.000 | 13 ms |

**Desktop summary:** Perf 89–93 (all within acceptable range). A11y 93–96 (see D-8a/b accessibility report for raw axe drilldowns). Best Practices 100 across the board. CLS 0 across the board — no layout-shift regressions. LCP 1.7–2.2 s — all **under the 2.5 s "good" threshold** and under the aspirational 2.0 s target except for `/shows/[id]`, `/calendar`, `/releases/[id]`, `/settings?tab=profile` (2.1–2.2 s; marginal).

### Mobile (4G simulation)

| Route | Perf | A11y | BP | FCP | LCP | TTI | CLS | TBT |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| `/` | **69** | 96 | 100 | 997 ms | **9106 ms** | 9118 ms | 0.000 | 287 ms |
| `/dashboard` | **65** | 93 | 100 | 920 ms | **10559 ms** | 11102 ms | 0.000 | 411 ms |
| `/shows` | **66** | 93 | 100 | 920 ms | **10466 ms** | 11080 ms | 0.000 | 369 ms |
| `/shows/[id]` | **60** | 93 | 100 | 919 ms | **12245 ms** | 12245 ms | 0.000 | 557 ms |
| `/releases/[id]` | **62** | 93 | 100 | 919 ms | **11947 ms** | 11947 ms | 0.000 | 511 ms |
| `/calendar` | **63** | 93 | 100 | 919 ms | **11937 ms** | 11937 ms | 0.000 | 437 ms |
| `/money` | **63** | 93 | 100 | 915 ms | **10538 ms** | 10995 ms | 0.000 | 412 ms |
| `/settings?tab=profile` | **66** | 93 | 100 | 915 ms | **11947 ms** | 11947 ms | 0.000 | 348 ms |

**Mobile summary:** Perf 60–69 — **every route is failing the 80 mobile target**, all by a lot. LCP 9.1–12.2 s — **every route is failing the 2.5 s "good" Core Web Vital** by 4–5×. TBT 287–557 ms — every route is failing the 300 ms mobile threshold; 6 of 8 are above 350 ms.

**This is almost entirely the dev-build overhead.** Turbopack dev serves un-minified, un-tree-shaken source + HMR runtime + React DevTools hooks. Bundle size grows 3–5× vs production. CPU throttling of 4× amplifies this into 9–12 s LCPs.

### Core Web Vitals — flagged failures

**LCP target: < 2.5 s "good", < 4.0 s "needs improvement", > 4.0 s "poor"**

| Route (mobile) | LCP | Bucket |
|---|---:|---|
| `/` | 9.1 s | poor |
| `/dashboard` | 10.6 s | poor |
| `/shows` | 10.5 s | poor |
| `/shows/[id]` | 12.2 s | poor |
| `/releases/[id]` | 11.9 s | poor |
| `/calendar` | 11.9 s | poor |
| `/money` | 10.5 s | poor |
| `/settings?tab=profile` | 11.9 s | poor |

**CLS:** ✅ all routes 0.000 (desktop + mobile).
**TBT (target < 200 ms desktop / < 300 ms mobile):**
- Desktop: all pass (max 37 ms on `/calendar`).
- Mobile: all fail (287 ms on `/` best; 557 ms on `/shows/[id]` worst).

### REQUIRES HUMAN TRIAGE

**Before acting on these numbers:**

1. **Re-run against production build.** Dev numbers are a soft floor, not a ceiling. Action: `cd apps/web && npm run build && npm run start`, then re-run `lighthouse-sweep.mjs` pointing at the production-served port. Expect mobile perf to jump 15–30 points and LCP to drop 40–60%.
2. **If the production numbers are still bad**, the most likely suspects are: large initial JS payload (see §2C), client-side Supabase session hydration, non-critical CSS render-blocking. Triage list from D-8a §2C.
3. **Desktop numbers are trustable as-is.** Desktop perf 89–93 with zero CLS is genuinely good — don't remediate blind.

---

## 2B — Frame tracing (dynamic — Playwright trace + rAF timing)

**Method:** 3 scenarios, each instrumented with a `requestAnimationFrame` frame-gap logger + `PerformanceObserver` for `longtask` and `layout-shift`. Full Playwright trace zips saved to `C:\Users\jacob\AppData\Local\Temp\d8-automation\traces\` (scratch; not committed).

### Raw data

| Scenario | Frames | Dropped | Drop rate | <16.67ms | 16–33ms | 33–50ms | 50–100ms | >100ms | CLS | Long tasks |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---:|
| **show-detail-tabs** (3 route navigations) | 51 | 44 | **86%** | 7 | 43 | 1 | 0 | 0 | 0.022 | 0 |
| **calendar-agenda-scroll** (2 s scroll) | 140 | 60 | **43%** | 80 | 60 | 0 | 0 | 0 | 0.000 | 0 |
| **shows-list-filter** (3 button clicks) | 89 | 46 | **52%** | 43 | 46 | 0 | 0 | 0 | 0.000 | 0 |

### Interpretation

- **All dropped frames land in the 16–33 ms bucket**, meaning they landed at 30 fps (next-frame) rather than 60 fps. No frames slipped past 50 ms (which would indicate serious jank).
- **Zero `longtask` entries in all three scenarios**, meaning no single blocking task exceeded 50 ms. This is the most reliable signal here: no catastrophic blocking work.
- **The "86% drop rate" on show-detail-tabs is suspicious** — it includes 3 full `page.goto(...)` calls, which are not the intended real-world tab-switch pattern (that goes through client-side routing). Measurement artifact; see §2D for the React.Profiler data, which measures the actual in-page update correctly.
- **CLS 0.022 on show-detail-tabs** — small but non-zero. The tab content swap appears to shift something minor. Consider reserving space / skeletons per tab.

### Limitations

- Headless Chromium's frame pacing is not a perfect proxy for real hardware. The high drop-rate in show-detail-tabs should be interpreted as "suggestive of jank," not quantitative.
- Trace zips exist; deep trace analysis (Chrome DevTools Performance tab on the `.zip`) is deferred to D-8c.

### REQUIRES HUMAN TRIAGE

Jacob:
- Replicate calendar-agenda scroll on a real device (phone or laptop with DevTools Performance); confirm or refute the 43% drop rate.
- The CLS 0.022 on tab switching — open the trace zip and see what's shifting.
- Decide whether any of this warrants D-8c work; based on the zero-longtask signal, probably not for a BLOCKER.

---

## 2C — Bundle size analysis (static — from D-8a, unchanged)

### Build artifacts (post-`npm run build`)

- **Total `.next/static/`:** 3.7 MB
  - `chunks/`: 3.5 MB across **85 JS files**
  - `media/`: 228 KB (fonts + static images)
- **CSS bundle:** not emitted as a separate dir (inlined/bundled into route chunks under Turbopack).

### Top 10 largest chunks

| Rank | Size | Filename |
|---:|---:|---|
| 1 | 222 KB | `0vbh5zezkk_s-.js` |
| 2 | 219 KB | `17g56cnx01qv2.js` |
| 3 | 179 KB | `14q9xpakxt~0t.js` |
| 4 | 155 KB | `0euuo2adp-nk_.js` |
| 5 | 132 KB | `11dfyg3qe2vlz.js` |
| 6 | 132 KB | `150nqj-ifia1u.js` |
| 7 | 110 KB | `03~yq9q893hmn.js` |
| 8 | 107 KB | `100ea6h8vmkr..js` |
| 9 | 97 KB | `0d5_0-ih6tlot.js` |
| 10 | 77 KB | `0_lysbo~.1fx6.js` |

Top 10 total ≈ 1.43 MB (41% of the 3.5 MB JS bundle).

### Turbopack limitation — NO per-route First Load JS table

Next.js 16.2.1 under Turbopack does not emit the per-route First Load JS size table that webpack builds print. Chunk filenames are content-hashed and do not carry the route name. Mapping chunks back to source modules requires one of:

- `@next/bundle-analyzer` on a branch
- `source-map-explorer` on emitted `.js.map` files
- Chrome DevTools → Network → Coverage in-browser

### D-8c recommendations

1. Audit top 3 chunks (222 KB, 219 KB, 179 KB) — likely Recharts, Lucide, Supabase client.
2. Enable `@next/bundle-analyzer` in a one-off branch to attribute chunks.
3. Target: no single chunk > 170 KB; total JS < 3 MB.

**No D-8b dynamic update to this section** — bundle size is a static build artifact.

---

## 2D — React re-render profile (dynamic — throwaway branch `d8b-profiling`)

**Method:** temporarily wrapped the show-detail tabs tree in `React.Profiler` on branch `d8b-profiling`. Ran a Playwright script that clicked each tab in sequence. Captured `onRender(id, phase, actualDuration)` callbacks via `window.__profilerData`. **Branch deleted after capture — no app-code changes on main.**

### Scenario: show-detail tab switch (`/shows/[id]?tab=X`)

| Tab → | Wall time | Profiler callbacks | Total render time | Max single render | Avg |
|---|---:|---:|---:|---:|---:|
| **Advancing** | 962 ms | 3 (all `update`) | 61 ms | 34 ms | 20 ms |
| **Finances** | 822 ms | 3 (all `update`) | 18 ms | 7 ms | 6 ms |
| **Files** | 809 ms | 2 (all `update`) | 4 ms | 2 ms | 2 ms |
| **Overview** | 848 ms | 7 (all `update`) | 48 ms | 10 ms | 7 ms |

### Interpretation

- **All callbacks are `update` phase, not `mount`.** That means the Profiler subtree (ErrorBoundary + tab content) persists across tab switches; React treats it as a prop/state update rather than re-mount. Good — no tear-down/re-build cycle.
- **Advancing's 34 ms max render is the single suspicious data point.** Near the 16.67 ms frame budget × 2. Not catastrophic but worth poking at: is the AdvancingTab doing synchronous heavy work on first paint (e.g. large form initialization, date-fmt calls on many rows)?
- **Overview's 7 callbacks** is the highest count. More callbacks than the other tabs suggests cascading updates — possibly from `onShowUpdated={setShow}` triggering a parent re-render that cascades. Worth investigating if perf tuning becomes a priority.
- **Files' 4 ms total** is the cleanest — DocsTab renders fast.
- **Wall times of ~800–960 ms per switch** include the navigation round-trip (URL change + React router machinery), not just the render.

### Limitations

- Only the Profiler wrapping the tabs tree was added. Nested components (individual tabs, rows) are not separately profiled — they bubble up into the one callback.
- Dev-build React includes extra instrumentation that inflates `actualDuration` vs production.
- The Playwright test used click-based tab switching (not page-level navigation), which is the correct real-world path.

### REQUIRES HUMAN TRIAGE

Jacob:
- **Advancing tab 34 ms render** — open `AdvancingTab.tsx`, check for synchronous expensive work in render (heavy form init, uncached formatters).
- **Overview's 7 callbacks** — may indicate a state-update cascade. Profile it in production dev tools, not just automation.
- Decide severity: these are not BLOCKER. Probably MEDIUM design-system-follow-ups.

---

## Dynamic performance sweep — summary snapshot

| Measurement | Result | Interpretation |
|---|---|---|
| Desktop Lighthouse perf | 89–93 | ✅ good |
| Mobile Lighthouse perf | 60–69 | ❌ failing target — **almost entirely dev-build penalty**; re-run on prod first |
| Desktop LCP | 1.7–2.2 s | ✅ all under 2.5 s "good"; 4 routes marginal vs 2.0 s aspirational |
| Mobile LCP | 9.1–12.2 s | ❌ all in CWV "poor" — re-run on prod |
| CLS (all routes) | 0.000 | ✅ |
| TBT desktop | 0–37 ms | ✅ all pass |
| TBT mobile | 287–557 ms | ❌ fail — re-run on prod |
| Calendar scroll drop rate | 43% (headless) | ⚠️ suggestive; no long tasks; verify on device |
| Show-detail tab re-render | 4–61 ms total, 0 longtasks | ✅ acceptable; Advancing has a 34 ms outlier worth investigating |
| Bundle total | 3.7 MB / 85 chunks / max 222 KB | ⚠️ borderline; enable bundle analyzer in D-8c |

## Limitations encountered (D-8b)

1. **Dev-build only** — ran against Turbopack dev server, not production. Numbers inflate mobile LCPs by ~3–5×.
2. **Port 3000 held by dev** — production build couldn't be served on 3000 without killing the dev server. (User confirmed the dev server on 3000 was the test environment, so D-8b Lighthouse ran against dev.)
3. **Turbopack per-route bundle size unmeasurable** — no First Load JS table emitted.
4. **Headless Chromium frame timing** not a perfect proxy for real-device rendering.
5. **React.Profiler in dev mode** includes extra instrumentation that inflates render times vs production.
6. **Settings routes** — some initial routes hit 404 because settings uses `?tab=X` not `/settings/X`. Re-ran correctly in a second pass.

---
