
# 📘 SUBQUERY IN POSTGRESQL — Detailed Notes (Q&A Format)

---

# 📌 What is a Subquery?

A **subquery** is a query written inside another query.

It is enclosed inside parentheses.

It can appear in:

* WHERE
* SELECT
* FROM
* HAVING

---

# 🔹 Example Table

```sql
CREATE TABLE employees (
    emp_id SERIAL PRIMARY KEY,
    name TEXT,
    dept TEXT,
    salary NUMERIC,
    hire_date DATE
);
```

Insert sample data:

```sql
INSERT INTO employees (name, dept, salary, hire_date) VALUES
('Souvik', 'IT', 60000, '2022-01-10'),
('Rahul', 'IT', 75000, '2021-03-15'),
('Ankit', 'HR', 50000, '2023-05-20'),
('Priya', 'HR', 55000, '2020-08-12'),
('Riya', 'Finance', 80000, '2019-11-01');
```

---

# 🟢 LEVEL 1 — Simple Scalar Subquery

### ❓ Question:

Find employees earning more than the average salary.

### ✅ Answer:

```sql
SELECT name, salary
FROM employees
WHERE salary > (
    SELECT AVG(salary)
    FROM employees
);
```

### 🔎 Explanation:

* Inner query calculates average salary.
* Outer query compares each employee’s salary to that value.
* Inner query returns a single value → scalar subquery.

---

# 🟢 LEVEL 2 — Subquery with Aggregate Condition

### ❓ Question:

Find employees earning the maximum salary.

### ✅ Answer:

```sql
SELECT name, salary
FROM employees
WHERE salary = (
    SELECT MAX(salary)
    FROM employees
);
```

---

# 🟢 LEVEL 3 — Subquery using IN

### ❓ Question:

Find employees who work in departments where salary > 70,000 exists.

### ✅ Answer:

```sql
SELECT name, dept
FROM employees
WHERE dept IN (
    SELECT dept
    FROM employees
    WHERE salary > 70000
);
```

### 🔎 Explanation:

* Inner query returns departments.
* Outer query fetches all employees in those departments.

---

# 🟡 LEVEL 4 — ANY and ALL

---

## 🔸 ANY

### ❓ Question:

Find employees earning more than ANY salary in HR department.

### ✅ Answer:

```sql
SELECT name, salary
FROM employees
WHERE salary > ANY (
    SELECT salary
    FROM employees
    WHERE dept = 'HR'
);
```

Meaning:
Salary > at least one HR employee salary.

---

## 🔸 ALL

### ❓ Question:

Find employees earning more than ALL HR employees.

### ✅ Answer:

```sql
SELECT name, salary
FROM employees
WHERE salary > ALL (
    SELECT salary
    FROM employees
    WHERE dept = 'HR'
);
```

Meaning:
Salary must be greater than highest HR salary.

---

# 🟡 LEVEL 5 — EXISTS

Good. Let’s slow it down and remove the confusion.

You’re not confused about syntax.
You’re confused about **how EXISTS actually works logically**.

---

# 🔴 First — Understand the Goal

### Question:

Find departments that have **at least one employee earning more than 75,000**.

This does NOT ask:

* Show employees.
* Show salaries.

It only asks:
👉 Which departments satisfy the condition.

---

### 🔵 Step 1 — Look at Example Data

| name   | dept    | salary |
| ------ | ------- | ------ |
| Souvik | IT      | 60000  |
| Rahul  | IT      | 75000  |
| Ankit  | HR      | 50000  |
| Priya  | HR      | 55000  |
| Riya   | Finance | 80000  |

Now answer mentally:

* IT → anyone > 75,000? ❌ (75000 is not greater)
* HR → anyone > 75,000? ❌
* Finance → anyone > 75,000? ✅ (80000)

Final Answer → **Finance**

That’s all the query is doing.

---

### 🔵 Now Understand the Query

```sql
SELECT DISTINCT dept
FROM employees e1
WHERE EXISTS (
    SELECT 1
    FROM employees e2
    WHERE e2.dept = e1.dept
    AND e2.salary > 75000
);
```

---

## 🧠 How PostgreSQL Executes This (Step-by-Step)

Think like this:

### Step 1:

Outer query picks first row:

Row → Souvik, IT

Now subquery runs:

```sql
SELECT 1
FROM employees e2
WHERE e2.dept = 'IT'
AND e2.salary > 75000;
```

Does any row match?

No.

So EXISTS = FALSE
IT is rejected.

---

### Step 2:

Next row → Rahul, IT

Same check → still false.

---

### Step 3:

Row → Riya, Finance

Subquery runs:

```sql
SELECT 1
FROM employees
WHERE dept = 'Finance'
AND salary > 75000;
```

Finds Riya (80000).

So EXISTS = TRUE
Finance is selected.

---

## 🟢 What EXISTS Really Means

EXISTS does NOT care what is selected.

It only checks:

👉 Does the subquery return at least one row?

If yes → TRUE
If no → FALSE

That’s it.

---

## 🔥 Why `SELECT 1` ?

Because EXISTS ignores actual values.

This would also work:

```sql
SELECT *
```

or

```sql
SELECT salary
```

Postgres only checks if a row exists.

---

## 🔵 Why `e2.dept = e1.dept` ?

This connects inner query to outer query.

Without it:

The subquery would just check if ANYONE in company earns > 75,000.

That’s different logic.

---

## 🔴 Simplified Mental Model

For each department:

> "Does there exist at least one employee in this department with salary > 75,000?"

If yes → keep it
If no → ignore it

---

## 🟡 Simpler Alternative Query (Same Logic)

To help you understand:

```sql
SELECT dept
FROM employees
WHERE salary > 75000
GROUP BY dept;
```

This gives same result.

But EXISTS is more general and powerful.

---

## 🔵 When Should You Use EXISTS?

Use EXISTS when:

* You only care about existence, not values
* Checking related rows
* Correlated condition needed
* Large dataset (EXISTS stops at first match)

---

# 🔴 Common Confusion

You might be thinking:

“Why not just use GROUP BY?”

Correct question.

In this simple case → GROUP BY is simpler.

But EXISTS becomes powerful when:

* Tables are different
* Complex relationships
* Checking presence of child records
* Filtering based on another table

---

# 🟠 LEVEL 6 — Correlated Subquery

Subquery depends on outer query row.

---

### ❓ Question:

Find employees earning more than the average salary of their department.

### ✅ Answer:

```sql
SELECT name, dept, salary
FROM employees e1
WHERE salary > (
    SELECT AVG(salary)
    FROM employees e2
    WHERE e2.dept = e1.dept
);
```

### 🔎 Explanation:

* Subquery runs for each row.
* Uses outer table alias.
* Harder but powerful.

---

# 🟠 LEVEL 7 — Subquery in SELECT Clause

---

### ❓ Question:

Show each employee with total number of employees in the company.

### ✅ Answer:

```sql
SELECT name,
       salary,
       (SELECT COUNT(*) FROM employees) AS total_employees
FROM employees;
```

---

# 🔴 LEVEL 8 — Subquery in FROM (Derived Table)

---

### ❓ Question:

Find departments whose average salary is greater than 60,000.

### ✅ Answer:

```sql
SELECT dept, avg_salary
FROM (
    SELECT dept, AVG(salary) AS avg_salary
    FROM employees
    GROUP BY dept
) AS dept_avg
WHERE avg_salary > 60000;
```

---

# 📌 Types of Subqueries

| Type       | Returns                     | Example           |
| ---------- | --------------------------- | ----------------- |
| Scalar     | Single value                | AVG, MAX          |
| Column     | One column multiple rows    | IN                |
| Row        | Single row multiple columns | Comparison        |
| Table      | Multiple rows & columns     | FROM subquery     |
| Correlated | Depends on outer query      | Dept-wise average |

---

# ⚠ Important Notes

1. Subqueries are slower than joins in large datasets.
2. Correlated subqueries can be expensive.
3. EXISTS is often faster than IN.
4. Use proper indexing.

---

# 🔥 Interview-Level Question

### ❓ Question:

Find the second highest salary.

### ✅ Answer:

```sql
SELECT MAX(salary)
FROM employees
WHERE salary < (
    SELECT MAX(salary)
    FROM employees
);
```

---

# 🚀 Final Understanding

Subquery =
“Use result of one query inside another query.”

Used when:

* Comparing against aggregate
* Filtering based on group result
* Checking existence
* Row-wise dependent logic

---
