## Project Overview

This team project was developed for the **ASBA Predictive Analytics Competition, Semester 2 (2025)**, hosted on Kaggle for Macquarie University students enrolled in BUSA3020 and BUSA8001.

Our objective was to predict listed car prices using vehicle attributes. The project followed a predictive analytics workflow, from understanding and preparing the data through feature engineering, regression modelling, evaluation, and submission.

**Outcome:** Our team achieved **1st place on the Kaggle competition leaderboard** and was recognised as the **highest achiever in the subject**.

## Problem and Data

Accurately estimating a vehicle's listed price is a regression problem. Vehicle records can contain missing, inconsistent, or otherwise challenging inputs, making data quality an important part of the modelling process.

| Item | Description |
| --- | --- |
| Task | Predict listed car prices |
| Training data | 8,000 records |
| Test data | 4,000 records |
| Target | `price` |
| Record identifier | `vin` |
| Evaluation metric | Mean Absolute Percentage Error (MAPE) |
| Submission format | CSV containing `vin` and predicted `price` |

The competition used a public leaderboard based on 50% of the test records and a final leaderboard based on the remaining 50%.

## My Contribution — Data Cleaning and Preparation

**My primary responsibility was data cleaning**, supporting the team's downstream feature engineering and predictive modelling work.

This stage focused on preparing the supplied vehicle data for reliable analysis and modelling. It involved examining the dataset's quality and addressing preprocessing requirements before model development.

## Results and Recognition

- **1st place** in the ASBA Predictive Analytics Competition S2 (2025) on Kaggle.
- **Highest achiever** in the subject, as reported by the project team.

The competition evaluated submissions using **Mean Absolute Percentage Error (MAPE)**:

$$
\mathrm{MAPE}=\frac{100}{n}\sum_{i=1}^{n}\left|\frac{y_i-\hat{y}_i}{y_i}\right|
$$

Lower MAPE indicates more accurate predictions, for nonzero actual prices.
