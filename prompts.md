# Prompt log: YipitData take-home

These are the main back-and-forths I had with my LLM assistant while working on this exercise. I'm not pasting raw chat logs I've collapsed each multi-turn conversation into a single block that captures what I asked, what the assistant came back with, and where I pushed back or it pushed back on me.

I'm including these five because they're the ones where the plan actually changed.

---

### Prompt 1 — Sanity-checking my first plan

> Here's the brief: I need to figure out whether the FRED monthly retail-sales index (RSXFS) is a useful leading indicator for Walmart's quarterly revenue, compared to a naive baseline. My first plan is this:
>
> 1. Load both CSVs.
> 2. Resample the FRED monthly series to quarterly by summing.
> 3. Compute YoY growth on both.
> 4. Fit OLS: `walmart_yoy ~ retail_yoy`.
> 5. Report R² and call it a day.
>
> Does this answer the question, and if not, what am I missing?

**What the assistant came back with:** Not yet at least four things wrong. (1) No baseline. R² in isolation is meaningless, I need to compare against a naive forecast like "this quarter equals same quarter last year." (2) In-sample evaluation. Fitting on the whole history and reporting R² tells me how well the model fits *the past*, nothing about prediction. (3) Calendar mismatch. Walmart's fiscal year ends Jan 31, so its quarters end Jan / Apr / Jul / Oct not the calendar Mar / Jun / Sep / Dec that `resample("QE")` produces. (4) Look-ahead bias. If I use this quarter's retail to predict this quarter's revenue, the feature might not have been available at the time the prediction would have been made.

That essentially gave me my plan, align the calendars first, build proper baselines, do out-of-sample evaluation, be careful about timing. I worked through each of these one by one in the prompts below.

---

### Prompt 2 — The fiscal-calendar alignment (and an early correction)

> OK, calendar first. Walmart's fiscal year ends Jan 31, so the four fiscal quarters end Jan / Apr / Jul / Oct. For each Walmart quarter, I want the matching 3 months of FRED data. Here's my guess:
>
> - Walmart Q ending Jan 31 → FRED Jan + Feb + Mar (the 3 months that include Jan)
> - Walmart Q ending Apr 30 → FRED Apr + May + Jun
> - etc.
>
> Then I aggregate by mean. Right?

**What the assistant came back with:** Both wrong. The fiscal quarter ending Jan 31 *covers* the 3 months Nov / Dec / Jan — those are the months whose sales rolled up into that quarterly revenue number. So the right mapping is the 3 months ending at the quarter-end, not starting from it:

- Walmart Q ending Jan 31 → FRED Nov + Dec + Jan
- Walmart Q ending Apr 30 → FRED Feb + Mar + Apr
- Walmart Q ending Jul 31 → FRED May + Jun + Jul
- Walmart Q ending Oct 31 → FRED Aug + Sep + Oct

It also corrected the aggregation sum, not mean. RSXFS is a *flow* (dollars sold per month), so quarterly retail-sales is the sum of the three months. Using mean would have given me a series with the right shape but the wrong scale, which would have looked fine on YoY growth but flagged as weird if I ever checked dollar values.

I built the alignment function based on this and spot-checked it, Walmart's Q ending Jan 31, 2020 → Nov 2019 + Dec 2019 + Jan 2020. Confirmed.

---

### Prompt 3 — The "leading indicator" question (this is where I changed my approach)

> Now the timing piece. If I'm trying to predict Walmart's quarter ending Apr 30, what data is actually available at:
>
> (a) the start of that quarter (Feb 1)?
> (b) the end of that quarter (Apr 30, before the 10-Q is filed)?
> (c) when the 10-Q is filed (~late May)?
>
> The brief specifically calls this out as look-ahead bias. I want to be honest about which version of "available" I'm building features for.

**What the assistant came back with:** This is where the analysis split into two variants. FRED publishes RSXFS monthly with about a 6-week lag so on Feb 1, the most recent FRED data is for December. On Apr 30, the most recent is February. On late May (10-Q filing time), the most recent is April.

So:

- A **true leading indicator** can only use data from before the target quarter starts. That means lag-1 use the *prior* quarter's retail-sales growth to predict the next quarter's Walmart revenue growth.
- A **concurrent** model uses retail data from the same quarter as the target. But that data only becomes fully known after the quarter has ended and just before Walmart files the 10-Q. That's not "leading" it's "concurrent but earlier than Walmart's own reporting."

This pushed me to run *both* and report them separately, because they answer different real-world questions. A PM would care about the lag-1 version (can I trade ahead of Walmart's earnings?). The concurrent version is interesting for nowcasting but it's not what the brief asks for.

---

### Prompt 4 — Backtest design (assistant got this wrong on the first try)

> Out-of-sample evaluation. My plan: split the data 80/20, train on the first 80%, test on the last 20%. Report MAE on test. Plus the two baselines. Good?

**What the assistant came back with first:** Suggested exactly that a single train/test split at around 2019, then evaluate on 2019–2026.

**Where I pushed back:** That's a single split. It only tests against one regime, and that regime happens to be COVID + post-COVID, which is exactly the weirdest stretch of the sample. The brief specifically calls out time-series-aware cross-validation, and a single split doesn't qualify.

**What we ended up with after my pushback:** An expanding-window backtest. Train on the first 20 quarters, predict the 21st. Then expand to 21 quarters of training, predict the 22nd. Continue to the end. The model gets re-fit at every step. The two baselines get re-computed at every step too (the drift baseline = mean YoY over the training window, which changes as the window grows).

This is more code than a single split, but it's the honest version. It also gives 40 out-of-sample predictions instead of just ~12, which is meaningful when N is this small.

The assistant also flagged a column-order bug in the `add_constant` call — `sm.add_constant` won't add a constant if the input already has one matching column, so I had to use `has_constant="add"` and reorder columns to match the training set's exog. Would have spent a while debugging that on my own.

---

### Prompt 5 — Metric choice, robustness, and writing the memo

> Two things left. First, what's the right primary metric? I was going to use MAPE on YoY. Secondm once I have results, draft a one-page memo for a portfolio manager.

**What the assistant came back with:**

On the metric pushed back on my MAPE-on-YoY plan. The target is YoY *growth*, which crosses zero (2020 Q2 Walmart YoY was near zero, and 2020 Q2 retail YoY went sharply negative). MAPE on a near-zero denominator blows up a single observation can swing the average metric by 100%. Better: MAE in percentage points (just `|y_true - y_pred| * 100`, no division). This is interpretable ("the model misses by 2.3 percentage points on average") and robust. I added dollar MAPE as a secondary metric by translating predicted YoY back into a predicted revenue dollar figure using the matched prior-year quarter the denominator there is large and never near zero, so MAPE works.

On COVID, I asked if I should drop those quarters, dummy them out, or leave them in. Decision: leave them in for the headline number (the PM cares about the real-world result, not the hypothetical clean-regime one), and report a second set of metrics with 2020 quarters excluded. Dummying COVID would introduce a parameter without a clean theory and wouldn't help out-of-sample.

For the memo — assistant drafted a skeleton:

1. Frame the question in one paragraph.
2. Lead with the answer in one sentence.
3. Show the metrics table.
4. Explain why the baseline is hard to beat (one paragraph).
5. List 3–4 worries.
6. End with a falsifiable headline.

I used the skeleton but rewrote the prose myself — the first draft had that polished AI rhythm that screams "this was generated." For the prompt log (this file), the assistant suggested keeping it as collapsed multi-turn summaries rather than raw chat logs, which is what the brief explicitly allows.

---

## Notes on what the assistant got right, where I pushed back, how I checked

The assistant was strongest at flagging the time-series pitfalls upfront it raised the in-sample-evaluation and look-ahead-bias problems in the very first response, before I'd specifically asked. It also corrected my calendar guess (off by 2 months) and my aggregation choice (mean → sum). The `add_constant` column-order debugging would have cost me an hour on my own.

I pushed back twice in ways that materially changed the analysis. First on the backtest: it initially offered a single 80/20 split, which I rejected because the brief explicitly warns against it. We rebuilt as an expanding window. Second on the metric: it offered MAPE on YoY as the headline, and I rejected it because of the zero-crossing issue in 2020. We switched to MAE in percentage points.

To check the assistant's outputs I did three things. I hand-computed YoY growth for the first 3 quarters and matched it against the panel. I spot-checked the calendar alignment by tracing one specific quarter end-to-end (Jan 31 2020 → sum FRED Nov-Dec-Jan). And for the OLS, I re-derived β on a 5-row toy dataset by hand and compared to `statsmodels` output to confirm the regression code wasn't mis-specified.
