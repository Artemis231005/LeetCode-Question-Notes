# LeetCode 507 — Perfect Number

## Metadata

* **LeetCode:** 507
* **Problem:** Perfect Number
* **Difficulty:** Easy
* **Topics:** Math
* **Pattern:** Divisor Sum via Square Root Bound
* **Key Technique:** Only check divisors up to `sqrt(n)`, adding both the divisor and its paired quotient at once
* **Optimal Complexity:** `O(sqrt(n))` Time, `O(1)` Space

---

## Problem Statement

Given an integer `num`, return `true` if it is a "perfect number" — a positive integer equal to the sum of its positive divisors, excluding the number itself.

---

## Approaches

1. **Brute Force — Check Every Number from 1 to n-1**
2. **Optimal — Divisor Pairs up to sqrt(n)**

---

# Approach 1 — Brute Force / Check Every Number from 1 to n-1

## Idea

Loop through every integer from `1` to `num - 1`, and whenever one evenly divides `num`, add it to a running sum. Compare the final sum to `num`.

## Dry Run

```text
num = 28
```

Check divisors from `1` to `27`:

```text
1 → 28 % 1 == 0 → sum = 1
2 → 28 % 2 == 0 → sum = 3
4 → 28 % 4 == 0 → sum = 7
7 → 28 % 7 == 0 → sum = 14
14 → 28 % 14 == 0 → sum = 28
```

Other values (`3, 5, 6, 8, ..., 27`) don't divide evenly and are skipped.

Final `sum = 28`, matches `num` → return `true`.

## Algorithm

1. If `num <= 1`, return `false` (no positive divisors excluding itself).
2. Initialize `sum = 0`.
3. For each `i` from `1` to `num - 1`:

   * If `num % i == 0`, add `i` to `sum`.
4. Return `sum == num`.

## Complexity

* **Time:** `O(n)`

  * A single pass checks every integer up to `num - 1` for divisibility.
* **Space:** `O(1)`

  * Only a running sum is tracked.

## Notes / Tips

* Correct but scans far more candidates than necessary — divisors always come in pairs `(i, num/i)`, so checking past `sqrt(num)` is redundant.
* Fine for small `num`, but degrades for large inputs since it's linear in the value itself, not its number of digits.

## Code

```cpp
class Solution {
public:
    bool checkPerfectNumber(int num) {
        if (num <= 1) {
            return false;
        }

        int sum = 0;
        for (int i = 1; i < num; i++) {
            if (num % i == 0) {
                sum += i;
            }
        }

        return sum == num;
    }
};
```

---

# Approach 2 — Optimal / Divisor Pairs up to sqrt(n)

## Idea

Every divisor `i` of `num` pairs with a complementary divisor `num / i`. So it's enough to check `i` from `1` up to `sqrt(num)` — for each divisor found, add both `i` and `num / i` to the sum at once (careful not to double-count when `i == num / i`, i.e. when `num` is a perfect square).

## Dry Run

```text
num = 28
sqrt(28) ≈ 5.29 → check i = 1 to 5
```

```text
i=1: 28 % 1 == 0 → add 1 and 28/1=28... 
```

Wait — since we exclude `num` itself, the pairing needs to skip adding `num` when `i == 1`. Correct dry run:

```text
i=1: 28 % 1 == 0 → add 1, and pair 28/1=28 → but 28 is num itself, exclude it
     sum = 1
i=2: 28 % 2 == 0 → add 2, and pair 28/2=14 → sum = 1+2+14 = 17
i=3: 28 % 3 != 0 → skip
i=4: 28 % 4 == 0 → add 4, and pair 28/4=7 → sum = 17+4+7 = 28
i=5: 28 % 5 != 0 → skip
```

Final `sum = 28`, matches `num` → return `true`.

## Algorithm

1. If `num <= 1`, return `false`.
2. Initialize `sum = 1` (since `1` always divides `num` for `num > 1`, and it's never equal to `num` itself so it's safe to include upfront).
3. For `i` from `2` to `sqrt(num)`:

   * If `num % i == 0`:

     * Add `i` to `sum`.
     * Compute the paired divisor `j = num / i`.
     * If `j != i` and `j != num`, add `j` to `sum` as well.
4. Return `sum == num`.

## Complexity

* **Time:** `O(sqrt(n))`

  * Only divisors up to `sqrt(num)` are checked, with the paired divisor derived directly instead of searched for.
* **Space:** `O(1)`

  * Only a running sum and loop variable are tracked.

## Notes / Tips

* The `j != i` check is essential when `num` is a perfect square (e.g. `num = 36`, `i = 6` pairs with itself) — without it, that divisor would be double-counted.
* The `j != num` check guards against including `num` itself as a "paired divisor" when `i = 1` (since `num / 1 = num`), which the problem explicitly excludes.
* Same square-root divisor-pairing trick used in primality testing and factorization problems — checking up to `sqrt(n)` instead of `n` is a very common optimization whenever "divisors of n" come up.

## Code

```cpp
class Solution {
public:
    bool checkPerfectNumber(int num) {
        if (num <= 1) {
            return false;
        }

        int sum = 1;

        for (int i = 2; (long long)i * i <= num; i++) {
            if (num % i == 0) {
                sum += i;

                int j = num / i;
                if (j != i && j != num) {
                    sum += j;
                }
            }
        }

        return sum == num;
    }
};
```

---

## Key Template

```text
if num <= 1: return false

sum = 1

for i from 2 to sqrt(num):
    if num % i == 0:
        sum += i
        j = num / i
        if j != i and j != num:
            sum += j

return sum == num
```