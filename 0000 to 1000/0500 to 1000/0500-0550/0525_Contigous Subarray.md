# LeetCode 525 — Contiguous Array

## Metadata

* **LeetCode:** 525
* **Problem:** Contiguous Array
* **Difficulty:** Medium
* **Topics:** Array, Hash Map, Prefix Sum
* **Pattern:** Prefix Sum + Hash Map (First-Occurrence Index)
* **Key Technique:** Convert 0s to -1, then find the longest subarray whose running sum returns to a previously seen value
* **Optimal Complexity:** `O(n)` Time, `O(n)` Space

---

## Problem Statement

Given a binary array `nums`, return the length of the longest contiguous subarray containing an equal number of `0`s and `1`s.

---

## Approaches

1. **Brute Force — Check Every Subarray**
2. **Better — Prefix Count Arrays + Nested Check**
3. **Optimal — Prefix Sum with -1/+1 Mapping + Hash Map**

---

# Approach 1 — Brute Force / Check Every Subarray

## Idea

For every possible subarray, count how many `0`s and `1`s it contains. If the counts are equal, check if it's the longest one found so far.

## Dry Run

```text
nums = [0, 1, 0]
```

Start `i = 0`:

```text
[0] → zeros=1, ones=0 → no
[0,1] → zeros=1, ones=1 → equal! length=2
[0,1,0] → zeros=2, ones=1 → no
```

Start `i = 1`:

```text
[1] → zeros=0, ones=1 → no
[1,0] → zeros=1, ones=1 → equal! length=2
```

Start `i = 2`:

```text
[0] → zeros=1, ones=0 → no
```

Longest found: `2`.

## Algorithm

1. Initialize `maxLen = 0`.
2. For each start index `i` from `0` to `n-1`:

   * Initialize `zeros = 0`, `ones = 0`.
   * For each end index `j` from `i` to `n-1`:

     * Increment `zeros` or `ones` based on `nums[j]`.
     * If `zeros == ones`, update `maxLen = max(maxLen, j - i + 1)`.
3. Return `maxLen`.

## Complexity

* **Time:** `O(n²)`

  * Every pair of `(start, end)` indices is checked, with counts accumulated incrementally in the inner loop.
* **Space:** `O(1)`

  * Only a couple of running counters — no extra structures allocated.

## Notes / Tips

* Already tracks counts incrementally rather than recounting from scratch for each subarray, but still checks every possible subarray explicitly.
* The core insight that unlocks the optimal approach: "equal zeros and ones" is the same as "the running difference between ones and zeros returns to a value it hit before."

## Code

```cpp
class Solution {
public:
    int findMaxLength(vector<int>& nums) {
        int n = nums.size();
        int maxLen = 0;

        for (int i = 0; i < n; i++) {
            int zeros = 0, ones = 0;
            for (int j = i; j < n; j++) {
                if (nums[j] == 0) zeros++;
                else ones++;

                if (zeros == ones) {
                    maxLen = max(maxLen, j - i + 1);
                }
            }
        }

        return maxLen;
    }
};
```

---

# Approach 2 — Better / Prefix Count Arrays + Nested Check

## Idea

Precompute prefix counts of zeros and ones up to every index. Then for every pair `(i, j)`, check in `O(1)` whether the counts of zeros and ones match within that range, instead of recounting with a nested loop.

## Dry Run

```text
nums = [0, 1, 0]
```

Prefix zero/one counts (`prefixZero[0] = 0`, `prefixOne[0] = 0`):

```text
prefixZero = [0, 1, 1, 2]
prefixOne  = [0, 0, 1, 1]
```

Check pair `(i=0, j=2)` (subarray `nums[0..1]`):

```text
zerosInRange = prefixZero[2] - prefixZero[0] = 1
onesInRange  = prefixOne[2] - prefixOne[0] = 1
equal → length = 2
```

Continue checking all pairs the same way → longest found: `2`.

## Algorithm

1. Build `prefixZero` and `prefixOne` arrays of size `n + 1`.
2. For each pair `i < j` (0-indexed into the prefix arrays):

   * `zerosInRange = prefixZero[j] - prefixZero[i]`.
   * `onesInRange = prefixOne[j] - prefixOne[i]`.
   * If equal, update `maxLen = max(maxLen, j - i)`.
3. Return `maxLen`.

## Complexity

* **Time:** `O(n²)`

  * Building prefix arrays is `O(n)`, but checking every pair of indices is still `O(n²)`.
* **Space:** `O(n)`

  * For the two prefix count arrays.

## Notes / Tips

* Still quadratic — precomputing prefix counts only removes the cost of recounting per subarray, not the cost of checking every pair.
* This is a useful stepping stone: subtracting the two prefix arrays (`prefixOne[i] - prefixZero[i]`) collapses them into a single running value, which is exactly the -1/+1 trick used in Approach 3.

## Code

```cpp
class Solution {
public:
    int findMaxLength(vector<int>& nums) {
        int n = nums.size();
        vector<int> prefixZero(n + 1, 0), prefixOne(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefixZero[i + 1] = prefixZero[i] + (nums[i] == 0 ? 1 : 0);
            prefixOne[i + 1] = prefixOne[i] + (nums[i] == 1 ? 1 : 0);
        }

        int maxLen = 0;
        for (int i = 0; i <= n; i++) {
            for (int j = i + 1; j <= n; j++) {
                int zerosInRange = prefixZero[j] - prefixZero[i];
                int onesInRange = prefixOne[j] - prefixOne[i];

                if (zerosInRange == onesInRange) {
                    maxLen = max(maxLen, j - i);
                }
            }
        }

        return maxLen;
    }
};
```

---

# Approach 3 — Optimal / Prefix Sum with -1/+1 Mapping + Hash Map

## Idea

Treat every `0` as `-1` and every `1` as `+1`. A subarray has equal zeros and ones exactly when its transformed sum is `0`, which happens exactly when two prefix sums are equal: `prefixSum[j] == prefixSum[i]`. So instead of comparing counts, track the **first index** each prefix sum value was seen at in a hash map — whenever the same sum reappears, the subarray between those two indices is balanced, and its length is `currentIndex - firstIndex`.

## Dry Run

```text
nums = [0, 1, 0]
```

Transform and accumulate sum, seeding map with `{0: -1}` (sum `0` seen before index `0`):

```text
map = {0: -1}
sum = 0
maxLen = 0
```

```text
i=0, nums[0]=0 → treat as -1 → sum = -1
   not in map → map[-1] = 0

i=1, nums[1]=1 → treat as +1 → sum = 0
   0 is in map at index -1 → length = 1 - (-1) = 2 → maxLen = 2

i=2, nums[2]=0 → treat as -1 → sum = -1
   -1 is in map at index 0 → length = 2 - 0 = 2 → maxLen stays 2
```

Final `maxLen = 2`.

## Algorithm

1. Initialize a hash map `firstIndex` with `{0: -1}` (sum `0` occurs "before" the array starts).
2. Initialize `sum = 0` and `maxLen = 0`.
3. For each index `i` and value `nums[i]`:

   * `sum += (nums[i] == 1) ? 1 : -1`.
   * If `sum` already exists in `firstIndex`, update `maxLen = max(maxLen, i - firstIndex[sum])`.
   * Otherwise, store `firstIndex[sum] = i` (only the **first** occurrence matters, since it maximizes subarray length).
4. Return `maxLen`.

## Complexity

* **Time:** `O(n)`

  * Single pass, each hash map lookup and insert is `O(1)` on average.
* **Space:** `O(n)`

  * For the hash map, which can hold up to `n + 1` distinct prefix sum values.

## Notes / Tips

* Only the **first** occurrence of each sum value should be stored — overwriting it with a later index would shrink the subarray length instead of maximizing it (this differs from LC 560, which counts occurrences instead of tracking first index, since that problem wants a count, not a max length).
* Seeding the map with `{0: -1}` correctly handles subarrays that start at index `0` and are already balanced.
* The -1/+1 transformation is the key trick — it converts "equal counts of two categories" into a single running sum, turning this into a pure prefix-sum-equality problem.

## Code

```cpp
class Solution {
public:
    int findMaxLength(vector<int>& nums) {
        unordered_map<int, int> firstIndex;
        firstIndex[0] = -1;

        int sum = 0, maxLen = 0;

        for (int i = 0; i < nums.size(); i++) {
            sum += (nums[i] == 1) ? 1 : -1;

            if (firstIndex.find(sum) != firstIndex.end()) {
                maxLen = max(maxLen, i - firstIndex[sum]);
            } else {
                firstIndex[sum] = i;
            }
        }

        return maxLen;
    }
};
```

---

## Key Template

```text
firstIndex = {0: -1}
sum = 0
maxLen = 0

for i, num in enumerate(nums):
    sum += 1 if num == 1 else -1

    if sum in firstIndex:
        maxLen = max(maxLen, i - firstIndex[sum])
    else:
        firstIndex[sum] = i

return maxLen
```