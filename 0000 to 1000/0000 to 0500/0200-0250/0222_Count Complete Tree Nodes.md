# 222. Count Complete Tree Nodes

## Metadata

* **Topic:** Binary Tree
* **Difficulty:** Medium
* **Pattern:** Complete Binary Tree + Binary Search
* **Key Pattern:** Compare leftmost and rightmost heights.

---

## Idea

A **complete binary tree** has all levels completely filled except possibly the last, and the last level is filled from **left to right**.

For a normal binary tree, counting nodes takes `O(n)`.

But because this is a complete tree, we can do better.

For every subtree:

* Find the height by going completely left.
* Find the height by going completely right.

If:

```text
leftHeight == rightHeight
```

the subtree is a **perfect binary tree**.

A perfect binary tree of height `h` contains:

```text
2^h - 1
```

nodes.

Otherwise, recursively count the left and right subtrees.

---

## Dry Run

```text
        1
       / \
      2   3
     / \  /
    4  5 6
```

For the root:

```text
left height  = 3
right height = 2
```

They are different, so the tree is not perfect.

Now recursively process the subtrees.

For node `2`:

```text
left height  = 2
right height = 2
```

So it is perfect.

Number of nodes:

```text
2^2 - 1 = 3
```

Similarly, node `3` contains `2` nodes.

Total:

```text
1 + 3 + 2 = 6
```

---

## Algorithm

1. If `root == NULL`, return `0`.
2. Find the leftmost height of the subtree.
3. Find the rightmost height of the subtree.
4. If both heights are equal:

   * The subtree is perfect.
   * Return `2^height - 1`.
5. Otherwise:

   * Recursively count the left subtree.
   * Recursively count the right subtree.
   * Add `1` for the current node.

---

## Complexity

* **Time:** `O(log² n)`
* **Space:** `O(log n)` due to recursion.

---

## Notes / Tips

### Perfect Tree Formula

For height `h`:

```text
nodes = 2^h - 1
```

### Important Observation

```text
leftHeight == rightHeight
        ↓
Perfect subtree
        ↓
2^h - 1
```

The height of a complete binary tree is `O(log n)`, and we calculate heights at each recursive level, giving `O(log² n)`.

### Key Template

```text
leftHeight = getLeftHeight(root)
rightHeight = getRightHeight(root)

if leftHeight == rightHeight:
    return (2^leftHeight) - 1

return 1 + count(left) + count(right)
```

---

## Code

```cpp
class Solution {
public:
    int getLeftHeight(TreeNode* root) {
        int height = 0;

        while (root != NULL) {
            height++;
            root = root->left;
        }

        return height;
    }

    int getRightHeight(TreeNode* root) {
        int height = 0;

        while (root != NULL) {
            height++;
            root = root->right;
        }

        return height;
    }

    int countNodes(TreeNode* root) {
        if (root == NULL) {
            return 0;
        }

        int leftHeight = getLeftHeight(root);
        int rightHeight = getRightHeight(root);

        if (leftHeight == rightHeight) {
            return (1 << leftHeight) - 1;
        }

        return 1 + countNodes(root->left) + countNodes(root->right);
    }
};
```
