# LeetCode 2485 — Find the Pivot Integer

## Metadata

* **LeetCode:** 2485
* **Problem:** Find the Pivot Integer
* **Difficulty:** Easy
* **Topics:** Math, Prefix Sum, Binary Search
* **Pattern:** Prefix Sum / Closed-Form Sum Formula
* **Key Technique:** Use the fact that `sum(1..x) = sum(x..n)` reduces algebraically to `x² = n(n+1)/2`, so the pivot (if it exists) is just a square root check
* **Optimal Complexity:** `O(1)` Time, `O(1)` Space

---

## Problem Statement

Given a positive integer `n`, find an integer `x` (`1 <= x <= n`) such that the sum of all integers from `1` to `x` equals the sum of all integers from `x` to `n`. Return `x`, or `-1` if no such value exists.

---

## Approaches

1. **Brute Force — Sum Both Sides for Every Candidate**
2. **Better — Prefix Sum + Total Sum**
3. **Optimal — Closed-Form Formula**

---

# Approach 1 — Brute Force / Sum Both Sides for Every Candidate

## Idea

For every candidate `x` from `1` to `n`, directly sum `1..x` and `x..n` from scratch and compare.

## Dry Run

```text
n = 8
```

`x = 6`:

```text
left sum = 1+2+3+4+5+6 = 21
right sum = 6+7+8 = 21
```

`21 == 21` → pivot found at `6`.

## Algorithm

1. For each `x` from `1` to `n`:

   * Sum `1..x` directly.
   * Sum `x..n` directly.
   * If equal, return `x`.
2. If no `x` qualifies, return `-1`.

## Complexity

* **Time:** `O(n²)`

  * Each of the `n` candidates triggers two fresh summation loops that can each scan up to `n` elements.
* **Space:** `O(1)`

  * Only a couple of running sum variables — no extra structures allocated.

## Notes / Tips

* Both sums are recomputed from scratch for every candidate, throwing away nearly all the work from the previous candidate.
* Correct but far too slow for large `n`.

## Code

```cpp
class Solution {
public:
    int pivotInteger(int n) {
        for (int x = 1; x <= n; x++) {
            int leftSum = 0, rightSum = 0;

            for (int i = 1; i <= x; i++) {
                leftSum += i;
            }
            for (int i = x; i <= n; i++) {
                rightSum += i;
            }

            if (leftSum == rightSum) {
                return x;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Better / Prefix Sum + Total Sum

## Idea

Compute the total sum `1..n` once. Scan `x` from `1` to `n`, keeping a running left sum. The right sum is derived as `total - leftSum + x` (since `x` itself is shared between both sides).

## Dry Run

```text
n = 8
total = 1+2+...+8 = 36
```

Process:

```text
x=1: leftSum=1, rightSum = 36 - 1 + 1 = 36 → 1 != 36
x=2: leftSum=3, rightSum = 36 - 3 + 2 = 35 → no
x=3: leftSum=6, rightSum = 36 - 6 + 3 = 33 → no
x=4: leftSum=10, rightSum = 36 - 10 + 4 = 30 → no
x=5: leftSum=15, rightSum = 36 - 15 + 5 = 26 → no
x=6: leftSum=21, rightSum = 36 - 21 + 6 = 21 → match! return 6
```

## Algorithm

1. Compute `total = n * (n + 1) / 2`.
2. Initialize `leftSum = 0`.
3. For each `x` from `1` to `n`:

   * `leftSum += x`.
   * `rightSum = total - leftSum + x`.
   * If `leftSum == rightSum`, return `x`.
4. If no `x` qualifies, return `-1`.

## Complexity

* **Time:** `O(n)`

  * A single pass over `x` from `1` to `n`, each step doing constant work.
* **Space:** `O(1)`

  * Only `total` and `leftSum` are tracked — no arrays used.

## Notes / Tips

* This is the same "total minus running left sum" identity used in LC 724 (Find Pivot Index) and LC 1991 — the key difference here is both sides share the pivot element itself, hence the `+ x` correction term.
* Already fast enough for typical constraints, but the closed-form approach removes the loop entirely.

## Code

```cpp
class Solution {
public:
    int pivotInteger(int n) {
        int total = n * (n + 1) / 2;
        int leftSum = 0;

        for (int x = 1; x <= n; x++) {
            leftSum += x;
            int rightSum = total - leftSum + x;

            if (leftSum == rightSum) {
                return x;
            }
        }

        return -1;
    }
};
```

---

# Approach 3 — Optimal / Closed-Form Formula

## Idea

Setting `sum(1..x) = sum(x..n)` algebraically:

```text
x(x+1)/2 = n(n+1)/2 - (x-1)x/2
```

Simplifying both sides collapses to:

```text
x² = n(n+1)/2
```

So `x` exists exactly when `n(n+1)/2` is a perfect square, and `x` is simply its square root.

## Dry Run

```text
n = 8
```

```text
n(n+1)/2 = 8*9/2 = 36
sqrt(36) = 6 → 6*6 = 36 ✓
```

Return `6`.

```text
n = 4
```

```text
n(n+1)/2 = 4*5/2 = 10
sqrt(10) ≈ 3.16 → not a perfect square
```

Return `-1`.

## Algorithm

1. Compute `total = n * (n + 1) / 2`.
2. Compute `x = round(sqrt(total))`.
3. If `x * x == total`, return `x`.
4. Otherwise, return `-1`.

## Complexity

* **Time:** `O(1)`

  * Just a fixed number of arithmetic operations and one square root call.
* **Space:** `O(1)`

  * No extra structures used.

## Notes / Tips

* Using `sqrt()` on integers can introduce floating-point rounding errors — always verify with `x * x == total` after rounding rather than trusting the square root output directly.
* This is a common pattern for "does some running sum split evenly" problems — algebraic manipulation often collapses a scan into a single closed-form check.
* `n(n+1)/2` can overflow a 32-bit `int` for large `n`; casting to `long long` before computing avoids that risk.

## Code

```cpp
class Solution {
public:
    int pivotInteger(int n) {
        long long total = (long long)n * (n + 1) / 2;
        long long x = (long long)sqrt((double)total);

        for (long long candidate = x - 1; candidate <= x + 1; candidate++) {
            if (candidate > 0 && candidate * candidate == total) {
                return (int)candidate;
            }
        }

        return -1;
    }
};
```

---

## Key Template

```text
total = n * (n + 1) / 2
x = round(sqrt(total))

check x-1, x, x+1 for x*x == total (guards floating-point rounding)

return matching x, or -1 if none match
```