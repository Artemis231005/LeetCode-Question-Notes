# LeetCode 75 — Sort Colors

## Metadata

* **LeetCode:** 75
* **Problem:** Sort Colors
* **Difficulty:** Medium
* **Topics:** Array, Two Pointers, Sorting
* **Pattern:** Dutch National Flag
* **Key Technique:** Three-way partitioning
* **Optimal Complexity:** `O(n)` time, `O(1)` space

---

## Problem

Given an array `nums` containing only:

```text
0, 1, 2
```

Sort the array **in-place** so that all `0`s come first, followed by all `1`s, then all `2`s.

You **cannot use the library sort function**.

---

## Approach 1 — Counting

### Idea

Since there are only three possible values (`0`, `1`, `2`), count how many of each value exists.

Then overwrite the array:

```text
count[0] → write 0s
count[1] → write 1s
count[2] → write 2s
```

### Dry Run

```text
nums = [2, 0, 2, 1, 1, 0]

Count:
0 → 2
1 → 2
2 → 2

Rewrite:

[0, 0, 1, 1, 2, 2]
```

### Algorithm

1. Create a count array of size `3`.
2. Traverse `nums` and increment the count of each value.
3. Traverse values `0`, `1`, and `2`.
4. Write each value according to its frequency.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(1)` because the count array has fixed size `3`.

### Notes / Tips

* This is simple and efficient.
* Since there are only three distinct values, counting works very well.
* However, the more important interview pattern for this problem is **Dutch National Flag / three-way partitioning**.

### Code

```cpp
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int count[3] = {0};

        for (int num : nums) {
            count[num]++;
        }

        int index = 0;

        for (int color = 0; color < 3; color++) {
            while (count[color] > 0) {
                nums[index] = color;
                index++;
                count[color]--;
            }
        }
    }
};
```

---

## Approach 2 — Dutch National Flag Algorithm

### Idea

Use three pointers:

```text
low   → position where next 0 should go
mid   → current element
high  → position where next 2 should go
```

Maintain three regions:

```text
[0 ... low-1]       → all 0s
[low ... mid-1]     → all 1s
[mid ... high]      → unknown
[high+1 ... n-1]    → all 2s
```

We inspect `nums[mid]`.

### Cases

#### Case 1: `nums[mid] == 0`

`0` belongs at the beginning.

Swap it with `nums[low]`:

```text
swap(nums[low], nums[mid])
```

Then:

```text
low++
mid++
```

#### Case 2: `nums[mid] == 1`

`1` is already in its correct middle region.

Just:

```text
mid++
```

#### Case 3: `nums[mid] == 2`

`2` belongs at the end.

Swap it with `nums[high]`:

```text
swap(nums[mid], nums[high])
```

Then:

```text
high--
```

**Do NOT increment `mid`.**

Why?

Because the element swapped from `high` into `mid` has not been processed yet.

### Dry Run

```text
nums = [2, 0, 2, 1, 1, 0]

low = 0
mid = 0
high = 5
```

#### Step 1

```text
nums[mid] = 2
```

Swap `mid` and `high`:

```text
[0, 0, 2, 1, 1, 2]
```

```text
high--
```

`mid` stays `0`.

---

#### Step 2

```text
nums[mid] = 0
```

Swap `low` and `mid`:

```text
[0, 0, 2, 1, 1, 2]
```

```text
low++
mid++
```

---

#### Step 3

```text
nums[mid] = 2
```

Swap with `high`:

```text
[0, 0, 1, 1, 2, 2]
```

```text
high--
```

`mid` stays.

---

#### Step 4

```text
nums[mid] = 1
```

Just:

```text
mid++
```

---

#### Step 5

```text
nums[mid] = 1
```

Just:

```text
mid++
```

Final:

```text
[0, 0, 1, 1, 2, 2]
```

### Algorithm

1. Initialize:

   ```cpp
   low = 0
   mid = 0
   high = n - 1
   ```
2. While `mid <= high`:

   * If `nums[mid] == 0`:

     * Swap `nums[low]` and `nums[mid]`.
     * Increment `low` and `mid`.
   * If `nums[mid] == 1`:

     * Increment `mid`.
   * If `nums[mid] == 2`:

     * Swap `nums[mid]` and `nums[high]`.
     * Decrement `high`.
     * Do **not** increment `mid`.
3. Stop when `mid > high`.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

### Notes / Tips

* This is the **optimal and preferred approach**.
* It is called the **Dutch National Flag Algorithm**, introduced by Edsger Dijkstra.
* The most important point:

  ```text
  0 → low++, mid++
  1 → mid++
  2 → high-- only
  ```
* For `2`, don't increment `mid` because the swapped element is still unprocessed.
* The loop condition is:

  ```cpp
  while (mid <= high)
  ```
* Think of it as maintaining **three sorted regions + one unknown region**.

### Code

```cpp
class Solution {
public:
    void sortColors(vector<int>& nums) {
        int low = 0;
        int mid = 0;
        int high = nums.size() - 1;

        while (mid <= high) {
            if (nums[mid] == 0) {
                swap(nums[low], nums[mid]);
                low++;
                mid++;
            }
            else if (nums[mid] == 1) {
                mid++;
            }
            else {
                swap(nums[mid], nums[high]);
                high--;
            }
        }
    }
};
```

---

## Key Takeaway

Remember the three-pointer rule:

```text
0 → swap with low → low++, mid++
1 → mid++
2 → swap with high → high--
```

The core invariant is:

```text
[0 ... low-1]   → 0
[low ... mid-1] → 1
[mid ... high]  → unknown
[high+1 ... n]  → 2
```

**Pattern:**

> Dutch National Flag = Three-way partitioning using `low`, `mid`, and `high`.
