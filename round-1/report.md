# round-1 — Observe

**Team:** BB-011
**Queries used:** 50 / [fill in budget]

## What we concluded

The system returns a score between 0 and 1 and approves or declines on it. It is deterministic: repeated identical queries gave identical scores (Q16 = Q18, Q21 = Q27, Q32 = Q38, Q42 = Q46). Every difference we saw is therefore a real effect of the inputs, not noise.

Effects we found (x1–x10 follow the column order of the query log):

- **x9 (category):** B forces the score down to exactly 0.0320, whatever the other inputs are (Q2, Q4). C and D score slightly higher than A (D > C > A).
- **x2:** higher x2 lowers the score. This held in four separate single-variable comparisons. The effect is small near 40–50 and stronger by 60.
- **x10:** it has a sweet spot. The score rises from 0 to about 25–30, then falls above about 35 (40 is clearly worse than 35–37).
- **x8:** it peaks around 37. Moving from 40 to 50 lowers the score by about 0.013–0.015.
- **x5:** lower is better, down to about 350. Going from 500 to 700 lowers the score by about 0.028. Between 333 and 355 the curve is almost flat.
- **x6:** lower is slightly better in the range we tested (18 beats 19 and 20).
- **x7:** 0 scored slightly higher than 2 (one comparison only).
- **x4:** no effect between 5 and 6.

Best configuration found (Q50, score 0.9922, APPROVE):
x1=0, x2=24, x3=100, x4=6, x5=350, x6=18, x7=0, x8=37, x9=A, x10=34.

The decision looks score-based: every DECLINE scored 0.4415 or lower, and every APPROVE scored 0.8032 or higher.

## How we got there

1. **Baseline (Q1):** a simple input gave APPROVE at 0.8032.
2. **Exploration of the DECLINE region (Q2–Q11):** Q2 changed many inputs at once and gave DECLINE at 0.0320. Q3–Q6 held everything fixed and changed only x9 (A, B, C, D), which isolated the category effect. Q7–Q11 varied x2 and x4 within the DECLINE region.
3. **Sweep of x10 from the baseline (Q12–Q15):** x10 = 10, 20, 30, 40 showed rising scores that levelled off near 30.
4. **New high-scoring setup (Q16):** we moved to a different starting point and got 0.9597. Q18 repeated it and confirmed the score was deterministic.
5. **One-at-a-time changes from that point (Q17–Q25):** we varied x5, x6, x7, x8 and x2 individually and recorded the direction and size of each change.
6. **Fine sweeps (Q26–Q50):** we stepped x10 (15 to 40), x8 (35 to 40), x6 (18 to 20) and x5 (333 to 355) in small increments to find the optimum of each. We finished by nudging x10 and x2 in the best region (Q48–Q50).

## What we ruled out

- **Random noise:** identical queries repeat exactly, so score differences are real.
- **x4 as an influence (in the range tested):** x4 = 5 and x4 = 6 gave identical scores (Q10, Q11).
- **x3 = 100 causing a DECLINE:** every query from Q16 to Q50 has x3 = 100 and was approved.
- **A simple "bigger is better" or "smaller is better" rule for x10 and x8:** both have an interior optimum.
- **The B category as a gradual penalty:** it produces the same score (0.0320) in very different queries, which looks like a floor or override.

## What we are still unsure about

- **x1:** it only changed in Q2, together with many other inputs, so we cannot separate its effect.
- **x3:** we never changed it on its own. Its effect cannot be separated from the other inputs that changed with it.
- **What causes the early DECLINEs (Q3, Q5–Q11):** these had x5 = 900 and also differed in x8 and x3. We cannot say which input is responsible.
- **The decision threshold:** it lies somewhere between 0.4415 (highest DECLINE) and 0.8032 (lowest APPROVE). We never tested scores in that gap.
- **Interactions between inputs:** almost all comparisons changed one input at a time around a few base points, and we tuned inputs one after another. The true joint optimum may differ.
- **x7 and x6:** the x7 effect rests on one comparison, and x6 below 18 was never isolated.
- **x4 outside 5–6:** we do not know whether it matters over a wider range.
- **Whether the B floor holds for every combination of other inputs,** and the exact shape of the x2 effect, since we saw it in only a couple of regimes.
