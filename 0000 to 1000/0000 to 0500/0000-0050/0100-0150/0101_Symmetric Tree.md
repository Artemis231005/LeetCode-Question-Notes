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

The tree is symmetric:

```text
3 ↔ 3
4 ↔ 4
2 ↔ 2
```

Result:

```text
true
```

A tree such as:

```text
        1
       / \
      2   2
       \   \
        3   3
```

is not symmetric because the `3`s are on the same side instead of being mirror images.

---

# Approach 1 — Recursive DFS

## Idea

A tree is symmetric if its **left subtree and right subtree are mirrors of each other**.

Therefore, instead of comparing:

```text
left.left  ↔ right.left
left.right ↔ right.right
```

we compare **mirror positions**:

```text
left.left  ↔ right.right
left.right ↔ right.left
```

For two nodes `p` and `q` to be mirrors:

1. Both must be `NULL`, or
2. Exactly one must be `NULL` → not a mirror, or
3. Their values must be equal.
4. Their outer children must match:

   ```text
   p->left  ↔ q->right
   ```
5. Their inner children must match:

   ```text
   p->right ↔ q->left
   ```

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
left  = 2
right = 2
```

Values match.

Compare outer children:

```text
left->left  = 3
right->right = 3
```

Values match.

Compare:

```text
3's left  ↔ 3's right
3's right ↔ 3's left
```

Both are `NULL`.

Now compare inner children:

```text
left->right = 4
right->left = 4
```

Values match.

Again, their children are mirror pairs.

Therefore:

```text
answer = true
```

## Important Difference From Same Tree

For **Same Tree (100)**:

```text
p->left  ↔ q->left
p->right ↔ q->right
```

For **Symmetric Tree (101)**:

```text
p->left  ↔ q->right
p->right ↔ q->left
```

The second tree is not being compared in the same orientation; it is being compared as a **mirror**.

## Notes / Tips

* The root does not need to be compared with itself; symmetry is checked between its left and right subtrees.
* The key is remembering the **cross comparison**:

  ```text
  left.left  ↔ right.right
  left.right ↔ right.left
  ```
* Values alone are not enough; the structure must also be mirrored.
* `NULL` positions are important for detecting structural asymmetry.
* This is essentially the **Same Tree** problem with mirrored child comparisons.

## Complexity

Let `n` be the number of nodes and `h` be the tree height.

* **Time:** `O(n)`
* **Space:** `O(h)` recursion stack
* **Worst-case Space:** `O(n)` for a skewed tree
* **Balanced Tree Space:** `O(log n)`

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

The recursive solution can be converted into an explicit stack.

Instead of recursively calling:

```text
isMirror(left->left, right->right)
isMirror(left->right, right->left)
```

store the pairs that need to be compared in a stack.

Each pair represents two nodes that should be mirror images.

## Dry Run

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

Push mirror pairs:

```text
(2->left, 2->right)   → (3, 3)
(2->right, 2->left)   → (4, 4)
```

Process `(4, 4)`:

```text
4 == 4
```

Their mirror children are both `NULL`.

Process `(3, 3)` similarly.

Stack becomes empty.

```text
answer = true
```

## Notes / Tips

* Store **pairs of nodes**, not individual nodes.
* Every pair in the stack represents a mirror comparison.
* The child pairs must always be pushed in cross order:

  ```text
  left.left  ↔ right.right
  left.right ↔ right.left
  ```
* This avoids recursion depth issues for very deep trees.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(h)` average/balanced
* **Worst-case Space:** `O(n)`

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

The same mirror comparison can be performed level by level using a queue.

Store corresponding **mirror-positioned pairs**:

```text
(left node, right node)
```

For every pair:

* Both `NULL` → continue.
* One `NULL` → `false`.
* Different values → `false`.
* Otherwise, add the two mirror child pairs.

## Dry Run

```text
        1
       / \
      2   2
     / \ / \
    3  4 4  3
```

Queue:

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

Process the next level.

Both pairs match.

Their children produce:

```text
(NULL, NULL)
(NULL, NULL)
```

All mirror pairs match.

```text
answer = true
```

## Notes / Tips

* BFS is useful when thinking in terms of levels.
* The queue stores mirror pairs rather than adjacent nodes.
* The same mirror relationships are maintained:

  ```text
  left.left  ↔ right.right
  left.right ↔ right.left
  ```
* BFS can use `O(n)` space in the worst case because an entire level may contain many nodes.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(w)`, where `w` is the maximum tree width
* **Worst-case Space:** `O(n)`

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

---

# Key Pattern

```text
              root
             /    \
            L      R
             \    /
              \  /
           Mirror Pair
```

For every pair `(L, R)`:

```text
L->val == R->val

L->left  ↔ R->right
L->right ↔ R->left
```

### Core Recursive Template

```cpp
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
