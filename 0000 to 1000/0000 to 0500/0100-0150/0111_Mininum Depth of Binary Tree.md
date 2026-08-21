# LeetCode 111 — Minimum Depth of Binary Tree

## Metadata

* **LeetCode:** 111
* **Problem:** Minimum Depth of Binary Tree
* **Difficulty:** Easy
* **Topics:** Tree, Binary Tree, Depth-First Search, Breadth-First Search
* **Pattern:** Minimum Depth, Level Order Traversal
* **Key Technique:** Find the first leaf using BFS
* **Key Pattern:** BFS for shortest path in a tree
* **Key Template:** Level-order BFS
* **Optimal Complexity:** `O(n)`

---

## Problem

Given the root of a binary tree, return the **minimum depth**.

The minimum depth is the number of nodes along the shortest path from the root node to the **nearest leaf node**.

A **leaf node** is a node with no left and right children.

---

## Idea

The important point is that **minimum depth means the first leaf we encounter**.

### BFS Approach

Use **Breadth-First Search (BFS)** because BFS explores the tree level by level.

* Start from the root.
* Process all nodes at depth `1`.
* Then all nodes at depth `2`.
* Continue until we find a **leaf**.
* The first leaf encountered gives the minimum depth.

### Why not simply use `min(leftDepth, rightDepth) + 1`?

That formula is dangerous when one child is `NULL`.

For example:

```text
    1
   /
  2
```

The left subtree has depth `1`, while the right subtree has depth `0`.

Using:

```text
min(1, 0) + 1 = 1
```

would be wrong because node `1` is not a leaf.

The correct logic must account for missing children.

BFS naturally avoids this issue because it only returns when it reaches an actual leaf.

---

## Dry Run

Consider:

```text
        3
       / \
      9   20
         /  \
        15   7
```

### Level 1

Queue:

```text
[3]
```

`3` is not a leaf.

Depth = `1`

Add `9` and `20`.

```text
Queue = [9, 20]
```

### Level 2

Process `9`.

`9` has no children → **leaf found**.

Therefore:

```text
Minimum Depth = 2
```

We do not need to explore the remaining nodes because BFS guarantees that this is the closest leaf.

---

## Algorithm

1. If `root == NULL`, return `0`.
2. Create a queue and insert `root`.
3. Set `depth = 1`.
4. While the queue is not empty:

   * Process all nodes at the current level.
   * For each node:

     * If it has no left and no right child, return `depth`.
     * If it has a left child, push it into the queue.
     * If it has a right child, push it into the queue.
   * Increment `depth`.
5. Return `depth`.

---

## Complexity

* **Time:** `O(n)`

  * In the worst case, every node is visited.
* **Space:** `O(n)`

  * The queue can contain up to `O(n)` nodes.

---

## Notes / Tips

* **Minimum depth = shortest path from root to a leaf.**
* BFS is a natural choice for minimum depth because it explores nodes level by level.
* **Return only when `node->left == NULL && node->right == NULL`.**
* Do not treat a node with only one missing child as a leaf.
* DFS can also solve this problem, but BFS can stop as soon as the first leaf is found.
* Edge case:

  ```text
  root = NULL → answer = 0
  ```
* Single-node tree:

  ```text
      1

  answer = 1
  ```

### Common Mistake

Incorrect:

```cpp
return min(leftDepth, rightDepth) + 1;
```

when one child is `NULL`.

Correct DFS logic would need to handle missing children separately:

```cpp
if (root->left == nullptr) {
    return 1 + minDepth(root->right);
}

if (root->right == nullptr) {
    return 1 + minDepth(root->left);
}

return 1 + min(minDepth(root->left), minDepth(root->right));
```

---

## Code

```cpp
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }

        queue<TreeNode*> q;
        q.push(root);

        int depth = 1;

        while (!q.empty()) {
            int size = q.size();

            for (int i = 0; i < size; i++) {
                TreeNode* node = q.front();
                q.pop();

                // First leaf encountered gives minimum depth
                if (node->left == nullptr && node->right == nullptr) {
                    return depth;
                }

                if (node->left != nullptr) {
                    q.push(node->left);
                }

                if (node->right != nullptr) {
                    q.push(node->right);
                }
            }

            depth++;
        }

        return depth;
    }
};
```
