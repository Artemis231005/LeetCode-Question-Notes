# LeetCode 102 — Binary Tree Level Order Traversal

## Metadata

* **LeetCode:** 102
* **Problem:** Binary Tree Level Order Traversal
* **Difficulty:** Medium
* **Topics:** Tree, Breadth-First Search, Depth-First Search, Binary Tree
* **Pattern:** Level Order Traversal, BFS
* **Key Technique:** Queue + Level Size
* **Optimal Complexity:** `O(n)` Time, `O(n)` Space

---

## Problem

Given the root of a binary tree, return its **level order traversal**.

Level order traversal means visiting nodes **level by level from left to right**.

### Example

```text
        3
       / \
      9   20
         /  \
        15   7
```

Output:

```text
[
    [3],
    [9, 20],
    [15, 7]
]
```

---

# Approach 1 — Recursive DFS

## Idea

Level order traversal is naturally a BFS problem, but it can also be solved using DFS.

Keep track of the **depth** of each node.

The depth determines which level of the result the node belongs to:

```text
depth 0 → result[0]
depth 1 → result[1]
depth 2 → result[2]
```

When visiting a node at depth `d`:

* If `result[d]` does not exist, create it.
* Add the node's value to `result[d]`.
* Recursively visit the left and right children with `depth + 1`.

## Dry Run

For:

```text
        3
       / \
      9   20
         /  \
        15   7
```

Start:

```text
node = 3
depth = 0

result = [[3]]
```

Visit `9`:

```text
depth = 1

result = [
    [3],
    [9]
]
```

Visit `20`:

```text
depth = 1

result = [
    [3],
    [9, 20]
]
```

Visit `15` and `7`:

```text
depth = 2

result = [
    [3],
    [9, 20],
    [15, 7]
]
```

## Algorithm

1. Create an empty `result`.
2. Perform DFS starting from `root` with `depth = 0`.
3. If the current node is `NULL`, return.
4. If `depth == result.size()`, create a new level.
5. Add the current node's value to `result[depth]`.
6. Recursively process the left child with `depth + 1`.
7. Recursively process the right child with `depth + 1`.
8. Return `result`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(h)` recursion stack, excluding the output
* **Worst-case Space:** `O(n)` for a skewed tree
* **Balanced Tree Space:** `O(log n)`

## Notes / Tips

* DFS normally does not process a tree level by level.
* The `depth` variable allows us to group nodes into their correct levels.
* Nodes are visited left before right, so their order within each level is preserved.
* BFS is generally more intuitive for this problem.

## Code

```cpp
class Solution {
public:
    void dfs(TreeNode* node, int depth, vector<vector<int>>& result) {
        if (node == NULL) {
            return;
        }

        if (depth == result.size()) {
            result.push_back({});
        }

        result[depth].push_back(node->val);

        dfs(node->left, depth + 1, result);
        dfs(node->right, depth + 1, result);
    }

    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;

        dfs(root, 0, result);

        return result;
    }
};
```

---

# Approach 2 — BFS Using Queue

## Idea

Level order traversal directly corresponds to **Breadth-First Search**.

Use a queue to process nodes in the order:

```text
current level → next level → next level → ...
```

The important part is determining which nodes belong to the current level.

Before processing a level:

```cpp
int levelSize = q.size();
```

At that moment, the queue contains exactly the nodes of the current level.

Process exactly `levelSize` nodes and add their children to the queue.

The newly added children will be processed in the next iteration.

## Dry Run

For:

```text
        3
       / \
      9   20
         /  \
        15   7
```

Initial:

```text
queue = [3]
```

### Level 0

```text
levelSize = 1
```

Process `3`.

Add its children:

```text
queue = [9, 20]
level = [3]
```

### Level 1

```text
levelSize = 2
```

Process `9` and `20`.

While processing `20`, add `15` and `7`.

```text
queue = [15, 7]
level = [9, 20]
```

### Level 2

```text
levelSize = 2
```

Process `15` and `7`.

```text
level = [15, 7]
```

Final result:

```text
[
    [3],
    [9, 20],
    [15, 7]
]
```

## Why Is `levelSize` Necessary?

Suppose:

```text
queue = [9, 20]
```

These two nodes belong to the **same level**.

While processing them, their children are added:

```text
queue = [15, 7]
```

If we simply processed nodes until the queue was empty, we would mix the current level with the next level.

By storing:

```cpp
int levelSize = q.size();
```

we establish a boundary:

```text
levelSize nodes → current level
newly added nodes → next level
```

## Algorithm

1. Create an empty `result`.
2. If `root == NULL`, return `result`.
3. Create a queue and push `root`.
4. While the queue is not empty:

   1. Store `levelSize = q.size()`.
   2. Create an empty `level`.
   3. Process exactly `levelSize` nodes:

      * Remove the front node.
      * Add its value to `level`.
      * Push its left child if it exists.
      * Push its right child if it exists.
   4. Add `level` to `result`.
5. Return `result`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`
* Each node is pushed into and removed from the queue exactly once.
* The output itself also requires `O(n)` space.

## Notes / Tips

* This is the standard BFS solution for level order traversal.
* Always save `q.size()` **before** processing the current level.
* The queue contains the next nodes to be processed.
* `levelSize` separates the current level from the next level.
* The left child must be pushed before the right child to preserve left-to-right order.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> levelOrder(TreeNode* root) {
        vector<vector<int>> result;

        if (root == NULL) {
            return result;
        }

        queue<TreeNode*> q;
        q.push(root);

        while (!q.empty()) {
            int levelSize = q.size();
            vector<int> level;

            for (int i = 0; i < levelSize; i++) {
                TreeNode* node = q.front();
                q.pop();

                level.push_back(node->val);

                if (node->left != NULL) {
                    q.push(node->left);
                }

                if (node->right != NULL) {
                    q.push(node->right);
                }
            }

            result.push_back(level);
        }

        return result;
    }
};
```
