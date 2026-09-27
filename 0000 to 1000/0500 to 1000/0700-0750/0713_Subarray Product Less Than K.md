# LeetCode 713 — Subarray Product Less Than K

## Metadata

* **LeetCode:** 713
* **Problem:** Subarray Product Less Than K
* **Difficulty:** Medium
* **Topics:** Array, Sliding Window
* **Pattern:** Sliding Window (Variable Size)
* **Key Technique:** Since all values are positive, a window's product only ever grows as it expands right and shrinks as it contracts left — this monotonic behavior lets a two-pointer window count all valid subarrays in one pass, using `right - left + 1` to count every subarray ending at the current position at once
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array of positive integers `nums` and an integer `k`, return the number of contiguous subarrays where the product of all elements is strictly less than `k`.

---

## Approaches

1. **Brute Force — Check Every Subarray**
2. **Optimal — Sliding Window**

---

# Approach 1 — Brute Force / Check Every Subarray

## Idea

For every possible subarray, accumulate its product incrementally and check whether it's strictly less than `k`.

## Dry Run

```text
nums = [10, 5, 2, 6], k = 100
```

Start `i = 0`:

```text
[10] = 10 → < 100 → count=1
[10,5] = 50 → < 100 → count=2
[10,5,2] = 100 → not < 100 → stop extending from i=0 (though brute force can still check further starts)
[10,5,2,6] = 600 → not < 100
```

Continue for every other start index the same way, accumulating the product incrementally in the inner loop.

Final count: `8`, matching the expected output.

## Algorithm

1. Initialize `count = 0`.
2. For each start index `i` from `0` to `n-1`:

   * Initialize `product = 1`.
   * For each end index `j` from `i` to `n-1`:

     * `product *= nums[j]`.
     * If `product < k`, increment `count`; otherwise, break early (since all further extensions from this `i` will only make the product larger, given all positive values).
3. Return `count`.

## Complexity

* **Time:** `O(n²)`

  * Every pair of `(start, end)` indices is checked, with the product accumulated incrementally in the inner loop rather than recomputed from scratch.
* **Space:** `O(1)`

  * Only a running product and counter are tracked — no extra structures allocated.

## Notes / Tips

* The early `break` (once a subarray's product reaches or exceeds `k`) is a valid optimization even within the brute force, since all positive values guarantee the product can only grow from there — but the overall approach is still quadratic since a fresh starting product is computed for every `i`.
* Recognizing that products only ever grow as the window extends (because every value is positive) is exactly the property that makes the sliding window approach valid.

## Code

```cpp
class Solution {
public:
    int numSubarrayProductLessThanK(vector<int>& nums, int k) {
        int n = nums.size();
        int count = 0;

        for (int i = 0; i < n; i++) {
            long long product = 1;
            for (int j = i; j < n; j++) {
                product *= nums[j];
                if (product < k) {
                    count++;
                } else {
                    break;
                }
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Optimal / Sliding Window

## Idea

Maintain a window `[left, right]` with a running product. Expand `right` one step at a time, multiplying the new value in. Whenever the product becomes `>= k`, shrink from the left (dividing out `nums[left]`) until the product is valid again. At each step where the window is valid, every subarray ending at `right` and starting anywhere from `left` to `right` has a product `< k` — that's exactly `right - left + 1` new valid subarrays, added directly to the count without enumerating them individually.

## Dry Run

```text
nums = [10, 5, 2, 6], k = 100
```

`left = 0`, `product = 1`, `count = 0`.

```text
right=0: product *= 10 → 10. 10 < 100 → ok.
   count += (0-0+1) = 1 → count=1

right=1: product *= 5 → 50. 50 < 100 → ok.
   count += (1-0+1) = 2 → count=3

right=2: product *= 2 → 100. 100 not < 100 → shrink:
   product /= nums[0]=10 → 10, left=1
   10 < 100 → ok now
   count += (2-1+1) = 2 → count=5

right=3: product *= 6 → 60. 60 < 100 → ok.
   count += (3-1+1) = 3 → count=8
```

Final count: `8`, matching the brute-force result.

## Algorithm

1. If `k <= 1`, return `0` immediately (no product of positive integers can ever be less than `1`, given the problem's constraints).
2. Initialize `left = 0`, `product = 1`, `count = 0`.
3. For each `right` from `0` to `n-1`:

   * `product *= nums[right]`.
   * While `product >= k`: `product /= nums[left]`, increment `left`.
   * `count += right - left + 1`.
4. Return `count`.

## Complexity

* **Time:** `O(n)`

  * `left` and `right` each move forward at most `n` times total across the whole array, giving amortized linear time despite the nested-looking `while` loop.
* **Space:** `O(1)`

  * Only a running product, two pointers, and a counter — no extra structures needed.

## Notes / Tips

* Adding `right - left + 1` at each step (rather than checking and counting each subarray individually) is the key trick that avoids an inner loop — since the window `[left, right]` is valid, every subarray `[x, right]` for `x` from `left` to `right` is guaranteed valid too (a shorter suffix of a valid-product window can only have an equal or smaller product, given all positive values), so they can all be counted at once.
* The `k <= 1` early return matters because with strictly positive integers, no subarray (not even a single element, since the smallest positive value is `1`) can ever have a product strictly less than `1` — without this guard, the main loop could behave unexpectedly if `k` is `0` or `1`.
* This is the standard "shrinkable sliding window" pattern for counting subarrays satisfying a monotonic condition — the same `right - left + 1` counting trick is used in similar sum-based sliding window problems (e.g. counting subarrays with sum less than a target, when all values are non-negative).

## Code

```cpp
class Solution {
public:
    int numSubarrayProductLessThanK(vector<int>& nums, int k) {
        if (k <= 1) {
            return 0;
        }

        int left = 0;
        long long product = 1;
        int count = 0;

        for (int right = 0; right < nums.size(); right++) {
            product *= nums[right];

            while (product >= k) {
                product /= nums[left];
                left++;
            }

            count += right - left + 1;
        }

        return count;
    }
};
```

---

## Key Template

```text
if k <= 1: return 0

left = 0
product = 1
count = 0

for right in 0..n-1:
    product *= nums[right]

    while product >= k:
        product /= nums[left]
        left += 1

    count += right - left + 1

return count
```