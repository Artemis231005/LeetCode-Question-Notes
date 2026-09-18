# LeetCode 2536 — Increment Submatrices by One

## Metadata

* **LeetCode:** 2536
* **Problem:** Increment Submatrices by One
* **Difficulty:** Medium
* **Topics:** Array, Matrix, Prefix Sum
* **Pattern:** 2D Difference Array (Range Update, Point Query)
* **Key Technique:** Mark each submatrix's effect at its four corners using inclusion-exclusion, then a single 2D prefix sum sweep applies every update to every cell at once
* **Optimal Complexity:** `O(n² + q)` Time, `O(n²)` Auxiliary Space

---

## Problem Statement

Given an integer `n` and a list of queries `[row1, col1, row2, col2]`, start with an `n x n` matrix of zeros, and for each query add `1` to every cell in the inclusive rectangle from `(row1, col1)` to `(row2, col2)`. Return the final matrix after applying all queries.

---

## Approaches

1. **Brute Force — Apply Every Query to Every Cell in Its Range**
2. **Optimal — 2D Difference Array**

---

# Approach 1 — Brute Force / Apply Every Query to Every Cell in Its Range

## Idea

For each query, directly loop over every cell in the given rectangle and add `1` to it.

## Dry Run

```text
n = 3, queries = [[1,1,2,2],[0,0,1,1]]
```

Apply `[1,1,2,2]`:

```text
add 1 to rows 1-2, cols 1-2
mat = [[0,0,0],
       [0,1,1],
       [0,1,1]]
```

Apply `[0,0,1,1]`:

```text
add 1 to rows 0-1, cols 0-1
mat = [[1,1,0],
       [1,2,1],
       [0,1,1]]
```

Final matrix matches the expected output.

## Algorithm

1. Initialize an `n x n` matrix `mat`, all zeros.
2. For each query `[row1, col1, row2, col2]`:

   * For `r` from `row1` to `row2`:

     * For `c` from `col1` to `col2`:

       * `mat[r][c] += 1`.
3. Return `mat`.

## Complexity

* **Time:** `O(q * n²)`

  * Each of the `q` queries can directly touch up to `n²` cells (the entire matrix, in the worst case).
* **Space:** `O(1)` (beyond the required output matrix)

  * No extra structures needed — the matrix is updated in place.

## Notes / Tips

* Directly adding `1` to every cell in a query's rectangle is redundant — a rectangle's effect only needs to be marked once, at its corners, rather than applied to every cell inside it, exactly like the 1D difference array trick (LC 1109, LC 2848) but extended into two dimensions.
* This is essentially the "Approach 1" pattern from LC 1074 and LC 363 (checking/summing every rectangle directly), just applying an increment instead of computing a sum.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> rangeAddQueries(int n, vector<vector<int>>& queries) {
        vector<vector<int>> mat(n, vector<int>(n, 0));

        for (auto& q : queries) {
            int row1 = q[0], col1 = q[1], row2 = q[2], col2 = q[3];

            for (int r = row1; r <= row2; r++) {
                for (int c = col1; c <= col2; c++) {
                    mat[r][c]++;
                }
            }
        }

        return mat;
    }
};
```

---

# Approach 2 — Optimal / 2D Difference Array

## Idea

Instead of incrementing every cell a query covers, mark the query's *effect* directly using a 2D difference array with the same inclusion-exclusion principle as a 2D prefix sum, but in reverse: add `+1` at the rectangle's top-left corner, subtract `1` just past its right edge, subtract `1` just past its bottom edge, and add `1` back at the corner just past both (to correct the double subtraction). After processing all queries, a single 2D prefix sum sweep converts these markers into the actual final value at every cell.

## Dry Run

```text
n = 3, queries = [[1,1,2,2],[0,0,1,1]]
```

Build difference array `diff` of size `(n+1) x (n+1)`, all zero.

Apply `[1,1,2,2]` (rows 1-2, cols 1-2 — so past-edge markers land at row 3, col 3):

```text
diff[1][1] += 1
diff[1][3] -= 1
diff[3][1] -= 1
diff[3][3] += 1
```

Apply `[0,0,1,1]` (rows 0-1, cols 0-1 — past-edge markers land at row 2, col 2):

```text
diff[0][0] += 1
diff[0][2] -= 1
diff[2][0] -= 1
diff[2][2] += 1
```

Combined `diff` (nonzero entries):

```text
diff[0][0]=1, diff[0][2]=-1
diff[1][1]=1, diff[1][3]=-1
diff[2][0]=-1, diff[2][2]=1
diff[3][1]=-1, diff[3][3]=1
```

Apply 2D prefix sum sweep (each cell = its own diff value + the cell above + the cell to the left - the cell diagonally above-left, accumulated left-to-right, top-to-bottom):

```text
row 0: [1, 1, 0]
row 1: [1, 2, 1]
row 2: [0, 1, 1]
```

Final matrix (trimmed to `n x n`, dropping the extra padding row/col):

```text
[[1,1,0],
 [1,2,1],
 [0,1,1]]
```

Matches the brute-force result.

## Algorithm

1. Create a difference array `diff` of size `(n+1) x (n+1)`, all zeros.
2. For each query `[row1, col1, row2, col2]`:

   * `diff[row1][col1] += 1`.
   * `diff[row1][col2 + 1] -= 1`.
   * `diff[row2 + 1][col1] -= 1`.
   * `diff[row2 + 1][col2 + 1] += 1`.
3. Build the final `n x n` matrix by sweeping `diff` with a running 2D prefix sum: `mat[r][c] = diff[r][c] + mat[r-1][c] + mat[r][c-1] - mat[r-1][c-1]` (treating out-of-bounds indices as `0`).
4. Return `mat`.

## Complexity

* **Time:** `O(n² + q)`

  * `O(q)` to apply all query markers, `O(n²)` to sweep through and accumulate the final matrix.
* **Space:** `O(n²)`

  * For the difference array, sized independently of `q`.

## Notes / Tips

* This is the exact 2D extension of the 1D difference array technique (LC 1109, LC 2848) — the four corner markers with alternating signs are the 2D analog of the `+1`/`-1` pair used in one dimension, both relying on inclusion-exclusion to cancel out correctly during the sweep.
* The sweep step here is structurally identical to building a 2D prefix sum table (as in LC 304) — the difference is *what's* being accumulated (increment markers instead of raw matrix values), not *how* it's accumulated.
* Padding `diff` to size `(n+1) x (n+1)` (rather than `n x n`) is essential so that a query touching the last row or column can still safely place its "past the edge" markers without going out of bounds.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> rangeAddQueries(int n, vector<vector<int>>& queries) {
        vector<vector<int>> diff(n + 1, vector<int>(n + 1, 0));

        for (auto& q : queries) {
            int row1 = q[0], col1 = q[1], row2 = q[2], col2 = q[3];

            diff[row1][col1] += 1;
            diff[row1][col2 + 1] -= 1;
            diff[row2 + 1][col1] -= 1;
            diff[row2 + 1][col2 + 1] += 1;
        }

        vector<vector<int>> mat(n, vector<int>(n, 0));

        for (int r = 0; r < n; r++) {
            for (int c = 0; c < n; c++) {
                int val = diff[r][c];
                if (r > 0) val += mat[r - 1][c];
                if (c > 0) val += mat[r][c - 1];
                if (r > 0 && c > 0) val -= mat[r - 1][c - 1];

                mat[r][c] = val;
            }
        }

        return mat;
    }
};
```

---

## Key Template

```text
diff = array of size (n+1) x (n+1), all 0

for [row1, col1, row2, col2] in queries:
    diff[row1][col1] += 1
    diff[row1][col2+1] -= 1
    diff[row2+1][col1] -= 1
    diff[row2+1][col2+1] += 1

mat = array of size n x n
for r in 0..n-1:
    for c in 0..n-1:
        val = diff[r][c]
        if r > 0: val += mat[r-1][c]
        if c > 0: val += mat[r][c-1]
        if r > 0 and c > 0: val -= mat[r-1][c-1]
        mat[r][c] = val

return mat
```