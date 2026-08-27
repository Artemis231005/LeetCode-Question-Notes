# LeetCode 643 — Maximum Average Subarray I

## Metadata

* **LeetCode:** 643
* **Problem:** Maximum Average Subarray I
* **Difficulty:** Easy
* **Topics:** Array, Sliding Window
* **Pattern:** Fixed-Size Sliding Window
* **Key Technique:** Maintain Window Sum
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Problem Statement

Find the **maximum average** of any contiguous subarray of length `k`.

---

# Brute Approach

## Idea
Since the subarray must have **exactly `k` elements**, try every possible starting position.

For each starting position:
1. Calculate the sum of the next `k` elements.
2. Calculate its average.
3. Keep track of the maximum average.

The same elements are repeatedly added when calculating overlapping subarrays, making this inefficient.

---

## Dry Run

For:
```text
nums = [1, 12, -5, -6, 50, 3]
k = 4
```

Possible subarrays:
```text
[1, 12, -5, -6]  → sum = 2   → avg = 0.5
[12, -5, -6, 50] → sum = 51  → avg = 12.75
[-5, -6, 50, 3]  → sum = 42  → avg = 10.5
```

Maximum average:
```text
12.75
```

---

## Algorithm

1. Initialize `maxAvg` to a very small value.
2. For every possible starting index `i`:

   * Calculate the sum of `k` elements starting from `i`.
   * Calculate `sum / k`.
   * Update `maxAvg`.
3. Return `maxAvg`.

---

## Complexity
* **TC:** `O(n × k)`

  * There are approximately `n` windows.
  * Each window requires `k` additions.
* **SC:** `O(1)`.

---

## Code
```cpp
class Solution {
public:
    double findMaxAverage(vector<int>& nums, int k) {
        int n = nums.size();
        double maxAvg = -1e9;

        for (int i = 0; i <= n - k; i++) {
            int sum = 0;

            for (int j = i; j < i + k; j++) {
                sum += nums[j];
            }

            maxAvg = max(maxAvg, (double)sum / k);
        }

        return maxAvg;
    }
};
```

---

# Better Approach

## Idea
Use a **prefix sum** array so that the sum of any subarray can be calculated in `O(1)`.

Create:
```text
prefix[i] = sum of elements from index 0 to i - 1
```

Then:
```text
sum of nums[l ... r] = prefix[r + 1] - prefix[l]
```

So each window's sum can be found in constant time.

---

## Dry Run

For:
```text
nums = [1, 12, -5, -6, 50, 3]
```

Prefix sum:
```text
[0, 1, 13, 8, 2, 52, 55]
```

For window `[12, -5, -6, 50]`:
```text
sum = prefix[5] - prefix[1]
    = 52 - 1
    = 51

average = 51 / 4
        = 12.75
```

---

## Algorithm

1. Create a prefix sum array.
2. Calculate the prefix sum of `nums`.
3. For every possible window of size `k`:

   * Calculate its sum using the prefix sum array.
   * Calculate its average.
   * Update the maximum average.
4. Return the maximum average.

---

## Complexity
* **TC:** `O(n)`

  * Building prefix sum: `O(n)`.
  * Checking all windows: `O(n)`.
* **SC:** `O(n)`.

---

## Code

```cpp
class Solution {
public:
    double findMaxAverage(vector<int>& nums, int k) {
        int n = nums.size();

        vector<int> prefix(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        double maxAvg = -1e9;

        for (int i = 0; i <= n - k; i++) {
            int sum = prefix[i + k] - prefix[i];
            maxAvg = max(maxAvg, (double)sum / k);
        }

        return maxAvg;
    }
};
```

---

# Optimal Approach

## Idea

Notice that consecutive windows overlap heavily.
Instead of recalculating the entire sum, maintain the sum of the current window and **slide it by one position**.

When the window moves:
```text
Remove the element leaving the window
        +
Add the new element entering the window
```

So:
```text
newSum = oldSum - nums[i - k] + nums[i]
```

This is the **fixed-size sliding window** pattern. LeetCode's constraints allow `n` up to `10^5`, making the `O(n)` approach appropriate.

---

## Dry Run

For:
```text
nums = [1, 12, -5, -6, 50, 3]
k = 4
```

### Initial Window
```text
[1, 12, -5, -6]
```

```text
sum = 2
avg = 0.5
```

### Slide Right

Remove `1`, add `50`:
```text
[12, -5, -6, 50]
```

```text
sum = 2 - 1 + 50
    = 51

avg = 51 / 4
    = 12.75
```

### Slide Again

Remove `12`, add `3`:
```text
[-5, -6, 50, 3]
```

```text
sum = 51 - 12 + 3
    = 42

avg = 42 / 4
    = 10.5
```

Therefore:
```text
maximum average = 12.75
```

---

## Algorithm

1. Calculate the sum of the first `k` elements.
2. Store it as `windowSum`.
3. Set `maxSum = windowSum`.
4. Start sliding the window from index `k`.
5. For every new element:

   * Add the new element.
   * Remove the element that left the window.
   * Update `maxSum`.
6. Return `(double)maxSum / k`.

---

## Complexity
* **TC:** `O(n)`

  * Each element enters the window once and leaves once.
* **SC:** `O(1)` auxiliary space.

---

## Code

```cpp
class Solution {
public:
    double findMaxAverage(vector<int>& nums, int k) {
        int n = nums.size();

        int windowSum = 0;

        // First window
        for (int i = 0; i < k; i++) {
            windowSum += nums[i];
        }

        int maxSum = windowSum;

        // Slide the window
        for (int i = k; i < n; i++) {
            windowSum += nums[i];
            windowSum -= nums[i - k];

            maxSum = max(maxSum, windowSum);
        }

        return (double)maxSum / k;
    }
};
```

---

## Notes / Tips

* This is a **fixed-size sliding window** problem.
* Whenever the subarray length is fixed at `k`, think:
  **"Can I maintain the previous window instead of recalculating it?"**
* The average denominator is always `k`, so maximizing the **average** is equivalent to maximizing the **sum**.
* Therefore, we only need to track `maxSum`; division by `k` is done once at the end.
* Sliding window reduces the brute-force `O(n × k)` solution to `O(n)`.
* Use `double` when returning the final average.
* `int` is sufficient for the sum under the given constraints, but `long long` can also be used safely.

---

## Key Template

```text
Fixed Window of Size K
      ↓
Calculate First Window
      ↓
Slide Window
      ↓
Add Incoming Element
      +
Remove Outgoing Element
      ↓
Update Maximum
      ↓
Return Result
```

**Pattern:** `Fixed-Size Sliding Window`
**Core Formula:**

```text
new window sum
  = old window sum
  - outgoing element
  + incoming element
```
