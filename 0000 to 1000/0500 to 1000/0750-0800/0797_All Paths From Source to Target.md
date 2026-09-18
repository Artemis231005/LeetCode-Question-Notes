# LeetCode 797 — All Paths From Source to Target

## Metadata

* **LeetCode:** 797
* **Problem:** All Paths From Source to Target
* **Difficulty:** Medium
* **Topics:** Backtracking, Depth-First Search, Breadth-First Search, Graph
* **Pattern:** DFS/BFS Path Enumeration on a DAG
* **Key Technique:** Since the graph is a **directed acyclic graph**, no cycle can ever trap a traversal — every path from `source` simply extends until it reaches `target` or a dead end, so plain DFS backtracking (or BFS carrying the full path) safely enumerates every path without needing a visited-set at all
* **Optimal Complexity:** `O(2^V * V)` Time, `O(V)` Auxiliary Space (excluding the output)

---

## Problem Statement

Given a directed acyclic graph (DAG) of `n` nodes labeled `0` to `n-1`, represented as an adjacency list `graph`, return all possible paths from node `0` to node `n-1`, in any order.

---

## Approaches

1. **DFS Backtracking**
2. **BFS Carrying the Full Path**

---

# Approach 1 — DFS Backtracking

## Idea

Starting from node `0`, explore every outgoing edge recursively, appending each visited node to a running `path`. Whenever the path reaches `n-1` (the target), record a copy of it. After exploring a branch, backtrack by removing the last node before trying the next neighbor — this naturally explores every distinct path without needing a visited-set, since the DAG guarantees no cycles can cause infinite recursion.

## Dry Run

```text
graph = [[1,2],[3],[3],[]]   (n = 4, source = 0, target = 3)
```

Start DFS at `0`, `path = [0]`:

```text
neighbor 1: path = [0,1]
    neighbor 3: path = [0,1,3] → reached target! record [0,1,3]
    backtrack: path = [0,1]
backtrack: path = [0]

neighbor 2: path = [0,2]
    neighbor 3: path = [0,2,3] → reached target! record [0,2,3]
    backtrack: path = [0,2]
backtrack: path = [0]
```

Final paths: `[0,1,3]` and `[0,2,3]`.

## Algorithm

1. Initialize an empty `path = [0]` and an empty `results` list.
2. Define a recursive `dfs(node)`:

   * If `node == n - 1` (target), record a copy of `path` into `results` and return.
   * For each neighbor of `node`:

     * Append `neighbor` to `path`.
     * Recursively call `dfs(neighbor)`.
     * Remove the last element from `path` (backtrack).
3. Call `dfs(0)`.
4. Return `results`.

## Complexity

* **Time:** `O(2^V * V)`

  * In the densest possible DAG, the number of distinct paths from source to target can be exponential in the number of nodes, and each path takes `O(V)` to copy into the results.
* **Space:** `O(V)` auxiliary (excluding the output)

  * For the recursion stack and the current `path`, both bounded by the number of nodes.

## Notes / Tips

* No `visited` array is needed here since the graph is guaranteed acyclic (edges always point "forward" in some topological sense), so DFS can never loop back on itself.
* Recording a **copy** of `path` (not a reference to the same mutable list) when the target is reached is essential — without copying, all recorded "paths" would end up pointing to the same list, which then gets mutated by subsequent backtracking.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> results;
    vector<int> path;
    vector<vector<int>> graph;
    int target;

    void dfs(int node) {
        if (node == target) {
            results.push_back(path);
            return;
        }

        for (int neighbor : graph[node]) {
            path.push_back(neighbor);
            dfs(neighbor);
            path.pop_back();
        }
    }

    vector<vector<int>> allPathsSourceTarget(vector<vector<int>>& graph) {
        this->graph = graph;
        target = graph.size() - 1;

        path.push_back(0);
        dfs(0);

        return results;
    }
};
```

---

# Approach 2 — BFS Carrying the Full Path

## Idea

Instead of recursion, use an explicit queue where each queued item is an **entire path so far** (not just a single node). Starting with the path `[0]`, repeatedly dequeue a path, look at its last node, and for each of that node's neighbors, enqueue a new path extending the current one by that neighbor. Whenever a dequeued path's last node is the target, record it.

## Dry Run

```text
graph = [[1,2],[3],[3],[]]   (n = 4, source = 0, target = 3)
```

Queue starts with `[[0]]`.

Dequeue `[0]`: last node `0`, not target. Extend by neighbors `1` and `2`:

```text
enqueue [0,1], [0,2]
queue = [[0,1],[0,2]]
```

Dequeue `[0,1]`: last node `1`, not target. Extend by neighbor `3`:

```text
enqueue [0,1,3]
queue = [[0,2],[0,1,3]]
```

Dequeue `[0,2]`: last node `2`, not target. Extend by neighbor `3`:

```text
enqueue [0,2,3]
queue = [[0,1,3],[0,2,3]]
```

Dequeue `[0,1,3]`: last node `3` == target → record `[0,1,3]`.

Dequeue `[0,2,3]`: last node `3` == target → record `[0,2,3]`.

Final paths: `[0,1,3]` and `[0,2,3]`, matching the DFS result.

## Algorithm

1. Initialize a queue with a single path `[0]`, and an empty `results` list.
2. While the queue is non-empty:

   * Dequeue a path.
   * If its last node equals `n - 1` (target), append it to `results`.
   * Otherwise, for each neighbor of the last node, enqueue a new path (the current path plus that neighbor appended).
3. Return `results`.

## Complexity

* **Time:** `O(2^V * V)`

  * Same worst-case bound as DFS — the number of paths can be exponential, and each queued path can be up to `O(V)` long to copy/extend.
* **Space:** `O(2^V * V)`

  * The queue can simultaneously hold many partial paths, each up to `O(V)` long, unlike DFS's `O(V)` auxiliary space which only ever holds one path at a time (via backtracking) plus the recursion stack.

## Notes / Tips

* BFS here trades DFS's low auxiliary space for a more straightforward, iterative structure with no recursion or explicit backtracking step. Each queued item is self-contained, so there's no need to "undo" anything.
* Because this problem asks for **all** paths (not the shortest one), BFS doesn't offer its usual advantage of finding the answer with the fewest edges first; both DFS and BFS end up doing comparable total work here, making DFS's lower space usage the more practical choice for this specific problem.
* Copying and extending an entire path for every neighbor (`current path + [neighbor]`) is what makes BFS's space usage grow faster than DFS's — DFS only ever needs one working copy of the path at a time, mutated and restored via backtracking, while BFS must keep every in-flight partial path alive in the queue simultaneously.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> allPathsSourceTarget(vector<vector<int>>& graph) {
        int target = graph.size() - 1;
        queue<vector<int>> q;
        q.push({0});

        vector<vector<int>> results;

        while (!q.empty()) {
            vector<int> path = q.front();
            q.pop();

            int lastNode = path.back();

            if (lastNode == target) {
                results.push_back(path);
                continue;
            }

            for (int neighbor : graph[lastNode]) {
                vector<int> newPath = path;
                newPath.push_back(neighbor);
                q.push(newPath);
            }
        }

        return results;
    }
};
```

---

## Key Template

```text
# DFS backtracking
path = [0]
results = []

function dfs(node):
    if node == target:
        results.append(copy of path)
        return

    for neighbor in graph[node]:
        path.append(neighbor)
        dfs(neighbor)
        path.pop()

dfs(0)
return results
```