# LeetCode 53 — Maximum Subarray

## Metadata

* **LeetCode:** 53
* **Problem:** Maximum Subarray
* **Difficulty:** Medium
* **Topics:** Array, Dynamic Programming
* **Pattern:** Kadane's Algorithm
* **Key Pattern:** Running maximum subarray sum
* **Key Technique:** At each position, decide whether to extend the current subarray or start a new one
* **Key Template:** Kadane's Algorithm
* **Optimal Complexity:** `O(n)` time, `O(1)` space

---

## Problem

Given an integer array `nums`, find the **subarray with the largest sum** and return its sum.

A subarray must contain **at least one element** and consist of **contiguous** elements.

Example:

```text
nums = [-2,1,-3,4,-1,2,1,-5,4]

Maximum subarray:
[4,-1,2,1]

Sum = 6
```

---

## Approach — Kadane's Algorithm

### Idea

At every index, maintain:

```text
currentSum = maximum sum of a subarray ending at the current index
```

For each element `nums[i]`, there are only two choices:

1. **Extend** the previous subarray:

   ```text
   currentSum + nums[i]
   ```

2. **Start a new subarray** from the current element:

   ```text
   nums[i]
   ```

So:

```text
currentSum = max(nums[i], currentSum + nums[i])
```

Then maintain the best answer seen so far:

```text
maxSum = max(maxSum, currentSum)
```

The key observation is:

> If the previous `currentSum` is negative, carrying it forward can only make the next subarray smaller, so start fresh.

---

### Dry Run

For:

```text
[-2, 1, -3, 4, -1, 2, 1, -5, 4]
```

Initialize:

```text
currentSum = -2
maxSum = -2
```

| Element | Calculation       | `currentSum` | `maxSum` |
| ------: | ----------------- | -----------: | -------: |
|    `-2` | `max(-2, 0 + -2)` |         `-2` |     `-2` |
|     `1` | `max(1, -2 + 1)`  |          `1` |      `1` |
|    `-3` | `max(-3, 1 - 3)`  |         `-2` |      `1` |
|     `4` | `max(4, -2 + 4)`  |          `4` |      `4` |
|    `-1` | `max(-1, 4 - 1)`  |          `3` |      `4` |
|     `2` | `max(2, 3 + 2)`   |          `5` |      `5` |
|     `1` | `max(1, 5 + 1)`   |          `6` |      `6` |
|    `-5` | `max(-5, 6 - 5)`  |          `1` |      `6` |
|     `4` | `max(4, 1 + 4)`   |          `5` |      `6` |

Final answer:

```text
6
```

The corresponding subarray is:

```text
[4, -1, 2, 1]
```

---

### Algorithm

1. Initialize:

   ```text
   currentSum = nums[0]
   maxSum = nums[0]
   ```
2. Traverse the array from index `1`.
3. For each element:

   * Decide whether to start a new subarray or extend the existing one:

     ```text
     currentSum = max(nums[i], currentSum + nums[i])
     ```
   * Update the global maximum:

     ```text
     maxSum = max(maxSum, currentSum)
     ```
4. Return `maxSum`.

---

### Complexity

* **Time:** `O(n)` — one pass through the array.
* **Space:** `O(1)` — only two variables are required.

---

### Notes / Tips

* Kadane's Algorithm is one of the most important **array DP patterns**.
* The DP interpretation is:

```text
dp[i] = maximum subarray sum ending at index i
```

with:

```text
dp[i] = max(nums[i], dp[i-1] + nums[i])
```

* Since we only need the previous DP value, we optimize:

```text
O(n) space → O(1) space
```

* **Do not initialize `maxSum = 0`** if the array can contain all negative numbers.

Example:

```text
[-5, -2, -8]
```

Correct answer:

```text
-2
```

If initialized to `0`, the algorithm would incorrectly return `0`.

* A useful way to remember the decision:

```text
Previous sum is helping?
→ Extend it.

Previous sum is hurting?
→ Start fresh.
```

* If the question asks for the **actual subarray**, not just its sum, track the starting and ending indices while running Kadane's Algorithm.

---

### Code

```cpp
class Solution {
public:
    int maxSubArray(vector<int>& nums) {
        int currentSum = nums[0];
        int maxSum = nums[0];

        for (int i = 1; i < nums.size(); i++) {
            currentSum = max(nums[i], currentSum + nums[i]);
            maxSum = max(maxSum, currentSum);
        }

        return maxSum;
    }
};
```

---

## Quick Revision

```text
Kadane's Algorithm

currentSum = best sum ending at current index
maxSum     = best sum found anywhere

For every element:

currentSum = max(nums[i], currentSum + nums[i])
maxSum = max(maxSum, currentSum)
```

### Core Template

```text
current = first element
answer = first element

for each remaining element:
    current = max(element, current + element)
    answer = max(answer, current)
```

### Key Insight

```text
Negative previous sum
        ↓
Discard it
        ↓
Start new subarray
```

**Pattern to remember:**
**Maximum contiguous subarray → Kadane's Algorithm → `max(current element, extend previous)`**
