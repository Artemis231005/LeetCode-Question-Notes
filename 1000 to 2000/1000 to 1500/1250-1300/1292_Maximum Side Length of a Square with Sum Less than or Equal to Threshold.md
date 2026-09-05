# LeetCode 1292 — Maximum Side Length of a Square with Sum Less than or Equal to Threshold

## Metadata

* **LeetCode:** 1292
* **Problem:** Maximum Side Length of a Square with Sum Less than or Equal to Threshold
* **Difficulty:** Medium
* **Topics:** Array, Binary Search, Matrix, Prefix Sum
* **Pattern:** 2D Prefix Sum + Binary Search on Answer
* **Key Technique:** Since all values are non-negative, feasibility of a side length is monotonic — larger squares only ever cost more, so binary search on side length works
* **Optimal Complexity:** `O(rows * cols * log(min(rows, cols)))` Time, `O(rows * cols)` Space

---

## Problem Statement

Given a matrix `mat` of non-negative integers and an integer `threshold`, find the maximum side length of a square submatrix whose sum is `<= threshold`. Return `0` if no such square exists.

---

## Approaches

1. **Brute Force — Check Every Square Directly**
2. **Better — 2D Prefix Sum + Linear Scan Over Side Lengths**
3. **Optimal — 2D Prefix Sum + Binary Search on Side Length**

---

# Approach 1 — Brute Force / Check Every Square Directly

## Idea

For every possible top-left corner and every possible side length, sum all the cells inside that square directly, and track the largest side length whose sum stays within `threshold`.

## Dry Run

```text
mat = [[1,1,3,2,4,3,2]], threshold small example simplified:
mat = [[1,1],[1,1]], threshold = 3
```

Side length 1, corner `(0,0)`:

```text
sum = 1 → <= 3 → maxSide = 1
```

Side length 2, corner `(0,0)`:

```text
sum = 1+1+1+1 = 4 → > 3 → fails
```

No side-2 square fits → `maxSide = 1`.

## Algorithm

1. For each possible side length `side` from `1` to `min(rows, cols)`:
2. For each possible top-left corner `(r, c)` such that the square fits in the matrix:

   * Sum every cell in that `side x side` square directly.
   * If the sum is `<= threshold`, update `maxSide = max(maxSide, side)`.
3. Return `maxSide` (or `0` if none found).

## Complexity

* **Time:** `O(rows * cols * min(rows, cols)³)`

  * For each of `O(min(rows,cols))` side lengths, checking all `O(rows*cols)` corner positions, each requiring `O(side²)` work to sum the square directly.
* **Space:** `O(1)`

  * Only a running sum and max tracker — no extra structures allocated.

## Notes / Tips

* Recomputing the sum of each square from scratch is the main bottleneck — this is exactly what a 2D prefix sum table removes.
* Only useful for very small matrices to sanity-check the approach.

## Code

```cpp
class Solution {
public:
    int maxSideLength(vector<vector<int>>& mat, int threshold) {
        int rows = mat.size(), cols = mat[0].size();
        int maxSide = 0;

        for (int side = 1; side <= min(rows, cols); side++) {
            for (int r = 0; r + side <= rows; r++) {
                for (int c = 0; c + side <= cols; c++) {
                    int sum = 0;
                    for (int i = r; i < r + side; i++) {
                        for (int j = c; j < c + side; j++) {
                            sum += mat[i][j];
                        }
                    }
                    if (sum <= threshold) {
                        maxSide = max(maxSide, side);
                    }
                }
            }
        }

        return maxSide;
    }
};
```

---

# Approach 2 — Better / 2D Prefix Sum + Linear Scan Over Side Lengths

## Idea

Precompute a 2D prefix sum table so any square's sum can be looked up in `O(1)` using inclusion-exclusion. Then check every side length from `1` up to `min(rows, cols)`, scanning all valid corner positions at each length, and keep the largest side length for which at least one square fits within `threshold`.

## Dry Run

```text
mat = [[1,1],[1,1]], threshold = 3
```

Build prefix sum table (size `(rows+1) x (cols+1)`):

```text
prefix[1][1] = 1
prefix[1][2] = 2
prefix[2][1] = 2
prefix[2][2] = 4
```

Side length 1, corner `(0,0)`:

```text
sum = prefix[1][1] - prefix[0][1] - prefix[1][0] + prefix[0][0] = 1
1 <= 3 → maxSide = 1
```

Side length 2, corner `(0,0)`:

```text
sum = prefix[2][2] - prefix[0][2] - prefix[2][0] + prefix[0][0] = 4
4 > 3 → fails
```

No side-2 square fits → `maxSide = 1`.

## Algorithm

1. Build the 2D prefix sum table for `mat` (same as LC 304).
2. For each side length `side` from `1` to `min(rows, cols)`:

   * For each valid top-left corner `(r, c)`:

     * Look up the square's sum in `O(1)` using the prefix table.
     * If `sum <= threshold`, update `maxSide = max(maxSide, side)`.
3. Return `maxSide`.

## Complexity

* **Time:** `O(rows * cols * min(rows, cols))`

  * Building the prefix table is `O(rows * cols)`; then for each of `O(min(rows,cols))` side lengths, every corner position is checked in `O(1)`.
* **Space:** `O(rows * cols)`

  * For the 2D prefix sum table.

## Notes / Tips

* Removing the innermost `O(side²)` summation is the main win here — the same win as LC 1074's Approach 2 over Approach 1.
* Still checks every side length one by one from smallest to largest (or largest to smallest with early exit) rather than skipping ahead — this is what Approach 3 improves on.

## Code

```cpp
class Solution {
public:
    int maxSideLength(vector<vector<int>>& mat, int threshold) {
        int rows = mat.size(), cols = mat[0].size();
        vector<vector<int>> prefix(rows + 1, vector<int>(cols + 1, 0));

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                prefix[r + 1][c + 1] = mat[r][c] + prefix[r][c + 1] + prefix[r + 1][c] - prefix[r][c];
            }
        }

        int maxSide = 0;

        for (int side = 1; side <= min(rows, cols); side++) {
            bool found = false;

            for (int r = 0; r + side <= rows && !found; r++) {
                for (int c = 0; c + side <= cols && !found; c++) {
                    int sum = prefix[r + side][c + side] - prefix[r][c + side]
                            - prefix[r + side][c] + prefix[r][c];
                    if (sum <= threshold) {
                        found = true;
                    }
                }
            }

            if (found) {
                maxSide = side;
            }
        }

        return maxSide;
    }
};
```

---

# Approach 3 — Optimal / 2D Prefix Sum + Binary Search on Side Length

## Idea

Since all matrix values are non-negative, a bigger square can never have a smaller sum than a smaller square nested inside it. This means feasibility is monotonic: if some square of side `L` fits within `threshold`, then some square of side `L-1` also does. That monotonic structure means binary search can find the maximum feasible side length instead of checking every length one at a time.

## Dry Run

```text
mat = [[1,1,3,2,4],[1,1,3,2,4],[1,1,3,2,4]], threshold = 4
```

Binary search bounds: `low = 0`, `high = min(rows, cols) = 3`.

```text
mid = 1: check if any 1x1 square has sum <= 4 → yes (e.g. sum=1) → feasible → low = 1

mid = 2: check if any 2x2 square has sum <= 4
   e.g. top-left (0,0): sum = 1+1+1+1 = 4 → feasible → low = 2

mid = 2 (recompute since low=2, high=3): actual next mid = (2+3+1)/2 = 3
   check if any 3x3 square has sum <= 4
   e.g. top-left (0,0): sum = (1+1+3)*3 = 15 → too big for all positions → not feasible → high = 2
```

Loop ends with `low = 2`, `high = 2` → answer `2`.

## Algorithm

1. Build the 2D prefix sum table for `mat`.
2. Define a helper `feasible(side)` that checks (using the prefix table) whether any `side x side` square has sum `<= threshold`.
3. Binary search `side` between `0` and `min(rows, cols)`:

   * If `feasible(mid)`, the answer is at least `mid` — move `low = mid`.
   * Otherwise, the answer is smaller — move `high = mid - 1`.
4. Return the largest feasible side length found (`low`).

## Complexity

* **Time:** `O(rows * cols * log(min(rows, cols)))`

  * Building the prefix table is `O(rows * cols)`; binary search runs `O(log(min(rows,cols)))` iterations, each doing an `O(rows * cols)` feasibility check.
* **Space:** `O(rows * cols)`

  * For the 2D prefix sum table.

## Notes / Tips

* The monotonicity argument (bigger square sum >= any nested smaller square's sum, since values are non-negative) is what justifies binary search here — this wouldn't work if the matrix could contain negative values.
* This is the standard "binary search on the answer" pattern: whenever a yes/no feasibility check is monotonic in some parameter, binary searching that parameter beats scanning it linearly.
* The feasibility check itself is identical in shape to Approach 2's single-side-length scan — binary search just changes how many times it's called and at which side lengths.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> prefix;

    bool feasible(int side, int rows, int cols, int threshold) {
        for (int r = 0; r + side <= rows; r++) {
            for (int c = 0; c + side <= cols; c++) {
                int sum = prefix[r + side][c + side] - prefix[r][c + side]
                        - prefix[r + side][c] + prefix[r][c];
                if (sum <= threshold) {
                    return true;
                }
            }
        }
        return false;
    }

    int maxSideLength(vector<vector<int>>& mat, int threshold) {
        int rows = mat.size(), cols = mat[0].size();
        prefix.assign(rows + 1, vector<int>(cols + 1, 0));

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                prefix[r + 1][c + 1] = mat[r][c] + prefix[r][c + 1] + prefix[r + 1][c] - prefix[r][c];
            }
        }

        int low = 0, high = min(rows, cols);

        while (low < high) {
            int mid = low + (high - low + 1) / 2;

            if (feasible(mid, rows, cols, threshold)) {
                low = mid;
            } else {
                high = mid - 1;
            }
        }

        return low;
    }
};
```

---

## Key Template

```text
build 2D prefix sum table

function feasible(side):
    for every valid top-left corner:
        sum = O(1) lookup via prefix table
        if sum <= threshold: return true
    return false

low = 0, high = min(rows, cols)
while low < high:
    mid = low + (high - low + 1) / 2
    if feasible(mid): low = mid
    else: high = mid - 1

return low
```