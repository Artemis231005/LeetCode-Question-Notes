# LeetCode 54 — Spiral Matrix

## Metadata

* **LeetCode:** 54
* **Problem:** Spiral Matrix
* **Difficulty:** Medium
* **Topics:** Array, Matrix, Simulation
* **Pattern:** Boundary-Based Matrix Traversal
* **Key Technique:** Maintain `top`, `bottom`, `left`, `right` boundaries
* **Optimal Complexity:** `O(n × m)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Return all elements of a matrix in **spiral order**, starting from the top-left corner.

---

## Idea

Maintain four boundaries:

```text
top    → first unvisited row
bottom → last unvisited row
left   → first unvisited column
right  → last unvisited column
```

Traverse the matrix in four directions:

```text
1. Left → Right
2. Top → Bottom
3. Right → Left
4. Bottom → Top
```

After completing each direction, move the corresponding boundary inward.

```text
        left → right
       ┌─────────────┐
       │ → → → → → → │
       │             ↓
       │             ↓
       │ ← ← ← ← ← ← │
       │ ↑           │
       └─────────────┘
```

The important part is preventing a row or column from being traversed **twice** when the remaining matrix becomes a single row or single column.

---

## Dry Run

For:

```text
1  2  3
4  5  6
7  8  9
```

Initial:

```text
top = 0
bottom = 2
left = 0
right = 2
```

### 1. Left → Right

```text
1  2  3
```

Then:

```text
top++
```

Now:

```text
top = 1
```

### 2. Top → Bottom

```text
6
9
```

Then:

```text
right--
```

Now:

```text
right = 1
```

### 3. Right → Left

```text
8  7
```

Then:

```text
bottom--
```

Now:

```text
bottom = 1
```

### 4. Bottom → Top

```text
4
```

Then:

```text
left++
```

Now:

```text
left = 1
```

Finally:

```text
5
```

Result:

```text
1 2 3 6 9 8 7 4 5
```

---

## Algorithm

1. Initialize:
   ```text
   top = 0
   bottom = n - 1
   left = 0
   right = m - 1
   ```

2. While `top <= bottom` and `left <= right`:
   * Traverse `top` from `left` to `right`.
   * Increment `top`.
   * Traverse `right` from `top` to `bottom`.
   * Decrement `right`.
   * If `top <= bottom`:

     * Traverse `bottom` from `right` to `left`.
     * Decrement `bottom`.
   * If `left <= right`:

     * Traverse `left` from `bottom` to `top`.
     * Increment `left`.

---

## Why Do We Need the `if` Conditions?
The first two traversals **always happen** because they start the current spiral layer.
But after those two traversals, the remaining matrix may have **disappeared**.

### Example: Single Row

```text
1 2 3 4
```

Initially:
```text
top = 0
bottom = 0
left = 0
right = 3
```

The first loop traverses:
```text
1 2 3 4
```

Then:
```text
top++
```

so:
```text
top = 1
bottom = 0
```

Now `top > bottom`.
There is **no row left**.
Therefore, we must NOT perform the `right → left` traversal.

That's why:
```cpp
if (top <= bottom)
```
is placed before the third loop.

---

### Example: Single Column

```text
1
2
3
4
```

The first loop takes:
```text
1
```

The second loop takes:
```text
2
3
4
```

Then:
```text
right--
```

so:
```text
left = 0
right = -1
```

There is **no column left**.
Therefore, we must NOT perform the `bottom → top` traversal.

That's why:
```cpp
if (left <= right)
```
is placed before the fourth loop.

---

## Why Don't We Put `if`s Before the First Two Loops?
Because the outer condition already guarantees that a valid matrix region exists:

```cpp
while (top <= bottom && left <= right)
```

At the beginning of every iteration, both conditions are true.
Therefore:

### First loop
```cpp
for (int i = left; i <= right; i++)
```

is guaranteed to have at least one element because:

```text
left <= right
```

### Second loop
After the first loop, we do:

```cpp
top++;
```

Could this make the second loop invalid?
Yes, **if the original matrix had only one row**.

But in that case, the second loop's range becomes:
```text
top > bottom
```

and the `for` loop simply executes **zero times**.

For example:
```cpp
for (int i = 1; i <= 0; i++)
```
does nothing.

So no `if` is required.

The same idea applies to the first loop: if its range were invalid, the loop would simply execute zero times, although the outer `while` already prevents that situation.

The real danger is **not an empty `for` loop**. The danger is that after changing a boundary, the third/fourth traversal can visit elements that were **already traversed**.

Hence the `if`s are needed specifically after the first two boundary updates.

---

## Complexity

Let the matrix have `n` rows and `m` columns.
* **Time:** `O(n × m)`

  * Every matrix element is added to `ans` at most once.
* **Auxiliary Space:** `O(1)`

  * Only four boundary variables are used.
* **Output Space:** `O(n × m)`

  * `ans` stores all matrix elements.

---

## Code

```cpp
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();

        int top = 0, bottom = n - 1;
        int left = 0, right = m - 1;

        vector<int> ans;

        while (top <= bottom && left <= right) {
            // Left to right
            for (int i = left; i <= right; i++) {
                ans.push_back(matrix[top][i]);
            }
            top++;

            // Top to bottom
            for (int i = top; i <= bottom; i++) {
                ans.push_back(matrix[i][right]);
            }
            right--;

            // Right to left
            if (top <= bottom) {
                for (int i = right; i >= left; i--) {
                    ans.push_back(matrix[bottom][i]);
                }
                bottom--;
            }

            // Bottom to top
            if (left <= right) {
                for (int i = bottom; i >= top; i--) {
                    ans.push_back(matrix[i][left]);
                }
                left++;
            }
        }

        return ans;
    }
};
```

---

## Notes / Tips

* Think in terms of **four shrinking boundaries**, not individual cells.
* After:
  * `top` traversal → `top++`
  * `right` traversal → `right--`
  * `bottom` traversal → `bottom--`
  * `left` traversal → `left++`
* The two `if`s prevent duplicate traversal when only **one row or one column** remains.
* `for` loops can naturally execute zero times when their starting condition is false, so an `if` isn't needed just to prevent an empty loop.
* Always check the boundaries **after modifying them**, because that is when the remaining region can disappear.

---

## Key Template

```text
while (top <= bottom && left <= right)

    → Left to Right
      top++

    ↓ Top to Bottom
      right--

    ← Right to Left
      bottom--

    ↑ Bottom to Top
      left++

    Check:
      if (top <= bottom)
      if (left <= right)
```
