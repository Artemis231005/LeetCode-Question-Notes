# LeetCode 4 — Median of Two Sorted Arrays

## Metadata

* **LeetCode:** 4
* **Problem:** Median of Two Sorted Arrays
* **Difficulty:** Hard
* **Topics:** Array, Binary Search, Divide and Conquer
* **Pattern:** Binary Search on Partition
* **Key Pattern:** Find the correct partition such that the left half contains exactly half of the elements and every left-side element is `<=` every right-side element
* **Key Template:** Binary Search on Partition
* **Key Technique:** Partition the smaller array and derive the partition in the larger array
* **Optimal Complexity:** `O(log(min(m, n)))` time, `O(1)` space

---

# Approaches

1. **Brute Force — Merge Both Arrays**
2. **Better — Two Pointers Without Extra Array**
3. **Optimal — Binary Search on Partition**

---

# Approach 1 — Brute Force / Merge Both Arrays

## Idea

Merge the two sorted arrays into one sorted array.

Once we have the complete sorted array, finding the median is straightforward.

For a total size `n`:

* If `n` is odd:

```text
median = arr[n / 2]
```

* If `n` is even:

```text
median = (arr[n / 2 - 1] + arr[n / 2]) / 2
```

This approach is simple but uses extra space.

## Dry Run

```text
nums1 = [1, 3]
nums2 = [2, 4]
```

Merge:

```text
[1, 2, 3, 4]
```

Total elements:

```text
n = 4
```

Median:

```text
(2 + 3) / 2 = 2.5
```

## Algorithm

1. Create an empty array `merged`.
2. Use two pointers to merge `nums1` and `nums2`.
3. Append remaining elements from either array.
4. Find the middle element(s).
5. Return the median.

## Complexity

Let:

```text
m = nums1.size()
n = nums2.size()
```

* **Time:** `O(m + n)`
* **Space:** `O(m + n)`

## Notes / Tips

* This works because both input arrays are already sorted.
* The main disadvantage is storing the complete merged array.
* It does not satisfy the required `O(log(min(m, n)))` complexity.

## Code

```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        vector<int> merged;

        int i = 0;
        int j = 0;

        while (i < nums1.size() && j < nums2.size()) {
            if (nums1[i] <= nums2[j]) {
                merged.push_back(nums1[i]);
                i++;
            }
            else {
                merged.push_back(nums2[j]);
                j++;
            }
        }

        while (i < nums1.size()) {
            merged.push_back(nums1[i]);
            i++;
        }

        while (j < nums2.size()) {
            merged.push_back(nums2[j]);
            j++;
        }

        int n = merged.size();

        if (n % 2 == 1) {
            return merged[n / 2];
        }

        return (merged[n / 2 - 1] + merged[n / 2]) / 2.0;
    }
};
```

---

# Approach 2 — Better / Two Pointers Without Extra Array

## Idea

We do not actually need to construct the merged array.

We only need to reach the middle element(s).

Maintain two pointers and simulate the merge process.

Stop once we reach the median position.

For an even number of elements, we need the two middle values.

## Dry Run

```text
nums1 = [1, 3]
nums2 = [2, 4]
```

Total:

```text
4
```

We need positions:

```text
1 and 2
```

Simulated merge:

```text
1 → first middle candidate
2 → second middle candidate
3
```

Median:

```text
(1 + 2) / 2 = 1.5
```

Wait — the indexing needs to be based on the **zero-indexed merged array**:

```text
[1, 2, 3, 4]
 ↑  ↑
 1  2
```

So:

```text
median = (2 + 3) / 2 = 2.5
```

## Algorithm

1. Let:

   ```text
   total = m + n
   ```
2. Determine the two middle positions:

   ```text
   mid1 = (total - 1) / 2
   mid2 = total / 2
   ```
3. Simulate merging using two pointers.
4. Keep track of the previous and current selected values.
5. Stop once `mid2` is reached.
6. If the total size is odd, return the current value.
7. Otherwise, return the average of the two middle values.

## Complexity

* **Time:** `O(m + n)`
* **Space:** `O(1)`

More precisely, because we stop at the middle:

* **Time:** `O((m + n) / 2)` = `O(m + n)`
* **Space:** `O(1)`

## Notes / Tips

* Better than Approach 1 because no merged array is created.
* However, the time complexity is still linear.
* The problem specifically asks for an `O(log(min(m, n)))` solution, so this is not optimal.

## Code

```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        int m = nums1.size();
        int n = nums2.size();

        int total = m + n;
        int mid1 = (total - 1) / 2;
        int mid2 = total / 2;

        int i = 0;
        int j = 0;

        int previous = 0;
        int current = 0;

        for (int k = 0; k <= mid2; k++) {
            previous = current;

            if (i < m && (j >= n || nums1[i] <= nums2[j])) {
                current = nums1[i];
                i++;
            }
            else {
                current = nums2[j];
                j++;
            }
        }

        if (total % 2 == 1) {
            return current;
        }

        return (previous + current) / 2.0;
    }
};
```

---

# Approach 3 — Optimal / Binary Search on Partition

## Idea

Instead of actually merging the arrays, find the point where the combined sorted array can be divided into:

```text
Left Half | Right Half
```

such that:

1. The left half contains exactly half of the elements.
2. Every element in the left half is `<=` every element in the right half.

Because both arrays are sorted, we only need to determine **where to partition each array**.

### Important Observation

Always binary-search the **smaller array**.

Suppose:

```text
nums1 = [1, 3, 8]
nums2 = [7, 9, 10, 11]
```

Choose a partition:

```text
nums1: [1, 3 | 8]
nums2: [7, 9 | 10, 11]
```

The combined partition is:

```text
[1, 3, 7, 9 | 8, 10, 11]
```

This is invalid because:

```text
9 > 8
```

So the partition in `nums1` must move to the right.

We binary-search until the partition becomes valid.

## Partition Conditions

Let:

```text
i = partition index in nums1
j = partition index in nums2
```

The number of elements on the left should be:

```text
(m + n + 1) / 2
```

Therefore:

```text
j = (m + n + 1) / 2 - i
```

Define:

```text
left1  = nums1[i - 1]
right1 = nums1[i]

left2  = nums2[j - 1]
right2 = nums2[j]
```

The partition is correct when:

```text
left1 <= right2
AND
left2 <= right1
```

These two conditions guarantee that every element on the left is smaller than or equal to every element on the right.

## Handling Boundaries

A partition can occur at the beginning or end of an array.

For example:

```text
| 1 2 3
```

There is no element on the left.

Or:

```text
1 2 3 |
```

There is no element on the right.

Use:

```text
-∞
```

when the left side is empty and:

```text
+∞
```

when the right side is empty.

In C++:

```cpp
INT_MIN
INT_MAX
```

can be used for integer arrays.

## Dry Run

```text
nums1 = [1, 3]
nums2 = [2, 4]
```

`nums1` is the smaller array.

```text
m = 2
n = 2
```

Required left size:

```text
(m + n + 1) / 2
= 5 / 2
= 2
```

Try:

```text
i = 1
j = 1
```

Partitions:

```text
nums1: [1 | 3]
nums2: [2 | 4]
```

So:

```text
left1  = 1
right1 = 3

left2  = 2
right2 = 4
```

Check:

```text
left1 <= right2
1 <= 4 ✓

left2 <= right1
2 <= 3 ✓
```

Partition is valid.

Left side:

```text
[1, 2]
```

Right side:

```text
[3, 4]
```

Since total number of elements is even:

```text
median = (max(1, 2) + min(3, 4)) / 2
       = (2 + 3) / 2
       = 2.5
```

## Algorithm

1. Ensure `nums1` is the smaller array. If not, swap the arrays.
2. Let:

   ```text
   m = nums1.size()
   n = nums2.size()
   ```
3. Binary-search the partition `i` in `nums1`.
4. Calculate the corresponding partition in `nums2`:

   ```text
   j = (m + n + 1) / 2 - i
   ```
5. Determine:

   ```text
   left1, right1
   left2, right2
   ```
6. Check whether the partition is valid:

   ```text
   left1 <= right2
   AND
   left2 <= right1
   ```
7. If `left1 > right2`, move the partition in `nums1` left.
8. If `left2 > right1`, move the partition in `nums1` right.
9. Once the partition is valid:

   * If total length is odd:

     ```text
     median = max(left1, left2)
     ```
   * Otherwise:

     ```text
     median = (max(left1, left2) + min(right1, right2)) / 2
     ```

## Complexity

* **Time:** `O(log(min(m, n)))`
* **Space:** `O(1)`

## Notes / Tips

* **Always binary-search the smaller array.**
* The key is not finding the median directly; it is finding the **correct partition**.
* The two conditions to remember are:

  ```text
  left1 <= right2
  left2 <= right1
  ```
* The left half must contain:

  ```text
  (m + n + 1) / 2
  ```

  elements.
* The `+1` makes the odd-length case convenient.
* For odd total length, the median is the **maximum value on the left**.
* For even total length, the median is the average of:

  ```text
  max(left side)
  min(right side)
  ```

## Code

```cpp
class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) {
        if (nums1.size() > nums2.size()) {
            return findMedianSortedArrays(nums2, nums1);
        }

        int m = nums1.size();
        int n = nums2.size();

        int low = 0;
        int high = m;

        while (low <= high) {
            int partition1 = low + (high - low) / 2;
            int partition2 = (m + n + 1) / 2 - partition1;

            int left1 = (partition1 == 0)
                        ? INT_MIN
                        : nums1[partition1 - 1];

            int right1 = (partition1 == m)
                         ? INT_MAX
                         : nums1[partition1];

            int left2 = (partition2 == 0)
                        ? INT_MIN
                        : nums2[partition2 - 1];

            int right2 = (partition2 == n)
                         ? INT_MAX
                         : nums2[partition2];

            if (left1 <= right2 && left2 <= right1) {
                if ((m + n) % 2 == 1) {
                    return max(left1, left2);
                }

                return (max(left1, left2) + min(right1, right2)) / 2.0;
            }

            if (left1 > right2) {
                high = partition1 - 1;
            }
            else {
                low = partition1 + 1;
            }
        }

        return 0.0;
    }
};
```

---

# Approach Comparison

| Approach                   |                Time |      Space | Status      |
| -------------------------- | ------------------: | ---------: | ----------- |
| Merge Both Arrays          |          `O(m + n)` | `O(m + n)` | Brute       |
| Two Pointers               |          `O(m + n)` |     `O(1)` | Better      |
| Binary Search on Partition | `O(log(min(m, n)))` |     `O(1)` | **Optimal** |

## Key Template

```text
Binary Search on Partition

1. Binary-search the smaller array.
2. Choose partition1.
3. Derive partition2 from the required left-half size.
4. Check:
      left1 <= right2
      left2 <= right1
5. If left1 > right2:
      move partition1 left
6. Else:
      move partition1 right
7. Once valid:
      odd  → max(left1, left2)
      even → (max(left1, left2) + min(right1, right2)) / 2
```

## Final Takeaway

The progression is:

```text
Brute:
Merge everything
        ↓
Better:
Simulate merging without storing the array
        ↓
Optimal:
Don't merge at all.
Find the correct partition using binary search.
```

The most important thing to recognize is:

> **Two sorted arrays + median + required logarithmic complexity → Binary Search on Partition.**
