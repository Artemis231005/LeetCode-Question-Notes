# LeetCode 560 — Subarray Sum Equals K

## Metadata

* **LeetCode:** 560
* **Problem:** Subarray Sum Equals K
* **Difficulty:** Medium
* **Topics:** Array, Hash Map, Prefix Sum
* **Pattern:** Prefix Sum + Hash Map Frequency Count
* **Key Technique:** Count how many earlier prefix sums equal `currentPrefixSum - k`, using a hash map for O(1) lookups
* **Optimal Complexity:** `O(n)` Time, `O(n)` Space

---

## Problem Statement

Given an array `nums` and an integer `k`, return the total number of contiguous subarrays whose sum equals `k`.

---

## Approaches

1. **Brute Force — Check Every Subarray**
2. **Better — Prefix Sum Array + Nested Check**
3. **Optimal — Prefix Sum + Hash Map**

---

# Approach 1 — Brute Force / Check Every Subarray

## Idea

Generate every possible subarray by fixing a start index and extending the end index, summing as you go, and counting whenever the running sum equals `k`.

## Dry Run

```text
nums = [1, 2, 3], k = 3
```

Start `i = 0`:

```text
[1] = 1 → no
[1,2] = 3 → count = 1
[1,2,3] = 6 → no
```

Start `i = 1`:

```text
[2] = 2 → no
[2,3] = 5 → no
```

Start `i = 2`:

```text
[3] = 3 → count = 2
```

Final count: `2`.

## Algorithm

1. Initialize `count = 0`.
2. For each start index `i` from `0` to `n-1`:

   * Initialize `sum = 0`.
   * For each end index `j` from `i` to `n-1`:

     * `sum += nums[j]`.
     * If `sum == k`, increment `count`.
3. Return `count`.

## Complexity

* **Time:** `O(n²)`

  * Every pair of `(start, end)` indices is checked, and the sum is accumulated incrementally within the inner loop rather than recomputed from scratch.
* **Space:** `O(1)`

  * Only a running sum and counter are tracked — no extra structures allocated.

## Notes / Tips

* Already avoids full recomputation of each subarray sum by accumulating incrementally in the inner loop, but still checks every possible subarray explicitly.
* Fine for small inputs, but doesn't scale — full recomputation from scratch for every subarray (summing `nums[i..j]` independently each time) would be `O(n³)`, which is worth knowing exists as an even worse baseline.

## Code

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        int count = 0;

        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = i; j < n; j++) {
                sum += nums[j];
                if (sum == k) {
                    count++;
                }
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Better / Prefix Sum Array + Nested Check

## Idea

Precompute the prefix sum array once. Any subarray sum `nums[i..j]` is then `prefix[j+1] - prefix[i]`. Check every pair `(i, j)` using this O(1) lookup instead of accumulating sums manually in a nested loop.

## Dry Run

```text
nums = [1, 2, 3], k = 3
```

Prefix sums (`prefix[0] = 0`):

```text
prefix = [0, 1, 3, 6]
```

Check all pairs `(i, j)` where `i < j`:

```text
prefix[1]-prefix[0] = 1-0 = 1 → no
prefix[2]-prefix[0] = 3-0 = 3 → count = 1
prefix[3]-prefix[0] = 6-0 = 6 → no
prefix[2]-prefix[1] = 3-1 = 2 → no
prefix[3]-prefix[1] = 6-1 = 5 → no
prefix[3]-prefix[2] = 6-3 = 3 → count = 2
```

Final count: `2`.

## Algorithm

1. Build `prefix` array of size `n + 1`, with `prefix[0] = 0` and `prefix[i+1] = prefix[i] + nums[i]`.
2. For each pair `i < j` (0-indexed into `prefix`, representing subarray `nums[i..j-1]`):

   * If `prefix[j] - prefix[i] == k`, increment `count`.
3. Return `count`.

## Complexity

* **Time:** `O(n²)`

  * Building the prefix array is `O(n)`, but checking every pair of prefix indices is still `O(n²)`.
* **Space:** `O(n)`

  * For the `prefix` array.

## Notes / Tips

* Still quadratic overall — precomputing prefix sums only removes the cost of summing each subarray, not the cost of checking every pair.
* Useful stepping stone toward Approach 3: this is exactly where a hash map replaces the nested pair-checking loop.

## Code

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> prefix(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        int count = 0;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j <= n; j++) {
                if (prefix[j] - prefix[i] == k) {
                    count++;
                }
            }
        }

        return count;
    }
};
```

---

# Approach 3 — Optimal / Prefix Sum + Hash Map

## Idea

A subarray `nums[i..j]` sums to `k` exactly when `prefix[j] - prefix[i] = k`, i.e. `prefix[i] = prefix[j] - k`. Instead of checking every earlier prefix sum explicitly, keep a hash map counting how many times each prefix sum value has occurred so far. At each step, look up how many earlier prefix sums equal `currentSum - k` — that count is exactly the number of valid subarrays ending at the current index.

## Dry Run

```text
nums = [1, 2, 3], k = 3
```

Initialize map with `{0: 1}` (empty prefix), `sum = 0`, `count = 0`.

```text
i=0, num=1: sum = 1
   need sum - k = 1 - 3 = -2 → map has 0 of these
   map[1] = 1 → map = {0:1, 1:1}

i=1, num=2: sum = 3
   need sum - k = 3 - 3 = 0 → map has 1 of these → count += 1 → count = 1
   map[3] = 1 → map = {0:1, 1:1, 3:1}

i=2, num=3: sum = 6
   need sum - k = 6 - 3 = 3 → map has 1 of these → count += 1 → count = 2
   map[6] = 1 → map = {0:1, 1:1, 3:1, 6:1}
```

Final count: `2`.

## Algorithm

1. Initialize a hash map `prefixCount` with `{0: 1}` (accounts for a subarray starting at index `0`).
2. Initialize `sum = 0` and `count = 0`.
3. For each value in `nums`:

   * `sum += num`.
   * If `prefixCount` contains `sum - k`, add its frequency to `count`.
   * Increment `prefixCount[sum]` by `1`.
4. Return `count`.

## Complexity

* **Time:** `O(n)`

  * Single pass, each hash map lookup and insert is `O(1)` on average.
* **Space:** `O(n)`

  * For the hash map, which can hold up to `n` distinct prefix sum values.

## Notes / Tips

* The `{0: 1}` seed value is essential — it accounts for subarrays that start from index `0` and sum exactly to `k`, which would otherwise be missed since there's no "previous" prefix sum before the array starts.
* Works correctly with negative numbers, since prefix sums aren't assumed to be monotonic — this is exactly why a two-pointer/sliding-window approach doesn't work here (unlike problems restricted to positive numbers).
* This is the standard "prefix sum + hash map" template — same shape solves problems like "Continuous Subarray Sum" (LC 523) and "Subarray Sums Divisible by K" (LC 974), just with a different key derived from the prefix sum.

## Code

```cpp
class Solution {
public:
    int subarraySum(vector<int>& nums, int k) {
        unordered_map<int, int> prefixCount;
        prefixCount[0] = 1;

        int sum = 0, count = 0;

        for (int num : nums) {
            sum += num;

            if (prefixCount.find(sum - k) != prefixCount.end()) {
                count += prefixCount[sum - k];
            }

            prefixCount[sum]++;
        }

        return count;
    }
};
```

---

## Key Template

```text
prefixCount = {0: 1}
sum = 0
count = 0

for num in nums:
    sum += num
    count += prefixCount.get(sum - k, 0)
    prefixCount[sum] += 1

return count
```