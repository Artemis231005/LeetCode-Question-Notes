# Diameter of Binary Tree

## Problem

Given the root of a binary tree, return the **diameter** of the tree.

The diameter is the **length of the longest path between any two nodes**.

The path does not necessarily have to pass through the root.

The answer is measured in **number of edges**.

Example:

```text
        1
       / \
      2   3
     / \
    4   5

Diameter = 3
Path: 4 → 2 → 1 → 3
```

---

## Approach 1: DFS + Height

### Idea

For every node, calculate its height.

The longest path passing through a node is:

```text
left height + right height
```

Maintain a global `diameter` and update it for every node.

At the same time, return the height of the current subtree:

```text
height = 1 + max(left height, right height)
```

The important observation is that **height is calculated bottom-up**, so postorder DFS is used.

### Dry Run

For:

```text
        1
       / \
      2   3
     / \
    4   5
```

```text
Node 4 → height = 1
Node 5 → height = 1

Node 2:
left height = 1
right height = 1
diameter = 1 + 1 = 2
height = 2

Node 3 → height = 1

Node 1:
left height = 2
right height = 1
diameter = 2 + 1 = 3

Answer = 3
```

### Algorithm

1. Initialize `diameter = 0`.
2. Perform DFS on the tree.
3. For each node:

   * Recursively calculate the left subtree height.
   * Recursively calculate the right subtree height.
   * Update `diameter = max(diameter, leftHeight + rightHeight)`.
   * Return `1 + max(leftHeight, rightHeight)`.
4. Return `diameter`.

### Complexity

* Time: `O(n)` because every node is visited once.
* Space: `O(h)` for the recursion stack, where `h` is the tree height.

### Code

```cpp
class Solution {
public:
    int diameter = 0;

    int height(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }

        int leftHeight = height(root->left);
        int rightHeight = height(root->right);

        diameter = max(diameter, leftHeight + rightHeight);

        return 1 + max(leftHeight, rightHeight);
    }

    int diameterOfBinaryTree(TreeNode* root) {
        height(root);
        return diameter;
    }
};
```

### Notes / Tips

* **Diameter is measured in edges**, while height here is measured in nodes.
* For every node:

```text
diameter through node = leftHeight + rightHeight
```

* The diameter **does not have to pass through the root**.
* Use postorder DFS because we need the children's heights before calculating the current node's height.
* The key pattern is: **return useful information from DFS while updating the answer globally**.
* Don't return the diameter from `height()`; return the height and update the diameter separately.

### Key Template

```text
global answer = 0

DFS(node):
    if node is null:
        return 0

    left = DFS(node.left)
    right = DFS(node.right)

    answer = max(answer, left + right)

    return 1 + max(left, right)

DFS(root)

return answer
```
