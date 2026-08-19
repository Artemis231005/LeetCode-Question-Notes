# LeetCode 36 — Valid Sudoku

## Metadata

* **LeetCode:** 36
* **Problem:** Valid Sudoku
* **Difficulty:** Medium
* **Topics:** Array, Hash Table, Matrix
* **Pattern:** Hash Set, Constraint Checking
* **Key Pattern:** Track values seen in each row, column, and 3×3 box
* **Key Technique:** Hash sets for duplicate detection
* **Optimal Complexity:** `O(1)` time and `O(1)` space
* **Key Template:** Row/Column/Box Hash Set Validation

---

## Problem

Determine if a partially filled `9 × 9` Sudoku board is valid.

A board is valid if:

1. Each **row** contains no duplicate digits.
2. Each **column** contains no duplicate digits.
3. Each **3 × 3 sub-box** contains no duplicate digits.

Only filled cells (`'1'` to `'9'`) need to be checked. Empty cells (`'.'`) can be ignored.

**Important:** The board does **not** need to be solved. We only need to check whether the current configuration is valid.

---

## Approach 1 — Three Sets per Cell

### Idea

For every filled cell, check three constraints simultaneously:

* Has this digit already appeared in its **row**?
* Has this digit already appeared in its **column**?
* Has this digit already appeared in its **3 × 3 box**?

Maintain:

* `rows[9]` → set of digits in each row
* `cols[9]` → set of digits in each column
* `boxes[9]` → set of digits in each 3 × 3 box

The box containing `(r, c)` is:

```text
box = (r / 3) * 3 + (c / 3)
```

### Dry Run

Consider:

```text
r = 4, c = 7
```

Then:

```text
r / 3 = 1
c / 3 = 2

box = 1 * 3 + 2
    = 5
```

So `(4,7)` belongs to box `5`.

Suppose:

```text
board[4][7] = '8'
```

Check:

```text
rows[4]   contains '8'?
cols[7]   contains '8'?
boxes[5]  contains '8'?
```

* If any is true → duplicate → invalid.
* Otherwise insert `'8'` into all three sets.

### Algorithm

1. Create `9` sets for rows.
2. Create `9` sets for columns.
3. Create `9` sets for 3 × 3 boxes.
4. Traverse every cell `(r, c)`.
5. If the cell contains `'.'`, skip it.
6. Find its box using:

   ```text
   box = (r / 3) * 3 + (c / 3)
   ```
7. If the digit already exists in the corresponding row, column, or box, return `false`.
8. Otherwise insert the digit into all three sets.
9. After checking every cell, return `true`.

### Complexity

* **Time:** `O(9 × 9)` → `O(1)`
* **Space:** `O(9 × 9)` → `O(1)`

Since the Sudoku board is always fixed at `9 × 9`, both are technically constant.

For a generalized `n × n` Sudoku:

* **Time:** `O(n²)`
* **Space:** `O(n²)`

### Notes / Tips

* The key formula is:

  ```cpp
  int box = (r / 3) * 3 + (c / 3);
  ```
* Integer division is what maps rows/columns to their 3 × 3 regions.
* Check before inserting.
* `unordered_set` gives average `O(1)` lookup.
* `set` gives `O(log n)` lookup, but either works here.
* Do not try to solve the Sudoku; this problem only asks for validation.

### Code

```cpp
class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
        vector<unordered_set<char>> rows(9);
        vector<unordered_set<char>> cols(9);
        vector<unordered_set<char>> boxes(9);

        for (int r = 0; r < 9; r++) {
            for (int c = 0; c < 9; c++) {
                if (board[r][c] == '.') {
                    continue;
                }

                char digit = board[r][c];
                int box = (r / 3) * 3 + (c / 3);

                if (rows[r].count(digit) ||
                    cols[c].count(digit) ||
                    boxes[box].count(digit)) {
                    return false;
                }

                rows[r].insert(digit);
                cols[c].insert(digit);
                boxes[box].insert(digit);
            }
        }

        return true;
    }
};
```

---

## Approach 2 — Single Hash Set with Encoded Keys

### Idea

Instead of maintaining three separate arrays of sets, store every constraint in one hash set.

For a digit `d` at `(r, c)`, create three unique keys:

```text
row-r-d
col-c-d
box-boxIndex-d
```

If any key already exists, the board is invalid.

For example, for digit `'5'` in row `2`:

```text
row2-5
```

For column `6`:

```text
col6-5
```

For box `4`:

```text
box4-5
```

This is the same logical approach, just encoded into one set.

### Dry Run

Suppose:

```text
r = 3
c = 5
digit = '7'
```

The box is:

```text
box = (3 / 3) * 3 + (5 / 3)
    = 1 * 3 + 1
    = 4
```

We check:

```text
row3-7
col5-7
box4-7
```

If any already exists → duplicate.

Otherwise insert all three.

### Algorithm

1. Create one empty hash set.
2. Traverse every cell.
3. Skip `'.'`.
4. Calculate the cell's 3 × 3 box.
5. Generate identifiers for:

   * row + digit
   * column + digit
   * box + digit
6. If any identifier already exists, return `false`.
7. Insert all three identifiers.
8. Return `true` after processing the entire board.

### Complexity

* **Time:** `O(9²)` → `O(1)`
* **Space:** `O(9²)` → `O(1)`

### Notes / Tips

* This approach is useful when learning the general idea of **encoding multiple constraints into one hash set**.
* The three-set approach is usually easier to read and understand.
* For interviews, either approach is acceptable.
* The important part is recognizing that every digit has **three independent constraints**.

### Code

```cpp
class Solution {
public:
    bool isValidSudoku(vector<vector<char>>& board) {
        unordered_set<string> seen;

        for (int r = 0; r < 9; r++) {
            for (int c = 0; c < 9; c++) {
                if (board[r][c] == '.') {
                    continue;
                }

                char digit = board[r][c];
                int box = (r / 3) * 3 + (c / 3);

                string rowKey = "row" + to_string(r) + digit;
                string colKey = "col" + to_string(c) + digit;
                string boxKey = "box" + to_string(box) + digit;

                if (seen.count(rowKey) ||
                    seen.count(colKey) ||
                    seen.count(boxKey)) {
                    return false;
                }

                seen.insert(rowKey);
                seen.insert(colKey);
                seen.insert(boxKey);
            }
        }

        return true;
    }
};
```

---

## Key Takeaways

```text
Valid Sudoku
     |
     +-- Check Row
     |
     +-- Check Column
     |
     +-- Check 3 × 3 Box
```

The main pattern is:

```text
For every element:
    Identify all constraints it belongs to
    Check whether it already exists
    If duplicate → invalid
    Otherwise mark it as seen
```

The most important formula to remember:

```cpp
int box = (r / 3) * 3 + (c / 3);
```

This pattern generalizes to many **constraint-validation problems** where one element must satisfy multiple independent uniqueness rules.
