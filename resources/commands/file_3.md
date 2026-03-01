
# 🔵 PART 1: WINDOW FUNCTIONS (Proper Notes)

---

## 1️⃣ What Problem Do Window Functions Solve?

Suppose you have this table:

```sql
sales
--------------------------------
id | date       | product | amount
--------------------------------
1  | 2025-01-01 | A       | 100
2  | 2025-01-02 | A       | 150
3  | 2025-01-03 | A       | 200
4  | 2025-01-01 | B       | 300
5  | 2025-01-02 | B       | 400
```

Now you want:

* Running total
* Ranking
* Compare current row with previous row

You CANNOT do this cleanly with GROUP BY.

Because:

> GROUP BY collapses rows
> Window functions keep rows and add calculation

That’s the entire difference.

---

## 2️⃣ What is a Window Function?

It performs calculation over a "window" of rows related to the current row
BUT does NOT reduce rows.

Syntax:

```sql
SELECT column,
       function() OVER (PARTITION BY ... ORDER BY ...)
FROM table;
```

Think of:

```
OVER() = define window
```

---

# 🔹 Example 1: Running Total

### Goal:

Running total of amount for each product.

```sql
SELECT product,
       date,
       amount,
       SUM(amount) OVER (
            PARTITION BY product
            ORDER BY date
       ) AS running_total
FROM sales;
```

### What happens:

For product A:

| date | amount | running_total |
| ---- | ------ | ------------- |
| Jan1 | 100    | 100           |
| Jan2 | 150    | 250           |
| Jan3 | 200    | 450           |

Notice:

Rows remain. Nothing collapsed.

---

# 🔹 Example 2: ROW_NUMBER()

```sql
SELECT product,
       amount,
       ROW_NUMBER() OVER (ORDER BY amount DESC) AS row_num
FROM sales;
```

Gives sequential numbering.

---

# 🔹 Example 3: RANK vs DENSE_RANK

Data:

| amount |
| ------ |
| 400    |
| 300    |
| 300    |
| 200    |

### RANK()

```sql
RANK() OVER (ORDER BY amount DESC)
```

Result:

```
1
2
2
4   ← skips 3
```

### DENSE_RANK()

```
1
2
2
3   ← no gap
```

---

# 🔹 Example 4: LAG and LEAD

Compare with previous row.

```sql
SELECT date,
       amount,
       LAG(amount) OVER (ORDER BY date) AS previous_day
FROM sales;
```

Output:

| date | amount | previous_day |
| ---- | ------ | ------------ |
| Jan1 | 100    | NULL         |
| Jan2 | 150    | 100          |
| Jan3 | 200    | 150          |

---

# 🔹 PARTITION BY vs No PARTITION

### Without PARTITION:

```sql
SUM(amount) OVER ()
```

Whole table = one window.

### With PARTITION:

```sql
SUM(amount) OVER (PARTITION BY product)
```

Each product = separate window.

---

# 🔹 Frame Clause (Advanced but Important)

This defines how many rows to look at.

Example: 3-day moving average

```sql
SELECT date,
       AVG(amount) OVER (
           ORDER BY date
           ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
       ) AS moving_avg
FROM sales;
```

Meaning:

Take current row + previous 2 rows.

---

# 🔹 Execution Order (Important for exams)

SQL execution:

1. FROM
2. WHERE
3. GROUP BY
4. HAVING
5. WINDOW FUNCTIONS
6. ORDER BY

Window functions run AFTER WHERE.

---

# 🟢 When Should You Use Window Functions?

Use when:

* Running totals
* Ranking
* Percentiles
* Compare rows
* Moving averages

If you are using self joins for previous row — you are doing it wrong.

---

---

# 🔵 PART 2: INDEXING (Proper Notes)

---

## 1️⃣ What is an Index?

An index is a data structure that speeds up data retrieval.

Without index:

Database scans entire table.

With index:

Database jumps directly to matching rows.

Analogy:

Book without index → read full book
Book with index → jump to page

---

## 2️⃣ How to Create Index

```sql
CREATE INDEX index_name
ON table_name (column_name);
```

Example:

```sql
CREATE INDEX idx_lastname
ON employees(last_name);
```

Now this query becomes faster:

```sql
SELECT * FROM employees
WHERE last_name = 'Smith';
```

---

## 3️⃣ Types of Indexes in PostgreSQL

### 🔹 B-Tree (Default)

Best for:

* =
* >
* <
* BETWEEN

Used 90% of time.

---

### 🔹 Hash Index

Only for equality:

```
WHERE id = 10
```

Rarely used.

---

### 🔹 GIN

For:

* Arrays
* Full-text search

---

### 🔹 BRIN

For very large tables
Uses block summaries instead of row index.

---

## 4️⃣ Composite Index (Multi-column)

```sql
CREATE INDEX idx_customer_date
ON orders(customer_id, order_date);
```

Important rule:

This works for:

```
WHERE customer_id = 101
```

AND

```
WHERE customer_id = 101 AND order_date >= '2025-01-01'
```

BUT NOT for:

```
WHERE order_date = '2025-01-01'
```

Order of columns matters.

---

## 5️⃣ Unique Index

```sql
CREATE UNIQUE INDEX idx_email
ON users(email);
```

Prevents duplicate emails.

---

## 6️⃣ Partial Index

Index only subset:

```sql
CREATE INDEX idx_active_users
ON users(email)
WHERE is_active = true;
```

Smaller, faster.

---

## 7️⃣ Expression Index

Index computed values:

```sql
CREATE INDEX idx_lower_email
ON users(LOWER(email));
```

Speeds up:

```sql
WHERE LOWER(email) = 'john@example.com';
```

---

# 🔴 Important Tradeoff

Index improves:

* SELECT speed

Index slows:

* INSERT
* UPDATE
* DELETE

Because index must be updated.

More indexes = slower writes.

---

# 🔹 How to Check Index Usage

```sql
SELECT indexname, idx_scan
FROM pg_stat_user_indexes;
```

If:

```
idx_scan = 0
```

Index is useless → consider dropping.

---

# 🔹 Rebuild Index

```sql
REINDEX TABLE employees;
```

Removes bloat.

---

# 🟢 When Should You Create Index?

Create if:

* Column used in WHERE frequently
* Used in JOIN
* Used in ORDER BY
* Foreign keys

Do NOT create if:

* Small table
* Column rarely used
* Column has few unique values

---

# 🧠 Final Mental Model

Window Functions = analytical calculations across rows
Indexing = performance optimization

They solve completely different problems.

---

