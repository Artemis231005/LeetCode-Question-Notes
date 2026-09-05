# LeetCode 304 — Range Sum Query 2D - Immutable

## Metadata

* **LeetCode:** 304
* **Problem:** Range Sum Query 2D - Immutable
* **Difficulty:** Medium
* **Topics:** Array, Design, Matrix, Prefix Sum
* **Pattern:** 2D Prefix Sum
* **Key Technique:** Precompute a 2D prefix sum table once so any rectangle sum becomes an O(1) inclusion-exclusion lookup
* **Optimal Complexity:** `O(1)` Time per query, `O(rows * cols)` Space

---

## Problem Statement

Given a 2D matrix, design a class `NumMatrix` that answers multiple `sumRegion(row1, col1, row2, col2)` queries, each returning the sum of the rectangle bounded by `(row1, col1)` (top-left) and `(row2, col2)` (bottom-right), inclusive. The matrix never changes between queries.

---

## Approaches

1. **Brute Force — Sum on Every Query**
2. **Optimal — 2D Prefix Sum Table**

---

# Approach 1 — Brute Force / Sum on Every Query

## Idea

Store the matrix as-is. For every `sumRegion` call, loop over every cell inside the requested rectangle and add it up directly.

## Dry Run

```text
matrix = [[3, 0, 1],
          [5, 6, 3],
          [1, 2, 0]]
```

`sumRegion(1, 1, 2, 2)`:

```text
cells: 6, 3, 2, 0
sum = 6 + 3 + 2 + 0 = 11
```

Each call rescans its own rectangle from scratch — no reuse between calls.

## Algorithm

1. Constructor: store `matrix` as-is.
2. `sumRegion(row1, col1, row2, col2)`:

   * Initialize `sum = 0`.
   * Loop `r` from `row1` to `row2`, and for each `r`, loop `c` from `col1` to `col2`, adding `matrix[r][c]` to `sum`.
   * Return `sum`.

## Complexity

* **Time:** `O(1)` for the constructor, `O(rows * cols)` per `sumRegion` call

  * Each query can scan the entire rectangle with no precomputed help.
* **Space:** `O(1)`

  * No extra structure beyond storing the input matrix reference.

## Notes / Tips

* Fine for a handful of queries, but degrades badly if `sumRegion` is called many times on large rectangles — total cost becomes `O(rows * cols * q)` for `q` queries.
* The matrix never changes ("immutable"), which is exactly the signal that repeated work can be precomputed once instead of redone per query (same signal as LC 303).

## Code

```cpp
class NumMatrix {
public:
    vector<vector<int>> matrix;

    NumMatrix(vector<vector<int>>& matrix) {
        this->matrix = matrix;
    }

    int sumRegion(int row1, int col1, int row2, int col2) {
        int sum = 0;
        for (int r = row1; r <= row2; r++) {
            for (int c = col1; c <= col2; c++) {
                sum += matrix[r][c];
            }
        }
        return sum;
    }
};
```

---

# Approach 2 — Optimal / 2D Prefix Sum Table

## Idea

Precompute a prefix sum table `prefix` where `prefix[r][c]` holds the sum of every cell in the rectangle from `(0,0)` to `(r-1,c-1)`. Any rectangle sum can then be computed using inclusion-exclusion: take the full rectangle from the origin to the bottom-right corner, then subtract the two overlapping strips above and to the left, and add back the doubly-subtracted top-left corner.

## Dry Run

```text
matrix = [[3, 0, 1],
          [5, 6, 3],
          [1, 2, 0]]
```

Build prefix (size `(rows+1) x (cols+1)`, zero-padded):

```text
prefix[1][1] = 3
prefix[1][2] = 3+0 = 3
prefix[1][3] = 3+0+1 = 4
prefix[2][1] = 3+5 = 8
prefix[2][2] = 3+0+5+6 = 14
prefix[2][3] = 3+0+1+5+6+3 = 18
prefix[3][1] = 3+5+1 = 9
prefix[3][2] = 3+0+5+6+1+2 = 17
prefix[3][3] = whole matrix sum = 21
```

`sumRegion(1, 1, 2, 2)`:

```text
prefix[3][3] - prefix[1][3] - prefix[3][1] + prefix[1][1]
= 21 - 4 - 9 + 3 = 11
```

Matches the brute-force result.

## Algorithm

1. Constructor:

   * Build `prefix` table of size `(rows+1) x (cols+1)`, all zero-initialized.
   * For each cell `(r, c)`: `prefix[r+1][c+1] = matrix[r][c] + prefix[r][c+1] + prefix[r+1][c] - prefix[r][c]`.
2. `sumRegion(row1, col1, row2, col2)`:

   * Return `prefix[row2+1][col2+1] - prefix[row1][col2+1] - prefix[row2+1][col1] + prefix[row1][col1]`.

## Complexity

* **Time:** `O(rows * cols)` for the constructor, `O(1)` per `sumRegion` call

  * The constructor does one full pass to build the prefix table; each query afterward is four lookups and three arithmetic operations.
* **Space:** `O(rows * cols)`

  * For the `prefix` table itself, one row and column larger than `matrix`.

## Notes / Tips

* This is the direct 2D extension of the 1D prefix sum idea from LC 303 — same "precompute once, query in O(1)" principle, just with inclusion-exclusion added for the second dimension.
* Padding the prefix table with an extra row and column of zeros (`(rows+1) x (cols+1)`) avoids special-casing queries that touch row `0` or column `0`.
* Common mistake: mixing up which terms get added vs. subtracted in the inclusion-exclusion formula — the top-left corner is added back because it gets subtracted twice (once in each overlapping strip).

## Code

```cpp
class NumMatrix {
public:
    vector<vector<int>> prefix;

    NumMatrix(vector<vector<int>>& matrix) {
        int rows = matrix.size(), cols = matrix[0].size();
        prefix.assign(rows + 1, vector<int>(cols + 1, 0));

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                prefix[r + 1][c + 1] = matrix[r][c] + prefix[r][c + 1] + prefix[r + 1][c] - prefix[r][c];
            }
        }
    }

    int sumRegion(int row1, int col1, int row2, int col2) {
        return prefix[row2 + 1][col2 + 1] - prefix[row1][col2 + 1] - prefix[row2 + 1][col1] + prefix[row1][col1];
    }
};
```

---

## Key Template

```text
prefix[0][*] = 0, prefix[*][0] = 0

for r in 0..rows-1:
    for c in 0..cols-1:
        prefix[r+1][c+1] = matrix[r][c] + prefix[r][c+1] + prefix[r+1][c] - prefix[r][c]

sumRegion(row1, col1, row2, col2) =
    prefix[row2+1][col2+1] - prefix[row1][col2+1] - prefix[row2+1][col1] + prefix[row1][col1]
```