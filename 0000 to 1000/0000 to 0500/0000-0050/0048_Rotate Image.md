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

* **Time:** `O(n²)`

  * Transposition takes approximately `n²/2`.
  * Reversing rows takes approximately `n²/2`.
  * Overall: `O(n²)`.
* **Space:** `O(1)` auxiliary space.

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

* **Clockwise 90°:** Transpose → Reverse Rows.
* **Anticlockwise 90°:** Transpose → Reverse Columns.
* During transpose, start `j` from `i + 1` so each pair is swapped **exactly once**.
* `i` goes only till `n - 2`; when `i = n - 1`, there is no element after it to swap.
* Although transpose traverses only half the matrix, it still takes **`O(n²)`**, not `O(n²/4)`.

---

## Key Template

```text
90° Clockwise
    ↓
Transpose
    ↓
Reverse Rows
```
