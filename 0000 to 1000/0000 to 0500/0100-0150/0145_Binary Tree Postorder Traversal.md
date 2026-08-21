# LeetCode 145 — Binary Tree Postorder Traversal

## Metadata

* **LeetCode:** 145
* **Problem:** Binary Tree Postorder Traversal
* **Difficulty:** Easy
* **Topics:** Tree, Binary Tree, Depth-First Search, Stack
* **Pattern:** Tree Traversal
* **Key Technique:** Left → Right → Root
* **Key Pattern:** Postorder DFS
* **Key Template:** Left → Right → Visit
* **Optimal Complexity:** `O(n)`

---

## Problem

Given the root of a binary tree, return its **postorder traversal**.

Postorder traversal visits nodes in this order:

```text
Left → Right → Root
```

Example:

```text
        1
       / \
      2   3
     / \
    4   5
```

Postorder traversal:

```text
[4, 5, 2, 3, 1]
```

---

## Idea

There are two common approaches:

1. **Recursive DFS**
2. **Iterative DFS using a Stack**

The recursive approach directly follows the definition:

```text
Traverse Left
↓
Traverse Right
↓
Visit Root
```

The iterative approach is slightly more tricky because the root must be processed **after both children**.

A useful trick is to perform a modified preorder:

```text
Root → Right → Left
```

and then reverse the result:

```text
Left → Right → Root
```

---

## Approach 1 — Recursive DFS

### Idea

At every node:

1. Traverse the left subtree.
2. Traverse the right subtree.
3. Add the current node to the answer.

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

Go left to `2`.

Go left to `4`:

```text
Result = [4]
```

Back to `2`, go right to `5`:

```text
Result = [4, 5]
```

Now process `2`:

```text
Result = [4, 5, 2]
```

Back to `1`, go right to `3`:

```text
Result = [4, 5, 2, 3]
```

Finally process `1`:

```text
Result = [4, 5, 2, 3, 1]
```

### Algorithm

1. If `root == nullptr`, return.
2. Recursively traverse the left subtree.
3. Recursively traverse the right subtree.
4. Add `root->val` to the result.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(h)`

  * `h` = height of tree.
  * Worst case: `O(n)`
  * Balanced tree: `O(log n)`

### Code

```cpp
class Solution {
public:
    vector<int> result;

    void postorder(TreeNode* root) {
        if (root == nullptr) {
            return;
        }

        postorder(root->left);
        postorder(root->right);

        result.push_back(root->val);
    }

    vector<int> postorderTraversal(TreeNode* root) {
        postorder(root);
        return result;
    }
};
```

---

## Approach 2 — Iterative DFS

### Idea

Postorder is:

```text
Left → Right → Root
```

Instead of directly implementing this with a stack, use:

```text
Root → Right → Left
```

Then reverse the result.

Why?

The reverse of:

```text
Root → Right → Left
```

is:

```text
Left → Right → Root
```

which is exactly postorder.

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
Stack = [1]
Result = []
```

Pop `1`:

```text
Result = [1]
```

Push left `2`, then right `3`.

Because the stack is LIFO, `3` is processed first.

```text
Stack = [2, 3]
```

Pop `3`:

```text
Result = [1, 3]
```

Pop `2`:

```text
Result = [1, 3, 2]
```

Push its left `4`, then right `5`:

```text
Stack = [4, 5]
```

Pop `5`:

```text
Result = [1, 3, 2, 5]
```

Pop `4`:

```text
Result = [1, 3, 2, 5, 4]
```

Now reverse:

```text
[4, 5, 2, 3, 1]
```

This is postorder.

### Algorithm

1. If `root == nullptr`, return an empty result.
2. Create a stack and push `root`.
3. While the stack is not empty:

   * Pop the top node.
   * Add it to the result.
   * Push its left child if it exists.
   * Push its right child if it exists.
4. Reverse the result.
5. Return the result.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`

  * Stack and result array.

### Code

```cpp
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) {
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

            if (node->left != nullptr) {
                st.push(node->left);
            }

            if (node->right != nullptr) {
                st.push(node->right);
            }
        }

        reverse(result.begin(), result.end());

        return result;
    }
};
```

---

## Notes / Tips

* Postorder means:

  ```text
  Left → Right → Root
  ```
* Recursive implementation directly follows the definition.
* A simple iterative trick is:

  ```text
  Root → Right → Left
  ```

  followed by reversal.
* In the modified preorder approach, **push left before right** so that right is processed first.
* For an empty tree:

  ```text
  root = nullptr → []
  ```

## Basic Template

### Recursive

```cpp
void postorder(TreeNode* root) {
    if (root == nullptr) {
        return;
    }

    postorder(root->left);
    postorder(root->right);

    // Visit root
}
```

### Iterative

```cpp
vector<int> result;
stack<TreeNode*> st;

st.push(root);

while (!st.empty()) {
    TreeNode* node = st.top();
    st.pop();

    result.push_back(node->val);

    if (node->left != nullptr) {
        st.push(node->left);
    }

    if (node->right != nullptr) {
        st.push(node->right);
    }
}

reverse(result.begin(), result.end());
```

### Reusable Pattern

```text
             Postorder
                 ↓
            Traverse Left
                 ↓
           Traverse Right
                 ↓
             Visit Root
```

### Quick Trick

```text
Preorder:
Root → Left → Right

Modified:
Root → Right → Left

Reverse:
Left → Right → Root

        ↓

Postorder
```
