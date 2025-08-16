# Write database functions to insert data in the product table around 100k rows


```sql
CREATE DEFINER=`root`@`localhost` PROCEDURE `insert_products_batch`(
    IN start_i INT,
    IN end_i INT,
    IN batch_size INT
)
BEGIN
    DECLARE i INT DEFAULT start_i;
    DECLARE counter INT;
    DECLARE values_sql MEDIUMTEXT;
    DECLARE rand_category INT;
    DECLARE category_max INT;

    -- Get max category_id to generate random category_id safely
    SELECT MAX(category_id) INTO category_max FROM categories;

    START TRANSACTION;

    WHILE i <= end_i DO
        SET counter = 0;
        SET values_sql = '';

        WHILE counter < batch_size AND i <= end_i DO
            -- Generate random category_id between 1 and category_max
            SET rand_category = FLOOR(1 + RAND() * category_max);

            SET values_sql = CONCAT(
                values_sql,
                IF(values_sql = '', '', ','),
                "('Product ", i, "', ",
                ROUND(RAND() * 100, 2), ", ",
                rand_category, ", ",
                FLOOR(RAND() * 100),
                ")"
            );

            SET i = i + 1;
            SET counter = counter + 1;
        END WHILE;

        SET @query = CONCAT(
            'INSERT INTO Products (name, price, category_id, stock_quantity) VALUES ',
            values_sql
        );

        PREPARE stmt FROM @query;
        EXECUTE stmt;
        DEALLOCATE PREPARE stmt;
    END WHILE;

    COMMIT;
END $$

DELIMITER ;
```