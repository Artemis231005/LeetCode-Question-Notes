# 257. Binary Tree Paths

## Metadata

* **Topic:** Binary Tree, DFS
* **Difficulty:** Easy
* **Pattern:** Root-to-Leaf DFS
* **Key Pattern:** Build the path while traversing; save it when a leaf is reached.

---

## Idea

We need to return all paths from the **root to every leaf**.

A node is a leaf when:

```text
node->left == NULL && node->right == NULL
```

Use DFS and maintain the current path.

At each node:

1. Add its value to the path.
2. If it is a leaf, save the path.
3. Otherwise, continue to its children.

Example:

```text
        1
       / \
      2   3
       \
        5
```

Paths:

```text
1 → 2 → 5
1 → 3
```

Result:

```text
["1->2->5", "1->3"]
```

---

## Dry Run

For:

```text
        1
       / \
      2   3
       \
        5
```

Start:

```text
path = "1"
```

Go left:

```text
path = "1->2"
```

Go right:

```text
path = "1->2->5"
```

`5` is a leaf, so save:

```text
"1->2->5"
```

Backtrack and explore the right subtree:

```text
"1->3"
```

`3` is a leaf, so save it.

Final:

```text
["1->2->5", "1->3"]
```

---

## Algorithm

1. If `root == NULL`, return an empty result.
2. Add `root->val` to the current path.
3. If the node is a leaf:

   * Add the path to the answer.
4. Otherwise:

   * Recursively process the left child.
   * Recursively process the right child.
5. Backtrack after processing the node.

---

## Complexity

* **Time:** `O(n * h)` in the worst case due to path construction/copying.
* **Space:** `O(h)` recursion/path space, excluding the output.

---

## Code

```cpp
class Solution {
public:
    vector<string> ans;

    void dfs(TreeNode* root, string path) {
        if (root == NULL) {
            return;
        }

        if (!path.empty()) {
            path += "->";
        }

        path += to_string(root->val);

        if (root->left == NULL && root->right == NULL) {
            ans.push_back(path);
            return;
        }

        dfs(root->left, path);
        dfs(root->right, path);
    }

    vector<string> binaryTreePaths(TreeNode* root) {
        dfs(root, "");
        return ans;
    }
};
```

---

## Notes / Tips

* Only **leaf nodes** produce an answer.
* The path should contain:

  ```text
  root → ... → leaf
  ```
* Passing `path` by value makes backtracking easy because every recursive call gets its own copy.
* If using a shared string, you must manually append and remove values while backtracking.

### Key Pattern

```text
DFS
 ↓
Build path
 ↓
Leaf?
 ↓
Save path
```

---

## Key Template

```text
dfs(node, path):

    if node == NULL:
        return

    add node to path

    if node is leaf:
        save path
        return

    dfs(node->left, path)
    dfs(node->right, path)
```
