
# Basic SELECT

* Select all rows/columns:

  ```sql
  SELECT * FROM employees;
  ```
* Select specific columns:

  ```sql
  SELECT fname, lname, dept FROM employees;
  ```

# Filtering: WHERE and comparisons

* Operators: `=`, `<>` (or `!=`), `<`, `>`, `<=`, `>=`, `IS NULL`, `IS NOT NULL`.
* Example:

  ```sql
  SELECT * FROM employees WHERE emp_id = 5;
  SELECT * FROM employees WHERE salary >= 50000;
  SELECT * FROM employees WHERE salary BETWEEN 50000 AND 60000;
  ```
* Logical connectors:

  ```sql
  -- AND
  SELECT * FROM employees WHERE dept = 'HR' AND salary > 46000;
  -- OR
  SELECT * FROM employees WHERE dept = 'HR' OR dept = 'Finance';
  -- IN
  SELECT * FROM employees WHERE dept IN ('IT','HR');
  -- NOT IN
  SELECT * FROM employees WHERE dept NOT IN ('IT','HR');
  ```

# Pattern matching: LIKE and ILIKE

* `%` matches any sequence (0 or more) of characters.
* `_` matches exactly one character.
* `LIKE` is case-sensitive (Postgres). `ILIKE` is case-insensitive.
* Examples from your practice:

  ```sql
  -- starts with A
  SELECT * FROM employees WHERE fname LIKE 'A%';

  -- ends with 'a'
  SELECT * FROM employees WHERE fname LIKE '%a';

  -- contains 'i'
  SELECT * FROM employees WHERE fname LIKE '%i%';

  -- second character is 'a'
  SELECT * FROM employees WHERE fname LIKE '_a%';

  -- exact pattern: department name with two characters
  SELECT * FROM employees WHERE dept LIKE '__';

  -- case-insensitive contains 'john'
  SELECT * FROM employees WHERE fname ILIKE '%john%';
  ```

# Distinct (unique values)

* `SELECT DISTINCT(column)` returns unique values for that column.

  ```sql
  SELECT DISTINCT dept FROM employees;
  ```

# Sorting and limits

* `ORDER BY` sorts results. Default ascending (`ASC`). Use `DESC` for descending.
* `LIMIT` restricts number of rows returned.

  ```sql
  SELECT * FROM employees ORDER BY fname, lname;
  SELECT * FROM employees ORDER BY salary;
  SELECT * FROM employees ORDER BY fname DESC LIMIT 3;
  ```

# Useful selection patterns you used

* Filter by department and salary:

  ```sql
  SELECT * FROM employees WHERE dept = 'HR' AND salary > 46000;
  ```
* Filter by membership:

  ```sql
  SELECT * FROM employees WHERE dept IN ('IT','HR');
  ```
* Range test:

  ```sql
  SELECT * FROM employees WHERE salary BETWEEN 50000 AND 60000;
  ```

---

Good. Now you're entering **aggregation + grouping**, which is where SQL actually becomes analytical.

Below are **structured notes** only on the new concepts you practiced.

---

# 🔹 Aggregate Functions

Aggregate functions operate on multiple rows and return **one value**.

## 1️⃣ COUNT()

Counts rows.

```sql
SELECT COUNT(emp_id) FROM employees;
```

Result: `10`

### Important:

* `COUNT(*)` → counts all rows.
* `COUNT(column)` → counts non-null values in that column.

---

## 2️⃣ SUM()

Adds numeric values.

```sql
SELECT SUM(salary) FROM employees;
```

Result: `521000.00`

Only works on numeric types.

---

## 3️⃣ AVG()

Average of numeric column.

```sql
SELECT AVG(salary) FROM employees;
```

Result: `52100.000000000000`

Note:

* PostgreSQL keeps high precision.
* Use `ROUND()` if needed:

```sql
SELECT ROUND(AVG(salary),2) FROM employees;
```

---

## 4️⃣ MIN()

Smallest value.

```sql
SELECT MIN(salary) FROM employees;
```

Result: `45000.00`

---

## 5️⃣ MAX()

Largest value.

```sql
SELECT MAX(salary) FROM employees;
```

Result: `61000.00`

---

## 🔹 GROUP BY

This is the most important new concept.

### What it does:

Groups rows that have the same value in specified column(s).

Then aggregates per group.

---

### ❌ Common Mistake You Made

```sql
SELECT * FROM employees GROUP BY dept;
```

Error:

> column must appear in GROUP BY or be used in aggregate function

#### Why?

Because once grouping happens, SQL doesn’t know which row’s `fname`, `salary`, etc. to show.

You must:

* Either group all selected columns
* Or aggregate them

---

## ✅ Correct Usage

### Count employees per department

```sql
SELECT dept, COUNT(dept)
FROM employees
GROUP BY dept;
```

Result:

```
Marketing | 2
Finance   | 2
IT        | 4
HR        | 2
```

---

### Sum of salaries per department

```sql
SELECT dept, SUM(salary)
FROM employees
GROUP BY dept;
```

Result:

```
Marketing | 102000
Finance   | 121000
IT        | 206000
HR        | 92000
```

---

## 🔹 Mental Model of GROUP BY

SQL execution order (simplified):

1. FROM
2. WHERE
3. GROUP BY
4. Aggregate Functions
5. SELECT
6. ORDER BY
7. LIMIT

So:

* First rows are filtered
* Then grouped
* Then aggregates are calculated

---

## 🔹 Advanced but Important

### GROUP BY multiple columns

```sql
SELECT dept, hire_date, COUNT(*)
FROM employees
GROUP BY dept, hire_date;
```

Now grouping is by unique combinations.

---

### Using ORDER BY with GROUP BY

```sql
SELECT dept, COUNT(*)
FROM employees
GROUP BY dept
ORDER BY COUNT(*) DESC;
```

---

## 🔹 Clean Version of Your Department Count

Better style:

```sql
SELECT dept,
       COUNT(*) AS total_employees
FROM employees
GROUP BY dept
ORDER BY total_employees DESC;
```

---

## 🔹 COUNT vs COUNT(column)

If dept had NULLs:

```sql
COUNT(dept)  -- ignores NULL
COUNT(*)     -- counts all rows
```

Very important difference in real datasets.

---

## 🔥 Common Beginner Mistakes (Avoid These)

1. Selecting non-grouped columns.
2. Forgetting that WHERE runs before GROUP BY.
3. Confusing DISTINCT with GROUP BY.
4. Using COUNT(column) when column might contain NULLs.

---

## 🔹 DISTINCT vs GROUP BY

These two:

```sql
SELECT DISTINCT dept FROM employees;
```

and

```sql
SELECT dept FROM employees GROUP BY dept;
```

produce same output.

But:

* DISTINCT → removes duplicates.
* GROUP BY → groups for aggregation.

---

## 🔥 Next Logical Concept

You are now ready for:

### HAVING clause

Because:

WHERE filters rows
HAVING filters groups

Example:

```sql
SELECT dept, COUNT(*)
FROM employees
GROUP BY dept
HAVING COUNT(*) > 2;
```

This would return only departments with more than 2 employees.

---

Good. That’s the right criticism.

You don’t want a rewrite.
You want a **delta patch** — only the concepts from screenshots that are missing in your original 300+ line notes.

I cannot see your previous full content directly (only partial metadata), so I’ll strictly extract **only the concepts visible in screenshots** that are typically missing in structured SQL notes and explain them properly in PostgreSQL context.

Below are the concepts that are usually not fully explained and were visible in screenshots.

---

# 🔴 ADD THESE SECTIONS TO YOUR EXISTING NOTES

---

# 1️⃣ LIMIT + OFFSET (Pagination Theory + Determinism)

You likely had LIMIT, but screenshots emphasize:

### ⚠ Important Concept: ORDER BY is REQUIRED for consistent LIMIT results

Without ORDER BY:

* PostgreSQL does NOT guarantee row order.
* LIMIT may return different rows across executions.

### Proper PostgreSQL Pagination

```sql
SELECT *
FROM employees
ORDER BY emp_id
LIMIT 10 OFFSET 20;
```

Meaning:

* Skip first 20 rows
* Return next 10 rows

### Execution Order Insight

Logical order:

```
FROM → WHERE → GROUP BY → HAVING → SELECT → ORDER BY → LIMIT/OFFSET
```

So LIMIT is applied **last**.

### ⚠ Performance Insight (important)

OFFSET becomes slow on large datasets because:

* PostgreSQL still scans skipped rows.
* For large pagination, use **keyset pagination** instead:

```sql
SELECT *
FROM employees
WHERE emp_id > 100
ORDER BY emp_id
LIMIT 10;
```

---

# 2️⃣ JOIN TYPES — Deep Clarification

Screenshots mention INNER, LEFT, RIGHT, FULL but did not explain behavior deeply.

Add this clarity:

---

## INNER JOIN

Returns only rows where join condition matches in both tables.

Mathematically:

```
A ∩ B
```

```sql
SELECT *
FROM orders o
INNER JOIN customers c
ON o.customer_id = c.customer_id;
```

---

## LEFT JOIN

Returns:

* All rows from left table
* Matched rows from right table
* NULLs where no match

Mathematically:

```
A ∩ B  +  (A − B)
```

Important use-case:
Finding unmatched rows:

```sql
SELECT c.*
FROM customers c
LEFT JOIN orders o
ON c.customer_id = o.customer_id
WHERE o.customer_id IS NULL;
```

This finds customers with NO orders.

---

## RIGHT JOIN

Mirror of LEFT JOIN.

In practice:

* Rarely used.
* You can rewrite it by swapping tables and using LEFT JOIN.

---

## FULL JOIN

Returns:

* All matching rows
* All unmatched rows from both sides

```sql
SELECT *
FROM a
FULL JOIN b
ON a.id = b.id;
```

Use-case:
Audit mismatches between two datasets.

---

# 3️⃣ UNION vs UNION ALL — Performance & Duplicate Logic

Your notes likely defined them but didn’t explain execution difference.

### UNION

* Removes duplicates.
* Internally performs sorting or hashing.
* Slower.

### UNION ALL

* Does NOT remove duplicates.
* Much faster.
* Use when duplicates are acceptable or expected.

### Execution Insight

PostgreSQL processes:

1. First SELECT
2. Second SELECT
3. Combines
4. If UNION → deduplicate

---

# 4️⃣ INTERSECT — Advanced Clarification

### Definition

Returns rows present in both queries.

Mathematically:

```
A ∩ B
```

### Important:

INTERSECT also removes duplicates.

PostgreSQL also supports:

```
INTERSECT ALL
```

Which keeps duplicates that exist in both sets.

Example:

```sql
SELECT col FROM t1
INTERSECT ALL
SELECT col FROM t2;
```

If value appears:

* 3 times in t1
* 2 times in t2
  Result → 2 times.

This is often missing in basic notes.

---

# 5️⃣ Set Operator Requirements (Critical but often skipped)

When using UNION / INTERSECT:

### Must satisfy:

1. Same number of columns
2. Compatible data types
3. Same column order

PostgreSQL checks types positionally, not by name.

Example of failure:

```sql
SELECT id, name FROM a
UNION
SELECT name, id FROM b;  -- wrong order
```

---

# 6️⃣ GROUP BY — Strict PostgreSQL Rule

PostgreSQL enforces strict SQL standard:

If a column appears in SELECT and:

* It is not inside aggregate
* It must appear in GROUP BY

This fails:

```sql
SELECT dept, salary
FROM employees
GROUP BY dept;
```

Why?
Because salary is neither grouped nor aggregated.

Correct:

```sql
SELECT dept, AVG(salary)
FROM employees
GROUP BY dept;
```

---

# 7️⃣ COUNT(*) vs COUNT(column)

This is often missing but critical.

### COUNT(*)

Counts rows.
Includes NULL values.

### COUNT(column)

Counts only non-null values in that column.

Example:

```sql
SELECT COUNT(*) FROM employees;
SELECT COUNT(bonus) FROM employees;
```

If bonus has NULLs:
Second count will be smaller.

---

# 8️⃣ HAVING vs WHERE — Execution-Level Difference

Screenshots mention difference but not execution logic.

### WHERE

Filters rows before grouping.

### HAVING

Filters groups after aggregation.

Invalid:

```sql
SELECT dept, COUNT(*)
FROM employees
WHERE COUNT(*) > 5
GROUP BY dept;
```

Correct:

```sql
SELECT dept, COUNT(*)
FROM employees
GROUP BY dept
HAVING COUNT(*) > 5;
```

---

# 9️⃣ GROUP BY Multiple Columns

Important structural concept:

Grouping by multiple columns creates unique combinations.

```sql
SELECT city, department, COUNT(*)
FROM employees
GROUP BY city, department;
```

This creates groups like:
(city1, dept1)
(city1, dept2)
(city2, dept1)
...

---

# 🔟 ORDER BY with GROUP BY

Order can use:

* Grouped column
* Aggregate alias

```sql
SELECT dept, SUM(salary) AS total
FROM employees
GROUP BY dept
ORDER BY total DESC;
```

PostgreSQL allows ordering by alias.

---

# 1️⃣1️⃣ Logical Query Processing Order (Very Important Concept)

This is missing in most beginner notes but visible implicitly in screenshots.

Actual logical order:

1. FROM
2. JOIN
3. WHERE
4. GROUP BY
5. HAVING
6. SELECT
7. DISTINCT
8. ORDER BY
9. LIMIT/OFFSET

Understanding this explains:

* Why WHERE cannot use aggregates
* Why HAVING can
* Why alias cannot be used in WHERE

---

# 1️⃣2️⃣ Conceptual Summary (Add to End of Notes)

### LIMIT

Controls number of rows returned.

### JOIN

Combines rows horizontally (columns increase).

### UNION / INTERSECT

Combines rows vertically (rows increase).

### GROUP BY

Reduces rows into aggregated summaries.

### HAVING

Filters aggregated results.

---

