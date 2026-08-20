# LeetCode 88 — Merge Sorted Array

## Metadata

* **LeetCode:** 88
* **Problem:** Merge Sorted Array
* **Difficulty:** Easy
* **Topics:** Array, Two Pointers, Sorting
* **Pattern:** Two Pointers — Merge from End
* **Key Technique:** Fill `nums1` from right to left
* **Key Pattern:** Reverse Two-Pointer Merge
* **Key Template:** Two Pointers
* **Optimal Complexity:** `O(m + n)` time, `O(1)` extra space

---

## Problem

You are given two sorted arrays:

```text
nums1
nums2
```

`nums1` has enough space to hold all elements of both arrays.

* `m` = number of actual elements in `nums1`
* `n` = number of elements in `nums2`

Merge `nums2` into `nums1` so that `nums1` becomes sorted.

Example:

```text
nums1 = [1,2,3,0,0,0]
m = 3

nums2 = [2,5,6]
n = 3
```

Output:

```text
[1,2,2,3,5,6]
```

---

## Approach — Two Pointers from the End

### Idea

A normal merge starts from the beginning.

But here, `nums1` already contains its elements at the beginning and has empty space at the end.

If we merge from the beginning, we may **overwrite elements in `nums1` that we still need**.

Instead, merge from **right to left**.

Use three pointers:

```text
i = m - 1       → last actual element of nums1
j = n - 1       → last element of nums2
k = m + n - 1   → last position of nums1
```

Compare:

```text
nums1[i] vs nums2[j]
```

Put the larger element at `nums1[k]`.

### Why From the End?

Example:

```text
nums1 = [1,2,3,0,0,0]
nums2 = [2,5,6]
```

The largest element is `6`.

There is already an empty position at the end:

```text
[1,2,3,0,0,6]
```

Then place `5`:

```text
[1,2,3,0,5,6]
```

Then `3`:

```text
[1,2,3,3,5,6]
```

This avoids overwriting useful elements.

### Dry Run

```text
nums1 = [1,2,3,0,0,0]
nums2 = [2,5,6]

i = 2
j = 2
k = 5
```

#### Step 1

```text
nums1[i] = 3
nums2[j] = 6
```

`6` is larger:

```text
nums1[5] = 6
```

Move:

```text
i = 2
j = 1
k = 4
```

Array:

```text
[1,2,3,0,0,6]
```

#### Step 2

```text
3 vs 5
```

Place `5`:

```text
[1,2,3,0,5,6]
```

#### Step 3

```text
3 vs 2
```

Place `3`:

```text
[1,2,3,3,5,6]
```

Now `i` moves past the valid portion of `nums1`.

#### Remaining Elements

`nums2` still contains:

```text
[2]
```

Place it:

```text
[1,2,2,3,5,6]
```

Final answer:

```text
[1,2,2,3,5,6]
```

### Algorithm

1. Set:

   ```cpp
   i = m - 1
   j = n - 1
   k = m + n - 1
   ```
2. While both arrays still have elements:

   * Compare `nums1[i]` and `nums2[j]`.
   * Put the larger element at `nums1[k]`.
   * Move the corresponding pointer backward.
   * Decrement `k`.
3. If elements remain in `nums2`, copy them into `nums1`.
4. No need to explicitly copy remaining elements of `nums1` because they are already in their correct positions.

### Complexity

* **Time:** `O(m + n)`
* **Space:** `O(1)`

### Notes / Tips

* This is a classic **two-pointer** problem.
* The key trick is:

  ```text
  Merge from the END.
  ```
* `k` always represents the next position to fill.
* Only `nums2` needs a final copy.
* If `nums2` becomes empty first, the remaining elements of `nums1` are already correctly placed.
* Never use:

  ```cpp
  sort(nums1.begin(), nums1.end());
  ```

  because the problem specifically expects an efficient merge.
* This same reverse-merge technique is useful whenever an array has **empty space at the end**.

### Code

```cpp
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) {
        int i = m - 1;
        int j = n - 1;
        int k = m + n - 1;

        while (i >= 0 && j >= 0) {
            if (nums1[i] > nums2[j]) {
                nums1[k] = nums1[i];
                i--;
            }
            else {
                nums1[k] = nums2[j];
                j--;
            }

            k--;
        }

        // Copy remaining elements from nums2
        while (j >= 0) {
            nums1[k] = nums2[j];
            j--;
            k--;
        }
    }
};
```

---

## Key Takeaway

The main idea is:

```text
nums1: [1, 2, 3, _, _, _]
                    ↑
                  fill here

Compare from the END
        ↓
Place larger element
        ↓
Move pointer backward
```

Remember the three pointers:

```text
i → last valid element of nums1
j → last element of nums2
k → last position of nums1
```

**Pattern:**

> When merging into an array that has empty space at the end, merge from the back to avoid overwriting unprocessed elements.
