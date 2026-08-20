# LeetCode 41 — First Missing Positive

## Metadata

* **LeetCode:** 41
* **Problem:** First Missing Positive
* **Difficulty:** Hard
* **Topics:** Array, Hash Table, Sorting, In-Place
* **Pattern:** Cyclic Sort, Index Placement
* **Key Pattern:** Place each positive number at its correct index
* **Key Technique:** In-place cyclic placement
* **Optimal Complexity:** `O(n)` time, `O(1)` extra space
* **Key Template:** Value-to-Index Mapping

---

## Problem

Given an unsorted integer array `nums`, find the smallest positive integer that does not appear in the array.

The solution must run in:

* `O(n)` time
* `O(1)` extra space

Example:

```text
nums = [1, 2, 0]

Output = 3
```

Another example:

```text
nums = [3, 4, -1, 1]

Output = 2
```

---

## Key Observation

For an array of size `n`, the answer must be somewhere in:

```text
1 → n + 1
```

Why?

If the array contains:

```text
[1, 2, 3, ..., n]
```

then the first missing positive is:

```text
n + 1
```

Otherwise, the missing positive will be somewhere between `1` and `n`.

Therefore, we only care about numbers:

```text
1 <= nums[i] <= n
```

Numbers such as:

```text
0
-1
-20
n + 1
100
```

can effectively be ignored.

---

## Approach — Cyclic Placement

### Idea

We want every valid number to be placed at its **correct index**.

For a positive number `x`:

```text
x → index x - 1
```

So:

```text
1 → index 0
2 → index 1
3 → index 2
4 → index 3
...
n → index n - 1
```

For example:

```text
nums = [3, 4, -1, 1]
```

The correct positions are:

```text
Value     Correct Index
  1   →       0
  2   →       1
  3   →       2
  4   →       3
```

We rearrange the array so that every number that can be placed correctly is placed there.

Afterward, scan the array:

```text
index:  0  1  2  3
value:  1  2  -1  4
```

The first index where:

```text
nums[i] != i + 1
```

is the answer.

Here:

```text
i = 2
nums[2] = -1
i + 1 = 3
```

Therefore:

```text
answer = 3
```

---

## Why `nums[i] - 1`?

The array uses **0-based indexing**, but positive integers start from `1`.

Therefore:

```text
Value 1 → Index 0
Value 2 → Index 1
Value 3 → Index 2
```

So:

```cpp
correctIndex = nums[i] - 1;
```

This value-to-index relationship is the heart of the problem.

---

## Dry Run

Consider:

```text
nums = [3, 4, -1, 1]
```

### Step 1

Start at index `0`:

```text
nums[0] = 3
```

`3` belongs at:

```text
index = 3 - 1 = 2
```

Swap:

```text
[3, 4, -1, 1]
 ↓
[-1, 4, 3, 1]
```

We stay at index `0` because the new value at index `0` may also need to be moved.

### Step 2

Now:

```text
nums[0] = -1
```

Ignore it because it is not in:

```text
[1, n]
```

Move to index `1`.

### Step 3

```text
nums[1] = 4
```

`4` belongs at index:

```text
4 - 1 = 3
```

Swap:

```text
[-1, 4, 3, 1]
 ↓
[-1, 1, 3, 4]
```

### Step 4

At index `1`:

```text
nums[1] = 1
```

`1` belongs at index `0`.

Swap:

```text
[-1, 1, 3, 4]
 ↓
[1, -1, 3, 4]
```

### Step 5

At index `1`:

```text
nums[1] = -1
```

Ignore it.

Final array:

```text
[1, -1, 3, 4]
```

Now scan:

```text
index 0 → expected 1 → found 1 ✓
index 1 → expected 2 → found -1 ✗
```

Therefore:

```text
answer = 2
```

---

## Why We Need the Duplicate Check

Consider:

```text
nums = [1, 1]
```

For the first `1`:

```text
correct index = 0
```

It is already there.

If we blindly swap:

```cpp
swap(nums[i], nums[nums[i] - 1]);
```

we would keep swapping `1` with itself forever.

Therefore we need:

```cpp
nums[nums[i] - 1] != nums[i]
```

The complete condition becomes:

```cpp
while (
    nums[i] >= 1 &&
    nums[i] <= n &&
    nums[nums[i] - 1] != nums[i]
)
```

This ensures:

1. The value is within the useful range.
2. The destination does not already contain the same value.

---

### Algorithm

1. Let `n = nums.size()`.
2. For every index `i`:

   * While `nums[i]` is between `1` and `n`:

     * Calculate its correct index:

       ```cpp
       nums[i] - 1
       ```
     * If that position already contains the same value, stop.
     * Otherwise swap the current value into its correct position.
3. Scan the array from left to right.
4. For every index `i`:

   * If:

     ```cpp
     nums[i] != i + 1
     ```

     return `i + 1`.
5. If every position is correct, return `n + 1`.

---

### Complexity

* **Time:** `O(n)`
* **Extra Space:** `O(1)`

Although there is a `while` loop inside a `for` loop, the overall complexity remains `O(n)`.

Each useful number is moved toward its correct position, and the number of swaps is bounded linearly.

---

### Notes / Tips

* This is a classic **Cyclic Sort / Index Placement** problem.
* The most important mapping is:

  ```text
  value x → index x - 1
  ```
* Ignore:

  ```text
  x <= 0
  x > n
  ```
* Duplicate values must not cause infinite swapping.
* The first index where:

  ```cpp
  nums[i] != i + 1
  ```

  gives the answer.
* If every position is correct, the answer is:

  ```cpp
  n + 1
  ```
* Do not sort the array because normal sorting takes `O(n log n)`, violating the required `O(n)` time.
* Do not use a hash set because it requires `O(n)` extra space.
* The trick is to use the **array itself as the hash/index structure**.

---

### Code

```cpp
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int n = nums.size();

        for (int i = 0; i < n; i++) {
            while (
                nums[i] >= 1 &&
                nums[i] <= n &&
                nums[nums[i] - 1] != nums[i]
            ) {
                swap(nums[i], nums[nums[i] - 1]);
            }
        }

        for (int i = 0; i < n; i++) {
            if (nums[i] != i + 1) {
                return i + 1;
            }
        }

        return n + 1;
    }
};
```

---

## Alternative Approach — Mark Using the Array

### Idea

Another `O(n)` time and `O(1)` space approach is to use the input array itself to mark which positive numbers exist.

The idea is:

1. Ignore values outside `[1, n]`.
2. Convert useful values into markers.
3. Use the sign of `nums[x - 1]` to indicate whether `x` exists.
4. Find the first positive position.

However, this approach requires carefully preserving information about values before marking and handling duplicates.

The cyclic placement approach is generally easier to implement and remember.

---

### Algorithm

1. Replace every value outside `[1, n]` with a harmless value such as `n + 1`.
2. For each value `x`:

   * Look at index `x - 1`.
   * Make that value negative to mark `x` as present.
3. Scan the array.
4. The first positive value at index `i` means `i + 1` is missing.
5. If all positions are negative, return `n + 1`.

---

### Complexity

* **Time:** `O(n)`
* **Extra Space:** `O(1)`

---

### Notes / Tips

The marking approach is useful to know, but the cyclic placement approach is usually the cleaner solution.

The key idea is still:

```text
Number x
   ↓
Index x - 1
   ↓
Mark as present
```

---

### Code

```cpp
class Solution {
public:
    int firstMissingPositive(vector<int>& nums) {
        int n = nums.size();

        // Replace irrelevant values
        for (int i = 0; i < n; i++) {
            if (nums[i] <= 0 || nums[i] > n) {
                nums[i] = n + 1;
            }
        }

        // Mark existing positive numbers
        for (int i = 0; i < n; i++) {
            int value = abs(nums[i]);

            if (value <= n) {
                nums[value - 1] = -abs(nums[value - 1]);
            }
        }

        // Find first missing positive
        for (int i = 0; i < n; i++) {
            if (nums[i] > 0) {
                return i + 1;
            }
        }

        return n + 1;
    }
};
```

---

## Key Takeaways

### Core Pattern

```text
First Missing Positive
        ↓
Only care about [1, n]
        ↓
Value x belongs at index x - 1
        ↓
Place values in their correct positions
        ↓
Find first incorrect index
        ↓
Answer = index + 1
```

### Cyclic Placement Template

```cpp
for (int i = 0; i < n; i++) {
    while (
        nums[i] >= 1 &&
        nums[i] <= n &&
        nums[nums[i] - 1] != nums[i]
    ) {
        swap(nums[i], nums[nums[i] - 1]);
    }
}
```

Then:

```cpp
for (int i = 0; i < n; i++) {
    if (nums[i] != i + 1) {
        return i + 1;
    }
}

return n + 1;
```

### Pattern to Remember

> **If a problem asks for missing numbers in `[1, n]` and requires `O(n)` time + `O(1)` space, think about using the array itself as an index/marker structure.**

For LeetCode 41 specifically:

```text
Value → Correct Index
  x   →    x - 1
```

That mapping is the main trick.
