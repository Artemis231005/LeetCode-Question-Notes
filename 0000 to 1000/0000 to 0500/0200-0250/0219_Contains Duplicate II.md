# LeetCode 219 — Contains Duplicate II

## Metadata

* **LeetCode:** 219
* **Problem:** Contains Duplicate II
* **Difficulty:** Easy
* **Topics:** Array, Hash Table, Sliding Window
* **Pattern:** Sliding Window (Fixed Size) via Hash Map of Last Seen Index
* **Key Technique:** Track the most recent index each value was seen at, and check if a repeat occurs within `k` positions — no need to scan backward
* **Optimal Complexity:** `O(n)` Time, `O(min(n, k))` Space

---

## Problem Statement

Given an integer array `nums` and an integer `k`, return `true` if there are two distinct indices `i` and `j` such that `nums[i] == nums[j]` and `abs(i - j) <= k`.

---

## Approaches

1. **Brute Force — Check Every Pair of Indices**
2. **Optimal — Hash Map of Last Seen Index**

---

# Approach 1 — Brute Force / Check Every Pair of Indices

## Idea

For every pair of indices `(i, j)` with `i < j`, check whether `nums[i] == nums[j]` and whether `j - i <= k`.

## Dry Run

```text
nums = [1, 2, 3, 1], k = 3
```

Check `(0, 3)`:

```text
nums[0]=1, nums[3]=1 → equal
j - i = 3 <= k=3 → match found → return true
```

## Algorithm

1. For each `i` from `0` to `n-1`:
2. For each `j` from `i+1` to `n-1`:

   * If `nums[i] == nums[j]` and `j - i <= k`, return `true`.
3. If no such pair is found, return `false`.

## Complexity

* **Time:** `O(n²)`

  * Every pair of indices is checked directly.
* **Space:** `O(1)`

  * Only loop counters are used — no extra structures allocated.

## Notes / Tips

* Checking every pair ignores the fact that only pairs within a fixed distance `k` ever matter — most of the pairs checked here are already too far apart to ever satisfy the condition, wasting comparisons.
* A smarter approach only needs to remember "have I seen this value recently," which a hash map handles directly.

## Code

```cpp
class Solution {
public:
    bool containsNearbyDuplicate(vector<int>& nums, int k) {
        int n = nums.size();

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (nums[i] == nums[j] && j - i <= k) {
                    return true;
                }
            }
        }

        return false;
    }
};
```

---

# Approach 2 — Optimal / Hash Map of Last Seen Index

## Idea

Scan through the array once, keeping a hash map from value to the index it was last seen at. For each element, check if it's already in the map — if so, the gap between the current index and that last seen index is exactly the distance to check. If that gap is `<= k`, a valid pair is found. Otherwise, update the map with the current index regardless (so future checks compare against the closest previous occurrence).

## Dry Run

```text
nums = [1, 2, 3, 1], k = 3
```

Process:

```text
i=0, num=1: not in map → map = {1:0}
i=1, num=2: not in map → map = {1:0, 2:1}
i=2, num=3: not in map → map = {1:0, 2:1, 3:2}
i=3, num=1: in map at index 0 → gap = 3-0 = 3 <= k=3 → return true
```

## Algorithm

1. Initialize an empty hash map `lastSeen`.
2. For each index `i` and value `nums[i]`:

   * If `nums[i]` is in `lastSeen` and `i - lastSeen[nums[i]] <= k`, return `true`.
   * Update `lastSeen[nums[i]] = i` (always store the most recent index).
3. If the loop completes without finding a match, return `false`.

## Complexity

* **Time:** `O(n)`

  * Single pass, each hash map lookup and update is `O(1)` on average.
* **Space:** `O(min(n, k))`

  * The map only ever needs to hold values whose most recent occurrence is within the last `k` indices to be useful for future checks; in the worst case with many distinct values it holds up to `min(n, k+1)` entries meaningfully, though a plain hash map may grow up to `O(n)` if not pruned — either way it's bounded by the number of distinct values seen so far, capped by `n`.

## Notes / Tips

* Always overwriting `lastSeen[nums[i]] = i` (even after a match is checked but not found) is important — this keeps the stored index as close as possible to the current position, which maximizes the chance of a future match falling within `k`.
* This is effectively a fixed-size sliding window expressed through a hash map instead of an explicit window structure — only the most recent occurrence within the last `k` positions ever matters, older ones can be safely forgotten (or just overwritten, since a closer occurrence is strictly better for satisfying the `<= k` condition).
* Same "map value to last seen index" idea generalizes directly to variations like "contains duplicate within a value difference `t`" (LC 220), which adds a value-closeness condition on top of this index-closeness check.

## Code

```cpp
class Solution {
public:
    bool containsNearbyDuplicate(vector<int>& nums, int k) {
        unordered_map<int, int> lastSeen;

        for (int i = 0; i < nums.size(); i++) {
            if (lastSeen.find(nums[i]) != lastSeen.end() && i - lastSeen[nums[i]] <= k) {
                return true;
            }

            lastSeen[nums[i]] = i;
        }

        return false;
    }
};
```

---

## Key Template

```text
lastSeen = {}

for i, num in enumerate(nums):
    if num in lastSeen and i - lastSeen[num] <= k:
        return true

    lastSeen[num] = i

return false
```