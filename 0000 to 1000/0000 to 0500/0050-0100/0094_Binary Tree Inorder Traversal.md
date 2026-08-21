# LeetCode 94 — Binary Tree Inorder Traversal

## Metadata

* **LeetCode:** 94
* **Problem:** Binary Tree Inorder Traversal
* **Difficulty:** Easy
* **Topics:** Stack, Tree, Depth-First Search, Binary Tree
* **Pattern:** Tree Traversal
* **Key Technique:** Left → Root → Right
* **Key Pattern:** Inorder Traversal
* **Key Template:** Recursive DFS / Iterative Stack
* **Optimal Complexity:** `O(n)` time, `O(h)` auxiliary space

---

## Problem

Given the root of a binary tree, return its **inorder traversal**.

Inorder traversal follows:

```text
Left → Root → Right
```

Example:

```text
        1
         \
          2
         /
        3
```

Inorder traversal:

```text
[1,3,2]
```

---

## Approach 1 — Recursion

### Idea

For every node:

1. Traverse its left subtree.
2. Process the current node.
3. Traverse its right subtree.

This directly follows:

```text
Left → Root → Right
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

Start at `1`.

```text
Go Left → 2
```

At `2`:

```text
Go Left → 4
```

`4` has no left child:

```text
Add 4
```

Return to `2`:

```text
Add 2
```

Then go right:

```text
Add 5
```

Return to `1`:

```text
Add 1
```

Go right:

```text
Add 3
```

Final:

```text
[4,2,5,1,3]
```

### Algorithm

1. Create an empty result array.
2. Define a recursive function `inorder(node)`.
3. If `node == nullptr`, return.
4. Recursively traverse `node->left`.
5. Add `node->val` to the result.
6. Recursively traverse `node->right`.
7. Return the result.

### Complexity

* **Time:** `O(n)`
* **Auxiliary Space:** `O(h)` recursion stack
* **Worst-case Space:** `O(n)` for a completely skewed tree
* **Balanced Tree:** `O(log n)` stack space

### Notes / Tips

* This is the most direct implementation because the recursive structure naturally matches inorder traversal.
* The order is always:

  ```text
  LEFT → ROOT → RIGHT
  ```
* For a **BST**, inorder traversal produces values in **sorted order**.

### Code

```cpp id="5dq2v1"
class Solution {
public:
    vector<int> ans;

    void inorder(TreeNode* root) {
        if (root == nullptr) {
            return;
        }

        inorder(root->left);

        ans.push_back(root->val);

        inorder(root->right);
    }

    vector<int> inorderTraversal(TreeNode* root) {
        inorder(root);
        return ans;
    }
};
```

---

## Approach 2 — Iterative Using Stack

### Idea

Recursion internally uses a **call stack**.

We can simulate the same process explicitly using a `stack`.

We first keep moving left while pushing nodes onto the stack.

When there is no more left child:

1. Pop a node.
2. Process it.
3. Move to its right child.
4. Repeat.

### Dry Run

Consider:

```text
        1
       / \
      2   3
     / \
    4   5
```

Start:

```text
stack = []
current = 1
```

Move left:

```text
push 1
push 2
push 4
```

Now:

```text
current = nullptr
stack = [1,2,4]
```

Pop `4`:

```text
answer = [4]
```

Move right of `4` → `nullptr`.

Pop `2`:

```text
answer = [4,2]
```

Move right → `5`.

Push `5`:

```text
stack = [1,5]
```

Pop `5`:

```text
answer = [4,2,5]
```

Pop `1`:

```text
answer = [4,2,5,1]
```

Move right → `3`.

Pop `3`:

```text
answer = [4,2,5,1,3]
```

### Algorithm

1. Create an empty stack.
2. Set:

   ```cpp
   current = root
   ```
3. Continue while either `current` exists or the stack is not empty.
4. While `current` is not `nullptr`:

   * Push `current` onto the stack.
   * Move to `current->left`.
5. Pop the top node.
6. Add its value to the result.
7. Move to its right child.
8. Repeat until both `current == nullptr` and the stack is empty.

### Complexity

* **Time:** `O(n)`
* **Auxiliary Space:** `O(h)`
* **Worst-case Space:** `O(n)`
* **Balanced Tree:** `O(log n)`

### Notes / Tips

* The stack replaces recursion.
* The key pattern is:

  ```text
  Keep going LEFT
       ↓
  Pop + process ROOT
       ↓
  Go RIGHT
  ```
* The loop condition must be:

  ```cpp
  while (current != nullptr || !st.empty())
  ```
* Do not use only `while (current != nullptr)` because after reaching `nullptr`, there may still be nodes waiting in the stack.

### Code

```cpp id="j4jv0e"
class Solution {
public:
    vector<int> inorderTraversal(TreeNode* root) {
        vector<int> ans;
        stack<TreeNode*> st;

        TreeNode* current = root;

        while (current != nullptr || !st.empty()) {
            while (current != nullptr) {
                st.push(current);
                current = current->left;
            }

            current = st.top();
            st.pop();

            ans.push_back(current->val);

            current = current->right;
        }

        return ans;
    }
};
```

---

## Key Takeaway

### Inorder

```text
LEFT → ROOT → RIGHT
```

### Recursive Template

```cpp
inorder(root->left);
process(root);
inorder(root->right);
```

### Iterative Template

```text
Go left while possible
        ↓
Push nodes into stack
        ↓
Pop node
        ↓
Process node
        ↓
Move right
```

**Important BST Property:**

```text
Inorder traversal of a BST
            ↓
Produces elements in sorted order
```

**Pattern:**

> Inorder = Left → Root → Right. Recursion is the natural approach; an explicit stack can simulate the recursive call stack.
