# 🛒E-commerce-Store-DB
## This project demonstrates a basic relational database design for an e-commerce platform using MySQL. It includes an ERD, schema diagram, SQL scripts to create and populate tables, sample reporting queries, and a denormalized version for performance testing.

---

## 📚 Features

 . Entity: Customers , Products , Categories , Order , Order-details
 
## 🗂️ Entity Relationship Diagram
   ![assets](assets/E-commerce-Tables.png)

##  Sql queries in e-commerce - table:   
 
### 1. Generate a daily report of the total revenue for a specific date.
<details>
  <summary>Query My-Sql</summary>
  
  ```

-- 2025-06-29 11:45:46
Select Date(order_date) , SUM(total_amount) AS DailyRevenue
   FROM Orders WHERE Date(order_date) = '2025-06-29'
   GROUP BY Date(order_date)   
  ```
  </details>

### 2. Generate a monthly report of the top-selling products in a given month.
<details>
  <summary>Query</summary>
  
  ```
Select DATE_FORMAT(order_date, '%Y-%m') AS Month, SUM(OD.quantity) AS TotalQuantity, 
     (Select P.name from Products P Where P.product_id = OD.product_id) AS Product_name , product_id 
     FROM orderdetails OD JOIN orders O 
      ON O.order_id = OD.order_id
       where DATE_FORMAT(order_date, '%Y-%m') = '2025-06'
     GROUP BY DATE_FORMAT(order_date, '%Y-%m') ,  product_id 
     ORDER BY TotalQuantity desc
     LIMIT 5  
  ```
  </details>  

###  3. Retrieve a list of customers who have placed orders totaling more than $500 in the past month.
<details>
  <summary>Query</summary>
  
  ```
  SELECT concat(C.first_name ,' ' , C.last_name) AS CustomerName, SUM(O.total_amount) AS TotalPrice , C.customer_id
  FROM Customers C join Orders O
    ON C.customer_id = O.customer_id
    WHERE O.Order_date >= DATE_SUB(NOW(), INTERVAL 1 MONTH)
  GROUP BY customer_id , CustomerName
  Having SUM(O.total_amount) > 500

```
</details>

### 4. Search for all products with the word "camera" in either the product name or description

<details>
  <summary>Query</summary>
  
  ```
CREATE fulltext index ft_index_Search ON products(name , description)

SELECT * from Products
where match(name , description)
AGAINST('phone')

  ```
</details>

