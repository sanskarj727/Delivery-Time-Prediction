# Methodology

How the notebook gets from the raw order file to the fitted model, and why each choice was made. Counts come from the original run on the full dataset.

## 1. Data

175,777 orders, 14 columns, no missing values (see [`data/README.md`](../data/README.md)). Timestamps are read as text, so they are converted to datetimes. `market_id`, `store_primary_category` and `order_protocol` are identifiers rather than quantities and become categorical.

## 2. Derived features

| Feature | Definition |
| --- | --- |
| `delivery_duration` | `actual_delivery_time` minus `created_at`, in minutes. This is the target. |
| `order_hour` | Hour of `created_at` (0 to 23) |
| `order_dayofweek` | Day of week of `created_at`, Monday = 0 |
| `isweekend` | True for Saturday and Sunday (day 5 or 6) |

The two raw timestamps are then dropped, leaving 15 features and the target.

## 3. Train/test split

80/20 random split with seed 42: 140,621 training orders and 35,156 test orders. The split happens before any exploration, so every later statistic (outlier bounds, scaling means and deviations) comes from the training set alone.

## 4. Exploration and feature reduction

Distributions, scatter plots, boxplots and a correlation heatmap are drawn on the training set. Three features are dropped for their weak correlation with the target:

| Dropped | Correlation with duration | Reason |
| --- | --- | --- |
| `min_item_price` | 0.02 | Near zero |
| `store_primary_category` | 0.03 | Near zero; the boxplot shows no clear difference between categories |
| `order_dayofweek` | 0.05 | Near zero and redundant with `isweekend` (correlation 0.81) |

Twelve features remain.

## 5. Outliers

Boxplots show long right tails in the target and in most numeric features. Rows outside 1.5 × IQR of any numeric column, including the target, are removed from the **training set**. The rule is applied one column at a time, with the quartiles recomputed after each pass. Removal was preferred to capping or replacement, which would shift the distribution towards the bound or the centre. The test set is left unchanged so that evaluation reflects raw data.

## 6. Scaling

Numeric features are standardised with `StandardScaler`, fitted on the cleaned training set and applied to the test set. Categorical columns are not scaled. Scaling does not change a linear regression's predictions.

## 7. Feature selection and model

RFE with a linear regression ranks the features. A loop fits a model for every feature count from 1 to 12 and records the test MAE:

| Features | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| MAE (minutes) | 6.91 | 5.45 | 4.25 | 4.10 | 3.15 | 2.61 | 2.53 | 2.39 | 2.36 | 2.33 | 2.32 | 2.31 |

Error falls steeply up to six features and then flattens. Eight features were kept as the point where further gains are small. The final model is an ordinary least-squares `LinearRegression` on those 8 features.

## 8. Evaluation

On the test set: MAE 2.39 minutes, RMSE 3.39 minutes, R² 0.87. Residuals are plotted against predictions, in a Q-Q plot and in a histogram.

## 9. Assumptions and trade-offs

- **Feature count chosen on test error.** Cross-validation on the training set would avoid a slightly optimistic estimate.
- **Identifiers as numbers.** `market_id`, `order_protocol` and the weekend flag enter the regression as numeric values rather than as one-hot columns.
- **Removal of outliers** trades coverage of unusual orders for a better fit on typical ones.
- **A single random split** gives one estimate of each metric with no measure of its variability.
