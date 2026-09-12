# LeetCode 363 — Max Sum of Rectangle No Larger Than K

## Metadata

* **LeetCode:** 363
* **Problem:** Max Sum of Rectangle No Larger Than K
* **Difficulty:** Hard
* **Topics:** Array, Binary Search, Matrix, Ordered Set, Prefix Sum
* **Pattern:** Column Compression + Sorted Prefix Sum Binary Search
* **Key Technique:** Fix a pair of columns, collapse the matrix between them into a 1D row-sum array, then reuse the "max subarray sum no more than k" trick via a sorted set of prefix sums and binary search
* **Optimal Complexity:** `O(min(rows,cols)² * max(rows,cols) * log(max(rows,cols)))` Time, `O(max(rows,cols))` Auxiliary Space

---

## Problem Statement

Given a non-empty 2D matrix `matrix` and an integer `k`, find the maximum sum of a rectangular submatrix such that its sum is `<= k`. It's guaranteed that at least one rectangle has a sum `<= k`.

---

## Approaches

1. **Brute Force — Check Every Submatrix Directly**
2. **Better — 2D Prefix Sum + Nested Boundary Check**
3. **Optimal — Row Compression + Sorted Prefix Sum Binary Search**

---

# Approach 1 — Brute Force / Check Every Submatrix Directly

## Idea

A submatrix is defined by four boundaries: `row1, row2, col1, col2`. For every possible combination of these boundaries, sum all the cells inside directly, and track the largest sum seen that's still `<= k`.

## Dry Run

```text
matrix = [[1,0,1],[0,-2,3]], k = 2
```

Submatrix `(0,0)-(0,0)`:

```text
sum = 1 → <= 2, best so far = 1
```

Submatrix `(0,0)-(1,2)` (whole matrix):

```text
sum = 1+0+1+0-2+3 = 3 → > 2, skip
```

Submatrix `(0,2)-(1,2)`:

```text
sum = 1 + 3 = 4 → > 2, skip
```

Submatrix `(1,0)-(1,2)`:

```text
sum = 0 + (-2) + 3 = 1 → <= 2, still less than best of 1... continue checking others
```

Continuing through all boundary combinations, the best valid sum found is `2` (from submatrix `(0,0)-(1,1)`: `1+0+0-2=-1`... actual best turns out to be submatrix `(1,0)-(1,1)`: `0+(-2)=-2`, or `(0,0)-(0,2)`: `1+0+1=2`) → answer `2`.

## Algorithm

1. For each `row1` from `0` to `rows-1`:
2. For each `row2` from `row1` to `rows-1`:
3. For each `col1` from `0` to `cols-1`:
4. For each `col2` from `col1` to `cols-1`:

   * Sum every cell in `matrix[row1..row2][col1..col2]` directly.
   * If the sum is `<= k`, update `best = max(best, sum)`.
5. Return `best`.

## Complexity

* **Time:** `O(rows² * cols² * rows * cols)`

  * Four nested loops choose the boundaries (`O(rows² * cols²)`), and each candidate submatrix requires summing all its cells directly (`O(rows * cols)` worst case).
* **Space:** `O(1)`

  * Only a running sum and best tracker per candidate — no extra structures allocated.

## Notes / Tips

* Extremely slow — recomputing the full sum for every candidate submatrix from scratch is the main bottleneck.
* Useful only to confirm correctness on tiny matrices before optimizing.

## Code

```cpp
class Solution {
public:
    int maxSumSubmatrix(vector<vector<int>>& matrix, int k) {
        int rows = matrix.size(), cols = matrix[0].size();
        int best = INT_MIN;

        for (int row1 = 0; row1 < rows; row1++) {
            for (int row2 = row1; row2 < rows; row2++) {
                for (int col1 = 0; col1 < cols; col1++) {
                    for (int col2 = col1; col2 < cols; col2++) {
                        int sum = 0;
                        for (int r = row1; r <= row2; r++) {
                            for (int c = col1; c <= col2; c++) {
                                sum += matrix[r][c];
                            }
                        }
                        if (sum <= k) {
                            best = max(best, sum);
                        }
                    }
                }
            }
        }

        return best;
    }
};
```

---

# Approach 2 — Better / 2D Prefix Sum + Nested Boundary Check

## Idea

Precompute a 2D prefix sum table so any submatrix sum can be looked up in `O(1)` using inclusion-exclusion, removing the innermost double summation loop from Approach 1.

## Dry Run

```text
matrix = [[1,0,1],[0,-2,3]]
```

Build 2D prefix sum (size `(rows+1) x (cols+1)`, zero-padded):

```text
prefix[1][1] = 1
prefix[1][2] = 1
prefix[1][3] = 2
prefix[2][1] = 1
prefix[2][2] = -1
prefix[2][3] = 3
```

Submatrix `(0,0)-(0,2)` sum:

```text
prefix[1][3] - prefix[0][3] - prefix[1][0] + prefix[0][0] = 2 - 0 - 0 + 0 = 2 → <= k=2 → best = 2
```

Continue checking all boundary combinations the same way to confirm `2` is the best.

## Algorithm

1. Build `prefix` table of size `(rows+1) x (cols+1)`, all zero-initialized.
2. For each cell `(r, c)`: `prefix[r+1][c+1] = matrix[r][c] + prefix[r][c+1] + prefix[r+1][c] - prefix[r][c]`.
3. For each combination of `row1 <= row2` and `col1 <= col2`:

   * `sum = prefix[row2+1][col2+1] - prefix[row1][col2+1] - prefix[row2+1][col1] + prefix[row1][col1]`.
   * If `sum <= k`, update `best = max(best, sum)`.
4. Return `best`.

## Complexity

* **Time:** `O(rows² * cols²)`

  * Building the prefix table is `O(rows * cols)`, but checking every combination of boundaries is still `O(rows² * cols²)`, each check now `O(1)`.
* **Space:** `O(rows * cols)`

  * For the 2D prefix sum table.

## Notes / Tips

* Removes the innermost summation loop entirely compared to Approach 1, but the boundary-checking loops are still quadratic in both dimensions — for large matrices this is still too slow.
* This is the natural stepping stone toward Approach 3: fixing a pair of rows/columns and asking "what's the best sub-range sum `<= k`" is exactly what the column-compression technique specializes.

## Code

```cpp
class Solution {
public:
    int maxSumSubmatrix(vector<vector<int>>& matrix, int k) {
        int rows = matrix.size(), cols = matrix[0].size();
        vector<vector<int>> prefix(rows + 1, vector<int>(cols + 1, 0));

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                prefix[r + 1][c + 1] = matrix[r][c] + prefix[r][c + 1] + prefix[r + 1][c] - prefix[r][c];
            }
        }

        int best = INT_MIN;
        for (int row1 = 0; row1 < rows; row1++) {
            for (int row2 = row1; row2 < rows; row2++) {
                for (int col1 = 0; col1 < cols; col1++) {
                    for (int col2 = col1; col2 < cols; col2++) {
                        int sum = prefix[row2 + 1][col2 + 1] - prefix[row1][col2 + 1]
                                - prefix[row2 + 1][col1] + prefix[row1][col1];
                        if (sum <= k) {
                            best = max(best, sum);
                        }
                    }
                }
            }
        }

        return best;
    }
};
```

---

# Approach 3 — Optimal / Row Compression + Sorted Prefix Sum Binary Search

## Idea

Fix a pair of columns `(col1, col2)`. 
Collapse every row between these two columns into a single value: the sum of that row's cells from `col1` to `col2`. This turns the 2D problem into a 1D "max subarray sum `<= k`" problem on the collapsed row-sum array. 

## Dry Run

```text
matrix = [[1,0,1],[0,-2,3]], k = 2
```

Fix `col1 = 0, col2 = 2` (whole width):

```text
row sums = [1+0+1, 0-2+3] = [2, 1]
```

Run the 1D max-subarray-sum-`<=k` check on `[2, 1]`:

```text
prefixSet = {0}
prefix = 0

element 2: prefix = 2
   find smallest prefixSet value >= prefix - k = 2 - 2 = 0 → found 0
   candidate sum = 2 - 0 = 2 → best = 2
   insert 2 into prefixSet → {0, 2}

element 1: prefix = 3
   find smallest prefixSet value >= prefix - k = 3 - 2 = 1 → found 2
   candidate sum = 3 - 2 = 1 → best stays 2
   insert 3 into prefixSet → {0, 2, 3}
```

This column range gives best `2`. Repeating for all other column pairs won't beat this — final answer `2`, matching the earlier approaches.

## Algorithm

1. If `rows < cols`, transpose the matrix (so the "pair" loop runs over the smaller dimension for efficiency).
2. For each `col1` from `0` to `cols-1`:

   * Initialize `rowSum` array of size `rows`, all `0`.
   * For each `col2` from `col1` to `cols-1`:

     * Add `matrix[r][col2]` into `rowSum[r]` for every row `r` (extends the column range by one column each iteration).
     * Run the 1D "max subarray sum `<= k`" check on `rowSum`:

       * Initialize a sorted set with `{0}`, and `prefix = 0`.
       * For each value in `rowSum`:

         * `prefix += value`.
         * Binary search the sorted set for the smallest element `>= prefix - k`.
         * If found, update `best = max(best, prefix - thatElement)`.
         * Insert `prefix` into the sorted set.
3. Return `best`.

## Complexity

* **Time:** `O(min(rows,cols)² * max(rows,cols) * log(max(rows,cols)))`

  * `O(min(rows,cols)²)` pairs of columns (after transposing to make columns the smaller dimension); for each pair, updating `rowSum` is `O(max(rows,cols))`, and the 1D check with a sorted set is `O(max(rows,cols) * log(max(rows,cols)))`.
* **Space:** `O(max(rows,cols))`

  * For the `rowSum` array and the sorted set used inside the 1D check.

## Notes / Tips

* This is a direct application of "max subarray sum no more than k" (a variant built on LC 560's prefix-sum idea, but using a sorted structure with binary search instead of a hash map, since here we need the smallest qualifying prefix rather than an exact match) nested inside a column-pair loop — the same overall shape as LC 1074's optimal approach, but solving a harder 1D subproblem at each step.
* Transposing when `rows < cols` keeps the outer `O(dim²)` loop over the smaller dimension, which matters a lot for skewed matrices (e.g. very wide but short).
* A `std::set<int>` (ordered by value) with `lower_bound` gives the "smallest prefix `>= prefix - k`" lookup in `O(log n)` — a hash map won't work here since an exact match isn't guaranteed to exist, only the tightest bound.

## Code

```cpp
class Solution {
public:
    int maxSumSubmatrix(vector<vector<int>>& matrix, int k) {
        int rows = matrix.size(), cols = matrix[0].size();

        if (rows < cols) {
            vector<vector<int>> transposed(cols, vector<int>(rows));
            for (int r = 0; r < rows; r++) {
                for (int c = 0; c < cols; c++) {
                    transposed[c][r] = matrix[r][c];
                }
            }
            matrix = transposed;
            swap(rows, cols);
        }

        int best = INT_MIN;

        for (int col1 = 0; col1 < cols; col1++) {
            vector<int> rowSum(rows, 0);

            for (int col2 = col1; col2 < cols; col2++) {
                for (int r = 0; r < rows; r++) {
                    rowSum[r] += matrix[r][col2];
                }

                set<int> prefixSet;
                prefixSet.insert(0);
                int prefix = 0;

                for (int val : rowSum) {
                    prefix += val;

                    auto it = prefixSet.lower_bound(prefix - k);
                    if (it != prefixSet.end()) {
                        best = max(best, prefix - *it);
                    }

                    prefixSet.insert(prefix);
                }
            }
        }

        return best;
    }
};
```

---

## Key Template

```text
if rows < cols: transpose(matrix), swap(rows, cols)

best = -infinity

for col1 in 0..cols-1:
    rowSum = array of size rows, all 0

    for col2 in col1..cols-1:
        for r in 0..rows-1:
            rowSum[r] += matrix[r][col2]

        # 1D max subarray sum <= k, via sorted prefix sums
        prefixSet = {0}
        prefix = 0
        for val in rowSum:
            prefix += val
            candidate = smallest element in prefixSet >= prefix - k
            if candidate exists:
                best = max(best, prefix - candidate)
            prefixSet.insert(prefix)

return best
```