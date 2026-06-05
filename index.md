---
layout: default
title: Understanding Power Outages (2000–2016)
---

<script src="https://cdn.plot.ly/plotly-2.27.0.min.js"></script>

# Understanding Power Outages (2000–2016)

**Author:** Jayden Wang

---

## Introduction

Large-scale power outages can disrupt essential services, impact millions of customers, and
impose significant economic costs. This project investigates the **U.S. Major Power Outage
dataset**, which records significant outage events across the United States between 2000 and
2016, alongside information about climate conditions, outage causes, affected populations, and
restoration metrics.

The dataset contains **1,534 rows**, where each row represents a single major power outage
event.

- [Download the dataset](https://engineering.purdue.edu/LASCI/research-data/outages)
- [Full dataset description and variable definitions](https://www.sciencedirect.com/science/article/pii/S2352340918307182#s0015)

Our central research question is:

> **Does the severity of global climate anomalies systematically correlate with the scale of
> customer impact during major power outages?**

This question matters because climate phenomena such as El Niño and La Niña influence weather
patterns across the U.S., potentially affecting infrastructure reliability and outage severity.
If stronger climate anomalies associate with larger outages, utility providers may be able to
improve risk assessment during periods of unusual climate activity.

The key columns used throughout this analysis are:

| Column | Description |
|---|---|
| `ANOMALY.LEVEL` | Oceanic Niño Index (ONI) value; positive (> 0.5) = El Niño, negative (< -0.5) = La Niña |
| `CLIMATE.CATEGORY` | Climate phase label: `warm`, `cold`, or `normal` |
| `CUSTOMERS.AFFECTED` | Number of customers impacted — primary measure of outage severity |
| `OUTAGE.START.DATE` | Date the outage began |
| `CAUSE.CATEGORY` | Broad cause of the outage (severe weather, equipment failure, etc.) |
| `U.S._STATE` | State where the outage occurred |

---

## Data Cleaning and Exploratory Data Analysis

### Cleaning Steps

After loading the raw Excel file, we performed the following cleaning steps:

- Dropped the `variables` and `OBS` metadata columns and the blank header row that the
  original spreadsheet uses for formatting
- Combined `OUTAGE.START.DATE` + `OUTAGE.START.TIME` into a single `OUTAGE.START` timestamp,
  and did the same for restoration, making time-series analysis straightforward
- `pd.read_excel` fills empty cells with `NaN` automatically, so no manual replacement was
  needed — missingness is evaluated formally in Step 3

Below is a sample from the cleaned DataFrame. Not all columns are present, and it only shows the 
head (first 5 rows):

<iframe src="assets/df_head.html" width="100%" height="220px"
style="border:none;"></iframe>

### Exploratory Analysis

We begin with a simple look at how climate anomalies are distributed across all outage events.

<iframe src="assets/fig_climate_distribution.html" width="100%" height="450px"
style="border:none;"></iframe>

La Niña (cold) conditions are noticeably more represented than El Niño (warm) conditions. This
raises a natural follow-up question: were there simply more La Niña years during the
2000–2016 window, or do outages genuinely cluster during cold anomaly periods?

<iframe src="assets/fig_climate_time_series.html" width="100%" height="620px"
style="border:none;"></iframe>

El Niño and La Niña periods appear roughly balanced across the timeline, suggesting the cold
skew in the histogram is not simply a reflection of which climate phases happened to occur
more often.

<iframe src="assets/fig_outages_by_year.html" width="100%" height="620px"
style="border:none;"></iframe>

Overlaying annual outage counts against average ONI reveals something striking: a dramatic
spike in outages between 2011 and 2015, coinciding with a La Niña period. However, the spike
is far too large to attribute to climate alone — prior years with similar ONI values show no
comparable surge. We investigate this next.

<iframe src="assets/fig_choropleth.html" width="100%" height="670px"
style="border:none;"></iframe>

No individual state dominates the cold-phase share, ruling out the possibility that a single
region experiencing disproportionate La Niña exposure is driving the pattern.

<iframe src="assets/fig_regional_breakdown.html" width="100%" height="690px"
style="border:none;"></iframe>

Cold-phase outages outnumber warm-phase outages across every U.S. climate region. As we will
see, this is largely an artifact of the 2011–2015 reporting surge rather than a universal
climate effect.

<iframe src="assets/fig_causes_by_year.html" width="100%" height="670px"
style="border:none;"></iframe>

The spike is almost entirely driven by **Intentional Attack** events beginning in 2011. Two
external factors explain the rise: a change in federal reporting standards that required
utilities to log critical infrastructure tampering, and a surge in copper theft driven by
rising commodity prices (~$4.50/lb in 2011).

<iframe src="assets/fig_cause_time_series.html" width="100%" height="620px"
style="border:none;"></iframe>

The 2011 spike is a textbook **artifact of measurement** — the federal government began
requiring utilities to report infrastructure tampering under a much wider definition, turning
previously invisible local incidents into entries in the national dataset.

<iframe src="assets/fig_attack_subtypes.html" width="100%" height="620px"
style="border:none;"></iframe>

Vandalism tracks closely with copper commodity prices. Once prices fell ~60% by 2015 and
reporting loopholes closed, the wave of infrastructure vandalism collapsed almost overnight.

<iframe src="assets/fig_avg_customers_by_cause.html" width="100%" height="670px"
style="border:none;"></iframe>

Despite the sheer number of Intentional Attack events, the average customers affected per
event is very low — consistent with small-scale vandalism rather than catastrophic grid
failures.

<iframe src="assets/fig_impact_over_time.html" width="100%" height="600px"
style="border:none;"></iframe>

In 2011, total customer impact rose only modestly while the average per event fell sharply —
confirming the spike consists of many small incidents, not large-scale failures.

<iframe src="assets/pivot_weather_climate.html" width="100%" height="500px"
style="border:none;"></iframe>

Focusing on severe weather events specifically, seasonal patterns are clear: Winter Storm
events peak in winter, Storm events are more frequent during La Niña phases, and Heavy Wind
events appear across both climate conditions.

---

## Assessment of Missingness

### NMAR Analysis

The `CAUSE.CATEGORY.DETAIL` column is likely **Not Missing At Random (NMAR)**.

When a major weather event hits the grid, utility operators face an immediate operational
bottleneck: dispatching repair crews, isolating damaged lines, and protecting substations.
Filing the mandatory federal DOE OE-417 report requires only a high-level cause code within a
strict timeline. Providing the granular meteorological trigger (e.g. "Derecho" or
"Microburst") requires additional post-incident forensic work that may never happen if the
event was chaotic or poorly documented.

Because missingness depends on the unobserved severity of the operational chaos — a variable
not present in the dataset — this satisfies the definition of NMAR.

To shift this column from NMAR to MAR, we would need external proxy data such as emergency
call volumes during the first 24 hours, number of repair crews dispatched simultaneously, or
National Weather Service radar data matched to the outage location and timestamp.

### Missingness Dependency Tests

We test whether `CAUSE.CATEGORY.DETAIL` missingness depends on two candidate variables:
outage duration (continuous) and cause category (categorical). We use **absolute difference
in group means** for the continuous test and **TVD** for the categorical test.

<iframe src="assets/fig_missingness_test1.html" width="100%" height="540px"
style="border:none;"></iframe>

**Test 1 — Outage Duration (p ≈ 0.08):** We fail to reject the null hypothesis. Despite a
descriptive difference in group means, the high variance in outage durations means this gap
is consistent with random chance. Duration does not systematically drive missingness.

<iframe src="assets/fig_missingness_test2.html" width="100%" height="540px"
style="border:none;"></iframe>

**Test 2 — Cause Category (p ≈ 0.000):** We reject the null hypothesis. The TVD of 0.41
between the cause-category distributions of missing vs. non-missing rows is so extreme that
zero out of 1,000 shuffles came close to it. Severe weather events almost always receive a
meteorological sub-label because NOAA provides standardized taxonomy, while intentional attack
and equipment failure events are routinely left blank. The missingness of
`CAUSE.CATEGORY.DETAIL` is strongly **MAR on `CAUSE.CATEGORY`**.

---

## Hypothesis Testing

### Research Question

> Does the severity of global climate anomalies systematically correlate with the scale of
> customer impact during major power outages?

### Setup

We compare outages during **normal climate phases** (|ONI| < 0.5) against outages during
**severe climate phases** (|ONI| ≥ 0.5).

- **Null Hypothesis (H₀):** The distribution of `CUSTOMERS.AFFECTED` is identical across normal and severe climate periods. Any observed difference is due to random chance.
- **Alternative Hypothesis (H₁):** The number of customers affected shifts systematically during severe climate phases.
- **Test Statistic:** Absolute difference in group means (two-tailed)
- **Significance Level:** α = 0.05
- **Method:** Permutation test with 1,000 iterations — preferred over a t-test because `CUSTOMERS.AFFECTED` is heavily right-skewed

<iframe src="assets/fig_hypothesis_test.html" width="100%" height="580px"
style="border:none;"></iframe>

### Results

**Observed shift:** ~18,112 customers. **p-value:** 0.28.

We **fail to reject H₀**. A difference of this magnitude occurs by random chance ~28% of the
time under the null model. The extreme right-skew and high variance in `CUSTOMERS.AFFECTED`
mean that macro-level ONI shifts are not a reliable predictor of per-event customer impact at
the statistical level.

---

## Framing a Prediction Problem

While our analysis found no statistically significant relationship between climate anomalies
and outage scale, a predictive model can still leverage a richer set of features to identify
dangerous outages early.

**Prediction Problem:** At the onset of a power grid failure, can we predict whether it will
escalate into a **Mass-Impact Event** — defined as affecting more than 50,000 customers?

This is a **binary classification** problem. The target variable is derived from
`CUSTOMERS.AFFECTED > 50,000`.

We use **F1-score** as our primary metric because the dataset is severely class-imbalanced —
most outages are localized, with mass-impact events forming a small minority. A model that
always predicts "no mass-impact" achieves high accuracy but catches zero emergencies. F1-score
penalizes both false alarms and missed emergencies equally.

To prevent data leakage, only features observable **at the moment the outage begins** are
permitted as inputs. Post-incident variables like `OUTAGE.DURATION`, `DEMAND.LOSS.MW`, and
`OUTAGE.RESTORATION.DATE` are strictly excluded.

---

## Baseline Model

Our baseline is a **Decision Tree Classifier** in a scikit-learn Pipeline, using three
features known at the time of the outage:

| Feature | Type | Encoding |
|---|---|---|
| `ANOMALY.LEVEL` | Quantitative continuous | Pass-through |
| `MONTH` | Quantitative ordinal | Pass-through |
| `CAUSE.CATEGORY` | Nominal categorical | One-Hot Encoding |

<iframe src="assets/fig_baseline_cv.html" width="100%" height="520px"
style="border:none;"></iframe>

The baseline achieves a mean cross-validated F1-score of **0.9177** across 5 stratified
folds. This is a strong starting point — the immediate incident trigger (`CAUSE.CATEGORY`)
combined with seasonality (`MONTH`) and climate context (`ANOMALY.LEVEL`) already carries
substantial predictive signal. The consistency across folds indicates the model is not getting
lucky on any particular split.

---

## Final Model

We upgrade to a **Random Forest Classifier**, adding two engineered features motivated by the
physical realities of power distribution:

| Feature | Engineering Rationale |
|---|---|
| `GRID_REGION` | Maps each state to its macro power-grid division (West / Midwest / South / Northeast). Regional balancing authorities and terrain differ substantially in structural resilience. |
| `POPDEN_URBAN` | Urban population density directly governs how many customers are exposed in a geographically compact failure zone. |

We tuned `n_estimators` ∈ {50, 100, 200} and `max_depth` ∈ {5, 10, 15, None} via
GridSearchCV with stratified 5-fold cross-validation. The optimal configuration was
`max_depth=10`, `n_estimators=200`.

<iframe src="assets/fig_model_comparison.html" width="100%" height="520px"
style="border:none;"></iframe>

<iframe src="assets/fig_feature_importance.html" width="100%" height="540px"
style="border:none;"></iframe>

The final model raises the cross-validated F1-score from **0.9177 to 0.9280** — a +1.03%
absolute improvement over the baseline.

Two additional features drove this improvement, both motivated by the physical realities of
how power outages propagate through infrastructure:

**`GRID_REGION`** maps each state to its macro power-grid division (West, Midwest, South,
Northeast). In the real world, grid infrastructure is governed by distinct regional balancing
authorities — CAISO in the West, PJM in the Northeast, ERCOT in Texas — each with different
interconnection topologies, terrain constraints, and reserve margins. An outage that cascades
in a densely interconnected Northeastern grid behaves very differently from one in the
sprawling Western Interconnection. By encoding this structural context, we give the model
information about the underlying physical system that governs how failures propagate, rather
than relying solely on the incident trigger.

**`POPDEN_URBAN`** captures urban population density at the outage location. The data
generating process for `CUSTOMERS.AFFECTED` is fundamentally spatial: a failure affecting a
dense urban distribution network instantly exposes thousands of customers within a small
geographic radius, whereas the same failure on a rural transmission line may cover a vast area
but reach very few people. This feature directly encodes that relationship, allowing the model
to distinguish high-exposure urban grid failures from low-exposure rural ones before the
outage resolves.

We also upgraded from a single **Decision Tree** to a **Random Forest** — an ensemble of 200
trees each trained on a bootstrapped data subset. Averaging predictions across many trees
dramatically reduces the variance that made the baseline tree prone to overfitting on
historical noise. The optimal hyperparameters identified via GridSearchCV were
`n_estimators=200` and `max_depth=10`, where limiting depth to 10 serves as a regularizer
that forces each tree to capture generalized regional patterns rather than memorizing
specific outage events in the training folds.

The feature importance chart confirms that `CAUSE.CATEGORY` still dominates predictions,
consistent with our earlier finding that cause type is the strongest signal for whether an
outage escalates. `POPDEN_URBAN` also ranks highly, validating the intuition that population
density is a key structural driver of outage scale.

---

## Fairness Analysis

### Setup

We evaluate whether the final model performs equitably across two groups defined by population
density, split at the median `POPDEN_URBAN` value of **2,315.2 people per square mile**:

- **Group X (Urban):** Outages in high-density areas (POPDEN_URBAN ≥ median)
- **Group Y (Rural):** Outages in low-density areas (POPDEN_URBAN < median)

| | |
|---|---|
| **Evaluation Metric** | F1-score |
| **Null Hypothesis (H₀)** | The model is fair. Any F1-score difference between urban and rural outages is due to random chance. |
| **Alternative Hypothesis (H₁)** | The model performs significantly differently for urban vs. rural outages. |
| **Test Statistic** | \|F1 Urban − F1 Rural\| |
| **Significance Level** | α = 0.05 |

<iframe src="assets/fig_fairness_test.html" width="100%" height="540px"
style="border:none;"></iframe>

### Results

The model achieved an F1-score of **0.9576** for urban outages and **0.9614** for rural
outages, producing an observed difference of **0.0038**. The permutation test yielded a
**p-value of 0.709**.

We **fail to reject the null hypothesis**. The model performs nearly identically across both
population density groups, and the small observed difference is consistent with random
variation. We find no statistically significant evidence of bias with respect to urban vs.
rural infrastructure context.

---

## Summary and Key Insights

This project traced the full data science lifecycle from raw infrastructure data to a
deployable predictive model, centered on one question: do global climate anomalies drive the
scale of power outage impacts in the United States?

The short answer is **no — at least not directly.** Our hypothesis test found no statistically
significant relationship between ONI severity and customer impact (p ≈ 0.28). The variance in
outage scale is simply too high for macro-level climate indices to serve as a reliable
per-event predictor.

The more interesting finding emerged from the EDA. The apparent La Niña dominance in the data
was almost entirely explained by the **2011–2015 intentional attack surge** — itself an
artifact of federal reporting standard changes and a copper theft wave tied to commodity
prices, not climate. This is a reminder that the most important signal in a dataset is
sometimes not the one you set out to find.

A few other key takeaways:

- **Cause type is the dominant predictor of outage severity.** Whether an outage is a severe
  weather event vs. intentional attack vs. equipment failure tells you more about likely
  customer impact than any climate or geographic variable.
- **Population density governs exposure.** The same failure in an urban distribution network
  and a rural transmission corridor affects vastly different numbers of people — encoding
  this physically motivated feature meaningfully improved the model.
- **Missingness is not random.** The near-zero p-value on the TVD test for `CAUSE.CATEGORY`
  confirmed that whether detail fields get filled in depends heavily on the type of incident,
  which has implications for any analysis that relies on granular cause sub-classifications.
- **The model is equitable.** The fairness analysis found no statistically significant
  performance gap between urban and rural outages (p ≈ 0.71), suggesting the classifier
  generalizes consistently across infrastructure contexts.

---

### Areas for Further Exploration

**Richer climate features.** The ONI index is a single scalar that averages a complex global
phenomenon. Incorporating regional temperature anomalies, precipitation deviations, or NOAA
storm severity indices would give the model a much more granular view of the climate
conditions at the time and location of each outage.

**Post-2016 data.** The dataset ends in 2016, missing nearly a decade of grid evolution
including the rapid growth of distributed solar, increased extreme weather frequency, and
new critical infrastructure attack patterns. Extending the dataset would likely shift the
feature importance landscape significantly.

**Demand loss as an alternative target.** `CUSTOMERS.AFFECTED` is a noisy proxy for severity
— a single hospital losing power may matter more than 10,000 residential customers. Modeling
`DEMAND.LOSS.MW` as a regression target, or building a multi-class severity tier classifier,
would produce more operationally useful predictions for utility dispatch teams.

**Grid topology features.** The current model treats geography as a static categorical
variable. Incorporating actual transmission network data — node degree, betweenness
centrality, reserve margin — would allow the model to reason about structural vulnerability
rather than just regional identity.

**Time-series modeling.** Each outage is treated as an independent event, but outages cluster
in time during major storm systems and heat waves. A sequential model that accounts for
temporal autocorrelation could substantially improve predictions during high-risk windows.