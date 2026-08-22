# 226. Invert Binary Tree

## Metadata

* **Topic:** Binary Tree
* **Difficulty:** Easy
* **Pattern:** Tree Recursion
* **Key Pattern:** Swap left and right children at every node.

---

## Idea

To invert a binary tree, swap the left and right child of **every node**.

Example:

```text
        4
       / \
      2   7
     / \ / \
    1  3 6  9
```

After inversion:

```text
        4
       / \
      7   2
     / \ / \
    9  6 3  1
```

We can recursively invert both subtrees after swapping them.

---

## Dry Run

```text
        4
       / \
      2   7
```

At node `4`:

```text
left  = 2
right = 7
```

Swap:

```text
left  = 7
right = 2
```

Then recursively invert the subtrees.

The same operation is performed for every node.

---

## Algorithm

1. If `root == NULL`, return `NULL`.
2. Swap `root->left` and `root->right`.
3. Recursively invert the left subtree.
4. Recursively invert the right subtree.
5. Return `root`.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(h)` for recursion, where `h` is tree height.

  * Worst case: `O(n)`
  * Balanced tree: `O(log n)`

---

## Notes / Tips

The entire problem is based on one operation:

```text
swap(left, right)
```

It can be done **before or after** recursive calls.

### Key Template

```text
invert(root):
    if root == NULL:
        return NULL

    swap(root->left, root->right)

    invert(root->left)
    invert(root->right)

    return root
```

---

## Code

```cpp
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (root == NULL) {
            return NULL;
        }

        swap(root->left, root->right);

        invertTree(root->left);
        invertTree(root->right);

        return root;
    }
};
```
