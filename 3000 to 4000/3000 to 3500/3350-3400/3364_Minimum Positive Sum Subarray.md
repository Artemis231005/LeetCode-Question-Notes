# LeetCode 3364 — Minimum Positive Sum Subarray

## Metadata

* **LeetCode:** 3364
* **Problem:** Minimum Positive Sum Subarray
* **Difficulty:** Easy
* **Topics:** Array, Sliding Window, Prefix Sum
* **Pattern:** Prefix Sum Range Queries over Bounded Window Lengths
* **Key Technique:** Precompute prefix sums once so the sum of any subarray is a single subtraction, then check every start index for every allowed length between `l` and `r`
* **Optimal Complexity:** `O(n * (r - l + 1))` Time, `O(n)` Space

---

## Problem Statement

Given an integer array `nums` and two integers `l` and `r`, find the minimum sum among all subarrays whose length is between `l` and `r` (inclusive) and whose sum is greater than `0`. Return `-1` if no such subarray exists.

---

## Approaches

1. **Brute Force — Sum Every Valid Subarray Directly**
2. **Optimal — Prefix Sum for O(1) Range Sums**

---

# Approach 1 — Brute Force / Sum Every Valid Subarray Directly

## Idea

For every allowed length from `l` to `r`, and every possible start index, add up the subarray's elements directly. If the sum is positive, compare it against the smallest positive sum found so far.

## Dry Run

```text
nums = [3, -2, 1, 4], l = 2, r = 3
```

Length `2`:

```text
[3,-2] = 1  → positive, best = 1
[-2,1] = -1 → not positive
[1,4]  = 5  → positive, best stays 1
```

Length `3`:

```text
[3,-2,1]  = 2 → positive, best stays 1
[-2,1,4]  = 3 → positive, best stays 1
```

Answer: `1`.

## Algorithm

1. Initialize `best = infinity`.
2. For each length `len` from `l` to `r`:

   * For each start index `i` from `0` to `n - len`:

     * Sum `nums[i .. i+len-1]` directly.
     * If the sum is `> 0`, update `best = min(best, sum)`.
3. Return `best`, or `-1` if it was never updated.

## Complexity

* **Time:** `O(n * (r - l + 1) * r)`

  * For each allowed length, each start index requires summing up to `r` elements directly.
* **Space:** `O(1)`

  * Only a running sum and the best value.

## Notes / Tips

* Re-adding every element of a subarray from scratch is redundant, since overlapping subarrays share most of their elements.
* Given the small constraint (`n <= 100`), this still runs instantly, but the repeated summation is exactly what a prefix sum removes.

## Code

```cpp
class Solution {
public:
    int minimumSumSubarray(vector<int>& nums, int l, int r) {
        int n = nums.size();
        int best = INT_MAX;

        for (int len = l; len <= r; len++) {
            for (int i = 0; i + len <= n; i++) {
                int sum = 0;
                for (int j = i; j < i + len; j++) {
                    sum += nums[j];
                }

                if (sum > 0) {
                    best = min(best, sum);
                }
            }
        }

        return best == INT_MAX ? -1 : best;
    }
};
```

---

# Approach 2 — Optimal / Prefix Sum for O(1) Range Sums

## Idea

Build a prefix sum array where `prefix[i]` is the sum of `nums[0..i-1]`. The sum of any subarray `nums[i..i+len-1]` is then `prefix[i + len] - prefix[i]`, computed in `O(1)`. Loop over every allowed length and every start index, and keep the smallest sum that is strictly positive.

## Dry Run

```text
nums = [3, -2, 1, 4], l = 2, r = 3
```

Prefix sums (`prefix[0] = 0`):

```text
prefix = [0, 3, 1, 2, 6]
```

Length `2`:

```text
i=0: prefix[2] - prefix[0] = 1 - 0 = 1  → positive, best = 1
i=1: prefix[3] - prefix[1] = 2 - 3 = -1 → skip
i=2: prefix[4] - prefix[2] = 6 - 1 = 5  → positive, best stays 1
```

Length `3`:

```text
i=0: prefix[3] - prefix[0] = 2 → positive, best stays 1
i=1: prefix[4] - prefix[1] = 3 → positive, best stays 1
```

Answer: `1`, matching the brute-force result.

### No valid subarray

```text
nums = [-2, 2, -3, 1], l = 2, r = 3
```

```text
every subarray of length 2 or 3 has sum <= 0 → best never updated → return -1
```

## Algorithm

1. Build `prefix` of size `n + 1`, with `prefix[0] = 0` and `prefix[i+1] = prefix[i] + nums[i]`.
2. Initialize `best = infinity`.
3. For each length `len` from `l` to `r`:

   * For each start index `i` from `0` to `n - len`:

     * `sum = prefix[i + len] - prefix[i]`.
     * If `sum > 0`, update `best = min(best, sum)`.
4. Return `best`, or `-1` if it was never updated.

## Complexity

* **Time:** `O(n * (r - l + 1))`

  * Building the prefix array is `O(n)`, then each (length, start) pair is checked in `O(1)`.
* **Space:** `O(n)`

  * For the prefix sum array.

## Notes / Tips

* Remember the condition is strictly `sum > 0`: sums of zero or below must be skipped, and the answer stays `-1` if every candidate is skipped.
* If `O(1)` extra space is preferred, the same result comes from fixing each start index and extending the end one element at a time while keeping a running sum, checking the sum only once the length reaches `l` and stopping at length `r`. This has the same time complexity without the prefix array.
* Same prefix sum template as LC 303: precompute once, then answer every range sum with one subtraction.

## Code

```cpp
class Solution {
public:
    int minimumSumSubarray(vector<int>& nums, int l, int r) {
        int n = nums.size();
        vector<int> prefix(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        int best = INT_MAX;

        for (int len = l; len <= r; len++) {
            for (int i = 0; i + len <= n; i++) {
                int sum = prefix[i + len] - prefix[i];

                if (sum > 0) {
                    best = min(best, sum);
                }
            }
        }

        return best == INT_MAX ? -1 : best;
    }
};
```

---

## Key Template

```text
prefix[0] = 0
for i in 0..n-1:
    prefix[i+1] = prefix[i] + nums[i]

best = infinity

for len in l..r:
    for i in 0..n-len:
        sum = prefix[i + len] - prefix[i]
        if sum > 0:
            best = min(best, sum)

return best if best != infinity else -1
```