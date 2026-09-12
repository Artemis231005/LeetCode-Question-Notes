# LeetCode 3870 — Count Commas in Range

## Metadata

* **LeetCode:** 3870
* **Problem:** Count Commas in Range
* **Difficulty:** Easy
* **Topics:** Math
* **Pattern:** Digit-Length Grouping
* **Key Technique:** Numbers with the same digit length always contain the same number of commas, so group by digit length instead of checking each number individually
* **Optimal Complexity:** `O(log n)` Time, `O(1)` Space (or `O(1)` Time exploiting this problem's specific constraint bound)

---

## Problem Statement

Given an integer `n`, return the total number of commas used when writing every integer from `1` to `n` in standard formatting (a comma inserted after every three digits from the right; numbers with fewer than 4 digits have no commas).

---

## Approaches

1. **Brute Force — Format Each Number and Count Commas**
2. **Better — Group by Digit Length**
3. **Optimal — Constraint-Specific Closed Form**

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

* Every number with the same digit count always produces the exact same number of commas — recomputing this per number throws away that repeated structure.
* Given `n <= 10^5`, this brute force runs comfortably fast in practice, but it's still doing far more work than necessary.

## Code

```cpp
class Solution {
public:
    int countCommas(int n) {
        int total = 0;

        for (int i = 1; i <= n; i++) {
            string s = to_string(i);
            int len = s.size();

            total += (len - 1) / 3;
        }

        return total;
    }
};
```

---

# Approach 2 — Better / Group by Digit Length

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

  * The number of digit-length groups is proportional to the number of digits in `n`, which is `O(log n)`.
* **Space:** `O(1)`

  * Only a running total and loop counters are used.

## Notes / Tips

* This is the standard "group by a shared property instead of checking every element" optimization — since comma count is a step function of digit length, working directly with digit-length ranges collapses up to `10^5` individual checks into at most `6` group computations.
* Clipping `end` with `min(n, 10^d - 1)` is what correctly handles the final partial group, where `n` falls in the middle of a digit-length range rather than at its boundary.
* This version is fully general — it works correctly no matter how large `n` gets, unlike Approach 3 below.

## Code

```cpp
class Solution {
public:
    int countCommas(int n) {
        long long total = 0;
        long long start = 1;
        int d = 1;

        while (start <= n) {
            long long end = min((long long)n, start * 10 - 1);
            long long count = end - start + 1;
            int commas = (d - 1) / 3;

            total += count * commas;

            start *= 10;
            d++;
        }

        return (int)total;
    }
};
```

---

# Approach 3 — Optimal / Constraint-Specific Closed Form

## Idea

This problem's constraint caps `n` at `10^5` (i.e. `100000`), which has only `6` digits. Checking the digit-length groups from Approach 2: `4`-digit, `5`-digit, and `6`-digit numbers all give `(d-1)/3 = 1` comma each (`(4-1)/3=1`, `(5-1)/3=1`, `(6-1)/3=1`) — none of them ever reach the `7`-digit threshold where the comma count would jump to `2`. This means, for this specific constraint range, **every** number from `1000` up to `n` contributes exactly `1` comma, and nothing below `1000` contributes any. So the total is simply the count of integers in `[1000, n]`, i.e. `n - 999` (or `0` if `n < 1000`).

## Dry Run

```text
n = 1002
```

```text
n >= 1000 → total = 1002 - 999 = 3
```

Matches both earlier approaches.

```text
n = 998
```

```text
n < 1000 → total = 0
```

## Algorithm

1. If `n < 1000`, return `0` (no number in range has 4+ digits, so no commas anywhere).
2. Otherwise, return `n - 999` (every number from `1000` to `n` contributes exactly one comma).

## Complexity

* **Time:** `O(1)`

  * A single comparison and subtraction — no loop at all.
* **Space:** `O(1)`

  * No extra structures used.

## Notes / Tips

* This shortcut is only valid because the problem's constraint (`n <= 10^5`) guarantees every qualifying number has between `4` and `6` digits, all of which map to exactly `1` comma — it would silently produce wrong answers if `n` could reach `7` digits or more (e.g. `n = 1000000` would actually need `2` commas for `1,000,000`, but this formula would incorrectly still add `1` per number).
* This is a good example of exploiting a problem's specific input bounds for a simpler solution, as opposed to Approach 2's fully general digit-grouping technique — worth recognizing the trade-off between "correct for this exact constraint" and "correct for any input" when choosing which to rely on.
* If the constraint were ever loosened, Approach 2's grouping technique would need to be used instead, since it correctly scales to any digit-length range.

## Code

```cpp
class Solution {
public:
    int countCommas(int n) {
        if (n < 1000) {
            return 0;
        }

        return n - 999;
    }
};
```

---

## Key Template

```text
# General (Approach 2):
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

# Constraint-specific shortcut (Approach 3, valid only for n <= 10^5):
if n < 1000: return 0
return n - 999
```