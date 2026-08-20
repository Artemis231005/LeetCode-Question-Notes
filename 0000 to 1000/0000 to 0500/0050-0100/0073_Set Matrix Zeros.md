# LeetCode 73 — Set Matrix Zeroes

## Metadata

* **LeetCode:** 73
* **Problem:** Set Matrix Zeroes
* **Difficulty:** Medium
* **Topics:** Array, Matrix, Hashing
* **Pattern:** In-place Matrix Marking
* **Key Technique:** Use first row and first column as markers
* **Optimal Complexity:** `O(m × n)` time, `O(1)` extra space

---

## Problem

Given an `m × n` integer matrix, if an element is `0`, set its **entire row and entire column** to `0`.

The operation must be done **in-place**.

---

## Approach 1 — Extra Space

### Idea

Use two arrays:

* `row[i]` → whether row `i` needs to become zero.
* `col[j]` → whether column `j` needs to become zero.

First scan the matrix and record every row and column containing `0`.

Then scan again and set `matrix[i][j] = 0` if:

```text
row[i] == true OR col[j] == true
```

### Dry Run

```text
Matrix:

1  1  1
1  0  1
1  1  1

Zero found at (1,1)

row = [false, true, false]
col = [false, true, false]

Final:

1  0  1
0  0  0
1  0  1
```

### Algorithm

1. Create `row` of size `m` and `col` of size `n`, initialized to `false`.
2. Traverse the matrix.
3. Whenever `matrix[i][j] == 0`:

   * Set `row[i] = true`.
   * Set `col[j] = true`.
4. Traverse the matrix again.
5. If `row[i]` or `col[j]` is `true`, set `matrix[i][j] = 0`.

### Complexity

* **Time:** `O(m × n)`
* **Space:** `O(m + n)`

### Notes / Tips

* This approach is straightforward and easy to understand.
* The main drawback is the extra `O(m + n)` space.
* Do **not** modify rows/columns during the first scan, otherwise newly created zeroes can incorrectly affect later processing.

### Code

```cpp
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int m = matrix.size();
        int n = matrix[0].size();

        vector<bool> row(m, false);
        vector<bool> col(n, false);

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (matrix[i][j] == 0) {
                    row[i] = true;
                    col[j] = true;
                }
            }
        }

        for (int i = 0; i < m; i++) {
            for (int j = 0; j < n; j++) {
                if (row[i] || col[j]) {
                    matrix[i][j] = 0;
                }
            }
        }
    }
};
```

---

## Approach 2 — Constant Extra Space

### Idea

We need to avoid `O(m + n)` arrays.

Use the **first row and first column of the matrix itself as marker arrays**.

For every zero at `(i, j)`:

```text
matrix[i][0] = 0
matrix[0][j] = 0
```

This tells us:

* `matrix[i][0] == 0` → row `i` must be zero.
* `matrix[0][j] == 0` → column `j` must be zero.

### Important Problem

The first row and first column themselves may originally contain zeroes.

So maintain two separate variables:

```cpp
bool firstRowZero;
bool firstColZero;
```

These remember whether the **original** first row or first column contained a zero.

### Dry Run

Consider:

```text
1  1  1
1  0  1
1  1  1
```

Zero at `(1,1)`.

Use it as a marker:

```text
1  0  1
0  0  1
1  1  1
```

Here:

```text
matrix[1][0] = 0  → row 1 becomes zero
matrix[0][1] = 0  → column 1 becomes zero
```

Now process the inner matrix:

```text
1  0  1
0  0  0
1  0  1
```

### Algorithm

1. Check whether the first row contains a zero.
2. Check whether the first column contains a zero.
3. Traverse the matrix excluding the first row and first column.
4. For every zero at `(i, j)`:

   * Set `matrix[i][0] = 0`.
   * Set `matrix[0][j] = 0`.
5. Traverse the inner matrix again.
6. If:

   ```cpp
   matrix[i][0] == 0 || matrix[0][j] == 0
   ```

   set `matrix[i][j] = 0`.
7. If `firstRowZero` is true, set the entire first row to zero.
8. If `firstColZero` is true, set the entire first column to zero.

### Complexity

* **Time:** `O(m × n)`
* **Space:** `O(1)`

### Notes / Tips

* This is the **optimal approach**.
* The first row and first column act as marker storage.
* Always save `firstRowZero` and `firstColZero` **before modifying the matrix**.
* Process the first row and first column **at the end**.
* If you zero the first row/column too early, you lose the marker information.
* This is a classic **in-place marking** pattern.

### Code

```cpp
class Solution {
public:
    void setZeroes(vector<vector<int>>& matrix) {
        int m = matrix.size();
        int n = matrix[0].size();

        bool firstRowZero = false;
        bool firstColZero = false;

        // Check first row
        for (int j = 0; j < n; j++) {
            if (matrix[0][j] == 0) {
                firstRowZero = true;
                break;
            }
        }

        // Check first column
        for (int i = 0; i < m; i++) {
            if (matrix[i][0] == 0) {
                firstColZero = true;
                break;
            }
        }

        // Use first row and first column as markers
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (matrix[i][j] == 0) {
                    matrix[i][0] = 0;
                    matrix[0][j] = 0;
                }
            }
        }

        // Set inner matrix cells to zero
        for (int i = 1; i < m; i++) {
            for (int j = 1; j < n; j++) {
                if (matrix[i][0] == 0 || matrix[0][j] == 0) {
                    matrix[i][j] = 0;
                }
            }
        }

        // Set first row to zero if needed
        if (firstRowZero) {
            for (int j = 0; j < n; j++) {
                matrix[0][j] = 0;
            }
        }

        // Set first column to zero if needed
        if (firstColZero) {
            for (int i = 0; i < m; i++) {
                matrix[i][0] = 0;
            }
        }
    }
};
```

---

## Key Takeaway

The main trick is:

```text
First row    → column markers
First column → row markers
```

And because the first row/column are also part of the answer, separately remember:

```cpp
firstRowZero
firstColZero
```

**Pattern to remember:**

> When extra arrays are needed only for marking rows/columns, check whether the input matrix itself can be reused as the marker storage.
