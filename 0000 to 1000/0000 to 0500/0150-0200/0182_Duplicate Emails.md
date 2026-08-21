# 182. Duplicate Emails

## Metadata

* **Topic:** SQL
* **Difficulty:** Easy
* **Pattern:** `GROUP BY` + `HAVING`
* **Key Pattern:** Group identical values and keep groups with count > 1.

---

## Idea

We need to find email addresses that appear **more than once**.

Group rows by `email` and count how many times each email occurs.

Then use:

```sql
HAVING COUNT(*) > 1
```

`HAVING` is used because we are filtering **groups**, not individual rows.

---

## Example

```text
Email
-----
a@x.com
b@x.com
a@x.com
c@x.com
b@x.com
```

After grouping:

```text
a@x.com → 2
b@x.com → 2
c@x.com → 1
```

Therefore:

```text
a@x.com
b@x.com
```

are duplicates.

---

## Algorithm

1. Group rows by `email`.
2. Count occurrences of each email.
3. Keep groups where count is greater than `1`.
4. Return the email.

---

## Complexity

* **Time:** Depends on the database execution plan.
* **Space:** Depends on the database execution plan.

---

## Notes / Tips

### `WHERE` vs `HAVING`

```text
WHERE   → filters individual rows
HAVING  → filters grouped results
```

Common duplicate-finding template:

```sql
SELECT column
FROM table
GROUP BY column
HAVING COUNT(*) > 1;
```

---

## Code

```sql
SELECT email
FROM Person
GROUP BY email
HAVING COUNT(*) > 1;
```
