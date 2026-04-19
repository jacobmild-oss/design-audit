# ArtistHQ Design System Audit

**Date:** 18 April 2026 (v3.1 — consolidated)
**Scope:** 62 screenshots — 32 from v2 audit + 30 closing the gap (7 Settings tabs, 4 Show detail tabs, 3 Release detail tabs, 3 Gear detail tabs, 2 Merch, 4 Contacts/Crew/Docs, 2 Money, 5 mobile variants)
**Reference benchmarks:** Linear, Stripe Dashboard, Attio, Fantastical/Cron, Superhuman, Vercel, Notion, Artist Growth

This document supersedes v2 (18 April 2026) and v3 (18 April 2026). v3.1 amendments are integrated inline. This is the canonical reference for all Batch D / E / F execution.

---

## TL;DR

The v2 diagnosis holds. The color-discipline root cause explains roughly 80% of the new pages' problems the same way it explained 80% of v2's — **settings-links** is v2's **contact-detail** problem multiplied by ten; **crew-list** is v2's **contacts-list** problem; **money-income** status pills, **money-band-bank** unpaid-amber, **release-detail-tasks** and **release-detail-tracks** empty-state amber — all flow from the same exhausted accent color.

What v2 couldn't see:

1. **The form template is already converging.** merch-new, merch-detail, contacts-new share a clean pattern. Name it, ship it as `<EntityForm>`, and stop re-deciding field stacks page by page.
2. **The Generated Document UI on show-detail-files is unusually sophisticated.** It's a micro-interface with generated-vs-uploaded split, section-progress dots, share state, and regenerate/edit/download triplet. This is a differentiator. Canonicalize the pattern before reimplementing it inconsistently across Releases and Gear.
3. **Mobile right-panel loss is systemic, not show-specific.** v2 flagged it on `show-detail-active-overview-mobile`. Confirmed on `release-detail-overview-mobile` (drops Rollout Timeline — arguably the most important meta on the page) and `contact-detail-mobile`. This is now 3.13-extended.
4. **money-income ships in two currencies on the same page.** Hero in USD (`$20,450 / $15,010 / $35,460`), rows in EUR (`€3,000`, `€1,500`). This is a data/presentation bug with design consequences — new issue 3.16.
5. **Settings' 9-tab row is an IA problem v2 under-weighted.** It wraps to 2 lines on every Settings surface shot. New issue 3.17.

Batch D grows — D-1 (color sweep) now touches 30% more surfaces. D-4 (role families) must include crew-list. A new **D-9 (hyperlink cleanup)** splits out of D-1 because its scope is large enough to track separately. **E-3 (right-panel extension)** now also covers crew-detail. A new **E-8 (Files page IA reconciliation)** appears. Batch F adds **F-10 (EntityForm component) — was invisible in v2.**

Estimates grow accordingly. Don't fake them.

---

# 0. Meta-Principles

These govern every rule that follows. When any specific principle, token, or fix below appears to conflict with something, §0 wins.

## 0.1 — Amplifiers, not conveyors

**Every design token should fail gracefully. Strip color → product still works. Strip motion → product still works. Strip typography hierarchy → layout still reads. This is the difference between craft and decoration.**

**Color, motion, and typography are amplifiers, not conveyors. They strengthen signal that is already present in structure and text — they do not replace it.**

Operational consequence: when implementing Batch D through F, any time a decision comes down to "should this state be communicated by color alone?" the answer is always no. Every status dot pairs with a text label. Every chart line gets a shape or pattern marker, not just hue. Every alert carries an icon. Every colored number pairs with a directional indicator (arrow, sign, label). This rule is not negotiable because violating it produces interfaces that fail for the 6% of men with red-green colorblindness, for anyone reading on a bad monitor, and for anyone briefly glancing at the screen.

This is also the meta-rule for tokens: any token that becomes the only channel carrying meaning is a failure of system design, regardless of whether it's implemented correctly.

## 0.2 — Prefer reversible decisions

**When a design call is hard to undo (deleting code paths, committing to a color system that cascades through schema, renaming database columns), make the conservative version now and the aggressive version later when you have data.**

**When a call is easy to undo (hiding a nav item, changing a hex, swapping a label), ship the aggressive version and revert if wrong.**

Hiding Week view from the tab row while keeping `?view=week` reachable is the reversible version. Deleting the code path is the irreversible version. Default to the first and gather data before doing the second.

This principle applies throughout the fix plan. Calls it out explicitly next to every batch item that sits on the reversible/irreversible border.

## 0.3 — Mobile parity is structural, not stylistic *(new in v3)*

**If a desktop surface carries information in a right-panel or secondary column, the mobile surface MUST carry that information somewhere. It may move (to a bottom drawer, to a summary strip above the tabs, to a dedicated Summary tab), but it may not disappear.**

Right-panel content is not "supplementary." On `show-detail-active-overview.png` the right panel holds the date display, payment status, and travel block — the structural meta of "what is this show?" On `release-detail-overview.png` the right panel holds the Rollout Timeline — the structural meta of "where is this release in its lifecycle?" Stripping these on mobile doesn't make the page cleaner; it makes the page a different page.

This is §0 territory because it's a rule that governs *every* principle below. Any fix in §1–§5 that would naturally produce a mobile variant must be inspected for parity.

---

# 1. Design Principles

## 1.1 Color discipline

The current system: amber = "anything important." That's not a system.

### The foundational rule

**Color is never load-bearing alone** (see §0.1). Every status, category, or emphasis communicated by color must also be communicated by text label, icon, position, or shape. Strip all color from the product — every piece of state must still be readable.

### Amber usage

**Amber (`--color-accent`, #f97316) appears on exactly these surfaces. Nowhere else. No exceptions.**

1. The single primary CTA per view (one button only: `+ Add Show`, `+ New Task`, `Save Changes`, `Upgrade to Plus`)
2. The active item in a navigation context (active sidebar row, active pill tab, active field border)
3. The input focus ring when a field is actively being edited
4. The favorite/star toggle in its "on" state
5. The primary brand wordmark on the landing page
6. **Toggle switches in the enabled/on state** *(new in v3 — named explicitly because `settings-notifications.png` has 17 enabled toggles in one view and the amber density challenged the rule. The rule holds: toggles-on are item 2 "active state." Naming this explicitly prevents future drift.)*

**Amber does NOT appear on:**

- Hyperlinks (email/phone on `contact-detail.png`, `crew-detail.png` — currently amber, should be `--color-fg` with subtle underline or `--color-info` if a link action)
- Link VALUES displayed in a form (Spotify URL, Instagram handle, website on `settings-links.png` — currently amber, should be `--color-fg` or `--color-info`)
- Status indicators (Day Of gets its own hue `#e8621f`, see below — do not let it drift back to `--color-accent`)
- Progress bars (release card progress on `releases-list.png`)
- Category dots (gear categories, contact roles, crew roles)
- Type badges ("EP" on releases, "Apparel" on merch — these are neutral meta, use `--color-fg-muted` chip)
- Financial figures (Artist Payout on `show-detail-settled-settlement.png` — should be white; Unpaid stat on `money-band-bank.png` — should migrate to `--color-status-warning`)
- **Marker chips for "default" / "primary" relationships** *(new in v3 — `settings-band.png` shows a filled-amber DEFAULT chip on the Equal Split rule; this is a marker, not a CTA or active state. Rule: marker chips use `--color-chip-default` neutral treatment with uppercase mono label.)*
- Hover underlines
- Icons
- Secondary CTAs (Export, Duplicate, etc. — these go neutral border)
- Action words inside tables ("Bring 6", "Add links →")
- Expand/collapse chevrons
- "in 30d" / "in 45d" countdown text
- Release status chips ("Mixing", "Mastering" — these are status, use release-status palette; see below)

### Status colors

Introduce a named status palette. Each color has one job.

| State | Color | Token |
|---|---|---|
| Hold / tentative | violet | `--color-status-hold` |
| Confirmed | green | `--color-success` |
| Day Of / active-now | `#e8621f` (distinct from accent) | `--color-status-day-of` |
| Settled / complete | muted green | `--color-status-settled` |
| Cancelled | neutral fg-muted strike | — |
| Overdue / warning | warm orange, not accent | `--color-status-warning` |
| Frozen / read-only | muted | `--color-fg-muted` |
| Unpaid (finance-specific) | `--color-status-warning` | — |
| Paid | `--color-success` | — |
| Partial | muted amber (40% opacity) | `--color-status-partial` *(new)* |

**Why Day Of has its own hue:** The v1 of this audit proposed `--color-status-day-of: var(--color-accent)` with an argued exception. That was fragile. A distinct `#e8621f` (warmer, red-shifted) preserves the "urgent warm hue" feel while removing any chance of Day Of competing with a CTA. The amber rule becomes absolute: **amber = CTA + active state + toggle-on + wordmark + favorite-on only, no exceptions.** Ship `#e8621f` as a flat color change; if the orange-on-orange adjacency feels off in real use, F-6 introduces a motion treatment as refinement.

### Release status colors *(new in v3)*

Release lifecycle is distinct from show lifecycle and needs its own palette subset:

| State | Color | Token |
|---|---|---|
| Writing / Demo | muted | `--color-fg-muted` |
| Recording | cyan (action state) | `--color-role-show-side` |
| Mixing | violet | `--color-status-hold` |
| Mastering | deeper violet | `--color-status-mastering` *(new)* |
| Released | green | `--color-success` |
| Archived | muted-strike | — |

`release-detail-overview.png`, `release-detail-overview-mobile.png`, and `release-detail-checklist.png` currently put "Mixing" in amber (`--color-accent`). Migrate to `--color-status-hold` violet. Mastering is close enough to Mixing in the process that it deserves a distinguishable but related hue — a deeper violet rather than drifting back to amber.

### Role / category colors (contacts, gear, expenses, **crew**)

Current state: hues assigned at random. Booker orange, Agent cyan, Promoter green, Venue Contact purple, Press/PR pink, Lawyer gray, Accountant gold, Label orange, Distributor yellow. Nine hues carrying no semantic relationship.

**Rule:** group roles into 3–4 semantic families, assign one color per family:

- **Show side** (Promoter, Venue Contact, Booker, Agent): cyan family
- **Commerce side** (Label, Distributor, Publisher): violet family
- **Advisory** (Lawyer, Accountant, Manager): neutral/gray
- **Press** (Press/PR, PR agencies): pink

**Crew roles (new in v3):** `crew-list.png` reveals 4 roles with 4 arbitrary hues — Sound Engineer cyan, Lighting Designer amber (!), Driver/Tech purple, Merch/Tour Aux muted. Amber on Lighting Designer is a Tier-1 violation — amber is a CTA color, never a category. Compress to:

- **Technical** (Sound Engineer, Lighting Designer, Backline Tech): cyan family
- **Logistics** (Driver, Tour Manager, Tour Aux): neutral/gray
- **Commercial** (Merch, Ticketing): violet family (if it needs one)

Same approach for gear categories. Instead of 6 arbitrary hues:

- **Stringed** (Guitars, Basses): one hue
- **Percussion** (Drums, Perc): one hue
- **Electronic** (Amps, Keys, Effects): one hue

This compresses the palette from chaos to structure. Family color appears as a dot; the specific role/category is always named in adjacent text (§0.1).

### Calendar color (D-7 architecture commitment)

**Calendar events are colored by source (artist), not by type.** Type is encoded by icon. See D-7 for full specification. This affects schema: `events.artist_id` must be queryable and `artists.brand_color` is the canonical source of truth.

Orphan events (system-generated, imported, artist-less): `--color-chart-neutral` muted gray.

## 1.2 Typography scale

Current: everything lives between 12px and 28px with minor weight variation. Flat.

### Proposed scale

| Token | Size | Weight | Font | Role |
|---|---|---|---|---|
| `--text-hero` | 48px | 700 | Syne | Landing H1, login wordmark |
| `--text-display` | 32px | 700 | Syne | Page-level entity titles ("The Lexington", "Nerve Drift", "Settings") |
| `--text-section` | 20px | 600 | System sans | Card titles ("Show Pipeline", "Quick Stats", "Expense Breakdown") |
| `--text-body` | 15px | 500 | System sans | Primary data values (venue names, amounts, names) |
| `--text-meta` | 13px | 400 | System sans | Secondary info (cities, dates in context, sub-lines) |
| `--text-label` | 11px | 500 | JetBrains Mono | All-caps labels (`DEAL`, `SCHEDULE`, `PRODUCTION`, `CATEGORY`, `VARIANTS`) |
| `--text-mono-data` | 13px | 400 | JetBrains Mono | Times, dates in tables, IDs, serial numbers |
| `--text-stat-hero` | 40px | 700 | Syne | Big display numbers ("20" in right panel) |
| `--text-stat` | 24px | 600 | System sans | Dashboard stat values (€11,700) |

Key moves:

- **Jump from 15px body straight to 24–32px display, skipping 18/20 for values.** This is what makes Linear and Stripe feel crafted.
- **Labels in JetBrains Mono at 11px with letter-spacing 0.05em.** You already do this selectively — make it consistent. Form labels on `merch-new.png`, `contacts-new.png`, `settings-preferences.png`, `settings-links.png` are already close — lock the treatment.
- **Stat values in bespoke weight (Syne or semibold system) at 24–40px.** Currently Revenue €11,700 is 28px, Net +€4,765 is 28px, but the hero "NEXT SHOW 03 May" is only 22–24px. Hero stat gets the biggest treatment.
- **Syne is the display font. Don't use it below 20px.** It's a display face, not a UI face. The Syne "ARTISTHQ" wordmark on login at ~24px is on the edge — acceptable for brand, but don't extend Syne into UI.

**§0.1 reminder:** hierarchy must survive type scale being stripped. If the only thing distinguishing a section title from body text is size, the structure fails when a screen-reader reads it, when reflow shrinks everything, or when a user overrides base font size. Pair size with weight shift, mono-vs-sans shift, or semantic HTML (`<h2>` vs `<p>`).

## 1.3 Spacing rhythm

Adopt a 4px base unit. Allowed multiples: 4, 8, 12, 16, 24, 32, 48, 64.

**Rules:**

- Inside cards: 20–24px padding
- Between cards in a grid: 16px (currently 12–16px varies)
- Between major sections on a page: 32px
- Between a label and its value (inside a card): 4px
- Between paired fields in a row: 16px
- Page outer margin: 32px desktop, 16px mobile

Places where this breaks today:

- **Dashboard**: stat cards are ~12px apart, should be 16px. Bento grid feels cramped.
- **Show detail overview**: tab row → first card has ~24px, acceptable; but between Location card and Tasks/Expenses row feels tighter than between Tasks/Expenses and Documents.
- **Settlement pages**: good spacing — use as the reference.
- **Settings pages** *(new in v3)*: form cards have 32px inner padding — comfortable. However, the gap between card and the "Save Changes" button at top-right is inconsistent relative to the same gap on `settings-notifications.png`. Standardize.
- **Money-band-bank** *(new in v3)*: payout row inside the card uses ~40px vertical padding — too roomy for a single-row payout entry. Tighten to 20px.

## 1.4 Interaction patterns

### Focus rings
- Input focus: 2px `--color-accent` ring — currently correct.
- Button focus: 2px `--color-accent` ring with 2px offset — verify.
- Card focus (for keyboard nav through lists): 1px `--color-accent` border inside — new token.

### Hover states
- Row hover in tables: background shifts from `--color-panel` to a subtle lift, not a color change. Currently unclear what the shows-list row hover does.
- Button hover: brightness shift, not color shift. Amber buttons go slightly lighter on hover, not to a different hue.
- Link hover: underline appears. Don't change color.

### Affordance signaling
- Primary CTA: filled amber.
- Secondary CTA: outlined neutral (`--color-edge` border, `--color-fg` text).
- Tertiary action: text-only with underline on hover.
- Destructive: outlined `--color-danger` for non-critical ("Delete contact"), filled `--color-danger` only for confirmations.

Currently the "Export" button in top-right on Money pages looks nearly identical to neutral UI chrome — needs a subtle border to read as interactive. **Same observation on `show-detail-active-day-sheet.png`** for "Copy to Clipboard" and "Print" — both read as disabled text.

### Form template *(new in v3)*

`merch-new.png`, `merch-detail.png`, `contacts-new.png`, `shows-new.png`, and `releases-new.png` are converging on a consistent pattern. Name it, lock it, ship it as `<EntityForm>`.

**The pattern:**

```
[< Back link]
<Page H1: "Add X" | "Edit X">
<Subtitle: single-sentence purpose>

  ┌─ Card wrapper ────────────────────────────────┐
  │ LABEL *                                        │
  │ [input, focused on mount if creating]          │
  │                                                │
  │ LABEL                                          │
  │ [input]                                        │
  │                                                │
  │ [SECTION BREAK: 11px mono label, divider]      │
  │                                                │
  │ LABEL          LABEL                           │
  │ [input]        [input]                         │
  │                                                │
  │ NOTES                                          │
  │ [textarea, 4 rows]                             │
  └────────────────────────────────────────────────┘

[Primary CTA: amber, full-width, 48px height]
[Destructive link: "Delete X" — only on edit mode, text link, below CTA]
```

**Rules:**
- Single card wrapper. Do not split fields across multiple cards unless a semantic section boundary warrants it (e.g., VARIANTS table on merch is legitimately a second card).
- Section breaks inside the card use 11px mono uppercase labels + 1px divider. They do NOT use the `--text-section` 20px Syne treatment — that's for page-level section headers, not form sub-sections.
- Required fields marked with `*` inline after the label.
- Status chip (e.g., "Active" on `merch-detail.png`) goes top-right of H1 row on edit mode only.
- Entity pager (`2/4`, `<`, `>`, `EDX-###`) top-right on edit mode only.

What this consolidates: `shows-new.png` and `releases-new.png` (per v2) have "sidebar leaking" issues from the full-page form layout. `merch-new.png` and `contacts-new.png` fix this by keeping the card wrapper self-contained. Use the latter as the reference. Fold into F-10 as a component-ization pass.

## 1.5 Information hierarchy

Three levels, always:

1. **The primary thing.** The venue name, the release title, the settlement amount. Biggest, whitest, most weight.
2. **The structural meta.** Date, city, status. Smaller, still white or high-contrast.
3. **The supporting detail.** Labels, secondary context, IDs. Mono, muted.

On `show-detail-active-overview.png` this works: "The Lexington" 32px white, "London, GB" 16px muted, €900 smaller — but then "Day Of Monday, 20 April 2026" lives ABOVE the venue name in a weird weight that doesn't fit either tier. That meta strip should be smaller and subordinate, not competing.

**Mobile parity amendment *(new in v3, governed by §0.3):*** When the desktop layout uses a right panel to hold level-2 meta (e.g., Rollout Timeline on releases, Day/Date/Amount on shows, Condition/Value/Warranty on gear), the mobile variant must carry equivalent information via one of three patterns:

1. **Summary strip above tabs** — compact horizontal band with the 3–5 most important meta values. Use on show-detail where date-at-a-glance matters.
2. **Collapsible Summary section** — a dedicated first card (or expandable drawer at page bottom) containing the right-panel equivalents. Use on release-detail where Rollout Timeline is complex.
3. **Summary tab** — explicit tab that holds the right-panel equivalent. Use when right-panel content is extensive enough to warrant its own surface (gear-detail's 4 stacked cards).

`release-detail-overview-mobile.png` currently drops Rollout Timeline, Contact, and Quick Stats entirely — this is the most severe mobile-parity violation in the product because Rollout Timeline is the page's primary navigational spine.

## 1.6 Generated document micro-interface *(new in v3)*

`show-detail-active-files.png` exposes a pattern that doesn't exist anywhere else in the product and isn't documented in v2. It's a differentiator and deserves canonicalization before it gets reimplemented inconsistently on Releases, Gear, and Contacts.

**The pattern — "Generatable Document Card":**

Each card represents a document that the system can generate from structured data (advancing sheet → Technical Rider, day sheet data → Day Sheet PDF, settlement data → Settlement Sheet). The card has four possible states, each with distinct affordances:

| State | Visual | Actions |
|---|---|---|
| Not generated | Neutral card, dotted progress ring at N% completeness of input data, icon | `Generate` (primary amber if completeness ≥70%, neutral if <70%) |
| Generated, not shared | Full card, solid progress ring | `Re-generate`, `Edit`, `Download` |
| Generated and shared | Full card + green "Shared" badge with dot | `Re-generate`, `Edit`, `Download`, share icon |
| Generated but stale (source data changed since generation) | Card with warning amber dot, "Re-generate recommended" microcopy | `Re-generate` (primary), `Edit`, `Download` |

**Constituent tokens:**
- `--doc-card-bg`: neutral panel
- `--doc-progress-complete`: green dot sequence
- `--doc-progress-partial`: muted dot sequence
- `--doc-share-badge-bg`: green-tinted with solid dot
- `--doc-stale-warning`: warning-amber microcopy

**Where this pattern must be applied consistently:**
- Show detail → Files tab (✓ already correct — this is the reference)
- Release detail → Files tab (currently a dumb upload-only page — promote to this pattern for EPK, Press Kit, Liner Notes)
- Gear detail → Files tab (currently upload-only — acceptable because gear docs aren't system-generated)
- Contact detail → Files tab (upload-only — acceptable)

**Why this matters now:** Without canonicalization, each page's Files tab will drift as you build. The show-detail implementation is strong; don't let it become an inconsistent island. Fold into a new **F-11** component and ship alongside the F-5 generated-docs brand color work.

---

# 2. Design Tokens (Updated + New)

## 2.1 Tokens to change

| Token | Current | Proposed | Why |
|---|---|---|---|
| `--color-accent` usage | ubiquitous | CTAs + active states + toggles-on + favorite-on + wordmark only | Restore exclusivity |
| `--color-chart-income` | #3fa473 (muted green) | keep | Already right |
| `--color-chart-expense` | #c4574f (muted brick) | keep | Already right |
| `--color-info` | present but unclear role | use for hyperlinks and neutral-informational highlights | Take over amber's link duty across `settings-links`, `crew-detail`, `contact-detail` |

## 2.2 New tokens

```css
/* Status palette */
--color-status-hold: #8b7ec9;              /* violet, hold/tentative, release "Mixing" */
--color-status-confirmed: var(--color-success);
--color-status-day-of: #e8621f;
/* Rendered as text color on dim chip backgrounds
   (bg-status-day-of-bg = rgba(232, 98, 31, 0.12)).
   Contrast vs --color-bg (#09090b): 5.99:1 — passes WCAG AA normal text.
   NOT rendered as solid fill with white text — that pattern yields
   only 3.17:1, which fails AA normal text. Solid-fill chip rendering
   must use --color-fg-muted or a different token.
   Verified: D-3 commit ff5af95. */
--color-status-settled: #4a9b6e;           /* deeper muted green for frozen/complete */
--color-status-warning: #d49450;           /* distinct from accent amber — unpaid, at-limit */
--color-status-frozen: var(--color-fg-muted);
--color-status-mastering: #6b5eb0;         /* NEW v3 — deeper violet for release mastering */
--color-status-partial: rgba(249, 115, 22, 0.4);  /* NEW v3 — partial payment, 40% amber */

/* Role families (contacts, gear categories, crew) */
--color-role-show-side: #5db8c2;           /* cyan — promoter, venue, booker, agent */
--color-role-commerce: #9176c5;            /* violet — label, distributor, publisher */
--color-role-advisory: var(--color-fg-muted);   /* lawyer, accountant, manager */
--color-role-press: #c47aa0;               /* muted pink */

/* Crew role families (NEW v3) */
--color-crew-technical: var(--color-role-show-side);   /* sound, lighting, backline */
--color-crew-logistics: var(--color-fg-muted);         /* driver, TM, tour aux */
--color-crew-commercial: var(--color-role-commerce);   /* merch, ticketing */

/* Gear category families */
--color-gear-stringed: #c87a4a;            /* warm earth */
--color-gear-percussion: #9b7a3f;          /* darker warm */
--color-gear-electronic: #6a8ac2;          /* cool */

/* Chip / marker variants (NEW v3) */
--color-chip-default-bg: var(--color-panel);
--color-chip-default-fg: var(--color-fg-muted);
--color-chip-default-border: var(--color-edge);
/* Used for DEFAULT, PRIMARY, SYSTEM markers — never amber */

/* Calendar / chart neutrals */
--color-chart-neutral: #6b7280;            /* orphan events, unowned system events */

/* Typography */
--text-hero: 48px;
--text-display: 32px;
--text-section: 20px;
--text-body: 15px;
--text-meta: 13px;
--text-label: 11px;
--text-mono-data: 13px;
--text-stat-hero: 40px;
--text-stat: 24px;

/* Spacing */
--space-1: 4px;
--space-2: 8px;
--space-3: 12px;
--space-4: 16px;
--space-6: 24px;
--space-8: 32px;
--space-12: 48px;
--space-16: 64px;

/* Right-panel pattern */
--panel-right-width: 320px;
--panel-right-gap: 24px;

/* Mobile summary pattern (NEW v3, supports §0.3) */
--panel-mobile-summary-bg: var(--color-panel);
--panel-mobile-summary-border: var(--color-edge);
--panel-mobile-summary-padding: 16px;
--panel-mobile-summary-sticky-offset: 56px;  /* accounts for top app bar */

/* Banner variants */
--banner-info-bg: rgba(var(--color-info-rgb), 0.08);
--banner-info-border: rgba(var(--color-info-rgb), 0.3);
--banner-warning-bg: rgba(var(--color-status-warning-rgb), 0.08);
--banner-warning-border: rgba(var(--color-status-warning-rgb), 0.3);
--banner-limit-bg: rgba(var(--color-danger-rgb), 0.06);   /* softer than current */
--banner-limit-border: rgba(var(--color-danger-rgb), 0.25);

/* Generated Document micro-interface (NEW v3, supports §1.6) */
--doc-card-bg: var(--color-panel);
--doc-card-border: var(--color-edge);
--doc-progress-complete-dot: var(--color-success);
--doc-progress-partial-dot: var(--color-fg-muted);
--doc-share-badge-bg: rgba(74, 155, 110, 0.15);
--doc-share-badge-fg: var(--color-success);
--doc-stale-warning-fg: var(--color-status-warning);

/* Motion (tokens defined now, applied in F-9) */
--duration-fast: 120ms;        /* hover, focus, small state changes */
--duration-base: 200ms;        /* panel transitions, modal enter */
--duration-slow: 320ms;        /* route transitions, large layout shifts */
--ease-out: cubic-bezier(0.2, 0.8, 0.2, 1);
--ease-in-out: cubic-bezier(0.4, 0, 0.2, 1);
--ease-emphasized: cubic-bezier(0.2, 0, 0, 1);   /* hero moments, onboarding */
```

All motion tokens must respect `prefers-reduced-motion: reduce` globally.

**Contrast discipline (v3.1):** Same notation discipline as `--color-status-day-of` applies to `--color-status-hold`, `--color-status-mastering`, `--color-status-warning`, and the `--color-role-*` / `--color-crew-*` family. D-8 measures and records actual ratios for each. Tokens that fail get revised before D-8 closes.

## 2.3 Gaps

1. **No "muted accent" token.** Currently for progress bars you're using full amber — should have a subdued variant at 40% opacity. `--color-status-partial` above partially addresses this.
2. **No "read-only" surface token.** Settled pages should visually differ from active pages. Currently `FROZEN` badges indicate this but the surface itself looks identical.
3. **No elevation/depth scale.** Only `--color-panel` and `--color-float`. You need a subtle hierarchy for right-panels vs. body cards vs. modals.
4. **No canonical `--color-link` token *(new v3)*.** Currently `--color-info` is proposed for hyperlink duty. Decide: does `--color-info` cover banner backgrounds AND hyperlinks, or do you split `--color-link` out? Recommendation: split. Hyperlinks need their own identity because they appear in dense text flows where the informational-banner color would read as decoration.
5. **No explicit currency-format token *(new v3)*.** `money-income.png` ships in USD and EUR simultaneously. Not a color token problem, but a "presentation contract" gap — where does `locale.currency` live and why does the hero ignore it? Tech debt disguised as design inconsistency.

---

# 3. Cross-Cutting Issues

## Issue 3.1 — Amber is overextended across every surface
**What's wrong:** Accent color carries six meanings and resolves to none. Links, progress, status, chevrons, countdowns, headers, primary CTAs all amber.

**Where it appears:**
- `contact-detail.png` — email + phone fully amber (hyperlink abuse)
- `contact-detail-mobile.png` *(new v3)* — same abuse, mobile
- `crew-detail.png` *(new v3)* — email + phone amber, same abuse; "Role: Sound Engineer" chip uses amber family color (cyan is correct)
- `settings-links.png` *(new v3)* — **ten amber link values on a single page**: website, Spotify URL, Apple Music URL, YouTube URL, Bandcamp URL, Instagram handle, TikTok handle, Twitter handle, Patreon URL, etc. This is the worst offender in the product.
- `release-detail-overview.png` — "Add links →", "NOTES" header, Contact name "James Murphy" all amber
- `release-detail-overview-mobile.png` *(new v3)* — "Mixing" status chip amber (should be `--color-status-hold`)
- `release-detail-checklist.png`, `release-detail-tasks.png`, `release-detail-tracks.png` *(new v3)* — "Mixing" chip amber on each
- `release-detail-tracks.png` *(new v3)* — "Add your first track" amber link in empty state (should be neutral)
- `release-detail-checklist.png` *(new v3)* — Contact block shows "James Murphy" name in amber
- `releases-list.png` — progress bars under release cards all amber
- `show-detail-settled-settlement.png` — "Artist Payout €1,200" amber (should be primary white)
- `show-detail-active-overview.png` — "Day Of" status chip competes with Star favorite button competes with "Share day sheet with crew" CTA
- `show-detail-active-day-sheet.png` *(new v3)* — timeline bullets all amber (should be neutral dot or muted-amber only if semantic "scheduled" state)
- `show-detail-active-guest-list.png` *(new v3)* — "VIP" pills amber (should be neutral chip with VIP label)
- `show-detail-active-setlist.png` *(new v3)* — "Add First Song" amber link in empty state
- `show-detail-active-files.png` *(new v3)* — "Setlist / Settlement Sheet / Generate" amber buttons are appropriate (primary CTA on not-generated cards); "Upload File" amber is appropriate; "Re-generate" button color inconsistent across cards
- `dashboard.png` — "All shows →", "All tasks →", "View →", "All files →", "View all →", "View →" all amber
- `gear-list.png` — category dots amber + cyan + purple + blue + yellow
- `gear-detail-packing.png`, `gear-detail-files.png` *(new v3)* — empty-state "Go to Packing Lists" and "link existing" amber links
- `crew-list.png` *(new v3)* — Lighting Designer role dot is **amber** — Tier-1 violation (§1.1 rule: amber is never a category color)
- `calendar-agenda.png` — status pills RELEASED, SETTLED, CONFIRMED, HOLD, MASTERING, MIXING all mixed accent + status
- `calendar-agenda-mobile.png` *(new v3)* — same issues, mobile
- `money-income.png` *(new v3)* — "Partial" payment status chip amber; "+ Log Income" CTA amber (appropriate); "Other Income" section uses amber for action link
- `money-band-bank.png` *(new v3)* — "Unpaid €10,398" stat in amber (should be `--color-status-warning` distinct orange)
- `money-overview-mobile.png` *(new v3)* — Pending stat amber (same as above)
- `settings-band.png` *(new v3)* — DEFAULT chip filled amber (should be neutral `--color-chip-default`); "Add rule" button text amber
- `settings-team.png` *(new v3)* — ADMIN chip amber; "Invite" button amber text
- `settings-notifications.png` *(new v3)* — 17 enabled toggles amber. **Legitimate under §1.1 item 6 carveout** — but visually dense. Acceptable; named rule makes future drift unlikely.
- `settings-account.png` *(new v3)* — "Delete Pale Wire" button is danger-outlined (correct); but Sign Out neutral-bordered button adjacent is ambiguous

**Fix:** Apply §1.1 color discipline rules globally. Day Of migrates to `#e8621f`. Mixing migrates to `--color-status-hold`. "DEFAULT" chip migrates to `--color-chip-default`. Hyperlinks migrate to `--color-info` (or new `--color-link`). This is Batch D-1 / D-3 / D-4 / **new D-9**.

**Ladders to:** Color discipline, §0.1

## Issue 3.2 — Financial values rendered in color; primary numbers should be white
**What's wrong:** Stripe Dashboard displays ALL primary financial values in white. Color only appears on deltas (+/-), status changes, or alert thresholds. ArtistHQ colors everything green or red by polarity of the number.

**Where it appears:**
- `dashboard.png` — Revenue €11,700 green, Expenses €6,935 red, Net +€4,765 green, Merch "THIS MONTH €948" green
- `money-overview.png` — hero row all four stats colored (green/red/green/orange); Crew Costs €10,220 in violet (?)
- `money-overview-mobile.png` *(new v3)* — same: Total Income green, Total Expenses red, Net Profit green, Pending amber, Crew Costs cyan
- `money-reports.png` — Fees green, Costs red, Net green per tour row; Income/Expenses/Net at Annual Summary — all colored
- `money-income.png` *(new v3)* — Show Income $20,450 and Other Income $15,010 both green; "CONFIRMED €11,700" subtotal green
- `money-band-bank.png` *(new v3)* — Earned white (correct), Paid green, Unpaid amber — mixed policy
- `show-detail-settled-settlement.png` — Artist Payout €1,200 in amber; balance due in white — inconsistent even within the same page
- `shows-list-upcoming.png` — "€12,600 confirmed" in green in header
- `shows-list-upcoming-mobile.png` *(new v3)* — same green

**Fix:**
- Primary financial values: `--color-fg` (white).
- Deltas only (month-over-month arrows, "+23%", "-102%"): colored green/red, always paired with ▲/▼ arrow per §0.1.
- Status indicators: colored status pill *next to* white number, not the number itself.

Example correct pattern (money-overview hero):
```
TOTAL INCOME         TOTAL EXPENSES         NET PROFIT         PENDING
€35,460.00           €16,124.00             €19,336.00         €8,500.00
13 shows + 10 other  19 entries             ▲ 55% margin       3 shows on hold
```
Only the "▲ 55%" delta would be green. Everything else white.

**Ladders to:** Color discipline, §0.1

## Issue 3.3 — Typography hierarchy is flat
**What's wrong:** Most text lives in the 12–16px band with weight variation. Nothing reads as dramatically more important than its neighbors.

**Where it appears:** Everywhere, but especially:
- `show-detail-active-overview.png` — venue name is good at 32px, but the tabs row + Day Of strip + city + €900 all read as roughly the same visual tier
- `money-reports.png` — Annual Summary heading is the same size as stat labels
- `settings-profile.png` — "Settings" Syne display is good; everything else is 14–15px flat
- `settings-preferences.png`, `settings-account.png`, `settings-team.png`, `settings-export.png` *(new v3)* — all confirm the same flatness. The "Settings" Syne H1 is strong; the card H2s ("Account", "Team", "Preferences") inside feel indistinguishable from body.
- `crew-detail.png` *(new v3)* — Oskar Holm 28px (close to display), "€350/day · oskar@soundcraft.se" 13px — no section header treatment between field groups
- `release-detail-tasks.png` *(new v3)* — "Approve Nerve Drift EP artwork" 15px body weight — this is the primary thing on the row and deserves more weight

**Fix:** Apply typography scale from §1.2.

**Ladders to:** Information hierarchy

## Issue 3.4 — Filter pills and navigation pills are visually identical
**What's wrong:** Tabs-within-view and filters-within-view use the same shape (pill, amber active state). Eye can't distinguish navigation from filtering.

**Where it appears:**
- `money-overview.png`, `money-expenses.png`, `money-income.png` *(new v3)*, `money-reports.png`, `money-budgets.png`, `money-band-bank.png` *(new v3)* — [Overview/Expenses/Income/Budgets/Reports/Band Bank] pills above [This Month/This Quarter/This Year/Last Year/All Time/Custom Range] pills
- `shows-list-upcoming.png` / `shows-list-upcoming-mobile.png` *(new v3)* — [Upcoming/Past] pills then below [All/Hold/Confirmed/Day Of] pills then below [TOUR: All/Nerve Drift Summer/No tour] pills = three pill rows
- `releases-list.png` — [Upcoming/Past] + [Grid/Pipeline] + [All Formats/Single/EP/Album] + [All/Mixing/Mastering/Released] = four pill rows
- `crew-list.png` *(new v3)* — [All / • Sound Engineer (1)] filter chips use same pill shape as top-level tabs elsewhere
- `calendar-agenda-mobile.png` *(new v3)* — [Month/Week/Agenda] nav pills then [Shows/Releases/Tours/Tasks/Financials] filter pills = same shape

**Fix:** Apply the existing `<Tabs>` pill variant ONLY to first-level navigation. For filters, use:
- Segmented control (grouped chips with connected borders) for mutually-exclusive filters (time range, type)
- Plain chip (rounded rect, no fill when inactive, subtle bg when active) for additive filters (category chips)
- Dropdown for filters with >5 options

This is the Attio pattern.

**Ladders to:** Interaction pattern rules

## Issue 3.5 — Right-panel pattern is underused
**What's wrong:** `show-detail-active-overview.png` has the best detail layout in the product: big date display, quick stats, travel block, copy link. That pattern doesn't extend to Release, Contact, or Gear detail consistently.

**Where it appears:**
- `release-detail-overview.png` — HAS a right panel (Rollout Timeline + Contact + Quick Stats + Duplicate/Delete). ✓ this is the best implementation.
- `release-detail-checklist.png`, `release-detail-tasks.png`, `release-detail-tracks.png` *(new v3)* — all preserve the right panel consistently across tabs ✓
- `show-detail-active-overview.png`, `show-detail-active-day-sheet.png`, `show-detail-active-setlist.png`, `show-detail-active-guest-list.png`, `show-detail-active-files.png` *(new v3 confirm)* — consistent right panel across all tabs ✓ excellent
- `gear-detail-overview.png`, `gear-detail-packing.png`, `gear-detail-files.png` *(new v3 confirm)* — right panel present across all tabs ✓
- `contact-detail.png` — DOESN'T. Single column. This is wrong.
- `crew-detail.png` *(new v3)* — DOESN'T. Single column. Same problem, scope extends.
- Settings pages don't have it, which is correct (settings is task-oriented, not entity-oriented).

**Fix:** Add right-panel to contact-detail AND crew-detail. For contact-detail: Avatar + contact card / Recent activity / Linked entities summary / Quick actions (Call, Email, Copy). For crew-detail: Avatar + Role chip / Total shows assigned / Avg per-show cost / Last assigned show / Quick actions (Call, Email, Book for show). E-3 scope grows.

**Ladders to:** Information hierarchy

## Issue 3.6 — Upgrade banner weight is too aggressive
**What's wrong:** Red-bordered panel with amber CTA button creates alarm-level urgency for a mundane feature-gating message.

**Where it appears:**
- `releases-list.png`, `merch-list.png`, `tasks-list.png`, `gear-list.png` — full-width red-bordered banner
- `money-budgets.png` — different treatment (amber border + amber CTA), less alarming — this is closer to correct

**Fix:** Standardize to:
- Muted amber background (8% opacity), amber border (25% opacity), amber left-aligned icon, dark text, amber CTA button right-aligned.
- Use red ONLY for destructive / error states.
- Height: 56px (compact), not 72px (currently).

**Ladders to:** Color discipline, interaction patterns

## Issue 3.7 — Calendar colors encode type; should encode source (artist)
**What's wrong:** Current calendar colors by type (show/release/tour/task). Execution also inconsistent: Tour banner pills are cyan, Tasks are purple, Releases are purple too on some views, Shows are orange/red.

Given the `FAVORITES` sidebar already implies multi-artist support, `artists.brand_color` already exists in schema, and multi-band is a stated positioning, **color-by-type is a dead end**. Shipping Batch E calendar polish on color-by-type would mean tearing it up to migrate to color-by-artist later.

**Where it appears:**
- `calendar-month.png` — Shows = amber/red, Tasks purple, Releases purple, Tours cyan
- `calendar-week.png` — tour pills and release pills both violet-ish
- `calendar-agenda.png` — different executions: SETTLED green pill, CONFIRMED green, HOLD purple, MASTERING amber, RELEASED green, DAY_OF amber
- `calendar-agenda-mobile.png` *(new v3)* — same inconsistencies reproduce on mobile

**Fix:** See D-7 for architecture commitment. In short: color encodes artist (pulled from `artists.brand_color`), icon encodes type, orphan/system events use `--color-chart-neutral`.

**Ladders to:** Color discipline, §0.2 (architectural irreversibility)

## Issue 3.8 — "Day Of" amber dot competes with primary CTA amber on show detail
**What's wrong:** The "Day Of" status chip, the favorite star, the "Share day sheet with crew" amber-bordered bar, and the (implicit) primary action all fight for attention on `show-detail-active-overview.png`.

**Where it appears:** `show-detail-active-overview.png`, `show-detail-active-advancing.png`, `show-detail-active-day-sheet.png` *(new v3)*, `show-detail-active-setlist.png` *(new v3)*, `show-detail-active-guest-list.png` *(new v3)*, `show-detail-active-files.png` *(new v3)* — confirmed across all Active-show tabs.

**Fix:** Day Of migrates to `--color-status-day-of` (#e8621f) which is distinct from `--color-accent`. Kill the amber border on "Share day sheet with crew" — make it a neutral primary button in the top action area, not a banner. Favorite star: amber when starred, neutral when not — but make sure the "off" state is clearly interactive.

**Ladders to:** Color discipline

## Issue 3.9 — Pricing inconsistency between marketing and in-app
**What's wrong:**
- `marketing-pricing.png` shows Free / Plus $12.50/mo / Pro $33/mo (annual billing shown)
- `settings-subscription.png` shows Plus $15/mo / Pro $40/mo (monthly billing shown)
- No label on marketing page clarifying "annual billing"; in-app has a Monthly/Annual toggle

Also: marketing page has a "Best Popular" typo on the Plus tier — should be "Most Popular".

**Fix:**
- Marketing page: lead with the price you want people to compare against competitors. Decide: annual (lower, more flattering) or monthly (truth). If annual, label prominently "$12.50/mo when billed annually — $150/yr" directly under price.
- In-app settings: show BOTH with toggle, already half-done. Make the toggle work (Annual save 17% visible when Monthly selected).
- Currency: marketing is USD, in-app is USD, Money pages are EUR. **Internationalization decision needed — see also new issue 3.16.** But unify within one surface.
- Fix the "Best Popular" typo (2-minute job, do in D-6 alongside other copy cleanups).

**Ladders to:** Information hierarchy

## Issue 3.10 — Stat number colors on dashboards don't semantic-encode anything
**What's wrong:** Related to 3.2 but worth calling separately: the colors on dashboard stat tiles don't mean anything consistent. Green for "this is good"? Then Expenses shouldn't be red (expenses ≠ bad news). Violet for Crew Costs? No reason.

**Where it appears:** `dashboard.png`, `money-overview.png`, `money-overview-mobile.png` *(new v3)*, `money-reports.png`, `money-income.png` *(new v3)*, `money-band-bank.png` *(new v3)*, `gear-list.png` (Total Items white, Declared Value white, Insured Value **cyan**, Needs Attention **red**), `show-detail-active-guest-list.png` *(new v3 — Names 8 / Total Heads 14 / Checked In 0 all white — correct, use as reference)*.

**Fix:**
- Dashboard stat tiles: all values in white. Label color can be `--color-fg-muted`.
- Alert/attention stats (like "Needs Attention: 2" in gear-list): use a small colored dot or pill next to the number, not colored number. Pair with icon per §0.1.
- Deltas (month-over-month): keep green/red, always paired with ▲/▼.

**Ladders to:** Color discipline, §0.1

## Issue 3.11 — Ambient page gradient / glow is missing
**What's wrong:** Every page is flat `--color-bg`. There's no sense of depth or atmosphere. Linear, Vercel, Attio all use a subtle radial-gradient glow near the top-left (where nav meets content) to create warmth without decoration.

**Where it appears:** Every page. The sidebar+content feels cold.

**Fix:** Add a very subtle radial gradient behind the content area:
```css
body {
  background: radial-gradient(
    ellipse 800px 600px at 20% 0%,
    rgba(249, 115, 22, 0.03),
    transparent 60%
  ), var(--color-bg);
}
```
Almost invisible. You only notice if it's removed. Parking-lot item you already flagged — worth doing in Batch F.

**Ladders to:** Craft

## Issue 3.12 — Settlement error surfaces as raw Postgres to the user
**What's wrong:** `show-detail-active-settlement.png` shows: `duplicate key value violates unique constraint "settlements_show_id_key"`

**Where it appears:** `show-detail-active-settlement.png`

**Fix:** This is tech, not design — but design-adjacent: wrap all Supabase errors in a user-friendly error boundary. "Something went wrong loading this settlement. Retry?" Not the SQL string.

**Priority:** Pre-beta blocker. Scope-wise belongs in E-6 but should land before any outside user sees the product.

**Ladders to:** Craft, trust

## Issue 3.13 — Mobile loses the right-panel information entirely
**What's wrong:** Right-panel content drops on mobile, violating §0.3 mobile-parity meta-principle. The information the right-panel holds is the structural meta of the entity — losing it changes what the page is.

**Where it appears:**
- `show-detail-active-overview-mobile.png` — no Quick Stats / Travel / Copy link visible.
- `release-detail-overview-mobile.png` *(new v3)* — **drops Rollout Timeline, Contact, Quick Stats entirely.** Rollout Timeline is the primary navigational spine of a release; its absence on mobile is the most severe parity violation in the product.
- `contact-detail-mobile.png` *(new v3)* — no right-panel content, but this is equivalent to desktop contact-detail which also lacks a right panel. Desktop bug first (3.5); mobile inherits.

**Fix:** Implement one of three mobile-parity patterns per §1.5:
1. Summary strip above tabs (show-detail)
2. Collapsible Summary section as first card (release-detail — Rollout Timeline is data-rich enough to warrant this)
3. Summary tab (gear-detail if desktop right-panel grows further)

F-3 scope: now covers show-detail + release-detail + (post-E-3) contact-detail + crew-detail mobile variants.

**Ladders to:** Information hierarchy, §0.3

## Issue 3.14 — Brand color field in Settings allows users to pick amber — clash with UI chrome
**What's wrong:** `settings-profile.png` shows Pale Wire's BRAND COLOR = `#f97316` — the same as ArtistHQ's accent. Every generated rider / EPK would visually merge with the UI.

**Fix:**
- Brand color field: document clearly it's used for generated docs, not UI.
- Default brand color should NOT be the ArtistHQ accent. Default to neutral white or a stored palette not including amber.
- Consider presets (band picks from 12 curated colors + custom) to prevent bad brand color decisions.
- Copy: "Used on generated documents (riders, EPK). Don't pick ArtistHQ amber — your docs will disappear into the UI."

**Ladders to:** Craft, product coherence. Critical before PDF generation skills ship.

## Issue 3.15 — Calendar Month shows too few events per day cell
**What's wrong:** `calendar-month.png` — of 30 days, 8 have events. The grid is 80% empty. Month is low-density for this audience. Bands don't have daily schedules; they have sparse major events.

**Where it appears:** `calendar-month.png`

**Fix:** Agenda should be the default calendar view for this user. Month is a weekly-planner view; Agenda is a tour-schedule view. Bands think in tours, not in calendar-months. `calendar-agenda.png` is already your best view — make it default.

Week view: hide from tab row, keep `?view=week` accessible via URL param (§0.2 reversibility). If analytics show no URL hits in 6 months, delete in a later batch.

**Ladders to:** Product-market fit, information hierarchy, §0.2

## Issue 3.16 — Money Income ships in two currencies simultaneously *(new in v3)*
**What's wrong:** `money-income.png` hero row reads `$20,450.00 / $15,010.00 / $35,460.00` in USD. Every row below that reads in EUR: `€3,000`, `€3,500`, `€1,200`, `€540`, `€890`, etc. The stats are not currency-converted totals; they're EUR figures displayed with a USD symbol. This is a presentation-layer bug that originates in locale handling.

**Where it appears:**
- `money-income.png` — hero vs rows mismatch
- Possibly elsewhere — audit all Money surfaces for currency format inheritance

The user's `settings-preferences.png` is set to `EUR (€)`. The hero ignores that setting.

**Fix:**
- Short-term (design-adjacent): ship currency format through a single formatter that reads `workspace.currency`. Ban literal `$` in templates.
- Medium-term: Money page needs a clear currency contract. Either (a) ALL values shown in user's preferred currency with server-side FX conversion, or (b) values shown in source currency with explicit per-row currency chips.
- Pre-beta: any surface mixing currencies on one page must display a small currency-note: "Showing €. Convert to $ USD."
- This is also tech debt — flag to pair with backend work. Design alone can't fix it.

**Ladders to:** Craft, trust, §0.1 (the symbol IS load-bearing and currently misleads)

## Issue 3.17 — Settings tab row overflows to two lines on every settings surface *(new in v3)*
**What's wrong:** The Settings top-nav is 9 tabs: Profile / Links / Preferences / Notifications / Subscription / Team / Band / Export / Account. On standard viewport widths, it wraps to 2 lines — second line has only 1 tab ("Account"), which makes the second row look like a misplaced active-tab indicator rather than an overflow.

**Where it appears:** `settings-account.png`, `settings-band.png`, `settings-export.png`, `settings-preferences.png`, `settings-team.png`, `settings-notifications.png` *(new v3)* — confirmed across every settings tab. Also true on `settings-profile.png` from v2.

Worse: tab row is near the Settings H1 and "Save Changes" button, so the visual relationship among H1 / subtitle / tabs / form card is noisy.

**Fix:** Three options, in order of preference:
1. **Left sidebar Settings nav.** Settings becomes its own layout with a left-column nav (like Linear settings). Nine items is comfortable in a sidebar, hostile in a tab row.
2. **Grouped tab row.** Compress to 5 top-level groups: Profile (includes Links, Account), Workspace (Preferences, Notifications), Billing (Subscription), Collaboration (Team, Band), Data (Export). Sub-tabs inside each.
3. **Overflow tab.** Keep 7 visible + "More ▾" dropdown containing Export + Account.

Option 1 is the right architectural call but larger scope. Ship Option 3 as a stopgap, move to Option 1 in Batch F.

**Ladders to:** Information hierarchy, navigation patterns

## Issue 3.18 — Files page uses a left subnav + folders IA that conflicts with top-tabs pattern used elsewhere *(new in v3)*
**What's wrong:** `docs-list.png` introduces a three-tier navigation structure unique in the product: sidebar (primary) → left subnav with All Files / Favorites + FOLDERS list (secondary) → content area. Every other top-level page uses top-pill tabs for secondary navigation (Money, Shows, Releases, Crew, Gear).

Two IA patterns for equivalent-rank operations creates a learning cost and a maintenance cost. The Files page isn't architecturally different from other list pages; it's treating itself as different.

**Where it appears:** `docs-list.png`

**Fix:** Two options:
1. **Conform Files to the top-tabs pattern.** [All / Favorites / Riders / Press Kit / Admin / Folders] as top pills. Folders become a filter, not a subnav. Files loses its left subnav.
2. **Accept Files as a file-manager special case.** Files actually has a natural left-tree structure (folders) that other pages don't. Keep the left subnav AND formalize it as a second layout archetype: "entity list" vs "file manager." Document the archetype so future pages (e.g., Media Library, Press Archive) can adopt it intentionally.

Recommendation: Option 2. The folder hierarchy on `docs-list.png` is genuinely useful for a band with 4+ years of show docs, press assets, and release materials. Don't flatten it to conform to a pattern not designed for hierarchical content. But **name the archetype** so it doesn't proliferate by accident.

**Ladders to:** Navigation patterns, information architecture

---

# 4. Per-Page Review

## 4.1 — `landing.png`
**Job:** Convert skeptics into waitlist signups.

**Works:**
- Tagline placement and weight: "Built by musicians. / For musicians who / **mean business.**" — accent on "business" is exactly where you want the payoff.
- Minimal nav (Features / Pricing / Sign in) — Linear-clean.
- Waitlist email + CTA above the fold.

**Needs fixing:**
- **The product screenshot is tiny and untreated.** Vercel, Linear, and Attio all lead with massive, gorgeously-framed product hero shots — glass frame, subtle shadow, scroll-animated. Yours is a sad 320x-ish thumbnail. Replace with full-bleed-near dashboard screenshot at 1200px+ with soft glow.
- **The landing page is ~2000px tall with maybe 400px of actual content.** Vast dead space between hero and pricing. Need 3-4 scroll sections: Hero → Product hero → "Built for touring" proof section → Differentiator triplet (Setlist→PRO / Show→Finance / Gear→Rider) → Pricing → CTA.
- **No social proof.** Even pre-launch, you can feature Bite Down as case study ("Built alongside Bite Down's spring tour"). Artist Growth does this well.
- **The amber gradient on "business"** is distinctive — keep it. Don't let color-discipline rules strip this.

**Benchmark:** Linear's landing hero is the reference. Attio is close second.

## 4.2 — `dashboard.png`
**Job:** Show the band the state of their business in one glance and get them to the most urgent action.

**Works:**
- Bento grid concept is right.
- "Good evening, Jacob" personalization is warm without being cloying.
- Three secondary cards (Tasks / Attention Needed / Recent Expenses) in one row is dense in the Artist Growth sense.

**Needs fixing:**
- **Hero stats colored top-to-bottom** (Next Show white, Revenue green, Expenses red, Net green). Make all white; put green/red only on deltas. Add deltas if not already present ("REVENUE €11,700  ▲23% vs last month").
- **NEXT SHOW card should be bigger.** It's the most actionable thing on the dashboard. Currently equal weight to the accounting stats.
- **Card gap is tight** — bump to 16px.
- **THIS WEEK row is good** — orange "TODAY" treatment works. Consider adding event dots below other days (The Lexington on MON).
- **"All shows →" / "All tasks →" / "View →"** links all amber. Kill the amber — make these neutral with amber hover.
- **RELEASES card**: progress bars amber. Kill — use muted amber at 40% or neutral.
- **PEOPLE & ASSETS card** at bottom is small. Demote to "Quick links" or move to right rail.
- **RECENT FILES** card: PDF icon red is fine (semantic), but consider just a file icon with file-type label.
- **Layout balance**: right column narrower than left. The three-card middle row creates visual "ladder". Tighten to 2-column with equal widths OR full-width sections.

**Benchmark:** Linear's workspace dashboard; Vercel's project dashboard. Note how Vercel hero-stats are ALL white.

## 4.3 — `shows-list-upcoming.png`
**Job:** Scan upcoming shows, filter by tour/status, click through to one.

**Works:**
- Table layout. Dates left, venue/tour stacked in primary column, city, status, prep, fee right-aligned.
- Venue name bold, tour name small muted — good typographic rhythm.
- "€12,600 confirmed" summary chip top-right — useful.
- JetBrains Mono on dates — consistent.

**Needs fixing:**
- **Huge right margin.** Table is ~70% of viewport, wasting 30%. Either widen table to 95% or use right side for a persistent filter/summary panel.
- **Three filter rows** above the table — see 3.4. Collapse.
- **"16/10 on Free · Upgrade"** pill in top bar is too low-urgency to be banner-shaped, too urgent to be a chip. Move to a persistent capacity strip in the sidebar or make it a small red pill.
- **Prep column** uses mixed iconography: checkmark ✓ or "?" circle. Switch to a single metaphor: progress ring with % or a single status dot.
- **Row density** could be tighter — currently ~56px rows. 44px would allow 2-3 more rows visible.

**Benchmark:** Linear issues list. Attio records list. Both have tighter row height.

## 4.4 — `releases-list.png`
**Job:** See upcoming and past releases, track rollout state.

**Works:**
- Grid vs Pipeline toggle — good for content that has two valid views.
- Cards show format (Single/EP), status (Mixing/Mastering), label/distributor, countdown.
- Pipeline view teaser available via button.

**Needs fixing:**
- **Only 2 cards on a wide viewport.** Grid should be 3-4 columns. Currently wastes horizontal space aggressively.
- **Progress bars amber** — kill.
- **Upgrade banner too loud** (see 3.6).
- **Album placeholder "1" and "EP" in empty square** — these covers look unloved. Either: embed Cover Art URL field into card workflow on creation, or use generated gradient placeholders like Spotify/Apple Music.
- **Filter chips on right** (All • Mixing (1) • Mastering (1) • Released (3)) with colored dots — good usage of color-as-filter-indicator, paired with text per §0.1. Keep this pattern.

**Benchmark:** Bandcamp artist dashboard (for card density), Linear Projects list (for pipeline view).

## 4.5 — `calendar-month.png`
**Job:** See a band's full calendar of shows, releases, tours, tasks.

**Works:**
- Fantastical-style mini-cal in right sidebar + agenda chunks below — the pattern is sound.
- Tour span pills at top ("Glass Meridian EU Spring" / "Nerve Drift Summer") — good concept.

**Needs fixing:**
- **Sparse grid**, 80% empty — see 3.15. Agenda as default.
- **Event blocks unclear encoding** — see 3.7 and D-7. Color-by-artist, icon-by-type.
- **Too much chrome**: four filter pills (Shows/Releases/Tours/Tasks) + $Financials + three action buttons (Plan Tour / Batch Add / +Show / +Release) + month nav + Today button. Compress.
- **Day numbers dim** on empty days (correct), but date "18" (today) gets amber pill — which conflicts with active day styling in mini-cal. Pick one treatment for "today" and use everywhere.

**Benchmark:** Fantastical dark theme. Cron before it got acquired.

## 4.6 — `money-overview.png`
**Job:** Show band their financial state at a glance.

**Works:**
- Revenue vs Expenses grouped bar chart with declared chart tokens — right palette, muted.
- Expense Breakdown + Income Sources donuts give category view.
- Recent Transactions list groups income and expense chronologically.
- Month-over-Month block at bottom shows deltas with arrows (good).

**Needs fixing:**
- **Every hero stat colored** (see 3.2 and 3.10).
- **Two donuts side-by-side** use different palettes — Expense Breakdown has purple/cyan/red/green; Income Sources has green/purple/cyan/red. Unify — or replace donuts with bar charts (easier to read at this category count).
- **Crew Costs card at bottom, violet number** — make white.
- **Chart axis labels**: €0.00, €2,765.00, €5,530.00 — too many decimals. Format as €0 / €2.7k / €5.5k / €8.3k / €11k.

**Benchmark:** Stripe Dashboard.

## 4.7 — `contacts-list.png`
**Job:** CRM — find and filter contacts by role, company, location.

**Works:**
- Columns: Name / Role / Company / Location / Email / Phone — covers 90% of needs.
- Avatar circles on left.
- Role badges with colored dots + text labels (§0.1 satisfied).

**Needs fixing:**
- **9 role colors** — see §1.1 and 3.1. Compress to 4 families.
- **Filter row is 9 pills wide**, each with count. Overwhelming. Use a role dropdown OR segmented control with "All / Show Side / Commerce / Advisory / Press".
- **Secondary meta row** "12 contacts · 3 promoters · 2 venue contacts · 1 booker · 1 label · 1 accountant" — too much detail. Either remove or move to tooltip/stats panel.
- **Table is full width** — good. Use as reference for shows-list width.
- **No row hover state visible** — add.

**Benchmark:** Attio contacts.

## 4.8 — `merch-list.png`
**Job:** Inventory management + restock planning for touring merch.

**Works:**
- Hero stats (Products / Total Units / At Cost / At Retail) quantify the business.
- Low Stock + Dead Stock alert cards — very tour-operations-useful.
- Per-product cards with variant stock and margin.
- **"Show Merch Planner"** with "Bring 6" recommendations per SKU per upcoming show — **this is a differentiator feature**. Don't touch it.

**Needs fixing:**
- **At Cost in red, At Retail in green** — why? Both are neutral valuations. Make white.
- **Low Stock header red, Dead Stock header red** — these warrant red (paired with text header per §0.1). ✓ keep.
- **Product card stock bars** color-coded by stock level — good use of color for alert. Pair with numeric count alongside.
- **"Bring 6" in amber** — kill amber, these are informational. Use neutral with bold weight.
- **"1 variant below reorder point"** warning chip uses amber — should migrate to `--color-status-warning` (distinct orange). ✓ keep meaning.
- **Show Merch Planner density** is excellent — it's the Linear-of-inventory view.

**Benchmark:** Shopify inventory view (but more restrained). Linear's issues → sub-issues structure as an analog.

## 4.9 — `gear-list.png`
**Job:** Gear inventory, serial/value tracking, ATA Carnet export.

**Works:**
- Grouped by category with colored category dots (paired with text labels per §0.1).
- Per-item row: icon / name / model / S/N / price / condition chip.
- "Needs Attention: 2 fair or poor" — good summary.
- **ATA Carnet button** — musician-specific feature, excellent.
- **Value by Category** bar chart at bottom with hues matching category dots — good visual reinforcement.

**Needs fixing:**
- **6 category colors** — compress per §1.1 rules.
- **Bar chart colors saturated** — reduce saturation to match chart-income/expense restraint.
- **"Declared Value €18,990 / Insured Value €19,990"** — both white.
- **"Needs Attention: 2" in red** — this is a true alert. ✓ keep, pair with icon.
- **Upgrade banner** (see 3.6).

**Benchmark:** Your own show-detail-active-advancing density — aim for that level of polish here.

## 4.10 — `tasks-list.png`
**Job:** Kanban-style task board for the band.

**Works:**
- Board/List toggle — good.
- Column headers with counts.
- Task cards with priority / type / due / linked entity tags.
- "To Do / In Progress / Done" column structure is standard-but-correct.

**Needs fixing:**
- **Hero stats** (To Do 8 purple, In Progress 2 cyan, Overdue 1 red, Done This Week 1 green) — colors don't map to column colors. Align: column dot color = stat color = priority color family.
- **Priority badges** (!! URGENT pink, ↑ HIGH red, − MEDIUM orange, ↓ LOW neutral) — symbols (!!, ↑, −, ↓) already satisfy §0.1 non-color-alone. Keep symbols; compress hue palette.
- **Tag colors inconsistent** — "logistics" cyan, "Release" amber, "shows" dark, "merch" amber, "Marketing" amber, "Admin" muted. Unify: tags are neutral chips with muted fg; entity reference can have small colored dot.
- **Card padding generous** — actually feels right. Good.
- **Quick add task input** — good pattern.

**Benchmark:** Linear.

## 4.11 — `show-detail-active-overview.png`
**Job:** Overview of a single show with quick-jump to deeper tabs.

**This is the strongest page in the product.** Measure everything else against it.

**Works:**
- Right panel (APR 20 big, Day Of/Invoiced chips, €900, Quick Stats, Travel, Copy link).
- Breadcrumb back link top-left.
- Tabs strip under entity header.
- "Share day sheet with crew" call-to-action bar under tabs (intent is right even if styling needs work).
- Documents pill rack at the bottom.

**Needs fixing:**
- **"Day Of Monday, 20 April 2026"** meta strip above venue title — sits in weird tier between meta and display. Split: "Day Of" pill (now #e8621f) to the right of entity name, date to muted line under city.
- **"Share day sheet with crew"** — convert from amber-bordered banner to a neutral primary button in the top-right action area.
- **Map card** takes significant real estate for marginal value. Replace with compact "Location" card and show map on click/hover.
- **"No expenses logged for this show"** empty state — icon good, text clear, consider adding inline "+ Log expense" CTA.
- **Notes block styling** is too low-weight — looks like placeholder text.
- **9 tabs** is the upper limit. Consider grouping: [Info: Overview | Advancing | Day Sheet | Setlist] and [Ops: Guest List | Crew | Files] and [Money: Finances | Settlement]. Or: promote Settlement to action button (terminates the show journey).

**Benchmark:** Attio records. Linear issue detail.

## 4.12 — `show-detail-active-advancing.png`
**Job:** Complete advancing sheet for a touring show.

**This is the killer feature for working bands.** The density is Artist Growth-level.

**Works:**
- Every advancing section present.
- JetBrains Mono on times, capacity, power specs — exactly right semantic use.
- PROVIDED/NEEDED backline badges with green/amber — semantic and readable, paired with text per §0.1.
- Strikethrough on completed checklist items.
- The 6×4 venue grid, 32A power, load-in times — this is what gets bands to recommend the product.

**Needs fixing:**
- **Section headers (DEAL, SCHEDULE, VENUE)** are tiny. Bump to `--text-label` at 11px bold mono — they should anchor each section.
- **Collapsible sections + sticky TOC** (F-4). Currently everything expanded.
- **"Edit advancing info" button at bottom** is weak — make prominent, ideally floating fixed at bottom-right during scroll.
- **Right panel** consistent with Overview — ✓ good.

**Benchmark:** Artist Growth advancing sheets. Your execution matches and in places beats.

## 4.13 — `show-detail-active-settlement.png`
**Job:** Settle a show after performance — reconcile guarantee vs deductions vs ticket sales.

**Works:**
- N/A — currently showing a Postgres error.

**Needs fixing:**
- **Wrap the SQL error** (see 3.12 / E-6). Pre-beta blocker.
- **Floating "1 Issue ×" pill bottom-left** — if Next.js error overlay, fine in dev. If prod-shipping UI, kill.
- Once error is fixed, this page should match `show-detail-settled-settlement.png` structure.

**Benchmark:** N/A until fixed.

## 4.14 — `show-detail-settled-settlement.png`
**Job:** Show a completed settlement, immutable, exportable.

**Critical workflow page. Well-executed.**

**Works:**
- **"Settled 09 Mar 2026 READ-ONLY" success bar** — green, clear, sets expectations. ✓
- **FROZEN badges** on each section — user knows they can't edit. ✓
- **Ticket Sales** with progress bars per tier, total GBOR — dense and readable.
- **Deductions** with category pills, red amounts, total.
- **Settlement Summary block** with GBOR → Deductions → NBOR → Artist Payout → Less deposit → Balance Due.
- **Band Split** with rule + per-member breakdown.
- **Sign-off section** with checkbox + signature + date.
- **Right panel Ticket Sales mini-chart** (445/500 capacity, 89% sold) with progress — useful.

**Needs fixing:**
- **Artist Payout €1,200 in amber** — should be white (biggest number, primary fact, see 3.2).
- **Deduction category pills**: RENTAL / PRODUCTION / CATERING in amber — these are neutral category tags, make them gray chips.
- **"Settled" and "Paid" chips at top-right** of right panel — both green but differently saturated; standardize to `--color-status-settled`.
- **"Mark as Settled" button** — green-fill for this HAPPY completion action is defensible. Make consistent with wherever else a "commit this" action exists.
- **Export Settlement Sheet** (amber) as secondary — if Mark as Settled is primary, Export is secondary and should be neutral-bordered, not amber.

**Benchmark:** You are the reference. Clean up 3.2.

## 4.15 — `release-detail-overview.png`
**Job:** Single release detail with rollout milestones and metadata.

**Works:**
- Cover art square + title + format + status chips + countdown.
- Tabs: Overview | Checklist | Tracks | Tasks (2) | Files.
- Right panel: ROLLOUT TIMELINE / CONTACT / QUICK STATS / Duplicate/Delete.

**Needs fixing:**
- **Right panel** ✓ good — this is the pattern to propagate.
- **"Add links →" Streaming Links CTA** in amber text — neutral bordered button.
- **Contact "James Murphy" name in amber** — make white.
- **"Mixing" status chip** amber — use a release-status color.
- **"NOTES" section header amber** — 11px mono label, no color.
- **Cover art placeholder** — same observation as releases-list: feels unloved.
- **Checklist/Tracks/Tasks tabs count badges** — Tasks (2) is good pattern. Extend.

**Benchmark:** Linear projects detail. Bandcamp releases manager.

## 4.16 — `contact-detail.png`
**Job:** Single contact with activity and linked shows/releases/files.

**Works:**
- Entity name + company + location as header block.
- Role chip next to name (paired with text per §0.1).
- Clear field stack (Email / Phone / Company / Role / Location / Notes).
- Linked Shows / Linked Releases / Linked Files sections with inline link+upload affordances.

**Needs fixing:**
- **No right panel** — see 3.5. Fix in E-3.
- **Email and phone fully amber** — strip. Use white text with "Copy" button as the amber affordance.
- **Avatar circle 80px** — too big. Shrink to 56px.
- **Delete contact** as text link at bottom — make it a neutral bordered danger button aligned right.
- **Notes: "A&R. Signed us Sept 2024."** — valuable detail, deserves more visual treatment.

**Benchmark:** Attio contact record. Superhuman contact sidebar.

## 4.17 — `gear-detail-maintenance.png`
**Job:** Single gear item with maintenance log.

**Works:**
- Item header: icon / name / type / GOOD pill.
- Tabs: Overview | Maintenance | Packing | Files.
- Maintenance: Total Spend / Last Service / Alerts cards + log rows.
- Right panel: CONDITION / VALUE / WARRANTY & PURCHASE / QUICK FACTS.

**Needs fixing:**
- **"GOOD" pill top-right** — consolidate with right-panel CONDITION block, don't duplicate.
- **Maintenance Log row** — amber "Setup" chip, date in mono, description. Chip color inconsistency: use neutral category tag.
- **"Expired 35d ago" in red** — correct use of danger, pair with icon (§0.1).
- **"+ Log Entry"** primary CTA amber — ✓ appropriate.
- **Quick Facts "Guitars"** with amber dot — use gear family color.

**Benchmark:** Linear issue detail with right-side meta panel.

## 4.18 — `money-reports.png`
**Job:** Periodic financial summary and tour profitability.

**Works:**
- Tour Profitability section at top is **genuinely great** — band manager / tour accountant view at a glance.
- Monthly Net Trend bar chart — green/red bars below/above zero line.
- Month table with Income / Expenses / Net per row.
- Expenses by Category + Income by Source at bottom.

**Needs fixing:**
- **Tour Profitability Fees/Costs/Net** all colored — fees should be white, costs red acceptable (paired with sign), net depends on sign (green with ▲ if positive).
- **Annual Summary hero stats** — white values, colored deltas only.
- **Monthly Net Trend**: saturated green on positive months — align with `--color-chart-income` muted tone.
- **Month table** shows Jul–Dec 2026 with dashes — good transparency. Make dashes lighter.
- **"This summary is for informational purposes only — not tax advice"** footer note — ✓ good CYA.

**Benchmark:** Stripe Dashboard reports view.

## 4.19 — `login-default.png`
**Job:** Sign in or sign up.

**Works:**
- Clean modal-centered card.
- ARTISTHQ wordmark (Syne) — ✓.
- Google SSO primary, magic-link secondary path, password tertiary.
- Low chrome, focused.

**Needs fixing:**
- **Card background** is subtly-darker-than-page — barely visible. Slight lift with stronger shadow would create more depth.
- Otherwise clean.

**Benchmark:** Linear login. Vercel login.

## 4.20 — `login-password-mode.png`
**Job:** Same as 4.19, password-auth variant.

**Works:**
- "Forgot password?" link in amber — fine, it's an action link.
- Sign in (amber filled) + Create account (neutral bordered) — good CTA hierarchy.
- "Use magic link instead" back-link bottom.

**Needs fixing:**
- Nothing. This page is quiet and correct.

**Benchmark:** N/A — this is fine.

## 4.21 — `shows-new.png`
**Job:** Add a show.

**Works:**
- Progressive disclosure — basics up top, collapsible Tour/Schedule/Deal/Hospitality sections below.
- Required-field asterisks.
- Clear section labels.

**Needs fixing:**
- **Collapsed sections at bottom** — label weight could bump to `--text-label` for consistency.
- **Amount field "0" placeholder + USD dropdown** — currency default: detect from artist location, don't default to USD for European band.
- **"Add Show" primary button full-width at bottom** — fine.
- **Left sidebar bg during modal-like flow** — sidebar appears transparent/leaky. Either lock the full-page form feel or make it clearly a modal overlay.

**Benchmark:** Linear "Create issue" form.

## 4.22 — `releases-new.png`
**Job:** Add a release.

**Works:**
- Clean form, Title required / Format / Status / Release Date required / Genre / Label / Distributor / Cover Art URL / Notes.
- Progressive from most important to least.

**Needs fixing:**
- **Same sidebar-leaking issue as shows-new**.
- **"Cover Art URL"** as URL input — bands rarely paste URLs. Make this upload or DistroKid integration in Phase 4. For now, add "or upload file" affordance.
- **Genre is free-text** — consider combobox with common genre suggestions.
- **Submit button** appears inactive (muted amber) — if disabled state awaiting Title, good. If active state, bump to full amber.

**Benchmark:** Linear issue create. Bandcamp release editor.

## 4.23 — `settings-profile.png`
**Job:** Manage artist profile info.

**Works:**
- Settings top-nav (Profile / Links / Preferences / Notifications / Subscription / Team / Band / Export / Account) is comprehensive.
- Hero "Settings" + subtitle + Save Changes top-right.
- Field grid: Name/Type, Genre/Country, Brand Color, Bio.
- Bio character counter — ✓.

**Needs fixing:**
- **Brand Color default = #f97316** — see 3.14 / D-6. Critical.
- **Avatar placeholder "PA"** — huge square, feels unloved. Round it.
- **Save Changes button always visible** — should be hidden/disabled until form is dirty.
- **Tab row wraps to 2 lines** — consider putting rare tabs (Export, Account) in an overflow dropdown. *(v3: escalated to cross-cutting issue 3.17.)*
- **"Type" dropdown with ×** close button — why? Band is already selected. Remove × for required fields.

**Benchmark:** Linear settings. Attio settings.

## 4.24 — `settings-subscription.png`
**Job:** See current plan, usage, upgrade options.

**Works:**
- "FREE" chip at top makes plan clear.
- Resource Usage grid with progress bars per resource — scannable.
- Upgrade card with Monthly/Annual toggle, Plus/Pro columns.

**Needs fixing:**
- **Most progress bars red** — accurate but aggressive. Differentiate "at-limit" (warning amber) from "over-limit" (red danger). Pair with count text per §0.1.
- **Price inconsistency with marketing** — see 3.9.
- **"Upgrade to Plus" / "Upgrade to Pro"** — both full amber. Pro should look less promoted so Plus is clearly the recommended path. Or: "Most Popular" tag on Plus.

**Benchmark:** Linear pricing. Vercel billing.

## 4.25 — `marketing-pricing.png`
**Job:** Convert prospects by clearly showing pricing tiers.

**Works:**
- 3 tiers (Free / Plus / Pro) — the right count.
- "Best Popular" badge on Plus — **typo, should be "Most Popular"**.
- Monthly/Annual toggle with "Save 2 months" highlight.
- Feature bullets per tier.

**Needs fixing:**
- **"Best Popular" → "Most Popular"** (D-6 copy fix).
- **Annual billing not labeled** prominently at large scale.
- **Feature bullets** use checkmarks + amber accent — checkmarks can be green or neutral white; amber feels wrong.
- **CTA colors**: Plus amber filled (✓), Free & Pro neutral bordered. Good.
- **Large dead space below pricing** — fill with FAQ, comparison table, or customer quote.
- **Consistent with in-app pricing** — see 3.9.

**Benchmark:** Linear pricing. Stripe pricing.

## 4.26 — `calendar-week.png`
**Job:** See a week of events hour-by-hour.

**Works:**
- Standard week layout.
- "Today" column (Sat 18) highlighted.
- Right sidebar still useful.

**Needs fixing:**
- **Week view almost always empty for bands** — see 3.15, E-7. Hide from tab row, keep `?view=week` accessible per §0.2.
- If kept visible in any form: compress to 14:00–02:00 (the actual hours touring musicians care about).

**Benchmark:** Fantastical's week view (but your density is too low to match).

## 4.27 — `calendar-agenda.png`
**Job:** Chronological list of all upcoming events.

**This should be the default calendar view.**

**Works:**
- Grouped by month.
- Per-event: date chip / icon / title / sub-line / amount / status chip.
- Scannable vertically.
- Ties together all four entity types in one feed.

**Needs fixing:**
- **Status chip colors** see 3.7 and D-7 (migrate to color-by-artist).
- **Month group header** just the name — add count ("April 2026 · 1 show · 6 tasks").
- **Very long scroll** (~2000px). Consider sticky month headers or collapsible months.
- **Hover states**: make rows hover-able and clickable.

**Benchmark:** Fantastical agenda. Linear issues sorted by date.

## 4.28 — `money-expenses.png`
**Job:** See all expenses, filter by category, log new.

**Works:**
- Table columns: checkbox / date (2-line) / description + sub-linked-show / category pill / amount.
- Amounts all red (consistent semantic).
- Category pills color-coded + labeled (§0.1 satisfied).
- "Log Expense" primary CTA left.
- Category filter pills.

**Needs fixing:**
- **Category pill colors inconsistent** — group into 2 families per D-4.
- **"Other" category** is overused — add Recording, Gear, Rehearsal as first-class categories.
- **Dates in 2-line form** (11 Apr / 2026) — ✓ good mono treatment.
- **TOTAL (19) row** — good. Add avg / max / date range context.
- **Filter pill row** — segmented control to differentiate from nav pills.

**Benchmark:** Stripe Dashboard payouts.

## 4.29 — `money-budgets.png`
**Job:** Budget tracking (plan-gated feature).

**Works:**
- Clear upgrade messaging.
- Tab and filter chrome remains visible (context preserved).

**Needs fixing:**
- **Upgrade banner styling** — see 3.6.
- **Everything below banner is empty** — add a placeholder mockup of what Budgets looks like (blurred/disabled) so users see the value.
- **Period filter pills visible but inactive** — disable too, since they're not actionable here.

**Benchmark:** Notion's plan-gated features.

## 4.30 — `show-detail-active-settlement-mobile.png`
**Job:** Settlement on mobile.

**Works:**
- **Very well-executed mobile layout.** Stacked cards with label/value rows, clear hierarchy.
- Settlement Summary with all math visible.
- Sign-off cards stacked with checkboxes.
- "Mark as Settled" green bordered primary + "Export Settlement Sheet" amber below — good hierarchy.

**Needs fixing:**
- **Artist Payout €900 amber** — white instead.
- **Three stacked empty cards** (Ticket Sales / Deductions / Merchandise, all "No X yet") feels sparse. Combine into one "Not yet settled" empty state that expands on data entry.

**Benchmark:** Stripe iOS app. Linear mobile issue view.

## 4.31 — `dashboard-mobile.png`
**Job:** Mobile dashboard.

**Works:**
- Same bento structure stacked vertically.
- All cards present.

**Needs fixing:**
- **Hero stats full-width stack** — consider 2x2 grid to preserve scan-ability.
- **Show Pipeline table** columns squeeze on mobile — consider dropping city or making row tappable to expand.
- **Attention Needed** card full-width — ✓ keep.
- **Tab bar (Home | Shows | Merch | Money | More)** — not visible in screenshot but per spec should be persistent. Verify.

**Benchmark:** Stripe iOS dashboard. Linear mobile.

## 4.32 — `show-detail-active-overview-mobile.png`
**Job:** Mobile show overview.

**Works:**
- Tabs scroll horizontally — right pattern.
- Share day sheet amber button.
- Cards stacked cleanly.
- Documents pill rack wraps.

**Needs fixing:**
- **Right panel content missing entirely** — see 3.13 / F-3.
- **€900 under show name** — just number alone, where's the "Guarantee + %" context?
- **Favorite / ... / 1/8 / ← → / SHOW-009** crammed top-right — on 375px viewport this is tight. Move pager into slide-down or overflow.

**Benchmark:** Stripe iOS customer detail.

## 4.33 — `settings-account.png` *(new in v3)*
**Job:** Manage login identity and the nuclear-option workspace delete.

**Works:**
- Simple two-field grid (Display Name / Email). Email correctly shown as uneditable (grayed).
- "Signed in as jacob.mild@gmail.com" + Sign Out on a separate row — clear separation from edit form.
- **Danger Zone treatment is correct**: red border, red bg tint, red "Delete 'Pale Wire'" button — appropriate escalation for an irreversible action. Matches §1.4 destructive rule.

**Needs fixing:**
- **"Save Changes" amber button at top-right** is always visible regardless of dirty state — same observation as settings-profile. Should gray out until form is actually dirty.
- **Tab row overflow** — see 3.17. Account is the orphan second-line tab; IA fix needed.
- **"Sign Out" neutral-bordered button** next to "Delete Pale Wire" is not strong enough visual separation. Sign-out is benign; Delete is catastrophic. Push Delete into its own Danger Zone card (it's close — the card wrapper is there — but the button needs explicit `are you sure?` confirmation UX on click, which isn't visible here but must be built).
- **Page subtitle** "Manage your artist profile, team, and preferences" duplicates across every Settings tab and doesn't describe Account specifically. Subtitle should contextualize: "Account — your login and workspace ownership."

**Benchmark:** Linear settings → Account. Vercel account settings.

## 4.34 — `settings-band.png` *(new in v3)*
**Job:** Define who counts as a band member (for payouts) and how splits work.

**Works:**
- Two-section structure: "Band members" (who) then "Split rules" (how). Correct mental model.
- Band-member toggle as a checkbox per person — clear semantic.
- ADMIN chip next to Jacob — role transparency.
- "Equal Split" with EQUAL + DEFAULT + "1 member" meta chips — lots of information in a compact row.

**Needs fixing:**
- **DEFAULT chip is filled amber** — this is a marker, not a CTA or active state. Migrate to `--color-chip-default` (neutral). This is a Tier-1 violation and the single cleanest example of §1.1's new "marker chips use neutral" rule.
- **EQUAL chip** is neutral-bordered — correct treatment, use as reference for DEFAULT fix.
- **ADMIN chip** amber — migrate to neutral; role markers aren't active states.
- **"Add rule" button** amber text — make it a neutral-bordered secondary button. Adding a split rule is a normal workflow action, not the primary page CTA.
- **Help text** below section headers is strong and specific ("Managers and other team members stay off to keep them out of payouts"). Keep this copy voice — it's musician-to-musician, not legalese.
- **Settlement preview missing** — a split rule's meaning is abstract without an example. Show: "For a €1,000 payout, Jacob receives €1,000." Attio does this for record rules.

**Benchmark:** Splitwise group settings. Linear cycle config for the "rule → preview" pattern.

## 4.35 — `settings-team.png` *(new in v3)*
**Job:** Invite collaborators to the workspace and manage their access.

**Works:**
- Single-row-per-person layout with avatar, name, email, role chip.
- "INVITE BY EMAIL" section clearly separated from member list.
- (you) annotation next to Jacob's name — warm, clarifies self.

**Needs fixing:**
- **ADMIN role chip amber** — same fix as settings-band. Neutral chip.
- **Invite button text amber + neutral border** — mixed affordance. Make it either (a) filled amber if it's the primary action on this page (which it arguably is — adding team is why you come here), or (b) clearly secondary. Currently it reads as "disabled" because the amber text on neutral bg is the weakest of the three CTA tiers.
- **No role dropdown on invitation** — a Pro plan will eventually want Member / Admin / Viewer roles. Even if only Admin exists today, put a role dropdown in the invite flow so the IA scales.
- **Empty "other members" state** — Jacob is the only member. Add encouragement: "Invite your bandmates so settlement splits and task assignments work out of the box."
- **No pending invitations section** — once you ship invites, pending invites need visibility.

**Benchmark:** Linear workspace members. Notion teamspace members.

## 4.36 — `settings-export.png` *(new in v3)*
**Job:** Let the band take their data out. Portability commitment.

**Works:**
- **Six downloads in a 2x3 grid** — Shows, Contacts, Releases, Expenses, Crew, Gear. This is correct scope for CSV export.
- **Neutral bordered buttons with download icon** — consistent affordance, no amber abuse here. ✓
- **"Download your data as CSV files"** subtitle — clear, no waffle.

**Needs fixing:**
- **This page is trivially empty.** The card wrapper is ~50% of viewport with six small buttons. Either (a) expand with export preview (number of rows per dataset, last modified) or (b) shrink the card to content-width so the page doesn't feel abandoned.
- **No "export all as ZIP"** — for GDPR data-portability requests and for artists leaving the platform, a single "Download everything" is the expected pattern. Add it.
- **No format options** — CSV only. Musicians sometimes want JSON (for devs) or XLSX (for accountants). Low priority, but flag.
- **No "Schedule automatic backups"** — Pro feature opportunity. "Weekly email with CSV attachments" would differentiate.
- **Tab row overflow** — same as other Settings tabs, see 3.17.

**Benchmark:** GitHub data export. Notion workspace export.

## 4.37 — `settings-links.png` *(new in v3)*
**Job:** Centralize band's outbound links for reuse in EPK, rider, web generation.

**Works:**
- Three clear sections (Website / Streaming / Social) + Patreon as coda.
- Mono-uppercase labels above each field.
- Empty states shown as placeholder text ("https://soundcloud.com/...", "@handle") — correct pattern.

**Needs fixing:**
- **Every filled link value displays in amber** — this is the worst single-page amber abuse in the product. 10+ amber strings on one page. **All of these must migrate to `--color-fg` or `--color-info`.** See 3.1 and new D-9.
- **Field values read as hyperlinks** — they're actually just text values stored for later use (in generated docs). Consider displaying them as "text with copy button" rather than "text styled as link." Currently ambiguous whether clicking does anything.
- **No validation indicators** — invalid Spotify URLs, wrong-domain YouTube links, malformed Bandcamp URLs all render the same as valid ones. Add inline validation with `--color-success` check when URL matches expected pattern.
- **Mixed naming convention** — "SPOTIFY" all-caps label, "YouTube" sentence-case elsewhere would be inconsistent. Here all are caps-mono, which is correct per §1.2. Keep.
- **Section separators** — STREAMING and SOCIAL labels float above fields but lack visual card-within-card separation. A thin divider or subtle background tint would clarify grouping.
- **"Streaming" vs "Social"** — arguably Patreon belongs in Commerce / Support, not its own hanging section. Create a third category or fold it in.

**Benchmark:** Linktree settings. Notion page with multi-field inputs.

## 4.38 — `settings-preferences.png` *(new in v3)*
**Job:** Set workspace-wide defaults (currency, date format, timezone, measurements).

**Works:**
- **2x2 grid** — perfect shape for 4 unrelated dropdowns.
- **EUR (€) / DD/MM/YYYY (31/12/2026) / Stockholm / Metric (kg/cm)** — each dropdown shows both the code/value and an explicit example. Excellent.
- **Quiet page, no amber misuse, no extra chrome.** This page is what other Settings tabs should aspire to.

**Needs fixing:**
- **This is the reference Settings page.** It's compact, clear, correct. Use as the template for the simpler Settings tabs (Account, Team, Export).
- **No "Apply to all artists in this workspace?"** — if a user has multiple bands, these settings probably shouldn't cascade silently. When multi-artist ships (implied by FAVORITES), add scope clarification.
- **Currency dropdown should drive `money-income.png` hero** — see 3.16. This preference is set to EUR but the Income hero shows USD. The preference is ignored by that surface. Fix 3.16 to honor this setting.
- **Tab row overflow** — 3.17.

**Benchmark:** Linear workspace preferences.

## 4.39 — `settings-notifications.png` *(new in v3)*
**Job:** Granular opt-in/out per notification type.

**Works:**
- **Six category groups** (Tasks / Shows / Releases / Money / Team / Gear) — covers product scope cleanly.
- **Master toggle per group + sub-toggles within** — correct two-tier UX. Matches Apple System Settings, Linear notification rules.
- **Sub-toggle copy is specific and actionable**: "Task overdue", "Payment overdue", "Release milestone", "Warranty expiring" — no vague "updates to your account" language.

**Needs fixing:**
- **17 toggles amber on the page at once** — legitimate under new §1.1 rule 6 (toggles-on = active state). But: confirm the off-state toggle is clearly distinguishable. Current dark theme can make "off" feel identical to "disabled" visually. Verify contrast.
- **No channel selector per notification** — today it's binary on/off. Bands usually want critical alerts by push + email and low-priority by email only. Defer to E/F scope: add Email / Push / In-app columns.
- **No priority hinting** — all notification types visually flat. Payment overdue and Task completed are wildly different urgencies. Consider subtle priority marker next to each row (small dot: red/amber/gray) pairing with icon. Do NOT color the toggle itself — only the priority marker.
- **No "quiet hours" or DND** — touring musicians are on stage, in airports, sleeping off drives. Quiet-hours is expected. F-scope.
- **Tab row overflow** — 3.17.

**Benchmark:** Linear notification rules. Slack notification preferences (the gold standard for density).

## 4.40 — `show-detail-active-day-sheet.png` *(new in v3)*
**Job:** Build and share the day-of-show timeline — what happens when.

**This is one of the differentiators in action.** Auto-fill from advancing data → Day Sheet PDF → share link to crew.

**Works:**
- **Timeline layout**: mono time + amber bullet + event name + × delete — scannable vertically.
- **"Auto-fill from advancing schedule?"** affordance with "3 new times to add" — beautifully concrete. This is the productized version of a message that would otherwise be "remember to update the day sheet." Surface more intelligence this way elsewhere.
- **Copy to Clipboard / Print / Export PDF / DOCX** — appropriate three-tier export. Good.
- **Quick Stats right-panel** — Advancing progress bar (7 of 7 done, green), Guest List count, Crew count, Setlist status, Finances — this is the entity-dashboard-in-a-sidebar pattern that works.

**Needs fixing:**
- **Timeline bullets all amber** — these are neutral markers, not CTAs. Migrate to muted dot or small square icon. If the design intent is "these are scheduled/committed times," use a semantic color (green for committed) paired with icon.
- **"Export PDF / DOCX"** amber text button — neutralize border, push amber to the single primary action (which is "Auto-fill").
- **"+ Add timeline item"** input row — styling is soft. Make the section border explicit so it reads as an active form, not muted chrome.
- **Auto-fill banner amber border** — OK if it's truly the primary action of the page; verify against 3.8's "no competing ambers" rule relative to Favorite star and Day Of chip.
- **No drag-to-reorder indicator** on timeline rows — times suggest order but users will want drag. At minimum show a grip icon on hover.

**Benchmark:** Artist Growth day sheets. Airtable timeline view.

## 4.41 — `show-detail-active-setlist.png` *(new in v3)*
**Job:** Build the setlist for the show. Powers the Setlist → PRO royalty export differentiator.

**Works:**
- **Three stat tiles at top** (Songs / Total Duration / Set Time) — the exact metrics a band thinks about when building a setlist. Perfect.
- **Empty state is clean**: icon + "No songs in the setlist yet" + "Add First Song" amber link CTA.
- **"+ Add Song" button** in the SETLIST card header — correct placement.

**Needs fixing:**
- **"Add First Song" amber text link** — acceptable as a single-action empty state CTA, but "+ Add Song" in the card header is already amber. Two amber CTAs competing. Make the empty-state link neutral; retain the header CTA as amber primary.
- **Set Time 20:30** — mono, right. But is this "start time" (20:30 local)? Or duration (20min 30s)? Ambiguous. Clarify with tiny label ("START TIME" vs "LENGTH") or adjacent context.
- **Songs 0 / Total Duration —** dashes for empty state are correct mono treatment.
- **No import affordance** — bands reuse setlists show-to-show. "Import from previous show" would be a one-click workflow worth a lot. Flag for Lager 2.
- **No BPM / key hints** — pro tours track tempo & key transitions between songs for flow. Not MVP but think about the data model now so Tracks table can support it.
- **Generate PRO report from setlist** — not visible on this tab. This is the differentiator feature. It should live here (right panel "Export for STIM" button) OR on the Files tab via the Generated Document pattern (see 4.43 and §1.6). Lock that placement before shipping.

**Benchmark:** setlist.fm's setlist builder. iReal Pro's setlist view.

## 4.42 — `show-detail-active-guest-list.png` *(new in v3)*
**Job:** Manage the guest list for a show — names, +ones, VIP, check-in.

**Works:**
- **Three stat tiles**: Names 8, Total Heads 14, Checked In 0 — operationally exact (names ≠ heads because of +ones). Strong.
- **All three stats in white** — correctly follows §1.5. Reference for other stat surfaces.
- **Add Guest form inline**: Name / +Ones / VIP checkbox / Add button. Compact. Good.
- **Sort: Added / A-Z / VIP** — three sensible sort modes.
- **Table columns**: Name + meta sub-line / +ONES / VIP / IN / × — covers the job.
- **Meta sub-lines under name** ("Label A&R — City Slang", "Booking agent", "Friend of band") — this is the at-door-recognition detail. Excellent.

**Needs fixing:**
- **VIP pills amber** — VIP is a neutral marker (not active state, not CTA). Migrate to `--color-chip-default` neutral OR use a dedicated "vip" tier color if you want emphasis. Amber is wrong either way.
- **IN column checkbox unchecked** — consider renaming to "CHECK-IN" so the empty state is self-explanatory. Also: checked-in guests should visually distinguish (struck-through? faded? green row?). Currently the checkbox state isn't reinforced elsewhere.
- **"Add" button** — amber. Correct as primary form action.
- **No door-mode** — at the actual door of a venue, a phone-optimized "scan name → check in" mode is the payoff. Mobile view of this page should automatically enter door-mode if a "Start Door" button is tapped.
- **+1 expansion** — +1s not shown as individual rows. Fine for now, but if a guest brings a named +1 ("James Murphy + Sarah"), that name is lost.

**Benchmark:** Eventbrite check-in. Lu.ma guest list. These guys optimize door-mode.

## 4.43 — `show-detail-active-files.png` *(new in v3)*
**Job:** Central hub for all documents tied to this show — both system-generated (riders, day sheet) and uploaded.

**This is §1.6's reference page.** Lock this pattern before it's reimplemented elsewhere inconsistently.

**Works:**
- **Three generated document cards** at top (Technical Rider / Hospitality Rider / Day Sheet) — each with icon, title, "Shared" green badge, dotted progress indicator (8/12 sections, etc.), and three-action row (Re-generate / Edit / Download).
- **Two not-generated cards** (Setlist / Settlement Sheet) — show "Not generated", dotted progress showing completeness of underlying data (1/3 sections, 3/6 sections), and a single "Generate" amber CTA.
- **UPLOADED FILES section** with Upload File + link existing — clear separation from generated docs.
- **Share state as green dot + "Shared" label** — §0.1 compliant (dot + label).

**Needs fixing:**
- **"Re-generate" button amber** on the three generated cards — too prominent. Users have already generated; re-generate is secondary. Neutral bordered. Reserve amber for the "Generate" primary CTA on not-yet-generated cards.
- **Action buttons inconsistent across states** — generated cards have three buttons; ungenerated has one. Fine semantically, but the visual weight shift is jarring. Standardize the row height so cards don't jump.
- **Section completeness dots** (e.g., 8/12 sections as dot sequence) — clever but terse. Add tooltip or hover expansion showing which sections are complete and which missing. This is the actionable info.
- **Setlist & Settlement Sheet at 3/6 and 1/3 sections** — clickable hint ("Complete Settlement to generate") would guide. Currently just a Generate button regardless of readiness.
- **Upload File vs Link Existing** — "link existing" as a small select + Link button is correct but visually tiny. Users will miss it. Equal weight or secondary bordered button.
- **No generated-but-stale state visible** — when advancing data changes AFTER Technical Rider has been generated, the rider should badge as "Source updated — re-generate recommended." This is the §1.6 stale state; build the logic.

**Benchmark:** Google Docs templates gallery + file state. Notion templates.

## 4.44 — `release-detail-tracks.png` *(new in v3)*
**Job:** Manage tracklist for a release. Powers the PRO royalty report.

**Works:**
- **"0 tracks Saved"** top-left with green Saved pill — clear state.
- **Empty state**: icon + "No tracks yet" + "Add your first track" amber link — consistent with Setlist empty state (4.41).
- **"+ Add Track"** neutral-bordered button top-right.

**Needs fixing:**
- **Two CTAs in empty state** competing — "Add your first track" text link and "+ Add Track" neutral button. Pick one treatment. Since the page is empty, the centered amber link is actually the better affordance; kill the header button until tracks exist.
- **"Mixing" status chip amber** in header — migrate to `--color-status-hold` violet (see §1.1 release-status palette).
- **No track template / import affordance** — if a track was created during Shows (setlists), it should auto-suggest. Bridge the Shows.setlist → Releases.tracklist data flow. Big product win.
- **ISRC / PRO metadata missing from UI** — tracks need writer splits, ISRC, BPM, key for the PRO-export differentiator to work. Build the fields, even if hidden under "More metadata" expander.
- **Right panel Rollout Timeline** — consistent across release-detail tabs ✓.

**Benchmark:** DistroKid track upload flow. Bandcamp release editor.

## 4.45 — `release-detail-checklist.png` *(new in v3)*
**Job:** Work through a reusable rollout checklist for this release.

**Works:**
- **Green "Saved" label** top-left — reassures autosave.
- **Empty state with three quick-start CTAs**: "+ Pre-release", "+ Release Day", "+ Post-release" — these are the three phases, pre-chunked. Excellent.
- **"Load Template"** button — anticipates that bands want reusable checklists across releases.

**Needs fixing:**
- **"Mixing" status chip amber** — same fix as everywhere.
- **Phase quick-start buttons** are neutral-bordered ✓ correct — they're secondary actions relative to "Load Template."
- **"Load Template"** is the primary action but also neutral-bordered. Promote to amber if you want bands to start there.
- **Missing: inline progress summary** — once items exist, show "4 of 12 completed · Pre-release 2/4, Release Day 1/4, Post-release 1/4" as a progress strip at top. Without this, the checklist is opaque.
- **No template library visible** — where do templates come from? Ship with 2-3 templates (single, EP, album) pre-loaded, let bands save their own.

**Benchmark:** Notion templates. Asana project templates.

## 4.46 — `release-detail-tasks.png` *(new in v3)*
**Job:** Track actionable to-dos specific to this release.

**Works:**
- **Quick add input** + "Add Task" amber button — simple, clear.
- **Task rows** show title / due date / status chip (IN PROGRESS cyan, TODO neutral) / priority chip (HIGH red, URGENT pink).
- **Priority colors** consistent with tasks-list.png ✓.
- **Two distinct tasks shown** — "Approve Nerve Drift EP artwork" (IN PROGRESS / HIGH) and "Send final master to AWAL" (TODO / URGENT). Real, specific, musician-shaped. Good seed data.

**Needs fixing:**
- **"Mixing" chip amber** — same fix.
- **Task rows lack clear tap-target hierarchy** — the entire row should be clickable to open task detail; currently it's unclear if it's interactive.
- **No assignee avatar** — even in a solo band setup, assignee column matters as soon as Julia's name or a crew member gets involved.
- **No linked files or comments indicators** on task rows — Linear shows these as small icons with counts. Add when feasible.
- **"Add Task" button state** — amber is correct as primary form action.
- **Overlaps with Checklist** — Tasks vs. Checklist is a UX divergence. Is checklist for "rollout phases" and tasks for "one-off actions"? Name that distinction explicitly in inline copy or the two surfaces will confuse users.

**Benchmark:** Linear sub-issues. Notion tasks database.

## 4.47 — `gear-detail-overview.png` *(new in v3)*
**Job:** Single gear item's core metadata — the record-of-truth for this asset.

**Works:**
- **Item header**: icon + name + subtitle ("Fender AVRI '62 Jazzmaster") + GOOD chip top-right.
- **Field stack layout** inside main card: Category / Brand / Model / Serial Number (with Copy!) / Condition / Status / Purchase Price / Purchase Date / Warranty Expires / Insurance Value / Notes.
- **Right panel**: Condition / Value / Warranty & Purchase (with "Expired 35d ago" red) / Quick Facts — all the right meta.
- **"Duplicate" button** near top — clever for bands who own multiples of one model.

**Needs fixing:**
- **GOOD chip top-right DUPLICATES the Condition right-panel card.** See 4.17 — consolidate. Pick one canonical surface for Condition.
- **Warranty expired in red** — ✓ correct use of danger, paired with "Expired 35d ago" text (§0.1 compliant).
- **Field stack is tall and narrow** — each field uses full row width for a short value. Consider a 2-column grid for the compact fields (Category / Brand, Model / Serial, Purchase Price / Purchase Date, Warranty / Insurance) to shorten the scroll.
- **Quick Facts "Guitars"** with amber dot — migrate to gear-family color per §1.1.
- **Edit button** floats between tabs row and content — awkward placement. Move to top-right of the content card or top-right of the page header.
- **"Notes: Main guitar — swapped pickups to Lollar '59"** — valuable detail, same treatment issue as contact-detail Notes: too low-weight.

**Benchmark:** Linear issue detail (item header + meta right panel is the same pattern).

## 4.48 — `gear-detail-packing.png` *(new in v3)*
**Job:** Show which packing lists this gear item appears on.

**Works:**
- **Empty state**: "This item isn't on any packing lists yet / Go to Packing Lists" amber link. Honest, direct, suggests next action.

**Needs fixing:**
- **This page is structurally underdeveloped.** When populated, what appears? Packing list names with dates? Current-tour flag? Pre-flight ready/not-ready indicator? Define the populated state before it ships with a placeholder.
- **"Go to Packing Lists"** amber text link — navigation away from this page is a valid action, but the empty state is asking the user to leave. Better: surface an inline "+ Add to packing list..." dropdown directly here.
- **Right panel consistent** with other gear-detail tabs — ✓.
- **No packing list preview** — when tours come together, a Packing tab might show: "Included in: Nerve Drift Summer 2026 [16 items]; NOT included in: Fall EU Tour." Flag-based visibility is the real job.

**Benchmark:** Snipe-IT asset → assignment history. Shopify product → "included in collections."

## 4.49 — `gear-detail-files.png` *(new in v3)*
**Job:** Attach documents to a gear item (manuals, insurance certs, receipts).

**Works:**
- **Upload File amber** + "or link existing" select + Link — clear primary + secondary affordances.
- **Empty state**: "No files linked / Attach files to keep everything organized in one place."

**Needs fixing:**
- **"link existing"** select + Link button is visually tiny compared to Upload File — see 4.43, same issue across Files tabs.
- **No pre-suggested attachments** — if the user already has `jazzmaster-insurance-2026.pdf` in their Files folder structure, suggest it inline.
- **Expected file types** — warranty, insurance cert, receipt, modification log. Could shape the upload flow with category selector on upload.
- **Right panel consistent** ✓.
- **Empty state icon** — generic file icon. Consider something more specific (paperclip or document-stack) to strengthen metaphor.

**Benchmark:** Snipe-IT asset documents. Dropbox file organization.

## 4.50 — `merch-detail.png` *(new in v3)*
**Job:** Edit an existing merch product — name, category, description, image, variants.

**Works:**
- **"Active" green pill top-right** — clear state.
- **Entity pager** (2/4, <, >, PROD-001) — consistent across entity edit pages.
- **Variants table** with columns Label / Stock / Cost / Price / Reorder / SKU + × delete — exactly the touring-band merch reality.
- **Pre-filled values** show Glass Meridian Tee: S(8), M(4), L(15), XL(15) — spotting M as low (below reorder 5) is something the UI should highlight.
- **"Save Product" primary amber full-width** + "Delete product" text link bottom — matches EntityForm pattern (§1.4).

**Needs fixing:**
- **Reorder-threshold violation not surfaced inline** — M variant has stock 4 with reorder 5. Should have a subtle warning indicator (small `--color-status-warning` dot next to the stock input). Pair with icon per §0.1.
- **"Size presets" button** in VARIANTS header — good. But it lives in the card header where "+ Add variant" also lives. Consider: Size presets as a button group OR as a select that seeds the variants table; otherwise two small actions compete.
- **No image actually rendered** — the product image slot shows a placeholder icon + "Change or remove the product image." If an image was uploaded, show it at 80×80 with change/remove below. The current state implies no image is uploaded; verify.
- **SKU fields are free-text** — no validation (duplicates, length limits, character set). Add subtle inline validation.
- **No inventory history** — view-only log ("-2 units sold 03 May at The Lexington") belongs somewhere. Not necessarily here; could be a separate tab.
- **"Delete product" at bottom as text link** — acceptable but make it neutral bordered and right-aligned to match 4.16's contact-detail fix.

**Benchmark:** Shopify product edit. Square product edit.

## 4.51 — `merch-new.png` *(new in v3)*
**Job:** Add a new merch product.

**Works:**
- **Same EntityForm pattern** as merch-detail minus the "Active" pill and with blank fields.
- **"e.g. Tour Tee 2026"** placeholder — warm, specific, musician-shaped. Good copywriting.
- **Default variant row** with "Default" label — sensible starting state, lets bands skip variant setup for simple products.
- **Subtitle**: "T-shirts, hoodies, vinyl, posters — add it and start tracking stock." — product-voice, concrete, correct.

**Needs fixing:**
- **Sidebar leak** — same as shows-new / releases-new (v2). The full-page form makes the sidebar feel leaky. Lock the form feel via modal overlay styling or commit to full-page with sidebar dimmed.
- **Category "Other" as default** — consider "Apparel" as default since tees dominate. Minor UX nudge.
- **Add Product button full-width amber** ✓ matches EntityForm.
- **Variants table empty state** — only shows Default row. When a user picks "Apparel", auto-seed S/M/L/XL rows (or offer the suggestion). Size presets button is the manual version; auto is better.
- **No SKU generation assistance** — for disciplined bands, "PW-GMT-S / M / L / XL" naming pattern is standard. Offer a formula: `[workspace]-[category]-[variant]` auto-suggestion.
- **Cost / Price in raw numbers, no currency** — inherit from settings-preferences (EUR in Jacob's case). Currently just "0" with no € prefix.

**Benchmark:** Shopify create product. Linear create issue (for form pattern).

## 4.52 — `contacts-new.png` *(new in v3)*
**Job:** Add a new contact.

**Works:**
- **EntityForm pattern** — card wrapper, required-field asterisk on Name, section divider "CONTACT INFO" as mono label.
- **Copy is musician-voiced**: "Bookers, promoters, press, crew — keep everyone in one place."
- **Notes placeholder**: "How you met, what markets they cover, preferred contact method..." — specific, prompts good hygiene.
- **Full-width amber "Add Contact"** at bottom + implicit cancel via back-link.

**Needs fixing:**
- **Role dropdown shows "Select..." with no default** — for a CRM, defaulting to the most-added role (or suggesting based on pattern) reduces friction. Low priority.
- **Sidebar leak** — same pattern problem as other new forms.
- **No "link to existing show/release" at creation** — most contacts are added in the context of a show ("James Murphy, A&R for City Slang, signed us during Nerve Drift rollout"). Consider adding "Link to" dropdown at bottom of the form so the relationship is captured at creation time.
- **Email/Phone fields are plain text inputs** — no format validation, no click-to-call / click-to-email affordances after creation. Validation is easy; add it.
- **Location split into City/Country** — reasonable. Add a lookup combobox for country to prevent typos ("DE" vs "Germany" vs "Deutschland").

**Benchmark:** Attio create contact. Notion database new-row quick input.

## 4.53 — `crew-list.png` *(new in v3)*
**Job:** Manage paid crew roster — roles, day rates, contact info.

**Works:**
- **Search bar, filter chips, +Add Crew button** — standard list page chrome.
- **Secondary meta row**: "4 crew members · 4 roles · avg $245/day" — useful aggregate. Currency mismatch (USD in label, EUR in rows) is the 3.16 bug again.
- **Table columns**: Avatar / Name / Role (with color dot) / Day Rate / Email / Phone — covers the job.
- **Alphabetical default sort** apparent.

**Needs fixing:**
- **Role color chaos** — 4 crew with 4 different hues (cyan Sound Engineer, amber Lighting Designer, purple Driver/Tech, muted Merch/Tour Aux). **Amber on Lighting Designer is Tier-1 §1.1 violation.** Migrate to crew-family palette (`--color-crew-technical` cyan for both Sound + Lighting). See new D-4 scope extension.
- **"avg $245/day"** — USD symbol; should follow workspace currency. 3.16.
- **"Sound Engineer (1)" filter chip** — pill shape competes with nav pills (3.4).
- **Email / Phone columns** — shown in muted neutral text, not amber. ✓ This is the correct treatment; contact-detail/crew-detail should match this, not the other way around.
- **No "assigned shows" count** in the table — a high-value column for booking decisions. Add: "16 shows assigned / last: Øya Festival."
- **No pay summary** — per-crew-member total YTD payout would connect crew-list to money-overview.

**Benchmark:** Linear team members list. Attio records.

## 4.54 — `crew-detail.png` *(new in v3)*
**Job:** Single crew member's profile + their show history.

**Works:**
- **Header**: avatar + Oskar Holm + Sound Engineer chip + "€350/day · oskar@soundcraft.se" meta.
- **Field stack**: Email / Phone / Role / Day Rate — compact.
- **Notes block**: "X32 + IEM specialist. Touring with band since 2023." — exactly the crew intel that matters.
- **ASSIGNED SHOWS list**: 13 rows showing date / venue / city / status chip. Strong.
- **Linked Files section** with Upload File / Link affordances.
- **"Remove crew member"** text link at bottom.

**Needs fixing:**
- **Email + Phone amber hyperlinks** — same abuse as contact-detail. Fix in D-9.
- **No right panel** — see 3.5. Crew needs a right panel with Total shows assigned, YTD payout, Avg show cost, Next assigned show. E-3 scope extension.
- **Assigned Shows list has no filter/sort** — 13 rows today; 50+ after a year. Add sort by date and filter by status.
- **No payroll tab** — showing individual payout history (date, show, amount, paid/unpaid) would close the loop from crew list → crew detail → money. Flag for Lager 2.
- **"Remove crew member"** — text link is too casual for a destructive action. Neutral bordered danger button, right-aligned.
- **No "Book for a show"** CTA — if you're on this page, the next action is often assignment. Add a right-panel button.
- **Role chip cyan** — correct per new crew-family palette. ✓

**Benchmark:** Attio person record. Roadie (crew app) profile.

## 4.55 — `docs-list.png` *(new in v3)*
**Job:** Central file repository across all entities.

**Works:**
- **Left subnav**: All Files / Favorites + FOLDERS list (Primavera Sound 20…, Nerve Drift EP, Press Kit, Admin) — hierarchical, right for files.
- **View toggle** (list/grid) — standard and useful.
- **Generate ▾ dropdown** next to Upload — brings the Generated Document pattern (§1.6) to the Files page. Smart cross-surfacing.
- **Table columns**: checkbox / PDF icon (red) / Name + "Link" sub-action / Category (pill) / Date (sortable arrow) / star / download / delete.
- **Category filter** ("All (3) / • Riders (2)") — matches releases-list pattern.

**Needs fixing:**
- **IA divergence** from rest of product — see 3.18. Decision needed: conform to top-tabs OR canonicalize "file manager" as a second layout archetype.
- **Generate ▾ dropdown** — confirm contents match the §1.6 pattern. Likely contains templates for Technical Rider / Hospitality Rider / Day Sheet etc. — but generation requires context (which show? which release?). How does this flow work without show context? Ambiguous; design it.
- **PDF icon red** — semantic for file type (PDF) but the red can read as danger/error. Use a neutral file-type badge ("PDF" mono label) instead of a red icon.
- **"Link" sub-action** under file name — small, unclear. Is this "this file is linked to an entity" or "click to get a share link"? Rename to either "Linked" badge or "Copy share link" action.
- **Folders creation UX invisible** — the "+" next to FOLDERS header is tiny. Make explicit "+ New folder" button.
- **No search result count or empty state for search** — once files grow past 30, search results state matters.

**Benchmark:** Dropbox file manager. Notion file embeds.

## 4.56 — `money-income.png` *(new in v3)*
**Job:** See all income grouped by source (shows vs other).

**Works:**
- **Three hero stats**: Show Income / Other Income / Combined — right mental model (show income is the operational core, other is the supplemental stream).
- **Show Income grouped by status** (CONFIRMED / PENDING (HOLD) / COMPLETED/SETTLED) with subtotals per group — tour-accounting shape, correct.
- **Per-row**: venue + city · date + Unpaid/Partial/Paid pill + amount.
- **Other Income section** below: merch orders, streaming royalties, sync, label advance, performance royalties — real, varied, specific.
- **"+ Log Income" amber** on Other Income header — primary action for that section.

**Needs fixing:**
- **Currency mismatch** — hero shows `$20,450.00 / $15,010.00 / $35,460.00` in USD. Rows show `€3,000`, `€1,500`, `€1,450`, etc. in EUR. New issue 3.16. **This is the most serious data-presentation bug in the product.**
- **Show Income $20,450 and Other Income $15,010 both green** — 3.2 violation; stats in white.
- **"CONFIRMED €11,700" subtotal green** — same fix.
- **"Unpaid" pills** — red; acceptable as status, pair with icon per §0.1.
- **"Partial" pills amber** — migrate to `--color-status-partial` (muted amber 40%) so it reads as "in-between" not "active/important."
- **"Paid" pills green** — ✓ correct.
- **Combined €35,460 in white** — ✓ correct, use as reference for other hero stats on this page.
- **No filter** — a band with 60+ income rows per year needs filter by status, by category, by date range.

**Benchmark:** Stripe Dashboard payouts. Xero income view.

## 4.57 — `money-band-bank.png` *(new in v3)*
**Job:** See per-band-member income/payout status.

**Works:**
- **Three stat tiles**: Earned (white) / Paid (green) / Unpaid (amber) — the three numbers that matter per member.
- **Per-member row**: avatar / name + "8 payouts" meta / right-aligned Earned/Paid/Unpaid.
- **Simple, quiet, correct shape** for a 1-member band; will scale to 4-5 members cleanly.

**Needs fixing:**
- **Unpaid €10,398 amber** — migrate to `--color-status-warning` distinct orange. §1.1 rule: amber is never a financial value color.
- **Paid €0.00 green** — arguably fine but inconsistent with "all financial values white, status pill beside" rule (3.2). Fix to white; the pill-less value speaks through the label.
- **Earned white** — ✓ reference.
- **Single member card only** — Jacob is solo for Pale Wire. For Bite Down (metalcore, multi-member), this page will be 4-5 rows. Confirm layout survives.
- **Per-member click-through** to detailed payout history missing — "8 payouts" should be a link.
- **No "Record payout" CTA** — this page will eventually be where Jacob clicks "Mark Jacob paid" after sending bank transfers. Add the action.
- **Band Bank nav pill amber active state** ✓ consistent with Money sub-nav.

**Benchmark:** Splitwise balances. Xero contacts → vendor balance.

## 4.58 — `shows-list-upcoming-mobile.png` *(new in v3)*
**Job:** Scan upcoming shows on a phone.

**Works:**
- **Upcoming (8) / Past (8) pill nav** — compact.
- **Plan Tour / + Add Show** action row — primary actions accessible.
- **Table rows** squeeze nicely to mobile: date 2-line / venue truncated / city truncated / status / prep tick / fee.
- **"€12,600 confirmed"** summary chip preserved.
- **Legend dots** (Confirmed • Day Of •) below the pills — helps decode status column.
- **TOUR filter** horizontally scrollable.

**Needs fixing:**
- **"16/10 on Free · Upgrade"** pill in red — correct escalation for over-limit, but the position next to summary chip makes it visually compete. Move to sidebar / overflow on mobile.
- **Venue truncation** — "The Lexin…" and "Pitchfork…" — cuts informative detail. Consider wrap to 2 lines or tap-to-expand row. Bandcamp Artist app does 2-line rows cleanly.
- **"Day Of" marker** — currently amber pill. Migrate to `--color-status-day-of` (#e8621f).
- **Prep column** — single ✓ mark or "?" — same inconsistency as desktop. Unify metaphor.
- **Per-row city truncation** — "Gothenb…" "Ferropol…" "Barcelo…" — same issue as venue. Truncation is preserving wrong info; consider dropping city on mobile entirely and showing on row-tap.
- **Hit-target sizing** — pills look about 32px tall. 44px minimum for WCAG AA touch targets.

**Benchmark:** Linear mobile issues list. Superhuman mobile.

## 4.59 — `money-overview-mobile.png` *(new in v3)*
**Job:** Financial overview on a phone.

**Works:**
- **Single-column stack** of hero stat cards + charts + lists — faithful to desktop structure.
- **All the pieces present**: Total Income / Expenses / Net Profit / Pending / Revenue vs Expenses chart / Expense Breakdown donut / Income Sources donut / Recent Transactions / Month over Month / Crew Costs.
- **Month-over-Month deltas** show "▼ 60% vs €11,000.00" / "▲ 24% vs €3,670.00" / "▼ 102% vs €7,700.00" — the MoM pattern with directional arrows is strong. §0.1 compliant.

**Needs fixing:**
- **Hero stats all colored** (Income green, Expenses red, Net Profit green, Pending amber) — same 3.2/3.10 fix. White with deltas only.
- **Crew Costs cyan** — "Based on day rates for crew assigned to 16 shows in this period" — useful caveat, keep; but number white.
- **Revenue vs Expenses chart** — on mobile the axis labels compress. Use abbreviated notation ("€5k" not "€5,000").
- **Two donuts stacked vertically** consume a lot of vertical space. Consider converting to small horizontal bar charts on mobile, keeping donuts for desktop.
- **Recent Transactions list** visible — good mobile treatment.
- **Pending €8,500 amber** — migrate to `--color-status-warning` consistent with other unpaid/pending indicators.

**Benchmark:** Stripe iOS dashboard (reference for mobile financial hero stats — all white).

## 4.60 — `calendar-agenda-mobile.png` *(new in v3)*
**Job:** Chronological event feed on a phone.

**Works:**
- **Month/Week/Agenda nav** with Agenda active ✓ (validates 3.15/E-7 direction).
- **Filter pill row**: Shows / Releases / Tours / Tasks / Financials — additive filters.
- **Month groups**: November 2025, December 2025, February 2026, March 2026, April 2026, May 2026, June 2026 — grouping is the right shape.
- **Per-event row** works: date chip / icon / title / sub-line / amount / status chip.
- **Linked entity sections at bottom** (SHOWS / TASKS / RELEASES summary) — unusual but useful as a quick index.

**Needs fixing:**
- **Status chip colors** — RELEASED green, CONFIRMED green, HOLD purple, SETTLED green, MASTERING amber, MIXING amber, DAY_OF amber. Same 3.7 fix (color-by-artist + icon-by-type). **Most important: migrate now before the migration hurts more.**
- **Month groups lack counts** — "March 2026" should be "March 2026 · 2 shows" on mobile where screen real estate is more precious.
- **Today button top-right** — good. But "Today" pill in red suggests alert state when it's just a navigation; neutralize.
- **Release rows** show amount (€1,500 / €4,000 / €2,500) — these are show fees presented as release fees on some rows. Verify data binding. (This may be the same 3.16 currency/data normalization bug surfacing.)
- **Bottom index (SHOWS / TASKS / RELEASES)** — duplicates above content. On mobile this feels redundant. Consider removing or promoting to a separate "Summary" tab.

**Benchmark:** Fantastical mobile. Google Calendar mobile agenda.

## 4.61 — `release-detail-overview-mobile.png` *(new in v3)*
**Job:** Release detail on a phone.

**Works:**
- **Header stack**: back / favorite / kebab / pager / REL-005 + EP cover placeholder / Nerve Drift title / EP pill + Mixing pill + "45 days to release" + "City Slang · AWAL · Post-Punk".
- **Tabs scroll horizontally** — Overview / Checklist / Tracks / Tasks (2).
- **Metadata card**: Release Date / Label / Distributor / Genre / Tracks — reads cleanly.
- **Streaming Links card with empty state** + "Add links →" amber link.
- **Notes card**: "Mixing at Studio Paradiso, Berlin. Mastering: Heba Kadry (Timber Studios, NY). Vinyl lead time 10wk." — specific, real, the kind of detail Jacob actually stores. ✓

**Needs fixing:**
- **Right panel content entirely missing.** Rollout Timeline, Contact (James Murphy), Quick Stats, Duplicate/Delete actions — all gone on mobile. **This is the most severe mobile-parity violation in the product** (see 3.13 extension). Rollout Timeline is the primary lifecycle navigator for a release; hiding it on mobile changes what the page is.
- **Fix:** Implement pattern 2 from §1.5 — collapsible Summary section as first card after the tabs, containing Rollout Timeline (compact vertical), Contact row, Quick Stats grid, Duplicate/Delete actions. F-3 scope expansion.
- **"Mixing" chip amber** — migrate to `--color-status-hold`.
- **"Add links →" amber** — neutral-bordered button.
- **Edit / Duplicate / Delete release** at bottom as tertiary text actions — acceptable structure but "Delete release" should be visually distanced from Duplicate (they are not similar-weight actions). Stack them with a gap or put Delete below with a separator.
- **Hit-target spacing on tabs** — Overview/Checklist/Tracks/Tasks(2) read tight.

**Benchmark:** Linear mobile issue detail. Bandcamp artist app release detail.

## 4.62 — `contact-detail-mobile.png` *(new in v3)*
**Job:** Contact detail on a phone.

**Works:**
- **Header**: back / favorite / kebab / pager / CON-006 + avatar + name + role chip + company meta.
- **Field cards**: Email (with Copy) / Phone (with Copy) / Company / Role / Location — clean stack.
- **Notes card**: "A&R. Signed us Sept 2024." — compact, valuable.
- **Linked Shows / Linked Releases / Linked Files** sections — consistent with desktop.
- **Linked Releases** shows 2 rows with status chips (Mixing amber, Released green) — real data, meaningful.

**Needs fixing:**
- **Email + Phone amber hyperlinks** — primary ongoing abuse. D-9 fix applies here on mobile too.
- **Role chip (Label, with amber dot)** — migrate to `--color-role-commerce` violet family dot; the chip itself should be neutral with the colored dot inside.
- **"Mixing" status chip on Linked Release** amber — migrate to `--color-status-hold`.
- **No right-panel equivalent** — since desktop contact-detail also lacks a right panel (3.5), mobile inherits the gap. E-3 fix (right-panel addition) + F-3 fix (mobile parity) will cascade here.
- **"Link existing" select + Link** in Linked Files — same tiny-affordance issue as 4.43 Files tabs.
- **Delete contact** text link at bottom — neutral bordered danger, see 4.16.
- **Copy buttons on Email/Phone** — ✓ good pattern, make sure tap-confirmation shows ("Copied!" toast).

**Benchmark:** iOS Contacts app. Superhuman contact detail.

---

# 5. Prioritized Fix Plan

Three batches, ordered by impact ÷ effort within each batch. **Estimates revised against the real 62-page basis; v2's estimates were generated against 32 pages and under-counted scope. Where item grew, it's explicit.**

## Batch D — High-impact cross-cutting fixes

Small scope, big perceptual improvement. Do these first.

### D-1 — Color discipline sweep
**Effort:** 1.5 days (was 1 day) — scope grows because 30 new surfaces need the same amber-stripping pass.
**Impact:** Every page feels more premium immediately.

- Remove amber from: email/phone links, progress bars, navigation arrows, chevrons, category dots, "Add X →" tertiary actions, financial primary values, notes/section mono labels, status pills that aren't Day Of, "Bring X" action hints, countdown text, VIP pills, timeline bullets, release-status chips (Mixing etc.), DEFAULT chips, ADMIN chips, hyperlink VALUES in settings-links.
- Replace with: white primary text, `--color-fg-muted` for tertiary, status-specific colors for status, muted 40% amber for progress bars.
- Audit every `color:` declaration referencing `--color-accent` against §1.1 rules.

**Touches:** every page, now confirmed 62 of them.

### D-2 — Financial numbers → white
**Effort:** 0.75 day (was 0.5 day) — grows with `money-income.png` and `money-band-bank.png` additions.
**Impact:** Money pages immediately read as serious financial tool.

- All v2 targets plus:
- `money-income.png` Show Income / Other Income / Combined → white
- `money-income.png` CONFIRMED / PENDING / SETTLED subtotals → white
- `money-band-bank.png` Earned (already white) ✓, Paid → white (with green pill beside), Unpaid → white (with warning pill beside)
- `money-overview-mobile.png` hero row (same fix as desktop but verify mobile inherits cleanly)

All colored deltas must pair with directional indicator (§0.1).

### D-3 — Status palette unification (with distinct Day Of hue)
**Effort:** 0.75 day (was 0.5 day) — adds release-status palette work.
**Impact:** Calendar becomes readable. Show list + task list + release list speak same color language.

- Define tokens from §2.2: `--color-status-hold`, `--color-status-confirmed`, `--color-status-day-of`, `--color-status-settled`, `--color-status-warning`, `--color-status-frozen`.
- **NEW (v3):** `--color-status-mastering`, `--color-status-partial`.
- Apply to: shows-list, dashboard pipeline, calendar event pills (coordinating with D-7), release status chips, task column headers, settlement FROZEN/SETTLED, Unpaid/Partial/Paid financial status (money-income, money-band-bank), Mixing/Mastering chips (release-detail all tabs).
- Every status pill pairs with text label.

### D-4 — Role color family compression
**Effort:** 0.75 day (was 0.5 day) — now covers **crew-list and crew-detail in addition to contacts.**
**Impact:** Contacts + Crew pages stop feeling chaotic.

- Map 9 contact roles to 4 families (§1.1).
- **NEW (v3):** map 4+ crew roles to 3 families (`--color-crew-technical`, `--color-crew-logistics`, `--color-crew-commercial`).
- Same treatment for gear categories (6 → 3 families).
- Update role badges, dots, filter pills across contacts-list, contacts-new, contact-detail, crew-list, crew-detail, gear-list, gear-detail. Every color appears with a text label.

### D-5 — Upgrade banner normalization
**Effort:** 0.25 day (unchanged)
**Impact:** Plan-gating stops feeling like a threat.

- One `<UpgradeBanner>` component, muted-amber treatment, 56px height.
- Applies to: releases-list, merch-list, tasks-list, gear-list, settings-subscription (usage-over-limit variant), money-budgets.

### D-6 — Brand color default + preset picker + copy cleanup
**Effort:** 0.5 day (unchanged)
**Impact:** Prevents brand color clashing with UI chrome; fixes marketing typo.

- Default brand color: `#f5f5f5` (neutral white) or a curated safe pick.
- Preset picker: 12 curated colors + custom hex, with explicit warning against ArtistHQ amber.
- Copy: "Used on generated documents (riders, EPK). Don't pick ArtistHQ amber — your docs will disappear into the UI."
- Marketing pricing: "Best Popular" → "Most Popular" (2-minute fix).
- Marketing pricing: label annual billing explicitly ("$12.50/mo when billed annually — $150/yr").

### D-7 — Calendar color-by-artist architecture ⚠️ GATES E-CALENDAR WORK
**Effort:** 1 day (unchanged)
**Impact:** Prevents tearing up Batch E calendar polish in Batch F.

**This is the irreversible decision (§0.2 territory). Make it now.**

- **Color encodes source:** `events.color` pulled from `artists.brand_color` (joined via `events.artist_id`).
- **Icon encodes type:** show / release / tour / task get distinct icons.
- **Orphan/system events** (no artist, imported, auto-generated): `--color-chart-neutral` muted gray.
- **Brand color guardrails:** if an artist's brand_color is too close to ArtistHQ amber (ΔE threshold against #f97316), system auto-shifts to nearest curated palette slot for calendar display (does not modify stored brand_color, just render-time adjustment).
- **FAVORITES sidebar becomes the color legend** implicitly.

Touches: every calendar view (`calendar-month`, `calendar-week`, `calendar-agenda`, `calendar-agenda-mobile`), tour span pills, mini-cal dots.

**Dependency:** E-7 (agenda default + Week hide) cannot ship until D-7 lands.

**Dependency on E-13 data:** E-13 must be live and collecting data for **at least 2 weeks** before D-7 ships. D-7's color-by-artist architecture needs orphan-event frequency data to validate the `--color-chart-neutral` fallback is correctly scoped. If orphans are rare (<5% of events), the fallback works as designed. If orphans are common (>20%), the fallback can't carry the load and a different approach is needed (e.g., per-orphan-category neutral variants, or forcing `artist_id` assignment at event creation). Shipping D-7 without this data is a guess.

**Edge case flagged (not executed):** Multi-artist workspace cross-membership. D-7 assumes single-artist-per-user-view. When Pro-tier multi-artist ships, edge cases arise: users in multiple workspaces seeing calendar events from both (legend disambiguation); tours spanning multiple artists not modeled by single `tour.artist_id`; team members with scoped access to some artists needing scoped filters. Flagged for future **F-14** when multi-artist Pro ships. Not blocking D-7 execution now.

### D-8 — Accessibility verification pass
**Effort:** 2 days verification + 0.5–1 day remediation (was 0.75 day in initial v3 draft — that estimate was under-scoped).
**Impact:** Ensures §0.1 is actually enforced, not just declared.

**Why the upward revision:** 62 pages × 3 colorblindness simulations + status-chip text-label check + chart shape-marker verification + alert icon presence + focus-state high-contrast across dark mode is 1.5–2 days minimum *if done thoroughly*. Remediation for anything failing is on top of that. Plan against 2.5–3 days total. Under-scoping accessibility at verification is the classic trap; this correction refuses it.

Verification checklist after D-1 through D-7 land, now including the 30 v3 pages:

- Colorblind simulation across every status surface (deuteranopia, protanopia, tritanopia).
- WCAG AA contrast: 4.5:1 minimum for body text, 3:1 for UI elements.
- Every status dot has adjacent text label.
- Every chart line/bar uses shape or pattern marker, not hue alone.
- Every alert has an icon.
- Focus states visible in high-contrast mode.
- **NEW (v3):** toggle off-state visually distinguishable from disabled state.
- **NEW (v3):** mobile touch targets ≥44px (per iOS HIG / Material).

### D-9 — Hyperlink cleanup (hived off D-1 for tracking) *(new in v3)*
**Effort:** 0.5 day
**Impact:** Kills the single worst amber-abuse pattern in the product.

- Migrate email/phone/URL hyperlink rendering across: `contact-detail`, `contact-detail-mobile`, `crew-detail`, `settings-links` (10+ values on one page).
- Rule: hyperlink values render as `--color-fg` with `--color-info` only on hover underline. The "Copy" button adjacent is the amber affordance.
- Ship a shared `<CopyableValue>` primitive so the pattern doesn't drift across surfaces.
- This was buried in D-1's scope in v2; hiving off because it's concentrated enough to track discretely and because settings-links alone is a 10+ amber instance single-page violation.

### D-10 — Analytics instrumentation + onboarding screen capture *(new in v3.1, labeled D-10 because scheduled in Batch D)*
**Effort:** 0.75 day
**Impact:** Unblocks data-dependent decisions in D-7, E-7, F-8 — and unblocks F-13 onboarding audit.

**Priority:** Ship early in Batch D sequence so the 2-week data-collection window for D-7 starts as soon as possible.

Without instrumentation, three downstream decisions are blind:
- **D-7 calendar architecture:** how often do events land without `artist_id` (orphan frequency)?
- **E-7 Week view deletion:** are Week-view `?view=week` URL hits zero, or non-trivial?
- **F-8 command palette:** which shortcuts actually get used?

**Implementation:**
- Vercel Analytics for page views + route performance (free on current plan).
- PostHog (self-hosted) or Umami for event tracking. PostHog wins if you want funnel analysis later.
- Lightweight event-firing wrapper so swapping providers is cheap.
- Respect GDPR / Do Not Track defaults from day one.

**Bundled onboarding screen capture (unblocks F-13):**
- Since D-10 is already touching infra/secret configuration, re-run `npm run audit:capture:onboarding` with `SUPABASE_SERVICE_ROLE_KEY` configured.
- Produces 4 onboarding screenshots × 2 viewports = 8 PNGs.
- Commit to `design-audit` branch + sync to public mirror.
- Unblocks F-13 from its current indefinite-backlog state.

**Slots into Batch D sequence after D-1** so the color-discipline sweep isn't dependent on event data, but **before D-7** with enough lead time for the 2-week collection window.

### Batch D execution sequence *(v3.1)*

Reflects priority sequencing + disambiguated pre-beta criteria + D-10's D-7 dependency:

1. **E-6** settlement error boundary — **PRE-BETA BLOCKER: blocks ALL external interaction.** Do first.
2. **D-6** brand color default + preset picker + marketing typo — blocks unsupervised PDF generation (before real beta).
3. **D-10** analytics instrumentation + onboarding capture — starts the 2-week data-collection clock for D-7.
4. **D-1** color discipline sweep — the big cross-cutting one.
5. **D-9** hyperlink cleanup — focused follow-up to D-1.
6. **D-2** financial numbers → white.
7. **D-3** status palette unification + Day Of `#e8621f`.
8. **D-4** role family compression (contacts + crew + gear in one cascading commit).
9. **D-5** upgrade banner normalization.
10. **D-7** calendar color-by-artist architecture (requires ≥2 weeks of D-10 data — if not ready, defer).
11. **D-8** accessibility verification (2 days verify + 0.5–1 day remediate).

**D-7 must land completely before E-7 touches calendar UI. No parallel execution on calendar work.**

If D-7 readiness is blocked on D-10 data collection, **D-8 can run first** — accessibility verification doesn't require calendar architecture to be finalized. Verify D-1 through D-5 + D-9, remediate, then circle back to D-7 once data is in.

## Batch E — Medium-scope polish

Component-level consistency work. After Batch D lands.

### E-1 — Filter vs nav pill disambiguation
**Effort:** 1.5 days (was 1 day) — add crew-list, calendar-agenda-mobile, money-income to migrations.
**Impact:** Every page with filters stops having visual pill collision.

- Introduce `<FilterControl>` component with two variants: `segmented` and `chip`.
- Migrate: money (all subpages including income, band-bank), shows-list, shows-list-mobile, releases-list, tasks-list, contacts-list, crew-list, calendar-agenda, calendar-agenda-mobile.

### E-2 — Typography scale rollout
**Effort:** 2.5 days (was 2 days) — add 30 new pages to rollout scope.
**Impact:** Dramatic hierarchy improvement across product.

- Define tokens per §1.2.
- Migrate all entity page titles → 32px Syne.
- Migrate all stat hero numbers → 40px Syne on right panels, 24px on dashboard cards.
- Migrate all section header labels → 11px mono.
- Key pages: v2 32 + 30 v3 pages.

### E-3 — Right-panel extension to Contact AND Crew detail *(v3 scope growth)*
**Effort:** 1.5 days (was 1 day) — now covers crew-detail as well.
**Impact:** Contacts + Crew gain parity with Show/Release/Gear detail.

- Build right-panel for contact-detail: avatar block, quick stats (shows linked, releases linked, last contact), recent activity, Call/Email/Copy quick actions.
- **NEW (v3):** Build right-panel for crew-detail: Total shows assigned, YTD payout, Avg show cost, Next assigned show, Call/Email/Book quick actions.

### E-4 — Stat card redesign (Dashboard + Money overview + Band Bank) *(v3 scope growth)*
**Effort:** 1 day (unchanged)
**Impact:** Hero row communicates state, not just numbers.

- Hero stats: bigger number (32–40px), white, muted label above, delta below with arrow + color.
- Apply to dashboard hero row, money-overview hero row, **money-income hero row (new)**, **money-band-bank stats (new)**, guest-list stats (already correct — reference).

### E-5 — Empty state audit
**Effort:** 0.75 day (was 0.5 day) — 30 new pages add empty-state surfaces.
**Impact:** Empty states stop feeling apologetic.

- Audit every empty state across 62 screenshots against primitive.
- Checks: correct variant, consistent copy pattern, CTA presence when actionable, icon restraint.
- New surfaces: gear-detail-packing, gear-detail-files, release-detail-tracks/checklist/tasks empty states, show-detail-setlist empty state, settings-team empty states.

### E-6 — Settlement error boundary ⚠️ PRE-BETA BLOCKER
**Effort:** 0.5 day (unchanged)
**Impact:** No more Postgres errors leaking to UI.

- Wrap Supabase queries in error boundary.
- Generic fallback UI with "Retry" + optional "Report" link.
- Promote ahead of other E items before any outside user sees the product.

### E-7 — Calendar: Agenda as default, Week view hidden
**Effort:** 0.5 day (unchanged)
**Impact:** Calendar becomes useful for the audience.

**Depends on D-7 landing first.**

- Agenda is the default calendar view, both desktop and mobile (confirmed aligned with calendar-agenda-mobile's current prominence).
- Week view: remove from tab row, keep route reachable via `?view=week` URL param (§0.2 reversibility).
- Add analytics event on Week view visits.

### E-8 — Files page IA reconciliation *(new in v3)*
**Effort:** 1 day
**Impact:** Settles the files-specific navigation pattern or conforms it to product defaults.

**Scope:** Decide 3.18 — conform Files to top-tabs pattern OR canonicalize "file manager" archetype.

- **Recommendation:** canonicalize. Document the "file manager" archetype in §1.4 extension: left-subnav + folders allowed when content is hierarchically structured.
- Confirm Generate ▾ dropdown on docs-list wires into §1.6 pattern consistently.
- Fix PDF icon red → neutral PDF badge.
- Clarify "Link" sub-action copy.
- Make "+ New folder" explicit.

### E-9a — Currency architecture 1-pager *(v3.1 prerequisite to E-9)*
**Effort:** 0.5 day
**Impact:** Prevents E-9 from executing against hidden assumptions.

Before E-9 (rendering) can execute, produce a short backend-design doc answering:
- What currency is stored per entity? (show fees, expenses, crew day rates)
- When is FX conversion applied — on storage, on display, both?
- What's the authoritative rate source — transaction-date spot rate frozen on entry, or live current rate?
- Per-entity display vs per-workspace display — which wins?
- How are multi-currency aggregates presented — single converted total, or per-currency subtotals with no combined row?

This is a half-day of design thinking that controls what E-9 execution actually looks like. Skipping it means E-9 executes against hidden assumptions.

### E-9 — Currency / locale contract enforcement *(new in v3)*
**Effort:** 1 day (plus backend work)
**Impact:** Resolves 3.16's USD/EUR mismatch; makes settings-preferences actually load-bearing.

**This is mostly tech debt but presents as a design problem.**

- All currency rendering flows through a single formatter that reads `workspace.currency` from settings-preferences.
- Ban literal `$` / `€` in templates.
- Money-income hero, money-band-bank stats, crew-list "avg $245/day", shows-list summary chips — all audit against contract.
- **Scope caveat:** if FX conversion is needed (band earned in multiple currencies on different shows), this grows to a bigger project. For MVP, show source currency per row and disable any "combined total" that crosses currencies; display per-currency subtotals instead.

### E-10 — Settings tab IA fix *(new in v3)*
**Effort:** 1 day
**Impact:** Resolves 3.17 tab wrap. Settings stops looking broken.

**Ship Option 3 (stopgap) in Batch E; reserve Option 1 (left sidebar layout) for Batch F.**

- Collapse Settings tab row to 7 visible + "More ▾" overflow (contains Export + Account).
- F-scope: migrate Settings to its own left-nav layout archetype.

### E-11 — Generated Document pattern canonicalization *(new in v3, rescoped in v3.1)*
**Effort:** 0.5 day (refinement of show-detail-files only)
**Impact:** The §1.6 pattern gets polished on its reference implementation.

- Refine show-detail-files treatment:
  - "Re-generate" neutralized on already-generated cards.
  - Section-completeness dots get tooltip.
  - Stale state shipped when underlying data changes post-generation.
- Wire Generate ▾ dropdown on docs-list to respect context (which show? which release?).

**Note on scope (v3.1):** v3 had E-11 cover both refinement AND propagation to Release detail Files (3–5 days total). That was optimistic — Release Files need distinct templates (EPK, Press Kit, Liner Notes), different data sources, different export formats, different brand-color application surfaces. The propagation work has been split off as **F-5a**, bundled under the F-5 brand color system where it belongs.

### E-12 — Plan-gating UX consistency pass *(new in v3.1)*
**Effort:** 0.5 day
**Impact:** Unifies three currently-divergent over-limit treatments.

**Problem:** `shows-list` shows "16/10 on Free · Upgrade" as a small red pill. `money-budgets` shows a full-page upgrade CTA. `settings-subscription` shows red progress bars at per-resource granularity. Three surfaces, three treatments, no shared primitive.

**Rule:**
- **At-limit** (approaching threshold): muted warning-amber progress, neutral bar, small "X/Y" label inline.
- **Over-limit** (actively blocking a create flow): D-5 `<UpgradeBanner>` with clear upgrade path, but doesn't block unrelated surfaces.
- **Locked feature** (plan-gated page): D-5 `<UpgradeBanner>` primitive, single treatment.

Scope: shows-list capacity strip, money-budgets, settings-subscription usage grid, releases/merch/tasks/gear upgrade banners. One pass.

## Batch F — Larger-scope structural work

Structural improvements. Do last, with more care.

### F-1 — Landing page rebuild
**Effort:** 3–5 days (unchanged)
**Impact:** Make the front door as crafted as the product.

- Full-bleed product hero screenshot with glass frame + subtle shadow + scroll-reveal animation.
- Three-differentiator section with animated mini-demos.
- Bite Down case study section.
- Compare-vs-Muzeek table (optional).
- Pricing section refreshed to match in-app.
- Footer with community links.

### F-2 — Ambient gradient / atmosphere layer
**Effort:** 1 day
**Impact:** Product feels inhabited, warmer.

### F-3 — Mobile right-panel surfacing *(v3 scope growth)*
**Effort:** 3 days (was 2 days)
**Impact:** Mobile detail views regain information density.

- On show-detail-mobile, release-detail-mobile, contact-detail-mobile, crew-detail-mobile, gear-detail-mobile: implement one of three patterns from §1.5:
  - Summary strip above tabs
  - Collapsible Summary section as first card
  - Summary tab
- **release-detail-overview-mobile is the priority** — its Rollout Timeline loss is the most severe parity gap.

### F-4 — Advancing sheet collapsible sections + sticky TOC
**Effort:** 1 day
**Impact:** Long advancing pages become navigable.

### F-5 — Generated docs (riders, EPK) brand color system
**Effort:** 2 days
**Impact:** Band's branded PDFs feel premium.

Depends on D-6 brand color defaults AND E-11 pattern canonicalization.

### F-5a — Release detail Files: generated document propagation *(new in v3.1, split from E-11)*
**Effort:** 3–5 days
**Impact:** Release Files tab gains the generated-doc pattern for EPK, Press Kit, Liner Notes.

Bundled under F-5 brand color system because Release Files need distinct templates, different data sources, different export formats, and different brand-color application surfaces than Show Files. Honest effort.

- Build EPK generator: release metadata + bio + cover art + links → PDF
- Build Press Kit generator: EPK content + high-res artwork + selected press quotes → PDF
- Build Liner Notes generator: tracklist + credits + thanks → PDF
- Apply §1.6 Generated Document micro-interface consistently across all three.
- Reserve Gear detail → Files as upload-only (no system-generated docs applicable).

### F-6 — Day Of motion treatment (optional refinement)
**Effort:** 1 day (was 0.5 day in initial v3 draft — motion work always takes longer than initially scoped)
**Impact:** Refinement if #e8621f adjacency feels off.

Motion iteration to taste (ring-pulse vs inner-glow, duration, easing) + validation across status-chip backgrounds + `prefers-reduced-motion` wiring takes a full focused day honestly.

### F-7 — Loading states
**Effort:** 2 days
**Impact:** Loading is where design quality reveals itself.

**Pairs with F-9 (motion tokens).**

### F-8 — Command palette + keyboard shortcuts
**Effort:** 1 week
**Impact:** 80% of the value of Linear/Superhuman muscle memory.

### F-9 — Motion language
**Effort:** 1 day
**Impact:** Tokens defined, then applied systematically.

### F-10 — `<EntityForm>` component extraction *(new in v3)*
**Effort:** 1.5 days
**Impact:** Stops re-deciding form layout page by page.

- Build `<EntityForm>` primitive encoding the §1.4 form template: card wrapper, required-asterisk, mono section-break labels, full-width amber CTA, destructive link below.
- Migrate: shows-new, shows-edit, releases-new, releases-edit, merch-new, merch-detail (edit), contacts-new, contact-edit (when shipped), crew-new (when shipped), gear-new (when shipped).
- Kills the sidebar-leak issue on shows-new / releases-new by making the component explicit about its container treatment.

### F-11 — Settings left-sidebar layout (Option 1 from 3.17) *(new in v3)*
**Effort:** 1.5 days
**Impact:** Permanent fix for Settings tab overflow.

- Migrate Settings from horizontal tabs to left-sidebar nav layout.
- E-10 overflow-dropdown stopgap can be retired.
- Nav groups: Profile + Links + Account (Identity) / Preferences + Notifications (Workspace) / Subscription + Export (Data) / Team + Band (Collaboration).

### F-12 — Generated Document stale-state engine *(new in v3)*
**Effort:** 1 day
**Impact:** Generated docs accurately reflect source-data freshness.

- When advancing schedule changes after a Technical Rider has been generated, the card surfaces "Re-generate recommended" via §1.6 stale state.
- Timestamps: `document.source_data_hash` vs `document.generated_at`.
- Applies across all Generated Document surfaces.

### F-13 — Onboarding flow audit + redesign *(new in v3.1)*
**Effort:** 2–3 days
**Impact:** Onboarding is the single most important conversion surface and was not in the v3 audit scope (service-role key capture gap). Unblocked by D-10 completion.

**Dependency:** D-10 completion (which includes the onboarding screen capture via `npm run audit:capture:onboarding`).

Once the onboarding steps are captured:
- Apply D-1/D-2/D-3/D-9 color discipline.
- Apply E-2 typography scale.
- Verify §0.3 mobile parity on every step (onboarding on a phone is the actual first-contact surface for most users).
- Verify §1.4 form template consistency (use `<EntityForm>` where applicable once F-10 lands).
- Audit skippable steps for friction-without-insight.
- Validate final step → first-run empty states on dashboard / shows / releases.

### F-14 — Multi-artist workspace support (flagged, not scheduled)
When Pro-tier multi-artist ships, revisit D-7 calendar architecture and §0.3 mobile parity for cross-artist views. See D-7 edge case note for specifics. Not blocking current execution; flagged so the extension path is visible.

---

# 6. Scope Realism

The "days" estimates above are focused-work days, not calendar days.

Solo + Claude Code + full-time climate job + active band = roughly 10–15 hours of focused design-execution time per week, optimistically.

**v3.1 consolidated totals:**

| Batch | Focused days | Calendar weeks |
|---|---|---|
| **D** (11 items including D-10 + revised D-8) | ~8.75 | 4–5 |
| **E** (12 items including E-9a + E-12) | ~11 | 7–9 |
| **F** (14 items including F-5a, F-13, F-14 flag) | ~4.5–5.5 weeks | 14–18 |

**Drivers of growth from v3 → v3.1:** D-8 realism (+1.5–2 days), E-9a prerequisite (+0.5), E-12 (+0.5), D-10 analytics+onboarding (+0.75), F-5a propagation honest scope (+3–5 days within F-5), F-6 honesty (+0.5), F-13 onboarding redesign (+2–3 days).

This is honest. Scope grew because 30 new pages revealed 30 new instances of cascading patterns, and v3.1 corrected several v3 under-estimates. Plan against v3.1, not v3. Don't feel behind schedule when Batch E takes longer than the E-4 estimate suggests — the estimate is for focused hours, not elapsed hours.

**Priority sequencing (v3.1):**

See the **Batch D execution sequence** subsection in §5 above for the canonical order. Broader sequencing principles:

1. **E-6** (settlement error boundary) happens before any external user interaction. See §6.1 pre-beta blocker criteria below.
2. **D-6** (brand color default) happens before real beta launch. See §6.1.
3. **D-10** (analytics + onboarding capture) happens early in Batch D so the 2-week data collection window for D-7 starts ASAP.
4. **D-7** (calendar architecture) before any E-calendar work. Don't run in parallel. Blocked on ≥2 weeks of D-10 data.
5. **D-1 through D-5, D-9** can run in any order between D-10 and D-7.
6. **D-8** (accessibility verification) at the end of Batch D — can run in parallel with or before D-7 if D-7 blocked on data.
7. **E-9a → E-9** (currency architecture 1-pager then rendering). E-9 pairs with backend work; flag early so scheduling aligns.
8. **E-10** (settings tab stopgap) before any settings-heavy user touches the product.
9. **E-11** (generated-doc pattern refinement) before F-5 brand color system. F-5a propagation is part of F-5 now.
10. **E-12** (plan-gating consistency) anywhere in Batch E.
11. **F-7 and F-9** pair together at the end.
12. **F-11** (settings left-sidebar layout) completes the E-10 stopgap.
13. **F-13** (onboarding redesign) requires D-10 complete.

## 6.1 Pre-beta blocker criteria *(v3.1 clarification)*

v3 and earlier language conflated two different failure modes under "pre-beta blocker." Disambiguated:

**E-6 (settlement error boundary) — blocks ALL external user interaction.** Raw Postgres errors leaking to the UI is trust-destroying. No beta signup, no advisor demo, no waitlist test email until E-6 lands. The failure mode (permanent loss of user confidence on first encounter) is not recoverable.

**D-6 (brand color default + preset picker) — blocks external users who will generate PDFs unsupervised.** Realistically every user eventually generates a rider or EPK, but the failure mode (user picks `#f97316` as brand color, generated PDF merges visually with ArtistHQ UI when previewed in-app) is narrower and aesthetics-degrading rather than trust-destroying. If D-6 isn't landed, you can still demo the product to an advisor on staging; flag the brand-color picker as known work and move on.

**Practical effect on sequencing:** E-6 is non-negotiable before any external touch. D-6 is non-negotiable before any external user is expected to generate docs unsupervised (i.e., before real beta — but not before advisor demos or closed waitlist tests).

---

# Appendix: What You Got Right

For balance, page-by-page highlights worth preserving through these batches. v3 extends v2's list with findings from the 30 new pages.

**From v2 (32-page basis):**

1. **`show-detail-active-advancing.png`** — density matches Artist Growth. Don't dilute.
2. **`show-detail-settled-settlement.png`** — settlement workflow is the most mature page in the product.
3. **`merch-list.png` Show Merch Planner** — "Bring 6 for Øya" is a real tour-operations feature no competitor ships.
4. **`gear-list.png` ATA Carnet button** — niche-correct. Touring bands dealing with EU/UK customs will notice.
5. **Right panel on show-detail / release-detail / gear-detail** — structure is right, just extend to contact-detail and crew-detail.
6. **Tour Profitability section on `money-reports.png`** — band-accountant view, done well.
7. **Calendar agenda** is the strongest calendar view. Lead with it.
8. **Landing tagline** — keep.
9. **JetBrains Mono on times / capacity / power specs** in advancing — exactly the right semantic use.
10. **PROVIDED / NEEDED backline badges** — clear, scannable, operationally useful.

**New in v3 (30-page basis):**

11. **`show-detail-active-files.png` Generated Document micro-interface** — the four-state pattern (not-generated / generated-not-shared / generated-shared / stale) is more sophisticated than most SaaS doc-management. Canonicalize before it drifts.
12. **`show-detail-active-day-sheet.png` auto-fill affordance** — "3 new times to add" as a productized nudge is the right cognitive level: not a notification, not a dumb button, a contextual suggestion. Extend this pattern.
13. **`show-detail-active-guest-list.png` three stats (Names / Total Heads / Checked In)** — captures the actual complexity of guest-list reality (names ≠ heads). All stats in white already. Use as reference for other stat rows.
14. **`settings-preferences.png` 2x2 dropdown grid** — quietest, cleanest page in Settings. Template for the other simpler Settings surfaces.
15. **`settings-notifications.png` grouped toggle categories** — the two-tier master + sub-toggle pattern is correctly scoped. Build on top with channels and priority markers in Batch E.
16. **`crew-detail.png` Notes copy** — "X32 + IEM specialist. Touring with band since 2023." — this is exactly the intel that Muzeek doesn't capture. The data field is correct; just let the amber hyperlinks go.
17. **`money-income.png` grouping by status** (CONFIRMED / PENDING / SETTLED) — tour-accounting-correct shape once currency bug is fixed.
18. **`contacts-new.png` / `merch-new.png` form pattern convergence** — the EntityForm template is emerging organically. Extract it in F-10.
19. **`release-detail-checklist.png` three-phase quick-start** (Pre-release / Release Day / Post-release) — maps to the actual rollout mental model. Strong.
20. **`docs-list.png` Generate ▾ dropdown** — cross-surfacing the §1.6 pattern to the files page is smart; it means generated docs aren't trapped inside entity detail pages.

---

# Changelog

**Date:** 18 April 2026 (v3.1 consolidated)
**Delta from v2:** 30 new pages added to per-page review, totaling 62 sections. Amendments to principles and tokens driven by specific new-page findings. v3.1 execution-readiness corrections folded inline throughout.

## Principles amended (v3)

- **§0.3 NEW — Mobile parity is structural, not stylistic.** Added as third meta-principle after `release-detail-overview-mobile.png` confirmed v2's 3.13 (show-detail-mobile drops right-panel) was systemic, not show-specific. Release rollout timeline disappearing on mobile is the most severe parity violation in the product.
- **§1.1 amended — Amber "appears on" list gains item 6: toggle switches in on state.** `settings-notifications.png` has 17 enabled toggles simultaneously; naming this carveout explicitly prevents future drift.
- **§1.1 amended — Amber "does NOT appear on" list gains: marker chips for default/primary (`settings-band.png` DEFAULT chip), link VALUES in forms (`settings-links.png`), and release status chips (Mixing/Mastering across release-detail tabs).**
- **§1.1 NEW — Release status palette** (Writing/Recording/Mixing/Mastering/Released/Archived) added to distinguish release lifecycle from show lifecycle. `release-detail-*` surfaces currently put Mixing in amber; migrate to `--color-status-hold` violet.
- **§1.1 amended — Role color families extended to crew.** `crew-list.png` revealed the same 4-hue chaos as contacts-list, with Lighting Designer in amber (Tier-1 violation). Crew families: Technical / Logistics / Commercial.
- **§1.4 NEW — Form template pattern.** `merch-new.png`, `merch-detail.png`, `contacts-new.png` converge on a consistent pattern. Named `<EntityForm>`, scheduled in F-10.
- **§1.5 amended — Mobile parity rule.** When a desktop layout uses a right-panel, mobile must carry equivalent information via one of three patterns. Codified against §0.3.
- **§1.6 NEW — Generated document micro-interface.** Documented after `show-detail-active-files.png` revealed a four-state pattern not covered in v2. Propagation in E-11 + F-5a, stale-state engine in F-12.

## Principles clarified (v3.1)

- **Principle precedence** rewritten to distinguish typological framing (what each principle governs) from deterministic hierarchical framing (what wins when two conflict). Hierarchy: Accessibility > Mobile parity > Typography consistency > Color discipline > Pattern purity. See "Principle precedence" section above.
- **§6.1 NEW — Pre-beta blocker criteria** disambiguates E-6 (blocks ALL external interaction) from D-6 (blocks unsupervised PDF generation). Previously conflated under single "pre-beta blocker" label.

## Tokens added (v3)

- `--color-status-mastering: #6b5eb0` — release lifecycle state between Mixing and Released.
- `--color-status-partial: rgba(249, 115, 22, 0.4)` — partial payment status (Unpaid/Partial/Paid triad on money-income).
- `--color-crew-technical / --color-crew-logistics / --color-crew-commercial` — crew role families.
- `--color-chip-default-bg / --color-chip-default-fg / --color-chip-default-border` — neutral chip tokens for DEFAULT / PRIMARY / ADMIN markers.
- `--panel-mobile-summary-*` — tokens for the §1.5 mobile summary pattern.
- `--doc-*` — tokens for the §1.6 Generated Document micro-interface.

## Tokens clarified (v3.1)

- **Contrast notation discipline** added to `--color-status-day-of` and extended as standard for `--color-status-hold`, `--color-status-mastering`, `--color-status-warning`, and `--color-role-*` / `--color-crew-*` families. D-8 measures and records actual ratios; tokens that fail get revised before D-8 closes.

## Cross-cutting issues added (v3)

- **3.16** — Money Income ships in two currencies simultaneously. Presentation bug with design consequences.
- **3.17** — Settings tab row overflows to two lines on every settings surface.
- **3.18** — Files page IA divergence from rest of product.

Also: **3.1** and **3.13** extended with v3 surface lists — not renumbered, but expanded.

## Batch plan revisions (v3)

- **D-1 through D-5, D-8 grew** by 0.25–0.5 day each due to 30 new surfaces to touch.
- **D-9 NEW** — hyperlink cleanup hived off D-1 for tracking.
- **E-3 grew** to cover crew-detail right-panel in addition to contact-detail.
- **E-4 grew** to cover money-income, money-band-bank hero stats.
- **E-8 NEW** — Files page IA reconciliation (3.18).
- **E-9 NEW** — Currency/locale contract enforcement (3.16).
- **E-10 NEW** — Settings tab IA stopgap (3.17).
- **E-11 NEW** — Generated document pattern canonicalization (§1.6).
- **F-3 grew** to cover release-detail-mobile, contact-detail-mobile, crew-detail-mobile.
- **F-10 NEW** — `<EntityForm>` component extraction.
- **F-11 NEW** — Settings left-sidebar layout (permanent fix for 3.17).
- **F-12 NEW** — Generated document stale-state engine.

## Batch plan revisions (v3.1)

- **D-8 effort corrected** from 0.75 day to 2 days verification + 0.5–1 day remediation. v3 estimate was under-scoped for the realistic 62-page × 3-colorblindness-simulation matrix.
- **D-10 NEW** — Analytics instrumentation + onboarding screen capture. Scheduled in Batch D because D-7 needs ≥2 weeks of orphan-event data before it can ship. Also unblocks F-13.
- **D-7 edge case flagged** — Multi-artist workspace cross-membership. Not in current scope; flagged for future F-14.
- **Batch D execution sequence** — Explicit 11-step ordering added at end of §5 Batch D section.
- **E-9a NEW** — Currency architecture 1-pager prerequisite before E-9. Prevents E-9 from executing against hidden backend assumptions.
- **E-11 rescoped** — refinement only (0.5 day). Propagation to Release detail Files split off as F-5a because Release Files need distinct templates, data sources, and export formats.
- **E-12 NEW** — Plan-gating UX consistency pass. Unifies three currently-divergent over-limit treatments (shows-list capacity pill, money-budgets upgrade CTA, settings-subscription progress bars).
- **F-5a NEW** — Release detail Files generated document propagation. 3–5 days, bundled under F-5 brand color system.
- **F-6 effort corrected** from 0.5 day to 1 day. Motion work always takes longer than initially scoped.
- **F-13 NEW** — Onboarding flow audit + redesign. Unblocked by D-10 completion.
- **F-14 flagged** — Multi-artist workspace support. Placeholder for Pro-tier work.

## Honest scope adjustment

v2 estimated Batch D at ~4 focused days, Batch E at ~6, Batch F at ~2–3 weeks. v3 revised to ~6 / ~10 / ~3.5–4 weeks. v3.1 revised further to ~8.75 / ~11 / ~4.5–5.5 weeks. Each revision is a correction, not a scope explosion — v2 under-counted surfaces (32 pages, not 62); v3 under-counted effort on specific items (D-8 verification, F-6 motion, E-11 propagation). The underlying diagnosis from v2 held up throughout — this isn't a new list of problems, it's the same problems at honest effort estimates.

---

# Principle precedence (when rules conflict)

Two framings — both matter. The typological list explains what each principle governs. The hierarchical list gives Claude Code a deterministic resolution path when principles genuinely conflict.

## Typological precedence (what each principle governs)

1. **§0.1 Amplifiers, not conveyors** — governs every design token decision. If stripping color breaks the page, fix the page, not the color rule.
2. **§0.3 Mobile parity** — governs every surface that has a mobile variant. A desktop that looks beautiful while the mobile equivalent loses critical structural meta is a failed design.
3. **§0.2 Prefer reversible decisions** — governs D/E/F sequencing. Irreversible changes (D-7 calendar architecture, schema-cascading tokens) happen now; reversible changes (hide nav item, change hex) happen aggressively and revert if wrong.
4. **§1.1 Color discipline** — governs Batch D entirely. No §4 per-page fix overrides §1.1 rules about amber reservation.
5. **§1.5 Information hierarchy + Mobile parity** — governs §4 per-page layout arguments.
6. **§1.6 Generated document micro-interface** — governs any Files tab across entity detail surfaces.

## Hierarchical precedence (deterministic resolution when principles conflict)

**Accessibility > Mobile parity > Typography consistency > Color discipline > Pattern purity**

Translated:
- If an amber CTA fails contrast on a mobile background, drop the color before dropping the CTA. §0.1 > §1.1.
- If mobile parity requires typography compression (summary strip uses smaller type than matching desktop panel), let parity win. §0.3 > §1.2.
- If a Files tab has to choose between §1.6 generated-doc pattern and §1.4 form pattern because both appear on the surface, they **compose** — generated-doc governs the card list, form pattern governs the upload row. If they genuinely conflicted, §1.6 (pattern specificity) wins because it's more semantic.
- Color discipline never wins against mobile parity. Never wins against accessibility.

This hierarchy gives Claude Code a deterministic resolution path during execution.

**Diagnostic closer (not a resolution rule):** when in doubt, strip color, strip motion, strip typography hierarchy. If the page still works, the design is honest. If it doesn't, something is leaning on decoration to do structure's job. Useful as a sanity check, not as an execution rule — the hierarchy above is the execution rule.

---

*This document (v3.1 consolidated) is the canonical reference for the ArtistHQ design audit. It supersedes v2 (18 April 2026) and v3 (18 April 2026). All v3.1 amendments are integrated inline throughout — there is no separate amendments section. Next update will be triggered by (a) Batch D completion, (b) first external user feedback after beta, or (c) addition of Pro-tier pages not yet in scope.*
