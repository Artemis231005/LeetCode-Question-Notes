# 104. Maximum Depth of Binary Tree

## Metadata
- **LeetCode:** 104
- **Topic:** Binary Tree, Recursion, DFS
- **Tags:** `Binary Tree` `DFS` `Recursion`
- **Difficulty:** Easy
- **Key Pattern:** Recursive Tree Traversal
- **Key Template:** `max(left subtree, right subtree) + 1`

--------------------------------

## Idea

The **maximum depth** of a binary tree is the number of nodes along the longest path from the root node to a leaf node.

For every node:
- If the node is `nullptr`, its depth is `0`.
- Recursively find the maximum depth of the left subtree.
- Recursively find the maximum depth of the right subtree.
- Add `1` for the current node.

### Formula

```text
depth(root) = max(depth(root->left), depth(root->right)) + 1
````

### Base Case

```text
root == nullptr → 0
```

---

## Dry Run

Example:

```text
        3
       / \
      9  20
         / \
        15  7
```

Starting from `3`:

```text
depth(3)
= max(depth(9), depth(20)) + 1

depth(9)
= max(0, 0) + 1
= 1

depth(20)
= max(depth(15), depth(7)) + 1
= max(1, 1) + 1
= 2

depth(3)
= max(1, 2) + 1
= 3
```

**Answer = 3**

---

## Algorithm

1. If `root == nullptr`, return `0`.
2. Recursively calculate the depth of the left subtree.
3. Recursively calculate the depth of the right subtree.
4. Take the maximum of the two depths.
5. Add `1` for the current node.
6. Return the result.

---

## Complexity

* **Time:** `O(n)` — every node is visited once.
* **Space:** `O(h)` — recursion stack, where `h` is the height of the tree.

  * Worst case: `O(n)` for a skewed tree.
  * Balanced tree: `O(log n)`.

---

## Notes / Tips

* This is a classic **divide-and-conquer recursion** problem.
* Each node only needs information from its left and right subtrees.
* The `+1` represents the **current node**.
* Remember the base case: `nullptr → 0`.
* This pattern is reusable for many binary tree problems:

```cpp
return max(left_result, right_result) + 1;
```

* An iterative **BFS/level-order traversal** can also solve this by counting the number of levels.

---

## Code

```cpp
class Solution {
public:
    int maxDepth(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }

        int leftDepth = maxDepth(root->left);
        int rightDepth = maxDepth(root->right);

        return max(leftDepth, rightDepth) + 1;
    }
};
```

```
