# LeetCode 34 — Find First and Last Position of Element in Sorted Array

## Metadata

* **LeetCode:** 34
* **Problem:** Find First and Last Position of Element in Sorted Array
* **Difficulty:** Medium
* **Topics:** Array, Binary Search
* **Pattern:** Boundary Binary Search
* **Key Technique:** Find First and Last Occurrence Separately
* **Key Pattern:** Lower Bound / Upper Bound
* **Optimal Complexity:** `O(log n)` time, `O(1)` space

---

## Problem

Given an array of integers `nums` sorted in **non-decreasing order**, find the starting and ending position of a given `target`.

If the target is not found, return:

```text
[-1, -1]
```

The algorithm must run in:

```text
O(log n)
```

### Example

```text
nums = [5, 7, 7, 8, 8, 10]
target = 8

Output = [3, 4]
```

Because `8` occurs from index `3` to index `4`.

---

# Approach 1 — Linear Search

## Idea

Traverse the entire array and find:

* The first index where `nums[i] == target`.
* The last index where `nums[i] == target`.

This is simple but does not satisfy the required `O(log n)` complexity.

## Dry Run

```text
nums = [5, 7, 7, 8, 8, 10]
target = 8
```

Scan:

```text
5 → no
7 → no
7 → no
8 → first occurrence → index 3
8 → last occurrence → index 4
10 → no
```

Answer:

```text
[3, 4]
```

## Algorithm

1. Initialize `first = -1` and `last = -1`.
2. Traverse the array.
3. When `nums[i] == target`:

   * If `first == -1`, set `first = i`.
   * Set `last = i`.
4. Return `{first, last}`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* Easy to implement.
* The sorted property of the array is completely unused.
* We can use binary search to find each boundary efficiently.

## Code

```cpp
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        int first = -1;
        int last = -1;

        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] == target) {
                if (first == -1) {
                    first = i;
                }

                last = i;
            }
        }

        return {first, last};
    }
};
```

---

# Approach 2 — Two Binary Searches

## Idea

Instead of searching for the target once, perform **two binary searches**:

1. Find the **first occurrence**.
2. Find the **last occurrence**.

### Finding First Occurrence

When:

```cpp
nums[mid] == target
```

we found a target, but there might be another target to the left.

So:

```cpp
right = mid - 1;
```

while remembering `mid` as a possible answer.

### Finding Last Occurrence

When:

```cpp
nums[mid] == target
```

there might be another target to the right.

So:

```cpp
left = mid + 1;
```

while remembering `mid` as a possible answer.

This is the key difference between the two binary searches.

## Dry Run

Consider:

```text
nums = [5, 7, 7, 8, 8, 10]
target = 8
```

### Finding First Occurrence

Start:

```text
left = 0
right = 5
```

#### Step 1

```text
mid = 2
nums[mid] = 7
```

Since:

```text
7 < 8
```

move right:

```text
left = 3
```

#### Step 2

```text
mid = 4
nums[mid] = 8
```

Found target.

Store:

```text
first = 4
```

But there might be another `8` to the left:

```text
right = 3
```

#### Step 3

```text
mid = 3
nums[mid] = 8
```

Found target again.

Update:

```text
first = 3
```

Move left:

```text
right = 2
```

Stop.

Therefore:

```text
first = 3
```

### Finding Last Occurrence

Start again:

```text
left = 0
right = 5
```

#### Step 1

```text
mid = 2
nums[mid] = 7
```

Move right:

```text
left = 3
```

#### Step 2

```text
mid = 4
nums[mid] = 8
```

Found target:

```text
last = 4
```

But there might be another `8` to the right:

```text
left = 5
```

#### Step 3

```text
mid = 5
nums[mid] = 10
```

Since:

```text
10 > 8
```

move left:

```text
right = 4
```

Stop.

Therefore:

```text
last = 4
```

Final answer:

```text
[3, 4]
```

## Algorithm

### First Occurrence

1. Set `left = 0`, `right = n - 1`.
2. While `left <= right`:

   * Calculate `mid`.
   * If `nums[mid] < target`, move right:

     ```cpp
     left = mid + 1;
     ```
   * If `nums[mid] > target`, move left:

     ```cpp
     right = mid - 1;
     ```
   * If `nums[mid] == target`:

     * Store `mid`.
     * Continue searching left:

       ```cpp
       right = mid - 1;
       ```

### Last Occurrence

1. Reset `left` and `right`.
2. Perform binary search again.
3. If `nums[mid] == target`:

   * Store `mid`.
   * Continue searching right:

     ```cpp
     left = mid + 1;
     ```

## Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

## Notes / Tips

The entire problem comes down to:

```text
Target found?
     ↓
Yes
     ↓
First occurrence → go LEFT
Last occurrence  → go RIGHT
```

### Important

Do **not** immediately return when `nums[mid] == target`.

For example:

```text
[1, 2, 2, 2, 2, 3]
       ↑
      mid
```

Finding one `2` does not tell us whether it is the first or last occurrence.

We must continue the binary search in the appropriate direction.

## Code

```cpp
class Solution {
public:
    int findFirst(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;
        int first = -1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                first = mid;
                right = mid - 1;
            }
            else if (nums[mid] < target) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }

        return first;
    }

    int findLast(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;
        int last = -1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                last = mid;
                left = mid + 1;
            }
            else if (nums[mid] < target) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }

        return last;
    }

    vector<int> searchRange(vector<int>& nums, int target) {
        int first = findFirst(nums, target);
        int last = findLast(nums, target);

        return {first, last};
    }
};
```

---

# Approach 3 — Lower Bound and Upper Bound

## Idea

This problem can also be understood using **boundary searches**.

### Lower Bound

Find the first position where:

```text
nums[i] >= target
```

This gives the potential first occurrence.

### Upper Bound

Find the first position where:

```text
nums[i] > target
```

Then:

```text
last occurrence = upperBound - 1
```

For:

```text
nums = [5, 7, 7, 8, 8, 10]
target = 8
```

we get:

```text
lower_bound(8) = 3
upper_bound(8) = 5
```

Therefore:

```text
first = 3
last = 5 - 1 = 4
```

## Dry Run

### Lower Bound

Find the first index with:

```text
nums[i] >= 8
```

```text
[5, 7, 7, 8, 8, 10]
          ↑
         3
```

Result:

```text
lowerBound = 3
```

### Upper Bound

Find the first index with:

```text
nums[i] > 8
```

```text
[5, 7, 7, 8, 8, 10]
                   ↑
                   5
```

Result:

```text
upperBound = 5
```

Therefore:

```text
first = 3
last = 5 - 1 = 4
```

Answer:

```text
[3, 4]
```

## Algorithm

1. Find the first index where `nums[mid] >= target`.
2. If that index is outside the array or `nums[index] != target`, return `{-1, -1}`.
3. Find the first index where `nums[mid] > target`.
4. Return:

   ```text
   {lowerBound, upperBound - 1}
   ```

## Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

## Notes / Tips

This approach is especially useful because **lower bound and upper bound are reusable binary-search patterns**.

### Lower Bound

```text
first position where nums[i] >= target
```

### Upper Bound

```text
first position where nums[i] > target
```

For this problem:

```text
first = lower_bound(target)
last  = upper_bound(target) - 1
```

This pattern appears frequently in sorted-array problems.

## Code

```cpp
class Solution {
public:
    int lowerBound(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] >= target) {
                right = mid;
            }
            else {
                left = mid + 1;
            }
        }

        return left;
    }

    int upperBound(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size();

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] > target) {
                right = mid;
            }
            else {
                left = mid + 1;
            }
        }

        return left;
    }

    vector<int> searchRange(vector<int>& nums, int target) {
        int first = lowerBound(nums, target);

        if (first == nums.size() || nums[first] != target) {
            return {-1, -1};
        }

        int last = upperBound(nums, target) - 1;

        return {first, last};
    }
};
```

---

# Comparison of Approaches

| Approach            |       Time |  Space | Main Idea                       |
| ------------------- | ---------: | -----: | ------------------------------- |
| Linear Search       |     `O(n)` | `O(1)` | Scan entire array               |
| Two Binary Searches | `O(log n)` | `O(1)` | Search left/right boundaries    |
| Lower + Upper Bound | `O(log n)` | `O(1)` | Find `>= target` and `> target` |

---

# Key Takeaway

The most important pattern is:

```text
Find First + Last Occurrence
            ↓
      Binary Search
            ↓
      Target found?
       ↙         ↘
   First         Last
     ↓             ↓
Go LEFT         Go RIGHT
```

### Key Template — First Occurrence

```cpp
int first = -1;

while (left <= right) {
    int mid = left + (right - left) / 2;

    if (nums[mid] == target) {
        first = mid;
        right = mid - 1;
    }
    else if (nums[mid] < target) {
        left = mid + 1;
    }
    else {
        right = mid - 1;
    }
}
```

### Key Template — Last Occurrence

```cpp
int last = -1;

while (left <= right) {
    int mid = left + (right - left) / 2;

    if (nums[mid] == target) {
        last = mid;
        left = mid + 1;
    }
    else if (nums[mid] < target) {
        left = mid + 1;
    }
    else {
        right = mid - 1;
    }
}
```

### Most Reusable Pattern

```text
Lower Bound → first index where nums[i] >= target

Upper Bound → first index where nums[i] > target

First occurrence = lower bound
Last occurrence  = upper bound - 1
```

**Remember:** When a sorted array contains duplicates and you need a specific occurrence, **don't stop when you find the target**. Keep searching toward the required boundary.

**Key Pattern:** **Boundary Binary Search → Find Leftmost / Rightmost Occurrence.**
