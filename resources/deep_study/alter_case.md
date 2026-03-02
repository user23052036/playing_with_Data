# 1. Add & Remove Columns

### Add column — fast & safe approach (recommended for large tables)

Best practice: add column nullable **without** default, then set default + backfill if needed, then set NOT NULL.

```sql
-- 1) add nullable column (fast)
ALTER TABLE employees ADD COLUMN bio TEXT;

-- 2) set default for future inserts (cheap)
ALTER TABLE employees ALTER COLUMN bio SET DEFAULT 'no bio';

-- 3) backfill existing rows in small batches (if needed)
UPDATE employees SET bio = 'no bio' WHERE bio IS NULL LIMIT 10000;

-- 4) after backfilling, make it NOT NULL (requires full check)
ALTER TABLE employees ALTER COLUMN bio SET NOT NULL;
```

### Add column — simple (small table)

```sql
ALTER TABLE employees ADD COLUMN created_at TIMESTAMP DEFAULT now() NOT NULL;
```

**Note:** On very large tables this can cause a table rewrite and long locks on older PostgreSQL versions — prefer the 4-step method.

---

### Remove column

```sql
-- simple
ALTER TABLE employees DROP COLUMN bio;

-- if other objects depend on it
ALTER TABLE employees DROP COLUMN bio CASCADE;  -- drops dependent objects too
```

**Caution:** `CASCADE` can drop indexes, views, constraints that reference the column.

---

# 2. Rename Table / Rename Column

### Rename table

```sql
ALTER TABLE contacts RENAME TO phone_contacts;
```

### Rename column

```sql
ALTER TABLE employees RENAME COLUMN fname TO first_name;
```

### Rename constraint

```sql
ALTER TABLE contacts RENAME CONSTRAINT old_constraint_name TO new_constraint_name;
```

---

# 3. Change datatype of an existing column

Use `USING` when implicit cast is not possible or you need transformation.

### Simple change (safe cast)

```sql
ALTER TABLE employees ALTER COLUMN salary TYPE numeric(10,2);
```

### Change with explicit transformation

Example: `mob` is VARCHAR that contains punctuation, convert to BIGINT (strip non-digits then cast):

```sql
ALTER TABLE contacts
ALTER COLUMN mob TYPE bigint
USING regexp_replace(mob, '\D', '', 'g')::bigint;
```

### Change from integer -> text without rewrite issues

```sql
ALTER TABLE employees ALTER COLUMN emp_id TYPE text USING emp_id::text;
```

### Caveats / checklist

* Indexes and constraints referencing the column may need re-creation.
* If type change cannot be done by cast, you must provide `USING`.
* Large tables: `ALTER TYPE` may require full table rewrite — schedule maintenance.
* If column is part of a primary/foreign key, you may need to drop the FK, alter, then recreate it.

---

# 4. Change column nullability & defaults

### Set column NOT NULL

```sql
ALTER TABLE employees ALTER COLUMN email SET NOT NULL;
```

### Drop NOT NULL

```sql
ALTER TABLE employees ALTER COLUMN email DROP NOT NULL;
```

### Set / Drop default

```sql
ALTER TABLE employees ALTER COLUMN created_at SET DEFAULT now();
ALTER TABLE employees ALTER COLUMN created_at DROP DEFAULT;
```

**Best practice for NOT NULL on large tables:** backfill values in batches then `SET NOT NULL`.

---

# 5. Check constraints — adding, dropping, validating

### Add a named check constraint

```sql
ALTER TABLE contacts
ADD CONSTRAINT mob_no_less_than_10digits CHECK (length(mob) >= 10);
```

**Important:** Do **not** write `mob != null` — use `mob IS NOT NULL`:

```sql
ALTER TABLE contacts
ADD CONSTRAINT mob_not_null CHECK (mob IS NOT NULL);
```

### Add constraint without scanning existing rows (deferred validation)

If table is large and you want to avoid immediate scan:

```sql
ALTER TABLE contacts
ADD CONSTRAINT mob_no_less_than_10digits CHECK (length(mob) >= 10) NOT VALID;
-- later (after you backfill/fix bad rows)
ALTER TABLE contacts VALIDATE CONSTRAINT mob_no_less_than_10digits;
```

`NOT VALID` means it won’t fail now for pre-existing rows; `VALIDATE` performs the check later.

### Drop a constraint

You must know the constraint name:

```sql
ALTER TABLE contacts DROP CONSTRAINT mob_no_less_than_10digits;
```

### Create constraint inline (during CREATE TABLE)

```sql
CREATE TABLE contacts (
  name varchar(50),
  mob varchar(15) UNIQUE,
  CONSTRAINT mob_no_less_than_10digits CHECK (length(mob) >= 10)
);
```

### Rename constraint

```sql
ALTER TABLE contacts RENAME CONSTRAINT mob_no_less_than_10digits TO mob_min_len_10;
```

### Notes on behavior

* `UNIQUE` and `PRIMARY KEY` automatically create indexes. Dropping these constraints may drop indexes.
* `CHECK` constraints do not create indexes.
* Use `EXPLAIN` / test queries if performance and indexing are important.

---

# 6. Examples combining constraints, rename, alter

**Scenario:** table `contacts(name, mob varchar(15))` exists. You want to:

1. Ensure `mob` not null,
2. Ensure length ≥ 10,
3. Prevent rewrite on a huge table.

Commands:

```sql
-- add NOT NULL safely:
ALTER TABLE contacts ALTER COLUMN mob DROP NOT NULL;    -- if present; or skip

-- add check but defer validation
ALTER TABLE contacts
ADD CONSTRAINT mob_min_len CHECK (length(mob) >= 10) NOT VALID;

-- backfill/fix bad rows in batches
UPDATE contacts SET mob = '0000000000' WHERE mob IS NULL LIMIT 10000;

-- validate constraint once satisfied
ALTER TABLE contacts VALIDATE CONSTRAINT mob_min_len;

-- now make column NOT NULL (only after existing rows fixed)
ALTER TABLE contacts ALTER COLUMN mob SET NOT NULL;
```

---

# 7. CASE expression — usage & examples

### Two syntaxes

**Simple CASE (compare expression)**

```sql
CASE dept
  WHEN 'IT' THEN 'Tech'
  WHEN 'HR' THEN 'People'
  ELSE 'Other'
END
```

**Searched CASE (conditions)**

```sql
CASE
  WHEN salary >= 55000 THEN 'high'
  WHEN salary BETWEEN 45000 AND 49999 THEN 'mid'
  ELSE 'low'
END
```

### Examples using `employees` table

**Select with CASE**

```sql
SELECT emp_id,
       fname,
       salary,
       CASE
         WHEN salary >= 55000 THEN 'high'
         WHEN salary BETWEEN 45000 AND 54999 THEN 'mid'
         ELSE 'low'
       END AS sal_category
FROM employees;
```

**CASE in GROUP BY**
You can group by the CASE expression (repeat it or alias in a subquery):

```sql
SELECT sal_category, count(*)
FROM (
  SELECT CASE
           WHEN salary >= 55000 THEN 'high'
           WHEN salary BETWEEN 45000 AND 54999 THEN 'mid'
           ELSE 'low'
         END AS sal_category
  FROM employees
) t
GROUP BY sal_category;
```

**CASE in UPDATE**

```sql
ALTER TABLE employees ADD COLUMN sal_cat text;

UPDATE employees
SET sal_cat = CASE
                WHEN salary >= 55000 THEN 'high'
                WHEN salary BETWEEN 45000 AND 54999 THEN 'mid'
                ELSE 'low'
              END;
```

**CASE with arithmetic**

```sql
SELECT fname, salary,
       CASE WHEN salary > 0 THEN round(salary * 0.10) ELSE 0 END AS bonus
FROM employees;
```

### Tips

* `CASE` returns a single value per row; ensure consistent types across branches or explicitly cast.
* Use `COALESCE` with `CASE` if NULL handling is needed.

---

# 8. Extras & Gotchas (cover-all)

* **ALTER TABLE locking:** Many `ALTER TABLE` commands take an exclusive lock (can block writes/readers). For large tables, plan maintenance windows. `ADD COLUMN` without default is fast (no table rewrite).
* **Serial / sequences:** `SERIAL` is a shorthand that creates a sequence. If you change type of serial column, ensure the sequence ownership and type align (use `ALTER SEQUENCE ... OWNED BY`).
* **Indexes & constraints:** Changing datatype or dropping/renaming columns can invalidate indexes or constraints — check `pg_indexes` and recreate indexes if needed.
* **Backfill in batches:** For large tables, always backfill in batches to avoid transaction bloat and long locks.
* **Validation trade-off:** Use `NOT VALID` for constraints when you need zero-downtime migration; but remember existing invalid data is still present until you `VALIDATE`.
* **NULL vs empty string:** For text columns, `''` is not NULL. Checks and uniqueness constraints may treat them differently — be explicit in logic.
* **Testing:** Always test schema changes on a staging copy (or `pg_dump` -> restore) before production.

---

# 9. Quick Reference — Useful commands

```sql
-- Add column (simple)
ALTER TABLE t ADD COLUMN col type;

-- Add column with default (may rewrite)
ALTER TABLE t ADD COLUMN col type DEFAULT expr;

-- Drop column
ALTER TABLE t DROP COLUMN col;
ALTER TABLE t DROP COLUMN col CASCADE;

-- Rename table/column
ALTER TABLE old_name RENAME TO new_name;
ALTER TABLE t RENAME COLUMN old_col TO new_col;

-- Change type (with USING)
ALTER TABLE t ALTER COLUMN col TYPE new_type USING (expression);

-- Set / drop NOT NULL
ALTER TABLE t ALTER COLUMN col SET NOT NULL;
ALTER TABLE t ALTER COLUMN col DROP NOT NULL;

-- Set / drop DEFAULT
ALTER TABLE t ALTER COLUMN col SET DEFAULT expr;
ALTER TABLE t ALTER COLUMN col DROP DEFAULT;

-- Add / drop named CHECK constraint
ALTER TABLE t ADD CONSTRAINT chk_name CHECK (your_expr);
ALTER TABLE t DROP CONSTRAINT chk_name;

-- Add constraint without validating existing rows
ALTER TABLE t ADD CONSTRAINT chk_name CHECK (expr) NOT VALID;
ALTER TABLE t VALIDATE CONSTRAINT chk_name;

-- Rename constraint
ALTER TABLE t RENAME CONSTRAINT old TO new;
```

---
