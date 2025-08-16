# 💰 [Customers with High Total Purchases]

###  Retrieve a list of customers who have placed orders totaling more than $500 in the past month.

```sql
Select concat(first_name , ' ' , last_name) as Customer , SUM(O.total_amount) as Top_selling
from customers C 
JOIN orders O ON C.customer_id = O.customer_id
WHERE O.order_date >= CURRENT_DATE - INTERVAL 1 MONTH
GROUP BY C.customer_id
Having Top_selling > 500
```


