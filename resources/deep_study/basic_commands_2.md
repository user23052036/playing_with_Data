
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

# 🔹 Aggregate Functions

Aggregate functions operate on multiple rows and return **one value**.

## 1️⃣ COUNT()

Counts rows.

```sql
SELECT COUNT(emp_id) FROM employees;
```

Result: `10`

### Important:

#### COUNT(*)

Counts rows.
Includes NULL values.

#### COUNT(column)

Counts only non-null values in that column.

Example:

```sql
SELECT COUNT(*) FROM employees;
SELECT COUNT(bonus) FROM employees;
```

If bonus has NULLs:
Second count will be smaller.

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

Groups rows that have the same value in specified column(s).

Then aggregates per group.

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

SQL doesn’t know which row’s `fname`, `salary`, etc. to show.

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

### GROUP BY Multiple Columns

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

### HAVING clause

WHERE filters rows<br>
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

## LIMIT + OFFSET (Pagination Theory + Determinism)

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
SELECT → FROM → WHERE → GROUP BY → HAVING → ORDER BY → LIMIT/OFFSET
```

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

## JOIN TYPES — Deep Clarification

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

## UNION vs UNION ALL — what’s the difference (fast, clear)

**Short answer**

* `UNION` = combine rows **and remove duplicates**. (Slower.)
* `UNION ALL` = combine rows **without removing duplicates**. (Fast.)

**Execution steps (conceptual)**

1. Run first `SELECT` → produces result A
2. Run second `SELECT` → produces result B
3. Combine results (concatenate A + B)
4. If operator = `UNION` → do deduplication (sort or hash) to remove duplicate rows
   If operator = `UNION ALL` → skip dedupe, return combined rows immediately

**Performance implication**

* Deduplication costs CPU and memory (usually a sort or a hash aggregate). That cost grows with row count and distinctness.
* `UNION ALL` is typically much faster and scales better because it avoids that step.

---

## Concrete examples

Create two little example result sets:

```sql
-- assume these are simple queries; I'm showing results, not creating tables
-- Result of SELECT from A:
--  a
--  1
--  2
--  3

-- Result of SELECT from B:
--  a
--  2
--  3
--  4
```

`UNION` (deduplicates and sorts implicitly only for dedupe — ordering is not guaranteed unless you use ORDER BY):

```sql
SELECT a FROM A
UNION
SELECT a FROM B;
```

**Result** (order not guaranteed unless ORDER BY; logically contains each distinct value once):

```
1
2
3
4
```

`UNION ALL` (no dedupe — all rows from both queries are returned):

```sql
SELECT a FROM A
UNION ALL
SELECT a FROM B;
```

**Result**:

```
1
2
3
2
3
4
```

---

## INTERSECT and INTERSECT ALL — exact behavior

**`INTERSECT`**
Returns rows that appear in both result sets, **duplicates removed** (like set intersection).

Example:

```
A: 1,1,2,3
B: 1,2,2,4
```

`INTERSECT` → distinct rows present in both:

```
1
2
```

**`INTERSECT ALL`** (Postgres supports this)
Keeps duplicates up to the minimum multiplicity found in both sets.

Using the same A and B above:

* `1` appears 2 times in A and 1 time in B → appears min(2,1)=1 time in result
* `2` appears 1 time in A and 2 times in B → appears min(1,2)=1 time

So `INTERSECT ALL` result:

```
1
2
```

If counts were different, result would show duplicates equal to the minimum count across the sets.

---

## Important requirements for set operators (UNION / INTERSECT / EXCEPT)

When combining queries with set operators:

1. **Same number of columns** in each `SELECT`.
2. **Each column must have compatible data types** (Postgres checks positionally). Example: `int` can often be combined with `numeric` but not with `text` without cast.
3. **Column meanings are positional**, not by name. `SELECT id,name ... UNION SELECT name,id ...` will compile but will mix types/logical columns — don’t do it.
4. `ORDER BY` **applies to the final combined result only** — if you put `ORDER BY` inside the first select, many engines will error or it will be ignored. Use `ORDER BY` at the very end (or wrap subqueries).
5. Column aliases from the first `SELECT` are used for the final combined output in many DBs (so alias carefully).

---

## Examples tying it together (Postgres)

Sample tables:

```sql
-- Venue_Master: (Event_Id text, Wifi text, Capacity int)
-- Booking_Master: (Booking_Id text, Enquiry_Id text, Total_Amount int, Mode_of_Pay text)
```

Combine venues with wifi and online bookings (ID, Value):

```sql
SELECT Event_Id AS id, Capacity AS value
FROM Venue_Master
WHERE Wifi = 'Yes'

UNION ALL

SELECT Enquiry_Id AS id, Total_Amount AS value
FROM Booking_Master
WHERE Mode_of_Pay = 'Online'

ORDER BY value DESC, id DESC;
```

Notes on that query:

* `UNION ALL` used because we do not want deduplication and we want better performance.
* `ORDER BY value DESC, id DESC` ensures deterministic order when `value` ties exist.

---

## Practical tips / gotchas (ruthless & useful)

* If you **need no duplicates** → use `UNION`. But measure: if sets are large, consider alternatives (e.g., use joins/exists or dedupe earlier).
* If you **don’t care about duplicates** or duplicates are meaningful → use `UNION ALL`. Always prefer this for performance unless dedupe is required.
* If exact multiplicity matters, use `UNION ALL` + `GROUP BY` or `INTERSECT ALL` / `EXCEPT ALL` where supported.
* **Always** provide a final `ORDER BY` if the test or UI expects deterministic ordering. When equal on primary sort column, add a secondary sort (e.g., `ORDER BY value DESC, id DESC`).
* If types differ, cast explicitly: `SELECT col::text` or `CAST(col AS text)`.
* If you must dedupe but want to avoid full-sort memory pressure, consider `SELECT DISTINCT` on a smaller intermediate set, or use a `HASH` aggregation if the engine supports it.
* `UNION` ≈ `UNION ALL` + `DISTINCT` on the combined result. So `UNION` cost ≈ cost of producing A+B plus cost of `DISTINCT` (which is sort/hash).

---

## Quick checklist before writing set queries

* [ ] Same column count?
* [ ] Column types compatible (cast if needed)?
* [ ] Do I want duplicates? (`UNION ALL`) or not? (`UNION`)
* [ ] Final `ORDER BY` present for deterministic output?
* [ ] If performance matters: test `UNION` vs `UNION ALL` + `DISTINCT` vs join/exists.

---

## HAVING vs WHERE — Execution-Level Difference

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