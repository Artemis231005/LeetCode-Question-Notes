# LeetCode 101 — Symmetric Tree

## Metadata

* **LeetCode:** 101
* **Problem:** Symmetric Tree
* **Difficulty:** Easy
* **Topics:** Tree, Depth-First Search, Breadth-First Search, Binary Tree
* **Pattern:** Mirror Tree, Recursive Comparison
* **Key Technique:** Compare mirror-positioned nodes
* **Optimal Complexity:** `O(n)` Time, `O(h)` Space

---

## Problem

Given the root of a binary tree, determine whether the tree is **symmetric around its center**.

A tree is symmetric if its left and right subtrees are **mirror images** of each other.

### Example

```text
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

Result:

```text
true
```

The corresponding nodes have equal values and appear in mirror positions.

---

# Approach 1 — Recursive DFS

## Idea

A tree is symmetric if its **left subtree and right subtree are mirrors** of each other.

For two nodes `left` and `right` to be mirrors:

1. Both are `NULL`, or
2. Exactly one is `NULL` → not a mirror, or
3. Their values are equal.
4. The outer children are mirrors:

   ```text
   left->left  ↔ right->right
   ```
5. The inner children are mirrors:

   ```text
   left->right ↔ right->left
   ```

The important difference from **Same Tree (100)** is that children are compared in **cross order**.

## Dry Run

```text
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

Start with:

```text
left = 2
right = 2
```

Values match.

Compare outer children:

```text
left->left  = 3
right->right = 3
```

Values match.

Compare inner children:

```text
left->right = 4
right->left = 4
```

Values match.

Continue recursively until all corresponding mirror positions are checked.

All pairs match:

```text
answer = true
```

## Algorithm

1. If both nodes are `NULL`, return `true`.
2. If exactly one node is `NULL`, return `false`.
3. If their values are different, return `false`.
4. Recursively compare `left->left` with `right->right`.
5. Recursively compare `left->right` with `right->left`.
6. Return `true` only if both recursive comparisons return `true`.
7. Start the comparison with `root->left` and `root->right`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(h)` for the recursion stack
* **Worst-case Space:** `O(n)` for a skewed tree
* **Balanced Tree Space:** `O(log n)`

## Notes / Tips

* The key idea is **mirror comparison**, not normal tree comparison.
* Remember:

  ```text
  left.left  ↔ right.right
  left.right ↔ right.left
  ```
* `NULL` positions are important because the structure must also be symmetric.
* This is essentially **Same Tree (100)** with the child comparisons reversed.

## Code

```cpp
class Solution {
public:
    bool isMirror(TreeNode* left, TreeNode* right) {
        if (left == NULL && right == NULL) {
            return true;
        }

        if (left == NULL || right == NULL) {
            return false;
        }

        if (left->val != right->val) {
            return false;
        }

        return isMirror(left->left, right->right) &&
               isMirror(left->right, right->left);
    }

    bool isSymmetric(TreeNode* root) {
        if (root == NULL) {
            return true;
        }

        return isMirror(root->left, root->right);
    }
};
```

---

# Approach 2 — Iterative DFS

## Idea

The recursive solution uses the function call stack to store pairs of nodes that still need to be compared.

We can explicitly maintain these pairs using a `stack`.

Each stack entry contains:

```text
(left node, right node)
```

The pair must represent two **mirror-positioned nodes**.

When a pair matches, push their mirror children:

```text
left->left  ↔ right->right
left->right ↔ right->left
```

## Dry Run

For:

```text
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

Initial stack:

```text
[(2, 2)]
```

Pop `(2, 2)`:

```text
2 == 2
```

Push:

```text
(3, 3)
(4, 4)
```

Process `(4, 4)`:

```text
4 == 4
```

Their corresponding children are both `NULL`.

Process `(3, 3)` similarly.

The stack eventually becomes empty without finding a mismatch.

```text
answer = true
```

## Algorithm

1. If `root == NULL`, return `true`.
2. Create a stack of pairs of tree nodes.
3. Push `{root->left, root->right}`.
4. While the stack is not empty:

   1. Pop a pair `(left, right)`.
   2. If both are `NULL`, continue.
   3. If exactly one is `NULL`, return `false`.
   4. If their values differ, return `false`.
   5. Push `{left->left, right->right}`.
   6. Push `{left->right, right->left}`.
5. If every pair matches, return `true`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)` worst case

## Notes / Tips

* Store **pairs of nodes**, not individual nodes.
* Every pair represents a mirror comparison.
* The order in which children are pushed is important.
* This avoids recursion depth issues for very deep trees.

## Code

```cpp
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if (root == NULL) {
            return true;
        }

        stack<pair<TreeNode*, TreeNode*>> st;
        st.push({root->left, root->right});

        while (!st.empty()) {
            auto [left, right] = st.top();
            st.pop();

            if (left == NULL && right == NULL) {
                continue;
            }

            if (left == NULL || right == NULL) {
                return false;
            }

            if (left->val != right->val) {
                return false;
            }

            st.push({left->left, right->right});
            st.push({left->right, right->left});
        }

        return true;
    }
};
```

---

# Approach 3 — Iterative BFS

## Idea

The same mirror comparison can be performed level by level using a `queue`.

Instead of processing individual nodes, store pairs of mirror-positioned nodes:

```text
(left, right)
```

For every pair:

* Both `NULL` → continue.
* Exactly one `NULL` → `false`.
* Different values → `false`.
* Otherwise, push their mirror child pairs.

## Dry Run

For:

```text
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

Initial queue:

```text
[(2, 2)]
```

Process:

```text
2 == 2
```

Add:

```text
(3, 3)
(4, 4)
```

Both pairs match.

Their children produce:

```text
(NULL, NULL)
(NULL, NULL)
```

All pairs are valid.

```text
answer = true
```

## Algorithm

1. If `root == NULL`, return `true`.
2. Create a queue of pairs of tree nodes.
3. Push `{root->left, root->right}`.
4. While the queue is not empty:

   1. Remove a pair `(left, right)`.
   2. If both are `NULL`, continue.
   3. If exactly one is `NULL`, return `false`.
   4. If their values differ, return `false`.
   5. Push `{left->left, right->right}`.
   6. Push `{left->right, right->left}`.
5. If all mirror pairs match, return `true`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(w)`, where `w` is the maximum width of the tree
* **Worst-case Space:** `O(n)`

## Notes / Tips

* BFS is useful when thinking about the tree level by level.
* The queue still stores **mirror pairs**, not ordinary parent-child pairs.
* The essential relationships remain:

  ```text
  left.left  ↔ right.right
  left.right ↔ right.left
  ```
* BFS can require more memory than recursive DFS for a very wide tree.

## Code

```cpp
class Solution {
public:
    bool isSymmetric(TreeNode* root) {
        if (root == NULL) {
            return true;
        }

        queue<pair<TreeNode*, TreeNode*>> q;
        q.push({root->left, root->right});

        while (!q.empty()) {
            auto [left, right] = q.front();
            q.pop();

            if (left == NULL && right == NULL) {
                continue;
            }

            if (left == NULL || right == NULL) {
                return false;
            }

            if (left->val != right->val) {
                return false;
            }

            q.push({left->left, right->right});
            q.push({left->right, right->left});
        }

        return true;
    }
};
```


