# E-9a — Currency Rendering Architecture (1-Pager)

**Date:** 2026-04-19
**Status:** DRAFT — awaiting Jacob review before E-9 implementation begins.
**Related:** `design-system.md §3.16 currency`, `design-system.md §5 Batch E`, E-9 (implementation, blocked on this).

---

## 1. Current state audit

Currency handling works for single-currency artists and quietly breaks for multi-currency ones.

**Central helpers live in `apps/web/lib/ui.ts:10-32`:**

```ts
export function fmtMoney(amount: number, currency = "USD"): string {
  return `${currencySymbol(currency)}${amount.toLocaleString("en-US", {
    minimumFractionDigits: 2, maximumFractionDigits: 2,
  })}`;
}
export function fmtMoneyShort(amount, currency = "USD") { /* no decimals */ }
export function fmtFee(amount | null, currency = "USD") { /* null → "—" */ }
export function currencySymbol(c: string): string {
  return CURRENCY_SYMBOLS[c] ?? "$";
}
const CURRENCY_SYMBOLS = {
  USD:"$", EUR:"€", GBP:"£", SEK:"kr", NOK:"kr", DKK:"kr",
  CAD:"CA$", AUD:"A$", JPY:"¥", BRL:"R$",
};
```

Three defects in this helper, in increasing severity:

1. **Locale is hard-coded `"en-US"`.** A German artist with EUR prices gets `€1,234.56` instead of the locally-expected `1.234,56 €`. Grouping/decimal style does not follow locale, only the symbol prefix does.
2. **Unknown currencies silently render as `$`.** `fmtMoney(100, "CHF")` → `$100.00`. No warning, no log.
3. **Default parameter is `"USD"`, not the user's preference.** Any callsite that forgets to pass a currency prints USD. This is not theoretical — there are confirmed live bugs (see §2).

**Storage is consistent but denormalized.** Every money-bearing row carries its own `currency TEXT DEFAULT 'USD'`: `shows`, `expenses`, `income`, `recurring_expenses`, `budgets`, `crew`, `merch_sales` (confirmed via `supabase/migrations/20260408050000_money.sql` and `20260406010000_add_crew_and_documents.sql`). The one schema gap is **`product_variants` has no currency column** — variant prices implicitly assume the artist's default currency.

**Artist default currency lives in `artists.preferences.currency` (JSONB).** The settings "Preferences" tab (`apps/web/app/(dashboard)/settings/page.tsx:821-861`) writes it via an `EntityPicker` over a hard-coded list of 10 supported currencies. Every page that cares reads it independently:

```ts
// money/page.tsx:45, 114 — repeated in merch/page.tsx:25, settings/page.tsx:217
setDefaultCurrency(prefs.currency ?? "USD");
```

**Rendering today:** symbol + en-US grouped amount. No ISO codes anywhere. No Intl.NumberFormat in currency code (it's used only in PDF/DOCX export templates for unrelated locale formatting).

**Live bugs that E-9 must fix (already identified in the audit):**

| File:line | Problem | User-visible effect |
|---|---|---|
| `IncomeTab.tsx:66,72,78,172` | `fmtMoney(total)` — currency arg omitted | Income totals render `$` regardless of actual currency |
| `ExpensesTab.tsx:247` | `fmtMoney(totalShown)` — currency arg omitted | Expense table footer always renders `$` |
| `OverviewTab.tsx:30-35` | Sums fees + expenses across all currencies before rendering with `defaultCurrency` | EUR shows + USD expenses produce a meaningless net number |
| `BandBankTab.tsx:164-170` | Sums `earned`/`paid` with no currency check, then renders with `defaultCurrency` | Mixed-currency earnings masked as single-currency total |

`ReportsTab.tsx:84-96` is the one place that handles mixed currencies correctly — it buckets income/expenses by currency into `currencyBuckets` and renders each bucket separately. This pattern is the model.

---

## 2. Problem statement (`design-system.md §3.16`)

Currency rendering needs to answer three questions consistently:

1. **Which locale formats the number?** Grouping separators, decimal marks, and symbol position vary by locale, not by currency. A French-speaking user seeing EUR wants `1 234,56 €`; an American seeing EUR wants `€1,234.56`.
2. **Which currency symbol or code is shown?** Only the symbol (`$1,234.56`), only the ISO code (`USD 1,234.56`), both (`US$ 1,234.56`), or context-dependent?
3. **What happens when values across currencies are aggregated?** Refuse (bucket by currency), convert (with what rate, stored where, refreshed how), or silently sum (current, broken)?

E-9a answers these three questions for E-9. E-9 then makes the implementation match.

---

## 3. Architectural decisions (A–D)

### A. Formatting engine — `Intl.NumberFormat` with locale as an explicit input

Replace the hand-rolled `currencySymbol` map + `toLocaleString("en-US", …)` with `Intl.NumberFormat(locale, { style: "currency", currency, currencyDisplay: "narrowSymbol" })`.

**Why:**

- Handles thousands of currency codes, not just the 10 we enumerate. Removes the silent `$` fallback.
- Produces locale-appropriate grouping/decimal marks automatically.
- `currencyDisplay: "narrowSymbol"` gives us `$` / `€` / `£` for common currencies and falls back to sensible compound symbols (`CA$`, `A$`) for the rest — matching how the hand-rolled map behaves today, but without the maintenance.
- Zero new dependencies — `Intl` is in the JS runtime.

**The symbol map stays only in one place:** the settings picker, where we need pretty labels (`USD ($)`) for the dropdown. Everywhere else defers to `Intl`.

### B. Locale — derive from artist preferences, not hard-coded

`artists.preferences.locale?: string` — new optional field. When unset, fall back to `navigator.language` on the client, `"en-US"` on the server.

**Why:**

- Decouples locale from currency. A US artist touring Europe with EUR fees should still see EUR formatted their way (`€1,234.56`), not suddenly see European formatting because the fee happens to be in EUR.
- JSONB additive change — no migration needed, just a settings UI addition (defer to E-9b / later).
- **E-9 scope: ship the formatter accepting locale and default it to `"en-US"`** — do not ship a locale picker yet. This preserves today's behavior exactly while unblocking future work.

### C. Aggregation — refuse mixed-currency arithmetic, bucket and render per currency

No conversion. No FX. No rate table. When a view needs a total across rows with different currencies, it must:

1. Group rows by currency.
2. Render a total per currency, either stacked (OverviewTab-style KPI cards) or bucketed side-by-side (ReportsTab-style).
3. Never produce a single number that pretends to represent mixed currencies.

**Why:**

- FX conversion introduces three hard problems at once: which rate source, how fresh, and how to display a converted number without misleading the user that it's an authoritative amount. Out of scope for the launch window (explicit scope guard from the user's directive).
- The `ReportsTab` pattern already works and already ships. We extend it, we don't invent.
- Bucketing is honest: it tells the artist "you have $X from US shows and €Y from EU shows" rather than pretending we know what that's worth in one currency.

### D. Defaults and fallbacks — a single source of truth

One helper to get "the currency this value should render in":

```ts
// apps/web/lib/currency.ts (new)
export function pickCurrency(row: { currency?: string | null }, artistDefault: string): string {
  return row.currency ?? artistDefault;
}
```

Every callsite that today writes `fmtMoney(value, row.currency)` or (buggy) `fmtMoney(total)` becomes explicit:

```ts
fmtMoney(value, pickCurrency(row, artistDefault))   // row-level
fmtMoney(total, artistDefault)                      // aggregate — only valid when bucket is single-currency
```

**The default parameter on `fmtMoney` itself goes away.** Forgetting to pass a currency becomes a TypeScript error, not a silent USD rendering.

---

## 4. Recommended architecture

A new `apps/web/lib/currency.ts` module:

```ts
// Inputs are explicit. No hidden defaults.
export function formatMoney(
  amount: number,
  currency: string,           // required — no default
  opts?: {
    locale?: string;          // default: "en-US"
    compact?: boolean;        // drop decimals, for KPI cards
    signed?: boolean;         // prefix "-" for expenses
  }
): string;

export function pickCurrency(
  row: { currency?: string | null },
  artistDefault: string,
): string;

// Aggregation primitive — groups rows into buckets, ready for rendering.
export function bucketByCurrency<T extends { currency: string; amount: number }>(
  rows: T[],
): Array<{ currency: string; total: number; rows: T[] }>;
```

`apps/web/lib/ui.ts` retains thin re-exports of `fmtMoney`/`fmtMoneyShort`/`fmtFee` for one version — each re-export forwards to the new module and requires the `currency` argument. Any callsite that omitted it fails the type-check and surfaces as a bug fix in E-9.

**Rendering surface** — no visual change on single-currency artists (the vast majority). On multi-currency artists:

- **KPI cards (OverviewTab):** stack two lines inside the card when the bucket count is 2+ (`$1,200 · €900`). Three or more currencies: render only the "primary" bucket (largest by magnitude) with a `+N more` affordance linking to a detail view.
- **Table footers (ExpensesTab, IncomeTab):** switch from a single total to a comma-separated list of per-currency totals. Already works at sub-linear width growth because most artists have 1–2 currencies.
- **ReportsTab:** already correct, no change required beyond formatter swap.
- **BandBankTab totals:** adopt the bucket pattern; this fixes today's mixed-currency masking bug.

---

## 5. E-9 implementation scope

**In scope (E-9):**

1. Ship `apps/web/lib/currency.ts` with `formatMoney`, `pickCurrency`, `bucketByCurrency`.
2. Re-export `fmtMoney`/`fmtMoneyShort`/`fmtFee` from `ui.ts` as thin wrappers with required `currency` arg (removes the `= "USD"` default).
3. Fix the five confirmed live bugs by passing the correct currency (`IncomeTab.tsx:66,72,78,172`; `ExpensesTab.tsx:247`).
4. Convert `OverviewTab`, `BandBankTab` to `bucketByCurrency` — stacked KPI cards when bucket count ≥ 2.
5. Leave `ReportsTab` aggregation logic alone (already correct); only swap the formatter.
6. Add a single Jest/Vitest spec for `bucketByCurrency` + `formatMoney` edge cases (unknown currency code, zero amount, negative amount, very large amounts).

**Explicitly out of scope (E-9):**

- Locale picker in settings (covered by §B; defer to E-9b or later).
- FX conversion, rate sourcing, rate storage, rate freshness UI.
- Currency switching UI (i.e. "view all my finances in USD regardless of what currency they're stored in"). Would require FX.
- `product_variants.currency` schema gap (variant-level currency). Defer to an explicit follow-up; requires product modeling decisions about multi-currency pricing.
- Landing page rebuild — untouched per user's scope guard.
- Mobile structural redesign — untouched per user's scope guard.

---

## 6. Out-of-scope and follow-ups

**Deferred to E-9b (or post-launch):**

- Locale picker in settings (`artists.preferences.locale`).
- Per-variant currency in merch (schema + UI).
- Compact KPI formatting for ≥3 currencies (the `+N more` affordance is listed here as rendering only; if 3+ currencies become common, a dedicated multi-currency detail view earns its keep).

**Will not ship until there is a real product need:**

- FX conversion pipeline (rate provider, storage, freshness).
- "Report in currency X" toggle (depends on FX).
- Historical exchange-rate pinning on transactions (e.g., "this EUR fee was worth $X on the show date").

**Non-goal:**

- Matching every native OS currency formatting edge case. `Intl` handles this for us; we don't reimplement.

---

## Decision points for Jacob

Before E-9 starts, confirm:

1. **Locale:** ship E-9 with locale hard-coded to `"en-US"` (preserves today's rendering exactly), add picker in E-9b? ✅ recommended
2. **Aggregation:** bucket-by-currency is the model, FX conversion stays off the table? ✅ recommended
3. **KPI cards on multi-currency artists:** stacked per-currency lines for 2 currencies, `+N more` for 3+? ✅ recommended
4. **Breaking change to `fmtMoney`:** remove the `currency = "USD"` default so forgotten arguments become type errors? ✅ recommended — this is how the five live bugs get surfaced and fixed
5. **`product_variants.currency`:** defer to explicit follow-up, out of E-9 scope? ✅ recommended

If all five are ✅, E-9 is unblocked and implementation proceeds per §5 scope.
