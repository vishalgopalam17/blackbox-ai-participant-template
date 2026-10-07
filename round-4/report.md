# Round 4 — Reconstruct

**Team:** BB-032  
**Queries used:** 50 / Round 1 
**Round 4 unseen queries used:** 0

## What we concluded

We reconstructed the observed behavior of the GK-05 black-box scoring model using the queries collected during Round 1 and Round 2.

The main findings are:

- `history_score` has a noticeable negative effect on the GK-05 score.
- `requested_zone` has a strong negative effect on the score.
- `site` has a strong effect, with different sites producing substantially different scores.
- `linked_badges` appears to have a negative effect on the score.
- `recent_denials` appears to reduce the score.
- `badge_age_days` appears to have a moderate negative effect.
- The observed decision boundary is clearly separated in our collected data:
  - Highest observed `DECLINE` score: `0.4415`
  - Lowest observed `APPROVE` score: `0.8032`
- Therefore, the exact internal decision threshold cannot be determined from the collected queries, but any threshold between these two values reproduces all observed decisions.

Our reconstruction is intended as an approximation of the black-box behavior rather than a claim that the original GK-05 source code or exact formula has been recovered.

## How we got there

We used the observed GK-05 queries from Round 1 and Round 2 as our training evidence.

The input features were:

1. `anomaly_ratio`
2. `badge_age_days`
3. `clearance_level`
4. `escorts`
5. `history_score`
6. `linked_badges`
7. `recent_denials`
8. `requested_zone`
9. `site`
10. `tenure_years`

The target was the continuous GK-05 `score`.

The categorical `site` feature was converted using one-hot encoding.

We used a Random Forest regression model to learn the relationship between the observed inputs and GK-05 scores.

The model was trained using the observed Round 1 and Round 2 data. The Round 4 queries are treated as unseen evaluation data and are not used for training.

### Evidence from observed queries

#### 1. History score

`history_score` showed a clear negative relationship with the output score.

For example:

- Query 39: `history_score = 350`, score = `0.9911`
- Query 38: `history_score = 500`, score = `0.9842`

Another comparison:

- Query 18: `history_score = 500`, score = `0.9597`
- Query 17: `history_score = 700`, score = `0.9317`

This suggests that increasing `history_score` generally decreases the GK-05 score.

#### 2. Requested zone

`requested_zone` showed a strong negative relationship with score.

For example, queries 31–35 use very similar inputs while changing the requested zone:

- Zone 35 → score `0.9836`
- Zone 36 → score `0.9835`
- Zone 37 → score `0.9842`
- Zone 38 → score `0.9832`
- Zone 39 → score `0.9827`

A larger change is visible when comparing:

- Query 26: zone 40 → score `0.9822`
- Query 25: zone 50 → score `0.9561`

This indicates that higher requested zones can substantially reduce the score.

#### 3. Site

Site was one of the strongest observed effects.

Queries 3–6 have the same major numeric inputs but different sites:

| Query | Site | Score |
|---|---|---:|
| 3 | A | 0.4335 |
| 4 | B | 0.0320 |
| 5 | C | 0.4375 |
| 6 | D | 0.4415 |

The large difference for site B indicates that the site cannot be treated as an irrelevant feature.

#### 4. Linked badges

Increasing `linked_badges` produced lower scores in a controlled group:

- 18 linked badges → `0.9842`
- 19 linked badges → `0.9833`
- 20 linked badges → `0.9796`

This suggests a negative effect from linked badges.

#### 5. Recent denials

Recent denials also appear to reduce the score.

For example:

- Query 21: `recent_denials = 0`, score = `0.9714`
- Query 20: `recent_denials = 2`, score = `0.9663`

This supports the hypothesis that recent denials negatively affect the output.

## What we ruled out

Based on the available queries, we ruled out the assumption that all input features contribute equally to the output.

We also ruled out treating `site` as a simple irrelevant categorical label. The observed scores show substantial site-dependent behavior.

We did not find evidence that the decision can be explained by a simple fixed rule based on only one feature.

We also avoided assuming that the observed `0.5` threshold is the exact internal GK-05 threshold. The data only shows that the actual decision boundary lies somewhere between the highest observed DECLINE score (`0.4415`) and the lowest observed APPROVE score (`0.8032`).

We did not claim that Random Forest has recovered the exact original GK-05 algorithm. It is a learned approximation based on observed black-box queries.

## What we are still unsure about

Several effects cannot be identified with high confidence because only 50 Round 1 and Round 2 queries are available.

In particular:

- The exact mathematical formula used by GK-05 is unknown.
- The exact decision threshold is unknown.
- The individual effect of `clearance_level` is not sufficiently isolated in the available queries.
- The individual effect of `escorts` is relatively weak or unclear from the available comparisons.
- Possible interactions between multiple features are not fully identified.
- The behavior of the model outside the ranges and combinations explored in Round 1 and Round 2 is uncertain.
- The Random Forest approximation may not exactly reproduce GK-05 scores for previously unseen inputs.

## Round 4 evaluation

The 80 queries allocated for Round 4 are treated as unseen data.

They must not be used to train or tune the reconstruction model.

The final evaluation should therefore follow this process:

1. Train using all available Round 1 and Round 2 queries.
2. Freeze the trained reconstruction model.
3. Apply it to the 80 Round 4 queries.
4. Compare the predicted scores and decisions with the actual GK-05 outputs.
5. Report the resulting evaluation metrics.

This prevents information from the Round 4 evaluation set from leaking into the training process and gives a more meaningful measure of generalization.

## Summary

Our current reconstruction indicates that GK-05 is influenced by multiple features rather than a single simple rule.

The strongest observed effects are associated with:

- `requested_zone`
- `site`
- `history_score`

with additional evidence for effects from:

- `linked_badges`
- `recent_denials`
- `badge_age_days`

The current model is therefore a data-driven approximation of the GK-05 black box. The final conclusion about generalization will be made only after evaluating the frozen model against the 80 unseen Round 4 queries.
