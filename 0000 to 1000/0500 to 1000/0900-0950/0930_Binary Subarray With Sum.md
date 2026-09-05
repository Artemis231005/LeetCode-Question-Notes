# LeetCode 930 — Binary Subarrays With Sum

## Metadata

* **LeetCode:** 930
* **Problem:** Binary Subarrays With Sum
* **Difficulty:** Medium
* **Topics:** Array, Hash Map, Sliding Window, Prefix Sum
* **Pattern:** Prefix Sum + Hash Map (or Sliding Window "At Most" Trick)
* **Key Technique:** Count subarrays with sum exactly `goal` as `atMost(goal) - atMost(goal - 1)`, using a two-pointer window since all values are non-negative
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given a binary array `nums` and an integer `goal`, return the number of non-empty subarrays whose sum equals `goal`.

---

## Approaches

1. **Brute Force — Check Every Subarray**
2. **Better — Prefix Sum + Hash Map**
3. **Optimal — Sliding Window "At Most" Trick**

---

# Approach 1 — Brute Force / Check Every Subarray

## Idea

For every possible subarray, accumulate its sum incrementally and check whether it equals `goal`.

## Dry Run

```text
nums = [1, 0, 1, 0, 1], goal = 2
```

Start `i = 0`:

```text
[1] = 1 → no
[1,0] = 1 → no
[1,0,1] = 2 → match! count = 1
[1,0,1,0] = 2 → match! count = 2
[1,0,1,0,1] = 3 → no
```

Continue for every other start index the same way to reach the full count.

## Algorithm

1. Initialize `count = 0`.
2. For each start index `i` from `0` to `n-1`:

   * Initialize `sum = 0`.
   * For each end index `j` from `i` to `n-1`:

     * `sum += nums[j]`.
     * If `sum == goal`, increment `count`.
3. Return `count`.

## Complexity

* **Time:** `O(n²)`

  * Every pair of `(start, end)` indices is checked, with the sum accumulated incrementally in the inner loop rather than recomputed from scratch.
* **Space:** `O(1)`

  * Only a running sum and counter are tracked — no extra structures allocated.

## Notes / Tips

* Correct but doesn't scale — same redundant re-derivation issue seen across all "count subarrays matching X" problems (LC 560, LC 974).
* Since all values are `0` or `1`, sums only ever increase or stay flat as the window extends — a hint toward the sliding window approach.

## Code

```cpp
class Solution {
public:
    int numSubarraysWithSum(vector<int>& nums, int goal) {
        int n = nums.size();
        int count = 0;

        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = i; j < n; j++) {
                sum += nums[j];
                if (sum == goal) {
                    count++;
                }
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Better / Prefix Sum + Hash Map

## Idea

A subarray `nums[i..j]` sums to `goal` exactly when `prefix[j] - prefix[i] = goal`. Keep a hash map counting how many times each prefix sum value has occurred so far. At each step, look up how many earlier prefix sums equal `currentSum - goal` — that count is the number of valid subarrays ending at the current index.

## Dry Run

```text
nums = [1, 0, 1, 0, 1], goal = 2
```

Initialize `prefixCount = {0: 1}`, `sum = 0`, `count = 0`.

```text
i=0, num=1: sum=1, need 1-2=-1 → 0 found
   map[1] = 1

i=1, num=0: sum=1, need 1-2=-1 → 0 found
   map[1] = 2

i=2, num=1: sum=2, need 2-2=0 → 1 found → count = 1
   map[2] = 1

i=3, num=0: sum=2, need 2-2=0 → 1 found → count = 2
   map[2] = 2

i=4, num=1: sum=3, need 3-2=1 → 2 found → count = 4
   map[3] = 1
```

Final count: `4`.

## Algorithm

1. Initialize a hash map `prefixCount` with `{0: 1}`.
2. Initialize `sum = 0` and `count = 0`.
3. For each value in `nums`:

   * `sum += num`.
   * If `prefixCount` contains `sum - goal`, add its frequency to `count`.
   * Increment `prefixCount[sum]`.
4. Return `count`.

## Complexity

* **Time:** `O(n)`

  * Single pass, each hash map lookup and insert is `O(1)` on average.
* **Space:** `O(n)`

  * For the hash map, which can hold up to `n + 1` distinct prefix sum values.

## Notes / Tips

* Exactly the same template as LC 560 (Subarray Sum Equals K) — this problem is just that one restricted to a binary array.
* The `{0: 1}` seed handles subarrays starting at index `0` that already sum to `goal`.
* This approach works even if `nums` weren't binary — the binary constraint is what makes the sliding-window optimization in Approach 3 possible.

## Code

```cpp
class Solution {
public:
    int numSubarraysWithSum(vector<int>& nums, int goal) {
        unordered_map<int, int> prefixCount;
        prefixCount[0] = 1;

        int sum = 0, count = 0;

        for (int num : nums) {
            sum += num;

            if (prefixCount.find(sum - goal) != prefixCount.end()) {
                count += prefixCount[sum - goal];
            }

            prefixCount[sum]++;
        }

        return count;
    }
};
```

---

# Approach 3 — Optimal / Sliding Window "At Most" Trick

## Idea

Since every element is `0` or `1`, the running sum of a window only ever increases or stays the same as the window grows — this monotonic behavior is exactly what a sliding window needs. Define `atMost(k)` = number of subarrays with sum `<= k`, computable with a simple two-pointer window in `O(n)`. Then the count of subarrays with sum **exactly** `goal` is `atMost(goal) - atMost(goal - 1)`.

## Dry Run

```text
nums = [1, 0, 1, 0, 1], goal = 2
```

### Compute atMost(2)

Expand window, shrink from the left whenever `sum > 2`:

```text
right=0: [1] sum=1 → window size 1 → add 1 (count=1)
right=1: [1,0] sum=1 → window size 2 → add 2 (count=3)
right=2: [1,0,1] sum=2 → window size 3 → add 3 (count=6)
right=3: [1,0,1,0] sum=2 → window size 4 → add 4 (count=10)
right=4: [1,0,1,0,1] sum=3 → shrink left until sum<=2
         remove nums[0]=1 → sum=2, left=1 → window size 4 → add 4 (count=14)
```

`atMost(2) = 14`

### Compute atMost(1)

```text
right=0: [1] sum=1 → size 1 → add 1 (count=1)
right=1: [1,0] sum=1 → size 2 → add 2 (count=3)
right=2: [1,0,1] sum=2 → shrink until sum<=1
         remove nums[0]=1 → sum=1, left=1 → size 2 → add 2 (count=5)
right=3: [0,1,0] sum=1 → size 3 → add 3 (count=8)
right=4: [0,1,0,1] sum=2 → shrink until sum<=1
         remove nums[1]=1 → sum=1, left=2 → size 3 → add 3 (count=11)
```

`atMost(1) = 11`

### Final Answer

```text
atMost(2) - atMost(1) = 14 - 11 = 4
```

Matches the brute-force result.

## Algorithm

1. Define a helper `atMost(k)`:

   * If `k < 0`, return `0` (no valid subarrays possible).
   * Use two pointers `left = 0`, running `sum = 0`, and `count = 0`.
   * For each `right` from `0` to `n-1`:

     * `sum += nums[right]`.
     * While `sum > k`, subtract `nums[left]` from `sum` and increment `left`.
     * Add `right - left + 1` to `count` (every subarray ending at `right` starting from `left` to `right` has sum `<= k`).
   * Return `count`.
2. Return `atMost(goal) - atMost(goal - 1)`.

## Complexity

* **Time:** `O(n)`

  * Each call to `atMost` runs a standard sliding window where `left` and `right` each move forward at most `n` times total, and `atMost` is called twice.
* **Space:** `O(1)`

  * Only a few pointers and running totals — no hash map or extra array needed.

## Notes / Tips

* This "at most k, then subtract" trick only works because all elements are non-negative — a negative value could shrink the sum after growing the window, breaking the monotonic assumption the sliding window depends on.
* `right - left + 1` inside the loop is what accounts for **all** subarrays ending at `right`, not just one — this is the same counting trick used in "count subarrays with sum at most k" style problems.
* Common mistake: forgetting the `k < 0` guard in `atMost`, which matters when `goal = 0` (since `atMost(-1)` must return `0`, not error or miscount).

## Code

```cpp
class Solution {
public:
    int atMost(vector<int>& nums, int k) {
        if (k < 0) return 0;

        int left = 0, sum = 0, count = 0;

        for (int right = 0; right < nums.size(); right++) {
            sum += nums[right];

            while (sum > k) {
                sum -= nums[left];
                left++;
            }

            count += right - left + 1;
        }

        return count;
    }

    int numSubarraysWithSum(vector<int>& nums, int goal) {
        return atMost(nums, goal) - atMost(nums, goal - 1);
    }
};
```

---

## Key Template

```text
function atMost(nums, k):
    if k < 0: return 0
    left = 0, sum = 0, count = 0

    for right in 0..n-1:
        sum += nums[right]
        while sum > k:
            sum -= nums[left]
            left += 1
        count += right - left + 1

    return count

numSubarraysWithSum(nums, goal) = atMost(nums, goal) - atMost(nums, goal - 1)
```