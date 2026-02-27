
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
