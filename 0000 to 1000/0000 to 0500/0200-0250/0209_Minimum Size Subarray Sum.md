# LeetCode 209 — Minimum Size Subarray Sum

## Metadata

* **LeetCode:** 209
* **Problem:** Minimum Size Subarray Sum
* **Difficulty:** Medium
* **Topics:** Array, Prefix Sum, Binary Search, Sliding Window
* **Pattern:** Variable-Size Sliding Window
* **Key Technique:** Expand until `sum >= target`, then shrink to find the minimum window.
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Problem Statement

Find the **minimum length of a contiguous subarray** whose sum is at least `target`.
All elements of `nums` are **positive integers**.

---

# Brute Approach

## Idea
Try every possible subarray and calculate its sum.

For every starting index `i`, keep extending the subarray to the right until the sum becomes at least `target`.
Once the sum reaches `target`, update the minimum length.

---

## Dry Run

```text
target = 7
nums = [2,3,1,2,4,3]
```

Starting from index `0`:
```text
[2]       → sum = 2
[2,3]     → sum = 5
[2,3,1]   → sum = 6
[2,3,1,2] → sum = 8 → length = 4
```

Starting from index `2`:
```text
[1,2,4] → sum = 7 → length = 3
```

Starting from index `4`:
```text
[4,3] → sum = 7 → length = 2
```

Answer:
```text
2
```

---

## Algorithm

1. Initialize `ans = INT_MAX`.
2. For every starting index `i`:

   * Set `sum = 0`.
   * Extend the subarray using `j`.
   * Add `nums[j]` to `sum`.
   * Whenever `sum >= target`, update `ans`.
3. Return `0` if no valid subarray exists.

---

## Complexity

* **TC:** `O(n²)`
* **SC:** `O(1)`

---

## Code

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int n = nums.size();
        int ans = INT_MAX;

        for (int i = 0; i < n; i++) {
            int sum = 0;

            for (int j = i; j < n; j++) {
                sum += nums[j];

                if (sum >= target) {
                    ans = min(ans, j - i + 1);
                    break;
                }
            }
        }

        return ans == INT_MAX ? 0 : ans;
    }
};
```

---

# Better Approach 1 — Prefix Sum

## Idea

Instead of calculating the sum of every subarray repeatedly, create a **Prefix Sum Array**.

Define:
```text
prefix[i] = sum of elements from index 0 to i - 1
```

Then the sum of subarray `[i...j]` is:
```text
sum(i...j) = prefix[j + 1] - prefix[i]
```

This allows us to calculate any subarray sum in `O(1)`.
However, we still have to try all possible `(i, j)` pairs, so the overall complexity is `O(n²)`.

---

## Dry Run

```text
target = 7
nums = [2,3,1,2,4,3]
```

Prefix sum:
```text
prefix = [0,2,5,6,8,12,15]
```

Consider:
```text
[2,3,1,2]
```

Indices:
```text
i = 0
j = 3
```

Sum:
```text
prefix[j + 1] - prefix[i]
= prefix[4] - prefix[0]
= 8 - 0
= 8
```

Length:
```text
j - i + 1
= 3 - 0 + 1
= 4
```

---

## Algorithm

1. Create a prefix sum array.
2. For every starting index `i`:

   * For every ending index `j`:
   * Calculate:

     ```text
     sum = prefix[j + 1] - prefix[i]
     ```
   * If `sum >= target`, update the minimum length.
3. Return `0` if no valid subarray exists.

---

## Complexity
* **TC:** `O(n²)`

  * Prefix sum construction: `O(n)`.
  * Checking all subarrays: `O(n²)`.
* **SC:** `O(n)`.

---

## Code

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int n = nums.size();

        vector<int> prefix(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        int ans = INT_MAX;

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j <= n; j++) {
                int sum = prefix[j] - prefix[i];

                if (sum >= target) {
                    ans = min(ans, j - i);
                }
            }
        }

        return ans == INT_MAX ? 0 : ans;
    }
};
```

---

# Better Approach 2 — Prefix Sum + Binary Search

## Idea

We can improve the Prefix Sum approach because **all numbers are positive**.

Therefore, the prefix sum array is **strictly increasing**:
```text
nums = [2,3,1,2,4,3]
prefix = [0,2,5,6,8,12,15]
```

For a starting position `i`, we need:
```text
prefix[j] - prefix[i] >= target
```

Rearrange:
```text
prefix[j] >= prefix[i] + target
```

So for each `i`, we can use **binary search** to find the first prefix sum that is at least:
```text
prefix[i] + target
```

That gives the shortest valid subarray starting at `i`.

---

## Dry Run

```text
target = 7
nums = [2,3,1,2,4,3]

prefix = [0,2,5,6,8,12,15]
```

For `i = 0`:
```text
prefix[i] + target
= 0 + 7
= 7
```

Find the first prefix value `>= 7`:
```text
8
```

This is at index `4`.

Therefore:
```text
length = 4 - 0 = 4
```

For `i = 3`:
```text
prefix[3] + target
= 6 + 7
= 13
```

First prefix value `>= 13` is:
```text
15
```

at index `6`.

Therefore:
```text
length = 6 - 3
       = 3
```

For `i = 4`:
```text
prefix[4] + target
= 8 + 7
= 15
```

Binary search finds `15` at index `6`.

Therefore:
```text
length = 6 - 4
       = 2
```

Answer:
```text
2
```

---

## Algorithm

1. Build the prefix sum array.
2. For every starting position `i`:

   * Calculate:

     ```text
     targetSum = prefix[i] + target
     ```
   * Binary search for the first index `j` such that:

     ```text
     prefix[j] >= targetSum
     ```
   * If found, update:

     ```text
     ans = min(ans, j - i)
     ```
3. Return `0` if no valid subarray exists.

---

## Complexity
* **TC:** `O(n log n)`

  * Build prefix sum: `O(n)`.
  * `n` binary searches: `O(n log n)`.
* **SC:** `O(n)`.

---

## Code

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int n = nums.size();

        vector<int> prefix(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        int ans = INT_MAX;

        for (int i = 0; i < n; i++) {
            int targetSum = prefix[i] + target;

            int left = i + 1;
            int right = n;
            int pos = -1;

            while (left <= right) {
                int mid = left + (right - left) / 2;

                if (prefix[mid] >= targetSum) {
                    pos = mid;
                    right = mid - 1;
                }
                else {
                    left = mid + 1;
                }
            }

            if (pos != -1) {
                ans = min(ans, pos - i);
            }
        }

        return ans == INT_MAX ? 0 : ans;
    }
};
```

---

# Optimal Approach — Sliding Window

## Idea

Since all numbers are **positive**, we can use a variable-size sliding window.

Maintain:
```text
left = 0
sum = 0
```

Expand the window using `right`.

When:
```text
sum >= target
```
the current window is valid.

Now shrink from the left as much as possible while keeping the sum at least `target`.

---

## Dry Run

```text
target = 7
nums = [2,3,1,2,4,3]
```

Expand:
```text
[2,3,1,2] → sum = 8 → length = 4
```

Shrink:
```text
[3,1,2] → sum = 6 → invalid
```

Continue:
```text
[3,1,2,4] → sum = 10 → length = 4
```

Shrink:
```text
[1,2,4] → sum = 7 → length = 3
[2,4]   → sum = 6 → invalid
```

Continue:
```text
[2,4,3] → sum = 9
[4,3]   → sum = 7 → length = 2
```

Answer:
```text
2
```

---

## Algorithm

1. Initialize `left = 0`, `sum = 0`, and `ans = INT_MAX`.
2. Move `right` through the array.
3. Add `nums[right]` to `sum`.
4. While `sum >= target`:

   * Update the minimum length.
   * Remove `nums[left]`.
   * Move `left` forward.
5. Return `0` if no valid subarray exists.

---

## Complexity
* **TC:** `O(n)`
* **SC:** `O(1)`

---

## Code

```cpp
class Solution {
public:
    int minSubArrayLen(int target, vector<int>& nums) {
        int left = 0;
        int sum = 0;
        int ans = INT_MAX;

        for (int right = 0; right < nums.size(); right++) {
            sum += nums[right];

            while (sum >= target) {
                ans = min(ans, right - left + 1);

                sum -= nums[left];
                left++;
            }
        }

        return ans == INT_MAX ? 0 : ans;
    }
};
```

---

## Notes / Tips

* **Brute:** Try every subarray → `O(n²)`.
* **Prefix Sum:** Quickly calculate subarray sums → `O(n²)` time, `O(n)` space.
* **Prefix Sum + Binary Search:** Use increasing prefix sums → `O(n log n)` time, `O(n)` space.
* **Sliding Window:** Exploit positive numbers → `O(n)` time, `O(1)` space.
* The key reason Sliding Window works is that **all numbers are positive**:

  * Expand → sum increases.
  * Shrink → sum decreases.
* Use `while`, not `if`, when shrinking because we need the **smallest valid window**.
* Prefix Sum + Binary Search works because positive numbers make the prefix sum **monotonically increasing**.
* The Prefix Sum + Binary Search condition is:

  ```text
  prefix[j] >= prefix[i] + target
  ```
* For this problem, **Sliding Window is better than Prefix Sum** because it achieves `O(n)` time and `O(1)` space.

---

## Key Templates

### Prefix Sum

```text
prefix[i + 1] = prefix[i] + nums[i]

sum(i...j) = prefix[j + 1] - prefix[i]
```

### Prefix Sum + Binary Search

```text
prefix[j] - prefix[i] >= target
        ↓
prefix[j] >= prefix[i] + target
        ↓
Binary Search for first valid j
```

### Variable-Size Sliding Window

```text
left = 0
sum = 0

for right = 0 to n - 1:
    sum += nums[right]
    while sum >= target:
        update answer
        sum -= nums[left]
        left++
```

**Core Pattern:**

```text
Expand → Become Valid → Shrink → Find Minimum
```
