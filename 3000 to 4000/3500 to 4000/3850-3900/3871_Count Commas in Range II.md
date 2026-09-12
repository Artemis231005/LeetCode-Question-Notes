# LeetCode 3871 — Count Commas in Range II

## Metadata

* **LeetCode:** 3871
* **Problem:** Count Commas in Range II
* **Difficulty:** Medium
* **Topics:** Math
* **Pattern:** Digit-Length Grouping
* **Key Technique:** Numbers with the same digit length always contain the same number of commas, so group by digit length instead of checking each number individually
* **Optimal Complexity:** `O(log n)` Time, `O(1)` Space

---

## Problem Statement

Given an integer `n` (up to `10^15`), return the total number of commas used when writing every integer from `1` to `n` in standard formatting (a comma inserted after every three digits from the right; numbers with fewer than 4 digits have no commas).

This is the same problem statement as LC 3870 (Count Commas in Range), but with `n` allowed up to `10^15` instead of `10^5`. That larger bound rules out both the brute force (too slow) and the LC 3870 constraint-specific shortcut (`n - 999`, which only works because that problem's `n` never exceeded 6 digits) — the general digit-grouping technique becomes the only viable approach.

---

## Approaches

1. **Brute Force — Format Each Number and Count Commas**
2. **Optimal — Group by Digit Length**

---

# Approach 1 — Brute Force / Format Each Number and Count Commas

## Idea

For every integer from `1` to `n`, actually build its comma-formatted string (inserting a comma every three digits from the right) and count how many commas appear in it, accumulating the total.

## Dry Run

```text
n = 1002
```

```text
1    → "1"    → 0 commas
...
999  → "999"  → 0 commas
1000 → "1,000" → 1 comma
1001 → "1,001" → 1 comma
1002 → "1,002" → 1 comma
```

Total commas: `1 + 1 + 1 = 3`.

## Algorithm

1. Initialize `total = 0`.
2. For each `i` from `1` to `n`:

   * Convert `i` to a string.
   * Insert a comma after every three digits counting from the right.
   * Count the commas in the resulting string and add to `total`.
3. Return `total`.

## Complexity

* **Time:** `O(n * log n)`

  * For each of the `n` numbers, formatting and comma-counting takes time proportional to its digit count (`O(log n)`).
* **Space:** `O(log n)`

  * For the temporary formatted string of each number.

## Notes / Tips

* With `n` up to `10^15`, this approach is nowhere near fast enough — looping over every single number is off the table entirely, unlike LC 3870 where it was slow but tolerable.
* Every number with the same digit count always produces the exact same number of commas — recomputing this per number throws away that repeated structure, and at this scale that redundancy is fatal rather than just wasteful.

## Code

```cpp
class Solution {
public:
    long long countCommas(long long n) {
        long long total = 0;

        for (long long i = 1; i <= n; i++) {
            int len = to_string(i).size();
            total += (len - 1) / 3;
        }

        return total;
    }
};
```

---

# Approach 2 — Optimal / Group by Digit Length

## Idea

The number of commas in a number depends only on how many digits it has: a `d`-digit number always has exactly `(d-1) / 3` commas (1-3 digits → 0, 4-6 digits → 1, 7-9 digits → 2, and so on). So instead of checking every number individually, iterate over each possible digit length, find how many numbers of that length fall within `[1, n]`, and multiply that count by the fixed comma count for that length.

## Dry Run

```text
n = 1002
```

Digit length `1` (range `1-9`):

```text
count = 9, commas per number = (1-1)/3 = 0 → contributes 0
```

Digit length `2` (range `10-99`):

```text
count = 90, commas per number = (2-1)/3 = 0 → contributes 0
```

Digit length `3` (range `100-999`):

```text
count = 900, commas per number = (3-1)/3 = 0 → contributes 0
```

Digit length `4` (range `1000-9999`, clipped to `n=1002`):

```text
clipped range = 1000-1002 → count = 3
commas per number = (4-1)/3 = 1 → contributes 3*1 = 3
```

No higher digit lengths reach `n`. Total: `0+0+0+3 = 3`.

### Larger example showing the higher-digit groups matter here

```text
n = 10^15 = 1000000000000000  (16 digits)
```

Digit length `16` alone (range `10^15..10^15`, just the single value `n`):

```text
commas per number = (16-1)/3 = 5
```

This 5-comma group is exactly the kind of case LC 3870's `n - 999` shortcut could never handle — it only ever assumed a fixed 1-comma group, which breaks down completely once digit lengths climb this high.

## Algorithm

1. Initialize `total = 0`.
2. For each digit length `d` starting from `1`, while `10^(d-1) <= n`:

   * Compute the range for this digit length: `start = 10^(d-1)`, `end = min(n, 10^d - 1)`.
   * `count = end - start + 1`.
   * `commas = (d - 1) / 3`.
   * `total += count * commas`.
3. Return `total`.

## Complexity

* **Time:** `O(log n)`

  * The number of digit-length groups is proportional to the number of digits in `n`, which is `O(log n)` — at most `16` groups even for `n = 10^15`.
* **Space:** `O(1)`

  * Only a running total and loop counters are used.

## Notes / Tips

* This is the standard "group by a shared property instead of checking every element" optimization — since comma count is a step function of digit length, working directly with digit-length ranges collapses up to `10^15` individual checks into at most `16` group computations.
* Clipping `end` with `min(n, 10^d - 1)` is what correctly handles the final partial group, where `n` falls in the middle of a digit-length range rather than at its boundary.
* Every value here must use a 64-bit type (`long long`) — `10^15` and its digit-length powers overflow a 32-bit `int` well before the computation finishes, unlike LC 3870 where 32-bit arithmetic was still safe.
* Unlike LC 3870, no constraint-specific shortcut exists at this scale — the comma count per number keeps climbing (`1, 2, 3, 4, 5...`) across a wide range of digit lengths, so this grouping technique is the actual intended solution here, not just the "more general but unnecessary" alternative.

## Code

```cpp
class Solution {
public:
    long long countCommas(long long n) {
        long long total = 0;
        long long start = 1;
        int d = 1;

        while (start <= n) {
            long long end = min(n, start * 10 - 1);
            long long count = end - start + 1;
            long long commas = (d - 1) / 3;

            total += count * commas;

            start *= 10;
            d++;
        }

        return total;
    }
};
```

---

## Key Template

```text
total = 0
start = 1
d = 1

while start <= n:
    end = min(n, start * 10 - 1)
    count = end - start + 1
    commas = (d - 1) / 3
    total += count * commas

    start *= 10
    d += 1

return total
```