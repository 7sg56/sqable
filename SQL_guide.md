# SQL CRUD Operations Guide

A comprehensive reference for Create, Read, Update, and Delete operations in SQL.

---

## Table of Contents
1. [Database Relationships](#database-relationships)
2. [CREATE Operations](#create-operations)
3. [READ Operations](#read-operations)
4. [Connecting Tables with JOINs](#connecting-tables-with-joins)
5. [UPDATE Operations](#update-operations)
6. [DELETE Operations](#delete-operations)

---

## CREATE Operations

### Creating Tables

#### Basic Table Creation
```sql
CREATE TABLE users (
    id INT PRIMARY KEY AUTO_INCREMENT,
    username VARCHAR(50) NOT NULL,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Table with Foreign Keys
```sql
CREATE TABLE orders (
    order_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    total_amount DECIMAL(10, 2),
    order_date DATE,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

#### Table with Constraints
```sql
CREATE TABLE products (
    product_id INT PRIMARY KEY,
    product_name VARCHAR(100) NOT NULL,
    price DECIMAL(10, 2) CHECK (price > 0),
    stock_quantity INT DEFAULT 0,
    category VARCHAR(50),
    CONSTRAINT unique_product UNIQUE (product_name, category)
);
```

### Inserting Data

#### Single Row Insert
```sql
INSERT INTO users (username, email)
VALUES ('john_doe', 'john@example.com');
```

#### Multiple Rows Insert
```sql
INSERT INTO users (username, email) VALUES
    ('alice', 'alice@example.com'),
    ('bob', 'bob@example.com'),
    ('charlie', 'charlie@example.com');
```

#### Insert with SELECT
```sql
INSERT INTO archived_orders (order_id, user_id, total_amount)
SELECT order_id, user_id, total_amount
FROM orders
WHERE order_date < '2023-01-01';
```

#### Insert with Default Values
```sql
INSERT INTO products (product_id, product_name, price)
VALUES (1, 'Widget', 19.99);
-- stock_quantity will use DEFAULT value of 0
```

---

## READ Operations

### Basic SELECT Queries

#### Select All Columns
```sql
SELECT * FROM users;
```

#### Select Specific Columns
```sql
SELECT username, email FROM users;
```

#### Select with Aliases
```sql
SELECT 
    username AS user_name,
    email AS user_email
FROM users;
```

### Filtering Data

#### WHERE Clause
```sql
SELECT * FROM products
WHERE price > 50 AND stock_quantity > 0;
```

#### IN Operator
```sql
SELECT * FROM users
WHERE username IN ('alice', 'bob', 'charlie');
```

#### BETWEEN Operator
```sql
SELECT * FROM orders
WHERE order_date BETWEEN '2024-01-01' AND '2024-12-31';
```

#### LIKE Pattern Matching
```sql
-- Starts with 'J'
SELECT * FROM users WHERE username LIKE 'J%';

-- Contains 'doe'
SELECT * FROM users WHERE username LIKE '%doe%';

-- Exactly 5 characters
SELECT * FROM users WHERE username LIKE '_____';
```

#### NULL Checks
```sql
SELECT * FROM users WHERE email IS NULL;
SELECT * FROM users WHERE email IS NOT NULL;
```

### Sorting and Limiting

#### ORDER BY
```sql
-- Ascending order (default)
SELECT * FROM products ORDER BY price ASC;

-- Descending order
SELECT * FROM products ORDER BY price DESC;

-- Multiple columns
SELECT * FROM products 
ORDER BY category ASC, price DESC;
```

#### LIMIT and OFFSET
```sql
-- First 10 records
SELECT * FROM users LIMIT 10;

-- Pagination: Skip 20, take 10
SELECT * FROM users LIMIT 10 OFFSET 20;
```

### Aggregate Functions

#### Basic Aggregates
```sql
SELECT 
    COUNT(*) AS total_users,
    COUNT(DISTINCT email) AS unique_emails
FROM users;

SELECT 
    AVG(price) AS average_price,
    MIN(price) AS lowest_price,
    MAX(price) AS highest_price,
    SUM(stock_quantity) AS total_stock
FROM products;
```

#### GROUP BY
```sql
SELECT 
    category,
    COUNT(*) AS product_count,
    AVG(price) AS avg_price
FROM products
GROUP BY category;
```

#### HAVING Clause
```sql
SELECT 
    category,
    COUNT(*) AS product_count
FROM products
GROUP BY category
HAVING COUNT(*) > 5;
```

### JOIN Operations

#### INNER JOIN
```sql
SELECT 
    orders.order_id,
    users.username,
    orders.total_amount
FROM orders
INNER JOIN users ON orders.user_id = users.id;
```

#### LEFT JOIN
```sql
SELECT 
    users.username,
    orders.order_id,
    orders.total_amount
FROM users
LEFT JOIN orders ON users.id = orders.user_id;
```

#### RIGHT JOIN
```sql
SELECT 
    users.username,
    orders.order_id
FROM users
RIGHT JOIN orders ON users.id = orders.user_id;
```

#### FULL OUTER JOIN
```sql
SELECT 
    users.username,
    orders.order_id
FROM users
FULL OUTER JOIN orders ON users.id = orders.user_id;
```

#### Multiple JOINs
```sql
SELECT 
    users.username,
    orders.order_id,
    products.product_name
FROM orders
INNER JOIN users ON orders.user_id = users.id
INNER JOIN order_items ON orders.order_id = order_items.order_id
INNER JOIN products ON order_items.product_id = products.product_id;
```

### Subqueries

#### Subquery in WHERE
```sql
SELECT * FROM products
WHERE price > (SELECT AVG(price) FROM products);
```

#### Subquery in FROM
```sql
SELECT category, avg_price
FROM (
    SELECT category, AVG(price) AS avg_price
    FROM products
    GROUP BY category
) AS category_averages
WHERE avg_price > 50;
```

#### EXISTS Subquery
```sql
SELECT * FROM users
WHERE EXISTS (
    SELECT 1 FROM orders 
    WHERE orders.user_id = users.id
);
```

---

## UPDATE Operations

### Basic Update

#### Update Single Column
```sql
UPDATE users
SET email = 'newemail@example.com'
WHERE username = 'john_doe';
```

#### Update Multiple Columns
```sql
UPDATE products
SET 
    price = 29.99,
    stock_quantity = 100
WHERE product_id = 1;
```

#### Update with Calculations
```sql
UPDATE products
SET price = price * 1.10
WHERE category = 'Electronics';
```

#### Update with CASE Statement
```sql
UPDATE products
SET price = CASE
    WHEN category = 'Electronics' THEN price * 1.15
    WHEN category = 'Clothing' THEN price * 1.10
    ELSE price * 1.05
END;
```

### Update with JOIN

```sql
UPDATE orders
INNER JOIN users ON orders.user_id = users.id
SET orders.discount = 0.10
WHERE users.username = 'premium_user';
```

### Update All Rows (Use Carefully!)

```sql
-- Updates every row in the table
UPDATE products
SET stock_quantity = 0;
```

### Conditional Updates

```sql
UPDATE users
SET status = 'inactive'
WHERE last_login < DATE_SUB(NOW(), INTERVAL 1 YEAR);
```

---

## DELETE Operations

### Basic Delete

#### Delete Specific Rows
```sql
DELETE FROM users
WHERE username = 'john_doe';
```

#### Delete with Multiple Conditions
```sql
DELETE FROM orders
WHERE order_date < '2023-01-01' 
  AND status = 'cancelled';
```

### Delete with Subquery

```sql
DELETE FROM products
WHERE product_id IN (
    SELECT product_id 
    FROM discontinued_products
);
```

### Delete with JOIN

```sql
DELETE orders
FROM orders
INNER JOIN users ON orders.user_id = users.id
WHERE users.status = 'deleted';
```

### Delete All Rows (Use Carefully!)

#### DELETE (Logs each deletion)
```sql
DELETE FROM temp_data;
```

#### TRUNCATE (Faster, resets auto-increment)
```sql
TRUNCATE TABLE temp_data;
```

### Cascading Deletes

When foreign keys are set up with CASCADE:
```sql
-- This will also delete related orders
DELETE FROM users WHERE id = 5;
```

---

## Table Management Operations

### Altering Tables

#### Add Column
```sql
ALTER TABLE users
ADD COLUMN phone_number VARCHAR(20);
```

#### Modify Column
```sql
ALTER TABLE users
MODIFY COLUMN email VARCHAR(150);
```

#### Drop Column
```sql
ALTER TABLE users
DROP COLUMN phone_number;
```

#### Add Constraint
```sql
ALTER TABLE products
ADD CONSTRAINT check_price CHECK (price >= 0);
```

#### Drop Constraint
```sql
ALTER TABLE products
DROP CONSTRAINT check_price;
```

### Dropping Tables

#### Drop Single Table
```sql
DROP TABLE IF EXISTS temp_users;
```

#### Drop Multiple Tables
```sql
DROP TABLE temp_table1, temp_table2;
```

### Renaming Tables

```sql
RENAME TABLE old_table_name TO new_table_name;
```

Or:
```sql
ALTER TABLE old_table_name RENAME TO new_table_name;
```

### Creating Indexes

```sql
-- Single column index
CREATE INDEX idx_username ON users(username);

-- Composite index
CREATE INDEX idx_name_email ON users(username, email);

-- Unique index
CREATE UNIQUE INDEX idx_email ON users(email);
```

### Dropping Indexes

```sql
DROP INDEX idx_username ON users;
```

---

## Best Practices

### Safety Tips

1. **Always use WHERE clauses** with UPDATE and DELETE operations
2. **Test with SELECT first** - Convert your WHERE clause to a SELECT to verify what will be affected
3. **Use transactions** for critical operations
4. **Backup before bulk operations**
5. **Use LIMIT** when testing DELETE/UPDATE queries

### Transaction Example

```sql
START TRANSACTION;

UPDATE accounts SET balance = balance - 100 WHERE account_id = 1;
UPDATE accounts SET balance = balance + 100 WHERE account_id = 2;

-- If everything looks good:
COMMIT;

-- If something went wrong:
-- ROLLBACK;
```

### Performance Tips

1. Use indexes on frequently queried columns
2. Avoid `SELECT *` in production - specify needed columns
3. Use JOINs instead of subqueries when possible
4. Use EXPLAIN to analyze query performance
5. Batch INSERT operations when adding multiple rows

---

## Common Patterns

### Upsert (Insert or Update)

MySQL:
```sql
INSERT INTO users (id, username, email)
VALUES (1, 'john_doe', 'john@example.com')
ON DUPLICATE KEY UPDATE
    username = VALUES(username),
    email = VALUES(email);
```

PostgreSQL:
```sql
INSERT INTO users (id, username, email)
VALUES (1, 'john_doe', 'john@example.com')
ON CONFLICT (id) 
DO UPDATE SET
    username = EXCLUDED.username,
    email = EXCLUDED.email;
```

### Soft Delete Pattern

```sql
-- Add deleted_at column
ALTER TABLE users ADD COLUMN deleted_at TIMESTAMP NULL;

-- Soft delete
UPDATE users SET deleted_at = CURRENT_TIMESTAMP WHERE id = 1;

-- Query only active records
SELECT * FROM users WHERE deleted_at IS NULL;
```

### Audit Trail Pattern

```sql
CREATE TABLE user_audit (
    audit_id INT PRIMARY KEY AUTO_INCREMENT,
    user_id INT,
    action VARCHAR(20),
    old_value TEXT,
    new_value TEXT,
    changed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

---

## Quick Reference

| Operation | Command | Example |
|-----------|---------|---------|
| Create Table | `CREATE TABLE` | `CREATE TABLE users (id INT, name VARCHAR(50));` |
| Insert Row | `INSERT INTO` | `INSERT INTO users VALUES (1, 'John');` |
| Select All | `SELECT *` | `SELECT * FROM users;` |
| Select Filtered | `SELECT ... WHERE` | `SELECT * FROM users WHERE id = 1;` |
| Update | `UPDATE ... SET` | `UPDATE users SET name = 'Jane' WHERE id = 1;` |
| Delete | `DELETE FROM` | `DELETE FROM users WHERE id = 1;` |
| Drop Table | `DROP TABLE` | `DROP TABLE users;` |
| Truncate | `TRUNCATE TABLE` | `TRUNCATE TABLE users;` |

---

*This guide covers standard SQL syntax. Some variations may exist between different database systems (MySQL, PostgreSQL, SQL Server, etc.).*