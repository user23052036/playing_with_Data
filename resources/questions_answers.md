
# 🔵 PART 1 — DDL / Constraints Section

### **Question:**

**Which of the following are DDL statements in PostgreSQL?**

Options:

* Drop table table_name;
* Delete from table_name;
* All the options
* Truncate table table_name;

✅ **Correct:**
Drop table table_name;
Truncate table table_name;

📌 DDL changes structure. DELETE is DML, not DDL.

---

### **Question:**

**Which SQL command is used to modify existing records in a PostgreSQL table?**

Options:

* INSERT
* MODIFY
* ALTER
* UPDATE

✅ **Correct:** UPDATE

📌 UPDATE changes existing rows.

---

### **Question:**

**Numeric datatype is used to store ________**

Options:

* Whole numbers
* Exact numeric values with optional decimal precision
* Natural numbers
* Date

✅ **Correct:** Exact numeric values with optional decimal precision

📌 NUMERIC/DECIMAL store precise decimal values.

---

### **Question:**

**What is the Default format for ‘Date’ datatype?**

Options:

* MM-YYYY-DD
* NONE
* DD-MON-YYYY
* YYYY-MM-DD

✅ **Correct:** YYYY-MM-DD

📌 PostgreSQL uses ISO date format.

---

### **Question:**

**Which of the following is a valid reference option for deleting rows in PostgreSQL when using ON DELETE in a foreign key constraint?**

Options:

* Restrict
* Set Null
* All the options
* Cascade

✅ **Correct:** All the options

📌 All three are valid ON DELETE actions.

---

### **Question:**

**Which of the following is a column-level constraint in PostgreSQL?**

Options:

* All the options
* column3 UNIQUE
* FOREIGN KEY (column1) REFERENCES table2(column2)
* PRIMARY KEY (column1, column2)

✅ **Correct:** column3 UNIQUE

📌 UNIQUE applied directly to a column is column-level.

---

### **Question:**

**Which constraint ensures that a column cannot have NULL values in PostgreSQL?**

Options:

* CHECK
* DEFAULT
* NOT NULL
* UNIQUE

✅ **Correct:** NOT NULL

📌 Explicitly prevents NULL values.

---

### **Question:**

**To remove a relation from PostgreSQL database, we use ________ command.**

Options:

* All the options
* Delete
* Drop
* Truncate

✅ **Correct:** Drop

📌 DROP removes table structure.

---

### **Question:**

**What does the ON DELETE CASCADE option do in PostgreSQL?**

Options:

* Ignores deletion of parent rows
* Deletes child rows automatically when parent row is deleted
* Prevents deletion of parent rows
* Sets child foreign key to NULL when parent row is deleted

✅ **Correct:** Deletes child rows automatically when parent row is deleted

📌 CASCADE propagates delete.

---

### **Question:**

**Which relationship type means that a single row in one table relates to multiple rows in another table?**

Options:

* One-to-One
* Many-to-Many
* None of the options
* One-to-Many

✅ **Correct:** One-to-Many

📌 One parent → many child rows.

---

# 🔵 PART 2 — Post-Quiz DML (Rewritten Properly)

---

## Question 1

**Which of the following SQL commands are part of DML (Data Manipulation Language) that Watson should focus on for performing DML operations?**

Options:

A. ALTER
B. CREATE
C. UPDATE
D. DELETE

✅ **Correct:** C and D

📌 DML modifies data inside tables. `UPDATE` and `DELETE` manipulate rows. `ALTER` and `CREATE` are DDL (structure changes).

---

## Question 2

**If a record with the same `product_id` already exists, Parker wants to update the `price` instead of inserting a duplicate row. Which PostgreSQL statement achieves this?**

Options:

A.

```sql
INSERT INTO products VALUES (101, 50)
ON DUPLICATE KEY UPDATE price = 50;
```

B.

```sql
UPDATE products SET price = 50 WHERE product_id = 101;
```

C.

```sql
INSERT INTO products (product_id, price)
VALUES (101, 50)
ON CONFLICT (product_id)
DO UPDATE SET price = EXCLUDED.price;
```

D.

```sql
MERGE INTO products VALUES (101, 50);
```

✅ **Correct:** C

📌 PostgreSQL uses `INSERT ... ON CONFLICT ... DO UPDATE` for UPSERT. Option A is MySQL syntax. B does not insert if row is missing. D is invalid in this context.

---

## Question 3

**Ethan wants to update the `orders` table to mark orders as 'Shipped' only if their current status is 'Processing'. Which query should he use?**

Options:

A.

```sql
UPDATE orders SET status = 'Shipped';
```

B.

```sql
UPDATE orders SET status = 'Shipped' WHERE status = 'Processing';
```

C.

```sql
ALTER orders SET status = 'Shipped';
```

D.

```sql
INSERT INTO orders (status) VALUES ('Shipped');
```

✅ **Correct:** B

📌 The `WHERE` clause ensures only rows with status `'Processing'` are updated.

---

## Question 4

**The `stock` column in the `products` table has a default value of 50. Insert only `product_id` and `product_name` so that `stock` uses its default value. Which statement is correct?**

Options:

A.

```sql
INSERT INTO products VALUES (101, 'Laptop');
```

B.

```sql
INSERT INTO products (product_id, product_name)
VALUES (101, 'Laptop');
```

C.

```sql
INSERT INTO products (stock)
VALUES (50);
```

D.

```sql
INSERT products (product_id, product_name)
VALUES (101, 'Laptop');
```

✅ **Correct:** B

📌 Omitting the `stock` column allows PostgreSQL to apply the default value automatically.

---

## Question 5

**Remove all employees who have NULL in the `phoneno` column. Which statement should be executed?**

Options:

A.

```sql
DELETE FROM employees WHERE phoneno = NULL;
```

B.

```sql
DELETE employees WHERE phoneno IS NULL;
```

C.

```sql
DELETE FROM employees WHERE phoneno IS NULL;
```

D.

```sql
REMOVE FROM employees WHERE phoneno IS NULL;
```

✅ **Correct:** C

📌 `IS NULL` must be used. `= NULL` does not work in SQL.

---

## Question 6

**Which statement is correct regarding PostgreSQL’s handling of MERGE/UPSERT functionality?**

Options:

A. PostgreSQL fully supports MERGE with MySQL syntax.
B. PostgreSQL does not support MERGE directly but supports `INSERT ... ON CONFLICT ... DO UPDATE`.
C. PostgreSQL requires triggers for UPSERT.
D. PostgreSQL automatically updates duplicates without syntax.

✅ **Correct:** B

📌 PostgreSQL implements UPSERT using `ON CONFLICT`, not MySQL-style MERGE syntax.

---

## Question 7

**An INSERT operation fails because the `phoneno` column is defined as `BIGINT NOT NULL`, but the query omitted the `phoneno` value. What is the reason?**

Options:

A. Data type mismatch
B. Value for `phoneno` is missing
C. Table does not exist
D. Duplicate primary key

✅ **Correct:** B

📌 NOT NULL constraint requires a value for that column.

---

## Question 8

**Increase the salary of employees in the 'Sales' department by 10%. Which query is correct?**

Options:

A.

```sql
UPDATE employees SET salary = salary + 10 WHERE department = 'Sales';
```

B.

```sql
UPDATE employees SET salary = salary * 1.1 WHERE department = 'Sales';
```

C.

```sql
UPDATE employees SET salary = 1.1 WHERE department = 'Sales';
```

D.

```sql
ALTER employees SET salary = salary * 1.1;
```

✅ **Correct:** B

📌 Multiplying by 1.1 increases salary by 10%.

---

## Question 9

**What action does the `COMMIT` statement perform in a transaction?**

Options:

A. Cancels all changes
B. Saves current transaction as savepoint
C. Makes all pending data changes permanent and ends the transaction
D. Deletes the table

✅ **Correct:** C

📌 `COMMIT` finalizes changes and ends the transaction block.

---

## Question 10

**Insert a valid record into the `student` table with columns (`stud_id`, `address`, `name`, `dob`). Which statement is correct?**

Options:

A.

```sql
INSERT student VALUES (101, 'Smith', '100 Main Street', '1994-02-01');
```

B.

```sql
INSERT INTO student (stud_id, address, name, dob)
VALUES (101, '100 Main Street', 'Smith', '1994-02-01');
```

C.

```sql
INSERT INTO student VALUES ('Smith', 101, '1994-02-01', '100 Main Street');
```

D.

```sql
INSERT INTO student (name, dob)
VALUES ('Smith', '1994-02-01');
```

✅ **Correct:** B

📌 Correct column order and proper `INSERT INTO` syntax required.

---

# 🔵 PART 3 — Pre-Quiz Select (Rewritten Properly)

---

## Question 1

**Which statements are true regarding constraints in PostgreSQL?**

Options:

A. A constraint can be disabled even if the constraint column contains data.
B. A column with the UNIQUE constraint can contain NULL values.
C. A PRIMARY KEY column can contain multiple NULL values.
D. A NOT NULL constraint allows one NULL value.

✅ **Correct:**
A and B

📌 PostgreSQL allows disabling constraints and UNIQUE columns can contain multiple NULL values. PRIMARY KEY and NOT NULL do not allow NULL.

---

## Question 2

**What does the following condition do?**

```sql
order_date BETWEEN '2024-01-01' AND '2024-01-31'
```

Options:

A. Retrieves orders strictly between the two dates (excluding boundaries).
B. Retrieves all orders from 2024-01-01 through 2024-01-31, including boundaries.
C. Retrieves only orders placed on 2024-01-15.
D. Causes a syntax error.

✅ **Correct:** B

📌 `BETWEEN` in SQL is inclusive of both boundary values.

---

## Question 3

**Which is the correct syntax to insert values into specific columns of a table?**

Options:

A. `INSERT table_name VALUES (val1, val2);`
B. `INSERT INTO table_name VALUES (val1, val2);`
C. `INSERT INTO table_name (col1, col2) VALUES (val1, val2);`
D. `INSERT table_name (col1, col2) VALUES (val1, val2);`

✅ **Correct:** C

📌 Proper syntax requires `INSERT INTO` and explicit column list before VALUES.

---

## Question 4

**What happens if ROLLBACK TO SAVEPOINT is issued after COMMIT?**

Options:

A. The rollback works normally.
B. The transaction partially rolls back.
C. The rollback generates an error.
D. The savepoint is recreated automatically.

✅ **Correct:** C

📌 COMMIT ends the transaction and removes all savepoints. Rolling back afterward causes an error.

---

## Question 5

**DELETE removes rows from a table. Which clause is used to remove specific rows?**

Options:

A. GROUP BY
B. HAVING
C. WHERE
D. ORDER BY

✅ **Correct:** C

📌 `WHERE` filters rows. Without it, all rows are deleted.

---

## Question 6

**What does the condition `emp_name LIKE '%N'` return?**

Options:

A. Names starting with N
B. Names containing N anywhere
C. Names that end with N
D. Names exactly equal to N

✅ **Correct:** C

📌 `%` means any sequence before the letter N. So the name must end with N.

---

## Question 7

**What does the condition `phone_no IS NULL` return?**

Options:

A. Customers whose phone number column contains NULL
B. Customers whose phone number is an empty string
C. Customers whose phone number is 0
D. Customers whose phone number starts with NULL

✅ **Correct:** A

📌 `IS NULL` checks for absence of value, not empty string or zero.

---

## Question 8

**Which clause is used to sort the result set of a SELECT query?**

Options:

A. SORT
B. GROUP BY
C. ORDER BY
D. HAVING

✅ **Correct:** C

📌 `ORDER BY` arranges results in ascending (default) or descending order.

---

## Question 9

**Which command removes all data from a table but keeps the table structure intact and allows rollback?**

Options:

A. DROP TABLE employee;
B. TRUNCATE TABLE employee;
C. DELETE FROM employee;
D. REMOVE employee;

✅ **Correct:** C

📌 `DELETE` can be rolled back in a transaction. `TRUNCATE` and `DROP` are not safely reversible in normal quiz context.

---

## Question 10

**Which statement is true when DROP TABLE is executed?**

Options:

A. Only table data is deleted.
B. Table structure remains intact.
C. The table structure and its deleted data cannot be rolled back and restored once executed.
D. The table is temporarily hidden.

✅ **Correct:** C

📌 `DROP TABLE` permanently removes the table definition and its data.

---


# 🔵 PART 4 — Advanced SELECT / JOIN / LIKE / LIMIT Section (30 Questions)

---

## Question 1

**Which query correctly returns employees whose names contain a literal underscore character (`_`)?**

**Options:**
A. `SELECT * FROM employees WHERE name LIKE '%\_%';`
B. `SELECT * FROM employees WHERE name LIKE '%_%';`
C. `SELECT * FROM employees WHERE name SIMILAR TO '%_%';`
D. `SELECT * FROM employees WHERE name LIKE '%!_%' ESCAPE '!';`

✅ **Correct:** D
📌 `_` is a single-character wildcard in `LIKE`. To match a literal underscore you must escape it and declare the escape character (example uses `!`).

---

## Question 2

**You need to count employees per department. Which SQL clause is required to group rows by department before applying an aggregate?**

**Options:**
A. `ORDER BY`
B. `LIMIT`
C. `GROUP BY`
D. `HAVING`

✅ **Correct:** C
📌 `GROUP BY department` groups rows so `COUNT(*)` (or other aggregates) computes per-department values.

---

## Question 3

**Which clause renames a column in the SELECT result set?**

**Options:**
A. `AS`
B. `RENAME`
C. `CHANGE COLUMN`
D. `ALIAS`

✅ **Correct:** A
📌 Use `SELECT col AS alias` to give a temporary name to a result column.

---

## Question 4

**Which statement is true about the default behavior of `ORDER BY` in PostgreSQL text sorts?**

**Options:**
A. In a character sort, the values are case-sensitive
B. Only SELECT columns can be used in ORDER BY
C. Numeric values are displayed from maximum to minimum by default
D. NULL values are not considered in sorting

✅ **Correct:** A
📌 By default text sorting is case-sensitive (collation can change this). Other options are false/general incorrect.

---

## Question 5

**What happens when you run: `SELECT "10" + 5;` in PostgreSQL?**

**Options:**
A. `105`
B. `10.5`
C. Error: cannot add string and number
D. `15`

✅ **Correct:** C
📌 Double quotes denote an identifier; literal strings use single quotes. Adding a string/identifier to a number causes a type error unless explicitly cast.

---

## Question 6

**Fix the bug: To display employees with no department the query uses `WHERE dept_id = NULL;`. What change is required?**

**Options:**
A. Change operator in WHERE condition
B. Change column
C. Add second condition
D. Create outer join

✅ **Correct:** A
📌 `= NULL` is wrong — use `IS NULL`. (`WHERE dept_id IS NULL`.)

---

## Question 7

**What does `SELECT * FROM employees LIMIT 10 OFFSET 5;` return?**

**Options:**
A. First 5 rows
B. Skips first 5 rows and returns next 10
C. Skips last 5 rows
D. Returns 15 rows

✅ **Correct:** B
📌 `OFFSET` skips rows; `LIMIT` controls how many rows are returned after the skip.

---

## Question 8

**What happens if you run `WHERE department_id IN ();` with an empty list?**

**Options:**
A. Syntax error
B. Returns all rows
C. Returns no rows
D. Returns NULL

✅ **Correct:** A
📌 Empty parentheses with `IN` is invalid syntax and raises an error.

---

## Question 9

**What does `SELECT 'Hello' || ' ' || 'World!';` return?**

**Options:**
A. `Hello World!`
B. Error
C. `Hello || World!`
D. `HelloWorld!`

✅ **Correct:** A
📌 `||` is the string concatenation operator in PostgreSQL, producing `Hello World!`.

---

## Question 10

**Which wildcard matches exactly one character in a `LIKE` pattern?**

**Options:**
A. `?`
B. `%`
C. `*`
D. `_`

✅ **Correct:** D
📌 `_` stands for exactly one character; `%` stands for any sequence (including zero characters).

---

## Question 11

**Retrieve employees whose salary is between 40,000 and 100,000 inclusive. Which is correct?**

**Options:**
A. `WHERE salary >= 40000 AND salary <= 100000`
B. `WHERE salary BETWEEN 40000 AND 100000`
C. `WHERE salary > 40000 AND salary < 100000`
D. Both A and B

✅ **Correct:** D
📌 Both `salary BETWEEN 40000 AND 100000` and the explicit `>=`/`<=` expression produce the same inclusive result.

---

## Question 12

**Find students whose last name is `Kumar` assuming `student_name` contains full name (first + space + last). Which pattern is appropriate?**

**Options:**
A. `WHERE student_name = 'Kumar'`
B. `WHERE student_name LIKE 'Kumar%'`
C. `WHERE student_name LIKE '% Kumar'`
D. `WHERE student_name LIKE '%Kumar%'`

✅ **Correct:** C
📌 `% Kumar` matches any leading text followed by a space and `Kumar` as the last token. (`= 'Kumar'` would only match exact single-name entries.)

---

## Question 13

**`skills` is a text array column. Which query finds rows where `'Python'` is one element of the array?**

**Options:**
A. `WHERE skills @> ARRAY['Python']`
B. `WHERE 'Python' = ANY(skills)`
C. `WHERE skills LIKE '%Python%'`
D. `WHERE 'Python' IN skills`

✅ **Correct:** B (A is also valid but different behavior)
📌 `'= ANY(skills)'` tests membership. Note: `skills @> ARRAY['Python']` also works (contains), but the quiz answer given earlier used `'= ANY(skills)'`.

---

## Question 14

**Admin wants to DELETE a row and immediately get the deleted row's values returned. Which keyword should be used?**

**Options:**
A. `FETCH`
B. `RETRIEVE`
C. `RETURNING`
D. `OUTPUT`

✅ **Correct:** C
📌 PostgreSQL supports `DELETE ... RETURNING *` to return deleted rows.

---

## Question 15

**Return all orders with their payments; include orders that have no payment record. Which JOIN is correct?**

**Options:**
A. `JOIN` (INNER JOIN)
B. `RIGHT JOIN payments ON ...`
C. `LEFT JOIN payments ON ...`
D. `FULL JOIN payments ON ...`

✅ **Correct:** C
📌 `LEFT JOIN` keeps all rows from `orders` and matches `payments` when present; unmatched payments appear as NULL.

---

## Question 16

**Which query returns authors who have written no books?**

**Options:**
A. `SELECT a.name FROM authors a JOIN books b ON a.id = b.author_id WHERE b.id IS NULL;`
B. `SELECT a.name FROM authors a LEFT JOIN books b ON a.id = b.author_id WHERE b.id IS NULL;`
C. `SELECT a.name FROM books b WHERE b.author_id IS NULL;`
D. `SELECT a.name FROM authors a WHERE a.id NOT IN (SELECT author_id FROM books);`

✅ **Correct:** B (D is a valid alternative but B matches the LEFT JOIN pattern asked)
📌 `LEFT JOIN ... WHERE b.id IS NULL` returns authors with no matching book rows. (Note: D also works but can behave differently with NULLs.)

---

## Question 17

**Return orders that have successful payments only. Which approach is correct?**

**Options:**
A. `LEFT JOIN payments` and `WHERE p.success = TRUE`
B. `INNER JOIN payments` and `WHERE p.success = TRUE`
C. `RIGHT JOIN payments` and `WHERE p.success = TRUE`
D. `FULL JOIN payments` and `WHERE p.success = TRUE`

✅ **Correct:** B
📌 `INNER JOIN` already restricts to matching orders/payments; adding `p.success = TRUE` ensures only successful payments are returned. (Using `LEFT JOIN` + WHERE on `p` would exclude unmatched orders.)

---

## Question 18

**Which query returns customers who have no orders?**

**Options:**
A. `SELECT c.* FROM customers c JOIN orders o ON c.id = o.customer_id WHERE o.id IS NULL;`
B. `SELECT c.* FROM customers c LEFT JOIN orders o ON c.id = o.customer_id WHERE o.id IS NULL;`
C. `SELECT * FROM orders WHERE customer_id IS NULL;`
D. `SELECT c.* FROM customers c WHERE c.id NOT IN (SELECT customer_id FROM orders);`

✅ **Correct:** B (D is an alternative but B uses the standard LEFT JOIN / IS NULL pattern)
📌 LEFT JOIN + `WHERE o.id IS NULL` yields customers with zero matching orders.

---

## Question 19

**Show each employee along with their manager's name using a self-join. Which query is correct?**

**Options:**
A. `SELECT e.name, m.name FROM employees e JOIN employees m ON e.id = m.manager_id;`
B. `SELECT e.name, m.name FROM employees e JOIN employees m ON e.manager_id = m.id;`
C. `SELECT e.name, m.name FROM employees e LEFT JOIN departments m ON e.manager_id = m.id;`
D. `SELECT e.name, m.name FROM employees e, employees m WHERE e.id = m.id;`

✅ **Correct:** B
📌 Self-join matching `e.manager_id = m.id` pairs employees with their manager records.

---

## Question 20

**A RIGHT JOIN of `logins` to `users` shows NULLs in user columns for some rows. What do these NULLs indicate?**

**Options:**
A. Query syntax errors
B. Tables have no foreign key defined
C. Logins exist without matching users
D. Usernames are NULL for those users

✅ **Correct:** C
📌 RIGHT JOIN keeps all `logins`; NULLs in `users` columns mean no corresponding `users` row exists for those login records.

---

## Question 21

**You used `LEFT JOIN employees m ON e.manager_id = m.id` then added `WHERE m.name LIKE 'J%'`. What is the effect on employees without managers?**

**Options:**
A. They will be included with NULL manager name
B. They will appear with empty string as manager name
C. They are excluded from the result
D. Query will error

✅ **Correct:** C
📌 The `WHERE` clause filters out rows where `m.name` is NULL, so employees without managers get excluded (left join effectively becomes inner join in this case).

---

## Question 22

**Two tables with 5 rows each are cross joined. How many rows result?**

**Options:**
A. Depends on NULLs
B. 5
C. 10
D. 25

✅ **Correct:** D
📌 CROSS JOIN produces Cartesian product: 5 × 5 = 25 rows.

---

## Question 23

**Table A has 10 rows, Table B has 5 rows, only 3 IDs overlap. How many rows will an INNER JOIN on ID return?**

**Options:**
A. 5
B. 3
C. 15
D. 10

✅ **Correct:** B
📌 INNER JOIN returns only matched rows — 3 overlapping IDs → 3 rows.

---

## Question 24

**What happens when you LEFT JOIN `employees` to `departments` using `ON e.id = d.id` (employee id matched with department id)?**

**Options:**
A. All employees appear with NULL departments if ids don't match
B. Query duplicates department rows
C. Only matched employees appear
D. Query throws an error

✅ **Correct:** A
📌 Employee IDs rarely equal department IDs; LEFT JOIN keeps employees and sets department fields to NULL when no match exists.

---

## Question 25

**Which of the following correctly selects employees in cities Chennai, Hyderabad, or Mumbai?**

**Options:**
A. `SELECT * FROM employees WHERE city = 'Chennai' OR city = 'Hyderabad' OR city = 'Mumbai';`
B. `SELECT * FROM employees WHERE city IN ('Chennai','Hyderabad','Mumbai');`
C. `SELECT * FROM employees WHERE city LIKE 'Chennai|Hyderabad|Mumbai';`
D. Both A and B

✅ **Correct:** D (A and B are equivalent; B is more concise)
📌 `IN` is shorthand for multiple OR conditions; `LIKE` with `|` is not valid for this use.

---

## Question 26

**Which query returns distinct department IDs from `employee` table (no duplicates)?**

**Options:**
A. `SELECT department_id FROM employee;`
B. `SELECT DISTINCT department_id FROM employee;`
C. `SELECT department_id GROUP BY department_id FROM employee;`
D. `SELECT UNIQUE department_id FROM employee;`

✅ **Correct:** B
📌 `SELECT DISTINCT` removes duplicate values; `UNIQUE` is not SQL syntax for SELECT.

---

## Question 27

**Select employees whose name starts with 'jo' regardless of case (e.g., 'John', 'joanna'). Which is correct in PostgreSQL?**

**Options:**
A. `WHERE emp_name LIKE 'jo%'`
B. `WHERE emp_name ILIKE 'jo%'`
C. `WHERE LOWER(emp_name) LIKE 'jo%'`
D. Both B and C

✅ **Correct:** D (B is direct; C works too)
📌 `ILIKE` is case-insensitive; `LOWER(...) LIKE ...` is equivalent.

---

## Question 28

**Which two queries are equivalent for selecting employees with salary between 40k and 100k?**

**Options:**
A. `salary >= 40000 AND salary <= 100000`
B. `salary BETWEEN 40000 AND 100000`
C. `salary > 40000 AND salary < 100000`
D. Both A and B

✅ **Correct:** D
📌 `BETWEEN` is inclusive, same as `>=` and `<=`.

---

## Question 29

**Select students whose name ends with 'Kumar'. Which pattern is appropriate?**

**Options:**
A. `WHERE student_name = 'Kumar'`
B. `WHERE student_name LIKE 'Kumar%'`
C. `WHERE student_name LIKE '%Kumar'`
D. `WHERE student_name LIKE '% Kumar %'`

✅ **Correct:** C
📌 `%Kumar` matches any prefix with `Kumar` at the end. (If you need to ensure a preceding space, use `'% Kumar'`.)

---

## Question 30

**Which query tests whether the array column `skills` contains the element `'Python'`?**

**Options:**
A. `WHERE skills LIKE '%Python%'`
B. `WHERE 'Python' = ANY(skills)`
C. `WHERE skills @> ARRAY['Python']`
D. Both B and C

✅ **Correct:** D
📌 `'= ANY(skills)'` checks element membership; `@>` checks whether array contains the given array (both valid ways to test membership).

---
