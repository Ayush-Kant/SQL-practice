# SQL Interview Case Study — Highest Salary in Each Department

## Problem Statement

We have two tables.

### Employee

| id | name  | department_id | salary |
| -: | ----- | ------------: | -----: |
|  1 | Rahul |            10 |  60000 |
|  2 | Priya |            20 |  50000 |
|  3 | Aman  |            10 |  80000 |
|  4 | Neha  |            30 |  70000 |
|  5 | Ravi  |            10 |  60000 |

### Department

| department_id | department_name |
| ------------: | --------------- |
|            10 | IT              |
|            20 | HR              |
|            30 | Sales           |
|            40 | Finance         |

### Interview Question

> **Find the employee with the highest salary in each department.**

Expected result:

| Employee | Department | Salary |
| -------- | ---------- | -----: |
| Aman     | IT         |  80000 |
| Priya    | HR         |  50000 |
| Neha     | Sales      |  70000 |

---

# 1. First Understand What `MAX()` Actually Gives

Consider:

```sql
SELECT department_id, MAX(salary) AS highest_salary
FROM Employee
GROUP BY department_id;
```

This gives:

| department_id | highest_salary |
| ------------: | -------------: |
|            10 |          80000 |
|            20 |          50000 |
|            30 |          70000 |

This query correctly answers:

> **What is the highest salary in each department?**

But it does **not** answer:

> **Who is earning that salary?**

That distinction is the heart of this problem.

---

# 2. My First Attempt — One Table

I initially tried:

```sql
SELECT name, MAX(salary) AS highest_salary
FROM Employee
GROUP BY department;
```

## Why this is wrong

Suppose the IT department contains:

| name  | salary |
| ----- | -----: |
| Rahul |  60000 |
| Aman  |  80000 |
| Ravi  |  60000 |

`MAX(salary)` is:

```text
80000
```

But SQL also has to decide which `name` to return.

Possible names:

```text
Rahul
Aman
Ravi
```

There is no instruction saying:

> "Return the name belonging to the employee whose salary is MAX(salary)."

Therefore, simply writing:

```sql
SELECT name, MAX(salary)
GROUP BY department;
```

does not correctly establish the relationship between the aggregate value and the row.

---

# 3. Important Concept — Aggregate vs Row

## Aggregate function

```sql
MAX(salary)
```

answers:

> What is the maximum salary value?

It does **not automatically give the complete row** that produced that value.

### Example

```text
MAX(salary) = 80000
```

doesn't automatically mean:

```text
name = Aman
```

We need an additional technique to connect:

```text
maximum salary
      ↓
employee row
```

---

# 4. What If We Have Two Tables?

Now suppose we use:

```sql
SELECT e.name, d.department_name, e.salary
FROM Employee e
JOIN Department d
ON e.department_id = d.department_id;
```

Now the result contains information from both tables:

| name  | department_name | salary |
| ----- | --------------- | -----: |
| Rahul | IT              |  60000 |
| Aman  | IT              |  80000 |
| Ravi  | IT              |  60000 |
| Priya | HR              |  50000 |
| Neha  | Sales           |  70000 |

The JOIN is useful, but it still doesn't solve the main problem.

We still need to find:

> Which employee has the maximum salary within each department?

---

# 5. My Second Attempt — Two Tables

I tried:

```sql
SELECT e.name, MAX(d.salary) AS highest_salary
FROM Employee e
LEFT JOIN Department d
ON e.employee_id = d.employeee_id
GROUP BY d.department;
```

There are multiple problems here.

---

## Mistake 1 — `salary` belongs to Employee

Our tables are:

```text
Employee
--------
id
name
department_id
salary
```

```text
Department
----------
department_id
department_name
```

Therefore:

```sql
e.salary
```

is correct.

But:

```sql
d.salary
```

is wrong because `Department` has no `salary` column.

---

## Mistake 2 — Incorrect JOIN condition

I used:

```sql
ON e.employee_id = d.employeee_id
```

But our tables are related through:

```sql
ON e.department_id = d.department_id
```

### Relationship

```text
Employee.department_id
          =
Department.department_id
```

---

## Mistake 3 — Wrong department column

I used:

```sql
d.department
```

But the actual column is:

```sql
d.department_name
```

and the key is:

```sql
d.department_id
```

---

# 6. Even After Fixing Those Mistakes...

Suppose I write:

```sql
SELECT e.name, MAX(e.salary) AS highest_salary
FROM Employee e
JOIN Department d
ON e.department_id = d.department_id
GROUP BY d.department_id;
```

This JOIN is now correct.

The `MAX()` is also correct.

But the query still has a fundamental problem:

```sql
SELECT e.name, MAX(e.salary)
```

How does SQL know that `e.name` must belong to the employee whose salary equals `MAX(e.salary)`?

It doesn't.

So the problem is **not the JOIN anymore**.

The problem is:

> **How do I retrieve the complete employee row corresponding to the maximum salary?**

---

# 7. Break the Problem Into Two Steps

This is the easiest way to understand Q9.

## Step 1 — Find maximum salary in each department

```sql
SELECT department_id, MAX(salary) AS highest_salary
FROM Employee
GROUP BY department_id;
```

Result:

| department_id | highest_salary |
| ------------: | -------------: |
|            10 |          80000 |
|            20 |          50000 |
|            30 |          70000 |

Now we know the maximum values.

But we still don't know the employee names.

---

## Step 2 — Find the employee having that salary

For IT:

```text
Department ID = 10
Highest salary = 80000
```

Look at employees in department 10:

| name  | department_id | salary |
| ----- | ------------: | -----: |
| Rahul |            10 |  60000 |
| Aman  |            10 |  80000 |
| Ravi  |            10 |  60000 |

The employee whose salary equals `80000` is:

```text
Aman
```

So:

```text
IT → Aman → 80000
```

We need SQL to perform both steps together.

---

# 8. When Do We Need a Subquery?

A **subquery** is a query inside another query.

The important signal is:

> **I need to calculate something first and then use that calculated result to filter/find rows.**

### Simple example

Find employees whose salary is above the overall average salary:

```sql
SELECT name, salary
FROM Employee
WHERE salary > (
    SELECT AVG(salary)
    FROM Employee
);
```

The inner query:

```sql
SELECT AVG(salary)
FROM Employee;
```

calculates the average.

The outer query:

```sql
SELECT name, salary
FROM Employee
WHERE salary > (...);
```

uses that value.

---

# 9. Subquery Solution for Q9

```sql
SELECT e.name, e.department_id, e.salary
FROM Employee e
WHERE e.salary = (
    SELECT MAX(e2.salary)
    FROM Employee e2
    WHERE e2.department_id = e.department_id
);
```

This solves the problem.

---

# 10. How the Subquery Works

Take one employee from the outer query.

Suppose the outer employee is:

```text
Aman
department_id = 10
salary = 80000
```

The inner query becomes conceptually:

```sql
SELECT MAX(e2.salary)
FROM Employee e2
WHERE e2.department_id = 10;
```

It returns:

```text
80000
```

The outer query checks:

```text
Aman's salary = 80000
Maximum salary in department 10 = 80000
```

So Aman is returned.

---

### Now consider Rahul

Rahul:

```text
salary = 60000
department_id = 10
```

Inner query:

```text
maximum salary in department 10 = 80000
```

Comparison:

```text
60000 = 80000
```

False.

Rahul is not returned.

---

### Now consider Priya

Priya:

```text
department_id = 20
salary = 50000
```

Inner query:

```text
maximum salary in department 20 = 50000
```

Comparison:

```text
50000 = 50000
```

True.

Priya is returned.

---

# 11. Why Is It Called a Correlated Subquery?

Look at:

```sql
WHERE e2.department_id = e.department_id
```

The inner query uses:

```sql
e.department_id
```

from the **outer query**.

Therefore, the inner query depends on the current outer row.

This is called a:

> **Correlated subquery**

### Mental model

```text
Take an employee
       ↓
Find that employee's department
       ↓
Find MAX salary in that department
       ↓
Compare employee salary with MAX
       ↓
If equal → return employee
```

---

# 12. Important Edge Case — Tied Salaries

Suppose IT has:

| name  | salary |
| ----- | -----: |
| Aman  |  80000 |
| Raj   |  80000 |
| Rahul |  60000 |

Maximum salary is:

```text
80000
```

Both Aman and Raj satisfy:

```text
salary = MAX(salary)
```

So the query returns both:

```text
Aman → 80000
Raj  → 80000
```

This is normally the correct behavior when the question says:

> Employees with the highest salary.

---

# 13. Why HAVING Is Not Enough

`HAVING` is used to filter groups.

Example:

```sql
SELECT department_id, MAX(salary)
FROM Employee
GROUP BY department_id
HAVING MAX(salary) > 70000;
```

This answers:

> Which departments have a maximum salary greater than 70000?

But it doesn't directly answer:

> Who is the employee with that maximum salary?

So:

```text
HAVING
→ filters groups

Subquery
→ can help compare/find individual rows using a calculated value
```

---

# 14. When Should I Think "SUBQUERY"?

These interview questions are strong signals.

### 1. Employee earning above overall average

```sql
SELECT name, salary
FROM Employee
WHERE salary > (
    SELECT AVG(salary)
    FROM Employee
);
```

### 2. Employee with highest salary

```sql
SELECT name, salary
FROM Employee
WHERE salary = (
    SELECT MAX(salary)
    FROM Employee
);
```

### 3. Employee with highest salary in each department

```sql
SELECT e.name, e.department_id, e.salary
FROM Employee e
WHERE e.salary = (
    SELECT MAX(e2.salary)
    FROM Employee e2
    WHERE e2.department_id = e.department_id
);
```

The third one is a **correlated subquery** because the inner query depends on the outer row.

---

# 15. What Each Query Actually Answers

## Query A

```sql
SELECT department_id, MAX(salary)
FROM Employee
GROUP BY department_id;
```

### Answers:

> **What is the maximum salary in each department?**

---

## Query B

```sql
SELECT name, salary
FROM Employee
WHERE salary = (
    SELECT MAX(salary)
    FROM Employee
);
```

### Answers:

> **Who has the maximum salary overall?**

---

## Query C

```sql
SELECT e.name, e.department_id, e.salary
FROM Employee e
WHERE e.salary = (
    SELECT MAX(e2.salary)
    FROM Employee e2
    WHERE e2.department_id = e.department_id
);
```

### Answers:

> **Who has the maximum salary within their department?**

---

# 16. Common Mistakes From Q9

| Mistake                                                    | Why                                                                                 |
| ---------------------------------------------------------- | ----------------------------------------------------------------------------------- |
| `SELECT name, MAX(salary)` with only `GROUP BY department` | MAX gives a value, not automatically the corresponding row                          |
| `MAX(d.salary)`                                            | `salary` belongs to Employee                                                        |
| `e.employee_id = d.employeee_id`                           | Wrong relationship/key                                                              |
| `d.department`                                             | Column doesn't exist                                                                |
| Missing `ON` condition                                     | Tables are not properly related                                                     |
| Using only `MAX()`                                         | Doesn't retrieve the employee row                                                   |
| Assuming JOIN solves the maximum-row problem               | JOIN combines related data, but doesn't identify the row having the aggregate value |

---

# 17. JOIN vs GROUP BY vs HAVING vs SUBQUERY

| Concept                 | Main Purpose                               |
| ----------------------- | ------------------------------------------ |
| `JOIN`                  | Combine related rows from tables           |
| `GROUP BY`              | Create groups                              |
| `COUNT/AVG/SUM/MAX/MIN` | Calculate aggregate values                 |
| `HAVING`                | Filter groups                              |
| `SUBQUERY`              | Use the result of one query inside another |

### Simple mental model

```text
JOIN
↓
Bring related data together

GROUP BY
↓
Create groups

Aggregate
↓
Calculate values for each group

HAVING
↓
Filter groups

SUBQUERY
↓
Use a calculated/query result to solve another part of the problem
```

---

# 18. The Most Important Lesson

## `MAX()` gives a value, not the row.

For example:

```text
MAX(salary) = 80000
```

does not automatically give:

```text
Aman | 80000 | IT
```

You need another mechanism to connect:

```text
MAX(salary)
     ↕
employee row
```

Possible solutions include:

```text
Subquery
JOIN with a derived result
Window Function
```

For now, the **subquery solution** is the next concept to master.

---

# 19. Interview Cheat Sheet

```text
"What is the highest salary in each department?"
→ GROUP BY + MAX()

"Who has the highest salary overall?"
→ MAX() + subquery

"Who has the highest salary in each department?"
→ Need the maximum value + corresponding employee row
→ Subquery / Window Function

"Employees earning above overall average?"
→ Subquery

"Departments with average salary > 60000?"
→ GROUP BY + HAVING

"Count employees including departments with zero employees?"
→ LEFT JOIN + COUNT(right_table.id)
```

---

# Final Mental Picture

```text
                 Q9
                  |
         "Highest salary
          in each department"
                  |
                  ↓
        GROUP BY + MAX()
                  |
                  ↓
       We get only the value
                  |
                  ↓
       "Who owns this value?"
                  |
                  ↓
          Need another step
                  |
        ┌─────────┴─────────┐
        ↓                   ↓
    SUBQUERY          WINDOW FUNCTION
```

This is the exact point where SQL moves from basic aggregation into **real interview-level querying**.
