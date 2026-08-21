# LeetCode 124 — Binary Tree Maximum Path Sum

## Metadata

* **LeetCode:** 124
* **Problem:** Binary Tree Maximum Path Sum
* **Difficulty:** Hard
* **Topics:** Tree, Binary Tree, Depth-First Search, Dynamic Programming
* **Pattern:** Tree DP, Postorder DFS
* **Key Technique:** Maximum downward gain + global maximum
* **Key Pattern:** Postorder DFS with global answer
* **Key Template:** `max(0, childGain)` + update global maximum
* **Optimal Complexity:** `O(n)`

---

## Problem

Given the root of a binary tree, find the **maximum path sum**.

A path:

* Can start and end at **any node**.
* Must follow connected parent-child edges.
* Does not have to pass through the root.
* Cannot visit the same node more than once.

Example:

```text
        -10
        /  \
       9    20
           /  \
          15   7
```

Maximum path:

```text
15 → 20 → 7
```

Sum:

```text
15 + 20 + 7 = 42
```

Answer:

```text
42
```

---

## Idea

At every node, we calculate two different things.

### 1. Maximum Gain Returned to Parent

When returning to the parent, the current node can continue through **only one child**.

So:

```text
gain = node->val + max(leftGain, rightGain)
```

But a negative subtree only decreases the sum, so ignore it:

```text
leftGain = max(0, leftGain)
rightGain = max(0, rightGain)
```

Therefore:

```text
gain = node->val + max(leftGain, rightGain)
```

### 2. Maximum Path Through Current Node

A complete path can use **both children**:

```text
left → node → right
```

So:

```text
currentPath = leftGain + node->val + rightGain
```

Update the global answer using this value.

### Key Difference

**Return to parent:**

```text
node + max(leftGain, rightGain)
```

because only one side can be extended.

**Update global answer:**

```text
leftGain + node + rightGain
```

because a complete path can use both sides.

---

## Dry Run

Consider:

```text
        -10
        /  \
       9    20
           /  \
          15   7
```

### Node `9`

```text
leftGain = 0
rightGain = 0

currentPath = 9
gain = 9
```

Global answer:

```text
9
```

### Node `15`

```text
currentPath = 15
gain = 15
```

Global answer:

```text
15
```

### Node `7`

```text
currentPath = 7
gain = 7
```

Global answer remains:

```text
15
```

### Node `20`

```text
leftGain = 15
rightGain = 7
```

Path through `20`:

```text
15 + 20 + 7 = 42
```

So:

```text
currentPath = 42
```

Gain returned to parent:

```text
20 + max(15, 7) = 35
```

Global answer:

```text
42
```

### Node `-10`

```text
leftGain = 9
rightGain = 35
```

Path through `-10`:

```text
9 + (-10) + 35 = 34
```

Since:

```text
34 < 42
```

the answer remains:

```text
42
```

---

## Algorithm

1. Initialize a global answer to `INT_MIN`.
2. Perform postorder DFS.
3. For every node:

   * Find the maximum gain from the left subtree.
   * Find the maximum gain from the right subtree.
4. Ignore negative gains:

   ```cpp
   leftGain = max(0, leftGain);
   rightGain = max(0, rightGain);
   ```
5. Calculate the maximum path passing through the current node:

   ```cpp
   currentPath = leftGain + node->val + rightGain;
   ```
6. Update the global answer.
7. Return the maximum gain that can be extended to the parent:

   ```cpp
   node->val + max(leftGain, rightGain)
   ```
8. Return the global answer.

---

## Complexity

* **Time:** `O(n)`

  * Every node is visited exactly once.
* **Space:** `O(h)`

  * Due to recursion stack.
  * Worst case: `O(n)`
  * Balanced tree: `O(log n)`

---

## Notes / Tips

* This is a classic **Tree DP** problem.
* Use **postorder DFS** because a node needs information from both children first.
* Maintain two concepts:

  * **Gain:** what the node can return to its parent.
  * **Path sum:** the best complete path passing through the node.
* Use `max(0, gain)` to discard negative subtrees.
* The global answer must be initialized to `INT_MIN`, not `0`.
* This is necessary because all node values can be negative.

### Important Example

```text
    -3
    /
  -2
```

The answer is:

```text
-2
```

not `0`.

Therefore:

```cpp
int ans = INT_MIN;
```

is important.

### Most Important Formula

```text
leftGain  = max(0, DFS(left))
rightGain = max(0, DFS(right))

currentPath = leftGain + node->val + rightGain

ans = max(ans, currentPath)

return node->val + max(leftGain, rightGain)
```

---

## Code

```cpp
class Solution {
public:
    int ans = INT_MIN;

    int maxGain(TreeNode* node) {
        if (node == nullptr) {
            return 0;
        }

        int leftGain = max(0, maxGain(node->left));
        int rightGain = max(0, maxGain(node->right));

        int currentPath = node->val + leftGain + rightGain;

        ans = max(ans, currentPath);

        return node->val + max(leftGain, rightGain);
    }

    int maxPathSum(TreeNode* root) {
        maxGain(root);
        return ans;
    }
};
```

## Template
```cpp
int dfs(TreeNode* node) {
    if (node == nullptr) {
        return 0;
    }

    int left = max(0, dfs(node->left));
    int right = max(0, dfs(node->right));

    ans = max(ans, left + node->val + right);

    return node->val + max(left, right);
}
```