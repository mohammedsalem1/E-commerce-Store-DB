# 📈 monthly top-selling products

### Write an SQL query to generate a monthly report of the top-selling products in a given month..

```sql
Select P.name  ,MONTH(order_date) , SUM(OD.quantity) as total_quantity   
FROM Products P
JOIN orderdetails OD ON P.product_id = OD.product_id
JOIN orders O ON OD.order_id = O.order_id
WHERE MONTH(order_date) = 10
GROUP BY MONTH(order_date) ,  P.product_id
ORDER BY total_quantity desc 
```