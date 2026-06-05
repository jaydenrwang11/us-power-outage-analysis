# U.S. Power Outages Analysis (2000–2016)

**Author:** Jayden Wang

An end-to-end data science project investigating whether global climate anomalies
systematically correlate with the scale of customer impact during major U.S. power outages.

## Website
https://jaydenrwang11.github.io/us-power-outage-analysis/

## Overview
This project follows the full data science lifecycle across 1,534 major outage events:
- **EDA** — exploring climate anomaly distributions, temporal trends, and the 2011–2015 reporting spike
- **Missingness Analysis** — NMAR assessment and permutation-based MAR dependency tests
- **Hypothesis Testing** — permutation test on climate severity vs. customer impact
- **Predictive Modeling** — binary classification of mass-impact events (>50,000 customers affected) using a Random Forest with stratified cross-validation (F1 = 0.928)
- **Fairness Analysis** — permutation test confirming equitable model performance across urban and rural outages

## Data
[U.S. Major Power Outage dataset](https://engineering.purdue.edu/LASCI/research-data/outages) —
Purdue University LASCI, covering outages reported to the DOE between 2000 and 2016.
