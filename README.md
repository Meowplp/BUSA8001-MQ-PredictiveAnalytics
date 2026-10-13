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
I built the preprocessing workflow that converted inconsistent vehicle listing fields into numerical, encoded, and engineered features for the team's regression models. My main technical contribution was a **custom hierarchical imputer** that uses statistics from comparable cars before resorting to global fallback values.

### 1. Standardise numeric fields

Vehicle dimensions and seating capacity arrived as text (for example, measurements with `in` suffixes or seating counts with `seats`). I cleaned `back_legroom`, `front_legroom`, `height`, `wheelbase`, `width`, and `maximum_seating` by stripping units and dash placeholders with regex, trimming whitespace, and converting values using `pd.to_numeric(errors='coerce')`. Unparseable values became missing values to handle in the imputation stage.

### 2. Extract structured information from compound strings

I split text fields into model-ready features:

| Raw field | Example | Derived fields |
| --- | --- | --- |
| `power` | `270 hp @ 5,600 RPM` | `power_hp`, `power_rpm` |
| `torque` | `269 lb-ft @ 5,200 RPM` | `torque_lbft`, `torque_rpm` |
| `engine_type` | `V6` | `cylarr_type`, `cylinders_num` |
| `transmission_display` | `9-Speed Automatic` | `gear_num` |
| `listed_date` | Date string | `listed_year`, `listed_month`, day-of-month `listed_date` |

I also derived `age` from manufacture year and `brand_avg_price` from the average training-set price per `make_name`. The original compound fields were removed after extraction.

### 3. Hierarchical, group-aware missing-value imputation

First, I normalised missing-value markers (`N/A`, `unknown`, `--`, `null`, blanks, etc.) to `NaN`. I then implemented an `Imputer` class with **progressive fallback chains**. Instead of assigning the same average to every missing vehicle attribute, the imputer starts with narrowly matched vehicles, then relaxes its matching criteria until a valid group statistic is found.

For example, missing **horsepower** is estimated in this order:

`make + model + cylinder arrangement + year` → `make + model + year` → `make + cylinder arrangement` → `make` → **global training-set mean**.

| Feature family | Statistic | Fallback hierarchy |
| --- | --- | --- |
| Appearance / colour | Mode | Make + model + year → make + model → make → body type → global |
| Mechanical specifications | Mode | Make + model + cylinder arrangement + year → broader vehicle groups → global |
| Engine power and fuel economy | Mean | Make + model + cylinder arrangement + year → broader vehicle groups → global |
| Dimensions and seating | Mean | Body type → make → global |
| Mileage | Median | Year + city + new/used status → year + new/used status → year → global |
| Seller rating | Median | Dealer ZIP → city → global |
| Savings and gear count | Median | Make + model + year → make + year → make → global |
| Latitude | Mean | City → dealer ZIP → overall dataset mean |

Global mode, mean, and median fallback values were **learned from the training set** and reused on the test set. Group-level statistics, however, were computed within each dataset during transformation. Discrete imputed values such as gear count, cylinder count, and seating capacity were rounded back to integers.

### 4. Encode categorical variables

- Mapped `franchise_dealer` and `is_new` to binary 0/1 values.
- Identified the five most frequent categories in each eligible column using training data; consolidated remaining or multi-valued entries into `other`.
- Applied `OneHotEncoder(drop='first', handle_unknown='ignore')` and aligned the final train/test feature columns.
- Excluded high-cardinality fields (`vin`, `dealer_zip`, `city`, `exterior_color`) from one-hot encoding; dropped the location and exterior-colour columns after processing while retaining `vin` as the submission identifier.

### 5. Handle extreme observations

I calculated the **99.8th percentile** of `mileage` and `daysonmarket` from training data and replaced values above those thresholds with the respective training-set medians in both datasets. I also clipped `seller_rating` to its valid **0–5** range. This treated extreme observations without deleting entire rows.

### 6. Engineer domain-informed features

I added features representing nonlinear vehicle usage, engine characteristics, listing behaviour, and seasonality:

| Feature | Implementation / signal |
| --- | --- |
| `log_mileage` | `log1p(mileage)` to reduce skew |
| `age_cubed` | Cubic vehicle-age effect |
| `fuel_efficiency_avg` | Average of city and highway fuel economy |
| `fuel_economy_diff` | Highway minus city fuel economy |
| `torque_hp_ratio` | Torque relative to power |
| `displacement_per_cylinder` | Engine displacement divided by cylinder count |
| `interior_space` | Front plus rear legroom |
| `market_urgency` | Days on market × savings amount |
| `new_car_age_interaction` | New-car flag × age |
| `horsepower_age_interaction` | Horsepower × age |
| `climate_proxy` | Absolute difference between latitude and 40 |
| Seasonal indicators | Flags based on listing month |

Additional engineered fields included `depreciation_rate`, `value_per_hp`, `power_to_weight` (implemented as horsepower divided by maximum seating, not vehicle mass), and `displacement_per_hp`. The preprocessing function dropped the original `mileage` and `listed_month` after generating derived features.


## Results and Recognition

- **1st place** in the ASBA Predictive Analytics Competition S2 (2025) on Kaggle.
- **Highest achiever** in the subject, as reported by the project team.

The competition evaluated submissions using **Mean Absolute Percentage Error (MAPE)**:

$$
\mathrm{MAPE}=\frac{100}{n}\sum_{i=1}^{n}\left|\frac{y_i-\hat{y}_i}{y_i}\right|
$$

Lower MAPE indicates more accurate predictions, for nonzero actual prices.
