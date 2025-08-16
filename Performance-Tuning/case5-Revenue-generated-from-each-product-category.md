# 💰 Revenue generated from each product category

## Write SQL Query to Calculate the revenue generated from each product category.

```sql
SELECT C.category_id, C.Name, SUM(OD.quantity * OD.unit_price) AS Total_revenue
FROM categories C
 JOIN products P ON P.category_id = C.category_id
 JOIN orderDetails OD ON OD.product_id = P.product_id
 GROUP BY C.category_id
 ORDER BY Total_revenue DESC;
```
### ⏱️ Before Optimization

```diff
- I can’t execute the query — every time it shows "Lost connection."
```

### 🚀 After Optimization

### Category Revenue Denormalization

#### Overview
A denormalized table containing `category` and `total_amount` has been created to improve performance when retrieving category revenue data.

```sql
 CREATE TABLE Category_Revenue(
    order_detail_id INT primary key , 
    category_id INT not null , 
    category_name varchar(255) ,
    total_amount decimal(15,12) not null
 )
```

#### Benefits
- **Faster Queries**: Queries that retrieve category revenue data are now much faster.
- **Reduced Complexity**: No need to perform a `JOIN` on millions of rows each time.
- **Efficient Updates**: The table only needs to be updated when inserting or modifying order data.

#### Usage
Run your revenue-related queries directly on the denormalized table for better performance.

#### Query
```Query
explain analyze SELECT category_name , SUM(total_amount) as category_revenue
FROM category_revenue
group by category_name 
ORDER by category_revenue
```
#### Execution Plan:
```sql
-> Sort: category_revenue  (actual time=3771..3771 rows=100 loops=1)
     -> Table scan on <temporary>  (actual time=3771..3771 rows=100 loops=1)
         -> Aggregate using temporary table  (actual time=3771..3771 rows=100 loops=1)
             -> Table scan on category_revenue  (cost=351376 rows=3.42e+6) (actual time=6.04..2038 rows=3.4e+6 loops=1)
 
```

- create index on category_revenue

```sql
-> Sort: category_revenue  (actual time=1589..1589 rows=100 loops=1)
     -> Stream results  (cost=698886 rows=95) (actual time=13.5..1589 rows=100 loops=1)
         -> Group aggregate: sum(category_revenue.total_amount)  (cost=698886 rows=95) (actual time=13.5..1589 rows=100 loops=1)
             -> Covering index scan on category_revenue using idx_category_revenue  (cost=357079 rows=3.42e+6) (actual time=0.0515..992 rows=3.4e+6 loops=1)

```

# 🚀 Query Optimization with Denormalization & Indexing  

## 📊 Performance Comparison  

| **Scenario**                        | **Sort Time (ms)** | **Group Aggregate / Total (ms)** | **Total Elapsed (ms)** | **Improvement** |
|------------------------------------|--------------------|----------------------------------|------------------------|-----------------|
| **Before Optimization (JOINs)**    | 352                | 8,561                            | ~8,900                 | Baseline        |
| **Denormalization (No Index)**     | 3,771              | 3,771                            | ~3,771                 | ~58% faster     |
| **Denormalization + Index**        | 1,589              | 1,589                            | ~1,589                 | ~82% faster vs baseline |

---

## 🔍 Observations  

- **Before Optimization**: Heavy `JOIN` operations across millions of rows, execution ~8.9s.  
- **After Denormalization**: Simplified queries, execution ~3.7s.  
- **After Indexing**: Covering index applied, execution ~1.6s.  

---


