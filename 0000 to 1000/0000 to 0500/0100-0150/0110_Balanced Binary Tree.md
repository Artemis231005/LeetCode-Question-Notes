# 110. Balanced Binary Tree

## Metadata

* **Topic:** Binary Tree, Recursion, Tree Height
* **Difficulty:** Easy
* **Pattern:** Bottom-Up DFS
* **Key Pattern:** Return height if balanced, otherwise return `-1`
* **Key Template:** Postorder recursion with sentinel value

---

## Idea

A binary tree is **height-balanced** if, for every node:

```text
|height(left) - height(right)| <= 1
```

We need to check this condition for **every node**.

A naive approach would calculate the height of the left and right subtree separately for every node, which can take **O(n²)** in the worst case.

Instead, use **bottom-up recursion**:

* Recursively calculate the height of the left subtree.
* Recursively calculate the height of the right subtree.
* If either subtree is already unbalanced, return `-1`.
* If the current node is unbalanced, return `-1`.
* Otherwise, return the height of the current subtree.

So each node is processed only once.

---

## Dry Run

### Example

```text
        3
       / \
      9  20
         / \
        15  7
```

Start from the leaves.

```text
9  → height = 1
15 → height = 1
7  → height = 1
```

At node `20`:

```text
left height  = 1
right height = 1

|1 - 1| = 0 <= 1

height = 1 + max(1, 1) = 2
```

At node `3`:

```text
left height  = 1
right height = 2

|1 - 2| = 1 <= 1

height = 1 + max(1, 2) = 3
```

Therefore:

```text
Balanced → true
```

### Unbalanced Example

```text
        1
       /
      2
     /
    3
```

At node `2`:

```text
left height  = 1
right height = 0

|1 - 0| = 1
```

So node `2` is balanced.

At node `1`:

```text
left height  = 2
right height = 0

|2 - 0| = 2 > 1
```

Therefore the tree is **not balanced**.

---

## Algorithm

1. Create a recursive function that returns the height of a subtree.
2. If the node is `nullptr`, return `0`.
3. Recursively calculate the height of the left subtree.
4. If the left subtree returns `-1`, immediately return `-1`.
5. Recursively calculate the height of the right subtree.
6. If the right subtree returns `-1`, immediately return `-1`.
7. Check:

   ```text
   |leftHeight - rightHeight| > 1
   ```
8. If true, return `-1`.
9. Otherwise, return:

   ```text
   1 + max(leftHeight, rightHeight)
   ```
10. The tree is balanced if the final result is not `-1`.

---

## Complexity

* **Time:** `O(n)` — every node is visited at most once.
* **Space:** `O(h)` — recursion stack, where `h` is the height of the tree.
* **Worst-case space:** `O(n)` for a completely skewed tree.
* **Balanced tree space:** `O(log n)`.

---

## Notes / Tips

* The key idea is to **combine height calculation and balance checking** into one DFS.
* `-1` acts as a **sentinel value** meaning "this subtree is unbalanced."
* This avoids repeatedly calculating subtree heights.
* The traversal is effectively **postorder**:

  ```text
  Left → Right → Root
  ```
* Remember:

  ```text
  nullptr → height 0
  leaf → height 1
  ```
* The important condition is:

  ```cpp
  abs(leftHeight - rightHeight) <= 1
  ```
* If a subtree is already unbalanced, there is no need to process further for that path.

---

## Code

```cpp
class Solution {
public:
    int height(TreeNode* root) {
        if (root == nullptr) {
            return 0;
        }

        int leftHeight = height(root->left);

        if (leftHeight == -1) {
            return -1;
        }

        int rightHeight = height(root->right);

        if (rightHeight == -1) {
            return -1;
        }

        if (abs(leftHeight - rightHeight) > 1) {
            return -1;
        }

        return 1 + max(leftHeight, rightHeight);
    }

    bool isBalanced(TreeNode* root) {
        return height(root) != -1;
    }
};
```
