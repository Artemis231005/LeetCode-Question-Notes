# 181. Employees Earning More Than Their Managers

## Metadata

* **Topic:** SQL
* **Difficulty:** Easy
* **Pattern:** Self Join
* **Key Pattern:** Join a table with itself to compare employee and manager rows.

---

## Idea

The `Employee` table contains both employees and managers.

We need employees whose salary is **greater than their manager's salary**.

Since employee and manager are stored in the **same table**, use a **self join**.

Match:

```text
employee.managerId = manager.id
```

Then compare:

```text
employee.salary > manager.salary
```

---

## Algorithm

1. Treat `Employee` as the employee table.
2. Treat `Employee` again as the manager table.
3. Join employee with their manager using `managerId`.
4. Keep rows where employee salary is greater than manager salary.
5. Return the employee's name.

---

## Example

```text
Employee
id | name  | salary | managerId
1  | Joe   | 70000  | 3
2  | Henry | 80000  | 4
3  | Sam   | 60000  | NULL
4  | Max   | 90000  | NULL
```

Joe earns `70000`, while his manager Sam earns `60000`.

Therefore:

```text
Joe
```

is included.

---

## Complexity

* **Time:** Depends on the database execution plan.
* **Space:** Depends on the database execution plan.

---

## Notes / Tips

**Self Join Template:**

```sql
SELECT ...
FROM TableA a
JOIN TableA b
    ON a.relation = b.id
WHERE ...;
```

Here:

```text
e → employee
m → manager
```

The same table gets two different aliases so we can compare two rows from it.

---

## Code

```sql
SELECT e.name AS Employee
FROM Employee e
JOIN Employee m
    ON e.managerId = m.id
WHERE e.salary > m.salary;
```
