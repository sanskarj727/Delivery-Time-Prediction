# Findings

Results from the delivery-time regression. Counts and chart readings come from the training set unless marked as test-set results. Values read off charts are approximate.

## The data

- **Delivery duration** runs from about 32 to about 110 minutes, peaks near 43 minutes and has a long right tail. Durations above 80 minutes are rare.
- **Order hour is bimodal.** Most orders fall between midnight and 5 am, with the peak at 2 am, and between 7 pm and 11 pm. No orders appear in the hours 9 to 13, and 7, 8 and 14 are very sparse. The time zone of the timestamps is not stated, so the peaks cannot be matched to local meal times.
- **Day of week.** Saturday and Sunday have the most orders, roughly 24,000 each against roughly 17,000 to 20,000 on other days.
- **Weekday versus weekend.** About 92,000 weekday orders against about 48,000 weekend orders compares five days with two; per day, weekends carry about 24,000 against about 18,000.
- **Skew.** Subtotal, item counts, item prices and the dasher and order counts are right-skewed.

![Delivery duration by order hour](images/duration-by-hour.png)

## What relates to delivery time

Correlation with delivery duration:

| Feature | Correlation |
| --- | --- |
| `distance` | 0.46 |
| `subtotal` | 0.41 |
| `total_outstanding_orders` | 0.39 |
| `order_hour` | −0.35 |
| `num_distinct_items` | 0.31 |
| `max_item_price` | 0.25 |
| `total_items` | 0.22 |
| `total_busy_dashers` | 0.21 |
| `total_onshift_dashers` | 0.17 |
| `order_protocol` | −0.14 |
| `isweekend` | 0.14 |
| `market_id` | −0.08 |
| `order_dayofweek` | 0.05 |
| `store_primary_category` | 0.03 |
| `min_item_price` | 0.02 |

- **Distance** is the strongest single relationship. Duration rises with distance, with wide scatter.
- **Time of day.** The median duration is highest for orders placed at 2 and 3 am (about 49 to 50 minutes) and lowest from 8 pm to 11 pm (about 39 to 41 minutes). Orders at 2 and 3 am also have the longest upper tail.
- **Capacity features overlap.** Partners on shift, partners busy and outstanding orders correlate 0.93 to 0.94 with each other.
- **Store category** shows no clear difference in median duration.

![Distance versus delivery duration](images/distance-vs-duration.png)

## Model performance

Linear regression on 8 features chosen by RFE, evaluated on 35,156 test orders:

| Metric | Value |
| --- | --- |
| MAE | 2.39 minutes |
| RMSE | 3.39 minutes |
| R² | 0.87 |

Test MAE by number of features falls from 6.91 (one feature) to 2.61 (six), 2.39 (eight) and 2.31 (twelve), so eight features capture most of the available accuracy.

## Residuals

- **Lower edge.** Residuals have a sharp diagonal lower boundary, consistent with a floor of about 32 minutes in actual delivery times.
- **Right skew.** Positive residuals reach about +25 minutes while negative residuals stop near −14. The histogram is right-skewed and the Q-Q plot leaves the line in the upper tail, so errors are not normally distributed.
- **Spread.** The scatter widens at higher predicted values: the model is least reliable for long deliveries.

![Residual diagnostics](images/residual-diagnostics.png)

## Limitations

1. **The feature count was chosen on test-set error,** so the metrics are slightly optimistic. Cross-validation on the training set is the cleaner approach.
2. **Results come from one random split** with no cross-validation and no estimate of variability.
3. **Identifiers enter as numbers.** `market_id`, `order_protocol` and the weekend flag are not one-hot encoded, and store category was dropped. Correlations for coded identifiers depend on how the codes are numbered.
4. **Collinear capacity features** make their individual effects impossible to separate.
5. **Outliers were removed from the training set only.** The model is fitted to typical orders and is weaker on long deliveries.
6. **The time zone of the timestamps is not stated,** and the training set has no orders for hours 9 to 13, so time-of-day findings describe the hours present in the data.
7. **The units of `distance` are not stated.**
8. **One model family.** Linear regression is the only model tried, and the residuals show non-normal, right-skewed errors.
9. **The dataset is not included,** so the notebook outputs reflect the original run and a fresh run needs the file.
