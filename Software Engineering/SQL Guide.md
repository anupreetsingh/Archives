# Relational Databases

A **relational database** is a database that stores data in **tables (also called relations)**.

Each table consists of:

- **Columns (fields)**: Attributes
- **Rows (records)**: Individual values of those attributes

## Schema

A **schema** is the blueprint for your tables.

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    email TEXT
);
```

This creates a **users** table

## Relationships between tables

**users Table**

| id | name  | email             |
|---:|-------|-------------------|
| 1  | Alice | <alice@example.com> |
| 2  | Bob   | <bob@example.com>   |

**orders Table**

| id  | user_id | total |
|----:|--------:|------:|
| 101 | 1       | 25.99 |
| 102 | 2       | 49.99 |

### Primary Key

A **primary key** is a column that uniquely identifies each row.

No two rows can have the same primary key.

In the **users Table**, the `id` is the primary key

### Foreign Key

A **foreign key** is a column that references another table's primary key. This creates a relationship between tables.

```sql
CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER,
    total REAL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

The `user_id` column in the **orders** table is set as a foreign key and it refers to the `id` column in the **users** table.

## CRUD Operation

CRUD =

- **Create**
- **Read**
- **Update**
- **Delete**

These are the core operations used to work with data.

### 1. Create

Create a table:

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    email TEXT
);
```

Insert values for specific attributes in one row:

```sql
INSERT INTO users (id, name, email)
VALUES (1, 'Alice', 'alice@example.com');
```

Insert values for specific attributes in multiple rows:

```sql
INSERT INTO users (id, name)
VALUES
    (2, 'Bob'),
    (3, 'Charlie');
```

For these two rows only the `id` and `name` columns are populated here and the `email` column is left empty.

### 2. Read

Retrieve all rows with all columns:

```sql
SELECT *
FROM users;
```

Retrieve all rows with specific columns:

```sql
SELECT name, email
FROM users;
```

Retrieve Filtered rows with all columns based on values of attributes in the rows:

```sql
SELECT *
FROM users
WHERE id = 1;
```

Retrieve Filtered rows with all columns based on multiple conditions on the values of the attributes in those rows:

```sql
SELECT *
FROM users
WHERE id > 1
AND name = 'Bob';
```

### 3. Update

Modify existing rows:

```sql
UPDATE users
SET email = 'newemail@example.com'
WHERE id = 1;
```

Without a `WHERE` clause:

```sql
UPDATE users
SET email = 'x@example.com';
```

Every row gets updated with same email value. This is often a mistake.

### 4. Delete

Delete specific rows:

```sql
DELETE FROM users
WHERE id = 1;
```

Delete all rows:

```sql
DELETE FROM users;
```

The table remains; only the data is removed.

## Other Helpful Operations

### Sorting Results

Ascending order:

```sql
SELECT *
FROM users
ORDER BY name ASC;
```

Descending order:

```sql
SELECT *
FROM users
ORDER BY name DESC;
```

### Limiting Results

```sql
SELECT *
FROM users
LIMIT 10;
```

Useful for pagination.

### Aggregations

For performing operations on all values in the rows of specific columns.

Count rows:

```sql
SELECT COUNT(*)
FROM users;
```

Returns one value: Number of rows

Average:

```sql
SELECT AVG(total)
FROM orders;
```

Sum:

```sql
SELECT SUM(total)
FROM orders;
```

Maximum:

```sql
SELECT MAX(total)
FROM orders;
```

### GROUP BY

Syntax:

```sql
SELECT
    non_aggregate_column_1,
    non_aggregate_column_2,
    ...,
    AGGREGATE_FUNCTION(column_a),
    AGGREGATE_FUNCTION(column_b),
    ...
FROM table_name
GROUP BY
    non_aggregate_column_1,
    non_aggregate_column_2,
    ...;
```

`GROUP BY` creates one group for each unique combination of the columns listed after `GROUP BY`.

Then `SELECT` returns one row per group:

- the grouped column values
- the aggregate results calculated for each group

Suppose we want total spending per user:

```sql
SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id;
```

From the orders table, Group by each unique value of `user_id` and then apply the aggregate function SUM(total) to each group and SELECT the results.

Result:

| user_id | sum |
|--------:|----:|
| 1       | 100 |
| 2       | 75  |

You can also save save this result as a new a table **user_totals**:

```sql
CREATE TABLE user_totals AS
SELECT user_id, SUM(total)
FROM orders
GROUP BY user_id;
```

### JOINs

`JOIN` is one of SQL's most important features.

Users:

| id | name  | email               |
|---:|-------|---------------------|
| 1  | Alice | <alice@example.com> |
| 2  | Bob   | <bob@example.com>   |

Orders:

| id  | user_id | total |
|----:|--------:|------:|
| 101 | 1       | 25    |
| 102 | 1       | 40    |
| 103 | 2       | 50    |

Combine rows from both tables:

```sql
SELECT users.name, orders.total
FROM users
JOIN orders
ON users.id = orders.user_id;
```

SQL looks at rows from the `users` table and rows from the `orders` table.

For each pair of rows from both tables where `users.id` equals `orders.user_id`, SQL combines those rows into one result row.

From each combined row, it SELECTS rows for `users.name` and `orders.total` columns to return.

Result:

| name  | total |
|-------|------:|
| Alice | 25    |
| Alice | 40    |
| Bob   | 50    |

Relational databases are powerful because you can connect data across tables.

### Example Use of Operations

Find all orders made by Alice:

```sql
SELECT orders.*
FROM users
JOIN orders
ON users.id = orders.user_id
WHERE users.name = 'Alice';
```

Result:

| id  | user_id | total |
|----:|--------:|------:|
| 101 | 1       | 25    |
| 102 | 1       | 40    |

## Indexes

Indexes speed up lookups.

Without an index:

```sql
SELECT *
FROM users
WHERE email = 'alice@example.com';
```

The database may scan every row.

With an index:

```sql
CREATE INDEX idx_users_email
ON users(email);
```

The database can find rows much faster.

Tradeoff:

- Faster reads
- Slightly slower inserts/updates
- More storage

## ORMs

In a backend application, for example using FastAPI:

```python
session.query(User).filter(User.id == 1)
```

An ORM like SQLAlchemy translates that into SQL:

```sql
SELECT *
FROM users
WHERE id = 1;
```

The database, such as PostgreSQL or SQLite, executes the SQL and returns the results.
