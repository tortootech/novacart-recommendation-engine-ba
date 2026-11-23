# DAX in the Power BI dashboard

Source: `dashboard/novacart.pbix`, built on the Olist Brazilian E-Commerce dataset.

## Data model
| From | To | Type |
|---|---|---|
| `olist_order_items_dataset[order_id]` | `olist_orders_dataset[order_id]` | many-to-one |
| `olist_order_items_dataset[product_id]` | `olist_products_dataset[product_id]` | many-to-one |
| `olist_orders_dataset[customer_id]` | `olist_customers_dataset[customer_id]` | one-to-one, both directions |

## Measures

### Total Revenue
```dax
Total Revenue =
SUMX(
    olist_order_items_dataset,
    olist_order_items_dataset[price] + olist_order_items_dataset[freight_value]
)
```
Revenue is item price plus freight. `SUMX` adds the two columns row by row, then sums. Result: 15.84M BRL.

### Order Count
```dax
Order Count = DISTINCTCOUNT(olist_orders_dataset[order_id])
```
Distinct orders, about 99K.

### Average Order Value
```dax
Average Order Value = DIVIDE([Total Revenue], [Order Count])
```
Result: 159.33 BRL. `DIVIDE` returns blank instead of an error when the denominator is zero.

### Customers with Repeat Orders
```dax
Customers with Repeat Orders =
CALCULATE(
    DISTINCTCOUNT(olist_customers_dataset[customer_unique_id]),
    FILTER(
        VALUES(olist_customers_dataset[customer_unique_id]),
        CALCULATE(DISTINCTCOUNT(olist_orders_dataset[order_id])) > 1
    )
)
```
Counts customers who placed more than one order. The inner `CALCULATE` turns each customer into a filter context (context transition), so the order count is evaluated per customer. `customer_unique_id` identifies a real person, since Olist assigns a new `customer_id` to each order.

### Repeat Purchase Rate
```dax
Repeat Purchase Rate =
DIVIDE(
    [Customers with Repeat Orders],
    DISTINCTCOUNT(olist_customers_dataset[customer_unique_id])
)
```
Result: 3.12%. This is the baseline the recommendation engine is meant to raise.

## Note
`Average Value` in the model is a second copy of `Average Order Value`. It can be deleted.
