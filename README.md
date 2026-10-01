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
Continue this directly after your previous Markdown notes:

````markdown
# 10. SQL JOINs — Interview Notes

## 10.1 Why do we use JOIN?

Data is often stored in multiple related tables.

Example:

### Employee

| id | name | department_id | salary |
|---:|---|---:|---:|
| 1 | Rahul | 10 | 60000 |
| 2 | Priya | 20 | 50000 |
| 3 | Aman | 10 | 80000 |
| 4 | Neha | 30 | 70000 |
| 5 | Ravi | 10 | 60000 |

### Department

| department_id | department_name |
|---:|---|
| 10 | IT |
| 20 | HR |
| 30 | Sales |
| 40 | Finance |

Employee stores `department_id`, while Department stores `department_name`.

A JOIN allows us to retrieve related data from both tables.

---

# 10.2 INNER JOIN

Returns only rows where a match exists in both tables.

```sql
SELECT e.name, d.department_name
FROM Employee e
JOIN Department d
ON e.department_id = d.department_id;
````

`JOIN` by itself means `INNER JOIN`.

### Result

```text
Rahul  → IT
Priya  → HR
Aman   → IT
Neha   → Sales
Ravi   → IT
```

Finance is not returned because no employee belongs to department `40`.

### Interview definition

> INNER JOIN returns only the matching rows from both tables.

---

# 10.3 LEFT JOIN

Returns **all rows from the left table**, plus matching rows from the right table.

```sql
SELECT d.department_name, e.name
FROM Department d
LEFT JOIN Employee e
ON d.department_id = e.department_id;
```

Finance also appears:

```text
IT       → Rahul
IT       → Aman
IT       → Ravi
HR       → Priya
Sales    → Neha
Finance  → NULL
```

### Key rule

```text
LEFT JOIN
→ Keep everything from LEFT table
→ Add matching data from RIGHT table
→ No match on right → NULL
```

---

# 10.4 RIGHT JOIN

Returns all rows from the right table plus matching rows from the left.

```sql
SELECT d.department_name, e.name
FROM Department d
RIGHT JOIN Employee e
ON d.department_id = e.department_id;
```

Less commonly used in interviews than `LEFT JOIN`.

---

# 10.5 FULL OUTER JOIN

Returns:

```text
Matching rows
+ Unmatched rows from left
+ Unmatched rows from right
```

MySQL does not directly support `FULL OUTER JOIN`.

A common implementation is:

```sql
SELECT e.name, d.department_name
FROM Employee e
LEFT JOIN Department d
ON e.department_id = d.department_id

UNION

SELECT e.name, d.department_name
FROM Employee e
RIGHT JOIN Department d
ON e.department_id = d.department_id;
```

### Remember

```text
FULL OUTER JOIN in MySQL
=
LEFT JOIN
UNION
RIGHT JOIN
```

`UNION` removes duplicate rows.

---

# 10.6 JOIN does NOT change the original tables

### Important confusion

Suppose:

```sql
Employee e
Department d
```

Then:

```text
e → columns of Employee
d → columns of Department
```

So:

```sql
e.name
```

is valid.

```sql
d.department_name
```

is valid.

But:

```sql
e.department_name
```

is invalid because `department_name` belongs to `Department`, not `Employee`.

### Important concept

> JOIN combines related rows for the query result. It does not permanently add the columns of one table to another table.

Think:

```text
Employee e
    +
Department d
    ↓
Temporary query result containing columns from both
```

The alias still tells SQL which original table the column belongs to.

---

# 10.7 JOIN + WHERE

After joining, we can filter using columns from either table.

### Example

Find IT employees earning more than 60000:

```sql
SELECT e.name, e.salary
FROM Employee e
JOIN Department d
ON e.department_id = d.department_id
WHERE d.department_name = 'IT'
AND e.salary > 60000;
```

Here:

```text
e.salary            → Employee table
d.department_name   → Department table
```

---

# 10.8 JOIN + GROUP BY

Find average salary of every department:

```sql
SELECT d.department_name, AVG(e.salary) AS avg_salary
FROM Department d
JOIN Employee e
ON d.department_id = e.department_id
GROUP BY d.department_name;
```

The JOIN first connects employees with departments.

Then `GROUP BY` creates department-wise groups.

Then `AVG()` calculates the average for each group.

---

# 10.9 JOIN + GROUP BY + HAVING

Find departments whose average salary is greater than 60000:

```sql
SELECT d.department_name, AVG(e.salary) AS avg_salary
FROM Department d
JOIN Employee e
ON d.department_id = e.department_id
GROUP BY d.department_name
HAVING AVG(e.salary) > 60000;
```

### Process

```text
JOIN
 ↓
Employee + Department information
 ↓
GROUP BY department
 ↓
Calculate AVG(salary)
 ↓
HAVING filters departments
```

---

# 10.10 IMPORTANT: COUNT(*) vs COUNT(column) with LEFT JOIN

This was an important confusion.

Suppose Finance has no employees.

After:

```sql
FROM Department d
LEFT JOIN Employee e
ON d.department_id = e.department_id
```

the intermediate result contains:

```text
Finance | NULL
```

There is still a row because LEFT JOIN must keep Finance.

---

## COUNT(*)

```sql
COUNT(*)
```

means:

> Count rows.

Therefore:

```text
Finance | NULL
```

is still one row.

So:

```sql
COUNT(*)
```

returns:

```text
1
```

even though Finance has zero employees.

---

## COUNT(e.id)

```sql
COUNT(e.id)
```

means:

> Count non-NULL values of `e.id`.

For Finance:

```text
Finance | NULL
```

`e.id` is NULL, so it is not counted.

Therefore:

```text
Finance → 0
```

### Correct query for number of employees per department:

```sql
SELECT d.department_name, COUNT(e.id) AS employee_count
FROM Department d
LEFT JOIN Employee e
ON d.department_id = e.department_id
GROUP BY d.department_name;
```

### Result

```text
IT       → 3
HR       → 1
Sales    → 1
Finance  → 0
```

---

# 10.11 Why does COUNT(*) become 1?

This is an important interview concept.

After LEFT JOIN:

```text
Finance | NULL
```

There is **one row**.

Therefore:

```sql
COUNT(*)
```

counts:

```text
1 row → 1
```

It does NOT ask:

> How many employees exist?

It asks:

> How many rows exist?

While:

```sql
COUNT(e.id)
```

asks:

> How many non-NULL employee IDs exist?

For Finance:

```text
NULL → ignored → 0
```

### Remember

```text
COUNT(*)         → counts rows
COUNT(column)    → counts non-NULL values
```

This becomes especially important with `LEFT JOIN`.

---

# 10.12 Common JOIN Mistakes

## Mistake 1: Forgetting the JOIN condition

Wrong:

```sql
SELECT e.name, d.department_name
FROM Employee e
JOIN Department d;
```

A JOIN normally needs a condition:

```sql
ON e.department_id = d.department_id
```

Without the appropriate condition, you may create a Cartesian product.

---

## Mistake 2: Selecting the wrong table's column

Wrong:

```sql
SELECT e.department_name
```

Correct:

```sql
SELECT d.department_name
```

because `department_name` belongs to `Department`.

---

## Mistake 3: Using INNER JOIN when zero-match rows must be preserved

Suppose the question says:

> Show all departments, including departments with no employees.

Using:

```sql
JOIN
```

will remove Finance because there is no matching employee.

Correct:

```sql
LEFT JOIN
```

---

## Mistake 4: Using COUNT(*)

For:

> Count employees in every department, including departments with zero employees.

Avoid:

```sql
COUNT(*)
```

Use:

```sql
COUNT(e.id)
```

because unmatched employees produce NULL.

---

## Mistake 5: Forgetting the grouping column

Wrong:

```sql
SELECT AVG(e.salary)
FROM Department d
JOIN Employee e
ON d.department_id = e.department_id
GROUP BY d.department_name;
```

This technically calculates the averages, but the output does not tell us which average belongs to which department.

Better:

```sql
SELECT d.department_name, AVG(e.salary)
FROM Department d
JOIN Employee e
ON d.department_id = e.department_id
GROUP BY d.department_name;
```

---

# 10.13 JOIN Types — Quick Interview Table

| JOIN            | Meaning                             |
| --------------- | ----------------------------------- |
| INNER JOIN      | Matching rows only                  |
| LEFT JOIN       | All left rows + matching right rows |
| RIGHT JOIN      | All right rows + matching left rows |
| FULL OUTER JOIN | All rows from both sides            |

### Easy memory trick

```text
INNER → Match only

LEFT → Keep left

RIGHT → Keep right

FULL → Keep both
```

---

# 10.14 Most Important JOIN Patterns

### 1. Get related data

```sql
SELECT e.name, d.department_name
FROM Employee e
JOIN Department d
ON e.department_id = d.department_id;
```

### 2. Filter after JOIN

```sql
SELECT e.name
FROM Employee e
JOIN Department d
ON e.department_id = d.department_id
WHERE d.department_name = 'IT';
```

### 3. Aggregate after JOIN

```sql
SELECT d.department_name, AVG(e.salary)
FROM Department d
JOIN Employee e
ON d.department_id = e.department_id
GROUP BY d.department_name;
```

### 4. Filter groups after JOIN

```sql
SELECT d.department_name, AVG(e.salary)
FROM Department d
JOIN Employee e
ON d.department_id = e.department_id
GROUP BY d.department_name
HAVING AVG(e.salary) > 60000;
```

### 5. Count related records including zero

```sql
SELECT d.department_name, COUNT(e.id)
FROM Department d
LEFT JOIN Employee e
ON d.department_id = e.department_id
GROUP BY d.department_name;
```

---

# 10.15 SQL Interview Mental Model

When reading a query, think in this order:

```text
FROM / JOIN
    ↓
Build the required rows
    ↓
WHERE
    ↓
Filter individual rows
    ↓
GROUP BY
    ↓
Create groups
    ↓
HAVING
    ↓
Filter groups
    ↓
SELECT
    ↓
Choose output columns
    ↓
ORDER BY
    ↓
Sort
    ↓
LIMIT
    ↓
Restrict number of rows
```

### Very important distinction

```text
WHERE
→ individual rows

HAVING
→ groups

COUNT(*)
→ rows

COUNT(column)
→ non-NULL column values

JOIN
→ combines related rows for the query result
```

---

# 10.16 Interview Questions to Practice Next

Using `Employee` and `Department`:

### Q1

Find the names of employees who work in the IT department and earn more than 60000.

### Q2

Find the average salary of each department.

### Q3

Find departments whose average salary is greater than 60000.

### Q4

Find the number of employees in each department, including departments with zero employees.

### Q5

Find departments having at least 2 employees.

### Q6

Find the department with the highest average salary.

### Q7

Find employees whose department does not exist in the Department table.

---

# Key Interview Takeaways

> JOIN combines related rows; it does not modify the original table structure.

> Use table aliases to clearly identify where each column comes from.

> `INNER JOIN` removes unmatched rows.

> `LEFT JOIN` preserves every row from the left table.

> In a LEFT JOIN, unmatched right-side columns become `NULL`.

> `COUNT(*)` counts rows, including a NULL-filled LEFT JOIN row.

> `COUNT(right_table.id)` ignores NULL and can correctly return `0` for no matches.

> `WHERE` filters rows before grouping.

> `GROUP BY` creates groups.

> `HAVING` filters groups after aggregation.

> For “including zero related records”, think `LEFT JOIN + COUNT(right_table.id)`.

> MySQL has no direct `FULL OUTER JOIN`; commonly use `LEFT JOIN UNION RIGHT JOIN`.

```
```
