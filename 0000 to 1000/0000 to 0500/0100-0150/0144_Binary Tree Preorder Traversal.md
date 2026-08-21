# LeetCode 144 — Binary Tree Preorder Traversal

## Metadata

* **LeetCode:** 144
* **Problem:** Binary Tree Preorder Traversal
* **Difficulty:** Easy
* **Topics:** Tree, Binary Tree, Depth-First Search, Stack
* **Pattern:** Tree Traversal
* **Key Technique:** Root → Left → Right
* **Key Pattern:** Preorder DFS
* **Key Template:** Visit → Left → Right
* **Optimal Complexity:** `O(n)`

---

## Problem

Given the root of a binary tree, return its **preorder traversal**.

Preorder traversal visits nodes in this order:

```text
Root → Left → Right
```

Example:

```text
        1
         \
          2
         /
        3
```

Preorder traversal:

```text
[1, 2, 3]
```

---

## Idea

There are two common ways to perform preorder traversal:

1. **Recursive DFS**
2. **Iterative DFS using a Stack**

The recursive approach directly follows the definition of preorder:

```text
Visit root
↓
Traverse left subtree
↓
Traverse right subtree
```

For the iterative approach, use a stack.

Since a stack is **LIFO**, push the right child first and the left child second.

That ensures:

```text
Left is processed before Right
```

---

## Approach 1 — Recursive DFS

### Idea

At every node:

1. Add the node's value to the answer.
2. Traverse the left subtree.
3. Traverse the right subtree.

This exactly matches:

```text
Root → Left → Right
```

### Dry Run

Consider:

```text
        1
       / \
      2   3
     / \
    4   5
```

Start at `1`:

```text
Visit 1
```

Go left:

```text
Visit 2
```

Go left:

```text
Visit 4
```

Back to `2`, go right:

```text
Visit 5
```

Back to `1`, go right:

```text
Visit 3
```

Result:

```text
[1, 2, 4, 5, 3]
```

### Algorithm

1. If `root == nullptr`, return.
2. Add `root->val` to the answer.
3. Recursively traverse `root->left`.
4. Recursively traverse `root->right`.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(h)`

  * `h` = tree height due to recursion stack.
  * Worst case: `O(n)`
  * Balanced tree: `O(log n)`

### Code

```cpp
class Solution {
public:
    vector<int> result;

    void preorder(TreeNode* root) {
        if (root == nullptr) {
            return;
        }

        result.push_back(root->val);

        preorder(root->left);
        preorder(root->right);
    }

    vector<int> preorderTraversal(TreeNode* root) {
        preorder(root);
        return result;
    }
};
```

---

## Approach 2 — Iterative DFS

### Idea

Use a stack to simulate recursion.

At each step:

1. Pop a node.
2. Visit it.
3. Push its right child.
4. Push its left child.

Why right first?

Because the stack is LIFO:

```text
Push Right
Push Left

Stack top → Left
```

So the left child gets processed first.

### Dry Run

For:

```text
        1
       / \
      2   3
     / \
    4   5
```

Initial:

```text
Stack = [1]
```

Pop `1`:

```text
Result = [1]
```

Push right `3`, then left `2`:

```text
Stack = [3, 2]
```

Pop `2`:

```text
Result = [1, 2]
```

Push `5`, then `4`:

```text
Stack = [3, 5, 4]
```

Pop `4`:

```text
Result = [1, 2, 4]
```

Pop `5`:

```text
Result = [1, 2, 4, 5]
```

Pop `3`:

```text
Result = [1, 2, 4, 5, 3]
```

Final answer:

```text
[1, 2, 4, 5, 3]
```

### Algorithm

1. If `root == nullptr`, return an empty result.
2. Create a stack and push `root`.
3. While the stack is not empty:

   * Pop the top node.
   * Add its value to the result.
   * Push its right child if it exists.
   * Push its left child if it exists.
4. Return the result.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(h)` average/balanced, `O(n)` worst case.

### Code

```cpp
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        vector<int> result;

        if (root == nullptr) {
            return result;
        }

        stack<TreeNode*> st;
        st.push(root);

        while (!st.empty()) {
            TreeNode* node = st.top();
            st.pop();

            result.push_back(node->val);

            if (node->right != nullptr) {
                st.push(node->right);
            }

            if (node->left != nullptr) {
                st.push(node->left);
            }
        }

        return result;
    }
};
```

---

## Notes / Tips

* Preorder means:

  ```text
  Root → Left → Right
  ```
* In iterative preorder, **push right before left**.
* Recursive DFS naturally follows the traversal order.
* Iterative DFS is useful when recursion depth could become too large.
* `nullptr` nodes should not be pushed into the stack.
* For an empty tree:

  ```text
  root = nullptr → []
  ```

---

## Basic Template

### Recursive

```cpp
void preorder(TreeNode* root) {
    if (root == nullptr) {
        return;
    }

    // Visit root

    preorder(root->left);
    preorder(root->right);
}
```

### Iterative

```cpp
stack<TreeNode*> st;
st.push(root);

while (!st.empty()) {
    TreeNode* node = st.top();
    st.pop();

    // Visit node

    if (node->right != nullptr) {
        st.push(node->right);
    }

    if (node->left != nullptr) {
        st.push(node->left);
    }
}
```

### Reusable Pattern

```text
             Preorder
                ↓
         Visit Root first
                ↓
          Traverse Left
                ↓
         Traverse Right
```
