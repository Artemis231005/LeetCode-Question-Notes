# LeetCode 867 — Transpose Matrix

## Metadata
* **LeetCode:** 867
* **Problem:** Transpose Matrix
* **Difficulty:** Easy
* **Topics:** Array, Matrix
* **Pattern:** Matrix Traversal
* **Key Technique:** Swap Rows and Columns
* **Optimal Complexity:** `O(n × m)` Time, `O(n × m)` Space

---

## Problem Statement
Return the **transpose** of a given matrix by converting its rows into columns.

---

## Idea
For a matrix with `n` rows and `m` columns:

```text
Original: n × m
Transpose: m × n
```

The element at:
```text
matrix[i][j]
```

moves to:
```text
ans[j][i]
```

So we simply traverse every element and place it at its transposed position.

Example:
```text
1 2 3
4 5 6
```

becomes:
```text
1 4
2 5
3 6
```

---

## Dry Run
Given:
```text
1 2 3
4 5 6
```

This is a `2 × 3` matrix.

Create a `3 × 2` result:
```text
_ _
_ _
_ _
```

### Row 0
```text
matrix[0][0] = 1
→ ans[0][0] = 1

matrix[0][1] = 2
→ ans[1][0] = 2

matrix[0][2] = 3
→ ans[2][0] = 3
```

Result:
```text
1 _
2 _
3 _
```

### Row 1
```text
matrix[1][0] = 4
→ ans[0][1] = 4

matrix[1][1] = 5
→ ans[1][1] = 5

matrix[1][2] = 6
→ ans[2][1] = 6
```

Final:
```text
1 4
2 5
3 6
```

---

## Algorithm
1. Let `n = matrix.size()` and `m = matrix[0].size()`.
2. Create an `m × n` result matrix.
3. Traverse the original matrix using `i` and `j`.
4. For every element:

   ```cpp
   ans[j][i] = matrix[i][j];
   ```
5. Return `ans`.

---

## Complexity
* **TC:** `O(n × m)`

  * Every element is visited exactly once.
* **SC:** `O(n × m)`

  * The transposed matrix contains the same total number of elements.

---

## Code
```cpp
class Solution {
public:
    vector<vector<int>> transpose(vector<vector<int>>& matrix) {
        int n = matrix.size();
        int m = matrix[0].size();

        vector<vector<int>> ans(m, vector<int>(n));

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                ans[j][i] = matrix[i][j];
            }
        }

        return ans;
    }
};
```

---

## Notes / Tips
* Transpose means:

  ```text
  Row → Column
  Column → Row
  ```
* The mapping to remember is:

  ```text
  matrix[i][j] → ans[j][i]
  ```
* For an `n × m` matrix, the transpose is `m × n`.
* Unlike transposing a **square matrix in-place**, here we create a new matrix because the dimensions can be different.
* For a square matrix, an in-place transpose is possible by swapping:

  ```text
  matrix[i][j] ↔ matrix[j][i]
  ```

  only for `j > i`.

* **In-place transpose is only possible for a square matrix (`n × n`).**
For a rectangular matrix (`n × m` where `n != m`), the transpose has dimensions `m × n`, so we need a new matrix.
For a square matrix, transpose in-place by swapping:
  `matrix[i][j] ↔ matrix[j][i]`
  for `j > i`.
Be careful with loop bounds: if `i` represents rows, its bound depends on `n`; if `j` represents columns, its bound depends on `m`.

---

## Key Template
```text
Original:       Transpose:

[i][j]    →     [j][i]

n × m     →     m × n
```

```cpp
for (int i = 0; i < n; i++) {
    for (int j = 0; j < m; j++) {
        ans[j][i] = matrix[i][j];
    }
}
```
