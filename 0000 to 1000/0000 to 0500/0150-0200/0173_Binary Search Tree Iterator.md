# LeetCode 173 — Binary Search Tree Iterator

## Metadata

- **LeetCode:** 173
- **Problem:** Binary Search Tree Iterator
- **Difficulty:** Medium
- **Topics:** Tree, Stack, Design, Binary Search Tree, Iterator
- **Pattern:** Controlled Inorder Traversal
- **Key Technique:** Use a stack to simulate inorder traversal one step at a time instead of precomputing the whole sequence

---

# Approaches

1. **Brute Force — Precompute Inorder Array**
2. **Optimal — Stack-Based Controlled Recursion**

---

# Approach 1 — Brute Force / Precompute Inorder Array

## Idea

Do a full inorder traversal upfront and store all node values in a list. `next()` just returns the next element from the list, `hasNext()` checks if there's anything left.

## Dry Run

```text
Tree:
        7
       / \
      3   15
         /  \
        9    20
```

Inorder traversal:
```text
[3, 7, 9, 15, 20]
```

Calls:
```text
next() → 3
next() → 7
hasNext() → true
next() → 9
```

## Algorithm

1. Traverse the tree inorder, storing values in a list.
2. Maintain a pointer/index starting at `0`.
3. `next()` returns `list[index]` and increments index.
4. `hasNext()` returns `index < list.size()`.

## Complexity

- **Time:** `O(n)` for initial traversal, `O(1)` per `next()`/`hasNext()`
- **Space:** `O(n)` — stores every node's value

## Notes / Tips

- Simple but defeats the purpose of an "iterator" — uses `O(n)` space even if only a few calls are made.
- Good starting point before optimizing to `O(h)` space.

## Code

```cpp
class BSTIterator {
public:
    vector<int> values;
    int index = 0;

    void inorder(TreeNode* node) {
        if (!node) return;
        inorder(node->left);
        values.push_back(node->val);
        inorder(node->right);
    }

    BSTIterator(TreeNode* root) {
        inorder(root);
    }

    int next() {
        return values[index++];
    }

    bool hasNext() {
        return index < values.size();
    }
};
```

---

# Approach 2 — Optimal / Stack-Based Controlled Recursion

## Idea

Simulate the inorder traversal manually using a stack instead of recursion, and only go one step at a time. Push all left children onto the stack first. Each `next()` pops the top, and if that node has a right child, push that child and all of its left descendants.

## Dry Run

```text
Tree:
        7
       / \
      3   15
         /  \
        9    20
```

Initialize (push left spine from root):
```text
stack = [7, 3]
```

`next()`:
```text
pop 3 → return 3
3 has no right child
stack = [7]
```

`next()`:
```text
pop 7 → return 7
7 has right child 15 → push 15, then push left spine of 15 (9)
stack = [15, 9]
```

`next()`:
```text
pop 9 → return 9
9 has no children
stack = [15]
```

`next()`:
```text
pop 15 → return 15
15 has right child 20 → push 20 (no left child)
stack = [20]
```

`next()`:
```text
pop 20 → return 20
stack = []
```

`hasNext()`:
```text
false
```

## Algorithm

1. Initialize a stack.
2. Push `root` and all its left descendants onto the stack (helper `pushLeft`).
3. `next()`:
   - Pop the top node, this is the answer.
   - If it has a right child, push that child and all of its left descendants.
   - Return the popped value.
4. `hasNext()` returns whether the stack is non-empty.

## Complexity

- **Time:** `O(1)` amortized per `next()` call (each node pushed/popped exactly once overall), `O(h)` for constructor
- **Space:** `O(h)` where `h` is tree height — only the current left spine is stored

## Notes / Tips

- This is the standard trick for turning any recursive traversal into a resumable iterator — replace the call stack with an explicit stack and pause between steps.
- Amortized `O(1)` for `next()` holds because total work across all calls is bounded by `O(n)`, even though a single call can do `O(h)` work when pushing a new left spine.
- Common mistake: pushing the entire subtree instead of just the left spine — that brings space back up to `O(n)`.

## Code

```cpp
class BSTIterator {
public:
    stack<TreeNode*> st;

    void pushLeft(TreeNode* node) {
        while (node) {
            st.push(node);
            node = node->left;
        }
    }

    BSTIterator(TreeNode* root) {
        pushLeft(root);
    }

    int next() {
        TreeNode* node = st.top();
        st.pop();
        if (node->right) {
            pushLeft(node->right);
        }
        return node->val;
    }

    bool hasNext() {
        return !st.empty();
    }
};
```

---

# Key Template

### Iterative Inorder Traversal (Resumable)

```text
stack = []

function pushLeft(node):
    while node:
        stack.push(node)
        node = node.left

function next():
    node = stack.pop()
    if node.right:
        pushLeft(node.right)
    return node.val

function hasNext():
    return stack not empty
```

## Pattern Recognition

When you see:

```text
Iterator
+
Tree traversal
+
O(h) space constraint
```

Think:

```text
Full traversal + array → O(n) space
        ↓
Optimal: explicit stack, push left spine, pop-and-expand on next()
```

The key observation is:

> **Any recursive traversal can be paused and resumed by replacing the call stack with an explicit stack, pushing only what's needed for the next step.**