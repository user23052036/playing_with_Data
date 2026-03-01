
# 📘 PostgreSQL String Functions – Complete Notes

---

# PART 1 — Function Definitions with Simple Examples

---

## 1️⃣ CONCAT & CONCAT_WS

### ✅ CONCAT()

Joins multiple strings.

```sql
SELECT CONCAT('Souvik', ' ', 'Mandal');
```

Output:

```
Souvik Mandal
```

✔ Treats NULL as empty string.

---

### ✅ CONCAT_WS()  (With Separator)

Adds separator between values.

```sql
SELECT CONCAT_WS('-', '2026', '03', '01');
```

Output:

```
2026-03-01
```

✔ Skips NULL values automatically.

---

## 2️⃣ SUBSTR (SUBSTRING)

Extracts part of a string.

```sql
SELECT SUBSTR('PostgreSQL', 1, 4);
```

Output:

```
Post
```

Index starts from **1**.

---

## 3️⃣ LEFT & RIGHT

```sql
SELECT LEFT('PostgreSQL', 4);
```

Output:

```
Post
```

```sql
SELECT RIGHT('PostgreSQL', 3);
```

Output:

```
SQL
```

---

## 4️⃣ LENGTH

```sql
SELECT LENGTH('Souvik');
```

Output:

```
6
```

Returns number of characters.

---

## 5️⃣ UPPER & LOWER

```sql
SELECT UPPER('hello');
```

Output:

```
HELLO
```

```sql
SELECT LOWER('HELLO');
```

Output:

```
hello
```

---

## 6️⃣ TRIM, LTRIM, RTRIM

```sql
SELECT TRIM('   hello   ');
```

Output:

```
hello
```

```sql
SELECT LTRIM('   hello');
```

```sql
SELECT RTRIM('hello   ');
```

Remove specific character:

```sql
SELECT TRIM('x' FROM 'xxxhelloxxx');
```

Output:

```
hello
```

---

## 7️⃣ REPLACE

```sql
SELECT REPLACE('I like Java', 'Java', 'PostgreSQL');
```

Output:

```
I like PostgreSQL
```

---

## 8️⃣ POSITION

```sql
SELECT POSITION('gre' IN 'PostgreSQL');
```

Output:

```
5
```

---

## 9️⃣ STRING_AGG

Used to combine multiple rows.

```sql
SELECT STRING_AGG(name, ', ')
FROM student;
```

---

# PART 2 — Applying All Functions on `employees` Table

### Table Structure:

```sql
employees(emp_id, fname, lname, email, dept, salary, hire_date)
```

---

## 1️⃣ Full Name Creation

```sql
SELECT CONCAT_WS(' ', fname, lname) AS full_name
FROM employees;
```

---

## 2️⃣ Extract First 3 Letters of First Name

```sql
SELECT fname,
       SUBSTR(fname, 1, 3) AS short_name
FROM employees;
```

---

## 3️⃣ Department Code (First 2 Letters)

```sql
SELECT dept,
       LEFT(dept, 2) AS dept_code
FROM employees;
```

---

## 4️⃣ Email Domain Extension (Last 4 Characters)

```sql
SELECT email,
       RIGHT(email, 4) AS domain_ext
FROM employees;
```

---

## 5️⃣ Email Length

```sql
SELECT email,
       LENGTH(email) AS email_length
FROM employees;
```

---

## 6️⃣ Convert Department to Uppercase

```sql
SELECT UPPER(dept) AS dept_upper
FROM employees;
```

---

## 7️⃣ Clean Extra Spaces in First Name

```sql
SELECT TRIM(fname) AS clean_name
FROM employees;
```

---

## 8️⃣ Replace Gmail with Company Domain

```sql
SELECT email,
       REPLACE(email, 'gmail.com', 'company.com') AS new_email
FROM employees;
```

---

## 9️⃣ Extract Username from Email

```sql
SELECT email,
       SUBSTR(email, 1, POSITION('@' IN email)-1) AS username
FROM employees;
```

---

## 🔟 Combine Employee Names Department-Wise

```sql
SELECT dept,
       STRING_AGG(CONCAT_WS(' ', fname, lname), ', ') AS employees
FROM employees
GROUP BY dept;
```

---

## 📊 Department Report (Realistic Example)

```sql
SELECT dept,
       COUNT(*) AS total_employees,
       STRING_AGG(fname, ', ') AS names,
       AVG(salary) AS avg_salary
FROM employees
GROUP BY dept;
```

---

# 🔥 Final Summary

| Function      | Purpose             |
| ------------- | ------------------- |
| CONCAT        | Join strings        |
| CONCAT_WS     | Join with separator |
| SUBSTR        | Extract substring   |
| LEFT / RIGHT  | Extract from ends   |
| LENGTH        | Count characters    |
| UPPER / LOWER | Change case         |
| TRIM          | Remove spaces       |
| REPLACE       | Replace text        |
| POSITION      | Find index          |
| STRING_AGG    | Combine rows        |

---
