# LeetCode 547 — Number of Provinces

## Metadata

* **LeetCode:** 547
* **Problem:** Number of Provinces
* **Difficulty:** Medium
* **Topics:** Array, Depth-First Search, Breadth-First Search, Union Find, Graph
* **Pattern:** Connected Components (DFS/BFS or Union-Find)
* **Key Technique:** Treat the adjacency matrix as a graph and count connected components — either by marking visited nodes during traversal, or by merging connected nodes into groups with a Union-Find structure
* **Optimal Complexity:** `O(n² * α(n))` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given an `n x n` matrix `isConnected` where `isConnected[i][j] = 1` if city `i` and city `j` are directly connected, return the total number of provinces (a province is a group of directly or indirectly connected cities).

---

## Approaches

1. **Brute Force — DFS/BFS Traversal to Mark Connected Components**
2. **Optimal — Union-Find (Disjoint Set Union)**

---

# Approach 1 — Brute Force / DFS/BFS Traversal to Mark Connected Components

## Idea

Treat each city as a node in a graph, with `isConnected[i][j] == 1` meaning an edge between city `i` and city `j`. For each unvisited city, run a DFS (or BFS) to mark every city reachable from it as visited — this entire reachable set is one province. Count how many times a fresh traversal needs to be started.

## Dry Run

```text
isConnected = [[1,1,0],
               [1,1,0],
               [0,0,1]]
```

Start at city `0` (unvisited):

```text
DFS from 0 → visits 0, then checks connections: 1 is connected → visit 1
1's connections: 0 (already visited) → done
visited = {0, 1}
provinces = 1
```

City `1` is already visited → skip.

City `2` (unvisited):

```text
DFS from 2 → visits 2, no other connections
visited = {0, 1, 2}
provinces = 2
```

Final: `2` provinces.

## Algorithm

1. Initialize a `visited` array of size `n`, all `false`, and `provinces = 0`.
2. For each city `i` from `0` to `n-1`:

   * If `visited[i]` is `false`:

     * Increment `provinces`.
     * Run DFS (or BFS) from `i`, marking every reachable city as `visited`.
3. Return `provinces`.

## Complexity

* **Time:** `O(n²)`

  * Each DFS/BFS call, across all starting points combined, visits every cell of the `n x n` adjacency matrix at most once.
* **Space:** `O(n)`

  * For the `visited` array and the recursion stack (or BFS queue), each bounded by `n`.

## Notes / Tips

* This is the standard, very natural way to count connected components in a graph — clean and efficient, and for this problem it's essentially just as fast as the Union-Find approach since the input is already a dense adjacency matrix (there's no faster way to even read the input than `O(n²)`).
* Recursion depth in DFS can reach `O(n)` in the worst case (a long chain of connections) — an iterative BFS with an explicit queue avoids any risk of stack overflow on very large inputs.

## Code

```cpp
class Solution {
public:
    void dfs(vector<vector<int>>& isConnected, vector<bool>& visited, int city) {
        visited[city] = true;

        for (int neighbor = 0; neighbor < isConnected.size(); neighbor++) {
            if (isConnected[city][neighbor] == 1 && !visited[neighbor]) {
                dfs(isConnected, visited, neighbor);
            }
        }
    }

    int findCircleNum(vector<vector<int>>& isConnected) {
        int n = isConnected.size();
        vector<bool> visited(n, false);
        int provinces = 0;

        for (int i = 0; i < n; i++) {
            if (!visited[i]) {
                provinces++;
                dfs(isConnected, visited, i);
            }
        }

        return provinces;
    }
};
```

---

# Approach 2 — Optimal / Union-Find (Disjoint Set Union)

## Idea

Treat each city as its own group initially. Scan every pair `(i, j)` in the adjacency matrix, and whenever `isConnected[i][j] == 1`, union their groups together using a Union-Find structure with path compression and union by rank. After processing all connections, the number of distinct root parents left is the number of provinces.

## Dry Run

```text
isConnected = [[1,1,0],
               [1,1,0],
               [0,0,1]]
```

Initialize `parent = [0, 1, 2]` (each city its own group).

```text
(0,1): isConnected[0][1] = 1 → union(0, 1) → parent = [0, 0, 2] (1's root becomes 0)
(1,0): already unioned, no-op
(2,2): self-connection, no-op
```

Count distinct roots by calling `find` on each city:

```text
find(0) = 0
find(1) = 0
find(2) = 2
```

Distinct roots: `{0, 2}` → `2` provinces.

## Algorithm

1. Initialize a `parent` array of size `n`, where `parent[i] = i` (each city is its own group).
2. For each pair `(i, j)` with `i < j`:

   * If `isConnected[i][j] == 1`, union the groups containing `i` and `j`:

     * Find each one's root using path compression.
     * If the roots differ, merge them (attach one root under the other, optionally using rank/size to keep the structure balanced).
3. Count the number of distinct roots by calling `find(i)` for every city and counting unique results.
4. Return that count.

## Complexity

* **Time:** `O(n² * α(n))`

  * Scanning the `n x n` matrix takes `O(n²)`; each union/find operation is nearly `O(1)` on average thanks to path compression and union by rank (`α` is the inverse Ackermann function, effectively constant).
* **Space:** `O(n)`

  * For the `parent` (and optional `rank`) arrays.

## Notes / Tips

* For this specific problem, Union-Find doesn't  beat DFS/BFS — both are `O(n²)` since the input itself is a dense `n x n` matrix that must be fully read. Union-Find becomes clearly advantageous in problems where connections arrive as a list of edges rather than a full matrix, or where connectivity needs to be queried/updated dynamically.
* Path compression (making every node on the find path point directly to the root) and union by rank/size are what keep each operation nearly constant time — skipping either still gives a correct but potentially much slower (`O(n)` per operation in the worst case) structure.
* This is the standard template for "count connected components" problems phrased as either a graph or a merge/grouping scenario — the same Union-Find skeleton reappears in problems like "Accounts Merge" or "Redundant Connection."

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

    int findCircleNum(vector<vector<int>>& isConnected) {
        int n = isConnected.size();
        parent.resize(n);
        for (int i = 0; i < n; i++) {
            parent[i] = i;
        }

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (isConnected[i][j] == 1) {
                    unite(i, j);
                }
            }
        }

        int provinces = 0;
        for (int i = 0; i < n; i++) {
            if (find(i) == i) {
                provinces++;
            }
        }

        return provinces;
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

for i in 0..n-1:
    for j in i+1..n-1:
        if isConnected[i][j] == 1:
            unite(i, j)

count distinct find(i) for i in 0..n-1
return count
```