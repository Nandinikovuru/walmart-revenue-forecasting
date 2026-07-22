# Does FRED retail-sales predict Walmart revenue?

**To:** PM, retail subsector
**From:** X Y
**Date:** 2026-05-13

## The question
Can the monthly FRED retail-sales index (RSXFS) be used as a leading indicator for Walmart's quarterly revenue? Specifically, does it predict better than a naive baseline?

## Short answer
No. Once I compare it to a proper baseline, the signal doesn't help.

I ran an expanding-window backtest across 39–40 fiscal quarters from 2016-Q2 to 2026-Q1 (using Walmart's fiscal calendar, which ends in Jan/Apr/Jul/Oct, not the calendar quarters). At every test quarter I re-fit the model on all prior data, so there's no peeking at the future. The "drift" baseline simply predicts each quarter's YoY growth as the mean YoY growth observed so far. That turns out to be very hard to beat.

| Model | MAE (pp) | Dollar MAPE | Skill vs. drift baseline |
|---|---:|---:|---:|
| Seasonal naive (0% YoY)                            | 4.04 | 3.83% | −75% |
| **Seasonal naive + drift (B2)**                    | **2.31** | **2.20%** | 0% (reference) |
| OLS on lag-1 retail YoY (true leading indicator)   | 2.80 | 2.67% | **−21%** |
| OLS on concurrent retail YoY                       | 2.56 | 2.43% | −11% |

Excluding COVID test quarters (2020), the lag-1 model is still 12% worse than the baseline. The concurrent variant ties the baseline ex-COVID, but it isn't actually a leading indicator: FRED publishes RSXFS with a roughly 6-week lag, so concurrent-quarter retail data is only available *after* Walmart's quarter has ended, just before the 10-Q drops.

The full-sample regression confirms the weakness: β = 0.062 on lag-1 retail YoY (t = 0.75, R² = 0.010). Contemporaneous correlation is 0.055; lag-1 correlation is 0.099. Both series move, but they barely move together once you account for Walmart's slow-moving growth.

## Why the baseline is so hard to beat
Walmart's YoY growth has hovered around 4–5% with low quarter-to-quarter dispersion outside of 2020. The variance the drift baseline misses is largely Walmart-specific stuff: e-commerce expansion, pricing decisions, mix shifts, store openings. A broad U.S. retail index can't see any of that.

## What I'd actually worry about

1. **Sample size.** 39–40 out-of-sample quarters isn't much. The 12–21% gap I'm reporting is well within sampling noise, so don't quote the exact number — just the direction.
2. **Regime sensitivity.** Drop 2020 and the gap closes. Drop 2008–2010 (not in this sample) and it might open up again. The conclusion is conditional on the regimes in the window.
3. **Possibly the wrong feature, not the wrong question.** RSXFS is *total* U.S. retail. Walmart is a specific mix: grocery, general merch, e-commerce. A subsector-matched FRED series (grocery, clothing, electronics) or a Walmart-specific alt-data source would be a fairer test. My result is "this aggregate signal doesn't help," not "no public signal can."
4. **Linear single-feature OLS is simple.** A richer model with lagged Walmart growth plus retail plus a recession dummy would probably match the drift baseline by construction, but I don't see it beating the baseline on a real hold-out.

## What would change my mind
- A subsector-matched FRED series showing ≥ 5% skill over the drift baseline on the same backtest setup.
- A break-detection result showing skill is positive in a stable regime (e.g. 2014–2019) and only loses on average because of one anomaly.
- A multivariate model in which retail YoY remains statistically significant after controlling for lagged Walmart growth and a recession indicator.

## Falsifiable headline
On 2016–2026 out-of-sample fiscal quarters, FRED RSXFS as a single-feature predictor fails to beat a seasonal-naive-plus-drift baseline for Walmart revenue YoY by roughly 12–21% in MAE. The result holds when COVID quarters are excluded.

Full code, figures, and intermediate checks in `analysis.ipynb`. Prompt log in `prompts.md`.
