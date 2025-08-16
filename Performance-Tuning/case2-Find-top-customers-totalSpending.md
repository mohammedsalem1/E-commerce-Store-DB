# 💲 Write SQL Query to Find the top customers by total spending.

``` sql
 Select concat(C.first_name,C.last_name)as fullName , C.customer_id  , sum(total_amount) as TotalSpending
  from customers C join Orders O
     ON C.customer_id = O.customer_id
         group by C.customer_id
           order by TotalSpending desc
```       
### ⏱️ Before Optimization
#### direct query
```sql
-> Sort: TotalSpending DESC  (actual time=12371..12418 rows=849500 loops=1)
     -> Stream results  (cost=1.11e+6 rows=948002) (actual time=7.43..11755 rows=849500 loops=1)
         -> Group aggregate: sum(o.total_amount)  (cost=1.11e+6 rows=948002) (actua...
```

#### make with test 
``` sql
WITH CustomerSpending AS (
    SELECT 
        C.customer_id,
        CONCAT(C.first_name, ' ', C.last_name) AS fullName,
        SUM(O.total_amount) AS TotalSpending
    FROM customers C
    JOIN Orders O ON C.customer_id = O.customer_id
    GROUP BY C.customer_id
)
SELECT 
    fullName,
    customer_id,
    TotalSpending
FROM CustomerSpending
ORDER BY TotalSpending DESC
```

### 🚀 After Optimization
  - ✔Composite Index Creation actually we will extend the current index that has been built already on CustomerID FK

``` sql
 CREATE INDEX idx_orders_customers_amount 
    ON orders(customer_id , total_amount desc) 
```

#### direct query

``` sql
 -> Sort: TotalSpending DESC  (actual time=5473..5522 rows=849500 loops=1)
     -> Stream results  (cost=1.47e+6 rows=948002) (actual time=6.78..4913 rows=849500 loops=1)
         -> Group aggregate: sum(o.total_amount)  (cost=1.47e+6 rows=948002) (actual time=6.77..4548 rows=849500 loops=1)
             -> Nested loop inner join  (cost=1.26e+6 rows=2.12e+6) (actual time=6.76..4217 rows=2e+6 loops=1)
                 -> Index scan on C using PRIMARY  (cost=100086 rows=948002) (actual time=6.73..978 rows=999999 loops=1)
                 -> Covering index lookup on O using idx_orders_customers_amount (customer_id=c.customer_id)  (cost=1 rows=2.24) (actual time=0.00244..0.00306 rows=2 loops=999999)
 

```

#### with index use with 

```sql
with index use with 
  -> Sort: customerspending.TotalSpending DESC  (cost=2.6..2.6 rows=0) (actual time=10830..10877 rows=849500 loops=1)
     -> Table scan on CustomerSpending  (cost=2.5..2.5 rows=0) (actual time=10155..10337 rows=849500 loops=1)
         -> Materialize CTE customerspending  (cost=0..0 rows=0) (actual time=10155..10155 rows=849500 loops=1)
             -> Table scan on <temporary>  (actual time=9389..9585 rows=849500 loops=1)
                 -> Aggregate using temporary table  (actual time=9389..9389 rows=849500 loops=1)
                     -> Nested loop inner join  (cost=1.31e+6 rows=1.91e+6) (actual time=6.77..3364 rows=2e+6 loops=1)
                         -> Filter: (o.customer_id is not null)  (cost=194803 rows=1.91e+6) (actual time=0.737..767 rows=2e+6 loops=1)
                             -> Covering index scan on O using idx_orders_customers_amount  (cost=194803 rows=1.91e+6) (actual time=0.735..669 rows=2e+6 loops=1)
                         -> Single-row index lookup on C using PRIMARY (customer_id=o.customer_id)  (cost=0.482 rows=1) (actual time=0.00118..0.0012 rows=1 loops=2e+6)
 
```
### 📈 Comparison Table


| **Scenario**                        | **Sort Time (ms)** | **Group Aggregate / Total (ms)** | **Total Elapsed (ms)** | **Improvement** |
|------------------------------------|------------------|---------------------------------|-----------------------|----------------|
| Without index, without WITH         | 12,371 – 12,418  | 7,430 – 11,755                 | ~11,755               | Baseline       |
| Without index, with WITH (CTE)      | 28,974 – 29,021  | 24,079 – 28,415                | ~29,000               | ~-147% slower  |
| With index, without WITH            | 5,473 – 5,522    | 6,770 – 4,913                  | ~5,500                | +53% faster    |
| With index, with WITH (CTE)         | 10,830 – 10,877  | 9,389 – 10,155                 | ~10,800               | +8% faster vs baseline, slower than direct index query |

### 💡 What I Learned 
 
- Best performance → With index, without CTE (~5.5 sec).

- CTE adds overhead, even with the index (~10.8 sec).

- Without index, queries are much slower, especially with CTE (~29 sec).

- Use idx_orders_customers_amount covering index to speed up aggregation and sorting.


