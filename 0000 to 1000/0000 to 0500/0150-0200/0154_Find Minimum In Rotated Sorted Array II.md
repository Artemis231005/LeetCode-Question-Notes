# LeetCode 154 — Find Minimum in Rotated Sorted Array II

## Metadata

* **LeetCode:** 154
* **Problem:** Find Minimum in Rotated Sorted Array II
* **Difficulty:** Hard
* **Topics:** Array, Binary Search
* **Pattern:** Modified Binary Search
* **Key Technique:** Handle duplicates while finding the rotation point
* **Key Pattern:** Binary Search with duplicate ambiguity
* **Key Template:** `nums[mid] == nums[right] → right--`
* **Optimal Complexity:** `O(log n)` average, `O(n)` worst case

---

## Problem

Given an array `nums` that was originally sorted in ascending order and then rotated at an unknown pivot, find the minimum element.

Unlike LeetCode 153, **duplicates are allowed**.

Example:

```text
nums = [2, 2, 2, 0, 1]
```

Minimum:

```text
0
```

Another example:

```text
nums = [1, 3, 5]
```

Minimum:

```text
1
```

The goal is to achieve `O(log n)` time when possible.

---

## Idea

This is almost the same as **LeetCode 153**, but duplicates create one additional case.

We compare:

```text
nums[mid] with nums[right]
```

### Case 1 — `nums[mid] > nums[right]`

The minimum must be **strictly to the right of `mid`**.

```text
left = mid + 1
```

Example:

```text
[4, 5, 6, 0, 1, 2]
       ↑        ↑
      mid      right

6 > 2
```

Therefore:

```text
left = mid + 1
```

### Case 2 — `nums[mid] < nums[right]`

The minimum is at `mid` or somewhere to its left.

```text
right = mid
```

We cannot discard `mid` because it could itself be the minimum.

### Case 3 — `nums[mid] == nums[right]`

This is the important difference from LeetCode 153.

Example:

```text
[1, 0, 1, 1, 1]
       ↑        ↑
      mid      right

1 == 1
```

We cannot determine whether the minimum is on the left or right.

However, since:

```text
nums[mid] == nums[right]
```

we know that `right` is not giving us any useful information that `mid` does not already provide.

So safely reduce the search space by one:

```text
right--
```

This may cause the worst-case complexity to become `O(n)`.

---

## Dry Run

Consider:

```text
nums = [2, 2, 2, 0, 1]
```

Initial:

```text
left = 0
right = 4
```

### Step 1

```text
mid = 2

nums[mid] = 2
nums[right] = 1
```

Since:

```text
2 > 1
```

minimum is to the right.

```text
left = mid + 1 = 3
```

Now:

```text
[2, 2, 2, 0, 1]
          L     R
```

### Step 2

```text
left = 3
right = 4
mid = 3
```

Compare:

```text
nums[mid] = 0
nums[right] = 1
```

Since:

```text
0 < 1
```

minimum is at `mid` or to the left.

```text
right = mid = 3
```

Now:

```text
left = right = 3
```

Answer:

```text
nums[3] = 0
```

---

## Dry Run — Duplicate Case

Consider:

```text
nums = [1, 1, 1, 0, 1]
```

Initial:

```text
left = 0
right = 4
mid = 2
```

Compare:

```text
nums[mid] = 1
nums[right] = 1
```

They are equal.

We cannot determine which side contains the minimum.

So:

```text
right--
```

Now:

```text
right = 3
```

Next:

```text
mid = 1
```

Compare:

```text
nums[mid] = 1
nums[right] = 0
```

Since:

```text
1 > 0
```

move right:

```text
left = mid + 1 = 2
```

Continue until:

```text
left = right = 3
```

Answer:

```text
0
```

---

## Algorithm

1. Set:

   ```text
   left = 0
   right = n - 1
   ```
2. While `left < right`:

   * Calculate:

     ```text
     mid = left + (right - left) / 2
     ```
3. Compare `nums[mid]` and `nums[right]`.
4. If:

   ```text
   nums[mid] > nums[right]
   ```

   set:

   ```text
   left = mid + 1
   ```
5. Else if:

   ```text
   nums[mid] < nums[right]
   ```

   set:

   ```text
   right = mid
   ```
6. Otherwise:

   ```text
   nums[mid] == nums[right]
   ```

   so safely reduce the search space:

   ```text
   right--
   ```
7. When `left == right`, return `nums[left]`.

---

## Complexity

* **Average Time:** `O(log n)`
* **Worst-case Time:** `O(n)`

  * Happens when duplicates prevent us from determining which half contains the minimum.
* **Space:** `O(1)`

### Why Can It Become `O(n)`?

Consider:

```text
[1, 1, 1, 1, 1, 1, 0, 1]
```

or even:

```text
[1, 1, 1, 1, 1, 1, 1]
```

If:

```text
nums[mid] == nums[right]
```

repeatedly, we can only do:

```text
right--
```

one element at a time.

Therefore, binary search can degrade to `O(n)`.

---

## Notes / Tips

* LeetCode 154 is the duplicate version of LeetCode 153.
* The first two cases remain the same:

  ```text
  mid > right → left = mid + 1
  mid < right → right = mid
  ```
* The new case is:

  ```text
  mid == right → right--
  ```
* When `mid == right`, we cannot know which side contains the minimum.
* Reducing `right` is safe because `nums[right] == nums[mid]`, so removing `right` cannot remove a uniquely identifiable smaller value.
* Use `right = mid`, not `right = mid - 1`, when `nums[mid] < nums[right]`.

### LeetCode 153 vs 154

| Condition      | 153 — Unique     | 154 — Duplicates |
| -------------- | ---------------- | ---------------- |
| `mid > right`  | `left = mid + 1` | `left = mid + 1` |
| `mid < right`  | `right = mid`    | `right = mid`    |
| `mid == right` | Cannot happen    | `right--`        |
| Worst Time     | `O(log n)`       | `O(n)`           |
| Extra Space    | `O(1)`           | `O(1)`           |

### Common Mistake

Do not write:

```cpp
right = mid - 1;
```

when:

```text
nums[mid] < nums[right]
```

`mid` itself can be the minimum.

Correct:

```cpp
right = mid;
```

---

## Code

```cpp
class Solution {
public:
    int findMin(vector<int>& nums) {
        int left = 0;
        int right = nums.size() - 1;

        while (left < right) {
            int mid = left + (right - left) / 2;

            if (nums[mid] > nums[right]) {
                left = mid + 1;
            } else if (nums[mid] < nums[right]) {
                right = mid;
            } else {
                right--;
            }
        }

        return nums[left];
    }
};
```

---

## Basic Template

```cpp
int findMin(vector<int>& nums) {
    int left = 0;
    int right = nums.size() - 1;

    while (left < right) {
        int mid = left + (right - left) / 2;

        if (nums[mid] > nums[right]) {
            left = mid + 1;
        } else if (nums[mid] < nums[right]) {
            right = mid;
        } else {
            right--;
        }
    }

    return nums[left];
}
```

### Reusable Pattern

```text
Compare nums[mid] with nums[right]

        mid > right
             ↓
      left = mid + 1

        mid < right
             ↓
        right = mid

        mid == right
             ↓
          right--
```

### Core Rule

```text
153 → Unique elements
      mid == right cannot happen

154 → Duplicates allowed
      mid == right
          ↓
       right--
```
