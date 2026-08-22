# Single Element in a Sorted Array

## Problem

Given a sorted array where every element appears exactly twice except for one element that appears only once, return the single element.

The solution must run in `O(log n)` time and `O(1)` space.

Example:

```text
nums = [1,1,2,3,3,4,4,8,8]
Output = 2
```

---

## Approach 1: Binary Search

### Idea

Before the single element, pairs start at **even indices**:

```text
[1,1] [2,2] [3,3]
 0 1   2 3   4 5
```

After the single element, this pattern shifts, so pairs start at **odd indices**.

For a middle index `mid`:

* If `mid` is even, compare `nums[mid]` with `nums[mid + 1]`.
* If they are equal, the single element is on the **right**.
* Otherwise, it is on the **left**, including `mid`.
* If `mid` is odd, compare `nums[mid]` with `nums[mid - 1]`.
* If they are equal, the single element is on the **right**.
* Otherwise, it is on the **left**.

A cleaner way is to always make `mid` even and compare `nums[mid]` with `nums[mid + 1]`.

### Dry Run

```text
nums = [1,1,2,3,3,4,4,8,8]

left = 0
right = 8

mid = 4
nums[4] == nums[5]?
3 != 4

→ pair is broken here
→ single element is at mid or to the left

right = 4

mid = 2
nums[2] == nums[3]?
2 != 3

right = 2

mid = 1
make mid even → mid = 0

nums[0] == nums[1]
1 == 1

→ single element is to the right

left = 2

left == right
→ nums[2] = 2
```

### Algorithm

1. Set `left = 0` and `right = n - 1`.
2. While `left < right`:

   * Calculate `mid`.
   * If `mid` is odd, decrement it by `1` to make it even.
   * Compare `nums[mid]` and `nums[mid + 1]`.
3. If they are equal:

   * This pair is valid.
   * Move `left = mid + 2`.
4. Otherwise:

   * The single element is at `mid` or before it.
   * Move `right = mid`.
5. Return `nums[left]`.

### Complexity

* Time: `O(log n)`
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    int singleNonDuplicate(vector<int>& nums) {
        int left = 0;
        int right = nums.size() - 1;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (mid % 2 == 1) {
                mid--;
            }

            if (nums[mid] == nums[mid + 1]) {
                left = mid + 2;
            }
            else {
                right = mid;
            }
        }

        return nums[left];
    }
};
```

### Notes / Tips

* The **sorted order** is what allows binary search.
* Before the single element, pairs are `(even, odd)`.
* After the single element, the pairing pattern becomes `(odd, even)`.
* Always make `mid` even, then simply compare `nums[mid]` and `nums[mid + 1]`.
* If the pair is correct, move right.
* If the pair is broken, move left.
* This is a classic **binary search on a pattern** problem.

### Key Template

```text
left = 0
right = n - 1

while left < right:
    mid = middle

    if mid is odd:
        mid--

    if nums[mid] == nums[mid + 1]:
        left = mid + 2
    else:
        right = mid

return nums[left]
```
