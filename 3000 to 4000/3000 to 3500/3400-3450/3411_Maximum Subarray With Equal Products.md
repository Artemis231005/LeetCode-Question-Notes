# LeetCode 3411 — Maximum Subarray With Equal Products

## Metadata

* **LeetCode:** 3411
* **Problem:** Maximum Subarray With Equal Products
* **Difficulty:** Easy
* **Topics:** Array, Math, Number Theory, Sliding Window, Enumeration
* **Pattern:** Subarray Enumeration with Incremental Aggregates and a Bounded Product
* **Key Technique:** For a fixed start, extend the end one element at a time while updating the product, GCD and LCM incrementally, and stop early once the product exceeds the largest value `lcm * gcd` could ever take
* **Optimal Complexity:** `O(n² log M)` Time, `O(1)` Space (`M` = max element)

---

## Problem Statement

Given an array of positive integers `nums`, an array is "product equivalent" if `prod(arr) == lcm(arr) * gcd(arr)`. Return the length of the longest product equivalent subarray of `nums`.

---

## Approaches

1. **Brute Force — Recompute prod, gcd, lcm for Every Subarray**
2. **Optimal — Incremental Aggregates with Early Termination**

---

# Approach 1 — Brute Force / Recompute prod, gcd, lcm for Every Subarray

## Idea

For every subarray, scan all of its elements from scratch to compute the product, GCD and LCM, then check whether `prod == lcm * gcd`. Track the longest subarray that passes.

## Dry Run

```text
nums = [2, 3, 4, 5, 6]
```

Subarray `[2,3]`:

```text
prod = 6, gcd = 1, lcm = 6 → 6 == 6 * 1 ✓ → length 2
```

Subarray `[2,3,4]`:

```text
prod = 24, gcd = 1, lcm = 12 → 24 != 12 ✗
```

Subarray `[3,4,5]`:

```text
prod = 60, gcd = 1, lcm = 60 → 60 == 60 * 1 ✓ → length 3
```

Subarray `[4,5,6]`:

```text
prod = 120, gcd = 1, lcm = 60 → 120 != 60 ✗
```

No longer valid subarray exists. Answer: `3`.

## Algorithm

1. Initialize `best = 0`.
2. For every start index `i` and end index `j >= i`:

   * Compute `prod`, `gcd` and `lcm` of `nums[i..j]` by scanning every element.
   * If `prod == lcm * gcd`, update `best = max(best, j - i + 1)`.
3. Return `best`.

## Complexity

* **Time:** `O(n³ log M)`

  * `O(n²)` subarrays, each needing an `O(n)` scan with a GCD computation per element.
* **Space:** `O(1)`

  * Only running aggregates.

## Notes / Tips

* Overflow is a real concern: with `n` up to `100` and values up to `10`, a raw product can reach `10^100`. Since `lcm * gcd` can never exceed `2520 * 10 = 25200` (`2520` is `lcm(1..10)`), any product above that bound can be rejected immediately, which also keeps the arithmetic safe.
* Rescanning each subarray from scratch repeats work that consecutive subarrays share.

## Code

```cpp
class Solution {
public:
    int maxLength(vector<int>& nums) {
        int n = nums.size();
        const long long LIMIT = 2520LL * 10;
        int best = 0;

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                long long prod = 1;
                long long g = 0, l = 1;
                bool tooBig = false;

                for (int k = i; k <= j; k++) {
                    prod *= nums[k];
                    if (prod > LIMIT) {
                        tooBig = true;
                        break;
                    }
                    g = __gcd(g, (long long)nums[k]);
                    l = l / __gcd(l, (long long)nums[k]) * nums[k];
                }

                if (!tooBig && prod == l * g) {
                    best = max(best, j - i + 1);
                }
            }
        }

        return best;
    }
};
```

---

# Approach 2 — Optimal / Incremental Aggregates with Early Termination

## Idea

Fix a start index `i` and extend the end `j` one element at a time, keeping `prod`, `gcd` and `lcm` as running values. Adding a new element updates each in a constant number of operations: multiply into the product, take `gcd(g, x)`, and set `lcm = lcm / gcd(lcm, x) * x`. All elements are at least `1`, so the product never decreases as the subarray grows, while `lcm * gcd` never exceeds `25200`. So once `prod > 25200`, no longer subarray from this start can qualify and the inner loop can stop.

A mismatch alone is not a reason to stop, because adding an element can raise the LCM and make the equation true again, so only the product bound is used for early termination.

## Dry Run

```text
nums = [1, 2, 1, 2, 1, 1, 1]
```

Start `i = 0`:

```text
j=0: [1]         prod=1, g=1, l=1 → 1 == 1*1 ✓ len 1
j=1: [1,2]       prod=2, g=1, l=2 → 2 == 2*1 ✓ len 2
j=2: [1,2,1]     prod=2, g=1, l=2 → ✓ len 3
j=3: [1,2,1,2]   prod=4, g=1, l=2 → 4 != 2 ✗
j=4..6: prod stays 4, l stays 2 → ✗
best for this start = 3
```

Start `i = 2`:

```text
j=2: [1]           ✓ len 1
j=3: [1,2]         prod=2, g=1, l=2 → ✓ len 2
j=4: [1,2,1]       ✓ len 3
j=5: [1,2,1,1]     ✓ len 4
j=6: [1,2,1,1,1]   ✓ len 5
```

Overall best: `5`, matching the expected output.

## Algorithm

1. Initialize `best = 0` and `LIMIT = 25200`.
2. For each start index `i`:

   * Initialize `prod = 1`, `g = 0`, `l = 1`.
   * For each end index `j` from `i` to `n - 1`:

     * `prod *= nums[j]`; if `prod > LIMIT`, break.
     * `g = gcd(g, nums[j])`.
     * `l = l / gcd(l, nums[j]) * nums[j]`.
     * If `prod == l * g`, update `best = max(best, j - i + 1)`.
3. Return `best`.

## Complexity

* **Time:** `O(n² log M)`

  * `O(n²)` (start, end) pairs, each doing `O(log M)` work for the GCD updates, where `M` is the maximum element.
* **Space:** `O(1)`

  * Only three running values and a counter.

## Notes / Tips

* The break condition is only safe because every element is positive: the product is non-decreasing in the subarray length, while `lcm * gcd` is bounded by a constant.
* Using `g = 0` as the starting GCD works because `gcd(0, x) = x`.
* Computing the LCM as `l / gcd(l, x) * x` (divide first, then multiply) avoids a larger intermediate value than `l * x / gcd(l, x)`.
* Same "fix the start, extend the end while maintaining running aggregates" pattern as the sliding window approaches earlier, except the aggregates here are not invertible, so a true shrinking window is not possible and enumeration per start is used.

## Code

```cpp
class Solution {
public:
    int maxLength(vector<int>& nums) {
        int n = nums.size();
        const long long LIMIT = 2520LL * 10;
        int best = 0;

        for (int i = 0; i < n; i++) {
            long long prod = 1;
            long long g = 0, l = 1;

            for (int j = i; j < n; j++) {
                prod *= nums[j];
                if (prod > LIMIT) {
                    break;
                }

                g = __gcd(g, (long long)nums[j]);
                l = l / __gcd(l, (long long)nums[j]) * nums[j];

                if (prod == l * g) {
                    best = max(best, j - i + 1);
                }
            }
        }

        return best;
    }
};
```

---

## Key Template

```text
LIMIT = 25200   # lcm(1..10) * 10
best = 0

for i in 0..n-1:
    prod = 1, g = 0, l = 1

    for j in i..n-1:
        prod *= nums[j]
        if prod > LIMIT: break

        g = gcd(g, nums[j])
        l = l / gcd(l, nums[j]) * nums[j]

        if prod == l * g:
            best = max(best, j - i + 1)

return best
```