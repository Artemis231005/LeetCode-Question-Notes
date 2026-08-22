# 199. Binary Tree Right Side View

## Metadata

* **Topic:** Binary Tree, BFS
* **Difficulty:** Medium
* **Pattern:** Level Order Traversal
* **Key Pattern:** Take the **last node of every level**.
* **Key Template:** BFS + level size

---

## Idea

We need to return the nodes visible when looking at the tree from the **right side**.

Using **BFS (level order traversal)**:

* Process one level at a time.
* The **last node** processed at each level is the rightmost node.
* Add that node to the answer.

Example:

```text
        1
       / \
      2   3
       \   \
        5   4
```

Right side view:

```text
[1, 3, 4]
```

At each level:

```text
Level 1 → 1
Level 2 → 3
Level 3 → 4
```

---

## Dry Run

```text
        1
       / \
      2   3
     / \   \
    5   6   4
```

### Level 1

```text
[1]
```

Last node → `1`

### Level 2

```text
[2, 3]
```

Last node → `3`

### Level 3

```text
[5, 6, 4]
```

Last node → `4`

Answer:

```text
[1, 3, 4]
```

---

## Algorithm

1. If `root == NULL`, return an empty answer.
2. Create a queue and push `root`.
3. While the queue is not empty:

   * Store the current level size.
   * Process exactly that many nodes.
   * Add children to the queue.
   * When processing the last node of the level, add its value to the answer.
4. Return the answer.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`

---

## Notes / Tips

The key line is:

```text
if (i == size - 1)
```

because the last node of every BFS level is the **rightmost node**.

### Right Side vs Left Side

For **Right Side View**:

```text
if (i == size - 1)
    ans.push_back(node->val);
```

For **Left Side View**, simply change it to:

```text
if (i == 0)
    ans.push_back(node->val);
```

Everything else in the BFS code remains the **same**.

So remember:

```text
Right View → first level's last node
Left View  → first level's first node
```

---

## Key Template

```text
queue = {root}

while queue is not empty:
    size = queue.size()

    for i = 0 to size - 1:
        node = queue.front()
        queue.pop()

        add children

        if i == size - 1:
            add node to answer
```

For left view:

```text
if i == 0
```

---

## Code

```cpp
class Solution {
public:
    vector<int> rightSideView(TreeNode* root) {
        vector<int> ans;

        if (root == NULL) {
            return ans;
        }

        queue<TreeNode*> q;
        q.push(root);

        while (!q.empty()) {
            int size = q.size();

            for (int i = 0; i < size; i++) {
                TreeNode* node = q.front();
                q.pop();

                if (node->left != NULL) {
                    q.push(node->left);
                }

                if (node->right != NULL) {
                    q.push(node->right);
                }

                if (i == size - 1) {
                    ans.push_back(node->val);
                }
            }
        }

        return ans;
    }
};
```

### Left Side View — Code Change

Only change:

```cpp
if (i == size - 1) {
    ans.push_back(node->val);
}
```

to:

```cpp
if (i == 0) {
    ans.push_back(node->val);
}
```

No other change is required.
The change is based on what BFS order gives us within each level.
From the left, we see the leftmost node of each level.