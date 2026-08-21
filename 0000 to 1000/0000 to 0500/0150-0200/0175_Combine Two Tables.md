# 175. Combine Two Tables

## Metadata

* **Topic:** SQL
* **Difficulty:** Easy
* **Pattern:** `LEFT JOIN`
* **Key Pattern:** Keep all rows from the first table and match data from the second.

---

## Idea

We have `Person` and `Address` tables.

Need:

```text
FirstName | LastName | City | State
```

Every person must appear, even if they have **no address**.

Therefore, use:

```sql
Person LEFT JOIN Address
```

Join using:

```sql
Person.PersonId = Address.PersonId
```

If a person has no matching address, `City` and `State` become `NULL`.

---

## Example

```text
Person
1 → Allen Wang
2 → Bob Alice

Address
2 → New York, New York
```

Result:

```text
Allen | Wang  | NULL     | NULL
Bob   | Alice | New York | New York
```

Person `1` is retained because we used `LEFT JOIN`.

---

## Algorithm

1. Select `FirstName` and `LastName` from `Person`.
2. Select `City` and `State` from `Address`.
3. `LEFT JOIN` `Address` with `Person`.
4. Match using `PersonId`.

---

## Complexity

* **Time:** Depends on database indexes/query execution plan.
* **Space:** Depends on database execution plan.

---

## Notes / Tips

* `LEFT JOIN` → keeps **all rows from left table**.
* `INNER JOIN` → keeps only matching rows.
* No matching right-side row → right-side columns are `NULL`.
* Use aliases to keep SQL concise.

**Template:**

```sql
SELECT columns
FROM TableA a
LEFT JOIN TableB b
    ON a.key = b.key;
```

---

## Code

```sql
SELECT
    p.FirstName,
    p.LastName,
    a.City,
    a.State
FROM Person p
LEFT JOIN Address a
    ON p.PersonId = a.PersonId;
```
