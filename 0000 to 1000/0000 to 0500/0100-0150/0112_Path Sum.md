# LeetCode 112 — Path Sum

## Metadata

* **LeetCode:** 112
* **Problem:** Path Sum
* **Difficulty:** Easy
* **Topics:** Tree, Binary Tree, Depth-First Search
* **Pattern:** Root-to-Leaf Path, DFS
* **Key Technique:** Subtract node value from target while traversing
* **Key Pattern:** Root-to-leaf DFS
* **Key Template:** Recursive DFS with remaining target
* **Optimal Complexity:** `O(n)`

---

## Problem

Given the root of a binary tree and an integer `targetSum`, determine whether the tree has a **root-to-leaf path** such that the sum of all node values on the path equals `targetSum`.

A **leaf** is a node with no left and right children.

---

## Idea

Use **DFS** to explore every root-to-leaf path.

At each node:

* Subtract the current node's value from `targetSum`.
* When we reach a leaf, check whether the remaining target is equal to the leaf's value.

A simpler way is to pass the **remaining sum**:

```text
remaining = targetSum - current node value
```

If we reach a leaf and:

```text
remaining == 0
```

then a valid path exists.

### Important

We must check the condition **only at a leaf**.

A path ending at an internal node does not count.

---

## Dry Run

Consider:

```text
          5
         / \
        4   8
       /   / \
      11  13  4
     /  \      \
    7    2      1
```

Target:

```text
22
```

Start:

```text
5 → remaining = 22 - 5 = 17
```

Go left:

```text
4 → remaining = 17 - 4 = 13
```

Go left:

```text
11 → remaining = 13 - 11 = 2
```

Go right:

```text
2 → remaining = 2 - 2 = 0
```

`2` is a leaf and remaining sum is `0`.

Therefore:

```text
5 → 4 → 11 → 2
```

has sum:

```text
5 + 4 + 11 + 2 = 22
```

Answer:

```text
true
```

---

## Algorithm

1. If `root == NULL`, return `false`.
2. Subtract `root->val` from `targetSum`.
3. Check whether the current node is a leaf.
4. If it is a leaf, return whether the remaining `targetSum` is `0`.
5. Recursively check the left subtree.
6. Recursively check the right subtree.
7. Return `true` if either subtree contains a valid path.

---

## Complexity

* **Time:** `O(n)`

  * In the worst case, every node is visited.
* **Space:** `O(h)`

  * `h` is the height of the tree due to the recursion stack.
  * Worst case: `O(n)`
  * Balanced tree: `O(log n)`

---

## Notes / Tips

* The path must start at the **root**.
* The path must end at a **leaf**.
* The path cannot stop at an internal node.
* Negative node values are allowed, so do not use assumptions based on the sum increasing.
* Passing the **remaining target** makes the recursive logic simple.
* `left || right` is enough because only one valid path is required.

### Common Mistake

Do **not** write:

```cpp
if (targetSum == 0) {
    return true;
}
```

before checking whether the node is a leaf.

The target can become `0` at an internal node, but that does not mean a valid root-to-leaf path has been found.

---

## Code

```cpp
class Solution {
public:
    bool hasPathSum(TreeNode* root, int targetSum) {
        if (root == nullptr) {
            return false;
        }

        targetSum -= root->val;

        // A valid path must end at a leaf
        if (root->left == nullptr && root->right == nullptr) {
            return targetSum == 0;
        }

        return hasPathSum(root->left, targetSum) ||
               hasPathSum(root->right, targetSum);
    }
};
```
