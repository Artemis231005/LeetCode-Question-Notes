# LeetCode 81 — Search in Rotated Sorted Array II

## Metadata

* **LeetCode:** 81
* **Problem:** Search in Rotated Sorted Array II
* **Difficulty:** Medium
* **Topics:** Array, Binary Search
* **Pattern:** Modified Binary Search
* **Key Technique:** Handle duplicates when determining the sorted half
* **Key Pattern:** Rotated Binary Search with Duplicates
* **Key Template:** Binary Search
* **Optimal Complexity:** `O(log n)` average, `O(n)` worst case
* **Space Complexity:** `O(1)`

---

## Problem

You are given an integer array `nums` sorted in **non-decreasing order** that has been rotated at an unknown position.

The array may contain **duplicates**.

Given a target value, return `true` if `target` exists in `nums`, otherwise return `false`.

Example:

```text
nums = [2, 5, 6, 0, 0, 1, 2]
target = 0

Output: true
```

Another example:

```text
nums = [2, 5, 6, 0, 0, 1, 2]
target = 3

Output: false
```

---

## Approach — Modified Binary Search

### Idea

Normally in a rotated sorted array, at least **one half is sorted**.

For:

```text id="5e1v9x"
[4,5,6,7,0,1,2]
```

If:

```text id="qkl4u0"
mid = 3
```

then:

```text id="k9v8r3"
left half  → [4,5,6,7]  sorted
right half → [0,1,2]    sorted
```

We determine which half is sorted and check whether the target lies inside that range.

### The New Problem: Duplicates

With duplicates, we can have:

```text id="3z8m5y"
nums = [1, 0, 1, 1, 1]
```

Suppose:

```text id="k1xg6a"
nums[left] == nums[mid] == nums[right]
```

We cannot determine which half is sorted.

Therefore, shrink the search space:

```cpp id="6d1x4j"
left++;
right--;
```

This is the crucial difference from **LeetCode 33**.

### Dry Run

Consider:

```text id="z2q4k7"
nums = [2, 5, 6, 0, 0, 1, 2]
target = 0
```

Initial:

```text id="k4n7dx"
left = 0
right = 6
mid = 3
nums[mid] = 0
```

Since:

```text id="4c4xmg"
nums[mid] == target
```

return:

```text id="5p1jvq"
true
```

### Duplicate Case

Consider:

```text id="gq9p8x"
nums = [1, 1, 1, 0, 1]
target = 0
```

Suppose:

```text id="k5w2sb"
left = 0
mid = 2
right = 4

nums[left] = 1
nums[mid]  = 1
nums[right] = 1
```

All three are equal.

We cannot determine which side is sorted.

So:

```text id="1m6k3r"
left++
right--
```

Now search the smaller range.

### Algorithm

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
3. If `nums[mid] == target`, return `true`.
4. If:

   ```cpp
   nums[left] == nums[mid] && nums[mid] == nums[right]
   ```

   then duplicates prevent us from identifying the sorted half:

   ```cpp
   left++;
   right--;
   ```
5. Otherwise, check whether the **left half is sorted**:

   ```cpp
   nums[left] <= nums[mid]
   ```
6. If the left half is sorted:

   * Check whether target lies inside:

     ```cpp
     nums[left] <= target && target < nums[mid]
     ```
   * If yes, search left:

     ```cpp
     right = mid - 1;
     ```
   * Otherwise, search right:

     ```cpp
     left = mid + 1;
     ```
7. Otherwise, the **right half is sorted**.
8. Check whether target lies inside:

   ```cpp
   nums[mid] < target && target <= nums[right]
   ```
9. If yes, search right; otherwise search left.
10. If the loop ends, return `false`.

### Complexity

* **Average Time:** `O(log n)`
* **Worst-case Time:** `O(n)`
* **Space:** `O(1)`

Why can it become `O(n)`?

Because duplicates can force us to repeatedly do:

```cpp
left++;
right--;
```

For example:

```text id="av2v1q"
[1,1,1,1,1,1,1]
```

There is not enough information to eliminate half the search space at every step.

### Notes / Tips

* This is an extension of **LeetCode 33 — Search in Rotated Sorted Array**.
* The key difference is **duplicates**.
* Without duplicates:

  ```text
  one half is always clearly sorted
  ```
* With duplicates:

  ```cpp
  nums[left] == nums[mid] == nums[right]
  ```

  means we cannot identify the sorted half.
* In that situation:

  ```cpp
  left++;
  right--;
  ```
* Use `<=` when checking whether a half is sorted:

  ```cpp
  nums[left] <= nums[mid]
  ```
* When checking the target range, be careful with boundaries:

  ```cpp
  nums[left] <= target && target < nums[mid]
  ```

  and:

  ```cpp
  nums[mid] < target && target <= nums[right]
  ```
* Do **not** blindly copy the solution for LeetCode 33; the duplicate-handling case is essential.

### Code

```cpp id="b4v1as"
class Solution {
public:
    bool search(vector<int>& nums, int target) {
        int left = 0;
        int right = nums.size() - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] == target) {
                return true;
            }

            // Cannot determine which half is sorted
            if (nums[left] == nums[mid] &&
                nums[mid] == nums[right]) {
                left++;
                right--;
            }

            // Left half is sorted
            else if (nums[left] <= nums[mid]) {
                if (nums[left] <= target &&
                    target < nums[mid]) {
                    right = mid - 1;
                }
                else {
                    left = mid + 1;
                }
            }

            // Right half is sorted
            else {
                if (nums[mid] < target &&
                    target <= nums[right]) {
                    left = mid + 1;
                }
                else {
                    right = mid - 1;
                }
            }
        }

        return false;
    }
};
```

---

## Key Takeaway

### Normal Rotated Binary Search

```text
One half is always sorted
        ↓
Check whether target belongs there
        ↓
Search that half
```

### With Duplicates

```text
left == mid == right
        ↓
Cannot identify sorted half
        ↓
left++
right--
```

The most important condition to remember is:

```cpp
if (nums[left] == nums[mid] && nums[mid] == nums[right]) {
    left++;
    right--;
}
```

**Pattern:**

> Rotated Binary Search + Duplicates = Identify sorted half, but shrink both ends when duplicates make the decision ambiguous.
