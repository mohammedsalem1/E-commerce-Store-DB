# Write SQL Query to List products that have low stock quantities of less than 10 quantities.

### Original Query
```sql
SELECT name , product_id , stock_quantity
  FROM products  
      where stock_quantity <= 10
```

### ⏱️ Before Optimization

```sql
-> Filter: (products.stock_quantity <= 10)  (cost=10106 rows=32394) (actual time=0.151..108 rows=11034 loops=1)
     -> Table scan on products  (cost=10106 rows=97193) (actual time=0.141..98.7 rows=100000 loops=1)
```

### 🚀 After Optimization


- ✔ Create index an stock_quantity
``` sql
CREATE INDEX idx_products_stock_quantity ON products (stock_quantity)
```

-  After add index 
```sql
-> Index range scan on products using idx_products_stock_quantity over (stock_quantity <= 10), with index condition: 
   (products.stock_quantity <= 10)  (cost=4966 rows=11034) (actual time=0.513..47.6 rows=11034 loops=1)
```

## 📈 Performance Comparison

| Scenario               | Scan Type        | Cost  | Rows   | Actual Time (ms) |
|------------------------|------------------|-------|--------|------------------|
| **Before Optimization**| Table Scan       | 10106 | 11,034 | 108              |
| **After Optimization** | Index Range Scan | 4966  | 11,034 | 47.6             |

---

### ✅ Key Improvements
- **Scan Type:** Switched from **full table scan** → **index range scan**
- **Cost Reduction:** **10106 → 4966** (~50% lower)
- **Execution Time:** **108 ms → 47.6 ms** (~56% faster)
