# LeetCode 114 — Flatten Binary Tree to Linked List

## Metadata

* **LeetCode:** 114
* **Problem:** Flatten Binary Tree to Linked List
* **Difficulty:** Medium
* **Topics:** Tree, Binary Tree, Depth-First Search, Linked List
* **Pattern:** Tree Transformation, Preorder Traversal
* **Key Technique:** Rewire pointers using preorder
* **Key Pattern:** Preorder traversal + pointer manipulation
* **Key Template:** Root → Left → Right transformation
* **Optimal Complexity:** `O(n)`

---

## Problem

Given the root of a binary tree, **flatten the tree into a linked list in-place**.

The resulting linked list must follow the same order as **preorder traversal**:

```text
Root → Left → Right
```

For every node:

* `left` must be `nullptr`
* `right` must point to the next node in preorder

Example:

```text
        1
       / \
      2   5
     / \   \
    3   4   6
```

Preorder:

```text
1 → 2 → 3 → 4 → 5 → 6
```

Flattened tree:

```text
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

---

## Idea

The main challenge is to modify the tree **in-place** without losing any nodes.

For each node:

1. If it has a left subtree, find the **rightmost node of the left subtree**.
2. Attach the current node's original right subtree to that rightmost node.
3. Move the left subtree to the right.
4. Set the left pointer to `nullptr`.
5. Continue with the new right subtree.

### Example

For:

```text
        1
       / \
      2   5
     / \   \
    3   4   6
```

At node `1`:

```text
Left subtree:
    2
   / \
  3   4
```

Rightmost node of left subtree = `4`.

Attach original right subtree (`5 → 6`) after `4`:

```text
    2
   / \
  3   4
       \
        5
         \
          6
```

Move the entire left subtree to the right of `1`:

```text
1
 \
  2
 / \
3   4
     \
      5
       \
        6
```

Set `1->left = nullptr`.

Then repeat for `2`, and so on.

---

## Dry Run

Initial tree:

```text
        1
       / \
      2   5
     / \   \
    3   4   6
```

### Step 1 — Node `1`

Left subtree exists.

Rightmost node of left subtree = `4`.

Attach original right subtree:

```text
4 → 5 → 6
```

Move left subtree to right:

```text
1
 \
  2
 / \
3   4
     \
      5
       \
        6
```

Now:

```text
1->left = nullptr
```

### Step 2 — Node `2`

Left subtree exists.

Rightmost node = `3`.

Attach original right subtree:

```text
3 → 4 → 5 → 6
```

Move left subtree to right:

```text
1
 \
  2
   \
    3
     \
      4
       \
        5
         \
          6
```

### Step 3 — Node `3`

No left subtree.

Move to `3->right`.

### Remaining nodes

`4`, `5`, and `6` have no left subtree.

Final result:

```text
1 → 2 → 3 → 4 → 5 → 6
```

---

## Algorithm

1. Start from `root`.
2. While `current != nullptr`:

   * If `current->left` exists:

     1. Store `current->right` as `rightSubtree`.
     2. Find the rightmost node of `current->left`.
     3. Attach `rightSubtree` to that node's `right`.
     4. Move `current->left` to `current->right`.
     5. Set `current->left = nullptr`.
   * Move `current = current->right`.
3. The tree is now flattened into a right-skewed linked list.

---

## Complexity

* **Time:** `O(n²)` in the worst case

  * Finding the rightmost node of each left subtree can take `O(n)` repeatedly.
* **Space:** `O(1)`

  * No recursion or extra data structure is used.

### Note

There are also `O(n)` approaches using recursive reverse preorder or a stack. The above approach is useful because it achieves **`O(1)` extra space**.

---

## Notes / Tips

* The flattened tree follows **preorder traversal**.
* Every `left` pointer must become `nullptr`.
* Every `right` pointer points to the next preorder node.
* Always save the original right subtree before overwriting `current->right`.
* The **rightmost node of the left subtree** is where the original right subtree must be connected.
* The operation must be performed **in-place**.
* A stack-based preorder solution uses `O(n)` space.
* Morris-style pointer manipulation can achieve `O(1)` extra space.

### Key Pointer Transformation

For a node:

```text
        root
       /    \
    left    right
```

Transform into:

```text
root
 \
  left
   \
    ... 
     \
      right
```

The original `right` subtree is attached to the **rightmost node of the left subtree**.

---

## Code

```cpp
class Solution {
public:
    void flatten(TreeNode* root) {
        TreeNode* current = root;

        while (current != nullptr) {
            if (current->left != nullptr) {
                // Find the rightmost node of the left subtree
                TreeNode* predecessor = current->left;

                while (predecessor->right != nullptr) {
                    predecessor = predecessor->right;
                }

                // Attach original right subtree
                predecessor->right = current->right;

                // Move left subtree to the right
                current->right = current->left;
                current->left = nullptr;
            }

            // Move to the next node in the flattened structure
            current = current->right;
        }
    }
};
```
