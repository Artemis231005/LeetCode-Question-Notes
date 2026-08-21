# LeetCode 153 — Find Minimum in Rotated Sorted Array

## Metadata

* **LeetCode:** 153
* **Problem:** Find Minimum in Rotated Sorted Array
* **Difficulty:** Medium
* **Topics:** Array, Binary Search
* **Pattern:** Modified Binary Search
* **Key Technique:** Compare `nums[mid]` with `nums[right]`
* **Key Pattern:** Find rotation point / minimum
* **Key Template:** `nums[mid] > nums[right] → search right`
* **Optimal Complexity:** `O(log n)`

---

## Problem

Given an array `nums` of **unique** integers that was originally sorted in ascending order and then rotated at an unknown pivot, find the minimum element.

Example:

```text
nums = [3, 4, 5, 1, 2]
```

Original sorted array:

```text
[1, 2, 3, 4, 5]
```

After rotation:

```text
[3, 4, 5, 1, 2]
```

Minimum:

```text
1
```

The solution must run in `O(log n)` time.

---

## Idea

The array consists of two sorted portions:

```text
[3, 4, 5] [1, 2]
```

The minimum is exactly where the rotation occurs.

Use **Binary Search**.

Maintain:

```text
left = 0
right = n - 1
```

At every step, calculate:

```text
mid = left + (right - left) / 2
```

Compare:

```text
nums[mid] with nums[right]
```

### Case 1 — `nums[mid] > nums[right]`

Example:

```text
[3, 4, 5, 1, 2]
       ↑     ↑
      mid   right
       5  >  2
```

`mid` lies in the **left sorted portion**.

Therefore, the minimum must be to the **right of `mid`**.

```text
left = mid + 1
```

### Case 2 — `nums[mid] < nums[right]`

Example:

```text
[5, 1, 2, 3, 4]
     ↑        ↑
    mid      right
     2   <    4
```

`mid` lies in the sorted portion containing the minimum, so the minimum could be `mid` itself.

Therefore:

```text
right = mid
```

### Why `right = mid` and not `mid - 1`?

Because `mid` itself can be the minimum.

Example:

```text
[2, 1, 3]
    ↑
   mid
```

So we must keep `mid`.

---

## Dry Run

Consider:

```text
nums = [4, 5, 6, 7, 0, 1, 2]
```

Initial:

```text
left = 0
right = 6
```

### Step 1

```text
mid = 3

nums[mid] = 7
nums[right] = 2
```

Since:

```text
7 > 2
```

minimum is to the right.

```text
left = mid + 1 = 4
```

Now:

```text
[4, 5, 6, 7, 0, 1, 2]
             L     R
```

### Step 2

```text
left = 4
right = 6
mid = 5
```

Compare:

```text
nums[5] = 1
nums[6] = 2
```

Since:

```text
1 < 2
```

minimum is at `mid` or to its left.

```text
right = mid = 5
```

### Step 3

```text
left = 4
right = 5
mid = 4
```

Compare:

```text
nums[4] = 0
nums[5] = 1
```

Since:

```text
0 < 1
```

keep `mid`:

```text
right = 4
```

Now:

```text
left = right = 4
```

Therefore:

```text
nums[left] = 0
```

Answer:

```text
0
```

---

## Algorithm

1. Set `left = 0` and `right = n - 1`.
2. While `left < right`:

   * Calculate `mid`.
   * If `nums[mid] > nums[right]`:

     ```text
     left = mid + 1
     ```
   * Otherwise:

     ```text
     right = mid
     ```
3. When `left == right`, return `nums[left]`.

---

## Complexity

* **Time:** `O(log n)`

  * Search space is approximately halved at every step.
* **Space:** `O(1)`

  * Only a few variables are used.

---

## Notes / Tips

* This is a **modified binary search** problem.
* The key comparison is:

  ```text
  nums[mid] vs nums[right]
  ```
* If:

  ```text
  nums[mid] > nums[right]
  ```

  the minimum is strictly to the right of `mid`.
* If:

  ```text
  nums[mid] < nums[right]
  ```

  `mid` could itself be the minimum, so keep it.
* Because all elements are **unique**, there is no equality case.
* The array may not be rotated:

  ```text
  [1, 2, 3, 4, 5]
  ```

  The algorithm still works.

### Important Observation

If:

```text
nums[mid] > nums[right]
```

then `mid` is definitely **not** the minimum.

So:

```text
left = mid + 1
```

But if:

```text
nums[mid] < nums[right]
```

then `mid` may be the minimum.

So:

```text
right = mid
```

### Common Mistake

Do not write:

```cpp
right = mid - 1;
```

in the second case.

You may remove the minimum itself.

Correct:

```cpp
right = mid;
```

---

## Code

```cpp
class Solution {
public:
    int findMin(vector<int>& nums) {
        int left = 0;
        int right = nums.size() - 1;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] > nums[right]) {
                left = mid + 1;
            } else {
                right = mid;
            }
        }

        return nums[left];
    }
};
```

---

## Basic Template

```cpp
int findMin(vector<int>& nums) {
    int left = 0;
    int right = nums.size() - 1;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] > nums[right]) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }

    return nums[left];
}
```

### Reusable Pattern

```text
left = 0, right = n - 1
        ↓
while (left < right)
        ↓
      mid
        ↓
nums[mid] > nums[right]?
      ↙          ↘
    Yes           No
     ↓             ↓
left = mid + 1  right = mid
     ↓             ↓
     └───────┬─────┘
             ↓
      left == right
             ↓
       nums[left]
```

### Core Rule

```text
nums[mid] > nums[right]
→ minimum is right of mid
→ left = mid + 1

nums[mid] < nums[right]
→ minimum is at or left of mid
→ right = mid
```
