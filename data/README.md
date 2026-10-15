# Data

The notebook reads one file, `porter_data_1.csv`, with 175,777 rows and 14 columns. It was supplied with the course the project began in and is not redistributed here, so it is excluded from version control (see `.gitignore`).

Expected location:

```
delivery-time-prediction/
└── data/
    └── porter_data_1.csv
```

## Columns

| Column | Type in the file | Meaning |
| --- | --- | --- |
| `market_id` | float | Market (area) of the store, as an ID |
| `created_at` | text | When the order was placed |
| `actual_delivery_time` | text | When the order was delivered |
| `store_primary_category` | integer | Store category, as a numeric code |
| `order_protocol` | float | How the order was placed, as a numeric code |
| `total_items` | integer | Items in the order |
| `subtotal` | integer | Order value |
| `num_distinct_items` | integer | Distinct items in the order |
| `min_item_price` | integer | Price of the cheapest item |
| `max_item_price` | integer | Price of the most expensive item |
| `total_onshift_dashers` | float | Delivery partners on shift when the order was placed |
| `total_busy_dashers` | float | Partners already busy with other orders |
| `total_outstanding_orders` | float | Orders pending fulfilment at that time |
| `distance` | float | Distance from the store to the customer (units not stated) |

None of the columns has missing values in the original file.
