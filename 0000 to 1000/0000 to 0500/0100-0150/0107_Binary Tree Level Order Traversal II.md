# LeetCode 107 — Binary Tree Level Order Traversal II

## Metadata

* **LeetCode:** 107
* **Problem:** Binary Tree Level Order Traversal II
* **Difficulty:** Medium
* **Topics:** Tree, Breadth-First Search, Binary Tree
* **Pattern:** Level-Order BFS with Reversed Output
* **Key Technique:** Perform a standard level-by-level BFS top to bottom, then reverse the collected levels at the end (or insert each new level at the front) to get bottom-up order
* **Optimal Complexity:** `O(n)` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given the root of a binary tree, return the bottom-up level order traversal of its nodes' values (i.e. from the leaf level up to the root level, left to right within each level).

---

## Approaches

1. **Brute Force — DFS Collecting Nodes by Depth, Then Reverse**
2. **Optimal — BFS Level by Level, Then Reverse**

---

# Approach 1 — Brute Force / DFS Collecting Nodes by Depth, Then Reverse

## Idea

Use DFS to visit every node while tracking its depth. Store each node's value in a list indexed by depth (using a hash map or a list-of-lists that grows as deeper levels are discovered). Once the traversal is complete, reverse the order of the depth-indexed lists to get bottom-up order.

## Dry Run

```text
        3
       / \
      9  20
        /  \
       15   7
```

DFS from root, tracking depth:

```text
visit 3 at depth 0 → levels[0] = [3]
visit 9 at depth 1 → levels[1] = [9]
visit 20 at depth 1 → levels[1] = [9, 20]
visit 15 at depth 2 → levels[2] = [15]
visit 7 at depth 2 → levels[2] = [15, 7]
```

Reverse the list of levels:

```text
[[15,7], [9,20], [3]]
```

## Algorithm

1. Define a recursive `dfs(node, depth, levels)`:

   * If `node` is null, return.
   * If `levels` doesn't yet have an entry for `depth`, create one.
   * Append `node->val` to `levels[depth]`.
   * Recurse into `node->left` and `node->right` with `depth + 1`.
2. Call `dfs(root, 0, levels)`.
3. Reverse the order of `levels`.
4. Return `levels`.

## Complexity

* **Time:** `O(n)`

  * Every node is visited exactly once.
* **Space:** `O(n)`

  * For the depth-indexed levels structure and the DFS recursion stack (up to `O(h)` deep, where `h` is tree height).

## Notes / Tips

* DFS naturally visits nodes depth-first rather than level-by-level, so appending to the correct depth bucket as nodes are encountered (potentially out of left-to-right order across different subtrees) requires care. This works correctly here because DFS still visits `node->left` before `node->right` at each step, preserving left-to-right order *within* each depth bucket even though depths are filled in an interleaved order overall.
* This approach is a valid alternative to BFS, but BFS (Approach 2) is more natural for level-order problems since it processes nodes in true level-by-level order without needing an explicit depth-tracking structure.

## Code

```cpp
class Solution {
public:
    void dfs(TreeNode* node, int depth, vector<vector<int>>& levels) {
        if (!node) return;

        if (depth == levels.size()) {
            levels.push_back({});
        }

        levels[depth].push_back(node->val);

        dfs(node->left, depth + 1, levels);
        dfs(node->right, depth + 1, levels);
    }

    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        vector<vector<int>> levels;
        dfs(root, 0, levels);

        reverse(levels.begin(), levels.end());
        return levels;
    }
};
```

---

# Approach 2 — Optimal / BFS Level by Level, Then Reverse

## Idea

Run a standard top-down level-order BFS: process the queue one full level at a time, collecting each level's values into its own list. Since BFS naturally produces levels from top to bottom, simply reverse the final list of levels (or insert each new level at the front as it's completed) to get the bottom-up order the problem asks for.

## Dry Run

```text
        3
       / \
      9  20
        /  \
       15   7
```

BFS starting with queue `[3]`:

Level 1: process `3`:

```text
level = [3]
enqueue children: 9, 20
queue = [9, 20]
```

Level 2: process `9` and `20`:

```text
level = [9, 20]
enqueue children of 20: 15, 7 (9 has no children)
queue = [15, 7]
```

Level 3: process `15` and `7`:

```text
level = [15, 7]
no children to enqueue
queue = []
```

Collected levels (top to bottom): `[[3], [9,20], [15,7]]`.

Reverse: `[[15,7], [9,20], [3]]`.

## Algorithm

1. If `root` is null, return an empty list.
2. Initialize a queue with `root`, and an empty list `result`.
3. While the queue is non-empty:

   * Record the current level's size.
   * Initialize an empty list `level`.
   * For each node in this level: pop it, append its value to `level`, and enqueue its non-null children.
   * Append `level` to `result`.
4. Reverse `result`.
5. Return `result`.

## Complexity

* **Time:** `O(n)`

  * Every node is enqueued and processed exactly once.
* **Space:** `O(n)`

  * For the BFS queue and the collected `result` levels, both bounded by the total number of nodes.

## Notes / Tips

* This is just the standard level-order BFS template (as in LC 102 — Binary Tree Level Order Traversal) with a single extra step (`reverse(result)`) tacked on at the end — recognizing that the only difference from the "normal" top-down version is the final output order avoids overcomplicating the traversal logic itself.
* An equally valid alternative to reversing at the end is inserting each newly completed level at the **front** of `result` as it's built (e.g. using a deque or `insert(result.begin(), level)`) — this avoids a separate reverse pass but can cost more per insertion depending on the underlying container; reversing once at the end is simpler and just as efficient overall.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        vector<vector<int>> result;
        if (!root) {
            return result;
        }

        queue<TreeNode*> q;
        q.push(root);

        while (!q.empty()) {
            int size = q.size();
            vector<int> level;

            for (int i = 0; i < size; i++) {
                TreeNode* node = q.front();
                q.pop();

                level.push_back(node->val);

                if (node->left) q.push(node->left);
                if (node->right) q.push(node->right);
            }

            result.push_back(level);
        }

        reverse(result.begin(), result.end());
        return result;
    }
};
```

---

## Key Template

```text
if root is null: return []

queue = [root]
result = []

while queue not empty:
    levelSize = queue.size()
    level = []

    for i in 0..levelSize-1:
        node = queue.pop()
        level.append(node.val)

        if node.left: queue.push(node.left)
        if node.right: queue.push(node.right)

    result.append(level)

reverse(result)
return result
```