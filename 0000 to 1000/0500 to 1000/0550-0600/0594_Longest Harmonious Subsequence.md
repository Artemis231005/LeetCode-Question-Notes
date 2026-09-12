# LeetCode 594 — Longest Harmonious Subsequence

## Metadata

* **LeetCode:** 594
* **Problem:** Longest Harmonious Subsequence
* **Difficulty:** Easy
* **Topics:** Array, Hash Table, Sorting
* **Pattern:** Frequency Counting + Adjacent Value Pairing
* **Key Technique:** Count occurrences of every value, then for each distinct value check the combined count with its value+1 neighbor — no need to look at any other value
* **Optimal Complexity:** `O(n)` Time, `O(n)` Space

---

## Problem Statement

Given an integer array `nums`, return the length of the longest subsequence where the difference between the maximum and minimum values is exactly `1`. A subsequence doesn't need to be contiguous, but elements must keep their relative order (though for this problem, only counts of values matter, not their positions).

---

## Approaches

1. **Brute Force — Check Every Pair of Values**
2. **Optimal — Frequency Map + Adjacent Value Lookup**

---

# Approach 1 — Brute Force / Check Every Pair of Values

## Idea

For every pair of elements in the array, check if their values differ by exactly `1`. If so, count how many elements in the entire array equal either of these two values — that count is a candidate harmonious subsequence length.

## Dry Run

```text
nums = [1, 3, 2, 2, 5, 2, 3, 7]
```

Check pair `(nums[0]=1, nums[2]=2)`:

```text
|1 - 2| = 1 → valid pair
count elements equal to 1 or 2: 1(x1) + 2(x3) = 4
```

Check pair `(nums[1]=3, nums[2]=2)`:

```text
|3 - 2| = 1 → valid pair
count elements equal to 2 or 3: 2(x3) + 3(x2) = 5
```

Continue checking all other pairs — the best found is `5`.

## Algorithm

1. Initialize `maxLen = 0`.
2. For each pair `(i, j)` with `i != j`:

   * If `abs(nums[i] - nums[j]) == 1`:

     * Count how many elements in `nums` equal `nums[i]` or `nums[j]`.
     * Update `maxLen = max(maxLen, count)`.
3. Return `maxLen`.

## Complexity

* **Time:** `O(n³)`

  * `O(n²)` pairs to check, and counting matches for each valid pair takes another `O(n)` scan.
* **Space:** `O(1)`

  * Only a running max and counters — no extra structures allocated.

## Notes / Tips

* Massively redundant — the same value pair like `(2, 3)` gets rechecked and recounted many times across different index pairs that happen to have those same two values.
* Precomputing how many times each value appears (a single frequency pass) removes almost all of this repeated work.

## Code

```cpp
class Solution {
public:
    int findLHS(vector<int>& nums) {
        int n = nums.size();
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                if (i != j && abs(nums[i] - nums[j]) == 1) {
                    int count = 0;
                    for (int num : nums) {
                        if (num == nums[i] || num == nums[j]) {
                            count++;
                        }
                    }
                    maxLen = max(maxLen, count);
                }
            }
        }

        return maxLen;
    }
};
```

---

# Approach 2 — Optimal / Frequency Map + Adjacent Value Lookup

## Idea

Count how many times each distinct value appears using a hash map. Then, for every distinct value `v` in the map, check if `v + 1` also exists in the map — if so, the combined count `freq[v] + freq[v+1]` is a valid harmonious subsequence length (since every element equal to `v` or `v+1` can be included, and their difference is exactly `1`). Track the maximum such combined count across all values.

## Dry Run

```text
nums = [1, 3, 2, 2, 5, 2, 3, 7]
```

Build frequency map:

```text
1: 1, 3: 2, 2: 3, 5: 1, 7: 1
```

Check each value against `value + 1`:

```text
v=1: check 2 in map → freq[1]+freq[2] = 1+3 = 4
v=3: check 4 in map → not present → skip
v=2: check 3 in map → freq[2]+freq[3] = 3+2 = 5
v=5: check 6 in map → not present → skip
v=7: check 8 in map → not present → skip
```

Best combined count: `5`.

## Algorithm

1. Build a frequency map `freq` counting occurrences of every value in `nums`.
2. Initialize `maxLen = 0`.
3. For each distinct value `v` in `freq`:

   * If `v + 1` also exists in `freq`, update `maxLen = max(maxLen, freq[v] + freq[v+1])`.
4. Return `maxLen`.

## Complexity

* **Time:** `O(n)`

  * One pass to build the frequency map, one pass over the distinct values (at most `n` of them) to check each `v + 1` pair.
* **Space:** `O(n)`

  * For the frequency map, which can hold up to `n` distinct values.

## Notes / Tips

* Only checking `v + 1` (not `v - 1` as well) is sufficient and avoids double-counting the same pair — every valid pair `(v, v+1)` gets checked exactly once starting from the smaller value.
* This is a classic "precompute counts, then look up a fixed neighbor" pattern — since the harmonious condition is a difference of exactly `1`, only two very specific values ever matter for any given `v`, making a hash map lookup ideal instead of scanning.
* A subsequence's *positions* don't actually matter here, only which values (and how many of each) are included — this is what makes pure frequency counting sufficient, unlike subsequence problems that also depend on relative order.

## Code

```cpp
class Solution {
public:
    int findLHS(vector<int>& nums) {
        unordered_map<int, int> freq;
        for (int num : nums) {
            freq[num]++;
        }

        int maxLen = 0;
        for (auto& [value, count] : freq) {
            if (freq.find(value + 1) != freq.end()) {
                maxLen = max(maxLen, count + freq[value + 1]);
            }
        }

        return maxLen;
    }
};
```

---

## Key Template

```text
freq = {}
for num in nums:
    freq[num] += 1

maxLen = 0
for value, count in freq.items():
    if (value + 1) in freq:
        maxLen = max(maxLen, count + freq[value + 1])

return maxLen
```