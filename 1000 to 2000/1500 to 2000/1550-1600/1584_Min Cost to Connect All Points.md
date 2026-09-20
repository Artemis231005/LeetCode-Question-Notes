# LeetCode 1584 — Min Cost to Connect All Points

## Metadata

* **LeetCode:** 1584
* **Problem:** Min Cost to Connect All Points
* **Difficulty:** Medium
* **Topics:** Array, Union Find, Graph, Minimum Spanning Tree, Heap (Priority Queue)
* **Pattern:** Minimum Spanning Tree (Kruskal or Prim)
* **Key Technique:** Since every pair of points is implicitly connected (a complete graph), building an MST over this dense graph favors Prim's array-based `O(n²)` form over Kruskal's edge-sorting approach, which pays an extra `log` factor sorting `O(n²)` edges
* **Optimal Complexity:** `O(n²)` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given an array of 2D points, the cost to connect two points is their Manhattan distance. Return the minimum total cost to connect all points such that there is exactly one path between any two points (i.e. build a minimum spanning tree).

---

## Approaches

1. **Brute Force — Kruskal's Algorithm (Sort All Edges + Union-Find)**
2. **Optimal — Prim's Algorithm (Dense, Array-Based, No Heap)**

---

# Approach 1 — Brute Force / Kruskal's Algorithm (Sort All Edges + Union-Find)

## Idea

Generate every possible edge between pairs of points, each weighted by their Manhattan distance. Sort all edges by weight. Process them in ascending order, using a Union-Find structure to add an edge only if it connects two previously-separate components — this greedily builds the minimum spanning tree, skipping any edge that would create a cycle.

## Dry Run

```text
points = [[0,0],[2,2],[3,10],[5,2],[7,0]]
```

Generate all pairwise edges (Manhattan distance), e.g.:

```text
(0,0)-(2,2): |0-2|+|0-2|=4
(0,0)-(5,2): |0-5|+|0-2|=7
(2,2)-(5,2): |2-5|+|2-2|=3
(5,2)-(7,0): |5-7|+|2-0|=4
... (10 total edges for 5 points)
```

Sort all edges by weight ascending, then process with Union-Find, adding an edge only if its endpoints are in different components:

```text
smallest edges processed first: (2,2)-(5,2)=3 → add, union
(0,0)-(2,2)=4 → add, union
(5,2)-(7,0)=4 → add, union
... continue until all points connected (4 edges needed for 5 points)
```

Sum of added edge weights gives the minimum spanning tree cost.

## Algorithm

1. Generate every edge `(i, j, manhattanDistance(i, j))` for all pairs of points.
2. Sort all edges by weight ascending.
3. Initialize a Union-Find structure with each point in its own set.
4. For each edge in sorted order:

   * If its two endpoints are in different sets, union them and add its weight to `totalCost`.
5. Return `totalCost` once `n - 1` edges have been added.

## Complexity

* **Time:** `O(n² log n)`

  * Generating all `O(n²)` edges takes `O(n²)`, and sorting them dominates at `O(n² log(n²)) = O(n² log n)`.
* **Space:** `O(n²)`

  * For storing every generated edge before sorting.

## Notes / Tips

* Kruskal's algorithm is entirely correct here and a completely standard MST technique — it's labeled "brute force" in this context only relative to Prim's dense-graph form, since sorting `O(n²)` edges introduces an extra logarithmic factor that a dense Prim's implementation avoids entirely.
* This is the more natural first choice when a graph is sparse (`E` much smaller than `V²`), but this problem's implicit "every point connects to every other point" structure makes it a poor fit for edge-sorting-based approaches at scale.

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

    bool unite(int x, int y) {
        int rootX = find(x), rootY = find(y);
        if (rootX == rootY) return false;
        parent[rootX] = rootY;
        return true;
    }

    int minCostConnectPoints(vector<vector<int>>& points) {
        int n = points.size();
        vector<array<int,3>> edges;

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                int dist = abs(points[i][0] - points[j][0]) + abs(points[i][1] - points[j][1]);
                edges.push_back({dist, i, j});
            }
        }

        sort(edges.begin(), edges.end());

        parent.resize(n);
        for (int i = 0; i < n; i++) parent[i] = i;

        int totalCost = 0, edgesUsed = 0;

        for (auto& [dist, u, v] : edges) {
            if (unite(u, v)) {
                totalCost += dist;
                edgesUsed++;
                if (edgesUsed == n - 1) break;
            }
        }

        return totalCost;
    }
};
```

---

# Approach 2 — Optimal / Prim's Algorithm (Dense, Array-Based, No Heap)

## Idea

Since this graph is complete (every point connects to every other point, `O(n²)` edges total), a heap-based Prim's algorithm doesn't help much — the array-based dense form of Prim's is a better fit and avoids `O(n² log n)` sorting entirely. Maintain a `minDist` array tracking, for each unvisited point, the cheapest known edge connecting it to the growing tree. Repeatedly pick the unvisited point with the smallest `minDist` (a linear scan, not a heap), add its cost to the total, then update `minDist` for all other unvisited points based on their distance to the newly added point.

## Dry Run

```text
points = [[0,0],[2,2],[3,10],[5,2],[7,0]]
```

Initialize `minDist = [0, inf, inf, inf, inf]` (start from point 0), `inTree = [false]*5`.

Round 1: pick point `0` (smallest `minDist=0`), add to tree, cost += 0.

```text
update minDist using distances from point 0:
dist(0,1)=4, dist(0,2)=13, dist(0,3)=7, dist(0,4)=7
minDist = [-, 4, 13, 7, 7]
```

Round 2: pick point `1` (smallest remaining `minDist=4`), add to tree, cost += 4.

```text
update minDist using distances from point 1:
dist(1,2)=9, dist(1,3)=3, dist(1,4)=7
minDist[2] = min(13,9)=9, minDist[3] = min(7,3)=3, minDist[4] = min(7,7)=7
minDist = [-, -, 9, 3, 7]
```

Round 3: pick point `3` (smallest remaining `minDist=3`), add to tree, cost += 3.

```text
update minDist using distances from point 3:
dist(3,2)=9, dist(3,4)=4
minDist[2] = min(9,9)=9, minDist[4] = min(7,4)=4
minDist = [-, -, 9, -, 4]
```

Round 4: pick point `4` (smallest remaining `minDist=4`), add to tree, cost += 4.

```text
update minDist using distances from point 4:
dist(4,2)=17
minDist[2] = min(9,17)=9
```

Round 5: pick point `2` (only one left, `minDist=9`), add to tree, cost += 9.

Total cost: `0 + 4 + 3 + 4 + 9 = 20`, matching the expected output.

## Algorithm

1. Initialize `minDist` array of size `n`, all `infinity`, except `minDist[0] = 0`, and `inTree` array of size `n`, all `false`.
2. Initialize `totalCost = 0`.
3. Repeat `n` times:

   * Find the unvisited point `u` with the smallest `minDist[u]` (linear scan).
   * Mark `inTree[u] = true`, add `minDist[u]` to `totalCost`.
   * For every other unvisited point `v`: compute `dist(u, v)`; if it's less than `minDist[v]`, update `minDist[v]`.
4. Return `totalCost`.

## Complexity

* **Time:** `O(n²)`

  * `n` rounds, each doing an `O(n)` linear scan to find the minimum and an `O(n)` update pass over all other points — no sorting and no heap needed.
* **Space:** `O(n)`

  * For the `minDist` and `inTree` arrays.

## Notes / Tips

* This dense (array-based) form of Prim's algorithm is specifically favored over a heap-based Prim's or Kruskal's here because the graph is complete: with `O(n²)` edges total, a heap-based approach's `O(E log V) = O(n² log n)` complexity is strictly worse than this simple `O(n²)` linear-scan version, since there's no sparsity for a heap to exploit.
* No actual edge list is ever built — Manhattan distances are computed on the fly whenever two points are compared, which is what keeps this approach's space usage at `O(n)` instead of `O(n²)`.
* This is the standard trade-off in MST algorithm selection: Kruskal (with sorting) tends to win on sparse graphs, while dense Prim's (array-based, no heap) tends to win on dense/complete graphs.

## Code

```cpp
class Solution {
public:
    int minCostConnectPoints(vector<vector<int>>& points) {
        int n = points.size();
        vector<int> minDist(n, INT_MAX);
        vector<bool> inTree(n, false);
        minDist[0] = 0;

        int totalCost = 0;

        for (int round = 0; round < n; round++) {
            int u = -1;
            for (int i = 0; i < n; i++) {
                if (!inTree[i] && (u == -1 || minDist[i] < minDist[u])) {
                    u = i;
                }
            }

            inTree[u] = true;
            totalCost += minDist[u];

            for (int v = 0; v < n; v++) {
                if (!inTree[v]) {
                    int dist = abs(points[u][0] - points[v][0]) + abs(points[u][1] - points[v][1]);
                    if (dist < minDist[v]) {
                        minDist[v] = dist;
                    }
                }
            }
        }

        return totalCost;
    }
};
```

---

## Key Template

```text
minDist = array of size n, all infinity
minDist[0] = 0
inTree = array of size n, all false
totalCost = 0

repeat n times:
    u = unvisited point with smallest minDist[u]   # linear scan
    inTree[u] = true
    totalCost += minDist[u]

    for each unvisited v:
        dist = manhattan(u, v)
        if dist < minDist[v]:
            minDist[v] = dist

return totalCost
```