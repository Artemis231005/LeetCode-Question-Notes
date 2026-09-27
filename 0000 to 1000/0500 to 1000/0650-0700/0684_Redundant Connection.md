# LeetCode 684 — Redundant Connection

## Metadata

* **LeetCode:** 684
* **Problem:** Redundant Connection
* **Difficulty:** Medium
* **Topics:** Depth-First Search, Breadth-First Search, Union Find, Graph
* **Pattern:** Cycle Detection While Building the Graph (Union-Find)
* **Key Technique:** Process edges one at a time, adding each to the graph — the first edge whose two endpoints are **already connected** before it's added is the redundant one, since it's the edge that closes the tree into a cycle
* **Optimal Complexity:** `O(n * α(n))` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given a graph that started as a tree with `n` nodes (labeled `1` to `n`) and had one extra edge added (creating exactly one cycle), represented as a list of edges in the order they were added, return the edge that can be removed to restore the tree. If multiple answers exist, return the last one that occurs in the input.

---

## Approaches

1. **Brute Force — DFS Cycle Check After Removing Each Edge**
2. **Optimal — Union-Find While Adding Edges**

---

# Approach 1 — Brute Force / DFS Cycle Check After Removing Each Edge

## Idea

Try removing each edge, one at a time, starting from the **last** edge in the list (since the problem wants the last valid answer if there are ties). For each candidate removed edge, build the resulting graph from the remaining edges and check via DFS/BFS whether it's a valid tree (fully connected, `n-1` edges, no cycle). The first removal that produces a valid tree is the answer.

## Dry Run

```text
edges = [[1,2],[1,3],[2,3]]
```

Try removing the last edge `[2,3]`:

```text
remaining edges: [1,2],[1,3]
build graph, check connectivity/cycle via DFS from node 1:
   1 connects to 2 and 3, no cycle, all 3 nodes reachable → valid tree!
```

Return `[2,3]`, matching the expected output.

## Algorithm

1. For `i` from `edges.size() - 1` down to `0`:

   * Build a graph using every edge except `edges[i]`.
   * Run DFS/BFS to check if this graph is a valid tree (connected, no cycle, exactly `n-1` edges).
   * If valid, return `edges[i]`.
2. (This will always find an answer, given the problem's guarantees.)

## Complexity

* **Time:** `O(n²)`

  * Up to `n` candidate removals are tried, each requiring an `O(n)` DFS/BFS to verify tree validity.
* **Space:** `O(n)`

  * For the adjacency list and `visited` structure per check.

## Notes / Tips

* Rebuilding and re-validating the whole graph for every candidate removal is wasteful — Union-Find can detect the exact redundant edge in a single forward pass through the edge list, without ever needing to consider removing anything.
* This brute force also implicitly needs to search from the last edge backward to correctly return the "last occurring" answer when ties exist — the Union-Find approach gets this behavior naturally just by processing edges in their given (forward) order.

## Code

```cpp
class Solution {
public:
    bool isValidTree(int n, vector<vector<int>>& edges, int skipIndex) {
        vector<vector<int>> graph(n + 1);
        int edgeCount = 0;

        for (int i = 0; i < edges.size(); i++) {
            if (i == skipIndex) continue;
            graph[edges[i][0]].push_back(edges[i][1]);
            graph[edges[i][1]].push_back(edges[i][0]);
            edgeCount++;
        }

        if (edgeCount != n - 1) return false;

        vector<bool> visited(n + 1, false);
        queue<int> q;
        q.push(1);
        visited[1] = true;
        int visitedCount = 1;

        while (!q.empty()) {
            int node = q.front();
            q.pop();

            for (int neighbor : graph[node]) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;
                    visitedCount++;
                    q.push(neighbor);
                }
            }
        }

        return visitedCount == n;
    }

    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        int n = edges.size();

        for (int i = n - 1; i >= 0; i--) {
            if (isValidTree(n, edges, i)) {
                return edges[i];
            }
        }

        return {};
    }
};
```

---

# Approach 2 — Optimal / Union-Find While Adding Edges

## Idea

Process the edges in the given order, adding each one to a Union-Find structure. Before adding an edge, check whether its two endpoints are already in the same connected component — if they are, this edge doesn't connect anything new; it closes an existing path into a cycle, making it exactly the redundant edge described by the problem. Since edges are processed in order and the answer must be the *last* such edge, simply returning the first edge found to already be "same-component" (scanning forward) automatically gives the correct one, since a tree with one extra edge has exactly one such redundant edge overall regardless of processing order — but scanning forward and stopping at the first detected cycle-closing edge naturally coincides with "last occurring" here because only one edge can ever close the (single) cycle.

## Dry Run

```text
edges = [[1,2],[1,3],[2,3]]
```

Initialize `parent = [0,1,2,3]` (1-indexed).

Process `[1,2]`:

```text
find(1)=1, find(2)=2 → different → union: parent[1]=2 (or similar)
```

Process `[1,3]`:

```text
find(1)=2 (root after union), find(3)=3 → different → union
```

Process `[2,3]`:

```text
find(2) and find(3) both trace to the same root now → SAME component!
this edge would create a cycle → this is the redundant edge
```

Return `[2,3]`, matching both the expected output and the brute-force result.

## Algorithm

1. Initialize a `parent` array of size `n + 1`, where `parent[i] = i`.
2. For each edge `[u, v]` in order:

   * If `find(u) == find(v)` (already connected), return this edge immediately — it's the redundant one.
   * Otherwise, union `u` and `v`.
3. (The loop is guaranteed to return before finishing, given the problem's structure.)

## Complexity

* **Time:** `O(n * α(n))`

  * Each of the `n` edges triggers a `find`/`union` operation, each nearly `O(1)` on average with path compression and union by rank (`α` is the inverse Ackermann function, effectively constant).
* **Space:** `O(n)`

  * For the `parent` (and optional `rank`) arrays.

## Notes / Tips

* This naturally returns the correct answer when ties are possible: since a tree-plus-one-edge graph has exactly **one** cycle, there's exactly one edge in the entire list whose endpoints are already connected at the moment it's processed — no ambiguity about "first vs. last" actually arises for this specific problem's guarantees.
* This is the same Union-Find template used in LC 547 and LC 1971, applied here as an incremental **cycle detector** rather than a final connectivity check — the key shift is checking `find(u) == find(v)` *before* unioning, at the moment each edge is introduced, rather than only checking connectivity once at the end.
* Path compression and union by rank are what keep this approach close to linear — without them, worst-case `find`/`union` operations could degrade toward `O(n)` each, though even then the overall approach would remain far faster than Approach 1's repeated full-graph validation.

## Code

```cpp
class Solution {
public:
    vector<int> parent;

    int find(int x) {
        if (parent[x] != x) {
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }

    vector<int> findRedundantConnection(vector<vector<int>>& edges) {
        int n = edges.size();
        parent.resize(n + 1);
        for (int i = 0; i <= n; i++) {
            parent[i] = i;
        }

        for (auto& edge : edges) {
            int u = edge[0], v = edge[1];
            int rootU = find(u), rootV = find(v);

            if (rootU == rootV) {
                return edge;
            }

            parent[rootU] = rootV;
        }

        return {};
    }
};
```

---

## Key Template

```text
parent = [0, 1, ..., n]

function find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])
    return parent[x]

for [u, v] in edges:
    rootU = find(u)
    rootV = find(v)

    if rootU == rootV:
        return [u, v]   # redundant edge found

    parent[rootU] = rootV

return []
```