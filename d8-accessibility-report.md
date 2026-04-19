# D-8a — Accessibility Verification (Static Slice)

**Date:** 2026-04-19
**Scope:** `apps/web/` — all dashboard + auth routes, calendar views
**Method:** source-only analysis. Dynamic sections (colorblind simulation, chart shape encoding, mobile touch targets) are scaffolded as checklists for D-8b manual verification.
**Related:** design-system.md §5 Batch D — D-8 Accessibility + performance pass.

---

## 1A — Colorblind simulation (deuteranopia, protanopia, tritanopia, achromatopsia)

**Status:** **REQUIRES LIVE BROWSER — see D-8b checklist below.**

Cannot be verified from source: requires rendered pages + Chrome DevTools Rendering → Emulate vision deficiencies (or equivalent). Source inspection confirms D-3 palette separation and D-7 artist-color architecture, but ΔE perception in each deficiency space can only be measured from the composited frame.

### D-8b checklist — Jacob, run this manually

For **each** route listed below, for **each** simulation mode, record:

- Can you still distinguish event types (show / release / tour / task) on the calendar?
- Can you still distinguish status states (confirmed / hold / day-of / cancelled)?
- Can you still distinguish income vs expense on Money views?
- Does the amber accent still read as the primary interactive hint?
- Are any two artist accent colors indistinguishable from each other?

**Routes:**
- `/` (landing)
- `/login`, `/signup`, `/reset-password`
- `/dashboard`
- `/calendar` (month, week, day, agenda views)
- `/shows`, `/shows/[id]`
- `/money` (all tabs: income, expenses, settlements, invoices)
- `/merch`
- `/releases`, `/releases/[id]`
- `/documents`
- `/contacts`, `/crew`, `/gear`, `/tasks`
- `/settings` (all tabs)

**Simulation modes:** deuteranopia, protanopia, tritanopia, achromatopsia (4 modes × ~16 routes = ~64 captures).

**Severity scale (use in findings):**
- **BLOCKER** — critical info is lost (e.g., paid vs unpaid indistinguishable)
- **HIGH** — info is recoverable but slower (e.g., two events look identical at a glance)
- **MEDIUM** — decorative degradation only
- **LOW** — cosmetic

**Record findings at:** `design-audit/d8b-colorblind-findings.md` (TBC).

---

## 1B — WCAG contrast math (source-derived)

Relative luminance per WCAG 2.x. Each color token measured against the four surface tokens (`--color-base`, `--color-panel`, `--color-raised`, `--color-float`). Values rounded to 2 decimals.

**Thresholds:**
- **AA body text:** ≥ 4.5:1
- **AA large text (≥18pt regular / 14pt bold):** ≥ 3.0:1
- **AA non-text UI:** ≥ 3.0:1

| Token | Hex | base `#09090b` | panel `#0e0e11` | raised `#131316` | float `#1a1a1f` | Notes |
|---|---|---:|---:|---:|---:|---|
| `--color-fg` | `#fafafa` | 19.06 | 18.47 | 17.77 | 16.61 | ✅ body text anywhere |
| `--color-fg-muted` | `#8a8a95` | 5.83 | 5.64 | 5.43 | 5.08 | ✅ body on all surfaces |
| `--color-fg-faint` | `#6e6e79` | 3.95 | 3.83 | 3.68 | 3.44 | ⚠️ **FAILS body AA** on all surfaces; passes large-text AA only |
| `--color-accent` | `#f97316` | 7.10 | 6.88 | 6.62 | 6.18 | ✅ body |
| `--color-accent-hover` | `#ea580c` | 5.59 | 5.41 | 5.21 | 4.87 | ✅ body |
| `--color-info` | `#00e5ff` | 12.93 | 12.53 | 12.06 | 11.27 | ✅ body |
| `--color-success` | `#00ff88` | 14.84 | 14.37 | 13.83 | 12.93 | ✅ body |
| `--color-warning` | `#FFB020` | 10.88 | 10.54 | 10.14 | 9.48 | ✅ body |
| `--color-danger` | `#ff3366` | 5.61 | 5.43 | 5.23 | 4.88 | ✅ body |
| `--color-hold` | `#a78bfa` | 7.31 | 7.08 | 6.81 | 6.37 | ✅ body |
| `--color-status-hold` | `#8b7ec9` | 5.61 | 5.44 | 5.23 | 4.89 | ✅ body |
| `--color-status-mastering` | `#6b5eb0` | 3.65 | 3.54 | 3.40 | 3.18 | ⚠️ **FAILS body AA** on all surfaces; passes large/UI AA |
| `--color-status-day-of` | `#e8621f` | 5.87 | 5.69 | 5.47 | 5.12 | ✅ body |
| `--color-status-settled` | `#4a9b6e` | 5.88 | 5.69 | 5.48 | 5.12 | ✅ body |
| `--color-status-warning` | `#d49450` | 7.72 | 7.48 | 7.20 | 6.73 | ✅ body |
| `--color-role-show-side` | `#5db8c2` | 8.62 | 8.35 | 8.04 | 7.51 | ✅ body |
| `--color-role-commerce` | `#9176c5` | 5.31 | 5.14 | 4.95 | 4.62 | ✅ body |
| `--color-role-press` | `#c47aa0` | 6.29 | 6.09 | 5.86 | 5.48 | ✅ body |
| `--color-gear-stringed` | `#c87a4a` | 6.01 | 5.82 | 5.60 | 5.23 | ✅ body |
| `--color-gear-percussion` | `#9b7a3f` | 4.97 | 4.82 | 4.64 | 4.33 | ✅ on base/panel/raised; ⚠️ **4.33 on float** — fails body AA by 0.17 |
| `--color-gear-electronic` | `#6a8ac2` | 5.71 | 5.53 | 5.32 | 4.98 | ✅ body |
| `--color-chart-income` | `#3fa473` | 6.41 | 6.21 | 5.98 | 5.59 | ✅ body |
| `--color-chart-expense` | `#c4574f` | 4.57 | 4.43 | 4.26 | 3.98 | ⚠️ **fails body AA on raised/float** (4.26 / 3.98); passes base/panel |
| `--color-chart-neutral` | `#6e6e78` | 3.95 | 3.82 | 3.68 | 3.44 | ⚠️ **FAILS body AA** on all surfaces; passes large/UI AA — this is **by design** (orphan events render muted per D-7) |

### 1B findings summary

Four tokens fail the AA body-text threshold on at least one surface:

1. **`--color-fg-faint` (#6e6e79) — 3.95:1 on base.**
   - **Intentional.** Per the token name ("faint") this is reserved for tertiary labels and large metadata (e.g., `AgendaView.tsx:70` `font-mono text-[12px] uppercase tracking-widest text-fg-faint` — uppercase label with tracking qualifies as decorative / large-text). Still, any place this is used for running prose is a finding.
   - **Audit action (D-8c):** grep every `text-fg-faint` usage; flag any ≥ 13px regular weight running text.

2. **`--color-status-mastering` (#6b5eb0) — 3.65:1 on base.**
   - Used as status-chip accent for Release "Mastering" state. In `AgendaView.tsx:104-108` / `EventPopover.tsx:57-63` it renders as a chip with tinted background (`bg-status-mastering/10` equivalent via `withEventAlpha`) and text in the full token color.
   - **Risk:** if rendered as plain text (not chip), body AA fails on all surfaces. Source scan did not find a plain-text usage, but D-8c should confirm.

3. **`--color-chart-neutral` (#6e6e78) — 3.95:1 on base.**
   - **Intentional per D-7.** Orphan events (no `artist_id`) render muted so they read as background context. `calendar-colors.ts` marks these `isOrphanColor` → view components apply `opacity-70`, lowering effective contrast further. This is a deliberate trade-off, not a bug.
   - **Accessibility risk:** a user with low vision may miss orphan events entirely. D-8c should consider a non-color cue (e.g., dashed border, "◇" prefix) to supplement the low contrast.

4. **`--color-chart-expense` (#c4574f) — 4.26/3.98 on raised/float.**
   - Used in Money views and MiniMonth legend. Passes on `base` and `panel` (the primary chart surfaces) but fails on `raised` (card interiors) and `float` (modals). Check whether any expense figure renders directly on `bg-raised` or `bg-float` without a wrapper.
   - **Audit action (D-8c):** grep `text-chart-expense` (if tokenized) or `text-danger` in card/modal contexts.

5. **`--color-gear-percussion` (#9b7a3f) on `float` — 4.33:1.**
   - Off AA by 0.17. Marginal. Applies only when a percussion-category chip renders inside a modal.

### Aliases (not measured separately, resolve to existing tokens)

- `--color-role-advisory`, `--color-crew-logistics`, `--color-status-frozen` → all alias `--color-fg-muted` → **5.83:1 on base, all pass body AA**.
- `--color-crew-technical` → alias `--color-role-show-side` → passes everywhere.
- `--color-crew-commercial` → alias `--color-role-commerce` → passes everywhere.
- `--color-status-confirmed` → alias `--color-success` → passes everywhere.

---

## 1C — Non-color cue audit (source scan)

WCAG 1.4.1 Use of Color: color must not be the *sole* means of conveying information. Each finding below cites the file and line where a status is encoded.

### PASS — status shown with adjacent label

| Surface | Cue | Source |
|---|---|---|
| Shows list | Status dot + status text | `app/(dashboard)/shows/page.tsx` (list rows include status name next to dot) |
| Contacts list | Role dot + role name | `app/(dashboard)/contacts/page.tsx` |
| Crew list | Role dot + role name | `app/(dashboard)/crew/page.tsx` |
| Releases list | Status dot + status text | `app/(dashboard)/releases/page.tsx` |
| Tasks cards | Priority icon + priority label + color | `app/(dashboard)/tasks/_components/TaskCard.tsx:36-41` (icon carries priority, not color alone) |
| Cancelled shows | `line-through` text decoration | `app/(dashboard)/shows/page.tsx`, `app/(dashboard)/dashboard/page.tsx`, calendar `shared.tsx`, `TaskCard.tsx`, `PackingTab` |
| Calendar event type | Icon glyph per source + color | `AgendaView.tsx:124-138` — icon carries TYPE, color carries ARTIST |

### FAIL — status shown by color alone

| Finding | Location | Severity |
|---|---|---|
| Dashboard `StatusDot` has no adjacent label in some card headers | `app/(dashboard)/dashboard/page.tsx:150-160` | **HIGH** — dot color alone encodes status; aria-label not confirmed |
| Calendar event dots (MiniMonth) use first-event artist color without any icon/label | `MiniMonth.tsx:73-80` — the bottom dot is pure color, no glyph | **MEDIUM** — legend is implicit; design-system.md §3.7 accepts this on density grounds, but a colorblind user has no fallback |
| Notification bell button has a red dot but the **button itself** has no `aria-label` or text to describe it | `app/(dashboard)/layout.tsx:55-66` — dot has `aria-label="Unread notifications"` but button wrapping it lacks one | **HIGH** — SR users hear the dot without knowing the button opens notifications |

### D-8c recommendations (do NOT implement in D-8a)

- Add `aria-label` to the notification bell button itself.
- Confirm `StatusDot` callers include a visible status label, or add `aria-label` with the status string.
- Add an optional icon-first rendering for MiniMonth event dots (e.g., a diamond for orphans, a small glyph for the dominant event type).

---

## 1D — Chart shape encoding (visual audit)

**Status:** **REQUIRES LIVE BROWSER — see D-8b checklist below.**

Cannot be fully verified from source: chart axis labels, legend markers, and tooltip affordances need to be seen at real render sizes. Source scan found:

- Money views use Recharts bars + lines (import in `app/(dashboard)/money/_components/*.tsx`).
- Legend markers are colored squares **without secondary shape encoding** (diamond/triangle/etc.).
- Bar/line colors are drawn from `--color-chart-income` and `--color-chart-expense` only (2 series) — distinguishable without shape for most users but not for achromatopsia.

### D-8b checklist — Jacob, run this manually

For each chart on the site (Money — income/expense bars, dashboard — financial summary, release timeline if present), confirm:

- Legend markers have both color **and** a non-color cue (shape, pattern, label) ✅/❌
- Line series are distinguishable via line weight or dash pattern, not just color ✅/❌
- Tooltip on hover shows the series name in text (not just a colored swatch) ✅/❌
- Under achromatopsia simulation, can you still tell which series is which? ✅/❌

**Record at:** `design-audit/d8b-chart-shape-findings.md`.

---

## 1E — Alert icon audit (source)

WCAG 1.3.3 Sensory Characteristics: alerts should combine color with an icon or role so SR users and colorblind users both pick them up.

| Location | Icon present? | `role="alert"`? | `aria-label`? | Verdict |
|---|---|---|---|---|
| `ErrorFallback.tsx:38` (page variant) | ✅ `<AlertCircle />` | ✅ implicit via `role="alert"` wrapper | — | ✅ PASS |
| `ErrorFallback.tsx:52` (inline variant) | ✅ `<AlertCircle />` | ✅ | — | ✅ PASS |
| `ErrorFallback.tsx:66` (panel variant) | ✅ `<AlertCircle />` | ✅ | — | ✅ PASS |
| `app/_components/Toast.tsx` | ✅ variant-specific icon (success / danger / info) | ✅ | — | ✅ PASS |
| `app/(dashboard)/_components/UpgradeBanner.tsx` | ✅ sparkle/amber icon | ✅ `role="region"` + `aria-label` | ✅ | ✅ PASS |
| `app/(auth)/login/page.tsx:249` | ❌ plain `<p role="alert">` with red text | ✅ | — | ❌ **FAIL** — colorblind users get no alert cue |
| `app/(auth)/reset-password/page.tsx:81, 122` | ❌ plain `<p role="alert">` | ✅ | — | ❌ **FAIL** — same |

### 1E findings

2 FAILs, both in auth flows. Risk is moderate (form error text is self-describing — "Invalid password" etc.), but WCAG 1.3.3 asks for a non-color cue. D-8c should add a small `<AlertCircle size={14} />` prefix to the auth error `<p>` tags.

---

## 1F — Focus ring source audit

WCAG 2.4.7 Focus Visible. Every interactive element must have a visible focus indicator. Tailwind `focus-visible:` utilities or equivalent CSS required.

### PASS — focus-visible implemented

| Component | Location | Ring style |
|---|---|---|
| `Tabs` | `app/_components/Tabs.tsx:120` | `focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-accent focus-visible:ring-offset-2` |
| `CopyableValue` | `app/_components/CopyableValue.tsx:54` | `focus-visible:ring-2 focus-visible:ring-accent/60` |

### FAIL — no focus-visible styling

| Component | File:Line | Severity |
|---|---|---|
| Calendar prev button | `CalendarHeader.tsx:68` | **HIGH** — keyboard users cannot see focus on main calendar nav |
| Calendar next button | `CalendarHeader.tsx:72` | **HIGH** — same |
| Calendar Today button | `CalendarHeader.tsx:75` | **HIGH** — same |
| `UpgradeBanner` CTA button | `app/(dashboard)/_components/UpgradeBanner.tsx` | **MEDIUM** — marketing CTA, still keyboard-reachable |
| `UpgradeBanner` dismiss button | `app/(dashboard)/_components/UpgradeBanner.tsx` | **MEDIUM** |
| `ErrorFallback` Retry button | `app/_components/ErrorFallback.tsx` (all 3 variants) | **HIGH** — users hitting an error need a clear way to recover via keyboard |
| `ErrorFallback` Report button | same | **HIGH** — same |
| `Shows` view-toggle buttons | `app/(dashboard)/shows/_components/ViewToggle.tsx` (approx) | **HIGH** — uses `title=` attribute instead of `aria-label`; no focus ring; no screen-reader text |

### 1F findings

7+ components lack `focus-visible:` classes. This is the largest single-class of a11y finding in the codebase. The fix is mechanical: add the Tabs pattern (`focus-visible:outline-none focus-visible:ring-2 focus-visible:ring-accent focus-visible:ring-offset-2 focus-visible:ring-offset-base`) to each. Assign to D-8c remediation.

---

## 1G — Mobile touch target audit (44×44 px minimum)

**Status:** **REQUIRES LIVE BROWSER — see D-8b checklist below.**

Apple HIG and WCAG 2.5.5 recommend 44×44 px minimum tappable area. Source inspection can flag obviously-small targets (`h-6 w-6`, `h-7 w-7`) but cannot verify hit-slop or parent padding at render time.

### Source-flagged candidates (need in-browser measurement)

| Element | File:Line | Nominal class | Risk |
|---|---|---|---|
| Calendar nav buttons | `CalendarHeader.tsx:68,72` | `h-8 w-8` (32px) | ❌ below 44px; hit-slop undefined |
| EventPopover close button | `EventPopover.tsx:29-31` | `h-6 w-6` (24px) | ❌ well below |
| MiniMonth prev/next | `MiniMonth.tsx:31, 35` | `h-6 w-6` (24px) | ❌ well below |
| MiniMonth day cells | `MiniMonth.tsx:66` | `h-7 w-7` (28px) | ❌ below; but 7-column grid, impossible to make 44px on mobile — expected pattern |
| Notification bell | `layout.tsx:55-66` | `h-9 w-9` (36px) | ⚠️ below |

### D-8b checklist — Jacob, run this manually

For each route below, using Chrome DevTools device emulation (iPhone 14 Pro Max, then iPhone SE):

1. Tap each interactive element and note whether it feels responsive
2. For elements < 44px, check whether parent padding or `::before` hit-slop extends the real target
3. Record: element, nominal size, effective hit area, pass/fail

**Priority routes (top 5 by mobile usage — Jacob's call):**
- `/dashboard`
- `/calendar`
- `/shows/[id]`
- `/money` (tab switcher)
- `/settings`

**Record at:** `design-audit/d8b-mobile-touch-findings.md`.

---

## 1H — Toggle: disabled vs off states

**Status:** Not applicable to current codebase.

**Finding:** no dedicated toggle/switch component exists. `grep -r "Switch\|Toggle" apps/web/app/_components` returns no matches. Settings pages use stacked buttons or `<input type="checkbox">` in native form contexts. Both checkbox states have the browser default styling, which is accessibility-compliant by construction.

**When a Switch component is added** (tracked as a future component in design-system.md §4 roster), the audit should confirm:
- **Disabled:** `opacity-50 cursor-not-allowed`, `aria-disabled="true"`, `tabindex="-1"`.
- **Off:** full opacity, `aria-checked="false"`, keyboard-focusable.
- These must be visually distinguishable (e.g., disabled uses `fg-faint`, off uses `fg-muted`).

No action for D-8c.

---

## D-8a findings summary — count by severity

| Severity | Count | Quick list |
|---|---|---|
| **BLOCKER** | 0 | — |
| **HIGH** | ~11 | Notification bell aria-label; dashboard StatusDot label; 3× Calendar nav focus rings; 2× ErrorFallback focus rings; Shows view-toggle focus+aria; auth error icons (2) |
| **MEDIUM** | ~4 | MiniMonth event dots (no glyph fallback); UpgradeBanner focus rings (2); chart-expense contrast on raised/float |
| **LOW** | ~2 | `fg-faint` running-text audit; `gear-percussion` float contrast (4.33) |

### Scaffolded for D-8b (live-browser verification)

- 1A Colorblind (4 modes × ~16 routes)
- 1D Chart shape encoding
- 1G Mobile touch targets (5 priority routes, 2 device sizes)

---
