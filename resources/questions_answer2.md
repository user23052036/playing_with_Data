
### **1. Which join is based on all columns in two tables that have the same data type?**

**Correct Answer:** Natural Join

**Explanation:**
A **NATURAL JOIN** automatically joins two tables based on all columns that have the same name and compatible data types in both tables. You do not explicitly specify the join condition.

---

### **2. Which join produces the cross product of two tables?**

**Correct Answer:** Cross Join

**Explanation:**
A **CROSS JOIN** returns the Cartesian product of two tables.
If table A has `m` rows and table B has `n` rows, the result will have `m × n` rows.

---

### **3. Which join retrieves records that do not meet the join condition?**

**Correct Answer:** Outer Join

**Explanation:**
An **OUTER JOIN** (LEFT, RIGHT, or FULL) returns unmatched rows along with matched ones.
It includes rows that do not satisfy the join condition by filling missing values with NULL.

---

### **4. Joining a table to itself is called as ______.**

**Correct Answer:** Self Join

**Explanation:**
A **SELF JOIN** is when a table is joined with itself.
It is typically used to compare rows within the same table (e.g., employee–manager relationships).

---

### **5. Equijoin is also called as ______.**

**Correct Answer:** Equal Join

**Explanation:**
An **Equijoin** is a join where the join condition uses the equality operator (`=`).
It is also referred to as an **Equal Join**.

---

## Window Function Questions

---

### **6. Ravi wants to find the previous month's salary of each employee to compare it with the current month's salary in the same result set. Which window function is most appropriate?**

**Correct Answer:** LAG()

**Explanation:**
`LAG()` accesses data from a previous row in the result set without using a self-join.
It is ideal for comparing current values with previous values (e.g., month-over-month salary comparison).

---

### **7. Which statement is true about indexes?**

**Correct Answer:** They improve SELECT but slow down INSERT/UPDATE

**Explanation:**
Indexes speed up data retrieval (SELECT queries).
However, during INSERT, UPDATE, or DELETE, the index must also be updated, which slows down write operations.

---

### **8. Neha is optimizing full-text search on a document column. Which index type should she use?**

**Correct Answer:** GIN

**Explanation:**
**GIN (Generalized Inverted Index)** is optimized for full-text search.
It is efficient for searching words inside documents (like text search).

---

### **9. Priya wants to assign the same rank to employees with equal salaries, but without skipping rank numbers. Which function should she use?**

**Correct Answer:** DENSE_RANK()

**Explanation:**

* `RANK()` skips numbers when ties occur.
* `DENSE_RANK()` does not skip numbers.
  Example:
  Salaries: 1000, 1000, 900
  DENSE_RANK → 1, 1, 2
  RANK → 1, 1, 3

---

### **10. Arun frequently searches the ORDERS table using the order_date column with range conditions. Which index type is most suitable?**

**Correct Answer:** B-tree index

**Explanation:**
**B-tree indexes** are ideal for range queries (`<`, `>`, `BETWEEN`).
They maintain sorted order, making them efficient for date-based range searches.

---