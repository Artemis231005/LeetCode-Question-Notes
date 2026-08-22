# 230. Kth Smallest Element in a BST

## Metadata

* **Topic:** Binary Search Tree, Tree Traversal
* **Difficulty:** Medium
* **Pattern:** Inorder Traversal
* **Key Pattern:** Inorder traversal of a BST gives elements in **sorted order**.

---

## Idea

In a Binary Search Tree:

```text
Left < Root < Right
```

Therefore, **inorder traversal**:

```text
Left → Root → Right
```

visits nodes in **ascending order**.

So the:

```text
kth visited node = kth smallest element
```

We can stop as soon as we visit the `kth` node.

---

## Dry Run

```text
        3
       / \
      1   4
       \
        2
```

Inorder traversal:

```text
1 → 2 → 3 → 4
```

For:

```text
k = 2
```

The 2nd visited node is:

```text
2
```

Therefore, the answer is `2`.

---

## Algorithm

1. Perform inorder traversal of the BST.
2. Traverse the left subtree.
3. Process the current node:

   * Decrement `k`.
   * If `k == 0`, this is the answer.
4. Traverse the right subtree.
5. Stop once the answer is found.

---

## Complexity

* **Time:** `O(h + k)` in the early-stop approach, where `h` is tree height.
* **Space:** `O(h)` for the recursion stack.

---

## Notes / Tips

### Most Important BST Property

```text
Inorder BST → Sorted Order
```

So whenever the question asks for:

```text
kth smallest in BST
```

think:

```text
Inorder + count
```

For **kth largest**, use **reverse inorder**:

```text
Right → Root → Left
```

### Key Template

```text
inorder(root):
    inorder(left)

    k--
    if k == 0:
        answer = root->val

    inorder(right)
```

---

## Code

```cpp
class Solution {
public:
    int kthSmallest(TreeNode* root, int k) {
        int ans = 0;

        function<void(TreeNode*)> inorder = [&](TreeNode* node) {
            if (node == NULL || k == 0) {
                return;
            }

            inorder(node->left);

            if (k > 0) {
                k--;

                if (k == 0) {
                    ans = node->val;
                    return;
                }
            }

            inorder(node->right);
        };

        inorder(root);

        return ans;
    }
};
```
