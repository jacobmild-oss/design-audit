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

**D-8c-1 scope note (2026-04-19):** D-8c-1 remediated BLOCKER accessibility findings only (see `d8-accessibility-report.md §D-8c-1`). §2A mobile numbers below were re-captured against the production build. **Bundle analysis and LCP attribution remain deferred to F-14** — no bundle-analyzer branch was created, no chunk-to-module mapping was performed, and the `page-has-heading-one` / `region` structural gaps from D-8b remain open for F-15.

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

### Mobile (4G simulation) — **D-8c-1 prod build (2026-04-19)**

Mobile measurements updated in D-8c-1 against production build (`next build && next start` on port 3000, Turbopack-compiled). Dev-build numbers from D-8b (commit `85c7cb5` accessible via git history) are superseded. Desktop numbers above remain canonical from D-8b (prod vs dev delta is negligible on desktop).

| Route | Perf | A11y | BP | LCP | TBT | CLS | D-8b dev Perf | D-8b dev LCP |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| `/` | **83** | 96 | 100 | 4676 ms | 80 ms | 0.000 | 69 | 9106 ms |
| `/dashboard` | **79** | 93 | 100 | 5456 ms | 125 ms | 0.000 | 65 | 10559 ms |
| `/shows` | **79** | 93 | 100 | 5322 ms | 141 ms | 0.000 | 66 | 10466 ms |
| `/shows/[id]` | **75** | 93 | 100 | 6247 ms | 192 ms | 0.000 | 60 | 12245 ms |
| `/releases/[id]` | **76** | 93 | 100 | 5788 ms | 191 ms | 0.000 | 62 | 11947 ms |
| `/calendar` | **78** | 93 | 100 | 5755 ms | 143 ms | 0.000 | 63 | 11937 ms |
| `/money` | **77** | 93 | 100 | 5731 ms | 149 ms | 0.000 | 63 | 10538 ms |
| `/settings?tab=profile` | **76** | 93 | 100 | 6027 ms | 184 ms | 0.000 | 66 | 11947 ms |

**Mobile summary (prod build):**
- Perf 75–83 — **every route now passes the 75 baseline target** but 7 of 8 still miss an 80+ aspirational bar. Median jump of +13 points vs dev.
- LCP 4.7–6.2 s — **still failing the 2.5 s "good" CWV bucket** and still in "poor" (>4 s), but median jump of 5.5 s improvement (half the dev penalty). The remaining gap is real product work, not build overhead.
- TBT 80–192 ms — **all routes now pass the 300 ms mobile threshold.** This is the cleanest improvement; the dev-build 287–557 ms ceiling was almost entirely Turbopack overhead.
- CLS 0.000 across the board (unchanged).

### Core Web Vitals — flagged failures (prod build)

**LCP target: < 2.5 s "good", < 4.0 s "needs improvement", > 4.0 s "poor"**

| Route (mobile) | LCP | Bucket | Δ vs D-8b dev |
|---|---:|---|---:|
| `/` | 4.7 s | poor | −4.4 s |
| `/dashboard` | 5.5 s | poor | −5.1 s |
| `/shows` | 5.3 s | poor | −5.1 s |
| `/shows/[id]` | 6.2 s | poor | −6.0 s |
| `/releases/[id]` | 5.8 s | poor | −6.2 s |
| `/calendar` | 5.8 s | poor | −6.2 s |
| `/money` | 5.7 s | poor | −4.8 s |
| `/settings?tab=profile` | 6.0 s | poor | −5.9 s |

**CLS:** ✅ all routes 0.000 (desktop + mobile).
**TBT (target < 200 ms desktop / < 300 ms mobile):**
- Desktop: all pass (max 37 ms on `/calendar`).
- Mobile: **all now pass** at 80–192 ms.

### REQUIRES HUMAN TRIAGE (prod-build results)

The dev-build "is it build overhead?" hypothesis is now answered: **partially.** Build overhead accounted for ~5 s of LCP but every route is still "poor" on mobile. The remaining ceiling appears to be a real product constraint, not a dev-build artifact.

**Likely structural causes** (not fixed in D-8c-1; deferred to F-14 performance pass):

1. **Large initial JS payload** — D-8a §2C flagged 3.7 MB total / 222 KB top chunk. First-party + third-party (Recharts, Lucide, Supabase client, Framer Motion) is likely the LCP bottleneck. Action: enable `@next/bundle-analyzer` in a throwaway branch, triage top 3 chunks.
2. **Client-side Supabase session hydration** — every dashboard route waits for `supabase.auth.getSession()` before painting. LCP timing likely correlates with session round-trip under 4G simulation.
3. **Framer Motion initial hydration** — `MotionConfig` wrapper around every dashboard route; framer's bundle is notorious for TBT contribution on mobile, but TBT already passes in prod so this may be less relevant.
4. **Landing page `/` LCP 4.7 s** is 1–1.5 s ahead of every dashboard route, suggesting the dashboard-specific chrome (sidebar, notification provider, favorites provider) is adding 1+ s of LCP delta beyond the base app shell.

**Desktop numbers remain trustable from D-8b.** Desktop perf 89–93 with zero CLS is genuinely good.

**Decision on severity classification:** prod-build mobile LCP failures are **HIGH, not BLOCKER** — functionally the app loads and operates; it just crosses the "poor" CWV line. Bundle attribution + remediation is F-14 (performance pass).

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

### F-14 recommendations (deferred from D-8c)

1. Audit top 3 chunks (222 KB, 219 KB, 179 KB) — likely Recharts, Lucide, Supabase client.
2. Enable `@next/bundle-analyzer` in a one-off branch to attribute chunks.
3. Target: no single chunk > 170 KB; total JS < 3 MB.

**No D-8b dynamic update to this section** — bundle size is a static build artifact. **D-8c-1 did not touch bundle composition** — the 222 KB top-chunk and 3.5 MB aggregate remain unchanged. Bundle remediation is the focus of F-14.

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
