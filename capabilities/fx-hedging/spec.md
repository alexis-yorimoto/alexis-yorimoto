---
title: Hospital Records FX Transaction Hedge Model, Technical Specification
scenario: hospital-records
llm_used: Claude (drafted from the course template; reviewed with Claude for Stage 2)
---
 
University of Hawaiʻi at Mānoa · Shidler College of Business
 
FIN 321: International Finance and Securities
 
FX Transaction Hedging Project — Technical Specification
 
# Hospital Records Software Firm — FX Transaction Hedge Model · Technical Specification
 
> Technical specification for the EUR receivable hedge model: the named-range contract, tab architecture, calculation flow, and validation checks, precise enough that an AI or a colleague could rebuild the workbook from this document alone.
 
| Field                   | Value                                                      |
| ----------------------- | ---------------------------------------------------------- |
| **Created by**          | Alexis Yorimoto                                            |
| **Updated by**          | Alexis Yorimoto                                            |
| **Date Created**        | 2026-11-06                                                 |
| **Date Updated**        | 2026-11-12                                                 |
| **Version**             | 0.2                                                        |
| **LLM Used** (optional) | Claude, used to fill the template and to review the spec   |
| **Role**                | Treasury Analyst                                           |
| **Audience**            | CFO / Director of Treasury                                 |
| **Companion Workbook**  | student build                                              |
 
---
 
## 1. Problem Statement
 
The hospital records software firm expects a EUR 2,200,000 receivable from an Austrian hospital network, settling in 182 days. The rate is quoted in USD per EUR, with spot at 1.1204, so the receivable is worth $2,464,880 today. A 5% fall in the euro over the horizon would cut realized USD proceeds by about $123,000, and a 10% fall by about $246,000, and that loss comes straight out of profit. The objective is to protect USD value at settlement at an acceptable cost. This specification documents the framework used to quantify and compare four strategies (**no hedge**, **forward hedge**, **money-market hedge**, and **option (put) hedge**) and to produce the sensitivity evidence behind the Stage 5 decision memo. A mirror **payable view** (§4, Step 7) shows how the same instruments, including a call option, would work if the firm owed EUR 2,200,000 instead of receiving it.
 
---
 
## 2. Inputs (Known Variables)
 
All inputs are exposed as workbook **named ranges**, so the Calculation Flow (§4) reads the same whether implemented in Excel, Python, or an AI prompt. Each input has a Stage 4 source (the series or quote to pull when the model is repopulated), and dates and access timestamps are recorded in the Notes tab. Market inputs (spot, forward, rates, premiums) are the only cells to adjust for scenario work.
 
### 2.1 Core Inputs
 
| Standardized Name | Description                                         | Unit        | Value                | Stage 4 source                                        |
| ----------------- | --------------------------------------------------- | ----------- | -------------------- | ----------------------------------------------------- |
| `FC_AMT`          | Foreign-currency notional (receivable)              | EUR         | 2,200,000            | Scenario 36 case facts                                |
| `S0_in`           | Spot exchange rate at inception                     | USD per EUR | 1.1204               | Yahoo Finance, EURUSD=X, dated quote                  |
| `F0_in`           | Forward rate to settlement                          | USD per EUR | 1.1705               | Scenario 36 quote sheet, 182-day forward              |
| `R_USD`           | USD interest rate to settlement                     | Annual %    | 4.18%                | CME Term SOFR 6-month (Global-Rates.com)              |
| `R_FC`            | EUR interest rate to settlement                     | Annual %    | 3.01%                | 6-month Euribor (euribor-rates.info)                  |
| `T_DAYS`          | Days to settlement                                  | Days        | 182                  | Scenario 36 case facts                                |
| `BASIS`           | Day-count denominator (USD and EUR both ACT/360)    | Days        | 360                  | Market convention, ACT/360                            |
| `K_PUT`           | Put option strike (at spot, per scenario)           | USD per EUR | 1.1204               | Scenario 36 quote sheet, 182-day put                  |
| `PREM_PUT`        | Put premium, USD per 1 EUR ($42,680 ÷ 2,200,000)    | USD per EUR | 0.0194               | Scenario 36 quote sheet, 182-day put                  |
| `K_CALL`          | Call option strike (at spot, per scenario)          | USD per EUR | 1.1204               | Scenario 36 quote sheet, 182-day call                 |
| `PREM_CALL`       | Call premium, USD per 1 EUR ($65,120 ÷ 2,200,000)   | USD per EUR | 0.0296               | Scenario 36 quote sheet, 182-day call                 |
 
### 2.2 Derived / Intermediate Values
 
| Name            | Description                                  | Formula                                   |
| --------------- | -------------------------------------------- | ----------------------------------------- |
| `DF_USD`        | USD accumulation factor                      | `1 + R_USD × T_DAYS / BASIS` = 1.02113    |
| `DF_FC`         | EUR accumulation factor                      | `1 + R_FC × T_DAYS / BASIS` = 1.01522     |
| `FV_PREM_PUT`   | Future value of put premium at settlement    | `−PREM_PUT × FC_AMT × DF_USD` ≈ −$43,582  |
| `FV_PREM_CALL`  | Future value of call premium at settlement   | `−PREM_CALL × FC_AMT × DF_USD`            |
| `S_T_grid`      | Sensitivity spot grid at settlement          | `S0_in` ± 5% in 1% steps (11 rows)        |
| `USD_NO_HEDGE`  | USD proceeds under no hedge                  | `S_T × FC_AMT`                            |
 
---
 
## 3. Assumptions & Constraints
 
- **Quote convention:** All rates are **USD per euro**. A higher quote means the euro has appreciated, which is good for a euro receivable.
- **Horizon:** Single maturity, `T_DAYS = 182`.
- **Day-count basis:** USD and EUR money-market quotes are both ACT/360, so one `BASIS = 360` named range is correct for both legs here. (The rigorous `BASIS_USD` / `BASIS_FC` split only matters for GBP.)
- **Parity:** The money-market hedge should replicate the forward under covered interest parity. Any gap is a test of parity in the quoted inputs, not a model error. Here the quoted forward (1.1705) sits about 4.35 cents above the parity forward (about 1.127), so a gap is expected and is itself a finding.
- **Option premium:** Paid upfront in USD, quoted per 1 EUR (no contract multiplier), treated as a negative cash flow at t₀ and carried forward at `R_USD`. This applies to both the put and the call.
- **Counterparty / credit risk:** Excluded. All derivatives are assumed frictionless and creditworthy.
- **Transaction costs & bid-ask spreads:** Excluded from the base case; flagged as a sensitivity candidate.
- **Tax / accounting treatment:** Excluded. Pre-tax cash outcomes only.
- **Scenario construction:** `S_T` is varied deterministically across a grid, with no probability weights and no implied-volatility distribution.
---
 
## 4. Calculation Flow
 
Named-range pseudocode, portable across Excel, Python, and AI prompts. Steps 1 to 6 are for a **receivable**. Step 7 is the mirror **payable view**.
 
### Step 1 — Derived inputs
 
1. `DF_USD` = `1 + R_USD × T_DAYS / BASIS`
2. `DF_FC` = `1 + R_FC × T_DAYS / BASIS`
3. `FV_PREM_PUT` = `−PREM_PUT × FC_AMT × DF_USD`
4. `FV_PREM_CALL` = `−PREM_CALL × FC_AMT × DF_USD`
### Step 2 — Forward hedge (certainty benchmark)
 
- `USD_FWD` = `FC_AMT × F0_in` = $2,575,100
- Locked in at t₀; constant across the `S_T` grid.
### Step 3 — Money-market hedge (parity check)
 
1. Borrow `FC_AMT / DF_FC` euros today (the present value of the receivable).
2. Convert to USD at spot: `(FC_AMT / DF_FC) × S0_in`.
3. Invest the USD to maturity: `USD_MM` = `(FC_AMT / DF_FC) × S0_in × DF_USD` ≈ $2,479,240.
4. At maturity the euro receivable repays the euro loan exactly, so the USD deposit is the locked-in proceed.
> **Parity check:** `USD_MM` implies a parity forward of `USD_MM / FC_AMT` ≈ 1.127. The quoted forward of 1.1705 beats this by about $96,000. This is the gap the Stage 5 memo has to explain.
 
### Step 4 — Option hedge (floor with upside)
 
Put-and-hold on the receivable. Pay `PREM_PUT × FC_AMT` = $42,680 today for a put on EUR with strike `K_PUT`. At settlement, for each `S_T` on the grid:
 
- `USD_PUT(S_T)` = `MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT`
### Step 5 — Sensitivity table (one row per grid point)
 
| Column                             | Output                | Formula                                             |
| ---------------------------------- | --------------------- | --------------------------------------------------- |
| No hedge                           | `USD_NO_HEDGE(S_T)`   | `S_T × FC_AMT`                                      |
| Forward                            | `USD_FWD`             | constant across rows                                |
| Money market                       | `USD_MM`              | constant across rows                                |
| Option (put)                       | `USD_PUT(S_T)`        | `MAX(S_T, K_PUT) × FC_AMT + FV_PREM_PUT`            |
| Hedge profit                       | `HEDGE_PROFIT_k(S_T)` | `USD_k − USD_NO_HEDGE`, one sub-column per strategy |
| Overall winner (incl. no hedge)    | label                 | `ARGMAX(USD_NO_HEDGE, USD_FWD, USD_MM, USD_PUT)`    |
| Best active hedge (excl. no hedge) | label                 | `ARGMAX(USD_FWD, USD_MM, USD_PUT)`                  |
 
### Step 6 — Summary metrics
 
- `USD_FLOOR_PUT` = `MIN(USD_PUT)` across `S_T_grid` (the minimum the put guarantees, net of premium) = `K_PUT × FC_AMT + FV_PREM_PUT` ≈ $2,421,298.
- `S_BREAKEVEN_PUT_FWD` = `F0_in − FV_PREM_PUT / FC_AMT` ≈ 1.1903, the spot at settlement above which the put outperforms the forward (about 6.2% above spot, outside the ±5% grid).
- `USD_BASE_k` = `USD_k` evaluated at `S_T = S0_in` for each strategy (feeds §5.1).
### Step 7 — Payable view (mirror case, includes the call)
 
If the firm instead **owed** `FC_AMT` euros at settlement, the risk flips: a stronger euro raises the USD cost. Costs are shown as negative USD cash flows so the sign convention matches the receivable.
 
- **Forward (payable):** `USD_FWD_PAY` = `−FC_AMT × F0_in`. Locked in at t₀.
- **Money market (payable):** buy `FC_AMT / DF_FC` euros at spot today, funded by borrowing USD, and invest the euros to maturity. `USD_MM_PAY` = `−(FC_AMT / DF_FC) × S0_in × DF_USD`.
- **Call option (payable):** pay `PREM_CALL × FC_AMT` today for a call on EUR with strike `K_CALL`. At settlement, for each `S_T` on the grid:
  - `USD_CALL(S_T)` = `−MIN(S_T, K_CALL) × FC_AMT + FV_PREM_CALL`
- **Call cap:** `USD_CAP_CALL` = `MAX(USD_CALL)` across `S_T_grid`, which is the worst-case cost the call allows, net of premium = `−K_CALL × FC_AMT + FV_PREM_CALL`.
- **Payable no hedge:** `USD_NO_HEDGE_PAY(S_T)` = `−S_T × FC_AMT`.
---
 
## 5. Outputs
 
| Output                           | Description                                                  | Format               | Purpose                  |
| -------------------------------- | ------------------------------------------------------------ | -------------------- | ------------------------ |
| Input panel                      | All named-range inputs with units, sources, and access dates | Top of each tab      | Single source of truth   |
| Strategy summary                 | `USD_FWD`, `USD_MM`, `USD_BASE_k`, and `USD_FLOOR_PUT`       | Table above the grid | Executive at-a-glance    |
| Sensitivity table                | USD proceeds for each strategy across `S_T_grid` ± 5%        | Table                | Core analytical evidence |
| Hedge-profit columns             | `USD_k − USD_NO_HEDGE` per strategy per row                  | Sub-table            | Isolates hedge value-add |
| Winner / best-hedge labels       | `ARGMAX` labels per row                                      | Two label columns    | Quick-read decision cue  |
| Sensitivity chart                | Line chart of USD proceeds vs. `S_T`, all four strategies    | Embedded chart       | Visual comparison        |
| Payable-view table               | `USD_FWD_PAY`, `USD_MM_PAY`, `USD_CALL(S_T)`, `USD_CAP_CALL` | Table on Options tab | Mirror case with the call |
| Executive summary (Stage 5 memo) | 1–2 paragraph narrative with explicit recommendation         | Separate memo        | Downstream deliverable   |
 
### 5.1 Computed Base-Case Values
 
Hand-computed at `S_T = S0_in = 1.1204`. These are the regression checkpoint for the Stage 3 build.
 
| Strategy     | USD Proceeds (Receivable)                                  | Hedge Profit vs. No Hedge |
| ------------ | ---------------------------------------------------------- | ------------------------- |
| No hedge     | $2,464,880                                                 | —                         |
| Forward      | $2,575,100                                                 | $110,220                  |
| Money market | ≈ $2,479,240                                               | ≈ $14,360                 |
| Option (put) | ≈ $2,421,298 (equals the put floor, because spot = strike) | ≈ −$43,582                |
 
### 5.2 Payable-View Base-Case Values
 
Hand-computed at `S_T = S0_in = 1.1204`, as USD cost (negative). The call is bought at strike = spot, so at the base case it sits exactly at its cap.
 
| Strategy     | USD Cost (Payable)                                      |
| ------------ | ------------------------------------------------------- |
| No hedge     | −$2,464,880                                             |
| Forward      | −$2,575,100                                             |
| Money market | ≈ −$2,479,240                                           |
| Option (call)| ≈ −$2,531,376 (equals the call cap, because spot = strike) |
 
---
 
## 6. Model Review — What Worked & What to Improve
 
This spec is written before the build, so §6 is a known-risks register.
 
### 6.1 What Worked
 
- The forward ($2,575,100) and no-hedge ($2,464,880) figures follow directly from `FC_AMT`, `F0_in`, and `S0_in` and reproduce by hand.
- The money-market hedge reproduces from the 4.18% and 3.01% rates at ACT/360 and lands within about $100 of the teacher's reference figure of $2,479,300, which supports the day-count choice.
### 6.2 What to Improve
 
- **Parity gap:** the quoted forward is about $96,000 above the money-market result. The build must show this gap explicitly rather than force `USD_FWD = USD_MM`, and the memo must say whether the quote is realistic or the inputs need re-checking.
- **Named ranges:** every formula uses the §2.1 names, with no bare cell references.
- **Chart:** build the four-strategy line chart with the put's kink visible at `K_PUT`.
- **Dated sources:** record the source and access date for spot, forward, both rates, and both option premiums in the Notes tab.
### 6.3 Independent Check (before the Stage 5 memo)
 
Before the memo is written, the model is checked in two ways:
 
1. **Hand recomputation:** I recompute `USD_FWD`, `USD_MM`, `USD_PUT` at the strike, and `FV_PREM_PUT` in a separate sheet with plain formulas, and compare them to the model outputs. Any difference over 0.05% is traced to its cause before proceeding.
2. **Outside reference:** I compare the parity forward (about 1.127) with a published market forward for the same tenor, and ask a second reader (a classmate or a fresh AI session) to rebuild the base case from this spec alone. If their numbers do not match §5.1, the spec is unclear and gets fixed.
### 6.4 Auditability Checklist
 
- [ ] Every input has a standardized named range from §2.1
- [ ] Every formula in §4 uses named ranges, with no bare `$F$n` references
- [ ] Money-market hedge compared with forward hedge and the gap reported (parity check)
- [ ] Put payoff at `S_T = K_PUT` equals `K_PUT × FC_AMT + FV_PREM_PUT` (kink verification)
- [ ] Call cost at `S_T ≥ K_CALL` equals `−K_CALL × FC_AMT + FV_PREM_CALL` (cap verification)
- [ ] Sensitivity grid is symmetric around `S_T = S0_in`
- [ ] Notes tab records spot / forward / rate / premium sources with access dates
- [ ] Cell colors match the legend: yellow inputs, blue assumptions, black formulas, green cross-tab links
- [ ] No cell on any tab shows an error value (`#DIV/0!`, `#REF!`, `#N/A`, `#VALUE!`, `#NAME?`)
- [ ] Every output cell is a formula, with no hard-coded results
- [ ] Independent check in §6.3 completed and results logged
---
 
## 7. Sensitivity Plan
 
- **Grid:** `S_T_grid` spans `S0_in × (1 ± 5%)` in 1% increments, giving 11 rows including the baseline.
- **Strategies plotted:** no hedge, forward, money market, put option.
- **Primary chart:** `S_T` on the x-axis, USD proceeds on the y-axis. Forward and money-market lines are horizontal, no-hedge is a straight line, and the put is piecewise-linear with a kink at `K_PUT`.
- **Secondary table:** hedge profit vs. no hedge for each strategy.
- **Payable view:** the same grid is applied to the payable table in Step 7, so the call's cap at `K_CALL` is visible.
- **What the chart should communicate:** the trade-off between **certainty** (forward, money market), **optionality** (put), and **naked exposure** (no hedge).
---
 
## 8. Tab Architecture
 
The workbook has nine tabs, in this order. Each tab reads inputs only from the Inputs tab through named ranges.
 
| # | Tab            | Purpose                                                                                                         |
| - | -------------- | --------------------------------------------------------------------------------------------------------------- |
| 1 | `Cover`        | Title, author, date, version, LLM used, and a list of data sources with access dates.                           |
| 2 | `Legend`       | Cell color key (yellow inputs, blue assumptions, black formulas, green cross-tab links) and the named-range key. |
| 3 | `Inputs`       | All §2.1 inputs as named ranges, plus the §2.2 derived values, with units and Stage 4 sources.                  |
| 4 | `Forward`      | Forward hedge outcome (§4 Step 2) and its payable mirror.                                                       |
| 5 | `MoneyMarket`  | Three-step money-market hedge (§4 Step 3), the parity forward, and the gap to the quoted forward.               |
| 6 | `Options`      | Put payoff and floor (§4 Step 4), and the call payoff and cap in the payable view (§4 Step 7).                  |
| 7 | `Sensitivity`  | The 11-row grid, hedge-profit columns, winner labels, the line chart, and the break-even (§4 Steps 5 and 6).    |
| 8 | `Checks`       | The §5.1 base-case comparison, the 0.05% tolerance test, the no-error rule, and the all-formulas rule.          |
| 9 | `Notes`        | Dated sources, assumptions from §3, and anything not covered on another tab.                                    |
 
---
 
## 9. Limitations & Next Steps
 
**Limitations.** This specification does not incorporate:
 
- Partial, layered, or dynamic hedging (static, full-notional hedge at t₀)
- Credit, counterparty, and settlement risk
- Implied-volatility-based option pricing (the premiums are scenario inputs)
- Hedge accounting (ASC 815 / IFRS 9)
- Multi-currency or multi-horizon portfolio effects
**Next steps.** Stage 3 builds the workbook from §4 and §8 and checks it against §5.1. Stage 4 repopulates it with refreshed market inputs from the sources in §2.1. Stage 5 runs the independent check in §6.3 before the decision memo, then writes the final recommendation to the CFO.
 
---
 
## Appendix A — Change Log
 
| Version | Date       | Author          | Change                                                                                                                                                                                                  |
| ------- | ---------- | --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 0.1     | 2026-11-06 | Alexis Yorimoto | Initial draft                                                                                                                                                                                           |
| 0.2     | 2026-11-12 | Alexis Yorimoto | Moved spec to `capabilities/fx-hedging/spec.md`. Added `K_CALL`, `PREM_CALL`, `FV_PREM_CALL` and the payable view with the call (§4 Step 7, §5.2). Added a Stage 4 source column to §2.1. Added the tab architecture (§8). Added the no-error and all-formulas rules to §6.4. Fixed `PREM_PUT` unit to USD per EUR, fixed the version mismatch, and fixed the §9 formatting. |
 
