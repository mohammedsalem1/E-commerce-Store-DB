# Write database Procedure to insert data in customer tables around 100k  rows 
```sql

 DELIMITER $$

 CREATE PROCEDURE insert_customers__batch(
     IN start_i INT, 
     IN end_i INT, 
     IN batch_size INT
 )
 BEGIN
     DECLARE i INT DEFAULT start_i;
     DECLARE counter INT;
     DECLARE values_sql TEXT;

     START TRANSACTION;

    WHILE i < end_i DO
         SET counter = 0;
         SET values_sql = '';

         -- Build batch values
         WHILE counter < batch_size AND i < end_i DO
             SET values_sql = CONCAT(
                 values_sql,
                 IF(values_sql = '', '', ','),
                 "('first", i, "', 'last", i, "', 'email", i, "@example.com', 'password", i, "')"
             );

             SET i = i + 1;
             SET counter = counter + 1;
         END WHILE;

         -- Run batch insert
         SET @query = CONCAT(
             'INSERT INTO customers (first_name, last_name, email, password) VALUES ',
             values_sql
         );
         PREPARE stmt FROM @query;
         EXECUTE stmt;
         DEALLOCATE PREPARE stmt;
     END WHILE;

     COMMIT;
 END$$

 DELIMITER ;