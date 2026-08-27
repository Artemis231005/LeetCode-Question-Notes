# LeetCode 48 — Rotate Image

## Metadata

* **LeetCode:** 48
* **Problem:** Rotate Image
* **Difficulty:** Medium
* **Topics:** Array, Matrix, Simulation
* **Pattern:** Matrix Transformation
* **Key Technique:** Transpose + Reverse Rows
* **Optimal Complexity:** `O(n²)` Time, `O(1)` Space

---

## Problem Statement

Rotate an `n × n` matrix **90 degrees clockwise in-place** without using another matrix.

---

# Brute Approach

## Idea
Create a **new `n × n` matrix** and directly place every element at its rotated position.

For an element at:
`matrix[i][j]`

after a 90° clockwise rotation, its new position is:
`result[j][n - 1 - i]`

Example:
```text
1 2 3
4 5 6
7 8 9
```

After rotation:
```text
7 4 1
8 5 2
9 6 3
```

This is the most straightforward approach, but it uses an extra matrix.

---

## Dry Run

For:
```text
1 2 3
4 5 6
7 8 9
```

Take `1` at `(0,0)`:
```text
new position = (0, 3 - 1 - 0)
             = (0,2)
```

Take `2` at `(0,1)`:
```text
new position = (1,2)
```

Take `3` at `(0,2)`:
```text
new position = (2,2)
```

So the result becomes:
```text
7 4 1
8 5 2
9 6 3
```

---

## Algorithm

1. Let `n = matrix.size()`.
2. Create an empty `n × n` matrix `result`.
3. Traverse every element `(i, j)` of the original matrix.
4. Place it at:
   `result[j][n - 1 - i] = matrix[i][j]`.
5. Copy `result` back into `matrix`.

---

## Complexity
* **TC:** `O(n²)`

  * Traverse all `n²` elements.
  * Copy the result back: `O(n²)`.
* **SC:** `O(n²)` auxiliary space.

---

## Code
```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();

        vector<vector<int>> result(n, vector<int>(n));

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < n; j++) {
                result[j][n - 1 - i] = matrix[i][j];
            }
        }

        matrix = result;
    }
};
```

---

# Better Approach

## Idea
Instead of creating a completely new matrix, rotate the matrix **in-place layer by layer**.

Each group of four corresponding elements can be rotated using a temporary variable.

For example:
```text
1 2 3
4 5 6
7 8 9
```

The four corners:
```text
1 → 3 → 9 → 7 → 1
```

can be rotated in-place.

This reduces the auxiliary space from `O(n²)` to `O(1)`.

---

## Dry Run
For the outer layer:
```text
1 2 3
4 5 6
7 8 9
```

Rotate:
```text
1 → 3
3 → 9
9 → 7
7 → 1
```

Result:
```text
7 2 1
4 5 6
9 8 3
```

Then rotate the remaining inner layer:
```text
5
```
which stays unchanged.

Final:
```text
7 4 1
8 5 2
9 6 3
```

---

## Algorithm

1. Let `n = matrix.size()`.
2. Process the matrix layer by layer.
3. For each layer, maintain:

   * `first` = starting index of the layer.
   * `last` = ending index of the layer.
4. For every element in the current layer:

   * Store the top element temporarily.
   * Move left → top.
   * Move bottom → left.
   * Move right → bottom.
   * Move the temporary top → right.
5. Move to the next inner layer.

---

## Complexity
* **TC:** `O(n²)`

  * Every element is processed approximately once.
* **SC:** `O(1)` auxiliary space.

---

## Code

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();

        for (int first = 0; first < n / 2; first++) {
            int last = n - 1 - first;

            for (int i = first; i < last; i++) {
                int offset = i - first;

                int top = matrix[first][i];

                // Left → Top
                matrix[first][i] = matrix[last - offset][first];

                // Bottom → Left
                matrix[last - offset][first] =
                    matrix[last][last - offset];

                // Right → Bottom
                matrix[last][last - offset] =
                    matrix[i][last];

                // Top → Right
                matrix[i][last] = top;
            }
        }
    }
};
```

---

# Optimal Approach

## Idea

A 90° clockwise rotation can be done in **two steps**:
1. **Transpose** the matrix.
2. **Reverse every row**.

Example:
```text
1 2 3        1 4 7        7 4 1
4 5 6   →    2 5 8   →    8 5 2
7 8 9        3 6 9        9 6 3
```

---

## Dry Run

### Step 1: Transpose

Swap:
```text
matrix[i][j] ↔ matrix[j][i]
```

for only the upper triangular part.


```text
1 2 3
4 5 6
7 8 9
```

becomes:
```text
1 4 7
2 5 8
3 6 9
```

### Step 2: Reverse Every Row

```text
1 4 7 → 7 4 1
2 5 8 → 8 5 2
3 6 9 → 9 6 3
```

Final:
```text
7 4 1
8 5 2
9 6 3
```

---

## Algorithm

1. Store the size of the matrix as `n`.
2. Transpose the matrix:

   * For `i = 0` to `n - 2`.
   * For `j = i + 1` to `n - 1`.
   * Swap `matrix[i][j]` and `matrix[j][i]`.
3. Reverse every row.
4. The matrix is now rotated 90° clockwise.

---

## Complexity
* **TC:** `O(n²)`

  * Transposition takes approximately `n²/2`.
  * Reversing rows takes approximately `n²/2`.
  * Overall: `O(n²)`.
* **SC:** `O(1)` auxiliary space.

---

## Code

```cpp
class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();

        // Transpose
        for (int i = 0; i < n - 1; i++) {
            for (int j = i + 1; j < n; j++) {
                swap(matrix[i][j], matrix[j][i]);
            }
        }

        // Reverse every row
        for (int i = 0; i < n; i++) {
            reverse(matrix[i].begin(), matrix[i].end());
        }
    }
};
```

---

## Notes / Tips

* **Brute:** Directly place elements into a new matrix using `result[j][n - 1 - i]`.
* **Better:** Rotate four corresponding elements in-place.
* **Optimal:** Transpose → Reverse Rows.
* Clockwise 90°: **Transpose → Reverse Rows**.
* Anticlockwise 90°: **Transpose → Reverse Columns**.
* During transpose, start `j` from `i + 1` so each pair is swapped **exactly once**.
* `i` goes only till `n - 2`; when `i = n - 1`, there is no element after it to swap.
* Although transpose traverses only half the matrix, it still takes **`O(n²)`**, not `O(n²/4)`.
* *Note that for Leetcode - 48, the question has a requirement for in-place, so only the optimal approach can be applied*


---

## Key Template

```text
90° Clockwise
    ↓
Transpose
    ↓
Reverse Rows
```



