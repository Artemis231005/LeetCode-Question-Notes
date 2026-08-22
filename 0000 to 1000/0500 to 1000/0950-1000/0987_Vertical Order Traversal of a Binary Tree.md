# Vertical Order Traversal of a Binary Tree

## Problem

Given the root of a binary tree, return its **vertical order traversal**.

For every node, assign coordinates:

* Root → `(row = 0, col = 0)`
* Left child → `(row + 1, col - 1)`
* Right child → `(row + 1, col + 1)`

The traversal must be ordered by:

1. **Column** from left to right.
2. **Row** from top to bottom.
3. If two nodes have the same row and column, sort their values in **ascending order**.

Example:

```text
        3
       / \
      9   20
         /  \
        15   7

Output:
[[9], [3,15], [20], [7]]
```

---

## Approach 1: DFS + Coordinates + Sorting

### Idea

Assign every node a `(row, column)` coordinate during DFS.

Store:

```text
(column, row, value)
```

for every node.

Then sort all nodes using:

1. Column ascending.
2. Row ascending.
3. Value ascending.

Finally, group nodes having the same column.

This directly follows the ordering rules of the problem.

### Dry Run

For:

```text
        3
       / \
      9   20
         /  \
        15   7
```

Coordinates:

```text
3  → (0,  0)
9  → (1, -1)
20 → (1,  1)
15 → (2,  0)
7  → (2,  2)
```

Sort by column, then row, then value:

```text
(-1, 1, 9)
( 0, 0, 3)
( 0, 2, 15)
( 1, 1, 20)
( 2, 2, 7)
```

Group by column:

```text
[9]
[3,15]
[20]
[7]
```

### Algorithm

1. Perform DFS from the root.
2. Pass the current `row` and `column` to each node.
3. Store `(column, row, value)` for every node.
4. For the left child, use `column - 1`.
5. For the right child, use `column + 1`.
6. Sort all stored nodes by:

   * column
   * row
   * value
7. Traverse the sorted nodes and group values having the same column.
8. Return the result.

### Complexity

* Time: `O(n log n)` due to sorting.
* Space: `O(n)` for storing nodes and the result.

### Code

```cpp
class Solution {
public:
    vector<tuple<int, int, int>> nodes;

    void dfs(TreeNode* root, int row, int col) {
        if (root == nullptr) {
            return;
        }

        nodes.push_back({col, row, root->val});

        dfs(root->left, row + 1, col - 1);
        dfs(root->right, row + 1, col + 1);
    }

    vector<vector<int>> verticalTraversal(TreeNode* root) {
        dfs(root, 0, 0);

        sort(nodes.begin(), nodes.end());

        vector<vector<int>> result;

        int previousColumn = INT_MIN;

        for (auto [col, row, value] : nodes) {
            if (col != previousColumn) {
                result.push_back({});
                previousColumn = col;
            }

            result.back().push_back(value);
        }

        return result;
    }
};
```

### Notes / Tips

* The key is to convert the tree problem into a **coordinate + sorting** problem.
* Coordinate rules:

  ```text
  left  → col - 1
  right → col + 1
  ```
* Sorting tuples is convenient because C++ automatically compares them lexicographically:

  ```text
  column → row → value
  ```
* The value tie-breaker is important when two nodes occupy the **same row and column**.
* Do not confuse this with ordinary vertical order traversal; **LeetCode 987 has the extra row and value ordering rules**.
* DFS and BFS can both be used if coordinates are stored correctly.

### Key Template

```text
DFS(node, row, col):
    store (col, row, node.value)

    DFS(left, row + 1, col - 1)
    DFS(right, row + 1, col + 1)

sort all nodes by:
    column
    row
    value

group values by column
```
