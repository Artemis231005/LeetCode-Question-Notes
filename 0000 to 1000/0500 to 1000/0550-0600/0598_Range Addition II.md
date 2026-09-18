# LeetCode 598 — Range Addition II

## Metadata

* **LeetCode:** 598
* **Problem:** Range Addition II
* **Difficulty:** Easy
* **Topics:** Array, Math
* **Pattern:** Rectangle Intersection Shortcut
* **Key Technique:** Since every operation's rectangle always starts at `(0,0)`, the region covered by **every** operation is exactly `[0, min(a)-1] x [0, min(b)-1]` — that overlap region is guaranteed to hold the maximum value, with no need to build the matrix at all
* **Optimal Complexity:** `O(len(ops))` Time, `O(1)` Space

---

## Problem Statement

Given an `m x n` matrix initialized to all `0`s, and a list of operations `ops[i] = [ai, bi]` (each incrementing every cell in the rectangle `[0, ai-1] x [0, bi-1]` by `1`), return the count of the maximum integer in the matrix after performing all operations.

---

## Approaches

1. **Brute Force — Build the Matrix and Scan for the Maximum**
2. **Optimal — Rectangle Intersection Shortcut**

---

# Approach 1 — Brute Force / Build the Matrix and Scan for the Maximum

## Idea

Actually construct the `m x n` matrix, apply every operation by incrementing each cell in its rectangle directly, then scan the finished matrix to find the maximum value and count how many cells hold it.

## Dry Run

```text
m = 3, n = 3, ops = [[2,2],[3,3]]
```

Apply `[2,2]` (increment rows 0-1, cols 0-1):

```text
[[1,1,0],
 [1,1,0],
 [0,0,0]]
```

Apply `[3,3]` (increment rows 0-2, cols 0-2 — the whole matrix):

```text
[[2,2,1],
 [2,2,1],
 [1,1,1]]
```

Scan for the maximum value `2`, and count its occurrences: `4` cells (`(0,0),(0,1),(1,0),(1,1)`).

Return `4`.

## Algorithm

1. Initialize an `m x n` matrix, all zeros.
2. For each operation `[a, b]`:

   * For `r` from `0` to `a-1`:

     * For `c` from `0` to `b-1`:

       * `matrix[r][c] += 1`.
3. Find the maximum value in the matrix, then count how many cells equal it.
4. Return that count.

## Complexity

* **Time:** `O(len(ops) * m * n)`

  * Each operation can directly touch up to the entire `m x n` matrix.
* **Space:** `O(m * n)`

  * For the full matrix.

## Notes / Tips

* Since every operation's rectangle always starts at `(0,0)`, the top-left corner of the matrix is covered by **every single operation**. It can never be anything but the maximum (or tied for it), which is exactly the insight that removes the need to build the matrix at all in Approach 2.
* Building the actual matrix is unnecessary work here as the answer only depends on the *smallest* rectangle among all operations, not on any of the matrix's individual cell values.

## Code

```cpp
class Solution {
public:
    int maxCount(int m, int n, vector<vector<int>>& ops) {
        vector<vector<int>> matrix(m, vector<int>(n, 0));

        for (auto& op : ops) {
            int a = op[0], b = op[1];
            for (int r = 0; r < a; r++) {
                for (int c = 0; c < b; c++) {
                    matrix[r][c]++;
                }
            }
        }

        int maxVal = 0;
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                maxVal = max(maxVal, matrix[r][c]);
            }
        }

        int count = 0;
        for (int r = 0; r < m; r++) {
            for (int c = 0; c < n; c++) {
                if (matrix[r][c] == maxVal) {
                    count++;
                }
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Optimal / Rectangle Intersection Shortcut

## Idea

Every operation's rectangle spans from `(0,0)` to `(a-1, b-1)` — they're all "anchored" at the same corner. This means the region `[0, minA-1] x [0, minB-1]` (where `minA` and `minB` are the smallest `a` and `b` values seen across all operations) is contained within **every single operation's** rectangle, and is therefore incremented by every operation — making it the maximum value in the matrix. No cell outside this region can reach that same count, since at least one operation (whichever contributed the smallest `a` or `b`) doesn't cover it. So the answer is simply the area of this intersection region: `minA * minB`.

## Dry Run

```text
m = 3, n = 3, ops = [[2,2],[3,3]]
```

Track the minimum `a` and minimum `b` across all operations:

```text
op [2,2]: minA=2, minB=2
op [3,3]: minA=min(2,3)=2, minB=min(2,3)=2
```

Final `minA = 2`, `minB = 2`.

Answer: `minA * minB = 2 * 2 = 4`, matching the brute-force result.

### Edge case: no operations

```text
m = 3, n = 3, ops = []
```

With no operations, the matrix stays all zeros — every cell ties for the maximum value `0`. Answer: `m * n = 9`.

## Algorithm

1. If `ops` is empty, return `m * n` (every cell is tied at value `0`).
2. Initialize `minA = m`, `minB = n`.
3. For each operation `[a, b]`:

   * `minA = min(minA, a)`.
   * `minB = min(minB, b)`.
4. Return `minA * minB`.

## Complexity

* **Time:** `O(len(ops))`

  * A single pass over the operations list, tracking two running minimums.
* **Space:** `O(1)`

  * Only two running minimum variables — no matrix or extra structures needed.

## Notes / Tips

* This is a special-case shortcut rather than a general difference-array or prefix-sum technique — it works specifically because every rectangle shares the same anchor corner `(0,0)`, which guarantees the overlap of *all* operations is itself always a rectangle (rather than a more complex, possibly disconnected shape it would be for arbitrarily positioned rectangles).
* The `ops` empty edge case is easy to overlook — without any operations, `minA`/`minB` would incorrectly default to something other than the full matrix dimensions, unless explicitly initialized to `m` and `n` or handled as a separate early return.
* Recognize that a problem's structure allows skipping the underlying data structure entirely (here, the matrix) because the question only asks about the *count/size* of a specific derived quantity rather than the full structure. This is a broader pattern-spotting skill worth carrying into similar-looking matrix/range problems.

## Code

```cpp
class Solution {
public:
    int maxCount(int m, int n, vector<vector<int>>& ops) {
        int minA = m, minB = n;

        for (auto& op : ops) {
            minA = min(minA, op[0]);
            minB = min(minB, op[1]);
        }

        return minA * minB;
    }
};
```

---

## Key Template

```text
minA = m
minB = n

for [a, b] in ops:
    minA = min(minA, a)
    minB = min(minB, b)

return minA * minB
```