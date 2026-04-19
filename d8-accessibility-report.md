# D-8a/b — Accessibility Verification

**Date (D-8a static):** 2026-04-19
**Date (D-8b dynamic):** 2026-04-19
**Scope:** `apps/web/` — all dashboard + auth routes, calendar views
**D-8a method:** source-only analysis (contrast math, grep audits).
**D-8b method:** Playwright + axe-core against a live session on `localhost:3000` (dev/test env, Turbopack).
**Related:** design-system.md §5 Batch D — D-8 Accessibility + performance pass.

**D-8b methodology note.** Measurements were taken against the locally running dev/test environment on `localhost:3000` (Turbopack dev build, PID 11100). Production-build numbers will differ. Axe sweep covered 40 route-viewport combinations (29 desktop + 11 mobile). Manual login handoff was used to capture `storageState.json`; the file was deleted from scratch before commit. All raw findings are in the scratch dir (`C:\Users\jacob\AppData\Local\Temp\d8-automation\*.json`) and are not committed.

---

## 1A — Colorblind simulation (dynamic — axe sweep stand-in; full simulation still requires human eyes)

**D-8b status:** partially answered by axe-core `color-contrast` rule. Full colorblind simulation (deuteranopia etc.) still requires human-eye review and is not automatable — deferred.

**D-8b finding (axe):** `color-contrast` fires on **35 of 40 route-viewport runs** (246 element-node occurrences across all runs). Impact: `serious`. WCAG 2 AA tag: `wcag143`.

This validates and amplifies D-8a §1B: the static contrast math flagged three body-text-failing tokens (`--color-fg-faint`, `--color-status-mastering`, `--color-chart-neutral`) and partial failures (`--color-chart-expense` on raised/float, `--color-gear-percussion` on float). Axe's runtime scan finds many more concrete instances where those tokens (or adjacent compositions) render below AA contrast on real surfaces.

**REQUIRES HUMAN TRIAGE.** Axe reports element-level contrast violations but cannot say which are the user-impact critical ones. Jacob: open `axe-findings.json` (scratch) and page through contrast violations by route; promote any touching running prose to HIGH, chip/decorative-text to MEDIUM.

### Deuteranopia / protanopia / tritanopia / achromatopsia — still required

Axe does not simulate colorblindness. Full D-8a scaffolded checklist stands:

- Open Chrome DevTools → Rendering → Emulate vision deficiencies
- Run the 5 calendar views, 4 money views, dashboard, all release/show details under each of 4 modes
- Record: events/statuses/amounts/accent distinguishable? artist color pairs distinguishable?

---

## 1B — WCAG contrast math (static, from D-8a — unchanged)

(Same contrast table as D-8a. Validated by axe-core: every token flagged in 1B also appears in axe's `color-contrast` violations.)

Relative luminance per WCAG 2.x. Each color token measured against the four surface tokens (`--color-base`, `--color-panel`, `--color-raised`, `--color-float`). Values rounded to 2 decimals.

**Thresholds:** AA body ≥ 4.5:1 / AA large ≥ 3.0:1 / AA UI ≥ 3.0:1.

| Token | Hex | base | panel | raised | float | Notes |
|---|---|---:|---:|---:|---:|---|
| `--color-fg` | `#fafafa` | 19.06 | 18.47 | 17.77 | 16.61 | ✅ |
| `--color-fg-muted` | `#8a8a95` | 5.83 | 5.64 | 5.43 | 5.08 | ✅ |
| `--color-fg-faint` | `#6e6e79` | 3.95 | 3.83 | 3.68 | 3.44 | ⚠️ fails body AA everywhere |
| `--color-accent` | `#f97316` | 7.10 | 6.88 | 6.62 | 6.18 | ✅ |
| `--color-accent-hover` | `#ea580c` | 5.59 | 5.41 | 5.21 | 4.87 | ✅ |
| `--color-info` | `#00e5ff` | 12.93 | 12.53 | 12.06 | 11.27 | ✅ |
| `--color-success` | `#00ff88` | 14.84 | 14.37 | 13.83 | 12.93 | ✅ |
| `--color-warning` | `#FFB020` | 10.88 | 10.54 | 10.14 | 9.48 | ✅ |
| `--color-danger` | `#ff3366` | 5.61 | 5.43 | 5.23 | 4.88 | ✅ |
| `--color-hold` | `#a78bfa` | 7.31 | 7.08 | 6.81 | 6.37 | ✅ |
| `--color-status-hold` | `#8b7ec9` | 5.61 | 5.44 | 5.23 | 4.89 | ✅ |
| `--color-status-mastering` | `#6b5eb0` | 3.65 | 3.54 | 3.40 | 3.18 | ⚠️ fails body AA everywhere |
| `--color-status-day-of` | `#e8621f` | 5.87 | 5.69 | 5.47 | 5.12 | ✅ |
| `--color-status-settled` | `#4a9b6e` | 5.88 | 5.69 | 5.48 | 5.12 | ✅ |
| `--color-status-warning` | `#d49450` | 7.72 | 7.48 | 7.20 | 6.73 | ✅ |
| `--color-role-show-side` | `#5db8c2` | 8.62 | 8.35 | 8.04 | 7.51 | ✅ |
| `--color-role-commerce` | `#9176c5` | 5.31 | 5.14 | 4.95 | 4.62 | ✅ |
| `--color-role-press` | `#c47aa0` | 6.29 | 6.09 | 5.86 | 5.48 | ✅ |
| `--color-gear-stringed` | `#c87a4a` | 6.01 | 5.82 | 5.60 | 5.23 | ✅ |
| `--color-gear-percussion` | `#9b7a3f` | 4.97 | 4.82 | 4.64 | 4.33 | ⚠️ fails AA on `float` |
| `--color-gear-electronic` | `#6a8ac2` | 5.71 | 5.53 | 5.32 | 4.98 | ✅ |
| `--color-chart-income` | `#3fa473` | 6.41 | 6.21 | 5.98 | 5.59 | ✅ |
| `--color-chart-expense` | `#c4574f` | 4.57 | 4.43 | 4.26 | 3.98 | ⚠️ fails AA on `raised`/`float` |
| `--color-chart-neutral` | `#6e6e78` | 3.95 | 3.82 | 3.68 | 3.44 | ⚠️ fails body AA everywhere (by D-7 design — orphan events) |

**Aliases:** `--color-role-advisory`, `--color-crew-logistics`, `--color-status-frozen` → `--color-fg-muted` (pass); `--color-status-confirmed` → `--color-success` (pass); `--color-crew-technical` → `--color-role-show-side` (pass); `--color-crew-commercial` → `--color-role-commerce` (pass).

### D-8c recommendations (carried over from D-8a)

1. Grep every `text-fg-faint` usage; flag any ≥ 13px regular-weight running text.
2. Grep `--color-status-mastering` usages; confirm no plain-text rendering (chip-only is safe).
3. For `--color-chart-neutral` (orphan events): add a secondary non-color cue (dashed border / "◇" prefix) so the intentionally-muted contrast doesn't erase the event for low-vision users.
4. `chart-expense` and `gear-percussion`: audit whether any card-interior (`bg-raised`) or modal (`bg-float`) usage exists; move to `bg-panel` if so.

---

## 1C — Non-color cue audit (static — unchanged)

Findings per D-8a source scan. Cross-reference with D-8b `button-name` axe finding: the 21-route pattern of unnamed buttons **includes** the notification-bell case I flagged in 1C + many more.

### PASS — status shown with adjacent label

- Shows / Contacts / Crew / Releases list pages: status/role dot + label text adjacent.
- Tasks cards: `TaskCard.tsx:36-41` — icon + priority label + color.
- Cancelled shows: `line-through` text decoration (`shows/page.tsx`, `dashboard/page.tsx`, calendar `shared.tsx`, `TaskCard.tsx`, `PackingTab`).
- Calendar event type: `AgendaView.tsx:124-138` — icon carries TYPE, color carries ARTIST.

### FAIL — status by color alone (validated / expanded by axe)

| Finding | Location | D-8b axe echo |
|---|---|---|
| Dashboard `StatusDot` no adjacent label in some card headers | `dashboard/page.tsx:150-160` | — (not directly detected by axe) |
| MiniMonth event dots (first-event artist color, no glyph) | `MiniMonth.tsx:73-80` | — (visual-only; axe can't see semantics) |
| Notification bell button lacks `aria-label` (dot has one) | `(dashboard)/layout.tsx:55-66` | ✅ axe `button-name` fires here (21 routes × ~3 nodes each = 78 total instances of unnamed buttons across the app) |

---

## 1D — Chart shape encoding (axe-assisted; full visual still requires human review)

**D-8b status:** axe did not flag chart-specific a11y issues on `/money` (1 contrast + 1 page-has-heading-one only). That's consistent with Money using Recharts, which produces `<svg>` with `role="img"` + `<title>`.

Axe cannot, however, tell whether legend markers are shape-encoded or just color-coded. **Still requires human eye.**

### Jacob checklist (unchanged from D-8a)

- Money views (income/expense bars, dashboard financial summary) — legend markers ✅/❌ shape-encoded?
- Under achromatopsia simulation, are series distinguishable?

---

## 1E — Alert icon audit (static — unchanged)

2 FAILs: `login/page.tsx:249` and `reset-password/page.tsx:81,122` — plain `<p role="alert">` red text, no icon. PASS: ErrorFallback, Toast, UpgradeBanner.

D-8b cross-check: axe does not flag this because `role="alert"` is semantically correct — the WCAG 1.3.3 "sensory characteristics" rule is not axe-automatable. **D-8a gap covered; no dynamic contradiction.**

---

## 1F — Focus ring audit (D-8a + D-8b limited dynamic walk)

### D-8a static findings (unchanged)

7 components lack `focus-visible:` classes: Calendar prev/next/Today, UpgradeBanner CTA + dismiss, ErrorFallback Retry + Report, Shows view-toggle.

### D-8b dynamic walk — HIGH LIMITATION, do not trust as refutation

Playwright walked the first ~60 visible focusable elements on 5 priority routes and programmatically called `el.focus()`. Results:

| Route | Missing indicator | Checked |
|---|---:|---:|
| `/dashboard` | 0 | 57 |
| `/shows` | 1 | 41 |
| `/shows/[id]` | 1 | 54 |
| `/calendar?view=agenda` | 0 | 60 |
| `/settings/profile` | 0 | 0 (route 404 in this dev run — settings uses `?tab=profile`, not pathed) |

**This appears to contradict D-8a**, but the methodology is flawed:

- `el.focus()` triggers the `:focus` pseudo-class (which browsers style with default `outline`), **not the `:focus-visible` pseudo-class** (which requires keyboard-originated focus for most heuristics).
- Tailwind's `focus-visible:ring-2 ...` pattern only applies styles under `:focus-visible`. A programmatic `.focus()` call may or may not trigger it.
- Conclusion: **D-8a source scan remains authoritative for focus-visible gaps.** D-8b dynamic walk is unreliable and should be re-done using real keyboard Tab events (Playwright `page.keyboard.press('Tab')`) in D-8c remediation follow-up.

### REQUIRES HUMAN TRIAGE

Jacob: open D-8a findings list (7 HIGH-severity components) and confirm via keyboard Tab in browser DevTools before D-8c remediation planning.

---

## 1G — Mobile touch target audit (dynamic — Playwright boundingBox sweep)

**D-8b method:** Playwright on iPhone 14 Pro (390×844) + Pixel 7 (412×915). DOM walk for `button, [role=button], a[href], [onclick], input[type=checkbox], [role=tab], [role=link]`. Flagged any element with `width < 44 OR height < 44`.

### Raw findings (across 5 routes × 2 viewports = 10 runs)

| Route | iPhone 14 Pro | Pixel 7 |
|---|---:|---:|
| `/calendar?view=agenda` | 49/125 | 49/125 |
| `/shows/[id]` | 38/55 | 38/55 |
| `/contacts/[id]` | 39/39 | 39/39 |
| `/releases/[id]` | 36/46 | 36/46 |
| `/money` | 40/40 | 40/40 |

**Totals:** 404 sub-44 target incidents across 10 runs; no variance between iPhone and Pixel (same Chromium render).

### Tag distribution of sub-44 targets

- 210× `button`
- 182× `a` (includes sidebar nav links)
- 12× `button[role=tab]`

### Size bucket (min-dimension)

- **94 targets < 24px** (severe — below browser default)
- **74 targets 24–31px**
- **236 targets 32–39px** (borderline; many are sidebar-nav links at ~35.5px tall)

### Representative samples

- Sidebar nav links on `/money`: 175×35.5 (width generous, height tight — 35.5px is the whole row)
- Sidebar nav links on `/calendar`: same 175×35.5 pattern on ~8 items
- Logo link "ArtistHQ": 82×25.5 (cosmetic — usually not a critical interaction)
- Search box button: 175×32 (wide, but short)
- Small icon buttons at 28×28 on `/money` header

### REQUIRES HUMAN TRIAGE

Jacob: axe's 44px threshold is WCAG 2.5.5 **AAA**; the **AA threshold is 24×24 (WCAG 2.5.8)**. Re-reading the raw data through the AA threshold: only the 94 sub-24 targets are AA failures; the 310 targets in 24–39 are AAA opportunities, not failures.

- Promote the 94 sub-24 findings to HIGH/BLOCKER triage in D-8c.
- Demote the 32–39 bucket to a design-system follow-up (increase `h-7` → `h-11` patterns where practical).
- Confirm the sidebar nav rows: they visually look large enough because of hover background, but hit-target is h-9 (35.5px). D-8c should extend parent padding or use `min-h-[44px]`.

---

## 1H — Toggle: disabled vs off states (static — unchanged)

No dedicated toggle/switch component exists. When added, verify disabled `aria-disabled` + visual distinction from off. No D-8c action.

---

## Dynamic accessibility sweep — raw data (D-8b)

**Axe-core run across 40 route-viewport combinations:**

| Metric | Count |
|---|---:|
| Total axe violations | **94** |
| `critical` impact | **26** |
| `serious` impact | **38** |
| `moderate` impact | **30** |
| `minor` impact | 0 |

**By rule:**

| Rule | Count | Impact | Rough meaning |
|---|---:|---|---|
| `color-contrast` | 35 routes (246 nodes) | serious | Text/background below WCAG AA (see §1B) |
| `button-name` | 21 routes (78 nodes) | critical | Buttons with no accessible name (echoes D-8a notification-bell finding × ~20 more sites) |
| `page-has-heading-one` | 18 routes | moderate | Page is missing an `<h1>` — semantic landmark gap |
| `region` | 10 (mostly mobile) | moderate | Content outside a landmark — navigation gaps |
| `label` | 5 routes (13 nodes) | critical | Form inputs without associated `<label>` |
| `heading-order` | 2 routes | moderate | Heading levels skip (e.g. `<h2>` then `<h4>`) |
| `link-name` | 2 routes (11 nodes) | serious | Anchors with no discernible text |
| `label-title-only` | 1 route | serious | Form element only labeled via `title=` (`/settings?tab=account`) |

### Per-route breakdown (critical/serious/moderate)

| Route | VP | C | S | M | Violated rules |
|---|---|---:|---:|---:|---|
| `/` | desktop | 0 | 1 | 0 | color-contrast |
| `/dashboard` | desktop | 0 | 1 | 1 | color-contrast, heading-order |
| `/dashboard` | mobile | 1 | 2 | 2 | button-name, color-contrast, heading-order, link-name, region |
| `/shows` | desktop | 0 | 1 | 1 | color-contrast, page-has-heading-one |
| `/shows` | mobile | 1 | 1 | 2 | button-name, color-contrast, page-has-heading-one, region |
| `/shows/[id]` | desktop | 1 | 1 | 0 | button-name, color-contrast |
| `/shows/[id]` | mobile | 1 | 1 | 1 | button-name, color-contrast, region |
| `/releases` | desktop | 1 | 1 | 1 | button-name, color-contrast, page-has-heading-one |
| `/releases/[id]` | desktop | 0 | 1 | 0 | color-contrast |
| `/releases/[id]` | mobile | 1 | 1 | 1 | button-name, color-contrast, region |
| `/tasks` | desktop | 0 | 1 | 1 | color-contrast, page-has-heading-one |
| `/contacts` | desktop | 0 | 1 | 1 | color-contrast, page-has-heading-one |
| `/contacts/[id]` | desktop | 0 | 1 | 0 | color-contrast |
| `/contacts/[id]` | mobile | 1 | 1 | 1 | button-name, color-contrast, region |
| `/crew` | desktop | 0 | 1 | 1 | color-contrast, page-has-heading-one |
| `/crew/[id]` | desktop | 0 | 1 | 0 | color-contrast |
| `/gear` | desktop | 1 | 2 | 1 | button-name, color-contrast, link-name, page-has-heading-one |
| `/gear/new` | desktop | 1 | 1 | 0 | color-contrast, label |
| `/merch` | desktop | 0 | 1 | 1 | color-contrast, page-has-heading-one |
| `/merch/[id]` | desktop | 2 | 1 | 0 | button-name, color-contrast, label |
| `/docs` | desktop | 2 | 1 | 1 | button-name, color-contrast, label, page-has-heading-one |
| `/money` | desktop | 0 | 1 | 1 | color-contrast, page-has-heading-one |
| `/calendar` | desktop | 1 | 1 | 1 | button-name, color-contrast, page-has-heading-one |
| `/calendar?view=agenda` | desktop | 1 | 1 | 1 | button-name, color-contrast, page-has-heading-one |
| `/calendar?view=agenda` | mobile | 1 | 1 | 2 | button-name, color-contrast, page-has-heading-one, region |
| `/calendar?view=agenda-mobile` | mobile | 1 | 1 | 2 | button-name, color-contrast, page-has-heading-one, region |
| `/calendar?view=month` | desktop | 1 | 1 | 1 | button-name, color-contrast, page-has-heading-one |
| `/calendar?view=month` | mobile | 1 | 1 | 2 | button-name, color-contrast, page-has-heading-one, region |
| `/calendar?view=week` | desktop | 1 | 1 | 1 | button-name, color-contrast, page-has-heading-one |
| `/calendar?view=week` | mobile | 1 | 1 | 2 | button-name, color-contrast, page-has-heading-one, region |
| `/settings?tab=profile` | desktop | 2 | 1 | 0 | button-name, color-contrast, label |
| `/settings?tab=profile` | mobile | 2 | 1 | 1 | button-name, color-contrast, label, region |
| `/settings?tab=links` | desktop | 0 | 1 | 0 | color-contrast |
| `/settings?tab=account` | desktop | 0 | 1 | 0 | label-title-only |
| `/settings?tab=preferences` | desktop | 0 | 0 | 0 | (clean) |
| `/settings?tab=notifications` | desktop | 1 | 0 | 0 | button-name |
| `/settings?tab=subscription` | desktop | 0 | 1 | 0 | color-contrast |
| `/settings?tab=team` | desktop | 0 | 0 | 0 | (clean) |
| `/settings?tab=band` | desktop | 0 | 1 | 0 | color-contrast |
| `/settings?tab=export` | desktop | 0 | 0 | 0 | (clean) |

---

## Cross-reference: D-8a findings validated by D-8b

| D-8a finding | D-8b result | Status |
|---|---|---|
| `--color-fg-faint` fails body AA everywhere | axe `color-contrast` fires on 35 routes | ✅ **validated** (amplified) |
| `--color-status-mastering` fails body AA | axe `color-contrast` | ✅ **validated** |
| `--color-chart-neutral` fails body AA | axe `color-contrast` | ✅ **validated** (intentional per D-7) |
| `--color-chart-expense` fails on raised/float | axe `color-contrast` | ✅ **validated** |
| Notification bell missing `aria-label` | axe `button-name` (21 routes, 78 nodes — much broader!) | ✅ **validated + expanded** |
| Auth alert `<p role="alert">` no icon | axe silent (sensory-characteristics not automatable) | ⚠️ D-8a-only |
| Calendar nav + ErrorFallback + UpgradeBanner lack focus-visible | D-8b dynamic walk unreliable; D-8a stands | ⚠️ D-8a-only (keep) |
| Shows view-toggle uses `title=` vs `aria-label` | axe silent here | ⚠️ D-8a-only |
| MiniMonth dots no glyph fallback | axe can't see | ⚠️ D-8a-only |

## D-8a gaps surfaced by D-8b

| D-8b finding | D-8a missed | New gap |
|---|---|---|
| `page-has-heading-one` on 18 routes | D-8a didn't audit landmark/heading structure | ✅ **NEW: D-8c** — add `<h1>` to every page shell or confirm via source that an `<h1>` is present |
| `region` on 10 mobile routes | D-8a didn't look for landmarks | ✅ **NEW: D-8c** — wrap main content in `<main>` where missing |
| `label` / `label-title-only` on 6 routes | D-8a's 1H was switches-only | ✅ **NEW: D-8c** — `/gear/new`, `/merch/[id]`, `/docs`, `/settings?tab=profile`, `/settings?tab=account` have form inputs without labels |
| `heading-order` on `/dashboard` | D-8a missed | ✅ **NEW: D-8c** — dashboard heading levels skip |
| `link-name` on 2 routes (11 nodes) | D-8a missed | ✅ **NEW: D-8c** — empty anchor text on /gear and /dashboard-mobile |
| Mobile-only `button-name` regressions | D-8a didn't test mobile viewport | ✅ **NEW**: mobile layout hides labels that desktop shows — check responsive icon-only patterns |

---

## REQUIRES HUMAN TRIAGE — severity promotion/demotion

Axe assigns impact levels (`critical/serious/moderate/minor`) based on generic WCAG weight, not the product's risk profile. The following are surfaced for Jacob to triage into BLOCKER/HIGH/MEDIUM/LOW:

### Critical axe count: 26
- 21 are `button-name` and 5 are `label` / `label-title-only`.
- Jacob should decide: is an unlabelled submit button on `/settings?tab=notifications` BLOCKER or HIGH? (It exists; SR users can't identify the button.)
- Are the 13 unlabelled form inputs on `/gear/new` / `/merch/[id]` / `/docs` BLOCKER? They're on create/edit flows — probably yes.

### Serious axe count: 38
- 35 are `color-contrast`. Most are expected (see §1B). A subset likely reproduces the `chart-expense`-on-raised findings from §1B and should be prioritized.
- 2 are `link-name` + 1 `label-title-only` — HIGH candidates.

### Moderate axe count: 30
- 18 `page-has-heading-one` — structural, ubiquitous; easy fix; probably MEDIUM all.
- 10 `region` on mobile — same.
- 2 `heading-order` on dashboard — probably MEDIUM.

---

## Limitations & caveats

1. **Focus walk is unreliable** — `el.focus()` doesn't trigger `:focus-visible` heuristically. D-8a source scan remains authoritative.
2. **Dev-build measurements** — Turbopack dev vs production parity is unverified. Axe a11y tree is the same in both; contrast the same; bundle-size different (see D-8b performance report).
3. **Some settings paths 404'd in the initial sweep** — settings uses `?tab=X` not `/settings/X`. Re-ran correctly in a second pass. Findings reflect the correct URLs.
4. **No full colorblind simulation** — axe cannot emulate deuteranopia etc. The 64-capture checklist in D-8a §1A still stands for a human review pass.
5. **Touch-target threshold of 44px is AAA** — findings are strictly 44px-AAA; re-read against 24px-AA for actual failures.
6. **Some routes (crew/[id], gear/[id], merch/[id]) were tested only on desktop** — mobile axe sweep was scoped to high-traffic routes only.

---
