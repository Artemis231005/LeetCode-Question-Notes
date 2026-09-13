# LeetCode 523 — Continuous Subarray Sum

## Metadata

* **LeetCode:** 523
* **Problem:** Continuous Subarray Sum
* **Difficulty:** Medium
* **Topics:** Array, Hash Table, Math, Prefix Sum
* **Pattern:** Prefix Sum Remainders + Hash Map (Earliest Index Lookup)
* **Key Technique:** Two prefix sums with the same remainder mod `k` mean everything between them sums to a multiple of `k` — track only the **earliest** index each remainder was seen at, since that maximizes the subarray length for the length-`>=2` requirement
* **Optimal Complexity:** `O(n)` Time, `O(min(n, k))` Space

---

## Problem Statement

Given an integer array `nums` and an integer `k`, return `true` if the array has a contiguous subarray of size **at least 2** whose sum is a multiple of `k` (`0` is considered a multiple of every `k`, including when `k = 0`, i.e. the subarray itself must sum to exactly `0` in that case).

---

## Approaches

1. **Brute Force — Check Every Subarray of Length >= 2**
2. **Optimal — Prefix Sum Remainders + Earliest-Index Hash Map**

---

# Approach 1 — Brute Force / Check Every Subarray of Length >= 2

## Idea

For every possible subarray of length at least `2`, accumulate its sum incrementally and check whether it's a multiple of `k` (handling `k == 0` as a special case, since checking divisibility by zero isn't valid — in that case the sum itself must be exactly `0`).

## Dry Run

```text
nums = [23, 2, 4, 6, 7], k = 6
```

Start `i = 0`:

```text
[23,2] = 25 → 25 % 6 != 0
[23,2,4] = 29 → no
[23,2,4,6] = 35 → no
[23,2,4,6,7] = 42 → 42 % 6 == 0 → return true
```

## Algorithm

1. For each start index `i` from `0` to `n-2`:

   * Initialize `sum = nums[i]`.
   * For each end index `j` from `i+1` to `n-1`:

     * `sum += nums[j]`.
     * If `k == 0`: check `sum == 0`. Otherwise: check `sum % k == 0`.
     * If true, return `true`.
2. If no subarray qualifies, return `false`.

## Complexity

* **Time:** `O(n²)`

  * Every pair of `(start, end)` indices with length `>= 2` is checked, with the sum accumulated incrementally in the inner loop.
* **Space:** `O(1)`

  * Only a running sum and loop counters — no extra structures allocated.

## Notes / Tips

* Same redundant re-derivation issue as LC 560 and LC 974 — every subarray sum is recomputed from a fresh starting point instead of reusing prefix information.
* The `k == 0` edge case must be handled separately here since modulo by zero is undefined — this quirk carries over into the optimal approach as well.

## Code

```cpp
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        int n = nums.size();

        for (int i = 0; i < n - 1; i++) {
            long long sum = nums[i];
            for (int j = i + 1; j < n; j++) {
                sum += nums[j];

                if (k == 0) {
                    if (sum == 0) return true;
                } else {
                    if (sum % k == 0) return true;
                }
            }
        }

        return false;
    }
};
```

---

# Approach 2 — Optimal / Prefix Sum Remainders + Earliest-Index Hash Map

## Idea

A subarray `nums[i+1..j]` sums to a multiple of `k` exactly when `prefix[j] % k == prefix[i] % k` (using `k`'s absolute value where relevant, and normalizing negative remainders). Instead of comparing every pair, keep a hash map from **remainder** to the **earliest** index at which that remainder was seen. Whenever the current remainder repeats, the length of the subarray between that earliest occurrence and now is `currentIndex - earliestIndex` — if that gap is `>= 2`, a valid subarray has been found.

## Dry Run

```text
nums = [23, 2, 4, 6, 7], k = 6
```

Initialize `remainderIndex = {0: -1}` (remainder `0` occurs "before" index `0`), `sum = 0`.

```text
i=0, num=23: sum=23, rem=23%6=5 → not in map → remainderIndex[5]=0
i=1, num=2: sum=25, rem=25%6=1 → not in map → remainderIndex[1]=1
i=2, num=4: sum=29, rem=29%6=5 → already in map at index 0 → length = 2-0 = 2 >= 2 → return true
```

Matches the brute-force result (found even earlier here, at index `2` instead of needing the full array).

## Algorithm

1. Handle `k == 0` up front (or fold into the general logic using remainder `sum` itself when `k == 0`, treating "remainder" as the raw sum in that case).
2. Initialize a hash map `remainderIndex` with `{0: -1}`.
3. Initialize `sum = 0`.
4. For each index `i` and value `nums[i]`:

   * `sum += nums[i]`.
   * Compute `rem = k == 0 ? sum : ((sum % k) + k) % k` (normalize for negative sums when `k != 0`).
   * If `rem` exists in `remainderIndex`:

     * If `i - remainderIndex[rem] >= 2`, return `true`.
   * Otherwise, store `remainderIndex[rem] = i` (only the **first** occurrence is kept, since it maximizes any future subarray length).
5. If the loop completes without finding a valid subarray, return `false`.

## Complexity

* **Time:** `O(n)`

  * Single pass, each hash map lookup and insert is `O(1)` on average.
* **Space:** `O(min(n, k))`

  * The map holds at most `k` distinct remainders (when `k != 0`), or up to `n` distinct sums (when `k == 0`) — bounded by whichever is smaller in practice.

## Notes / Tips

* Only the **first** occurrence of each remainder should ever be stored as overwriting it with a later index would shrink the potential subarray length instead of maximizing the chance of hitting the `>= 2` length requirement.
* Seeding the map with `{0: -1}` is what correctly allows a subarray starting at index `0` to qualify, without it, a subarray like `nums[0..1]` summing to a multiple of `k` would be missed since there'd be no "earlier" remainder-`0` entry to compare against.
* The length check `i - remainderIndex[rem] >= 2` allows us to fulfill requirement a minimum length of `2`, and is exactly why the earliest index (not just any matching index) must be tracked and compared.

## Code

```cpp
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) {
        unordered_map<long long, int> remainderIndex;
        remainderIndex[0] = -1;

        long long sum = 0;

        for (int i = 0; i < nums.size(); i++) {
            sum += nums[i];

            long long rem = (k == 0) ? sum : ((sum % k) + k) % k;

            if (remainderIndex.find(rem) != remainderIndex.end()) {
                if (i - remainderIndex[rem] >= 2) {
                    return true;
                }
            } else {
                remainderIndex[rem] = i;
            }
        }

        return false;
    }
};
```

---

## Key Template

```text
remainderIndex = {0: -1}
sum = 0

for i, num in enumerate(nums):
    sum += num
    rem = sum if k == 0 else ((sum % k) + k) % k

    if rem in remainderIndex:
        if i - remainderIndex[rem] >= 2:
            return true
    else:
        remainderIndex[rem] = i

return false
```