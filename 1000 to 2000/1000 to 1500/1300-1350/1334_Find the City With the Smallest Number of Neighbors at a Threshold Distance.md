# LeetCode 1334 — Find the City With the Smallest Number of Neighbors at a Threshold Distance

## Metadata

* **LeetCode:** 1334
* **Problem:** Find the City With the Smallest Number of Neighbors at a Threshold Distance
* **Difficulty:** Medium
* **Topics:** Dynamic Programming, Graph, Shortest Path
* **Pattern:** All-Pairs Shortest Path (Floyd-Warshall)
* **Key Technique:** Since the graph is small and dense, compute shortest distances between **every** pair of cities at once with Floyd-Warshall, rather than running a separate single-source shortest path search from each city
* **Optimal Complexity:** `O(n³)` Time, `O(n²)` Space

---

## Problem Statement

Given `n` cities connected by weighted edges and a distance `threshold`, find the city that has the **smallest number** of other cities reachable within `threshold` distance. If there's a tie, return the city with the **greatest label**.

---

## Approaches

1. **Brute Force — Dijkstra From Every City**
2. **Optimal — Floyd-Warshall (All-Pairs Shortest Path)**

---

# Approach 1 — Brute Force / Dijkstra From Every City

## Idea

Treat this as `n` separate single-source shortest path problems: for each city, run Dijkstra's algorithm to find its shortest distance to every other city, then count how many of those distances fall within `threshold`. Track the city with the fewest such neighbors (breaking ties by preferring the larger label).

## Dry Run

```text
n = 4, edges = [[0,1,3],[1,2,1],[1,3,4],[2,3,1]], threshold = 4
```

Run Dijkstra from city `0`:

```text
dist from 0: [0, 3, 4, 5]
within threshold=4: cities 1 (dist 3), 2 (dist 4) → count = 2
```

Run Dijkstra from city `3`:

```text
dist from 3: [5, 4, 1, 0]
within threshold=4: cities 1 (dist 4), 2 (dist 1) → count = 2
```

Continue for cities `1` and `2`, then compare all four counts to find the minimum, breaking ties toward the larger label.

## Algorithm

1. Build a weighted adjacency list from `edges`.
2. For each city `i` from `0` to `n-1`:

   * Run Dijkstra's algorithm from `i` to get shortest distances to every other city.
   * Count how many other cities have distance `<= threshold`.
3. Track the city with the minimum count, preferring the larger label on ties.
4. Return that city.

## Complexity

* **Time:** `O(n * E log n)`

  * Running Dijkstra `n` times, each costing `O(E log n)` with a binary heap; in a dense graph (`E` up to `O(n²)`), this becomes `O(n³ log n)`.
* **Space:** `O(n + E)`

  * For the adjacency list, plus `O(n)` per Dijkstra run for distances and the heap.

## Notes / Tips

* Running Dijkstra independently from every source re-derives a lot of overlapping structure — Floyd-Warshall computes all pairwise distances together in a single unified computation, and for a dense graph (this problem's typical case, given small `n`), it actually beats repeated Dijkstra asymptotically due to the extra `log n` factor Dijkstra pays per run.
* Dijkstra is still a perfectly valid approach here and is what's needed if the graph were sparse and `n` were much larger — it's labeled "brute force" only relative to Floyd-Warshall's better fit for this problem's small, dense constraints.

## Code

```cpp
class Solution {
public:
    int findTheCity(int n, vector<vector<int>>& edges, int threshold) {
        vector<vector<pair<int,int>>> graph(n);
        for (auto& e : edges) {
            graph[e[0]].push_back({e[1], e[2]});
            graph[e[1]].push_back({e[0], e[2]});
        }

        int bestCity = -1, bestCount = INT_MAX;

        for (int src = 0; src < n; src++) {
            vector<int> dist(n, INT_MAX);
            dist[src] = 0;

            priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
            pq.push({0, src});

            while (!pq.empty()) {
                auto [d, node] = pq.top();
                pq.pop();

                if (d > dist[node]) continue;

                for (auto& [next, weight] : graph[node]) {
                    if (d + weight < dist[next]) {
                        dist[next] = d + weight;
                        pq.push({dist[next], next});
                    }
                }
            }

            int count = 0;
            for (int city = 0; city < n; city++) {
                if (city != src && dist[city] <= threshold) {
                    count++;
                }
            }

            if (count <= bestCount) {
                bestCount = count;
                bestCity = src;
            }
        }

        return bestCity;
    }
};
```

---

# Approach 2 — Optimal / Floyd-Warshall (All-Pairs Shortest Path)

## Idea

Build an `n x n` distance matrix, initialized with direct edge weights (and `infinity` elsewhere). Run the Floyd-Warshall triple loop: for every intermediate city `k`, check if routing through `k` shortens the known distance between every pair `(i, j)`. After this single pass, the matrix holds the shortest distance between **every** pair of cities at once. Then, for each city, count how many others are within `threshold` and pick the best.

## Dry Run

```text
n = 4, edges = [[0,1,3],[1,2,1],[1,3,4],[2,3,1]], threshold = 4
```

Initialize `dist` matrix from direct edges (symmetric, `infinity` elsewhere, `0` on the diagonal):

```text
dist[0][1]=3, dist[1][2]=1, dist[1][3]=4, dist[2][3]=1 (and symmetric entries)
```

Floyd-Warshall relaxation, trying each `k` as an intermediate:

```text
k=1: dist[0][2] = min(inf, dist[0][1]+dist[1][2]) = min(inf, 3+1) = 4
     dist[0][3] = min(inf, dist[0][1]+dist[1][3]) = min(inf, 3+4) = 7
k=2: dist[0][3] = min(7, dist[0][2]+dist[2][3]) = min(7, 4+1) = 5
     dist[1][3] = min(4, dist[1][2]+dist[2][3]) = min(4, 1+1) = 2
k=3: no further improvements found
```

Final distances from city `0`: `[0, 3, 4, 5]`. From city `1`: `[3, 0, 1, 2]`. From city `2`: `[4, 1, 0, 1]`. From city `3`: `[5, 2, 1, 0]`.

Count neighbors within `threshold=4` for each city:

```text
city 0: distances [3,4,5] → within 4: {1,2} → count=2
city 1: distances [3,1,2] → within 4: {0,2,3} → count=3
city 2: distances [4,1,1] → within 4: {0,1,3} → count=3
city 3: distances [5,2,1] → within 4: {1,2} → count=2
```

Minimum count is `2`, tied between cities `0` and `3` → prefer the larger label → return `3`.

## Algorithm

1. Initialize `dist` matrix of size `n x n`: `0` on the diagonal, direct edge weights where given, `infinity` elsewhere.
2. Run Floyd-Warshall: for `k` from `0` to `n-1`, for `i` from `0` to `n-1`, for `j` from `0` to `n-1`:

   * If `dist[i][k] + dist[k][j] < dist[i][j]`, update `dist[i][j] = dist[i][k] + dist[k][j]`.
3. For each city `i`, count how many other cities `j` have `dist[i][j] <= threshold`.
4. Track the city with the minimum count, preferring the larger label on ties.
5. Return that city.

## Complexity

* **Time:** `O(n³)`

  * The Floyd-Warshall triple loop dominates; counting neighbors afterward is an additional `O(n²)`.
* **Space:** `O(n²)`

  * For the `dist` matrix.

## Notes / Tips

* Floyd-Warshall is a natural fit whenever `n` is small (here, typically `<= 100` per this problem's constraints) and **all** pairwise distances are needed — computing them together in one unified triple loop avoids the overhead of `n` separate single-source searches.
* The `k` loop must be outermost for correctness, same as in LC 1462's transitive closure — `dist[i][j]` may depend on paths through `k` that themselves rely on `dist[i][k]` and `dist[k][j]` already being finalized for smaller values of `k`.
* When comparing counts across cities, using `<=` (not `<`) when updating the best city is what implements the tie-breaking rule "prefer the larger label" — since cities are checked in increasing order, a later city with an equal count will always overwrite an earlier one.

## Code

```cpp
class Solution {
public:
    int findTheCity(int n, vector<vector<int>>& edges, int threshold) {
        vector<vector<int>> dist(n, vector<int>(n, INT_MAX / 2));

        for (int i = 0; i < n; i++) {
            dist[i][i] = 0;
        }

        for (auto& e : edges) {
            dist[e[0]][e[1]] = e[2];
            dist[e[1]][e[0]] = e[2];
        }

        for (int k = 0; k < n; k++) {
            for (int i = 0; i < n; i++) {
                for (int j = 0; j < n; j++) {
                    if (dist[i][k] + dist[k][j] < dist[i][j]) {
                        dist[i][j] = dist[i][k] + dist[k][j];
                    }
                }
            }
        }

        int bestCity = -1, bestCount = INT_MAX;

        for (int i = 0; i < n; i++) {
            int count = 0;
            for (int j = 0; j < n; j++) {
                if (i != j && dist[i][j] <= threshold) {
                    count++;
                }
            }

            if (count <= bestCount) {
                bestCount = count;
                bestCity = i;
            }
        }

        return bestCity;
    }
};
```

---

## Key Template

```text
dist = matrix of size n x n, all infinity, 0 on diagonal
fill in direct edge weights (symmetric)

for k in 0..n-1:
    for i in 0..n-1:
        for j in 0..n-1:
            if dist[i][k] + dist[k][j] < dist[i][j]:
                dist[i][j] = dist[i][k] + dist[k][j]

bestCity = -1, bestCount = infinity
for i in 0..n-1:
    count = number of j != i with dist[i][j] <= threshold
    if count <= bestCount:
        bestCount = count
        bestCity = i

return bestCity
```