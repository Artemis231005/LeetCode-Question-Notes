# 196. Delete Duplicate Emails

## Metadata

* **Topic:** SQL
* **Difficulty:** Easy
* **Pattern:** Self Join + `DELETE`
* **Key Pattern:** Delete the duplicate row with the larger `id`.

---

## Idea

We need to delete duplicate emails while keeping the row with the **smallest `id`**.

For two rows with the same email:

```text
p1.id < p2.id
```

`p2` is the duplicate, so delete it.

Use a **self join**:

```sql
Person p1
JOIN Person p2
```

where:

```sql
p1.email = p2.email
AND p1.id < p2.id
```

---

## Dry Run

```text
id | email
---|-------
1  | a@x.com
2  | b@x.com
3  | a@x.com
4  | c@x.com
5  | b@x.com
```

Duplicate pairs:

```text
1, 3 → delete 3
2, 5 → delete 5
```

Remaining:

```text
1 | a@x.com
2 | b@x.com
4 | c@x.com
```

---

## Algorithm

1. Join `Person` with itself.
2. Match rows having the same email.
3. Identify the row with the larger `id`.
4. Delete that row.

---

## Complexity

* **Time:** Depends on the database execution plan.
* **Space:** Depends on the database execution plan.

---

## Notes / Tips

The important condition is:

```sql
p1.email = p2.email
AND p1.id < p2.id
```

This means:

```text
same email + larger id → duplicate → delete
```

### Key Template

```sql
DELETE p2
FROM Table p1
JOIN Table p2
    ON p1.email = p2.email
WHERE p1.id < p2.id;
```

---

## Code

```sql
DELETE p2
FROM Person p1
JOIN Person p2
    ON p1.email = p2.email
WHERE p1.id < p2.id;
```
