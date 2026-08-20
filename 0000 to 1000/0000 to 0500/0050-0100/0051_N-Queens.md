# LeetCode 51 — N-Queens

## Metadata

* **LeetCode:** 51
* **Problem:** N-Queens
* **Difficulty:** Hard
* **Topics:** Array, Backtracking, Matrix
* **Pattern:** Backtracking, Constraint Checking
* **Key Pattern:** Place one queen per row and backtrack when a placement becomes invalid
* **Key Technique:** Track occupied columns and diagonals using sets
* **Key Template:** Backtracking / Choose → Explore → Undo
* **Optimal Complexity:** `O(N!)` time, `O(N²)` space

---

## Problem

Given an integer `n`, return all distinct solutions to the **N-Queens** puzzle.

Place `n` queens on an `n × n` chessboard such that:

* No two queens are in the same row.
* No two queens are in the same column.
* No two queens share the same diagonal.

Return all valid board configurations.

---

## Approach — Backtracking

### Idea

We place queens **row by row**.

For each row:

1. Try placing a queen in every column.
2. Check whether that position is safe.
3. If safe, place the queen and recursively solve the next row.
4. If the recursive call fails or finishes, remove the queen and try another column.

Since we place exactly one queen in each row, **row conflicts are automatically avoided**.

We only need to check:

* Column
* Main diagonal: `row - col`
* Anti-diagonal: `row + col`

For a cell `(row, col)`:

```text
Column       → col
Main diagonal → row - col
Anti-diagonal → row + col
```

These values uniquely identify the corresponding column/diagonal.

---

### Dry Run

For `n = 4`:

Start with an empty board.

```text
Row 0:
Try col 0 → place Q

Q...
....
....
....
```

Row 1:

```text
col 0 → same column ❌
col 1 → same diagonal ❌
col 2 → safe ✅
```

```text
Q...
..Q.
....
....
```

Row 2:

```text
col 0 → same column ❌
col 1 → diagonal conflict ❌
col 2 → same column ❌
col 3 → diagonal conflict ❌
```

No valid position → **backtrack**.

Remove queen from `(1,2)`.

Try another position for row 1:

```text
col 3 → safe
```

```text
Q...
...Q
....
....
```

Continue placing queens.

Eventually, one valid solution is:

```text
.Q..
...Q
Q...
..Q.
```

which corresponds to:

```text
[".Q..",
 "...Q",
 "Q...",
 "..Q."]
```

Another valid solution is:

```text
..Q.
Q...
...Q
.Q..
```

Thus, for `n = 4`, there are **2 solutions**.

---

### Algorithm

1. Create an `n × n` board filled with `'.'`.
2. Maintain three sets:

   * `cols` → occupied columns
   * `diag1` → occupied main diagonals using `row - col`
   * `diag2` → occupied anti-diagonals using `row + col`
3. Start backtracking from `row = 0`.
4. For the current row, try every column `col`.
5. If:

   * `col` is already in `cols`, skip.
   * `row - col` is already in `diag1`, skip.
   * `row + col` is already in `diag2`, skip.
6. Otherwise:

   * Place `'Q'`.
   * Add the column and diagonal identifiers to their sets.
   * Recursively solve `row + 1`.
7. After recursion:

   * Remove the queen.
   * Remove the column and diagonal identifiers from the sets.
8. When `row == n`, the board contains a complete valid solution:

   * Convert each row into a string.
   * Add the board to the answer.
9. Return all solutions.

---

### Complexity

Let `N = n`.

* **Time:** `O(N!)` approximately

  * At most `N` choices for the first row, `N-1` for the second, etc.
  * Constraint checking is `O(1)` using sets.
  * Constructing/storing each solution adds `O(N²)` work.
* **Space:** `O(N²)`

  * Board: `O(N²)`
  * Recursion + sets: `O(N)`
  * Output itself can require `O(N²)` per solution.

---

### Notes / Tips

* **One queen per row** means we never need to check rows separately.
* `row - col` identifies one diagonal direction.
* `row + col` identifies the other diagonal direction.
* Use **sets** so checking whether a column/diagonal is occupied takes `O(1)` average time.
* Backtracking always follows:

```text
Choose
↓
Explore
↓
Undo
```

* The most important part is the diagonal trick:

```text
(row, col)

Main diagonal  → row - col
Anti diagonal  → row + col
```

* Example:

```text
(0, 1) → row - col = -1
(1, 2) → row - col = -1
(2, 3) → row - col = -1
```

All three cells lie on the same diagonal.

---

### Code

```cpp
class Solution {
public:
    vector<vector<string>> ans;
    vector<string> board;
    unordered_set<int> cols;
    unordered_set<int> diag1;
    unordered_set<int> diag2;
    int n;

    void backtrack(int row) {
        if (row == n) {
            ans.push_back(board);
            return;
        }

        for (int col = 0; col < n; col++) {
            int d1 = row - col;
            int d2 = row + col;

            if (cols.count(col) || diag1.count(d1) || diag2.count(d2)) {
                continue;
            }

            board[row][col] = 'Q';
            cols.insert(col);
            diag1.insert(d1);
            diag2.insert(d2);

            backtrack(row + 1);

            board[row][col] = '.';
            cols.erase(col);
            diag1.erase(d1);
            diag2.erase(d2);
        }
    }

    vector<vector<string>> solveNQueens(int n) {
        this->n = n;
        board = vector<string>(n, string(n, '.'));

        backtrack(0);

        return ans;
    }
};
```

---

## Quick Revision

```text
N-Queens
   ↓
Place one queen per row
   ↓
For every column:
   ↓
Check:
   column
   row - col
   row + col
   ↓
Valid?
   ↓
Place Q
   ↓
Recurse to next row
   ↓
Undo placement
   ↓
Try next column
```

**Core formula:**

```text
Column conflict       → col
Main diagonal conflict → row - col
Anti diagonal conflict → row + col
```

**Core pattern:**

```text
backtrack(row):
    if row == n:
        save solution

    for each column:
        if position is invalid:
            continue

        place queen
        backtrack(row + 1)
        remove queen
```
