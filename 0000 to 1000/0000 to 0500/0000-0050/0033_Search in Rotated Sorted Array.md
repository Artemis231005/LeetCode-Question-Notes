# LeetCode 33 — Search in Rotated Sorted Array

## Metadata

* **LeetCode:** 33
* **Problem:** Search in Rotated Sorted Array
* **Difficulty:** Medium
* **Topics:** Array, Binary Search
* **Pattern:** Modified Binary Search
* **Key Technique:** Identify the Sorted Half
* **Key Pattern:** Binary Search on Rotated Array
* **Optimal Complexity:** `O(log n)` time, `O(1)` space

---

## Problem

You are given an array of **distinct integers** `nums` that was originally sorted in ascending order and then rotated at an unknown pivot.

Return the index of `target` if it exists. Otherwise, return `-1`.

The algorithm must run in:

```text
O(log n)
```

### Example

```text
nums = [4, 5, 6, 7, 0, 1, 2]
target = 0

Output = 4
```

The original sorted array was:

```text
[0, 1, 2, 4, 5, 6, 7]
```

It was rotated to:

```text
[4, 5, 6, 7, 0, 1, 2]
```

---

# Approach 1 — Linear Search

## Idea

Simply scan every element and return the index when `nums[i] == target`.

This works, but it does not use the sorted structure of the array.

## Dry Run

```text
nums = [4, 5, 6, 7, 0, 1, 2]
target = 0
```

Scan:

```text
4 → no
5 → no
6 → no
7 → no
0 → found
```

Return:

```text
4
```

## Algorithm

1. Traverse the array from left to right.
2. If `nums[i] == target`, return `i`.
3. If the entire array is searched, return `-1`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* Simple but does not satisfy the required `O(log n)` complexity.
* The important observation is that **at least one half of the array is always sorted**.

## Code

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] == target) {
                return i;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Modified Binary Search

## Idea

Even though the entire array is not sorted, **one half of the current search range is always sorted**.

For:

```text
[4, 5, 6, 7, 0, 1, 2]
```

if:

```text
left = 0
mid = 3
right = 6
```

we have:

```text
left half  = [4, 5, 6, 7]  → sorted
right half = [0, 1, 2]     → sorted
```

We determine which half is sorted and then check whether `target` lies inside that sorted range.

### Case 1 — Left Half Is Sorted

If:

```cpp
nums[left] <= nums[mid]
```

then:

```text
[left ... mid]
```

is sorted.

Now check whether:

```text
nums[left] <= target < nums[mid]
```

If yes, search the left half.

Otherwise, search the right half.

### Case 2 — Right Half Is Sorted

Otherwise:

```text
[mid ... right]
```

is sorted.

Check whether:

```text
nums[mid] < target <= nums[right]
```

If yes, search the right half.

Otherwise, search the left half.

## Dry Run

Consider:

```text
nums = [4, 5, 6, 7, 0, 1, 2]
target = 0
```

### Step 1

```text
left = 0
mid = 3
right = 6
```

Values:

```text
4 5 6 7 0 1 2
↑     ↑       ↑
L     M       R
```

Check:

```text
nums[left] <= nums[mid]
4 <= 7
```

So the left half is sorted:

```text
[4, 5, 6, 7]
```

Is `0` inside this range?

```text
4 <= 0 < 7 → false
```

Therefore, target must be in the right half:

```text
left = mid + 1 = 4
```

### Step 2

Now:

```text
left = 4
mid = 5
right = 6
```

Values:

```text
[0, 1, 2]
 ↑   ↑   ↑
 L   M   R
```

Again:

```text
nums[left] <= nums[mid]
0 <= 1
```

Left half is sorted.

Is target `0` inside:

```text
0 <= 0 < 1
```

Yes.

Therefore:

```text
right = mid - 1 = 4
```

### Step 3

Now:

```text
left = 4
right = 4
mid = 4
```

```text
nums[4] = 0
```

Target found.

Return:

```text
4
```

## Algorithm

1. Initialize:

   ```cpp
   left = 0
   right = nums.size() - 1
   ```
2. While `left <= right`:

   * Calculate:

     ```cpp
     mid = left + (right - left) / 2
     ```
   * If `nums[mid] == target`, return `mid`.
   * Determine which half is sorted:

     ```cpp
     nums[left] <= nums[mid]
     ```
3. If the left half is sorted:

   * Check whether target lies inside:

     ```cpp
     nums[left] <= target && target < nums[mid]
     ```
   * If yes, search left.
   * Otherwise, search right.
4. Otherwise, the right half is sorted:

   * Check whether target lies inside:

     ```cpp
     nums[mid] < target && target <= nums[right]
     ```
   * If yes, search right.
   * Otherwise, search left.
5. If the target is never found, return `-1`.

## Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

## Notes / Tips

### The Most Important Observation

For every binary-search range:

> **At least one half is guaranteed to be sorted.**

Use that sorted half to decide where the target can possibly be.

### Why `<=` on the Left?

Use:

```cpp
nums[left] <= nums[mid]
```

because `left == mid` is possible when only one element remains.

### Target Range Conditions

For the sorted left half:

```cpp
nums[left] <= target && target < nums[mid]
```

For the sorted right half:

```cpp
nums[mid] < target && target <= nums[right]
```

Notice the different `<` and `<=` boundaries.

### Why Does This Work?

A rotation creates one "break" in the sorted order.

For example:

```text
[4, 5, 6, 7 | 0, 1, 2]
              ↑
           rotation
```

Even though the whole array is not sorted, each side of the break remains sorted.

Binary search can therefore be adapted by identifying the sorted half at every step.

### Common Mistake

Do **not** simply compare:

```cpp
target < nums[mid]
```

and decide the direction like normal binary search.

The entire array is not sorted, so you must first determine **which half is sorted**.

## Code

```cpp
class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return mid;
            }

            // Left half is sorted
            if (nums[left] <= nums[mid]) {
                if (nums[left] <= target && target < nums[mid]) {
                    right = mid - 1;
                }
                else {
                    left = mid + 1;
                }
            }
            // Right half is sorted
            else {
                if (nums[mid] < target && target <= nums[right]) {
                    left = mid + 1;
                }
                else {
                    right = mid - 1;
                }
            }
        }

        return -1;
    }
};
```

---

# Approach 3 — Find Pivot + Normal Binary Search

## Idea

Another way to think about the problem is:

1. Find the rotation point (pivot).
2. This divides the array into two sorted arrays.
3. Perform normal binary search on the appropriate half.

For:

```text
[4, 5, 6, 7, 0, 1, 2]
```

the pivot is:

```text
7 | 0
```

So we have:

```text
[4, 5, 6, 7]
[0, 1, 2]
```

Both are sorted.

## Dry Run

```text
nums = [4, 5, 6, 7, 0, 1, 2]
target = 0
```

Pivot:

```text
index = 4
value = 0
```

Since:

```text
target = 0
```

lies in the second sorted portion:

```text
[0, 1, 2]
```

perform binary search there.

```text
mid = 5
nums[5] = 1
```

Target is smaller:

```text
right = 4
```

Now:

```text
nums[4] = 0
```

Found.

Return `4`.

## Algorithm

1. Find the index of the smallest element.
2. This index is the rotation pivot.
3. Determine which sorted portion contains `target`.
4. Perform normal binary search on that portion.
5. Return the result or `-1`.

## Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

## Notes / Tips

* This approach is valid but requires more steps than the modified binary search.
* The direct modified binary search is usually preferred in interviews.
* Understanding the pivot approach is still useful because it explains the structure of the rotated array.

## Code

```cpp
class Solution {
public:
    int binarySearch(vector<int>& nums, int left, int right, int target) {
        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return mid;
            }
            else if (nums[mid] < target) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }

        return -1;
    }

    int search(vector<int>& nums, int target) {
        int n = nums.size();

        // Find pivot
        int left = 0;
        int right = n - 1;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] > nums[right]) {
                left = mid + 1;
            }
            else {
                right = mid;
            }
        }

        int pivot = left;

        // Array is not rotated
        if (pivot == 0) {
            return binarySearch(nums, 0, n - 1, target);
        }

        // Target lies in the left sorted portion
        if (target >= nums[0]) {
            return binarySearch(nums, 0, pivot - 1, target);
        }

        // Target lies in the right sorted portion
        return binarySearch(nums, pivot, n - 1, target);
    }
};
```

---

# Comparison of Approaches

| Approach               |       Time |  Space | Main Idea               |
| ---------------------- | ---------: | -----: | ----------------------- |
| Linear Search          |     `O(n)` | `O(1)` | Check every element     |
| Modified Binary Search | `O(log n)` | `O(1)` | Find sorted half        |
| Pivot + Binary Search  | `O(log n)` | `O(1)` | Find pivot, then search |

---

# Key Takeaway

The most important pattern is:

```text
Rotated Sorted Array
        ↓
At least one half is sorted
        ↓
Identify sorted half
        ↓
Check whether target lies in it
        ↓
Discard the other half
        ↓
Repeat
```

### Key Template

```cpp
int left = 0;
int right = nums.size() - 1;

while (left <= right) {
    int mid = left + (right - left) / 2;

    if (nums[mid] == target) {
        return mid;
    }

    if (nums[left] <= nums[mid]) {
        // Left half is sorted

        if (nums[left] <= target && target < nums[mid]) {
            right = mid - 1;
        }
        else {
            left = mid + 1;
        }
    }
    else {
        // Right half is sorted

        if (nums[mid] < target && target <= nums[right]) {
            left = mid + 1;
        }
        else {
            right = mid - 1;
        }
    }
}

return -1;
```

**Mental Model:**

> Don't ask "Is the whole array sorted?"
> Ask **"Which half is sorted?"**

Then ask:

> **"Can my target lie inside that sorted half?"**

If yes, search there. Otherwise, search the other half.

**Key Pattern:** **Modified Binary Search → Identify Sorted Half → Check Target Range → Discard Half.**
