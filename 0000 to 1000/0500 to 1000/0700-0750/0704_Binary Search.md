# Binary Search

## Problem

Given a sorted array `nums` and a target value, return the index of `target` if it exists in the array.

If `target` is not present, return `-1`.

Example:

```text
nums = [-1,0,3,5,9,12]
target = 9

Output = 4
```

---

## Approach 1: Binary Search

### Idea

Since the array is **sorted**, repeatedly divide the search space into half.

Maintain:

* `left` → beginning of search space
* `right` → end of search space
* `mid` → middle element

For every `mid`:

* If `nums[mid] == target` → found.
* If `nums[mid] < target` → target must be on the right.
* If `nums[mid] > target` → target must be on the left.

### Dry Run

```text
nums = [-1,0,3,5,9,12]
target = 9

left = 0, right = 5

mid = 2
nums[2] = 3 < 9
→ search right half

left = 3, right = 5

mid = 4
nums[4] = 9
→ found

Answer = 4
```

### Algorithm

1. Initialize `left = 0` and `right = nums.size() - 1`.
2. While `left <= right`:

   * Calculate `mid = left + (right - left) / 2`.
   * If `nums[mid] == target`, return `mid`.
   * If `nums[mid] < target`, set `left = mid + 1`.
   * Otherwise, set `right = mid - 1`.
3. If the loop ends, the target does not exist.
4. Return `-1`.

### Complexity

* Time: `O(log n)`
* Space: `O(1)`

### Code

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
            else if (nums[mid] < target) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }

        return -1;
    }
};
```

### Notes / Tips

* Binary search requires a **sorted search space**.
* After every comparison, eliminate half of the remaining elements.
* Use:

```text
left + (right - left) / 2
```

instead of `(left + right) / 2` to avoid potential integer overflow.

* Remember the boundary updates:

  * Target bigger → `left = mid + 1`
  * Target smaller → `right = mid - 1`
* The standard condition is `left <= right` when both boundaries are inclusive.

### Key Template

```text
left = 0
right = n - 1

while left <= right:
    mid = left + (right - left) / 2

    if nums[mid] == target:
        return mid
    else if nums[mid] < target:
        left = mid + 1
    else:
        right = mid - 1

return -1
```
