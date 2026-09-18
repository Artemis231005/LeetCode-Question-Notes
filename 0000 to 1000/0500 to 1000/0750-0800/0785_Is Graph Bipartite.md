# LeetCode 785 — Is Graph Bipartite?

## Metadata

* **LeetCode:** 785
* **Problem:** Is Graph Bipartite?
* **Difficulty:** Medium
* **Topics:** Depth-First Search, Breadth-First Search, Union Find, Graph
* **Pattern:** Two-Coloring via Graph Traversal
* **Key Technique:** Try to color every node one of two colors such that every edge connects differently colored nodes — a coloring conflict during traversal means the graph isn't bipartite
* **Optimal Complexity:** `O(V + E)` Time, `O(V)` Auxiliary Space

---

## Problem Statement

Given an undirected graph represented as an adjacency list `graph`, return `true` if the graph is bipartite — meaning its nodes can be split into two disjoint sets such that every edge connects a node from one set to a node in the other.

---

## Approaches

1. **Brute Force — Try All 2-Colorings via Backtracking**
2. **Optimal — BFS/DFS Two-Coloring**

---

# Approach 1 — Brute Force / Try All 2-Colorings via Backtracking

## Idea

Attempt to assign each node one of two colors, one node at a time, backtracking whenever an assignment creates a conflict with an already-colored neighbor. Explore all possible color assignments (not just the ones forced by traversal order) to determine if any valid 2-coloring exists.

## Dry Run

```text
graph = [[1,3],[0,2],[1,3],[0,2]]
```

Try coloring node `0` as color `A`:

```text
node 0 = A
```

Try coloring node `1` (neighbor of 0, must differ):

```text
node 1 = B (forced, since it's adjacent to A)
```

Try coloring node `2` (neighbor of 1, must differ from B):

```text
node 2 = A
```

Try coloring node `3` (neighbor of both 0 and 2, both colored A):

```text
node 3 must differ from A → node 3 = B
check: node 3 is also adjacent to node 2 (A) → consistent, B != A → ok
```

All nodes colored without conflict → return `true`.

## Algorithm

1. Initialize a `color` array of size `n`, all uncolored (e.g. `0`).
2. Define a recursive `tryColor(node, c)`:

   * If `node` is already colored: return `color[node] == c`.
   * Assign `color[node] = c`.
   * For each neighbor of `node`, recursively try `tryColor(neighbor, -c)`; if any fails, backtrack (`color[node] = 0`) and return `false`.
   * If all neighbors succeed, return `true`.
3. For each uncolored node (handles disconnected components), attempt `tryColor(node, 1)`. If any fails, return `false`.
4. Return `true` if all components are successfully colored.

## Complexity

* **Time:** `O(2^V)` in the worst case

  * Backtracking with undoing color assignments can, in pathological cases, re-explore many coloring possibilities before settling — though in practice most inputs resolve quickly since coloring is almost always forced once a starting color is picked.
* **Space:** `O(V)`

  * For the `color` array and the recursion stack.

## Notes / Tips

* For this specific problem, coloring is actually never ambiguous once a component's starting node is colored — every neighbor's color is *forced* to be the opposite, so true backtracking (trying alternate colorings) never actually helps or is needed. This makes the "brute force" framing here mostly about being less careful/optimized in implementation rather than a fundamentally different, slower algorithm.

## Code

```cpp
class Solution {
public:
    vector<int> color;
    vector<vector<int>> graph;

    bool tryColor(int node, int c) {
        if (color[node] != 0) {
            return color[node] == c;
        }

        color[node] = c;

        for (int neighbor : graph[node]) {
            if (!tryColor(neighbor, -c)) {
                return false;
            }
        }

        return true;
    }

    bool isBipartite(vector<vector<int>>& graph) {
        this->graph = graph;
        int n = graph.size();
        color.assign(n, 0);

        for (int i = 0; i < n; i++) {
            if (color[i] == 0) {
                if (!tryColor(i, 1)) {
                    return false;
                }
            }
        }

        return true;
    }
};
```

---

# Approach 2 — Optimal / BFS/DFS Two-Coloring

## Idea

Traverse the graph (via BFS or DFS), assigning each unvisited node the opposite color of the node that discovered it. Since a valid bipartition forces every neighbor's color, there's no need for backtracking — if a neighbor is ever found already colored the **same** as the current node, the graph isn't bipartite. Process every connected component separately, since the graph may not be fully connected.

## Dry Run

```text
graph = [[1,3],[0,2],[1,3],[0,2]]
```

Start BFS from node `0`, color `= 1`:

```text
color = [1, 0, 0, 0]
queue = [0]
```

Process `0`: neighbors `1` and `3`, both uncolored → color them `-1`:

```text
color = [1, -1, 0, -1]
queue = [1, 3]
```

Process `1`: neighbors `0` (colored `1`, opposite of `1`'s `-1` → consistent) and `2` (uncolored → color `1`):

```text
color = [1, -1, 1, -1]
queue = [3, 2]
```

Process `3`: neighbors `0` (colored `1`, opposite of `3`'s `-1` → consistent) and `2` (colored `1`, opposite of `3`'s `-1` → consistent):

```text
no new nodes to color
```

Process `2`: neighbors `1` (colored `-1`, opposite of `2`'s `1` → consistent) and `3` (colored `-1`, opposite of `2`'s `1` → consistent):

```text
no conflicts found
```

All nodes processed without conflict → return `true`.

## Algorithm

1. Initialize a `color` array of size `n`, all `0` (uncolored).
2. For each node `i` from `0` to `n-1` (to cover disconnected components):

   * If `color[i] != 0`, skip (already processed).
   * Otherwise, start BFS (or DFS) from `i`:

     * Color `i` with `1`.
     * For each node popped from the queue, examine its neighbors:

       * If a neighbor is uncolored, assign it the opposite color and enqueue it.
       * If a neighbor is already colored the **same** as the current node, return `false` immediately (conflict found).
3. If every component is processed without conflict, return `true`.

## Complexity

* **Time:** `O(V + E)`

  * Every node is colored and enqueued once, and every edge is examined at most twice (once from each endpoint).
* **Space:** `O(V)`

  * For the `color` array and the BFS queue (or DFS recursion stack).

## Notes / Tips

* Coloring with `1` and `-1` (rather than two arbitrary labels) is a convenient trick — flipping to the opposite color is just negation, and checking "same color" is a simple equality check.
* Iterating over every node as a potential BFS/DFS start (not just node `0`) is essential — a graph can have multiple disconnected components, each of which must be independently checked for a valid 2-coloring, and skipping this would silently ignore components that are only reachable by starting elsewhere.
* This two-coloring technique is the standard test for bipartiteness in any graph representation — the same idea works identically whether traversal is done via BFS, DFS, or even Union-Find (grouping each node with the "opposite" set of its neighbors and checking for a same-set conflict).

## Code

```cpp
class Solution {
public:
    bool isBipartite(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<int> color(n, 0);

        for (int start = 0; start < n; start++) {
            if (color[start] != 0) {
                continue;
            }

            queue<int> q;
            q.push(start);
            color[start] = 1;

            while (!q.empty()) {
                int node = q.front();
                q.pop();

                for (int neighbor : graph[node]) {
                    if (color[neighbor] == 0) {
                        color[neighbor] = -color[node];
                        q.push(neighbor);
                    } else if (color[neighbor] == color[node]) {
                        return false;
                    }
                }
            }
        }

        return true;
    }
};
```

---

## Key Template

```text
color = array of size n, all 0

for start in 0..n-1:
    if color[start] != 0: continue

    queue = [start]
    color[start] = 1

    while queue not empty:
        node = queue.pop()

        for neighbor in graph[node]:
            if color[neighbor] == 0:
                color[neighbor] = -color[node]
                queue.push(neighbor)
            elif color[neighbor] == color[node]:
                return false

return true
```