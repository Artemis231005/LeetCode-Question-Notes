# LeetCode 37 — Sudoku Solver

## Metadata

* **LeetCode:** 37
* **Problem:** Sudoku Solver
* **Difficulty:** Hard
* **Topics:** Array, Backtracking, Matrix
* **Pattern:** Backtracking, Constraint Satisfaction
* **Key Pattern:** Try → Validate → Recurse → Undo
* **Key Technique:** Backtracking with row, column, and 3 × 3 box constraints
* **Optimal Complexity:** `O(9^(N))` worst case, where `N` is the number of empty cells
* **Key Template:** Backtracking / Constraint Satisfaction

---

## Problem

Write a program to solve a Sudoku puzzle by filling the empty cells.

A Sudoku solution must satisfy:

1. Every row contains digits `1-9` without repetition.
2. Every column contains digits `1-9` without repetition.
3. Every `3 × 3` sub-box contains digits `1-9` without repetition.

Empty cells are represented by `'.'`.

You must modify the board **in-place**.

There is guaranteed to be exactly one solution.

---

## Approach — Backtracking

### Idea

This is a classic **backtracking** problem.

For every empty cell:

1. Try placing digits `1` to `9`.
2. Check whether the digit is valid in:

   * the current row
   * the current column
   * the current `3 × 3` box
3. If valid, place it.
4. Recursively solve the remaining board.
5. If the recursive call succeeds, the Sudoku is solved.
6. If it fails, **undo the choice** and try the next digit.

The key idea is:

```text
Choose
  ↓
Check
  ↓
Place
  ↓
Explore
  ↓
Success? ── Yes → Return true
  ↓ No
Undo
  ↓
Try next choice
```

This is exactly the standard backtracking template:

```text
for every possible choice:
    if choice is valid:
        make choice

        if solve():
            return true

        undo choice

return false
```

---

### Dry Run

Consider an empty cell:

```text
. 3 4
6 7 8
9 1 2
```

Suppose the empty cell is at:

```text
row = 0
col = 0
```

Try digits:

```text
1 → invalid because 1 exists in column
2 → invalid because 2 exists in column
3 → invalid because 3 exists in row
4 → invalid because 4 exists in row
5 → valid
```

Place `5`:

```text
5 3 4
6 7 8
9 1 2
```

Then recursively solve the next empty cell.

Suppose eventually we reach a situation where no digit can be placed:

```text
Current cell:
.
```

and all `1-9` are invalid.

This means the previous choice was wrong.

So we **backtrack**:

```text
Remove 5
↓
Try the next possible digit
```

The algorithm continues until the entire board is solved.

---

### 3 × 3 Box Calculation

For a cell `(row, col)`, its box can be identified using:

```cpp
int box = (row / 3) * 3 + (col / 3);
```

For example:

```text
row = 4
col = 7

box = (4 / 3) * 3 + (7 / 3)
    = 1 * 3 + 2
    = 5
```

So `(4, 7)` belongs to box `5`.

The box occupies:

```text
Rows:    3, 4, 5
Columns: 6, 7, 8
```

---

### Validity Check

For a candidate digit, check:

```text
1. Row
2. Column
3. 3 × 3 Box
```

For the box, calculate its starting coordinates:

```cpp
int startRow = (row / 3) * 3;
int startCol = (col / 3) * 3;
```

Then check:

```text
startRow → startRow + 2
startCol → startCol + 2
```

Example:

```text
row = 4, col = 7

startRow = (4 / 3) * 3 = 3
startCol = (7 / 3) * 3 = 6
```

So inspect:

```text
(3,6) (3,7) (3,8)
(4,6) (4,7) (4,8)
(5,6) (5,7) (5,8)
```

---

### Algorithm

1. Find an empty cell.
2. If no empty cell exists, return `true`.
3. Try every digit from `'1'` to `'9'`.
4. Check whether the digit can be placed in the cell.
5. If valid:

   * Place the digit.
   * Recursively call the solver.
6. If recursion returns `true`, propagate `true`.
7. Otherwise:

   * Reset the cell to `'.'`.
   * Try the next digit.
8. If no digit works, return `false`.

---

### Complexity

Let `N` be the number of empty cells.

* **Time:** `O(9^N)` worst case
* **Space:** `O(N)` recursion stack

For a standard `9 × 9` Sudoku, the board size is fixed, so the practical input size is bounded.

The worst-case exponential complexity comes from potentially trying multiple digits for every empty cell.

---

### Notes / Tips

* **Do not use a simple greedy approach.** A locally valid digit may lead to an impossible board later.
* The moment a choice causes a contradiction, **undo it**.
* The most important operation is:

  ```cpp
  board[row][col] = '.';
  ```

  This is the **backtracking/undo step**.
* Always return `true` immediately when the recursive call succeeds.
* Since the problem guarantees a unique solution, we only need to find one valid solution.
* You can improve performance using hash sets, boolean arrays, or bitmasks to track used digits.
* A useful mental model is:

  ```text
  Empty cell
      ↓
  Try 1 → recurse
      ↓ fail
  Try 2 → recurse
      ↓ fail
  ...
      ↓
  Try valid digit
      ↓
  Continue
      ↓
  Contradiction?
      ↓
  Undo → try another
  ```

---

### Code

```cpp
class Solution {
public:
    bool isValid(vector<vector<char>>& board, int row, int col, char digit) {
        // Check row
        for (int c = 0; c < 9; c++) {
            if (board[row][c] == digit) {
                return false;
            }
        }

        // Check column
        for (int r = 0; r < 9; r++) {
            if (board[r][col] == digit) {
                return false;
            }
        }

        // Check 3 x 3 box
        int startRow = (row / 3) * 3;
        int startCol = (col / 3) * 3;

        for (int r = startRow; r < startRow + 3; r++) {
            for (int c = startCol; c < startCol + 3; c++) {
                if (board[r][c] == digit) {
                    return false;
                }
            }
        }

        return true;
    }

    bool solve(vector<vector<char>>& board) {
        // Find an empty cell
        for (int row = 0; row < 9; row++) {
            for (int col = 0; col < 9; col++) {

                if (board[row][col] != '.') {
                    continue;
                }

                // Try every digit
                for (char digit = '1'; digit <= '9'; digit++) {

                    if (!isValid(board, row, col, digit)) {
                        continue;
                    }

                    // Choose
                    board[row][col] = digit;

                    // Explore
                    if (solve(board)) {
                        return true;
                    }

                    // Undo
                    board[row][col] = '.';
                }

                // No digit worked for this cell
                return false;
            }
        }

        // No empty cells remain
        return true;
    }

    void solveSudoku(vector<vector<char>>& board) {
        solve(board);
    }
};
```

---

## Optimized Approach — Tracking Constraints

### Idea

The previous solution repeatedly scans the row, column, and box every time we try a digit.

We can avoid these repeated scans by maintaining three boolean structures:

```cpp
rows[9][10]
cols[9][10]
boxes[9][10]
```

Where:

```text
rows[r][d]   = digit d already used in row r
cols[c][d]   = digit d already used in column c
boxes[b][d]  = digit d already used in box b
```

Then checking whether a digit is valid becomes `O(1)`.

This gives us the same backtracking strategy, but with faster constraint checking.

---

### Dry Run

Suppose:

```text
row = 4
col = 7
digit = 5
```

Calculate:

```cpp
int box = (4 / 3) * 3 + (7 / 3);
```

which gives:

```text
box = 5
```

Check:

```cpp
rows[4][5]
cols[7][5]
boxes[5][5]
```

If all are `false`, digit `5` is valid.

Mark:

```cpp
rows[4][5] = true;
cols[7][5] = true;
boxes[5][5] = true;
```

Place the digit and recurse.

If the choice eventually fails, undo:

```cpp
rows[4][5] = false;
cols[7][5] = false;
boxes[5][5] = false;

board[4][7] = '.';
```

---

### Algorithm

1. Create boolean arrays for rows, columns, and boxes.
2. Scan the initial board and mark all existing digits.
3. Find an empty cell.
4. Calculate its box index.
5. Try digits `1-9`.
6. Check the three boolean arrays.
7. If valid:

   * Mark the digit as used.
   * Place it.
   * Recursively solve.
8. If recursion succeeds, return `true`.
9. Otherwise:

   * Remove the digit from all three tracking arrays.
   * Reset the cell to `'.'`.
10. If no digit works, return `false`.
11. If there are no empty cells, the puzzle is solved.

---

### Complexity

Let `N` be the number of empty cells.

* **Time:** `O(9^N)`
* **Constraint checking:** `O(1)` per candidate
* **Space:** `O(N + 9²)` → `O(N)`

The asymptotic worst case remains exponential because backtracking is still required.

---

### Notes / Tips

This is generally a better implementation because:

```text
Normal Backtracking:
Check row    → O(9)
Check column → O(9)
Check box    → O(9)

Optimized:
Check row    → O(1)
Check column → O(1)
Check box    → O(1)
```

The **backtracking pattern does not change**. Only the constraint lookup becomes faster.

The three things that must always stay synchronized are:

```cpp
rows[row][digit]
cols[col][digit]
boxes[box][digit]
```

When placing:

```text
mark → place → recurse
```

When undoing:

```text
unmark → remove
```

---

### Code

```cpp
class Solution {
public:
    bool rows[9][10] = {};
    bool cols[9][10] = {};
    bool boxes[9][10] = {};

    bool solve(vector<vector<char>>& board) {
        for (int row = 0; row < 9; row++) {
            for (int col = 0; col < 9; col++) {

                if (board[row][col] != '.') {
                    continue;
                }

                int box = (row / 3) * 3 + (col / 3);

                for (int digit = 1; digit <= 9; digit++) {

                    if (rows[row][digit] ||
                        cols[col][digit] ||
                        boxes[box][digit]) {
                        continue;
                    }

                    // Choose
                    board[row][col] = '0' + digit;
                    rows[row][digit] = true;
                    cols[col][digit] = true;
                    boxes[box][digit] = true;

                    // Explore
                    if (solve(board)) {
                        return true;
                    }

                    // Undo
                    board[row][col] = '.';
                    rows[row][digit] = false;
                    cols[col][digit] = false;
                    boxes[box][digit] = false;
                }

                return false;
            }
        }

        return true;
    }

    void solveSudoku(vector<vector<char>>& board) {
        // Initialize constraints from the given board
        for (int row = 0; row < 9; row++) {
            for (int col = 0; col < 9; col++) {
                if (board[row][col] == '.') {
                    continue;
                }

                int digit = board[row][col] - '0';
                int box = (row / 3) * 3 + (col / 3);

                rows[row][digit] = true;
                cols[col][digit] = true;
                boxes[box][digit] = true;
            }
        }

        solve(board);
    }
};
```

---

## Key Takeaways

### Core Backtracking Template

```cpp
bool solve() {
    find an empty position;

    if (no empty position) {
        return true;
    }

    for (each possible choice) {
        if (choice is valid) {

            make choice;

            if (solve()) {
                return true;
            }

            undo choice;
        }
    }

    return false;
}
```

### LeetCode 36 vs LeetCode 37

```text
36 — Valid Sudoku
        ↓
Check whether current board is valid
        ↓
No recursion
No modifications

37 — Sudoku Solver
        ↓
Find a valid complete board
        ↓
Backtracking
Modify + Undo
```

The major pattern to remember from **LeetCode 37** is:

> **Try a choice → recurse → if it fails, undo the choice.**

This is the same fundamental pattern used in problems such as **N-Queens, Combination Sum, Permutations, Word Search, and many constraint-satisfaction problems**.
