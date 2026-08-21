# LeetCode 98 — Validate Binary Search Tree

## Metadata

* **LeetCode:** 98
* **Problem:** Validate Binary Search Tree
* **Difficulty:** Medium
* **Topics:** Tree, Depth-First Search, Binary Search Tree
* **Pattern:** BST Validation
* **Key Technique:** Maintain valid range `(min, max)`
* **Key Pattern:** Recursive Range Validation
* **Key Template:** DFS with Bounds
* **Optimal Complexity:** `O(n)` time, `O(h)` auxiliary space

---

## Problem

Given the root of a binary tree, determine whether it is a **valid Binary Search Tree (BST)**.

For every node:

```text
Left subtree  → values < node
Right subtree → values > node
```

The entire left and right subtrees must also satisfy the BST property.

Example:

```text
        2
       / \
      1   3
```

Output:

```text
true
```

Invalid example:

```text
        5
       / \
      1   4
         / \
        3   6
```

Output:

```text
false
```

Even though `3 < 4`, it is in the **right subtree of 5**, so it must be greater than `5`.

---

## Approach 1 — Recursive Range Validation

### Idea

The common mistake is to check only the immediate children:

```cpp
node->left->val < node->val
node->right->val > node->val
```

This is **not enough**.

Instead, every node must satisfy a valid range.

Initially:

```text id="n4n1x4"
(-∞, +∞)
```

For a node with value `x`:

```text id="h9r2fj"
Left subtree  → (-∞, x)
Right subtree → (x, +∞)
```

The range gets narrower as we move down the tree.

### Dry Run

Consider:

```text id="8q0mgl"
        5
       / \
      3   7
     / \   \
    2   4   8
```

Start:

```text id="6xqjv3"
5 → (-∞, +∞)
```

Left child `3`:

```text id="b2t3p8"
3 → (-∞, 5)
```

Its left child `2`:

```text id="a7z5fy"
2 → (-∞, 3)
```

Its right child `4`:

```text id="7p9h2w"
4 → (3, 5)
```

Right child `7`:

```text id="n5r8kb"
7 → (5, +∞)
```

Its right child `8`:

```text id="q9k4s1"
8 → (7, +∞)
```

Every node satisfies its range, so the tree is valid.

### Invalid Example

```text id="1z5f3d"
        5
       / \
      1   6
         /
        3
```

For `6`:

```text id="x8v0dn"
6 → (5, +∞)
```

Therefore its left child `3` must be:

```text id="c5w2ya"
(5, 6)
```

But:

```text id="3 < 5"
```

So the node violates the valid range.

Return `false`.

### Algorithm

1. Start DFS from the root with range:

   ```text
   (-∞, +∞)
   ```
2. If the node is `nullptr`, return `true`.
3. Check whether:

   ```text
   min < node->val < max
   ```
4. If not, return `false`.
5. Recursively validate the left subtree with:

   ```text
   (min, node->val)
   ```
6. Recursively validate the right subtree with:

   ```text
   (node->val, max)
   ```
7. Return the result of both subtrees.

### Complexity

* **Time:** `O(n)`
* **Auxiliary Space:** `O(h)` recursion stack
* **Worst-case Space:** `O(n)`
* **Balanced Tree:** `O(log n)`

### Notes / Tips

* The range must be **strict**:

  ```text
  min < node->val < max
  ```

  because duplicate values are not allowed in a valid BST for this problem.
* Checking only the immediate children is incorrect.
* A node is constrained by **all of its ancestors**, not just its parent.
* Use `long long` for the bounds so values at `INT_MIN` and `INT_MAX` are handled safely.
* This is one of the most important BST validation patterns.

### Code

```cpp id="0s8xq3"
class Solution {
public:
    bool validate(TreeNode* root, long long minVal, long long maxVal) {
        if (root == nullptr) {
            return true;
        }

        if (root->val <= minVal || root->val >= maxVal) {
            return false;
        }

        return validate(root->left, minVal, root->val) &&
               validate(root->right, root->val, maxVal);
    }

    bool isValidBST(TreeNode* root) {
        return validate(root, LLONG_MIN, LLONG_MAX);
    }
};
```

---

## Approach 2 — Inorder Traversal

### Idea

A very useful BST property is:

> **Inorder traversal of a valid BST produces values in strictly increasing order.**

Inorder:

```text
Left → Root → Right
```

So we can perform inorder traversal and check whether every value is greater than the previous value.

Example:

```text id="n1jzq5"
        5
       / \
      3   7
     / \
    2   4
```

Inorder:

```text id="c8u2v0"
[2,3,4,5,7]
```

Strictly increasing → valid BST.

Invalid example:

```text id="q1r4z6"
        5
       / \
      1   4
```

Inorder:

```text id="m0x3av"
[1,5,4]
```

Since:

```text id="3l4w9c"
4 < 5
```

the sequence is not increasing.

### Dry Run

Consider:

```text id="9a4v2d"
        2
       / \
      1   3
```

Inorder traversal:

```text id="7m0s4k"
1 → 2 → 3
```

Keep track of the previous value:

```text id="r3f7xm"
prev = 1
current = 2
2 > 1 ✓

prev = 2
current = 3
3 > 2 ✓
```

Therefore:

```text id="5z8q2c"
true
```

### Algorithm

1. Perform an inorder traversal.
2. Maintain the previously visited value.
3. For every current node:

   * Check:

     ```cpp
     current->val > prev
     ```
   * If not, return `false`.
4. Update `prev`.
5. Continue until the entire tree is processed.
6. If every value is strictly increasing, return `true`.

### Complexity

* **Time:** `O(n)`
* **Auxiliary Space:** `O(h)` recursion stack
* **Worst-case Space:** `O(n)`

### Notes / Tips

* This approach relies on the key BST property:

  ```text
  BST → inorder is strictly increasing
  ```
* Use `long long` for `prev` to safely handle `INT_MIN`.
* The comparison must be:

  ```cpp
  current->val <= prev
  ```

  for detecting invalidity.
* This approach is especially elegant when you already know inorder traversal well.
* The range approach is generally more directly tied to the BST definition.

### Code

```cpp id="8m5wz2"
class Solution {
public:
    long long prev = LLONG_MIN;

    bool inorder(TreeNode* root) {
        if (root == nullptr) {
            return true;
        }

        if (!inorder(root->left)) {
            return false;
        }

        if (root->val <= prev) {
            return false;
        }

        prev = root->val;

        return inorder(root->right);
    }

    bool isValidBST(TreeNode* root) {
        return inorder(root);
    }
};
```

---

## Key Takeaway

### Best Pattern to Remember

```text
BST Validation
      ↓
Every node must satisfy ALL ancestor constraints
      ↓
Use a valid range
```

```text
Left  → (min, node->val)
Right → (node->val, max)
```

### Important BST Property

```text
Valid BST
   ↓
Inorder Traversal
   ↓
Strictly Increasing Sequence
```

### Common Mistake

Do **not** only check:

```cpp
left < root < right
```

for each node.

A node must satisfy constraints imposed by its **entire path from the root**.

**Pattern:**

> To validate a BST, either maintain a valid `(min, max)` range during DFS or verify that inorder traversal is strictly increasing.
