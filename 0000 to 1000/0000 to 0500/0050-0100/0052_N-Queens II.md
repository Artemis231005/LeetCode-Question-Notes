# LeetCode 52 — N-Queens II

## Metadata

* **LeetCode:** 52
* **Problem:** N-Queens II
* **Difficulty:** Hard
* **Topics:** Array, Backtracking, Matrix
* **Pattern:** Backtracking, Constraint Checking
* **Key Pattern:** Place one queen per row and count valid configurations
* **Key Technique:** Track occupied columns and diagonals using sets
* **Key Template:** Backtracking / Choose → Explore → Undo
* **Optimal Complexity:** `O(N!)` time, `O(N)` auxiliary space

---

## Problem

Given an integer `n`, return the **number of distinct solutions** to the N-Queens puzzle.

Place `n` queens on an `n × n` chessboard such that:

* No two queens share the same row.
* No two queens share the same column.
* No two queens share the same diagonal.

Unlike **N-Queens (51)**, we do **not** need to construct or return the boards. We only need the count.

---

## Approach — Backtracking

### Idea

The logic is almost identical to **LeetCode 51 — N-Queens**.

Place one queen in each row.

For every position `(row, col)`, check:

```text
Column       → col
Main diagonal → row - col
Anti-diagonal → row + col
```

If the position is safe:

1. Place the queen.
2. Mark its column and diagonals as occupied.
3. Recursively solve the next row.
4. Remove the queen and unmark everything.

When `row == n`, a complete valid arrangement has been found, so increment the answer.

The major difference from **N-Queens I** is:

```text
51 → store every board
52 → only count valid boards
```

Therefore, no board needs to be stored in the final answer.

---

### Dry Run

For `n = 4`:

Start at row `0`.

```text
Row 0
├── Col 0
│   └── Explore...
│       └── eventually backtrack
├── Col 1
│   └── Explore...
│       └── finds solution
├── Col 2
│   └── Explore...
│       └── finds solution
└── Col 3
    └── Explore...
        └── eventually backtrack
```

The two complete solutions are:

```text
.Q..
...Q
Q...
..Q.
```

and

```text
..Q.
Q...
...Q
.Q..
```

We don't store these boards.

Instead:

```text
First complete board  → count = 1
Second complete board → count = 2
```

So:

```text
n = 4
answer = 2
```

---

### Algorithm

1. Initialize `count = 0`.
2. Create three sets:

   * `cols` → occupied columns.
   * `diag1` → occupied `row - col` diagonals.
   * `diag2` → occupied `row + col` diagonals.
3. Start backtracking from `row = 0`.
4. For every column in the current row:

   * Calculate:

     ```text
     d1 = row - col
     d2 = row + col
     ```
   * If `col`, `d1`, or `d2` is already occupied, skip this position.
5. Otherwise:

   * Mark the column and diagonals.
   * Recursively process `row + 1`.
6. After recursion:

   * Remove the column and diagonals from the sets.
7. When `row == n`:

   * A valid arrangement has been found.
   * Increment `count`.
8. Return `count`.

---

### Complexity

Let `N = n`.

* **Time:** `O(N!)` approximately

  * We explore possible queen placements using backtracking.
  * Each validity check is `O(1)` using sets.
* **Auxiliary Space:** `O(N)`

  * Recursion depth: `O(N)`.
  * Column set: `O(N)`.
  * Two diagonal sets: `O(N)`.
* **Output Space:** `O(1)`

  * Only the count is returned, unlike LeetCode 51 where all boards must be stored.

---

### Notes / Tips

* **N-Queens II is basically N-Queens I without storing the boards.**
* No actual `board` is required because we only care whether a position is valid.
* The three constraints are enough:

```text
col
row - col
row + col
```

* The backtracking template remains:

```text
Choose
↓
Explore
↓
Undo
```

* Base case:

```cpp
if (row == n) {
    count++;
    return;
}
```

* Important distinction:

```text
LeetCode 51:
valid arrangement → save board

LeetCode 52:
valid arrangement → count++
```

---

### Code

```cpp
class Solution {
public:
    int count = 0;
    int n;

    unordered_set<int> cols;
    unordered_set<int> diag1;
    unordered_set<int> diag2;

    void backtrack(int row) {
        if (row == n) {
            count++;
            return;
        }

        for (int col = 0; col < n; col++) {
            int d1 = row - col;
            int d2 = row + col;

            if (cols.count(col) || diag1.count(d1) || diag2.count(d2)) {
                continue;
            }

            cols.insert(col);
            diag1.insert(d1);
            diag2.insert(d2);

            backtrack(row + 1);

            cols.erase(col);
            diag1.erase(d1);
            diag2.erase(d2);
        }
    }

    int totalNQueens(int n) {
        this->n = n;

        backtrack(0);

        return count;
    }
};
```

---

## Quick Revision

```text
N-Queens II
    ↓
Place one queen per row
    ↓
Try every column
    ↓
Check:
    col
    row - col
    row + col
    ↓
Valid?
    ↓
Mark
    ↓
Recurse
    ↓
Unmark
    ↓
row == n?
    ↓
count++
```

### Core Difference from LeetCode 51

```text
51 N-Queens
→ Generate and store all valid boards

52 N-Queens II
→ Generate implicitly and only count them
```

### Core Formula

```text
Column conflict        → col
Main diagonal conflict → row - col
Anti-diagonal conflict → row + col
```
