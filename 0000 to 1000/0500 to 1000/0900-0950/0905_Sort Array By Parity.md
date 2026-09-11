# LeetCode 905 — Sort Array By Parity

## Metadata

* **LeetCode:** 905
* **Problem:** Sort Array By Parity
* **Difficulty:** Easy
* **Topics:** Array, Two Pointers, Sorting
* **Pattern:** Two Pointers from the Ends (Partitioning)
* **Key Technique:** Swap elements between a left pointer (seeking an odd value) and a right pointer (seeking an even value) to partition the array in place
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an integer array `nums`, move all the even integers to the beginning of the array, followed by all the odd integers. Any order within each group is acceptable. Return the resulting array.

---

## Approaches

1. **Brute Force — Separate Into Two Lists, Then Combine**
2. **Optimal — Two Pointers from the Ends (In-Place Partition)**

---

# Approach 1 — Brute Force / Separate Into Two Lists, Then Combine

## Idea

Scan through the array once, placing every even number into one list and every odd number into another. Concatenate the even list followed by the odd list to form the result.

## Dry Run

```text
nums = [3, 1, 2, 4]
```

Scan and separate:

```text
3 → odd → odds = [3]
1 → odd → odds = [3, 1]
2 → even → evens = [2]
4 → even → evens = [2, 4]
```

Combine:

```text
evens + odds = [2, 4, 3, 1]
```

## Algorithm

1. Create two empty lists: `evens` and `odds`.
2. For each number in `nums`:

   * If even, append to `evens`.
   * Otherwise, append to `odds`.
3. Concatenate `evens` followed by `odds` and return.

## Complexity

* **Time:** `O(n)`

  * Single pass to separate, one more pass to combine.
* **Space:** `O(n)`

  * For the two new lists, even though the problem can be solved without any extra array.

## Notes / Tips

* Correct and easy to follow, but allocates extra memory that isn't needed — the array can be partitioned directly in place instead.
* Preserves relative order within each group (stable), which the two-pointer swap approach does not guarantee — worth knowing if stability is ever required by a similar problem variant.

## Code

```cpp
class Solution {
public:
    vector<int> sortArrayByParity(vector<int>& nums) {
        vector<int> evens, odds;

        for (int num : nums) {
            if (num % 2 == 0) {
                evens.push_back(num);
            } else {
                odds.push_back(num);
            }
        }

        evens.insert(evens.end(), odds.begin(), odds.end());
        return evens;
    }
};
```

---

# Approach 2 — Optimal / Two Pointers from the Ends (In-Place Partition)

## Idea

Use two pointers starting at opposite ends of the array. Advance `left` while it points to an even number (already in the right place), and advance `right` while it points to an odd number (already in the right place). When both pointers stop on a misplaced value (`left` on an odd number, `right` on an even number), swap them — this fixes both at once — then continue.

## Dry Run

```text
nums = [3, 1, 2, 4]
```

`left = 0` (value 3, odd → misplaced), `right = 3` (value 4, even → misplaced):

```text
swap nums[0] and nums[3] → nums = [4, 1, 2, 3]
left++ → left=1, right-- → right=2
```

`left = 1` (value 1, odd → misplaced), `right = 2` (value 2, even → misplaced):

```text
swap nums[1] and nums[2] → nums = [4, 2, 1, 3]
left++ → left=2, right-- → right=1
```

`left > right` (`2 > 1`) → loop ends.

Final:

```text
[4, 2, 1, 3]
```

## Algorithm

1. Initialize `left = 0`, `right = n - 1`.
2. While `left < right`:

   * If `nums[left]` is even, it's already correctly placed — increment `left` and continue.
   * Else if `nums[right]` is odd, it's already correctly placed — decrement `right` and continue.
   * Otherwise, `nums[left]` is odd and `nums[right]` is even — swap them, then increment `left` and decrement `right`.
3. Return `nums`.

## Complexity

* **Time:** `O(n)`

  * `left` and `right` together move at most `n` steps total across the whole array.
* **Space:** `O(1)`

  * Done entirely in place — only two pointers used, no extra arrays.

## Notes / Tips

* This is the same "partition around a condition" shape used in quicksort's partition step (e.g. Dutch National Flag problem) — anytime elements need to be grouped into two categories in place, two pointers from opposite ends is the standard technique.
* Order within each group isn't preserved (unlike Approach 1) — this is fine here since the problem explicitly allows any order within evens/odds.
* Common mistake: swapping and moving both pointers unconditionally on every iteration instead of first checking whether either pointer already sits on a correctly placed value — that check is what avoids unnecessary swaps and keeps the total number of swaps minimal.

## Code

```cpp
class Solution {
public:
    vector<int> sortArrayByParity(vector<int>& nums) {
        int left = 0, right = nums.size() - 1;

        while (left < right) {
            if (nums[left] % 2 == 0) {
                left++;
            } else if (nums[right] % 2 == 1) {
                right--;
            } else {
                swap(nums[left], nums[right]);
                left++;
                right--;
            }
        }

        return nums;
    }
};
```

---

## Key Template

```text
left = 0, right = n - 1

while left < right:
    if nums[left] % 2 == 0:
        left += 1
    elif nums[right] % 2 == 1:
        right -= 1
    else:
        swap(nums[left], nums[right])
        left += 1
        right -= 1

return nums
```