# Write database Procedure to insert data in  categories table around 100 row

```sql
CREATE PROCEDURE PROCEDURE insert_categories()
BEGIN
   DECLARE i INT DEFAULT 1;

   WHILE i <= 100 DO
      INSERT INTO Categories (name)
      VALUES (CONCAT('Category ', i));
      SET i = i + 1;
   END WHILE;
END
```
