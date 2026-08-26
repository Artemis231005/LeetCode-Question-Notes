# LeetCode 59 — Spiral Matrix II

## Metadata

* **LeetCode:** 59
* **Problem:** Spiral Matrix II
* **Difficulty:** Medium
* **Topics:** Array, Matrix, Simulation
* **Pattern:** Boundary-Based Matrix Filling
* **Key Technique:** Four Boundaries + Directional Filling
* **Optimal Complexity:** `O(n²)` Time, `O(n²)` Space

---

## Problem Statement

Generate an `n × n` matrix filled with numbers `1` to `n²` in **spiral order**.

---

## Idea

This is essentially the **reverse of LeetCode 54 — Spiral Matrix**.
In LeetCode 54, we **read** elements in spiral order.
Here, we **place** elements in spiral order.

Maintain four boundaries:

```text
top    → first unfilled row
bottom → last unfilled row
left   → first unfilled column
right  → last unfilled column
```

Fill the matrix in four directions:

```text
1. Left → Right
2. Top → Bottom
3. Right → Left
4. Bottom → Top
```

After completing each direction, move the corresponding boundary inward.

---

## Dry Run
For `n = 3`:

Initially:
```text
1 2 3
4 5 6
7 8 9
```

but instead of using existing values, we fill them sequentially.

### 1. Left → Right
```text
1  2  3
_  _  _
_  _  _
```

Then:
```text
top++
```

### 2. Top → Bottom
```text
1  2  3
_  _  4
_  _  5
```

Then:
```text
right--
```

### 3. Right → Left
```text
1  2  3
_  _  4
7  6  5
```

Then:
```text
bottom--
```

### 4. Bottom → Top
```text
1  2  3
8  _  4
7  6  5
```

Then:
```text
left++
```

The remaining cell gets `9`:

```text
1 2 3
8 9 4
7 6 5
```

---

## Algorithm

1. Create an `n × n` matrix initialized with `0`.
2. Initialize:

   ```text
   top = 0
   bottom = n - 1
   left = 0
   right = n - 1
   num = 1
   ```
3. While `top <= bottom` and `left <= right`:

   * Fill `top` from `left` to `right`.
   * Increment `top`.
   * Fill `right` from `top` to `bottom`.
   * Decrement `right`.
   * If `top <= bottom`:

     * Fill `bottom` from `right` to `left`.
     * Decrement `bottom`.
   * If `left <= right`:

     * Fill `left` from `bottom` to `top`.
     * Increment `left`.
4. Return the matrix.

---

## Why Do We Need the `if`s?
The reasoning is the same as **LeetCode 54**.
After filling the first two sides, the remaining region may have disappeared.

### `if (top <= bottom)`
Suppose `n = 1`:
```text
_
```

The first loop fills:
```text
1
```

Then:
```text
top++
```

Now:
```text
top = 1
bottom = 0
```

There is no remaining row.
Therefore, we should not perform the `right → left` traversal.

```cpp
if (top <= bottom)
```
prevents that.

### `if (left <= right)`
Similarly, after the first two traversals, the remaining region may contain no column.

So before performing `bottom → top`, we check:
```cpp
if (left <= right)
```

This prevents filling cells outside the remaining boundary or unnecessarily traversing an already-filled region.

The **first two loops don't need explicit `if`s** because the outer condition:
```cpp
while (top <= bottom && left <= right)
```
guarantees that the current region exists at the start of every iteration.

---

## Complexity

* **TC:** `O(n²)`

  * Every cell is filled exactly once.
* **Auxiliary SC:** `O(1)`

  * Apart from the output matrix, only boundary variables are used.
* **Output SC:** `O(n²)`

  * The returned matrix itself contains `n²` elements.
* **Total SC:**  O(n²)
```

---

## Code
```cpp
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) {
        vector<vector<int>> matrix(n, vector<int>(n));

        int top = 0, bottom = n - 1;
        int left = 0, right = n - 1;
        int num = 1;

        while (top <= bottom && left <= right) {
            // Left to right
            for (int i = left; i <= right; i++) {
                matrix[top][i] = num++;
            }
            top++;

            // Top to bottom
            for (int i = top; i <= bottom; i++) {
                matrix[i][right] = num++;
            }
            right--;

            // Right to left
            if (top <= bottom) {
                for (int i = right; i >= left; i--) {
                    matrix[bottom][i] = num++;
                }
                bottom--;
            }

            // Bottom to top
            if (left <= right) {
                for (int i = bottom; i >= top; i--) {
                    matrix[i][left] = num++;
                }
                left++;
            }
        }

        return matrix;
    }
};
```

---

## Notes / Tips

* **LeetCode 54:** Traverse/read matrix in spiral order.
* **LeetCode 59:** Fill/write matrix in spiral order.
* Both use the **same four-boundary template**.
* The only major difference is:

  * 54 → `ans.push_back(matrix[...])`
  * 59 → `matrix[...] = num++`
* For an `n × n` matrix, there are exactly `n²` values to place.
* The two `if`s are important for handling the final **single row/column/cell** safely.

---

## Key Template

```text
top = 0
bottom = n - 1
left = 0
right = n - 1

while (top <= bottom && left <= right):

    → Left to Right
      top++

    ↓ Top to Bottom
      right--

    ← Right to Left
      bottom--

    ↑ Bottom to Top
      left++

    if (top <= bottom)
        → Right to Left

    if (left <= right)
        → Bottom to Top
```
