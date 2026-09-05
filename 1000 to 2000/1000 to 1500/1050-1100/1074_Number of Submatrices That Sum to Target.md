# LeetCode 1074 — Number of Submatrices That Sum to Target

## Metadata

* **LeetCode:** 1074
* **Problem:** Number of Submatrices That Sum to Target
* **Difficulty:** Hard
* **Topics:** Array, Hash Map, Matrix, Prefix Sum
* **Pattern:** 2D Prefix Sum + Row Compression + Subarray Sum Equals K
* **Key Technique:** Fix a pair of rows, collapse the matrix between them into a 1D column-sum array, then reuse the "subarray sum equals k" hash map trick
* **Optimal Complexity:** `O(rows² * cols)` Time, `O(cols)` Auxiliary Space

---

## Problem Statement

Given a 2D matrix and an integer `target`, return the number of non-empty submatrices whose sum equals `target`.

---

## Approaches

1. **Brute Force — Check Every Submatrix Directly**
2. **Better — 2D Prefix Sum + Nested Check**
3. **Optimal — Row Compression + Subarray Sum Equals K**

---

# Approach 1 — Brute Force / Check Every Submatrix Directly

## Idea

A submatrix is defined by four boundaries: `row1, row2, col1, col2`. For every possible combination of these boundaries, sum all the cells inside directly and check against `target`.

## Dry Run

```text
matrix = [[1,-1],[-1,1]], target = 0
```

Submatrix `(0,0)-(0,0)`:

```text
sum = 1 → no
```

Submatrix `(0,0)-(1,1)` (whole matrix):

```text
sum = 1 + (-1) + (-1) + 1 = 0 → match! count = 1
```

Submatrix `(0,0)-(0,1)`:

```text
sum = 1 + (-1) = 0 → match! count = 2
```

Continue for all boundary combinations to reach the full count.

## Algorithm

1. For each `row1` from `0` to `rows-1`:
2. For each `row2` from `row1` to `rows-1`:
3. For each `col1` from `0` to `cols-1`:
4. For each `col2` from `col1` to `cols-1`:

   * Sum every cell in `matrix[row1..row2][col1..col2]` directly.
   * If the sum equals `target`, increment `count`.
5. Return `count`.

## Complexity

* **Time:** `O(rows² * cols² * rows * cols)`

  * Four nested loops choose the boundaries (`O(rows² * cols²)`), and each candidate submatrix requires summing all its cells directly (`O(rows * cols)` worst case).
* **Space:** `O(1)`

  * Only a running sum and counter per candidate — no extra structures allocated.

## Notes / Tips

* Extremely slow — recomputing the full sum for every candidate submatrix from scratch is the main bottleneck.
* Useful only to confirm correctness on tiny matrices before optimizing.

## Code

```cpp
class Solution {
public:
    int numSubmatrixSumTarget(vector<vector<int>>& matrix, int target) {
        int rows = matrix.size(), cols = matrix[0].size();
        int count = 0;

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
                        if (sum == target) {
                            count++;
                        }
                    }
                }
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Better / 2D Prefix Sum + Nested Check

## Idea

Precompute a 2D prefix sum table where `prefix[r][c]` holds the sum of the rectangle from `(0,0)` to `(r-1,c-1)`. Any submatrix sum can then be computed in `O(1)` using inclusion-exclusion, removing the innermost double loop from Approach 1.

## Dry Run

```text
matrix = [[1,-1],[-1,1]]
```

Build 2D prefix sum (size `(rows+1) x (cols+1)`, all zero-padded):

```text
prefix[1][1] = 1
prefix[1][2] = 1 + (-1) = 0
prefix[2][1] = 1 + (-1) = 0
prefix[2][2] = matrix sum of whole grid = 0
```

Submatrix `(0,0)-(1,1)` (whole matrix) sum:

```text
prefix[2][2] - prefix[0][2] - prefix[2][0] + prefix[0][0]
= 0 - 0 - 0 + 0 = 0 → matches target 0
```

Submatrix `(0,0)-(0,1)` sum:

```text
prefix[1][2] - prefix[0][2] - prefix[1][0] + prefix[0][0]
= 0 - 0 - 0 + 0 = 0 → matches target 0
```

## Algorithm

1. Build `prefix` table of size `(rows+1) x (cols+1)`, all zero-initialized.
2. For each cell `(r, c)`: `prefix[r+1][c+1] = matrix[r][c] + prefix[r][c+1] + prefix[r+1][c] - prefix[r][c]`.
3. For each combination of `row1 <= row2` and `col1 <= col2`:

   * `sum = prefix[row2+1][col2+1] - prefix[row1][col2+1] - prefix[row2+1][col1] + prefix[row1][col1]`.
   * If `sum == target`, increment `count`.
4. Return `count`.

## Complexity

* **Time:** `O(rows² * cols²)`

  * Building the prefix table is `O(rows * cols)`, but checking every combination of boundaries is still `O(rows² * cols²)`, each check now `O(1)`.
* **Space:** `O(rows * cols)`

  * For the 2D prefix sum table.

## Notes / Tips

* Removes the innermost summation loop entirely compared to Approach 1, but the boundary-checking loops are still quadratic in both dimensions.
* This is the natural extension of the 1D prefix sum idea (LC 303) into two dimensions — same inclusion-exclusion principle used for any 2D range-sum query.

## Code

```cpp
class Solution {
public:
    int numSubmatrixSumTarget(vector<vector<int>>& matrix, int target) {
        int rows = matrix.size(), cols = matrix[0].size();
        vector<vector<int>> prefix(rows + 1, vector<int>(cols + 1, 0));

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                prefix[r + 1][c + 1] = matrix[r][c] + prefix[r][c + 1] + prefix[r + 1][c] - prefix[r][c];
            }
        }

        int count = 0;
        for (int row1 = 0; row1 < rows; row1++) {
            for (int row2 = row1; row2 < rows; row2++) {
                for (int col1 = 0; col1 < cols; col1++) {
                    for (int col2 = col1; col2 < cols; col2++) {
                        int sum = prefix[row2 + 1][col2 + 1] - prefix[row1][col2 + 1]
                                - prefix[row2 + 1][col1] + prefix[row1][col1];
                        if (sum == target) {
                            count++;
                        }
                    }
                }
            }
        }

        return count;
    }
};
```

---

# Approach 3 — Optimal / Row Compression + Subarray Sum Equals K

## Idea

Fix a pair of rows `(row1, row2)`. Collapse every column between these two rows into a single value: the sum of that column's cells from `row1` to `row2`. This turns the 2D problem into a 1D "subarray sum equals target" problem (same as LC 560) on the collapsed column-sum array — solved in linear time with a hash map. Repeat this for every pair of rows.

## Dry Run

```text
matrix = [[1,-1],[-1,1]], target = 0
```

Fix `row1 = 0, row2 = 0`:

```text
column sums = [1, -1]
```

Run LC560-style subarray-sum-equals-target on `[1, -1]` with `target = 0`:

```text
prefixMap = {0: 1}, sum = 0
1  → sum=1, need 1 → 0 found → count += 0; map[1]=1
-1 → sum=0, need 0 → 0 found in map → count += 1; map[0]=2
```

Contributes `1` to total count.

Fix `row1 = 0, row2 = 1`:

```text
column sums = [1 + (-1), -1 + 1] = [0, 0]
```

Run LC560-style check on `[0, 0]` with `target = 0`:

```text
prefixMap = {0: 1}, sum = 0
0 → sum=0, need 0 → 1 found → count += 1; map[0]=2
0 → sum=0, need 0 → 2 found → count += 2; map[0]=3
```

Contributes `3` to total count.

Fix `row1 = 1, row2 = 1`:

```text
column sums = [-1, 1]
```

Same shape as the first row pair → contributes `1`.

Total:

```text
1 + 3 + 1 = 5
```

## Algorithm

1. For each `row1` from `0` to `rows-1`:

   * Initialize `colSum` array of size `cols`, all `0`.
   * For each `row2` from `row1` to `rows-1`:

     * Add `matrix[row2][c]` into `colSum[c]` for every column `c` (extends the row range by one row each iteration).
     * Run the LC560 "subarray sum equals target" hash map technique on `colSum` and add the result to `count`.
2. Return `count`.

## Complexity

* **Time:** `O(rows² * cols)`

  * `O(rows²)` pairs of rows, and for each pair, updating `colSum` and running the linear subarray-sum-equals-k check both take `O(cols)`.
* **Space:** `O(cols)`

  * For the `colSum` array and the hash map used inside the subarray-sum-equals-k check (bounded by the number of columns).

## Notes / Tips

* This is a direct application of LC 560 nested inside a row-pair loop — recognizing that fixing two rows reduces a 2D matrix to a 1D array is the entire trick.
* `colSum` is built incrementally as `row2` increases, rather than resumming the full range each time, keeping the row-pair loop itself at `O(rows²)` instead of `O(rows³)`.
* The inner hash map must be reset for every new `row1` (starting a completely new column-sum array), but `colSum` itself carries over correctly across increasing `row2`.

## Code

```cpp
class Solution {
public:
    int numSubmatrixSumTarget(vector<vector<int>>& matrix, int target) {
        int rows = matrix.size(), cols = matrix[0].size();
        int count = 0;

        for (int row1 = 0; row1 < rows; row1++) {
            vector<int> colSum(cols, 0);

            for (int row2 = row1; row2 < rows; row2++) {
                for (int c = 0; c < cols; c++) {
                    colSum[c] += matrix[row2][c];
                }

                unordered_map<int, int> prefixCount;
                prefixCount[0] = 1;
                int sum = 0;

                for (int c = 0; c < cols; c++) {
                    sum += colSum[c];

                    if (prefixCount.find(sum - target) != prefixCount.end()) {
                        count += prefixCount[sum - target];
                    }

                    prefixCount[sum]++;
                }
            }
        }

        return count;
    }
};
```

---

## Key Template

```text
count = 0

for row1 in 0..rows-1:
    colSum = array of size cols, all 0

    for row2 in row1..rows-1:
        for c in 0..cols-1:
            colSum[c] += matrix[row2][c]

        # LC560-style subarray sum equals target on colSum
        prefixCount = {0: 1}
        sum = 0
        for c in 0..cols-1:
            sum += colSum[c]
            count += prefixCount.get(sum - target, 0)
            prefixCount[sum] += 1

return count
```