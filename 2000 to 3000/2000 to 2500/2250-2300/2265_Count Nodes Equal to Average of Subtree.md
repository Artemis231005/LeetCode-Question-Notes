# LeetCode 2265 — Count Nodes Equal to Average of Subtree

## Metadata

* **LeetCode:** 2265
* **Problem:** Count Nodes Equal to Average of Subtree
* **Difficulty:** Medium
* **Topics:** Tree, Depth-First Search, Binary Tree
* **Pattern:** Post-Order DFS with Aggregated Return Values
* **Key Technique:** Recurse into both children first to get each subtree's sum and node count, combine them for the current node, then check the average — all in a single bottom-up pass
* **Optimal Complexity:** `O(n)` Time, `O(h)` Auxiliary Space

---

## Problem Statement

Given the root of a binary tree, return the number of nodes where the node's value equals the **average** (rounded down via integer division) of all values in its subtree (including itself).

---

## Approaches

1. **Brute Force — Recompute Each Subtree's Sum and Count Separately**
2. **Optimal — Single Post-Order DFS Returning Sum and Count**

---

# Approach 1 — Brute Force / Recompute Each Subtree's Sum and Count Separately

## Idea

For every node in the tree, run a full separate traversal of its subtree to compute the sum of all values and the number of nodes, then check if the node's value equals `sum / count`. Do this independently for every node in the tree.

## Dry Run

```text
        4
       / \
      8   5
     / \   \
    0   1   6
```

For node `4` (root): traverse its entire subtree:

```text
sum = 4+8+5+0+1+6 = 24, count = 6
average = 24/6 = 4 → matches node value 4 → counts as valid
```

For node `8`: traverse its subtree:

```text
sum = 8+0+1 = 9, count = 3
average = 9/3 = 3 → does not match node value 8 → not valid
```

Continue this full re-traversal for every node (`5`, `0`, `1`, `6`) independently to get the final count.

## Algorithm

1. Define a helper `subtreeSumAndCount(node)` that traverses `node`'s entire subtree and returns its total sum and node count.
2. For each node in the tree (via any traversal):

   * Call `subtreeSumAndCount(node)` to get `sum` and `count` for that node's subtree.
   * If `node->val == sum / count`, increment the answer.
3. Return the answer.

## Complexity

* **Time:** `O(n²)`

  * For each of the `n` nodes, a full subtree traversal can visit up to `n` nodes in the worst case (e.g. a skewed tree).
* **Space:** `O(h)`

  * For the recursion stack of each individual subtree traversal, where `h` is the tree height — not counting the repeated traversal cost itself.

## Notes / Tips

* Every subtree's sum and count get recomputed from scratch for every node, even though a parent's subtree sum is just its own value plus its children's subtree sums — massive redundant work.
* This redundancy is exactly what a single post-order traversal (computing and reusing child results directly) eliminates in Approach 2.

## Code

```cpp
class Solution {
public:
    int answer = 0;

    pair<int, int> subtreeSumAndCount(TreeNode* node) {
        if (!node) {
            return {0, 0};
        }

        auto left = subtreeSumAndCount(node->left);
        auto right = subtreeSumAndCount(node->right);

        int sum = node->val + left.first + right.first;
        int count = 1 + left.second + right.second;

        return {sum, count};
    }

    int averageOfSubtree(TreeNode* root) {
        if (!root) {
            return 0;
        }

        // Recompute independently for every node (redundant on purpose, for brute force)
        function<void(TreeNode*)> visit = [&](TreeNode* node) {
            if (!node) return;

            auto [sum, count] = subtreeSumAndCount(node);
            if (node->val == sum / count) {
                answer++;
            }

            visit(node->left);
            visit(node->right);
        };

        visit(root);
        return answer;
    }
};
```

---

# Approach 2 — Optimal / Single Post-Order DFS Returning Sum and Count

## Idea

Since a node's subtree sum and count are exactly its own value plus the sums/counts of its two children's subtrees, a single post-order traversal can compute every subtree's sum and count in one pass — visit both children first, combine their results with the current node's value, and check the average immediately using values already computed (no separate re-traversal needed).

## Dry Run

```text
        4
       / \
      8   5
     / \   \
    0   1   6
```

Post-order visits leaves first:

```text
node 0: sum=0, count=1, avg=0/1=0 → matches value 0 → answer=1
node 1: sum=1, count=1, avg=1/1=1 → matches value 1 → answer=2
node 8: sum = 8 + (sum from 0) + (sum from 1) = 8+0+1=9
        count = 1 + 1 + 1 = 3
        avg = 9/3 = 3 → does not match value 8 → answer stays 2
node 6: sum=6, count=1, avg=6/1=6 → matches value 6 → answer=3
node 5: sum = 5 + 0(no left) + 6(from right) = 11
        count = 1 + 0 + 1 = 2
        avg = 11/2 = 5 (integer division) → matches value 5 → answer=4
node 4 (root): sum = 4 + 9(from 8) + 11(from 5) = 24
        count = 1 + 3 + 2 = 6
        avg = 24/6 = 4 → matches value 4 → answer=5
```

Final answer: `5`.

## Algorithm

1. Define a recursive helper `dfs(node)` that returns `{sum, count}` for `node`'s subtree.
2. Base case: if `node` is `null`, return `{0, 0}`.
3. Recursively call `dfs` on the left and right children.
4. Compute `sum = node->val + leftSum + rightSum` and `count = 1 + leftCount + rightCount`.
5. If `node->val == sum / count`, increment a shared answer counter.
6. Return `{sum, count}` to the parent call.
7. Return the final answer after the traversal completes.

## Complexity

* **Time:** `O(n)`

  * Each node is visited exactly once, doing constant work per node (combining two child results).
* **Space:** `O(h)`

  * For the recursion call stack, where `h` is the tree's height (`O(log n)` for a balanced tree, `O(n)` for a skewed one).

## Notes / Tips

* This is the standard "post-order DFS with aggregated return values" template — any problem where a node's answer depends on totals from its entire subtree (sum, count, height, diameter, etc.) benefits from computing and returning that aggregate directly during a single bottom-up pass, rather than re-traversing per node.
* Returning a `pair<sum, count>` (or a small struct) from each recursive call is what lets the parent combine children's results in `O(1)` instead of needing a second traversal — the same technique generalizes to returning more than two aggregated values when a problem needs them (e.g. min, max, and sum together).

## Code

```cpp
class Solution {
public:
    int answer = 0;

    pair<int, int> dfs(TreeNode* node) {
        if (!node) {
            return {0, 0};
        }

        auto left = dfs(node->left);
        auto right = dfs(node->right);

        int sum = node->val + left.first + right.first;
        int count = 1 + left.second + right.second;

        if (node->val == sum / count) {
            answer++;
        }

        return {sum, count};
    }

    int averageOfSubtree(TreeNode* root) {
        dfs(root);
        return answer;
    }
};
```

---

## Key Template

```text
answer = 0

function dfs(node):
    if node is null: return (sum=0, count=0)

    leftSum, leftCount = dfs(node.left)
    rightSum, rightCount = dfs(node.right)

    sum = node.val + leftSum + rightSum
    count = 1 + leftCount + rightCount

    if node.val == sum / count:
        answer += 1

    return (sum, count)

dfs(root)
return answer
```