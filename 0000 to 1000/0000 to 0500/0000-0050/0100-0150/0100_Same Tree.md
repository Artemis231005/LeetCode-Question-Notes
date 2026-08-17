# LeetCode 100 — Same Tree

## Metadata

* **LeetCode:** 100
* **Problem:** Same Tree
* **Difficulty:** Easy
* **Topics:** Tree, Depth-First Search, Breadth-First Search, Binary Tree
* **Pattern:** Tree Traversal, Recursive Comparison
* **Key Technique:** Compare corresponding nodes
* **Optimal Complexity:** `O(n)` Time, `O(h)` Space

---

## Problem

Given the roots of two binary trees `p` and `q`, determine whether the two trees are **the same**.

Two binary trees are considered the same if:

1. They have the same structure.
2. Corresponding nodes have the same values.

Return `true` if the trees are identical, otherwise return `false`.

### Example

```text
Tree p:        1
              / \
             2   3

Tree q:        1
              / \
             2   3

Result = true
```

```text
Tree p:        1
              /
             2

Tree q:        1
                \
                 2

Result = false
```

The values are the same, but the structure is different.

---

# Approach 1 — Recursive DFS

## Idea

Compare the two trees recursively.

For two corresponding nodes `p` and `q`:

### Case 1 — Both are `NULL`

```text
p = NULL
q = NULL
```

There is nothing left to compare.

```text
→ true
```

### Case 2 — Exactly One Is `NULL`

```text
p = NULL
q != NULL
```

or

```text
p != NULL
q = NULL
```

The structures are different.

```text
→ false
```

### Case 3 — Values Are Different

```text
p->val != q->val
```

The trees cannot be identical.

```text
→ false
```

### Case 4 — Values Are Equal

If the current nodes match, recursively compare:

```text
p->left  with q->left
p->right with q->right
```

Both must be identical.

```text
sameTree(p, q)
    ↓
p and q match?
    ↓
compare left subtrees
    AND
compare right subtrees
```

## Dry Run

```text
p:          1
           / \
          2   3

q:          1
           / \
          2   3
```

Start:

```text
p = 1, q = 1
1 == 1 → continue
```

Left:

```text
p = 2, q = 2
2 == 2 → continue

left:
NULL == NULL → true

right:
NULL == NULL → true
```

Right:

```text
p = 3, q = 3
3 == 3 → continue

left:
NULL == NULL → true

right:
NULL == NULL → true
```

Every corresponding node matches.

```text
answer = true
```

## Dry Run — Different Structure

```text
p:          1
           /
          2

q:          1
             \
              2
```

Root:

```text
1 == 1
```

Left subtree:

```text
p->left  = 2
q->left  = NULL
```

Exactly one is `NULL`.

Therefore:

```text
answer = false
```

## Notes / Tips

* Both **value** and **structure** must match.
* Checking only node values is not enough.
* The `NULL` checks are what detect structural differences.
* Once one mismatch is found, the remaining tree does not need to be examined.
* For every pair of corresponding nodes, the same three checks are used:

  1. Both `NULL`
  2. One `NULL`
  3. Values equal → recursively compare children

## Complexity

Let `n` be the number of nodes examined across the trees and `h` be the tree height.

* **Time:** `O(n)`
* **Space:** `O(h)` recursion stack
* **Worst-case Space:** `O(n)` for a completely skewed tree
* **Balanced Tree Space:** `O(log n)`

## Code

```cpp
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        // Both nodes are NULL
        if (p == NULL && q == NULL)
            return true;

        // Exactly one node is NULL
        if (p == NULL || q == NULL)
            return false;

        // Values are different
        if (p->val != q->val)
            return false;

        // Compare corresponding subtrees
        return isSameTree(p->left, q->left) &&
               isSameTree(p->right, q->right);
    }
};
```

---

# Approach 2 — Iterative DFS

## Idea

The recursive solution uses the call stack to remember which nodes still need to be compared.

We can explicitly maintain this stack instead.

Push the pair:

```text
(p, q)
```

For every pair:

1. If both are `NULL`, continue.
2. If exactly one is `NULL`, return `false`.
3. If their values differ, return `false`.
4. Push:

   * `(p->left, q->left)`
   * `(p->right, q->right)`

If every pair matches, the trees are identical.

## Dry Run

```text
p:          1
           / \
          2   3

q:          1
           / \
          2   3
```

Initial stack:

```text
[(1,1)]
```

Pop:

```text
(1,1)
```

Values match.

Push:

```text
(2,2)
(3,3)
```

Process `(3,3)`:

```text
3 == 3
```

Push:

```text
(NULL,NULL)
(NULL,NULL)
```

Both match.

Process `(2,2)` similarly.

Stack becomes empty.

```text
answer = true
```

## Notes / Tips

* This is the iterative equivalent of recursive DFS.
* Store **pairs of corresponding nodes**, not individual nodes.
* The pair `(p, q)` represents one comparison that still needs to be performed.
* The logic for detecting mismatches remains exactly the same as the recursive solution.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(h)` average/balanced, `O(n)` worst case

## Code

```cpp
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        stack<pair<TreeNode*, TreeNode*>> st;

        st.push({p, q});

        while (!st.empty()) {
            auto [a, b] = st.top();
            st.pop();

            if (a == NULL && b == NULL)
                continue;

            if (a == NULL || b == NULL)
                return false;

            if (a->val != b->val)
                return false;

            st.push({a->left, b->left});
            st.push({a->right, b->right});
        }

        return true;
    }
};
```

---

# Approach 3 — Iterative BFS

## Idea

Instead of DFS, compare the trees level by level using a queue.

Store corresponding nodes together:

```text
(p, q)
```

For every pair:

* Both `NULL` → continue.
* One `NULL` → `false`.
* Different values → `false`.
* Otherwise, add corresponding children to the queue.

## Dry Run

```text
p:          1
           / \
          2   3

q:          1
           / \
          2   3
```

Queue:

```text
[(1,1)]
```

Process:

```text
1 == 1
```

Add:

```text
(2,2)
(3,3)
```

Process both pairs.

Then add their corresponding `NULL` children.

All pairs match.

```text
answer = true
```

## Notes / Tips

* BFS is completely valid, but it does not provide a complexity advantage over DFS.
* The queue can become large when the tree has a wide level.
* DFS is generally more natural for this particular recursive tree-comparison problem.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(w)`, where `w` is the maximum width of the tree
* **Worst-case Space:** `O(n)`

## Code

```cpp
class Solution {
public:
    bool isSameTree(TreeNode* p, TreeNode* q) {
        queue<pair<TreeNode*, TreeNode*>> qu;

        qu.push({p, q});

        while (!qu.empty()) {
            auto [a, b] = qu.front();
            qu.pop();

            if (a == NULL && b == NULL)
                continue;

            if (a == NULL || b == NULL)
                return false;

            if (a->val != b->val)
                return false;

            qu.push({a->left, b->left});
            qu.push({a->right, b->right});
        }

        return true;
    }
};
```

---

# Key Pattern

For comparing two binary trees:

```text
Compare current nodes
        ↓
Both NULL?
   ↓          ↓
  Yes         No
  ↓           ↓
true     One NULL?
             ↓
          Different?
             ↓
           false
             ↓
       Values equal
             ↓
     Compare left pairs
             AND
    Compare right pairs
```


