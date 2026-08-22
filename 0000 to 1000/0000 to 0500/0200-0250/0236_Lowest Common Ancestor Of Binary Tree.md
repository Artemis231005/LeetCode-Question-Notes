# 236. Lowest Common Ancestor of a Binary Tree

## Metadata

* **Topic:** Binary Tree
* **Difficulty:** Medium
* **Pattern:** Postorder DFS
* **Key Pattern:** If `p` and `q` are found in different subtrees, the current node is their LCA.

---

## Idea

The **Lowest Common Ancestor (LCA)** of two nodes is the deepest node that has both `p` and `q` as descendants.

For every node:

* If it is `NULL`, return `NULL`.
* If it is `p` or `q`, return the node itself.
* Recursively search the left and right subtrees.
* If **both sides return a node**, the current node is the LCA.
* If only one side returns a node, pass that node upward.

Example:

```text
        3
       / \
      5   1
     / \ / \
    6  2 0  8
      / \
     7   4
```

For `p = 5`, `q = 1`:

```text
left  → 5
right → 1
```

Both are found below `3`, so:

```text
LCA = 3
```

---

## Dry Run

For:

```text
p = 5
q = 4
```

Start at `3`.

```text
        3
       /
      5
       \
        2
         \
          4
```

Search left subtree:

* Node `5` is `p`, so return `5`.
* The subtree of `5` also contains `4`.

At node `3`:

```text
left  → 5
right → NULL
```

So return `5`.

Therefore:

```text
LCA(5, 4) = 5
```

---

## Algorithm

1. If `root == NULL`, return `NULL`.
2. If `root == p` or `root == q`, return `root`.
3. Recursively find `p`/`q` in the left subtree.
4. Recursively find `p`/`q` in the right subtree.
5. If both results are non-NULL, return `root`.
6. Otherwise, return whichever side is non-NULL.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(h)` for recursion, where `h` is the tree height.

---

## Code

```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if (root == NULL || root == p || root == q) {
            return root;
        }

        TreeNode* left = lowestCommonAncestor(root->left, p, q);
        TreeNode* right = lowestCommonAncestor(root->right, p, q);

        if (left != NULL && right != NULL) {
            return root;
        }

        if (left != NULL) {
            return left;
        }

        return right;
    }
};
```

---

## Notes / Tips

* This is essentially **postorder DFS**:

  ```text
  Left → Right → Root
  ```
* `root == p || root == q` is important because one node can be an ancestor of the other.
* If both `left` and `right` are non-NULL, `p` and `q` lie in different directions, so the current node is their LCA.
* If only one side is non-NULL, that side contains both nodes or the LCA.

### Key Cases

```text
root == p/q
    → return root

left != NULL && right != NULL
    → root is LCA

only left != NULL
    → return left

only right != NULL
    → return right
```

---

## Key Template

```text
LCA(root, p, q):

    if root == NULL or root == p or root == q:
        return root

    left = LCA(root->left, p, q)
    right = LCA(root->right, p, q)

    if left != NULL and right != NULL:
        return root

    return left if left != NULL else right
```
