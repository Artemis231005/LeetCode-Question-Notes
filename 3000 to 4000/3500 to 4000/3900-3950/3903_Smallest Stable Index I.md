# LeetCode 3903 — Smallest Stable Index I

## Metadata

* **LeetCode:** 3903
* **Problem:** Smallest Stable Index I
* **Difficulty:** Easy
* **Topics:** Array, Prefix Sum (Prefix Max / Suffix Min)
* **Pattern:** Precompute One Side, Sweep the Other
* **Key Technique:** Precompute a suffix-min array from the back, then sweep forward keeping a running prefix max, comparing the two at each index
* **Optimal Complexity:** `O(n)` Time, `O(n)` Space

---

## Problem Statement

Given an integer array `nums` of length `n` and an integer `k`, define the instability score at index `i` as `max(nums[0..i]) - min(nums[i..n-1])`. An index is "stable" if its instability score is `<= k`. Return the smallest stable index, or `-1` if none exists.

---

## Approaches

1. **Brute Force — Scan Both Sides for Every Index**
2. **Optimal — Suffix Min Array + Running Prefix Max**

---

# Approach 1 — Brute Force / Scan Both Sides for Every Index

## Idea

For each candidate index `i`, directly scan `nums[0..i]` to find the max and scan `nums[i..n-1]` to find the min, then check if their difference is `<= k`.

## Dry Run

```text
nums = [5, 0, 1, 4], k = 3
```

`i = 0`:

```text
max([5]) = 5, min([5,0,1,4]) = 0 → score = 5
```

`i = 1`:

```text
max([5,0]) = 5, min([0,1,4]) = 0 → score = 5
```

`i = 2`:

```text
max([5,0,1]) = 5, min([1,4]) = 1 → score = 4
```

`i = 3`:

```text
max([5,0,1,4]) = 5, min([4]) = 4 → score = 1
```

`1 <= 3` → stable → return `3`.

## Algorithm

1. For each index `i` from `0` to `n-1`:

   * Compute `maxLeft` by scanning `nums[0..i]`.
   * Compute `minRight` by scanning `nums[i..n-1]`.
   * If `maxLeft - minRight <= k`, return `i`.
2. If no index qualifies, return `-1`.

## Complexity

* **Time:** `O(n²)`

  * Each of the `n` candidates triggers two fresh scans (left max and right min) that can each cover up to `n` elements.
* **Space:** `O(1)`

  * Only a couple of running max/min variables per iteration — no extra structures allocated.

## Notes / Tips

* Recomputing both the max and min from scratch at every index throws away almost all the work from the previous index — the left max only ever grows or stays the same as `i` increases, and the right min only ever shrinks or stays the same.
* This "left side only grows, right side only shrinks" observation is exactly what unlocks the optimal approach.

## Code

```cpp
class Solution {
public:
    int smallestStableIndex(vector<int>& nums, int k) {
        int n = nums.size();

        for (int i = 0; i < n; i++) {
            int maxLeft = nums[0];
            for (int j = 0; j <= i; j++) {
                maxLeft = max(maxLeft, nums[j]);
            }

            int minRight = nums[i];
            for (int j = i; j < n; j++) {
                minRight = min(minRight, nums[j]);
            }

            if (maxLeft - minRight <= k) {
                return i;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Optimal / Suffix Min Array + Running Prefix Max

## Idea

Precompute a `right` array where `right[i]` holds the minimum of `nums[i..n-1]`, built with a single backward pass. Then sweep forward, maintaining a running `left` value that tracks the maximum of `nums[0..i]` as it grows. At each index, compare `left - right[i]` against `k` directly — no need to rescan either side.

## Dry Run

```text
nums = [5, 0, 1, 4], k = 3
```

Build `right` (suffix min), scanning backward:

```text
right[3] = 4
right[2] = min(4, 1) = 1
right[1] = min(1, 0) = 0
right[0] = min(0, 5) = 0
```

```text
right = [0, 0, 1, 4]
```

Sweep forward, tracking running max `left`:

```text
i=0: left = max(0(init), 5) = 5 → score = 5 - right[0] = 5 - 0 = 5 → 5 > 3
i=1: left = max(5, 0) = 5 → score = 5 - right[1] = 5 - 0 = 5 → 5 > 3
i=2: left = max(5, 1) = 5 → score = 5 - right[2] = 5 - 1 = 4 → 4 > 3
i=3: left = max(5, 4) = 5 → score = 5 - right[3] = 5 - 4 = 1 → 1 <= 3 → return 3
```

Matches the brute-force result.

## Algorithm

1. Build `right` array of size `n`, with `right[n-1] = nums[n-1]`.
2. For `i` from `n-2` down to `0`: `right[i] = min(right[i+1], nums[i])`.
3. Initialize `left = 0` (or negative infinity, since values will always overwrite it on the first comparison).
4. For each index `i` from `0` to `n-1`:

   * `left = max(left, nums[i])`.
   * If `left - right[i] <= k`, return `i`.
5. If no index qualifies, return `-1`.

## Complexity

* **Time:** `O(n)`

  * One backward pass to build `right`, one forward pass to check each index — both linear.
* **Space:** `O(n)`

  * For the `right` (suffix min) array.

## Notes / Tips

* The suffix min array must be built first (back to front) since `right[i]` depends on `right[i+1]` — this is the mirror image of a standard prefix computation.
* `left` only ever needs a single running variable during the forward sweep since it's monotonically non-decreasing — no need to store a full prefix-max array, unlike `right` which is needed at every index ahead of time.
* Same "precompute one direction, sweep the other while combining" shape as problems like Trapping Rain Water (LC 42) — anytime a value at index `i` depends on both "everything before" and "everything after," this two-pass pattern applies.

## Code

```cpp
class Solution {
public:
    int smallestStableIndex(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> right(n);
        right[n - 1] = nums[n - 1];

        for (int i = n - 2; i >= 0; i--) {
            right[i] = min(right[i + 1], nums[i]);
        }

        int left = INT_MIN;
        for (int i = 0; i < n; i++) {
            left = max(left, nums[i]);
            if (left - right[i] <= k) {
                return i;
            }
        }

        return -1;
    }
};
```

---

## Key Template

```text
right[n-1] = nums[n-1]
for i from n-2 down to 0:
    right[i] = min(right[i+1], nums[i])

left = -infinity
for i from 0 to n-1:
    left = max(left, nums[i])
    if left - right[i] <= k:
        return i

return -1
```