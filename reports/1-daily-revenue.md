# 🗓️ Daily Revenue Report

### 1. Generate a daily report of the total revenue for a specific date.


-- '2025-08-10 12:22:36'
```sql
Select Date(order_date) , SUM(total_amount) AS DailyRevenue
FROM Orders 
WHERE Date(order_date) = '2025-08-10'
GROUP BY Date(order_date)   
```