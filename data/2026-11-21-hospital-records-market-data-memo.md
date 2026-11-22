# Market-Data Memo: Hospital Records FX Hedge (Stage 4)
 
**Scenario:** hospital-records (Scenario 36). EUR 2,200,000 receivable, 182 days to settlement.
**Author:** Alexis Yorimoto, Treasury Analyst. 
**Date:** 2026-11-21. 
**Market data as of:** Friday 2026-11-20 close (last session before the date I began).
**Workbook:** `capabilities/fx-hedging/model.xlsx`, populated through the named-range input cells.
 
## 1. Provenance table
 
| Input | Value | Source | Retrieved / as of | Proxy or computation |
| --- | --- | --- | --- | --- |
| `S0_in` EURUSD spot | **1.0971** | EUR/USD previous close, 20/11/2026 23:55 (moneyswapp.com) | Retrieved 2026-11-21; close of 2026-11-20 | Cross-checks: Yahoo Finance EUR=X close 0.9116 (inverse, implies 1.0970), Macrotrends 1.0973, Trading Economics 1.0967, ECB reference fix 1.0990 (14:15 CET, not a close). |
| `R_USD` | **4.3838%** | CME Term SOFR 6-month, 2026-11-20 = 4.38380% (Global-Rates.com, CME data) | Retrieved 2026-11-21; value dated 2026-11-20 | None. |
| `R_FC` | **3.043%** | 6-month Euribor, 2026-11-19 = 3.043% (Global-Rates.com, EMMI data) | Retrieved 2026-11-21; value dated 2026-11-19 | The 2026-11-20 fixing was not yet posted on the page, so the latest available value is used (one business day older than `R_USD`). |
| `F0_in` forward | **1.1705** | Scenario 36 quote sheet, 182-day forward (the bank's quote) | Given with the case | **Kept as quoted.** CIP-implied forward computed in §3 for comparison. |
| `K_PUT`, `K_CALL` | **1.0971** each | Set at live spot, per the scenario's at-the-money convention | 2026-11-21 | Equal to `S0_in`. |
| `PREM_PUT` | 0.0194 USD/EUR ($42,680) | Scenario 36 quote sheet | Given with the case | **Assumption: scenario premium kept**, not re-priced. |
| `PREM_CALL` | 0.0296 USD/EUR ($65,120) | Scenario 36 quote sheet | Given with the case | **Assumption: scenario premium kept**, not re-priced. |
| `FC_AMT`, `T_DAYS` | EUR 2,200,000; 182 days | Scenario 36 case facts | Given with the case | None. |
| `BASIS` | 360 | ACT/360 market convention, USD and EUR | n/a | None. |
 
**Changes from the placeholders:** spot 1.1204 → 1.0971 (−2.08%); `R_USD` 4.18% → 4.3838%; `R_FC` 3.01% → 3.043%; both strikes 1.1204 → 1.0971.
 
## 2. Rate choice
 
I used the **6-month** rates, not 1-year, because the scenario's tenor is 182 days and a 1-year rate would not match the cash flow.
- **USD:** CME Term SOFR 6-month is the standard forward-looking USD benchmark for that tenor and is quoted ACT/360, matching `BASIS`.
- **EUR:** 6-month Euribor is the matching EUR interbank benchmark, also ACT/360.
- Both are unsecured/benchmark rates rather than government yields. That is closer to what a bank uses to price a forward.
## 3. CIP-implied forward versus the bank's quote
 
```
F0 = S0 × (1 + R_USD × T/360) / (1 + R_FC × T/360)
   = 1.0971 × 1.022163 / 1.015384 = 1.10442
```
 
| | Rate (USD per EUR) | USD on EUR 2,200,000 |
| --- | --- | --- |
| Bank quote (`F0_in`) | 1.1705 | $2,575,100 |
| CIP-implied forward (`F_PARITY`) | 1.10442 | $2,429,733 |
| **Gap (quote − CIP)** | **0.06608** (about 6.0% of the CIP forward) | **$145,367** |
 
The quote sits 6.7% above live spot, while the interest-rate differential supports a forward only about 0.7% above spot. I did not change any input to close this. The gap is larger than the roughly $96,000 gap at the placeholder inputs, because spot fell about 2% while the quote did not move. Stage 5 should treat it as the main finding. The quote looks stale against live spot, so I am asking the instructor (acting as the bank) whether to request a requote.
 
## 4. Workbook population and structural fixes
 
- Live values were entered only in `Inputs!D7`, `D9`, `D10`, `D13`, `D15`. No formula was edited.
- Recalculation: **497 formulas, 0 errors.** `CHECKS_STATUS` = "ALL CHECKS PASS (1 item pending)". The pending item is the outside-reference check (`MKT_FWD_REF` is blank, see §5).
- Text-only provenance cells were updated to the real sources and dates (Inputs column E, Cover data-sources block, Notes dated sources).
- **No structural fix was needed.** The two issues I expected from the Stage 3 audit did not arise:
  - The put-floor check passes because the strikes are set equal to live spot, so the strike is inside the grid.
  - The regression checks pass because they recompute from current inputs.
- **Known stale item:** the `CHK_*` checkpoint block on Inputs (rows 34–45) holds the spec §5.1/§5.2 values for the original case inputs. It is labeled documentation-only and the Checks tab does not use it, so I left it unchanged.
## 5. Results at live inputs (receivable, base case S_T = spot)
 
| Strategy | USD proceeds | Hedge profit vs no hedge |
| --- | --- | --- |
| No hedge | $2,413,620 | n/a |
| Forward (at quote) | $2,575,100 | +$161,480 |
| Money market | $2,429,733 | +$16,113 |
| Put option | $2,369,994 (equals the floor) | −$43,626 |
 
- Put break-even versus the forward: spot at settlement of about 1.1903, which is 8.5% above spot and outside the ±5% grid.
- At every one of the 11 grid points the forward is the best strategy, because the quote is far above what the money-market hedge can replicate.
- Payable view: call cap −$2,480,183; forward −$2,575,100; money market −$2,429,733.
## 6. FX Lab cross-check (to complete before submitting)
 
Enter the live inputs into the [FX Hedging Lab](https://adamwstauffer.github.io/ai-lms/fxlab.html) and compare these workbook outputs: forward $2,575,100, CIP forward 1.10442, money market $2,429,733, put floor $2,369,994.
- **Result:** _not yet run. Record the Lab's figures and any discrepancy here, with the resolution._
## 7. Open items and limitations
 
- **Outside reference (§6.3 item 2):** no published 182-day EURUSD forward was found from a reliable source, so `MKT_FWD_REF` is blank and that check stays PENDING.
- **Spot source:** the spec names Yahoo `EURUSD=X`. The page I retrieved was the inverse pair `EUR=X`. I used the 1.0971 close and list Yahoo as a cross-check. Re-pull before submitting.
- **Rate dating:** `R_FC` is dated one business day before `R_USD` (see §1).
- **Premiums:** scenario premiums were set when spot was 1.1204. At-the-money premiums at the new spot would differ, so the put and call costs are an assumption, not a market price.
 
