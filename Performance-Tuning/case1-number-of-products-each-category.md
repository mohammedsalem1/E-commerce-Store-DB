# 🔢 Daily Revenue Report

## Write an SQL query to retrive total number of products for each category.

``` sql
 select C.name , count(P.product_id) as totalProducts
    from Categories C join Products P
      ON C.category_id = P.category_id
        group by C.category_id 
```

### 📊 Execution Plan (Before)
 
 ```sql 
 -> Group aggregate: count(p.product_id)  (cost=19500 rows=100) (actual time=1.25..60 rows=100 loops=1)
     -> Nested loop inner join  (cost=9781 rows=97193) (actual time=0.25..51.5 rows=100000 loops=1)
         -> Index scan on C using PRIMARY  (cost=10.2...
  ```

### 🔧 Optimization Applied
- In MySQL (with the InnoDB engine), a foreign key column automatically gets an index if one does not already exist

- ✅ Added an index on category(category_id , name) 

### 🚀 After Optimization
 
 ```sql 
 -> Group aggregate: count(p.product_id)  (cost=19500 rows=100) (actual time=0.95..58 rows=100 loops=1)
     -> Nested loop inner join  (cost=9781 rows=97193) (actual time=0.144..49.5 rows=100000 loops=1)
         -> Covering index scan on C using idx_category_name  (cost=10.2 rows=100) (actual time=0.0305..0.108 rows=100 loops=1)
         -> Covering index lookup on P using category_id (category_id=c.category_id)  (cost=1.49 rows=972) (actual time=0.116..0.418 rows=1000 loops=100)
 ```

 ### Query Performance Comparison


| **Metric**            | **Before Index**                       | **After Index (categories covering)**               | **Improvement**     |
| --------------------- | -------------------------------------- | --------------------------------------------------- | ------------------- |
| **Logical Reads**     | ~100,000 (category × product lookups) | ~100,000 (almost same, only category scan smaller) | Slight ↓ (~0.1–1%) |
| **CPU Time (ms)**     | ~51.5                                 | ~49.5                                              | ~4% faster         |
| **Elapsed Time (ms)** | ~60                                   | ~58                                                | ~3% faster         |


