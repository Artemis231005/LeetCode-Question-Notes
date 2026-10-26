# LeetCode 1984 — Minimum Difference Between Highest and Lowest of K Scores

## Metadata

* **LeetCode:** 1984
* **Problem:** Minimum Difference Between Highest and Lowest of K Scores
* **Difficulty:** Easy
* **Topics:** Array, Sliding Window, Sorting
* **Pattern:** Sort + Fixed-Size Sliding Window
* **Key Technique:** After sorting, the best group of `k` scores is always `k` consecutive elements — within any such window, the difference between highest and lowest is just `nums[i + k - 1] - nums[i]`, so only adjacent windows need checking
* **Optimal Complexity:** `O(n log n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `nums` of student scores and an integer `k`, pick the scores of any `k` students such that the difference between the highest and lowest of those `k` scores is minimized. Return that minimum difference.

---

## Approaches

1. **Brute Force — Check Every Combination of k Scores**
2. **Optimal — Sort + Sliding Window of Size k**

---

# Approach 1 — Brute Force / Check Every Combination of k Scores

## Idea

Try every possible way to choose `k` scores out of `n`, compute the highest and lowest in each chosen group, and track the smallest difference found.

## Dry Run

```text
nums = [9, 4, 1, 7], k = 2
```

Check every pair:

```text
(9,4): max-min = 5
(9,1): 8
(9,7): 2
(4,1): 3
(4,7): 3
(1,7): 6
```

Minimum difference: `2`.

## Algorithm

1. Initialize `best = infinity`.
2. For every combination of `k` indices from `nums`:

   * Compute `max - min` among the chosen values.
   * Update `best = min(best, max - min)`.
3. Return `best`.

## Complexity

* **Time:** `O(C(n, k) * k)`

  * There are `C(n, k)` possible groups, each requiring `O(k)` to find its max and min.
* **Space:** `O(k)`

  * For the recursion/current-combination tracking.

## Notes / Tips

* Enumerating every combination is extremely wasteful here — most chosen groups are obviously suboptimal, since the best group of `k` scores always consists of values that are close together in sorted order.
* This is mainly useful to confirm correctness on tiny inputs before applying the sorting insight in Approach 2.

## Code

```cpp
class Solution {
public:
    int best = INT_MAX;

    void choose(vector<int>& nums, int k, int start, vector<int>& current) {
        if ((int)current.size() == k) {
            int mx = *max_element(current.begin(), current.end());
            int mn = *min_element(current.begin(), current.end());
            best = min(best, mx - mn);
            return;
        }

        for (int i = start; i < (int)nums.size(); i++) {
            current.push_back(nums[i]);
            choose(nums, k, i + 1, current);
            current.pop_back();
        }
    }

    int minimumDifference(vector<int>& nums, int k) {
        vector<int> current;
        choose(nums, k, 0, current);
        return best;
    }
};
```

---

# Approach 2 — Optimal / Sort + Sliding Window of Size k

## Idea

Sort the array first. Once sorted, any group of `k` scores with the smallest possible spread must be `k` **consecutive** elements — if a group skipped over an element in between, swapping in that skipped element could only keep the spread the same or shrink it. So only windows of `k` adjacent sorted elements need checking, and within each such window, the highest is `nums[i + k - 1]` and the lowest is `nums[i]` — their difference is computed in `O(1)`.

## Dry Run

```text
nums = [9, 4, 1, 7], k = 2
```

Sort: `[1, 4, 7, 9]`.

Slide a window of size `2`:

```text
i=0: nums[1] - nums[0] = 4 - 1 = 3
i=1: nums[2] - nums[1] = 7 - 4 = 3
i=2: nums[3] - nums[2] = 9 - 7 = 2
```

Minimum difference: `2`, matching the brute-force result.

## Algorithm

1. If `k == 1`, return `0` (a single score has no spread).
2. Sort `nums` in ascending order.
3. Initialize `best = infinity`.
4. For each `i` from `0` to `n - k`:

   * `best = min(best, nums[i + k - 1] - nums[i])`.
5. Return `best`.

## Complexity

* **Time:** `O(n log n)`

  * Dominated by sorting; the sliding window scan afterward is `O(n)`.
* **Space:** `O(1)` auxiliary (beyond the sort's own space)

  * Only a running minimum — no extra structures.

## Notes / Tips

* The key insight is that sorting turns "choose any `k` of `n` elements to minimize spread" into "check only windows of `k` adjacent elements" — spread is determined entirely by the smallest and largest chosen value, and the tightest possible group of `k` values will always sit contiguously in sorted order.
* The `k == 1` early return handles the edge case where the window has only one element — the formula `nums[i + k - 1] - nums[i]` would still correctly give `0` here (since `i + 0 - i = 0`), but returning early makes the intent explicit.
* This "sort, then slide a fixed-size window" pattern appears whenever a problem asks to pick a subset minimizing a range-based quantity (max minus min) — same idea underlies problems like "Minimum Difference Between Largest and Smallest Value in Three Moves" (LC 1509).

## Code

```cpp
class Solution {
public:
    int minimumDifference(vector<int>& nums, int k) {
        if (k == 1) {
            return 0;
        }

        sort(nums.begin(), nums.end());

        int best = INT_MAX;
        for (int i = 0; i + k - 1 < (int)nums.size(); i++) {
            best = min(best, nums[i + k - 1] - nums[i]);
        }

        return best;
    }
};
```

---

## Key Template

```text
if k == 1: return 0

sort(nums)
best = infinity

for i in 0..n-k:
    best = min(best, nums[i + k - 1] - nums[i])

return best
```