---
type: brief
engagement: fx-hedging
capability: fx-hedging
title: Hospital Records Hedge Brief
date: 2026-10-23
status: committed
LLM Used: Claude AI
hypothesis: I expect the quoted 1.1705 forward to beat a money-market hedge by about $96,000, because 1.1705 sits ~4.4 cents above the ~1.127 that USD and EUR interest rates justify.
---

# Executive Memo (Scenario 36)

**Created by:** Alexis Yorimoto  
**Updated by:** Alexis Yorimoto  
**Date Created:** [10/30/2026]  
**Date Updated:** [10/30/2026]  
**Version:** 0.0
**LLM Used:** [Claude AI]

---

## Executive Summary
This project looks at how a US hospital-records software firm can protect itself from exchange rate changes on a EUR 2,200,000 payment due from an Austrian hospital network in 182 days (April 23, 2027). The current EURUSD spot rate is 1.1204, so the payment is worth about $2,464,880 today. If the euro weakens, we receive fewer dollars when the customer pays. The analysis compares three strategies: a forward contract, a money market hedge, and an options hedge. I expect the forward contract to come out ahead, because the forward rate we were quoted (1.1705) is higher than interest rates suggest it should be. The final recommendation will come after the model is built and tested.

---

## Background & Objectives
We bill our Austrian customer in euros, so the amount we are owed stays the same, but what it is worth in dollars changes every day. If the euro falls 5%, we lose about $123,000. If it falls 10%, we lose about $246,000. That comes straight out of profit on a contract we have already sold. The euro hit a 17-month low against the dollar this month, so this risk is real right now.

The main goal is to find the best way to lock in or protect the dollar value of this payment. A second goal is to check whether the forward rate we were quoted is fair compared with current interest rates and option prices.

---

## Methods
To meet these goals, I will compare three ways of managing the risk, step by step.
1. **Collect current market data:** the EURUSD spot rate (1.1204), the quoted forward rate (1.1705), option prices, and 6-month interest rates for both currencies (about 4.18% for USD and 3.01% for EUR).
2. **Calculate the dollars received under each hedge:**
  - *Forward Contract:* sell euros at 1.1705, which gives about $2,575,100 with no uncertainty. *Pro:* a certain budget. *Con:* we give up any gain if the euro rises.
  - *Money Market Hedge:* borrow euros today, convert them to dollars, and earn interest on the dollars. This gives about $2,479,300 with no uncertainty. *Pro:* the same certainty, built from the interest rates. *Con:* it uses our borrowing capacity and has more steps.
  - *Options Hedge:* pay $42,680 up front for the right to sell euros at today's rate. This guarantees at least about $2,421,300 but keeps the upside if the euro rises. *Pro:* protection  plus upside. *Con:* the premium is paid even if we never need it.
3. **Check the forward rate:** compare the quoted forward to what interest rates and option prices imply (about 1.127 to 1.131).
4. **Build and test in Excel:** create a workbook that shows the results under different exchange rates, then check it carefully before making a recommendation.

---

## Limitations & Next Steps
**Limitations:** This analysis assumes the spot rate, forward rate, and option prices we were given are accurate. It ignores bank fees and our own borrowing costs, and it assumes the hospital pays in full and on time. I would test these with more time. I could be wrong if live market data shows a forward rate close to 1.127, which would mean the 1.1705 quote was out of date, or if the finished model shows the money market hedge doing as well as or better than the forward.

**Next Steps:** Before writing the memo, I will have a second source verify the model. I will recompute the forward outcome (EUR 2,200,000 × 1.1705 ≈ $2,575,100), the money-market hedge (about $2,479,300), and the put cost ($42,680) by hand in a spreadsheet, and compare them with the model’s outputs. I will also compare my 1.127 parity forward against a published forward quote from a bank or market data site. If any figure differs by more than 0.5%, I will find the cause before going further. Then I will write the final recommendation.

---

## References
Yahoo! Finance. (2026). EUR/USD (EURUSD=X) live rate, chart & news. https://finance.yahoo.com/quote/EURUSD=X/

CME Group. (2026). 6-month CME Term SOFR interest rate [Data set]. Global-Rates.com. https://www.global-rates.com/en/interest-rates/cme-term-sofr/3/term-sofr-interest-6-months/

Euribor-rates.info. (2026). 6-month Euribor. https://euribor-rates.info/en/euribor-rate/6-month-rate/
