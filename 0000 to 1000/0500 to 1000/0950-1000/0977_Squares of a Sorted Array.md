# LeetCode 977 — Squares of a Sorted Array

## Metadata

* **LeetCode:** 977
* **Problem:** Squares of a Sorted Array
* **Difficulty:** Easy
* **Topics:** Array, Two Pointers, Sorting
* **Pattern:** Two Pointers from the Ends
* **Key Technique:** The largest square always comes from one of the two ends (most negative or most positive), so fill the result array back to front by comparing absolute values at both pointers
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an integer array `nums` sorted in non-decreasing order, return an array of the squares of each number, also sorted in non-decreasing order.

---

## Approaches

1. **Brute Force — Square Then Sort**
2. **Optimal — Two Pointers from the Ends**

---

# Approach 1 — Brute Force / Square Then Sort

## Idea

Square every element directly, then sort the resulting array to restore ascending order (since squaring can reorder negative values).

## Dry Run

```text
nums = [-4, -1, 0, 3, 10]
```

Square every element:

```text
[16, 1, 0, 9, 100]
```

Sort:

```text
[0, 1, 9, 16, 100]
```

## Algorithm

1. Create a `result` array by squaring every element of `nums`.
2. Sort `result` in ascending order.
3. Return `result`.

## Complexity

* **Time:** `O(n log n)`

  * Dominated by the sorting step.
* **Space:** `O(n)`

  * For the result array (or `O(log n)` extra for the sort's recursion, depending on implementation, beyond the required output).

## Notes / Tips

* Correct and simple, but ignores the fact that the input is already sorted — sorting again throws away that structure.
* The only reason squaring disrupts the order is that negative numbers can produce large squares — this is exactly the insight the two-pointer approach exploits.

## Code

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        vector<int> result;

        for (int num : nums) {
            result.push_back(num * num);
        }

        sort(result.begin(), result.end());

        return result;
    }
};
```

---

# Approach 2 — Optimal / Two Pointers from the Ends

## Idea

Since `nums` is sorted, the largest-magnitude values sit at the two ends: the most negative number (largest square when squared) is at the start, and the most positive number is at the end. Compare the absolute values at both ends, place the larger square at the back of the result array, and move the corresponding pointer inward — building the result from largest to smallest.

## Dry Run

```text
nums = [-4, -1, 0, 3, 10]
```

`left = 0` (value -4), `right = 4` (value 10), fill position `4` (last index) first:

```text
|-4| = 4, |10| = 10 → 10 is bigger → result[4] = 100, right-- → right=3
```

`left = 0` (value -4), `right = 3` (value 3), fill position `3`:

```text
|-4| = 4, |3| = 3 → 4 is bigger → result[3] = 16, left++ → left=1
```

`left = 1` (value -1), `right = 3` (value 3), fill position `2`:

```text
|-1| = 1, |3| = 3 → 3 is bigger → result[2] = 9, right-- → right=2
```

`left = 1` (value -1), `right = 2` (value 0), fill position `1`:

```text
|-1| = 1, |0| = 0 → 1 is bigger → result[1] = 1, left++ → left=2
```

`left = 2 == right = 2`, fill position `0`:

```text
result[0] = 0*0 = 0
```

Final:

```text
[0, 1, 9, 16, 100]
```

## Algorithm

1. Initialize `left = 0`, `right = n - 1`, and a `result` array of size `n`.
2. For `i` from `n - 1` down to `0`:

   * Compare `abs(nums[left])` and `abs(nums[right])`.
   * If `abs(nums[left]) > abs(nums[right])`, set `result[i] = nums[left] * nums[left]` and increment `left`.
   * Otherwise, set `result[i] = nums[right] * nums[right]` and decrement `right`.
3. Return `result`.

## Complexity

* **Time:** `O(n)`

  * Single pass, `left` and `right` together move exactly `n` steps total.
* **Space:** `O(1)`

  * Aside from the required output array, only two pointers are used — no sorting or extra structures.

## Notes / Tips

* Filling the result array from the back (largest values first) is essential — since the two-pointer comparison naturally produces the largest remaining square at each step, building front-to-back would require extra bookkeeping to reverse the order afterward.
* This is a variant of the classic two-pointer "merge from both ends" pattern — same underlying idea as merging two sorted halves, just applied to a single array split conceptually into a negative half and a non-negative half.
* Common mistake: using `<` instead of `<=`/`>` consistently when `left == right` — since both pointers can point to the same element in the final iteration, the comparison must still handle that case correctly (it does here, since either branch produces the same correct result when `left == right`).

## Code

```cpp
class Solution {
public:
    vector<int> sortedSquares(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n);
        int left = 0, right = n - 1;

        for (int i = n - 1; i >= 0; i--) {
            if (abs(nums[left]) > abs(nums[right])) {
                result[i] = nums[left] * nums[left];
                left++;
            } else {
                result[i] = nums[right] * nums[right];
                right--;
            }
        }

        return result;
    }
};
```

---

## Key Template

```text
left = 0, right = n - 1
result = array of size n

for i = n-1 down to 0:
    if abs(nums[left]) > abs(nums[right]):
        result[i] = nums[left] * nums[left]
        left += 1
    else:
        result[i] = nums[right] * nums[right]
        right -= 1

return result
```