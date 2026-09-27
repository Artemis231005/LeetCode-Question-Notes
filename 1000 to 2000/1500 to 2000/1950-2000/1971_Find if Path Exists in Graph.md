# LeetCode 1971 — Find if Path Exists in Graph

## Metadata

* **LeetCode:** 1971
* **Problem:** Find if Path Exists in Graph
* **Difficulty:** Easy
* **Topics:** Depth-First Search, Breadth-First Search, Union Find, Graph
* **Pattern:** Connectivity Check (DFS/BFS or Union-Find)
* **Key Technique:** Treat the edges as an undirected graph and check whether `source` and `destination` end up in the same connected component — either by traversing from `source` and seeing if `destination` is reached, or by union-ing every edge and comparing final group membership
* **Optimal Complexity:** `O(V + E * α(V))` Time, `O(V + E)` Auxiliary Space

---

## Problem Statement

Given `n` vertices labeled `0` to `n-1`, a list of undirected edges, a `source` vertex, and a `destination` vertex, return `true` if there is a valid path from `source` to `destination`.

---

## Approaches

1. **Brute Force — DFS/BFS from Source**
2. **Optimal — Union-Find (Disjoint Set Union)**

---

# Approach 1 — Brute Force / DFS/BFS from Source

## Idea

Build an adjacency list from the edges, then run a standard graph traversal (DFS or BFS) starting from `source`, marking every reachable vertex. Once the traversal finishes, check whether `destination` was among the reachable vertices.

## Dry Run

```text
n = 6, edges = [[0,1],[0,2],[3,5],[5,4],[4,3]], source = 0, destination = 5
```

BFS from `0`:

```text
visit 0 → neighbors: 1, 2
visit 1 → neighbors: 0 (already visited)
visit 2 → neighbors: 0 (already visited)
queue empty, all reachable vertices from 0: {0, 1, 2}
```

`5` is not in `{0, 1, 2}` → return `false`.

## Algorithm

1. Build an adjacency list from `edges` (undirected, so add both directions).
2. Initialize a `visited` array, all `false`.
3. Run BFS (or DFS) from `source`, marking every reachable vertex as visited.
4. Return `visited[destination]`.

## Complexity

* **Time:** `O(V + E)`

  * Every vertex and edge is visited at most once during the traversal.
* **Space:** `O(V + E)`

  * For the adjacency list, the `visited` array, and the BFS queue (or DFS recursion stack).

## Notes / Tips

* This is already an efficient, standard approach for a single connectivity query — "brute force" here is relative only to Union-Find, which can be a more natural fit when the graph structure needs to support many such queries or when edges arrive incrementally.
* For just one `source`/`destination` pair (as this problem asks), a single BFS/DFS is arguably the simplest and most direct solution — Union-Find's advantage becomes more apparent in variants of this problem involving multiple queries.

## Code

```cpp
class Solution {
public:
    bool validPath(int n, vector<vector<int>>& edges, int source, int destination) {
        vector<vector<int>> graph(n);
        for (auto& e : edges) {
            graph[e[0]].push_back(e[1]);
            graph[e[1]].push_back(e[0]);
        }

        vector<bool> visited(n, false);
        queue<int> q;
        q.push(source);
        visited[source] = true;

        while (!q.empty()) {
            int node = q.front();
            q.pop();

            if (node == destination) {
                return true;
            }

            for (int neighbor : graph[node]) {
                if (!visited[neighbor]) {
                    visited[neighbor] = true;
                    q.push(neighbor);
                }
            }
        }

        return visited[destination];
    }
};
```

---

# Approach 2 — Optimal / Union-Find (Disjoint Set Union)

## Idea

Treat each vertex as its own group initially. Process every edge, union-ing the two endpoints' groups together. After processing all edges, `source` and `destination` have a valid path between them exactly when they end up in the same group — checked with a single `find` comparison.

## Dry Run

```text
n = 6, edges = [[0,1],[0,2],[3,5],[5,4],[4,3]], source = 0, destination = 5
```

Initialize `parent = [0,1,2,3,4,5]`.

```text
edge (0,1): union(0,1) → parent = [0,0,2,3,4,5]
edge (0,2): union(0,2) → parent = [0,0,0,3,4,5]
edge (3,5): union(3,5) → parent = [0,0,0,3,3,5] (root of 5 becomes 3)
edge (5,4): union(5,4) → find(5)=3, find(4)=4 → parent[3]=4 (or similar merge)
edge (4,3): union(4,3) → already same root by now
```

Check `find(0)` vs `find(5)`:

```text
find(0) → 0 (or whatever 0's group ended up as)
find(5) → traces through the {3,4,5} group, different from 0's group
```

Different roots → return `false`, matching the BFS result.

## Algorithm

1. Initialize a `parent` array of size `n`, where `parent[i] = i`.
2. For each edge `[u, v]`, union the groups containing `u` and `v` (using path compression and union by rank/size for efficiency).
3. Return whether `find(source) == find(destination)`.

## Complexity

* **Time:** `O(V + E * α(V))`

  * `O(V)` to initialize `parent`, and `O(E)` union operations, each nearly `O(1)` on average thanks to path compression and union by rank (`α` is the inverse Ackermann function, effectively constant).
* **Space:** `O(V)`

  * For the `parent` (and optional `rank`) arrays.

## Notes / Tips

* For a single query (one `source`/`destination` pair, as in this exact problem), Union-Find and BFS/DFS are both essentially `O(V + E)` in practice and neither is meaningfully "better" — Union-Find's real advantage shows up when checking connectivity for **many** pairs across the same graph, or when edges are added incrementally over time, since each new edge only costs a cheap union rather than triggering a fresh traversal.
* This is the same Union-Find template used in LC 547 (Number of Provinces) and LC 200 (Number of Islands, alternate solution) — building the `parent` structure once and answering connectivity via `find` comparisons is a broadly reusable pattern.
* Path compression (flattening the tree during `find`) and union by rank/size (attaching the smaller tree under the larger one) are both necessary to keep this approach's near-constant-time behavior — omitting either can degrade worst-case performance toward `O(V)` per operation.

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

    void unite(int x, int y) {
        int rootX = find(x);
        int rootY = find(y);

        if (rootX != rootY) {
            parent[rootX] = rootY;
        }
    }

    bool validPath(int n, vector<vector<int>>& edges, int source, int destination) {
        parent.resize(n);
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }

        for (auto& e : edges) {
            unite(e[0], e[1]);
        }

        return find(source) == find(destination);
    }
};
```

---

## Key Template

```text
parent = [0, 1, ..., n-1]

function find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])
    return parent[x]

function unite(x, y):
    rootX = find(x)
    rootY = find(y)
    if rootX != rootY:
        parent[rootX] = rootY

for [u, v] in edges:
    unite(u, v)

return find(source) == find(destination)
```