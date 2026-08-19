# Relational Databases

A **relational database** is a database that stores data in **tables (also called relations)**.

Each table consists of:

- **Columns (fields)**: Attributes
- **Rows (records)**: Individual values of those attributes

## Syntax

- An **SQL statement** (commonly called an SQL Query) is a complete instruction sent to a database. A statement can create database structures, retrieve data, insert rows, update rows, or delete rows.
- **SQL syntax** is the set of rules for writing a valid statement. The words and punctuation in a statement must appear in the correct order.

### Punctuation

Punctuation symbols help organize an SQL statement:

- `;` ends an SQL statement. It also separates statements when more than one is submitted at the same time.
- `,` separates items, such as multiple column names or values.
- `(` and `)` enclose information that belongs together. Their exact purpose depends on where they are used, and they can only appear where the SQL syntax allows them.
- `.` connects a table name to one of its column names, as in `users.name`.
- Single quotation marks (`'`) surround text, as in `'Alice'`.

### Spaces and Line Breaks

Spaces and line breaks separate parts of a statement but normally do not change its meaning. It is customary to place major clauses on separate lines because that makes longer statements easier to read.

For example, these queries are equivalent:

```sql
SELECT name, email FROM users WHERE id = 1;
```

```sql
SELECT name, email
FROM users
WHERE id = 1;
```

### Keywords

**Keywords** are predefined words that have a special meaning in SQL. The first keyword usually identifies the broad action, and the following **clauses** add details. A clause is a section of a statement that performs a particular job and usually begins with a keyword, such as `SELECT`, `FROM`, `WHERE`, or `ORDER BY`.

Important: SQL keywords are **case-insensitive**, so `AS` and `as` have the same meaning.

Here is a rundown of the common SQL keywords:

- `CREATE`: Create a database object, such as a table or index.
  Example: `CREATE TABLE users (...)`

- `ALTER`: Change the structure of an existing table.
  Example: `ALTER TABLE users ADD COLUMN age INTEGER`

- `DROP`: Delete a database object.
  Example: `DROP TABLE users`

- `INSERT INTO`: Add new rows to a table.
  Example: `INSERT INTO users (...)`

- `VALUES`: Provide the row values for an `INSERT`.
  Example: `VALUES (1, 'Alice')`

- `SELECT`: Choose which columns or expressions to return.
  Example: `SELECT name, email`

- `FROM`: Choose the table the data comes from.
  Example: `FROM users`

- `JOIN`: Combine rows from another table, usually determined by an `ON` condition.
  Example: `JOIN orders`

- `CROSS JOIN`: Combine every row from one table with every row from another table.
  Example: `CROSS JOIN products`

- `ON`: Define how joined tables match.
  Example: `ON users.id = orders.user_id`

- `WHERE`: Filter individual rows before grouping.
  Example: `WHERE id = 1`

- `GROUP BY`: Group rows that share the same value.
  Example: `GROUP BY user_id`

- `HAVING`: Filter groups after `GROUP BY`.
  Example: `HAVING SUM(total) > 100`

- `ORDER BY`: Sort the result rows.
  Example: `ORDER BY name ASC`

- `LIMIT`: Return only a certain number of rows.
  Example: `LIMIT 10`

- `UPDATE`: Modify existing rows.
  Example: `UPDATE users`

- `SET`: Choose the new values during an `UPDATE`.
  Example: `SET email = 'x@example.com'`

- `DELETE FROM`: Remove rows from a table.
  Example: `DELETE FROM users`

- `AS`: Give a temporary name to a column or table.
  Example: `FROM users AS u`

- `DISTINCT`: Remove duplicate result rows.
  Example: `SELECT DISTINCT user_id`

- `AND` / `OR`: Combine filter conditions.
  Example: `WHERE id > 1 AND name = 'Bob'`

- `IN`: Match against a list of possible values.
  Example: `WHERE id IN (1, 2, 3)`

- `LIKE`: Match a text pattern.
  Example: `WHERE email LIKE '%@example.com'`

- `IS NULL`: Check for missing values.
  Example: `WHERE email IS NULL`

- `PRIMARY KEY`: Mark a column as the unique row identifier.
  Example: `id INTEGER PRIMARY KEY`

- `FOREIGN KEY`: Mark a column as referencing another table.
  Example: `FOREIGN KEY (user_id) REFERENCES users(id)`

- `REFERENCES`: Identify the table and column a foreign key points to.
  Example: `REFERENCES users(id)`

#### Clause Writing Order

```sql
SELECT columns
FROM table
JOIN other_table
  ON match_condition
WHERE row_filter
GROUP BY grouping_columns
HAVING group_filter
ORDER BY sorting_columns
LIMIT row_count;
```

Think of it like this:

- `SELECT` says what columns you want in the final result.
- `FROM` says the main table.
- `JOIN` adds another table.
- `ON` says how the tables match.
- `WHERE` filters individual rows.
- `GROUP BY` groups rows so aggregate functions can run per group.
- `HAVING` filters grouped results.
- `ORDER BY` sorts the final output.
- `LIMIT` restricts how many rows are returned.

#### Clause Processing Order

SQL clauses are not logically evaluated in the order in which they are written.

```text
FROM / JOIN / ON
        ↓
WHERE
        ↓
GROUP BY
        ↓
HAVING
        ↓
SELECT
        ↓
DISTINCT
        ↓
ORDER BY
        ↓
LIMIT / OFFSET
```

This is the **logical processing order** used to reason about a query. The database may use a different physical execution plan when optimizing it.

The is useful to know because of Aliases:

- An alias defined in `SELECT` generally cannot be used in `WHERE` because `WHERE` is logically processed before `SELECT`.
- An alias defined in `FROM` can be used in `SELECT` because `SELECT` is processed afterwards.

### Table Schema

A **Table schema** describes how data is organized in a database. It defines:

- The tables in the database
- The columns in each table
- The type of data each column can store
- Rules for the data, called **constraints**, such as `NOT NULL`, `UNIQUE`, and `PRIMARY KEY`
- Relationships between tables through foreign keys

In PostgreSQL, **schema** also has a specific meaning: a named namespace inside a database that contains objects such as tables, views, functions, and data types. For example, `public.users` refers to the `users` table in the `public` schema, and different schemas can contain objects with the same name.

The schema describes the structure of the database; the rows contain the actual data. The following is a MySQL-style example because the syntax for `ENUM` and automatically generated IDs differs between database systems:

```sql
CREATE TABLE users (
    id BIGINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
    username VARCHAR(50) NOT NULL UNIQUE,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) NOT NULL UNIQUE,
    password_hash CHAR(60) NOT NULL,
    status ENUM('pending', 'active', 'suspended', 'closed')
        NOT NULL DEFAULT 'pending',
    birth_date DATE,
    account_balance DECIMAL(10, 2) NOT NULL DEFAULT 0.00,
    reputation_score DOUBLE NOT NULL DEFAULT 0,
    is_email_verified BOOLEAN NOT NULL DEFAULT FALSE,
    bio TEXT,
    profile_picture BLOB,
    preferences JSON,
    last_login_at TIMESTAMP NULL,
    created_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP NOT NULL DEFAULT CURRENT_TIMESTAMP
        ON UPDATE CURRENT_TIMESTAMP,

    CONSTRAINT check_account_balance
        CHECK (account_balance >= 0),
    CONSTRAINT check_username_length
        CHECK (CHAR_LENGTH(username) >= 3)
);
```

Each column definition contains a name, a data type, and optionally one or more constraints. This schema demonstrates several common rules:

- `PRIMARY KEY` uniquely identifies each user, while `AUTO_INCREMENT` generates the next `id`.
- `NOT NULL` makes a value mandatory, and `UNIQUE` prevents duplicates.
- `ENUM` restricts `status` to one of four predefined values.
- `DEFAULT` supplies a value when an inserted row omits that column.
- `DECIMAL(10, 2)` stores exact monetary values with two decimal places.
- `CHECK` rejects values that fail a condition, such as a negative account balance or a username shorter than three characters.
- Columns without `NOT NULL`, such as `birth_date` and `bio`, are optional and may contain `NULL`.
- `created_at` records when the row is inserted, while `updated_at` is refreshed whenever MySQL updates the row.

### Data Types

A column's **data type** determines what kind of values it can store and which operations can be performed on those values. Common SQL types can be grouped by the kind of data they represent.

#### Numeric Types

| Type | Stores | Example |
|------|--------|---------|
| `SMALLINT`, `INTEGER`, `BIGINT` | Whole numbers in different size ranges | `42` |
| `DECIMAL(p, s)` or `NUMERIC(p, s)` | Exact decimal numbers; `p` is the total number of digits and `s` is the number after the decimal point | `DECIMAL(10, 2)` for `12345678.90` |
| `REAL`, `FLOAT`, `DOUBLE PRECISION` | Approximate floating-point numbers | `3.14159` |

#### Character Types

| Type | Stores | Example |
|------|--------|---------|
| `CHAR(n)` | Fixed-length text | `CHAR(2)` for a state code |
| `VARCHAR(n)` | Variable-length text with a maximum length | `VARCHAR(255)` for an email address |
| `TEXT` | Variable-length text, usually without a specified limit | `'A long description'` |

#### Boolean Type

| Type | Stores | Example |
|------|--------|---------|
| `BOOLEAN` | A true or false value | `TRUE` |

#### Temporal Types

Temporal types represent points in time or durations. `DATE`, `TIME`, and `TIMESTAMP` store temporal values, while `INTERVAL` represents an amount of time that can be used in temporal arithmetic.

| Type | Stores | Example |
|------|--------|---------|
| `DATE` | A calendar date | `'2026-08-18'` |
| `TIME` | A time of day | `'14:30:00'` |
| `TIMESTAMP` | A date and time | `'2026-08-18 14:30:00'` |
| `INTERVAL` | A duration expressed using units such as days, months, or hours | `INTERVAL '1 day'` in PostgreSQL |

For example, PostgreSQL can add a one-day interval to each recorded date:

```sql
SELECT
    id,
    recordDate,
    recordDate + INTERVAL '1 day' AS nextDate
FROM Weather;
```

If `recordDate` is `2015-01-01`, `nextDate` is `2015-01-02`. Inside the interval literal, `1` specifies the quantity and `day` specifies the temporal unit to add.

The syntax is database-specific:

```sql
-- PostgreSQL
recordDate + INTERVAL '1 day'

-- MySQL
recordDate + INTERVAL 1 DAY
```

#### Binary Types

| Type | Stores | Example |
|------|--------|---------|
| `BINARY`, `VARBINARY`, `BLOB` | Binary data such as bytes or files | An image's bytes |

#### Specialized Types

| Type | Stores | Example |
|------|--------|---------|
| `ENUM` | One value from a predefined set of allowed values | `'pending'` from `ENUM('pending', 'shipped', 'delivered')` |
| `JSON` | Structured JSON data | `'{"theme": "dark"}'` |
| `UUID` | A universally unique identifier | `'550e8400-e29b-41d4-a716-446655440000'` |

Type names, size limits, and behavior vary between database systems. For example, SQLite uses a flexible type system, while PostgreSQL, MySQL, and SQL Server provide different sets of specialized types. PostgreSQL provides `INTERVAL` as a data type, whereas MySQL uses `INTERVAL` as part of date-arithmetic expressions such as `recordDate + INTERVAL 1 DAY` rather than as a column type. `ENUM` is also database-specific: some systems provide it as a native type, while others represent the same rule with a `CHECK` constraint or a reference table. Consult the documentation for the database being used before relying on a particular type.

#### Type Casting

Type casting explicitly converts a value or expression to another data type. Standard SQL uses `CAST(expression AS type)`. PostgreSQL also supports the shorter `expression::type` syntax, but `CAST` is more portable across database systems.

```sql
SELECT CAST('42' AS INTEGER);
SELECT '42'::INTEGER; -- PostgreSQL shorthand
```

## Relationships between tables

### Primary Key

A **primary key** is a column, or combination of columns, whose value uniquely identifies each row.

Primary-key values cannot be duplicated or `NULL`. If an operation tries to insert a duplicate or `NULL` primary-key value, the database rejects that operation.

#### Single-Column Primary Key

In the `users` table, each user has a unique `id`:

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);
```

#### Composite Primary Key

A **composite primary key** uses two or more columns together to identify each row:

```sql
CREATE TABLE enrollments (
    student_id INTEGER,
    course_id INTEGER,
    enrolled_at DATE,
    PRIMARY KEY (student_id, course_id)
);
```

A student can enroll in multiple courses, and a course can contain multiple students. However, the same `student_id` and `course_id` combination cannot appear more than once.

### Foreign Key

A **foreign key** is a column, or combination of columns, that references a primary key or another unique key. It creates a relationship between a child table and the referenced parent table.

#### Single-Column Foreign Key

The `user_id` column in `orders` references the single-column primary key `users.id`:

```sql
CREATE TABLE orders (
    id INTEGER PRIMARY KEY,
    user_id INTEGER,
    total REAL,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

Each non-`NULL` `user_id` must match an existing `users.id`; otherwise, the database rejects the operation.

#### Composite Foreign Key

A foreign key that references a composite primary key must include all of its columns. The following table references the `(student_id, course_id)` primary key of `enrollments`:

```sql
CREATE TABLE grades (
    student_id INTEGER,
    course_id INTEGER,
    assignment_id INTEGER,
    grade DECIMAL(5, 2),

    PRIMARY KEY (student_id, course_id, assignment_id),
    FOREIGN KEY (student_id, course_id)
        REFERENCES enrollments(student_id, course_id)
);
```

The complete `(student_id, course_id)` pair must exist in `enrollments`. The foreign-key columns must correspond to the referenced columns in number, order, and compatible data types.

## CRUD Operation

CRUD =

- **Create**
- **Read**
- **Update**
- **Delete**

These are the core operations used to work with data.

### 1. Create

#### Create a table

In a normal Create Table operation, it is necessary to have the columns be enclosed in ()

```sql
CREATE TABLE users (
    id INTEGER PRIMARY KEY,
    name TEXT,
    email TEXT
);
```

Create a table from the output of another SQL query:

```sql
CREATE TABLE user_totals AS
SELECT user_id, SUM(total) AS total_spent
FROM orders
GROUP BY user_id;
```

#### Inserting a value

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

#### Getting NULL Values

`NULL` represents a missing or unknown value. Use `IS NULL` or `IS NOT NULL` to test for it; comparisons such as `birth_date = NULL` evaluate to `UNKNOWN` and do not match rows.

Suppose the `users` table contains:

| id | name    | birth_date   |
|---:|---------|--------------|
| 1  | Alice   | `1995-04-12` |
| 2  | Bob     | `NULL`       |
| 3  | Charlie | `2010-09-23` |

```sql
SELECT id, name, birth_date
FROM users
WHERE birth_date < '2000-01-01'
   OR birth_date IS NULL;
```

This returns Alice and Bob: Alice was born before 2000, and Bob's birth date is unknown. A `WHERE` clause keeps only rows whose condition evaluates to `TRUE`; this rule also applies to `UPDATE` and `DELETE`.

#### Filtering Duplicates

Use `DISTINCT` to remove duplicate rows from a query result:

```sql
SELECT DISTINCT name, email
FROM users;
```

SQL considers rows duplicates based on the complete combination of selected columns. Here, each unique `name` and `email` combination appears once.

### 3. Update

Modify existing rows:

With a `WHERE` clause:
Rows with value of `id` = 1 gets updated.

```sql
UPDATE users
SET email = 'newemail@example.com'
WHERE id = 1;
```

Without a `WHERE` clause:
Every row gets updated with same email value. This is often a mistake.

```sql
UPDATE users
SET email = 'x@example.com';
```

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

### Common Scalar Functions

Scalar functions transform one value at a time and can be used in clauses such as `SELECT`, `WHERE`, and `ORDER BY`.

| Function | Purpose |
|----------|---------|
| `CHAR_LENGTH(text)` | Count the characters in text |
| `LOWER(text)` / `UPPER(text)` | Convert text to lowercase or uppercase |
| `TRIM(text)` | Remove leading and trailing spaces |
| `ABS(number)` | Return a number's absolute value |
| `ROUND(number, places)` | Round a number to a specified number of decimal places |
| `COALESCE(value, fallback)` | Return the first value that is not `NULL` |

Functions can be combined and reused in the same query. This example trims each name, filters out names of 20 characters or fewer, and returns the remaining names in uppercase with their lengths:

```sql
SELECT
    UPPER(TRIM(name)) AS normalized_name,
    CHAR_LENGTH(TRIM(name)) AS name_length
FROM users
WHERE CHAR_LENGTH(TRIM(name)) > 20;
```

Function names can vary between database systems. PostgreSQL and SQLite support `LENGTH(text)`, SQL Server uses `LEN(text)` and excludes trailing spaces, and MySQL's `LENGTH(text)` counts bytes while `CHAR_LENGTH(text)` counts characters.

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

Multiple sort columns are applied from left to right. The first column has the highest priority; the next column is used only when earlier values are tied:

```sql
SELECT *
FROM users
ORDER BY name, id;
```

This sorts users by `name` alphabetically. If two users have the same name, `id` determines their order. When no direction is specified, each column defaults to ascending order.

### Limiting Results

```sql
SELECT *
FROM users
LIMIT 10;
```

Useful for pagination.

### Aggregations

Aggregate functions calculate a result from multiple rows. Suppose `orders` contains:

| id | user_id | total |
|---:|--------:|------:|
| 1  | 1       | 40    |
| 2  | 1       | 60    |
| 3  | 2       | 25    |
| 4  | 2       | 50    |

Without `GROUP BY`, all qualifying rows are treated as one group:

```sql
SELECT
    COUNT(*) AS order_count,
    AVG(total) AS average_total,
    SUM(total) AS total_sales,
    MIN(total) AS smallest_order,
    MAX(total) AS largest_order
FROM orders;
```

Result:

| order_count | average_total | total_sales | smallest_order | largest_order |
|------------:|--------------:|------------:|---------------:|--------------:|
| 4           | 43.75         | 175         | 25             | 60            |

Because all rows belong to one group, the query returns one row.

Common aggregate functions include:

- `COUNT(*)`: Count rows
- `AVG(column)`: Calculate the average
- `SUM(column)`: Calculate the total
- `MIN(column)`: Find the smallest value
- `MAX(column)`: Find the largest value

### GROUP BY

`GROUP BY` divides the rows into groups before calculating the aggregate functions. It creates one group for each unique combination of the columns listed in the `GROUP BY` clause.

For example, the following query calculates order statistics separately for each user:

```sql
SELECT
    user_id,
    COUNT(*) AS order_count,
    AVG(total) AS average_total,
    SUM(total) AS total_spent
FROM orders
GROUP BY user_id;
```

The rows are divided into these groups:

```text
user_id 1: totals 40, 60
user_id 2: totals 25, 50
```

The aggregate functions are calculated independently for each group:

| user_id | order_count | average_total | total_spent |
|--------:|------------:|--------------:|------------:|
| 1       | 2           | 50.00         | 100         |
| 2       | 2           | 37.50         | 75          |

The query returns one row for each unique `user_id`. Without aliases such as `AS total_spent`, the database generates labels for the aggregate columns; those labels vary by database, so aliases make the result clearer.

General syntax:

```sql
SELECT
    grouping_column_1,
    grouping_column_2,
    AGGREGATE_FUNCTION(column_1),
    AGGREGATE_FUNCTION(column_2)
FROM table_name
GROUP BY
    grouping_column_1,
    grouping_column_2;
```

Each column in `SELECT` should generally be either listed in `GROUP BY` or passed to an aggregate function. For example, this query fails in most databases:

**Invalid Query:**

```sql
SELECT
    id,
    user_id,
    SUM(total) AS total_spent
FROM orders
GROUP BY user_id;
```

Each user can have multiple orders and therefore multiple `id` values. SQL cannot choose one order ID to represent the entire user group. Adding `id` to `GROUP BY` would make the query valid, but because `id` uniquely identifies an order, it would create one group per order instead of one group per user.

The grouped columns determine what each result row represents. For example:

```sql
GROUP BY user_id
```

produces one row per user, while:

```sql
GROUP BY user_id, total
```

produces one row for every unique user-and-total combination.

### JOINs

`JOIN` means combine rows from tables based on a condition. That condition is usually an `ON` condition and decides which row from one table is joined with which row from the other table.

Users:

| id | name  | email               |
|---:|-------|---------------------|
| 1  | Alice | <alice@example.com> |
| 2  | Bob   | <bob@example.com>   |
| 3  | Charlie | <charlie@example.com> |

Orders:

| id  | user_id | total |
|----:|--------:|------:|
| 101 | 1       | 25    |
| 102 | 1       | 40    |
| 103 | 2       | 50    |

Combine rows from both tables:

```sql
-- JOIN rows that have users.id = orders.user_id 
SELECT users.name, orders.total
FROM users JOIN orders
ON users.id = orders.user_id; 
```

SQL looks at rows from the `users` table and rows from the `orders` table.

For each pair of rows from both tables where `users.id` equals `orders.user_id`, SQL combines those rows into one result row.

From each combined row, `SELECT` chooses the columns to return. It does not filter out rows; `WHERE` is used for row filtering.

Result:

| name  | total |
|-------|------:|
| Alice | 25    |
| Alice | 40    |
| Bob   | 50    |

Relational databases are powerful because you can connect data across tables.

#### Types of JOIN

The kind of join controls which rows remain when there is no match. A **match** occurs when the `ON` condition is true—for example, `users.id = orders.user_id`.

`SELECT` controls which columns are displayed. The join controls which combined rows exist.

##### INNER JOIN

An `INNER JOIN` returns only rows that have a match in both tables. `JOIN` without a modifier means `INNER JOIN`.

```sql
SELECT users.name, orders.total
FROM users
INNER JOIN orders
    ON users.id = orders.user_id;
```

Charlie is not returned because no order has `user_id = 3`. Alice appears twice because she has two matching orders.

##### LEFT JOIN (or LEFT OUTER JOIN)

A `LEFT JOIN` keeps every row from the table on the left side of `JOIN`. Matching columns from the right table are added; if there is no match, those right-table columns are `NULL`.

```sql
SELECT users.name, orders.total
FROM users
LEFT JOIN orders
    ON users.id = orders.user_id;
```

Result:

| name    | total |
|---------|------:|
| Alice   | 25    |
| Alice   | 40    |
| Bob     | 50    |
| Charlie | NULL  |

This is useful when you want all users, including users who have never placed an order.

##### RIGHT JOIN (or RIGHT OUTER JOIN)

A `RIGHT JOIN` is the mirror image of a `LEFT JOIN`: it keeps every row from the table on the right and fills unmatched left-table columns with `NULL`.

```sql
SELECT users.name, orders.total
FROM users
RIGHT JOIN orders
    ON users.id = orders.user_id;
```

In practice, you can usually rewrite a `RIGHT JOIN` as a `LEFT JOIN` by swapping the table order. Support for this join varies by database system.

##### FULL OUTER JOIN

A `FULL OUTER JOIN` keeps every row from both tables. Matching rows are combined; an unmatched row gets `NULL` in the columns belonging to the other table.

```sql
SELECT users.name, orders.total
FROM users
FULL OUTER JOIN orders
    ON users.id = orders.user_id;
```

`FULL OUTER JOIN` is useful when you need to see matches and non-matches on both sides. Support varies by database system; when it is unavailable, it can be approximated by combining a `LEFT JOIN` and a reverse `LEFT JOIN` with `UNION`.

##### CROSS JOIN

A `CROSS JOIN` does not use a matching condition. It returns every possible pair of rows: each left-table row is combined with each right-table row.

```sql
SELECT users.name, products.name
FROM users
CROSS JOIN products;
```

If `users` has 3 rows and `products` has 4 rows, the result has `3 × 4 = 12` rows. Use it intentionally because the result can grow quickly.

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
