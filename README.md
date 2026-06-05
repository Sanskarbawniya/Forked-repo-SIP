# Master Architecture Document (MAD)

Project: [Online SIP Calculator](https://onlinesipcalculator.com) (`onlinesipcalculator`)

**Author:** System Architect & Quality Controller

**Version:** 5.0.0 (32-Tool Ecosystem, 15-Bank Variants, 800 pSEO Value Pages & Canonical URL Migration)

**Target Architecture:** 11ty (Eleventy v3.x ESM) + Vanilla JS + `math.js`

---

## 1. Executive Summary & Core Objective

The purpose of this document is to establish an unyielding technical blueprint for a hyper-performant, programmatic-first financial calculator web platform. The primary business objective is to capture high-intent, long-tail search traffic and outperform legacy institutions like Groww.

### The Competitive Thesis

Legacy platforms rely on heavy single-page application (SPA) frameworks (e.g., Next.js, React) which suffer from layout shifts, hydration lag, and dynamic client-side math computation holes. By stripping away runtime frameworks and implementing an automated 11ty static compilation engine, our architecture delivers zero-bloat, raw HTML files pre-computed with perfect mathematical precision directly to edge servers.

**Current scale:** 32 live calculator tools, 7 bank-branded variant families (105 variant URLs across 15 banks), **800 amount-based pSEO value pages** (SIP, FD, RD, PPF — ₹1,000–₹2,00,000 in ₹1,000 steps), **~940 URLs** in the auto-generated sitemap, educational content + FAQ for every tool, shareable URL state, legacy URL redirects, auto-generated sitemap/`_redirects`, and formula benchmarks for every tool in the registry.

---

## 2. Technical Stack Specifications

The core architecture strictly forbids client-side UI frameworks, asset bundlers, or third-party tracking scrapers that compromise performance. Lightweight analytics (Google Analytics via `site.gaMeasurementId`) is permitted when configured in `src/_data/site.js`.

* **Build Engine:** Eleventy (11ty) v3.x configured natively for ECMAScript Modules (ESM).
* **Template Layer:** Nunjucks (`.njk`) for modular layouts and clean token injection.
* **Computation Library:** `math.js` v14.3.1 (npm + CDN importmap) configured for `BigNumber` execution environments.
* **Frontend Engine:** Native Semantic HTML5, Vanilla CSS3, and Isolated Client-Side Vanilla JavaScript (Web Components).
* **Build pipeline:** `npm run build` → `build:css` → `generate-sitemap.js` (calculator + pSEO URLs) → `clean-stale-build.js` → Eleventy (incl. 800 pSEO pages via pagination) → PostCSS hook (`eleventy.after`).
* **Analytics:** Google Analytics (`G-15LYSSGFYT`) via `site.gaMeasurementId` in `src/_data/site.js`.

---

## 3. Enhanced Project Directory Architecture

```text
├── eleventy.config.js           # 11ty ESM config, passthrough copies, filters (inr, variantContent, valueContent, hubContent)
├── package.json                 # build, serve, test scripts (Node ≥ 20)
├── sitemap.xml                  # Auto-generated at build time (passthrough to _site/)
├── _redirects                   # Auto-generated Netlify/hosting 301 rules (passthrough)
├── robots.txt                   # Crawler rules + sitemap pointer (passthrough)
├── automation-tests/            # Exterior to src/ — compliance & precision gates
│   ├── compliance-audit.js      # HTML structure + YMYL checks on _site/
│   ├── precision-validator.js   # Runs formula benchmarks; optional pSEO HTML math
│   └── formula-benchmarks.js    # Per-tool whole-INR expected values (Groww, ClearTax, etc.)
├── postcss.config.js            # Autoprefixer config
├── scripts/
│   ├── build-css.js             # PostCSS pass → _site/css/ (eleventy.after hook)
│   ├── generate-sitemap.js      # Builds sitemap.xml + _redirects from calculators.js
│   └── clean-stale-build.js     # Prunes stale _site/ paths after renames (pre-build)
├── src/
│   ├── _data/                   # Global data (no formula implementations)
│   │   ├── calculators.js       # Registry, categories, SSR defaults, searchIndex, legacyRedirects
│   │   ├── calculatorContent.js # Per-tool educational sections + FAQ
│   │   ├── legal.js             # Canonical YMYL disclaimer copy
│   │   ├── seoTemplates.js      # Hub + per-calculator titles/descriptions
│   │   └── site.js              # Site name, canonical URL, GA measurement ID
│   ├── _includes/
│   │   ├── base.njk             # HTML shell, math CDN importmap, script slots, YMYL
│   │   └── components/          # Nunjucks partials (server-rendered)
│   │       ├── nav.njk          # Header nav + category dropdown
│   │       ├── footer.njk
│   │       ├── calculator-hub.njk       # Hub grid partial
│   │       ├── related-calculators.njk  # Tool-page sidebar from registry
│   │       ├── more-calculators.njk     # Bank-variant sibling links sidebar
│   │       ├── calculator-page-layout.njk  # Ad sidebar + main + content shell
│   │       ├── calculator-content.njk    # Educational body from calculatorContent.js
│   │       └── ymyl-disclaimer.njk
│   ├── assets/css/
│   │   ├── normalize.css        # Cross-browser baseline (loaded first)
│   │   ├── main.css             # Layout, hub, nav, calculator grid, disclaimer
│   │   └── typography.css       # Form fields, calculator custom-element layout
│   ├── core-engine/             # HEADLESS FINANCIAL FORMULAS (Node + browser ESM)
│   │   ├── math-config.js       # Shared mathjs BigNumber instance (precision: 20)
│   │   ├── format-inr.js        # Whole-INR rounding + Intl display formatting
│   │   └── *.js                 # 30 pure formula modules (see §3.2)
│   ├── calculators/             # One subfolder per tool (no loose templates here)
│   │   ├── index.njk            # Calculator hub grid
│   │   ├── sip/                 # index.njk (paginated) + sip-runtime.js
│   │   ├── fd/, rd/, ppf/, emi/, car-loan-emi/, home-loan-emi/  # Paginated base + bank variants
│   │   └── {tool}/              # 25 single-URL tools: index.njk + {tool}-runtime.js
│   ├── components/              # Web Components (UI + input handling only)
│   │   ├── calculator-ui.js     # Shared UI helpers (chart, fields, bindFieldPair)
│   │   ├── calculator-url-state.js  # Shareable URL query-param parse/sync/copy
│   │   ├── emi-calculator-shared.js # Shared EMI UI wiring for EMI loan variants
│   │   ├── global-search.js     # Client search against /search-index.json
│   │   ├── site-nav.js          # Mobile nav + categories dropdown behavior
│   │   └── {tool}-calculator.js # One custom element per live tool (32 files)
│   ├── legal/disclaimer/index.njk
│   ├── index.njk                # Homepage
│   ├── legacy-redirects.njk     # HTML meta-refresh pages at legacy /calculators/{slug}/ URLs
│   ├── search-index.njk         # Emits /search-index.json from calculators.searchIndex
│   └── pseo/                    # Amount-based programmatic SEO value pages
│       ├── value-pages.js       # PSEO_TOOL_CONFIGS, page factory, buildAllValuePages()
│       ├── value-page.njk       # Paginated layout (one static HTML file per amount)
│       ├── value-page.11tydata.js  # Eleventy pagination data source
│       ├── value-content.js     # Per-amount valueLead + valueFaq copy builders
│       ├── calculator-widget.njk   # SSR custom-element attrs (ignored as standalone page)
│       ├── pseo-validation.njk     # Hidden data-* block for build-time math audit
│       └── preview-runtime.js   # Frozen-preview mode + hub CTA with query-param deep link
```

### Partial vs Web Component Naming

| Path | Purpose |
| --- | --- |
| `src/_includes/components/` | Nunjucks partials (nav, footer, related calculators, YMYL disclaimer) — server-rendered at build time |
| `src/components/` | Custom-element UI widgets and global client scripts — visuals and input handling only; no formulas |

### Canonical Module Boundaries

| Layer | Path | DOM access | May import |
| --- | --- | --- | --- |
| Pure formulas | `src/core-engine/*.js` | No | `mathjs`; other `core-engine/` modules only |
| Build permutations | `src/_data/*.js` | No | `core-engine/` only |
| pSEO page factory | `src/pseo/value-pages.js`, `value-content.js` | No | `core-engine/` only |
| Client runtimes | `src/calculators/{tool}/{tool}-runtime.js` | Yes | `core-engine/`; `components/calculator-url-state.js` |
| pSEO preview runtime | `src/pseo/preview-runtime.js` | Yes | `components/calculator-ui.js`, `calculator-url-state.js` only — no formula imports |
| UI widgets | `src/components/*.js` | Yes | Must not embed formula logic; may import `calculator-ui.js`, `calculator-url-state.js`, `emi-calculator-shared.js` |

### Architectural Quality Rules

1. **Feature Isolation:** Every financial tool must reside exclusively within its own subfolder inside `src/calculators/`. No loose page templates are permitted in the root of `src/`.
2. **Mathematical Separation:** Data generation files (`src/_data/`) handle mapping, looping, and permutation logic only. They must import pure calculation functions from `src/core-engine/` and must not contain formula implementations.
3. **Decoupled Runtimes:** Web Components in `src/components/` handle visual interaction only. Per-tool wiring lives in `src/calculators/{tool}/{tool}-runtime.js` so pages download only their corresponding code footprint (~40–80 lines glue).
4. **Exterior Testing Boundaries:** The `automation-tests/` framework remains completely exterior to the `src/` compilation root, preserving native build performance during hot-reloads.
5. **YMYL Disclosure Mandate:** Every calculator, hub, and content page must render the canonical legal warning via `ymyl-disclaimer.njk` included from `base.njk` (homepage and `/legal/disclaimer/` are exempt). No template may omit, override, or shorten disclaimer text except through `src/_data/legal.js`.

### 3.1 Live Calculator Inventory (32 tools)

| Category | Tools |
| --- | --- |
| **Mutual Funds** | SIP, Lumpsum, Step-Up SIP, SWP, Mutual Fund Returns, ROI, CAGR, Brokerage, Stock Average, XIRR |
| **Fixed Income** | FD, RD, PPF, SSY, NSC, Simple Interest, Compound Interest, Inflation, Post Office MIS, SCSS |
| **Loans & EMI** | EMI, Car Loan EMI, Home Loan EMI, Flat vs Reducing Rate |
| **Tax & Salary** | GST, TDS, Salary, Income Tax, Gratuity |
| **Retirement** | EPF, NPS, APY |

All 32 tools are `available: true` in `calculators.js` with SSR defaults, educational content in `calculatorContent.js`, formula benchmarks, and passthrough runtimes in `eleventy.config.js`.

**Canonical URL pattern:** `/calculators/{slug}-calculator/` (e.g. `/calculators/sip-calculator/`). Legacy paths `/calculators/{slug}/` redirect via `legacy-redirects.njk`, `_redirects`, and `calculators.legacyRedirects`.

### 3.2 Core Engine Modules (`src/core-engine/`)

| File | Responsibility |
| --- | --- |
| `math-config.js` | Single `mathjs` instance; `number: 'BigNumber'`, `precision: 20` |
| `format-inr.js` | `toWholeInr()` and `formatInrDisplay()` for all calculators |
| `sip.js` | Flat SIP FV (annuity due); exports `getEffectiveMonthlyRate()` for shared monthly compounding |
| `lumpsum.js` | One-time investment FV with annual CAGR |
| `step-up-sip.js` | Step-up SIP via month-by-month simulation; imports rate helper from `sip.js` |
| `swp.js` | Systematic withdrawal plan — corpus depletion simulation |
| `mutual-fund-returns.js` | One-time MF investment returns |
| `fd.js` / `rd.js` | Fixed / recurring deposit with quarterly compounding |
| `ppf.js` / `ssy.js` / `nsc.js` | Government savings schemes |
| `post-office-mis.js` / `scss.js` | Post Office MIS and Senior Citizen Savings Scheme |
| `emi.js` | Reducing-balance EMI (shared by EMI, car loan, home loan) |
| `flat-vs-reducing-rate.js` | Side-by-side flat vs reducing loan comparison |
| `simple-interest.js` / `compound-interest.js` | Basic interest formulas |
| `inflation.js` | Future cost / purchasing power |
| `gst.js` / `tds.js` / `salary.js` / `tax.js` / `gratuity.js` | Tax and payroll tools (Income Tax uses `tax.js` + `calculateTaxComparison`) |
| `epf.js` / `nps.js` / `apy.js` | Retirement corpus tools |
| `cagr.js` / `roi.js` / `xirr.js` | Return metrics |
| `brokerage.js` / `stock-average.js` | Equity trade charges and weighted average buy price |

Engines may import each other only within `core-engine/` (e.g. step-up → sip rate, XIRR defaults derived from SIP). They must not import `_data/`, components, or runtimes.

### 3.3 Bank-Branded Variant Pages (Programmatic SEO)

Seven tools expose bank-branded long-tail URLs via **Eleventy pagination** on a single `index.njk` per tool — no duplicate template files per bank.

| Base tool | Example variant slug | Banks (×15 each) |
| --- | --- | --- |
| `sip` | `sbi-bank-sip-calculator/` | SBI, HDFC, ICICI, Axis Bank, Kotak, YES Bank, IDBI, BOB, BOI, PNB, UCO, Canara, Indian Bank, Indian Overseas Bank, IndusInd |
| `fd` | `hdfc-bank-fd-calculator/` | same |
| `rd` | `icici-bank-rd-calculator/` | same |
| `ppf` | `axis-bank-ppf-calculator/` | same |
| `emi` | `kotak-bank-emi-calculator/` | same |
| `car-loan-emi` | `sbi-bank-car-loan-emi-calculator/` | same |
| `home-loan-emi` | `hdfc-bank-home-loan-emi-calculator/` | same |

**Data layer (`calculators.js`):**

- `POPULAR_BANKS` — 15 bank entries with `key`, `bankName`, and display `label`.
- `createBankVariants(toolId, displayName)` builds variant metadata (`slug: {key}-{toolId}`).
- `pagesByTool[toolId]` feeds pagination (`data: calculators.pagesByTool.sip`, etc.).
- `variantSiblingsByToolId` powers the “More calculators” sidebar (`more-calculators.njk`).
- `variantSearchIndex` adds bank pages to `/search-index.json`.
- `legacyRedirects` maps old `/calculators/{slug}/` paths to canonical `-calculator` URLs.

**Template pattern (`src/calculators/sip/index.njk` et al.):**

```njk
pagination:
  data: calculators.pagesByTool.sip
  size: 1
  alias: page
permalink: "{{ page.url }}"
```

Variant pages reuse the same custom element and runtime; only title, description, H1, and educational copy differ. Bank-aware prose uses `{bankPrefix}`, `{bankName}`, `{bankLead}`, `{bankVia}`, `{bankPlatform}` tokens in `calculatorContent.js`, resolved at build time via the `variantContent` Nunjucks filter in `eleventy.config.js`.

**Sitemap & redirects:** `scripts/generate-sitemap.js` runs during `npm run build` and emits `sitemap.xml` from `calculators.pagesByTool` (base + all bank variant URLs), `buildAllValuePages()` (pSEO value URLs), plus static pages (`/`, `/calculators/`, `/legal/disclaimer/`). The same script writes `_redirects` from `calculators.legacyRedirects` for hosting-level 301s. `src/legacy-redirects.njk` provides HTML fallback redirects at legacy paths. `scripts/clean-stale-build.js` removes orphaned `_site/` output after URL migrations.

### 3.4 Amount-Based pSEO Value Pages

Four tools expose long-tail **amount-specific** URLs via Eleventy pagination on `src/pseo/value-page.njk` — no duplicate template files per amount.

| Tool | URL pattern | Amount range | Fixed defaults |
| --- | --- | --- | --- |
| SIP | `/calculators/{amount}-sip-calculator/` | ₹1,000–₹2,00,000 (step ₹1,000) | 10 years, 12% p.a. |
| FD | `/calculators/{amount}-fd-calculator/` | same | 5 years, 7% p.a. |
| RD | `/calculators/{amount}-rd-calculator/` | same | 5 years, 7% p.a. |
| PPF | `/calculators/{amount}-ppf-calculator/` | same | 15 years, 7.1% p.a. |

**Scale:** 200 amounts × 4 tools = **800 static HTML pages** at build time.

**Data layer (`src/pseo/value-pages.js`):**

- `PSEO_TOOL_CONFIGS` — per-tool amount range, fixed tenure/rate, hub URL, and script bundle.
- `buildAllValuePages()` — loops amounts, calls `calculate*` from `core-engine/`, attaches formatted results and copy.
- `buildValueLead()` / `buildValueFaq()` in `value-content.js` — amount-unique educational blocks prepended/appended on value pages only.

**Template pattern (`value-page.11tydata.js`):**

```javascript
pagination: { data: "valuePages", size: 1, alias: "page" }
```

**Page bundle (differs from hub calculators):**

| Artifact | Role |
| --- | --- |
| `value-page.njk` | Layout shell, intro paragraph, includes `calculator-widget.njk` + `pseo-validation.njk` |
| `calculator-widget.njk` | SSR custom element with pre-computed result attributes (no `{tool}-runtime.js`) |
| `pseo-validation.njk` | Visually hidden `<section data-pseo-calculator="…">` with raw inputs + `data-maturity-amount` for `precision-validator.js` |
| `preview-runtime.js` | **Preview mode** — results stay frozen at SSR values; sliders update a hub "Calculate" CTA link via `buildHubShareUrl()` |

**Nunjucks filters (`eleventy.config.js`):**

- `valueContent` — replaces `{amount}`, `{tenure}`, `{rate}`, `{maturity}`, `{invested}`, `{gains}`, `{hubUrl}`, `{hubLink}` tokens in educational copy on value pages.
- `hubContent` — same token set with illustrative defaults on hub calculators (no fixed amount).
- `variantContent` — unchanged; bank-variant pages only.

**Content rendering:** `calculator-content.njk` accepts optional `valuePage` context — renders `valueLead` before shared sections and `valueFaq` after the hub FAQ. Hub sections pass through `valueContent` / `hubContent` filters for token substitution.

**Eleventy ignores:** `calculator-widget.njk` and `pseo-validation.njk` are partials only (included by `value-page.njk`, not standalone routes).

**Sitemap:** pSEO URLs are included at `priority: 0.8`, `changefreq: weekly` via `buildAllValuePages()` in `generate-sitemap.js`.

**Stale cleanup:** `clean-stale-build.js` removes `_site/pseo` and legacy permalink folders matching `sip-{n}-per-month-{n}-years-{n}-percent-calculator` before each build.

### 3.5 Legacy URL Migration

Production canonical paths use the `-calculator` suffix. Older `/calculators/{slug}/` URLs are preserved for SEO equity via three mechanisms:

| Mechanism | File | Purpose |
| --- | --- | --- |
| Hosting 301 | `_redirects` (auto-generated) | Netlify/Cloudflare-style permanent redirects |
| HTML fallback | `src/legacy-redirects.njk` | Meta-refresh + `location.replace` pages at legacy permalinks |
| Stale cleanup | `scripts/clean-stale-build.js` | Deletes leftover `_site/calculators/{old-slug}/` after renames |

Special case: `/calculators/tax/` redirects to `/calculators/income-tax-calculator/` (tool was renamed from `tax` to `income-tax`). `base.njk` includes a client-side guard for the bare `/calculators/tax` path.

### Per-Calculator Page Bundle

Each live tool under `src/calculators/{tool}/` ships exactly:

| Artifact | Role |
| --- | --- |
| `index.njk` | Page layout, SEO front matter, SSR default attributes on the custom element |
| `{tool}-runtime.js` | Listens for `{tool}-input` event, calls `calculate*` from `core-engine/`, updates result attributes; wires shareable URL state via `calculator-url-state.js` |

Matching Web Component: `src/components/{tool}-calculator.js` (or `sip-calculator.js` naming pattern).

### Adding a New Calculator (Checklist)

1. **`src/core-engine/{tool}.js`** — pure `calculate*` function; use `math-config.js` and `format-inr.js`.
2. **`src/components/{tool}-calculator.js`** — custom element; dispatch `{tool}-input` with input detail; no math imports.
3. **`src/calculators/{tool}/index.njk`** + **`{tool}-runtime.js`** — `layout: base.njk`, `useMathCdn: true`, `scripts` array, SSR defaults from `_data`.
4. **`src/_data/calculators.js`** — registry entry (`available: true`), `default{Tool}` via engine import, `searchIndex` row.
5. **`src/_data/seoTemplates.js`** — `hubTitle` / `hubDescription` for the tool.
6. **`eleventy.config.js`** — passthrough line for `{tool}-runtime.js`.
7. **Registry-driven SEO artifacts** — adding a registry entry automatically includes the tool in `generate-sitemap.js` output and `legacyRedirects`. No manual `sitemap.xml` edit required unless adding non-calculator static pages.
8. **`automation-tests/formula-benchmarks.js`** — at least one verified case per tool; include degeneracy cases (e.g. `annualStepUpPct: 0` for flat SIP) where applicable.
9. **CSS & page shell** — use shared classes (`calculator-tool__inputs`, `calculator-tool__results`) from `calculator-ui.js`; include `calculator-page-layout.njk` in `index.njk`; add copy to `calculatorContent.js`; load `calculator-ui.js` before the tool component script.
10. **Shareable URL state** — add a schema entry in `calculator-url-state.js` (`URL_SCHEMAS` or `createEmiUrlSchema`); wire via `wireCalculatorRuntime()` or `wireCalculatorUrlShare()` in `{tool}-runtime.js`. Encode **inputs only** (short query keys); never encode computed results.
11. **pSEO value pages (optional)** — add a `PSEO_TOOL_CONFIGS` entry in `src/pseo/value-pages.js`, a branch in `calculator-widget.njk`, a `PREVIEW_TOOLS` entry in `preview-runtime.js`, and copy builders in `value-content.js`. Extend `extractPseoSections()` in `precision-validator.js` if HTML audit coverage is needed beyond SIP.

### Shareable Calculator State (URL Query Params)

Every live calculator exposes shareable links that encode the user's **inputs** in the URL query string. Recipients opening the link see the same inputs and recalculated results via the existing client runtime and `core-engine/` — no results are stored in the URL.

| Rule | Specification |
| --- | --- |
| **Encoding** | Inputs only, as short query keys (e.g. SIP: `?m=5000&t=10&r=12`). Results are never serialized. |
| **Module** | `src/components/calculator-url-state.js` — parse, apply, `history.replaceState`, copy-to-clipboard. Schemas live in `URL_SCHEMAS` / `createEmiUrlSchema()`. |
| **Runtime wiring** | Each `{tool}-runtime.js` calls `wireCalculatorRuntime()` or `wireCalculatorUrlShare()` after the custom element is present. |
| **Validation** | URL values are clamped/validated the same way as slider limits; invalid or missing params fall back to SSR defaults. |
| **URL sync** | Debounced `replaceState` (~300 ms) on input change; does not push history entries. |
| **Share UI** | “Copy shareable link” button injected into `.calculator-tool__results` by the URL-state module. |
| **Canonical SEO** | `<link rel="canonical">` remains the clean path without query params (`base.njk`). Title/meta stay static from `seoTemplates.js`. |
| **vs pSEO** | Share links restore live inputs on **hub** calculators via `replaceState`. pSEO value pages (`src/pseo/`) are build-time permutations with **frozen SSR results**; `preview-runtime.js` deep-links to the hub via `buildHubShareUrl()` on the "Calculate" CTA. Permalink patterns must not overlap (hub: `/calculators/{slug}-calculator/`; pSEO: `/calculators/{amount}-{slug}-calculator/`). |

---

## 4. Core Mathematical Precision Specifications

To guarantee consumer trust and search engine compliance, native IEEE 754 floating-point arithmetic is banned for financial projections. Binary floating-point errors multiply aggressively over long investment durations.

---

### Global Scale Configurations

All mathematical contexts—both build-time Node.js generators and client-side runtime processors—must instantiate an immutable structural calculation layout:

```javascript
math.config({ number: 'BigNumber', precision: 20 });
```

### Dual Runtime Loading

| Environment | `math.js` source | Module path |
| --- | --- | --- |
| Build-time (Node) | npm `mathjs` dependency | `src/core-engine/` imported by `_data/` and pagination templates |
| Client-side (browser) | math.js v14.3.1 CDN importmap in `base.njk` (`useMathCdn: true`) | `{tool}-runtime.js` imports passthrough copies of `core-engine/` modules |

Both environments must call `math.config()` identically before any calculation. No bundler may rewrite or tree-shake formula modules.

### Display & Validation Rounding

Internal compound calculations use BigNumber **precision 20** (see `math-config.js`). All user-facing currency values displayed in HTML or UI must be **rounded to the nearest whole INR** (integer rupees) via `toWholeInr()`. `formula-benchmarks.js` and `precision-validator.js` compare whole-INR integers only.

---

## 5. Quality Control & Performance Validation Matrix

The platform must maintain strict engineering performance ceilings. Any code deployment exceeding these quality gates will trigger an automatic build pipeline failure.

| Engineering Metric | Competitor Benchmark (Groww) | Platform Standard Requirement | Enforcement Mechanism |
| --- | --- | --- | --- |
| **Total Transfer Weight** | 800 KB - 1.5 MB | **< 65 KB Combined Assets** | Automated Build Lint Gate |
| **Largest Contentful Paint (LCP)** | ~1.8s - 2.5s | **< 350 Milliseconds** | Zero framework runtime hydration |
| **Interaction to Next Paint (INP)** | ~140ms | **15 Milliseconds** | Micro-second native DOM mutation loops |
| **Mathematical Accuracy** | Varied Floating Drift | **0.00% Fractional Deviation** | `formula-benchmarks.js` via `precision-validator.js` (`npm test`) |
| **Layout Shift Index (CLS)** | > 0.10 | **0.00 Perfect Structural Lock** | Pre-sized input fields, static layout elements |

### Mandatory Quality Validation Script Pipeline (`automation-tests/`)

Before pushing a compiled directory to live edge nodes, run **`npm test`** (`build` + both scripts below).

1. **`compliance-audit.js`**: Confirms that every static HTML output has `<h1>`, `<title>`, `link[rel="canonical"]`, and the mandatory YMYL legal warning (see §6).

2. **`precision-validator.js`**: Runs all cases in **`formula-benchmarks.js`** against `core-engine/` functions; then scans built HTML for pSEO validation blocks (`data-pseo-calculator` in `pseo-validation.njk`) and re-runs `calculateSip()` against embedded `data-maturity-amount` values. Currently validates **SIP pSEO pages only** — extend `extractPseoSections()` when adding FD/RD/PPF HTML audits.

### Formula Benchmark Registry (`formula-benchmarks.js`)

Each calculator exports a block: `{ tool, calculate, cases[] }`. Every case supplies `input` and `expect` (whole-INR fields such as `maturityAmount`, `totalInvested`, `wealthGained`, or tool-specific keys). The registry covers **all 32 live tools** (33 blocks — Income Tax has separate `tax` and `tax-comparison` benchmark groups). New calculators must add benchmarks before merge.

---

## 6. YMYL Legal Disclosure & Regulatory Warning

Financial calculator pages are classified as **Your Money or Your Life (YMYL)** content. Search quality raters and regulators expect clear, conspicuous disclaimers that the site provides **tools and education only**, not personalized financial advice or regulated investment services.

### 6.1 Placement & Presentation Rules

| Requirement | Specification |
| --- | --- |
| **Position** | Final child inside `<main>`, after all page-specific content and **before** `</main>`. The site footer (`footer.njk`) follows `</main>`. |
| **Inclusion** | Injected once per page through `src/_includes/base.njk` via `{% include "components/ymyl-disclaimer.njk" %}`. **Excluded** on the homepage (`/`) and the full disclaimer page (`/legal/disclaimer/`). Individual templates must not duplicate or skip the include elsewhere. |
| **Semantics** | Wrapper: `<aside class="ymyl-disclaimer" aria-labelledby="ymyl-disclaimer-heading">`. Heading: `<h2 id="ymyl-disclaimer-heading">` (visually de-emphasized via CSS; not omitted for accessibility). |
| **Visibility** | Must be present in the static HTML at build time (not loaded by JavaScript). Must remain readable without interaction (no accordions, no “click to agree” gates). |
| **Styling** | `src/assets/css/main.css` defines `.ymyl-disclaimer` — subdued typography (smaller than body), sufficient contrast (WCAG AA), adequate padding. Disclaimer must not be hidden, zero-height, or `display:none`. |
| **Copy source** | All strings live in `src/_data/legal.js`. Templates reference data keys only; inline disclaimer prose in page templates is forbidden. |

### 6.2 Canonical Disclaimer Copy (`src/_data/legal.js`)

The following text is the **mandatory baseline** for all locales until legal review approves a revision. Updates require a MAD version bump and re-run of `compliance-audit.js`.

**Heading (`legal.ymyl.heading`):** `Important legal notice`

**Body (`legal.ymyl.paragraphs`) — render as separate `<p>` elements in order:**

1. The calculators, charts, and figures on this website are provided for **general information and educational purposes only**. They do not constitute financial, investment, tax, or legal advice, and they are not a recommendation, offer, or solicitation to buy or sell any security, mutual fund, insurance product, or other financial instrument.

2. **We are not a SEBI-registered investment adviser, broker, or distributor.** Nothing on this site creates a fiduciary, advisory, or client relationship between you and the operator of this website.

3. **Results are illustrative estimates** based on the inputs you supply and stated assumptions (including constant rates of return unless otherwise noted). Actual outcomes may differ materially due to market volatility, fees, taxes, inflation, timing of cash flows, regulatory changes, and other factors not modeled here.

4. **Past or projected performance is not a guarantee of future results.** Where returns are shown, they are hypothetical unless explicitly labeled as historical data from a cited source.

5. **Tax and regulatory treatment** varies by jurisdiction and personal circumstances. Calculators may simplify or omit tax, surcharge, cess, TDS, capital-gains treatment, and product-specific charges. Consult a qualified chartered accountant or tax professional before acting.

6. **You are solely responsible** for decisions you make using these tools. The operator disclaims liability for losses or damages arising from reliance on calculator outputs, errors, omissions, interruption, or outdated assumptions.

7. **Verify all figures** with official scheme documents, AMC/KRA statements, lender amortization schedules, or licensed professionals before investing, redeeming, borrowing, or filing returns.

**Footer link (`legal.ymyl.fullDisclaimerUrl`):** `/legal/disclaimer/` — full disclaimer and privacy policy page (must exist before production launch).

### 6.3 Template Implementation Contract

`src/_includes/components/ymyl-disclaimer.njk` must:

- Iterate `legal.ymyl.paragraphs` and emit one `<p>` per entry.
- Include a link to the full disclaimer: `<a href="{{ legal.ymyl.fullDisclaimerUrl }}">Read full disclaimer</a>`.
- Avoid `nofollow` on the internal legal link; use `rel="noopener"` only on external links if added later.

`src/_includes/base.njk` must call the include as the **final child inside `<main>`** — no alternate placement patterns are permitted.

### 6.4 Compliance Audit Extensions

`automation-tests/compliance-audit.js` must fail the build if any HTML output under `_site/`:

- Lacks an element matching `aside.ymyl-disclaimer` (or `[data-ymyl-disclaimer="true"]` if that attribute is adopted in the partial).
- Lacks the heading text `Important legal notice` (or the current `legal.ymyl.heading` value from data).
- Contains fewer than seven disclaimer `<p>` elements inside the aside (count tracks `legal.ymyl.paragraphs` length).
- Omits a link whose `href` ends with `/legal/disclaimer/`.

### 6.5 YMYL Content Guidelines (Editorial)

- Do not use language implying guaranteed returns, “safe” investments, or beat-the-market claims in calculator UI or pSEO copy.
- When citing benchmarks or historical averages, attribute the source and date in page body copy (outside the disclaimer block).
- Programmatic SEO titles and meta descriptions must not promise specific returns for the user.

---

## 7. Content Governance & Editorial Integrity

All textual content for calculator body sections, educational blocks, and FAQ modules lives in `src/_data/calculatorContent.js` and `src/_data/seoTemplates.js`. Copy must be factual, YMYL-safe (no guaranteed-return language), and consistent with the tone used across existing tools. When a separate `CONTENT_MASTER.md` editorial guide is added, it becomes the authoritative tone-of-voice reference; until then, match the patterns in existing calculator entries.

---

## 8. Cross-Platform CSS Strategy

Financial calculator UIs must render identically across **Chromium** (Chrome, Edge, Android WebView) and **WebKit** (Safari, iOS). Layout drift — especially vertical expansion of `.calculator-card--tool` when sliders move — erodes trust on a YMYL site. This section defines the CSS toolchain, layout-stability contract, and validation workflow.

### 8.1 Toolchain

| Tool | Role | Integration |
| --- | --- | --- |
| **`normalize.css`** | Shared cross-browser baseline (box model, form inheritance, tap highlight) | Loaded **first** in `base.njk` |
| **PostCSS + Autoprefixer** | Vendor prefixes for Flexbox, Grid, `line-clamp`, transforms | `scripts/build-css.js` runs on `eleventy.after`; outputs to `_site/css/` |
| **`browserslist`** | Autoprefixer target matrix | `last 2` Chrome, Firefox, Safari, iOS, Android in `package.json` |

CSS is **not** passthrough-copied. Author styles in `src/assets/css/` only; run `npm run build` before deploy.

### 8.2 Calculator Card Layout Stability (CLS = 0)

The calculator card must **not change height** when sliders move or result values update. These rules are mandatory:

| Rule | Implementation |
| --- | --- |
| **Fixed field block height** | `.field--tool` uses CSS variables (`--field-label-height`, `--field-input-height`, `--field-slider-height`) to lock each input column to `--field-tool-block-height` |
| **No flex stretch** | Remove `min-height: 100%` and `margin-top: auto` on range sliders; use explicit `flex: 0 0` heights |
| **Grid align start** | `.calculator-tool__inputs { align-items: start }` — columns must not stretch to match tallest neighbour |
| **Locked result row** | `.result-card__value` and summary-table `td` use fixed `min-height: var(--result-value-line-height)` with `white-space: nowrap; overflow: hidden; text-overflow: ellipsis` |
| **Visual-only scaling** | `calculator-ui.js` `applyResultValueScale()` uses CSS `transform: scale()` — never font-size changes — so long INR values shrink visually without reflowing layout |
| **Layout containment** | `.calculator-card--tool { contain: layout style }` prevents internal reflow from propagating outward |
| **Select-only fields** | Fields without range sliders get `padding-bottom` equal to the slider row so mixed grids stay aligned |

### 8.3 Cross-Browser Authoring Standards

| Rule | Rationale |
| --- | --- |
| **Prefer fluid layouts** | Use `minmax()`, `clamp()`, `min()`, `rem`, `fr` — not fixed pixel widths for structural columns |
| **Use `dvh` with `vh` fallback** | `--viewport-height: 100vh` then `100dvh`; mobile nav uses `calc(100dvh - var(--header-height))` |
| **Reset native form appearance** | Inputs, selects, range sliders, buttons: `-webkit-appearance: none; appearance: none` |
| **iOS input zoom guard** | Mobile text inputs: `font-size: 1rem` (16px) minimum |
| **Grid/Flex overflow** | Truncating flex/grid children need `min-width: 0` and/or `min-height: 0` |
| **No manual vendor prefixes** | Write standard CSS; Autoprefixer adds prefixes (except slider pseudo-elements) |
| **iOS scroll lock** | `body.site-nav-open { overflow: hidden; position: fixed; width: 100% }` |

### 8.4 Stylesheet Load Order (`base.njk`)

```html
<link rel="stylesheet" href="/css/normalize.css">
<link rel="stylesheet" href="/css/typography.css">
<link rel="stylesheet" href="/css/main.css">
```

### 8.5 Cross-Platform Validation Workflow

1. **`npm run build && npm run serve`** — confirm PostCSS output in `_site/css/`.
2. **Chromium:** Chrome DevTools → Pixel 7 + iPhone 14 Pro; drag all sliders on `/calculators/sip-calculator/` — card height must not jump.
3. **WebKit:** Safari on macOS + physical iPhone — verify mobile nav, range thumb alignment, number inputs (no spin buttons, no focus zoom), scroll lock.
4. **Playwright (recommended CI):** Visual snapshots at 390×844 and 1280×800 using `chromium` + `webkit` on `/calculators/sip-calculator/`.
5. **BrowserStack (release gate):** Safari 17+ on macOS/iOS before production.
6. **`npm test`** — compliance + formula gates must pass.

### 8.6 Golden Rules (Team Workflow)

1. **CSS Reset/Normalization:** `normalize.css` is the first stylesheet — never reorder it.
2. **Enforce PostCSS/Autoprefixer:** Do not hand-write `-webkit-` prefixes except slider pseudo-elements; run `npm run build` before committing CSS.
3. **Test slider interaction on real WebKit:** Drag every slider on iOS — if the card height changes, the layout-stability contract in §8.2 is broken.

---