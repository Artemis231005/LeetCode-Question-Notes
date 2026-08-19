# LeetCode 27 — Remove Element

## Metadata

* **LeetCode:** 27
* **Problem:** Remove Element
* **Difficulty:** Easy
* **Topics:** Array, Two Pointers
* **Pattern:** In-place Array Modification
* **Key Technique:** Two Pointers
* **Key Pattern:** Fast & Slow Pointer
* **Optimal Complexity:** `O(n)` time, `O(1)` space

---

## Problem

Given an integer array `nums` and an integer `val`, remove all occurrences of `val` **in-place**.

The order of the remaining elements does not matter.

Return the number of elements `k` that are not equal to `val`.

The first `k` positions of `nums` must contain the elements that are not equal to `val`.

---

# Approach 1 — Brute Force

## Idea

Whenever `val` is found, remove that element by shifting all elements after it one position to the left.

This works, but every removal can require shifting many elements, making it inefficient.

## Dry Run

```text
nums = [3, 2, 2, 3]
val = 3

Find 3 at index 0:
Shift elements left

[2, 2, 3, 3]

Find 3 at index 2:
Shift elements left

[2, 2, 3, 3]

Remaining elements = [2, 2]
k = 2
```

## Algorithm

1. Start from index `0`.
2. If `nums[i] == val`:

   * Shift every element after `i` one position to the left.
   * Decrease the effective array size.
   * Check the same index again because a new element has moved there.
3. Otherwise, move to the next index.
4. Return the final effective size.

## Complexity

* **Time:** `O(n²)` in the worst case
* **Space:** `O(1)`

## Notes / Tips

* The shifting operation is the reason this approach becomes `O(n²)`.
* We can avoid shifting completely using two pointers.

## Code

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int n = nums.size();
        int i = 0;

        while (i < n) {
            if (nums[i] == val) {
                for (int j = i; j < n - 1; j++) {
                    nums[j] = nums[j + 1];
                }

                n--;
            }
            else {
                i++;
            }
        }

        return n;
    }
};
```

---

# Approach 2 — Two Pointers / Fast & Slow Pointer

## Idea

Instead of actually deleting elements, overwrite every occurrence of `val` with the next valid element.

Use two pointers:

* `read` → scans every element.
* `write` → points to the position where the next valid element should be placed.

Whenever `nums[read] != val`, copy it to `nums[write]` and increment `write`.

At the end, `write` is the number of elements that were not equal to `val`.

## Dry Run

```text
nums = [3, 2, 2, 3]
val = 3

read = 0
nums[0] = 3 → skip

read = 1
nums[1] = 2 → nums[write] = 2
[2, 2, 2, 3]
write = 1

read = 2
nums[2] = 2 → nums[write] = 2
[2, 2, 2, 3]
write = 2

read = 3
nums[3] = 3 → skip

Return write = 2
```

Only the first `2` elements matter:

```text
[2, 2]
```

## Algorithm

1. Initialize `write = 0`.
2. Traverse the array using `read`.
3. If `nums[read] != val`:

   * Store `nums[read]` at `nums[write]`.
   * Increment `write`.
4. If `nums[read] == val`, simply skip it.
5. After processing the entire array, return `write`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* This is the standard optimal approach.
* We do **not** actually resize the vector.
* Only the first `k` elements matter after the operation.
* The relative order of remaining elements is preserved.
* This is a classic **Fast & Slow Pointer** pattern:

  * `read` = fast pointer
  * `write` = slow pointer
* The same pattern is useful for problems such as **Remove Duplicates from Sorted Array**.

## Code

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int write = 0;

        for (int read = 0; read < nums.size(); read++) {
            if (nums[read] != val) {
                nums[write] = nums[read];
                write++;
            }
        }

        return write;
    }
};
```

---

# Approach 3 — Two Pointers from Both Ends

## Idea

Because the problem says that the order of the remaining elements **does not matter**, we can optimize further in terms of writes.

Use:

* `left` → scans from the beginning.
* `right` → points to the end of the valid portion.

When `nums[left] == val`, replace it with the element at `right` and decrease `right`.

This avoids preserving the order of elements.

## Dry Run

```text
nums = [3, 2, 2, 3]
val = 3

left = 0
right = 3

nums[left] == 3
Replace nums[0] with nums[3]

[3, 2, 2, 3] → [3, 2, 2, 3]
right = 2

nums[left] == 3
Replace nums[0] with nums[2]

[2, 2, 2, 3]
right = 1

Now left > right

k = right + 1 = 2
```

The first two elements are:

```text
[2, 2]
```

## Algorithm

1. Initialize `left = 0` and `right = nums.size() - 1`.
2. While `left <= right`:

   * If `nums[left] == val`:

     * Replace it with `nums[right]`.
     * Decrease `right`.
   * Otherwise:

     * Increment `left`.
3. Return `right + 1`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* This approach is useful when **order does not matter**.
* It can reduce unnecessary writes because elements equal to `val` are replaced directly from the end.
* If maintaining the original order is important, use the Fast & Slow Pointer approach instead.
* For this problem, both two-pointer approaches satisfy the optimal `O(n)` time and `O(1)` space requirement.

## Code

```cpp
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int left = 0;
        int right = nums.size() - 1;

        while (left <= right) {
            if (nums[left] == val) {
                nums[left] = nums[right];
                right--;
            }
            else {
                left++;
            }
        }

        return right + 1;
    }
};
```

---

# Key Takeaway

The most important pattern here is:

```text
Fast pointer → scans the array
Slow pointer → stores valid elements
```

For **Remove Element**, the cleanest solution is:

```cpp
int write = 0;

for (int read = 0; read < nums.size(); read++) {
    if (nums[read] != val) {
        nums[write++] = nums[read];
    }
}

return write;
```

**Remember:** When asked to modify an array **in-place** while keeping only elements satisfying some condition, think **Fast & Slow Pointer**.
