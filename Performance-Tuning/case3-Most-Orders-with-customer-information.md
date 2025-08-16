#📃 Recent 1000 Order

## Write SQL Query to Retrieve the most recent orders with customer information with 1000 orders.

#### Original query
``` sql
SELECT concat(C.first_name,C.last_name)as CustomerName , order_id , order_date   
    from customers C join Orders O
		ON C.customer_id = O.customer_id
		  order by order_date desc
           limit 1000
```
### ⏱️ Before Optimization

```sql
-> Limit: 1000 row(s)  (cost=2.05e+6 rows=1000) (actual time=1061..1194 rows=1000 loops=1)
     -> Nested loop inner join  (cost=2.05e+6 rows=1.91e+6) (actual time=1061..1194 rows=1000 loops=1)
         -> Sort: o.order_date DESC  (cost=193763 rows=1.91e+6) (actual time=1061..1061 rows=1000 loops=1)
             -> Filter: (o.customer_id is not null)  (cost=193763 rows=1.91e+6) (actual time=8.34..601 rows=2e+6 loops=1)
                 -> Table scan on O  (cost=193763 rows=1.91e+6) (actual time=8.34..521 rows=2e+6 loops=1)
         -> Single-row index lookup on C using PRIMARY (customer_id=o.customer_id)  (cost=0.869 rows=1) (actual time=0.133..0.133 rows=1 loops=1000)
 
```

### 🚀 After Optimization

- ✔Composite Index Creation actually we will extend the current index that has been built already on CustomerID FK

``` sql
CREATE INDEX idx_orders_customer_date ON 
  Orders(Customer_Id, Order_Date DESC);
```

### use index
``` sql
-> Limit: 1000 row(s)  (cost=2.05e+6 rows=1000) (actual time=1004..1008 rows=1000 loops=1)
     -> Nested loop inner join  (cost=2.05e+6 rows=1.91e+6) (actual time=1004..1008 rows=1000 loops=1)
         -> Sort: o.order_date DESC  (cost=195345 rows=1.91e+6) (actual time=1004..1004 rows=1000 loops=1)
             -> Filter: (o.customer_id is not null)  (cost=195345 rows=1.91e+6) (actual time=2.77..433 rows=2e+6 loops=1)
                 -> Index scan on O using idx_orders_customer_date  (cost=195345 rows=1.91e+6) (actual time=2.77..343 rows=2e+6 loops=1)
         -> Single-row index lookup on C using PRIMARY (customer_id=o.customer_id)  (cost=0.869 rows=1) (actual time=0.00336..0.00338 rows=1 loops=1000)
```

### 📈 Comparison Table

| **Scenario**                        | **Sort Time (ms)** | **Group Aggregate / Total (ms)** | **Total Elapsed (ms)** | **Improvement** |
|------------------------------------|------------------|---------------------------------|-----------------------|----------------|
| Without index                       | 1061             | 133                             | ~1194                 | Baseline       |
| With index (`idx_orders_customer_date`) | 1004             | 3.38                            | ~1008                 | +15% faster    |


# If have more time test denormlization