# Build audit: Hospital Records FX hedge workbook (2026-11-13)
 
Workbook: `capabilities/fx-hedging/model.xlsx`, built by Claude from spec v0.2.
Audit method: scripted checks plus a LibreOffice recalculation. I have not yet opened the file in Excel, so the Name Manager and chart look still need a manual pass.
 
## Results against the audit questions
 
- **Named ranges exist:** 53 defined names, including all 11 §2.1 inputs (`FC_AMT`, `S0_in`, `F0_in`, `R_USD`, `R_FC`, `T_DAYS`, `BASIS`, `K_PUT`, `PREM_PUT`, `K_CALL`, `PREM_CALL`). None missing.
- **Calculated cells are formulas:** 0 typed numbers on Forward, MoneyMarket, Options, Sensitivity and Checks. No formula on those tabs points directly at the Inputs tab, only through names.
- **Changing `S0_in` moves the table:** at 1.1204 the grid centre is 1.1204 with no-hedge proceeds of $2,464,880. At 1.20 the centre is 1.2000 with $2,640,000, the money-market line rises to $2,655,382, and the winner labels change.
- **Recalculation:** 497 formulas, 0 errors, at base inputs. Checks tab status: ALL CHECKS PASS (1 item pending).
## Findings
 
1. **Spec defect: call cap uses MAX instead of MIN (fixed in workbook, not yet in spec).**
   §4 Step 7 says `USD_CAP_CALL = MAX(USD_CALL)`. Payable costs are negative, so MAX returns the cheapest outcome (about -$2,408,132). The worst case is about -$2,531,376, which equals the spec's own closed form and its §5.2 value. The workbook uses MIN. The spec must be edited at the source (MAX to MIN in Step 7, and the §6.4 call-cap wording), then the workbook regenerated.
2. **Spec count mismatch: "ten named ranges".**
   §2.1 lists 11 inputs and the spec has no "build contract" section. The build named all 11 and treated §2 to §8 as the contract. The spec should state the count and name the contract section.
3. **Regression checks are pinned to the base case.**
   After the `S0_in = 1.20` change, 9 checks fail (§5.1 and §5.2 comparisons and the parity forward). This is correct behaviour, because those reference values are hand-computed for the original inputs, but the spec should say they are valid only at the case inputs.
4. **Put-floor check gives a false failure when spot moves.**
   `K_PUT` and `K_CALL` are separate inputs ("at spot, per scenario"), not linked to `S0_in`. At `S0_in = 1.20` the strike sits below the grid, so MIN over the grid no longer equals the closed-form floor and "Put kink / floor" fails. The spec should either define strikes as `S0_in` or make the check apply only when the strike is inside the grid.
5. **Source access dates are missing.**
   §6.2 requires an access date for spot, forward, both rates and both premiums. The spec gives none, so the workbook says "not recorded yet" and `MKT_FWD_REF` is blank, leaving the outside-reference check PENDING.
6. **Parity gap is real.**
   Parity forward is 1.12693 against a quoted 1.1705, a gap of about $95,859. This matches the spec's "about $96,000" and is reported, not forced to zero. The money-market result of $2,479,241 is about $59 from the teacher figure of $2,479,300.
## Next steps
 
1. Edit the spec on GitHub for findings 1 to 5, then give the updated spec to Claude and regenerate. Do not patch the workbook by hand.
2. Open `model.xlsx` in Excel: check Name Manager, the chart (put kink visible at the strike), and that no repair prompt appears.
3. Record the access dates and enter a published forward in `MKT_FWD_REF`.
