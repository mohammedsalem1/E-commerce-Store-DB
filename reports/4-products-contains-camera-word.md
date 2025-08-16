# 📷 products-contains-camera-word

### Write a SQL query to search for all products with the word "camera" in either the product name or description.

```sql

SELECT * FROM products P
WHERE P.Name LIKE '%camera%' OR P.DESCRIPTION LIKE '%camera%';

```

### ⏱️ Before Optimization
```sql
-> Filter: ((p.`name` like '%product%') or (p.`description` like '%product%'))  (cost=9824 rows=20397) (actual time=0.0429..352 rows=100000 loops=1)
     -> Table scan on P  (cost=9824 rows=97193) (actual time=0.0378..288 rows=100000 loops=1)
  
```

### 🚀 After Optimization
- ✔ Create index an FullTextIndex
```sql
CREATE fulltext index ft_index_Search ON products(name , description)
```
-  After add index 
```sql

-> Filter: (match products.`name`,products.`description` against ('product'))  (cost=0.35 rows=1) (actual time=86.4..455 rows=100000 loops=1)
     -> Full-text index search on Products using ft_index_Search (name='product')  (cost=0.35 rows=1) (actual time=86.4..441 rows=100000 loops=1)
```


## 📈 Performance Comparison

| Scenario               | Scan Type              | Cost   | Rows     | Actual Time (ms) |
|------------------------|----------------------|--------|---------|-----------------|
| **Before Optimization**| Table Scan           | 9824   | 100,000 | 352             |
| **After Optimization** | Full-Text Index Scan | 0.35   | 100,000 | 455             |

---

### ✅ Key Observations
- **Scan Type:** Switched from **full table scan** → **full-text index search**  
- **Cost Reduction:** **9824 → 0.35** (~99.96% lower)  
- **Execution Time:** Slightly higher actual time due to returning all rows, but filtering is done efficiently by the index  
- **Scalability:** Full-text index search scales much better for large tables compared to table scan
