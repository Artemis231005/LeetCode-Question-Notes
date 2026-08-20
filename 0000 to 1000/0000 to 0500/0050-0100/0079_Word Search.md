# LeetCode 79 — Word Search

## Metadata

* **LeetCode:** 79
* **Problem:** Word Search
* **Difficulty:** Medium
* **Topics:** Array, Backtracking, Depth-First Search, Matrix
* **Pattern:** Grid DFS + Backtracking
* **Key Technique:** Explore 4 directions and mark cells as visited
* **Key Pattern:** Backtracking on a Grid
* **Key Template:** DFS + Choose → Explore → Undo
* **Optimal Complexity:** `O(m × n × 4^L)` time, `O(L)` auxiliary space

---

## Problem

Given an `m × n` grid of characters and a string `word`, return `true` if the word exists in the grid.

The word can be constructed from letters of **sequentially adjacent cells**, where adjacent cells are:

```text
Up
Down
Left
Right
```

The **same cell cannot be used more than once** in a single word path.

Example:

```text
board =
[
  ['A','B','C','E'],
  ['S','F','C','S'],
  ['A','D','E','E']
]

word = "ABCCED"

Output: true
```

Path:

```text
A → B → C
        ↓
        C → E → D
```

---

## Approach — DFS + Backtracking

### Idea

Treat every cell as a possible starting point.

For each cell:

1. Check whether it matches the current character of `word`.
2. If it matches, recursively search in all **4 directions**.
3. Temporarily mark the current cell as visited.
4. After exploring, restore the cell so it can be used in another path.

The recursion represents:

```text
dfs(row, col, index)
```

where `index` tells us which character of `word` we are currently trying to match.

### Base Case

If:

```cpp
index == word.size()
```

then every character has been matched.

Return:

```cpp
true
```

### Invalid Cases

Return `false` if:

* Row is outside the board.
* Column is outside the board.
* Current cell does not match `word[index]`.
* Current cell has already been visited.

### Dry Run

Consider:

```text
board =
A B C
D E F
G H I

word = "ABE"
```

Start at `A`:

```text
A matches word[0]
```

Mark `A` as visited.

Search neighbors:

```text
B → matches word[1]
```

Mark `B`.

Search neighbors of `B`:

```text
E → matches word[2]
```

All characters matched:

```text
A → B
    ↓
    E
```

Return `true`.

### Backtracking Example

Suppose we try a path that eventually fails:

```text
A → B → X
```

If `X` does not match the required character, we undo the previous choice:

```text
A → B
```

and try another direction.

This is why we restore the cell after the recursive call.

### Algorithm

1. Let `m` be the number of rows and `n` the number of columns.
2. Traverse every cell in the board.
3. For every cell, call:

   ```cpp
   dfs(i, j, 0)
   ```
4. In `dfs`:

   * If `index == word.size()`, return `true`.
   * If the cell is out of bounds, return `false`.
   * If the current cell does not match `word[index]`, return `false`.
5. Store the current character.
6. Mark the cell as visited.
7. Recursively search:

   * Up
   * Down
   * Left
   * Right
8. Restore the original character.
9. Return whether any direction successfully finds the remaining word.

### Complexity

Let:

* `m` = number of rows
* `n` = number of columns
* `L` = length of the word

There are `m × n` possible starting cells.

From each cell, we can explore up to 4 directions.

* **Time:** `O(m × n × 4^L)`
* **Auxiliary Space:** `O(L)` recursion stack

The board is modified temporarily, but no separate visited matrix is required.

### Notes / Tips

* This is a classic **backtracking + DFS** problem.
* Only 4 directions are allowed:

  ```cpp
  int dr[] = {-1, 1, 0, 0};
  int dc[] = {0, 0, -1, 1};
  ```
* Marking the cell directly avoids using an extra `visited` matrix.
* Always **restore the cell after recursion**:

  ```cpp
  board[r][c] = original;
  ```
* The restore step is the **undo** part of backtracking.
* Do not allow diagonal movement.
* A cell can be reused in different starting paths, but **not within the same path**.
* The same board can therefore be explored again after backtracking.

### Code

```cpp
class Solution {
public:
    int rows;
    int cols;

    bool dfs(vector<vector<char>>& board, string& word,
             int r, int c, int index) {

        if (index == word.size()) {
            return true;
        }

        if (r < 0 || r >= rows || c < 0 || c >= cols) {
            return false;
        }

        if (board[r][c] != word[index]) {
            return false;
        }

        char original = board[r][c];
        board[r][c] = '#';

        int dr[] = {-1, 1, 0, 0};
        int dc[] = {0, 0, -1, 1};

        for (int i = 0; i < 4; i++) {
            int nr = r + dr[i];
            int nc = c + dc[i];

            if (dfs(board, word, nr, nc, index + 1)) {
                board[r][c] = original;
                return true;
            }
        }

        board[r][c] = original;

        return false;
    }

    bool exist(vector<vector<char>>& board, string word) {
        rows = board.size();
        cols = board[0].size();

        for (int i = 0; i < rows; i++) {
            for (int j = 0; j < cols; j++) {
                if (dfs(board, word, i, j, 0)) {
                    return true;
                }
            }
        }

        return false;
    }
};
```

---

## Key Takeaway

The core pattern is:

```text
Choose current cell
       ↓
Mark visited
       ↓
Explore 4 directions
       ↓
If successful → return true
       ↓
Otherwise restore cell
       ↓
Try another path
```

### Backtracking Template

```cpp
if (invalid) {
    return false;
}

mark current choice;

for (each possible next choice) {
    if (dfs(next)) {
        return true;
    }
}

undo current choice;

return false;
```

**Pattern:**

> Word Search = Grid DFS + Backtracking + Visited State.

**Quick recognition:**
If a problem asks you to **find a sequence/path in a grid while cells cannot be reused**, think **DFS + Backtracking**.
