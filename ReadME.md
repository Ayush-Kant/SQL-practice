````markdown
# SQL — Aggregate Functions, GROUP BY & HAVING

## 1. Mistakes I Made

### Mistake 1: Not selecting the grouping column

I wrote:

```sql
SELECT SUM(salary)
FROM Employee
GROUP BY department;
````

The calculation is correct, but the result does not tell me **which department** each sum belongs to.

### Correct:

```sql
SELECT department, SUM(salary)
FROM Employee
GROUP BY department;
```

### Rule:

Whenever I use `GROUP BY`, I should usually include the grouped column in `SELECT`.

---

## Mistake 2: Using SELECT aliases inside HAVING

I wrote:

```sql
SELECT department, COUNT(*) AS cnt
FROM Employee
GROUP BY department
HAVING cnt > 1;
```

This may work in some databases, but it is **not portable SQL**.

### Interview-safe version:

```sql
SELECT department, COUNT(*) AS cnt
FROM Employee
GROUP BY department
HAVING COUNT(*) > 1;
```

Same idea:

```sql
HAVING AVG(salary) > 60000
```

```sql
HAVING SUM(salary) > 100000
```

### Rule:

For interviews, prefer the **aggregate expression directly inside HAVING**.

---

## Mistake 3: Wrong ORDER BY direction

I wrote:

```sql
ORDER BY avg_salary
LIMIT 1;
```

By default, `ORDER BY` uses ascending order (`ASC`).

So this finds the **lowest** average salary.

### To find the highest:

```sql
ORDER BY avg_salary DESC
LIMIT 1;
```

### Remember:

```text
ASC  → Small → Large
DESC → Large → Small
```

---

## Mistake 4: Inconsistent table name

I sometimes wrote:

```sql
FROM Employees
```

instead of:

```sql
FROM Employee
```

SQL depends on the actual table name.

### Rule:

Be consistent with table and column names.

---

# 2. Aggregate Functions

Aggregate functions perform calculations on multiple rows.

| Function   | Meaning        |
| ---------- | -------------- |
| `COUNT(*)` | Number of rows |
| `SUM()`    | Total          |
| `AVG()`    | Average        |
| `MAX()`    | Maximum value  |
| `MIN()`    | Minimum value  |

### Examples

```sql
SELECT COUNT(*)
FROM Employee;
```

```sql
SELECT SUM(salary)
FROM Employee;
```

```sql
SELECT AVG(salary)
FROM Employee;
```

```sql
SELECT MAX(salary), MIN(salary)
FROM Employee;
```

---

# 3. GROUP BY

`GROUP BY` combines rows having the same value into groups.

### Example:

```sql
SELECT department, COUNT(*)
FROM Employee
GROUP BY department;
```

Meaning:

> Group employees by department and count how many employees are in each department.

Example result:

```text
IT       3
HR       1
Sales    1
```

### Common pattern:

```sql
SELECT grouping_column, AGGREGATE_FUNCTION(column)
FROM table
GROUP BY grouping_column;
```

Example:

```sql
SELECT department, AVG(salary)
FROM Employee
GROUP BY department;
```

This gives the **average salary for each department**.

---

# 4. HAVING

`HAVING` is used to filter **groups** after `GROUP BY`.

### Example:

```sql
SELECT department, COUNT(*)
FROM Employee
GROUP BY department
HAVING COUNT(*) > 1;
```

Meaning:

> Show only departments containing more than 1 employee.

---

# 5. WHERE vs HAVING

This is a very common interview question.

### WHERE

Filters **individual rows** before grouping.

```sql
SELECT *
FROM Employee
WHERE salary > 60000;
```

### HAVING

Filters **groups** after grouping.

```sql
SELECT department, AVG(salary)
FROM Employee
GROUP BY department
HAVING AVG(salary) > 60000;
```

### Easy way to remember:

```text
WHERE  → filters ROWS
HAVING → filters GROUPS
```

---

# 6. Important Query Pattern

### Departments whose average salary is greater than 60000:

```sql
SELECT department, AVG(salary)
FROM Employee
GROUP BY department
HAVING AVG(salary) > 60000;
```

### Departments whose total salary is greater than 100000:

```sql
SELECT department, SUM(salary)
FROM Employee
GROUP BY department
HAVING SUM(salary) > 100000;
```

### Department with highest average salary:

```sql
SELECT department, AVG(salary) AS avg_salary
FROM Employee
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 1;
```

---

# 7. SQL Logical Execution Order

Remember this order:

```text
FROM
  ↓
WHERE
  ↓
GROUP BY
  ↓
HAVING
  ↓
SELECT
  ↓
ORDER BY
  ↓
LIMIT
```

### Important idea:

`WHERE` happens before grouping, while `HAVING` happens after grouping.

---

# 8. Interview Checklist

When I see a SQL question, ask:

1. **Do I need filtering?** → `WHERE`
2. **Do I need calculation?** → `COUNT / SUM / AVG / MAX / MIN`
3. **Do I need calculation for each category/department?** → `GROUP BY`
4. **Do I need to filter those groups?** → `HAVING`
5. **Do I need highest/lowest group?** → `ORDER BY ... DESC/ASC + LIMIT`

---

# 9. Core Patterns to Memorize

```sql
-- Count per department
SELECT department, COUNT(*)
FROM Employee
GROUP BY department;
```

```sql
-- Average per department
SELECT department, AVG(salary)
FROM Employee
GROUP BY department;
```

```sql
-- Filter groups
SELECT department, COUNT(*)
FROM Employee
GROUP BY department
HAVING COUNT(*) > 1;
```

```sql
-- Highest group value
SELECT department, AVG(salary) AS avg_salary
FROM Employee
GROUP BY department
ORDER BY avg_salary DESC
LIMIT 1;
```

## Key Takeaways

> `WHERE` = filter rows
> `GROUP BY` = create groups
> `HAVING` = filter groups
> `COUNT/SUM/AVG/MAX/MIN` = aggregate calculations
> `DESC + LIMIT 1` = find the highest
> `ASC + LIMIT 1` = find the lowest

```
```
