# LeetCode 743 — Network Delay Time

## Metadata

* **LeetCode:** 743
* **Problem:** Network Delay Time
* **Difficulty:** Medium
* **Topics:** Depth-First Search, Breadth-First Search, Graph, Heap (Priority Queue), Shortest Path
* **Pattern:** Single-Source Shortest Path (Dijkstra's Algorithm)
* **Key Technique:** Find the shortest time for a signal to reach every node from the source using Dijkstra's algorithm, then the answer is the **maximum** of all those shortest times (the last node to receive the signal determines the total delay)
* **Optimal Complexity:** `O(E log V)` Time, `O(V + E)` Auxiliary Space

---

## Problem Statement

Given `n` network nodes labeled `1` to `n`, a list of directed edges `times[i] = [u, v, w]` (a signal takes `w` time to travel from `u` to `v`), and a source node `k`, return the minimum time for a signal sent from `k` to reach **all** nodes. Return `-1` if it's impossible for all nodes to receive the signal.

---

## Approaches

1. **Brute Force — Bellman-Ford (Relax All Edges Repeatedly)**
2. **Optimal — Dijkstra's Algorithm (Min-Heap)**

---

# Approach 1 — Brute Force / Bellman-Ford (Relax All Edges Repeatedly)

## Idea

Initialize the shortest known time to reach every node as infinity, except the source (`0`). Repeatedly relax every edge — if traveling through edge `(u, v, w)` would improve the known shortest time to `v`, update it. Since a shortest path in a graph with `V` nodes uses at most `V-1` edges, repeating this relaxation process `V-1` times guarantees convergence to the true shortest distances.

## Dry Run

```text
times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
```

Initialize `dist = [inf, inf, 0, inf, inf]` (1-indexed, `dist[2]=0` since `k=2`).

Round 1 — relax every edge:

```text
edge 2->1 (1): dist[2]+1=1 < dist[1]=inf → dist[1]=1
edge 2->3 (1): dist[2]+1=1 < dist[3]=inf → dist[3]=1
edge 3->4 (1): dist[3]+1=inf+1=inf → no update yet (dist[3] was still inf at the start of this round if using a strict "snapshot" version, or already updated to 1 if updating in place — Bellman-Ford conventionally allows in-place updates within a round)
```

Using in-place updates (standard Bellman-Ford), `dist[3]` is already `1` by the time edge `3->4` is relaxed in the same round:

```text
edge 3->4 (1): dist[3]+1=1+1=2 < dist[4]=inf → dist[4]=2
```

After round 1: `dist = [-, 1, 0, 1, 2]`.

Further rounds make no additional improvements → converged.

Maximum distance among reachable nodes: `max(1, 0, 1, 2) = 2`, matching the expected output.

## Algorithm

1. Initialize a `dist` array of size `n+1`, all `infinity`, except `dist[k] = 0`.
2. Repeat `n - 1` times:

   * For each edge `(u, v, w)` in `times`: if `dist[u] + w < dist[v]`, update `dist[v] = dist[u] + w`.
3. Find the maximum value among `dist[1..n]`. If any is still `infinity`, return `-1`.
4. Otherwise, return that maximum.

## Complexity

* **Time:** `O(V * E)`

  * `V - 1` rounds, each relaxing all `E` edges.
* **Space:** `O(V)`

  * For the `dist` array.

## Notes / Tips

* Bellman-Ford is correct here and handles the single-source shortest path problem without needing a priority queue, but it does significantly more work than necessary — Dijkstra's algorithm (Approach 2) reaches the same result much faster by always processing the closest unprocessed node next, rather than blindly relaxing every edge repeatedly.
* Bellman-Ford's main advantage over Dijkstra — correctly handling negative edge weights — isn't needed here, since edge weights (`wi`) are guaranteed positive per the problem's constraints, making Dijkstra strictly the better fit.

## Code

```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<int> dist(n + 1, INT_MAX);
        dist[k] = 0;

        for (int round = 0; round < n - 1; round++) {
            for (auto& t : times) {
                int u = t[0], v = t[1], w = t[2];
                if (dist[u] != INT_MAX && dist[u] + w < dist[v]) {
                    dist[v] = dist[u] + w;
                }
            }
        }

        int maxDist = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == INT_MAX) {
                return -1;
            }
            maxDist = max(maxDist, dist[i]);
        }

        return maxDist;
    }
};
```

---

# Approach 2 — Optimal / Dijkstra's Algorithm (Min-Heap)

## Idea

Since all edge weights are positive, Dijkstra's algorithm applies directly: use a min-heap to always process the currently-closest unvisited node next. Starting from `k` with distance `0`, repeatedly pop the closest node, finalize its shortest distance, and relax all its outgoing edges — pushing any improved distances onto the heap. Once every reachable node has been finalized, the answer is the maximum finalized distance (or `-1` if not every node was reached).

## Dry Run

```text
times = [[2,1,1],[2,3,1],[3,4,1]], n = 4, k = 2
```

Min-heap starts with `(dist=0, node=2)`.

Pop `(0, 2)`:

```text
dist[2] = 0 (finalized)
relax 2->1 (1): 0+1=1 < inf → dist[1]=1, push (1,1)
relax 2->3 (1): 0+1=1 < inf → dist[3]=1, push (1,3)
heap = [(1,1), (1,3)]
```

Pop `(1, 1)` (or `(1,3)`, tie-broken arbitrarily):

```text
dist[1] = 1 (finalized)
node 1 has no outgoing edges → nothing to relax
```

Pop `(1, 3)`:

```text
dist[3] = 1 (finalized)
relax 3->4 (1): 1+1=2 < inf → dist[4]=2, push (2,4)
```

Pop `(2, 4)`:

```text
dist[4] = 2 (finalized)
no outgoing edges
```

Heap empty. Finalized distances: `dist[1]=1, dist[2]=0, dist[3]=1, dist[4]=2`. Maximum: `2`, matching the brute-force result.

## Algorithm

1. Build an adjacency list from `times`.
2. Initialize a `dist` array of size `n+1`, all `infinity`, except `dist[k] = 0`.
3. Initialize a min-heap with `(0, k)`.
4. While the heap is non-empty:

   * Pop the `(currentDist, node)` with smallest `currentDist`.
   * If `currentDist > dist[node]`, skip (a stale, already-improved-upon heap entry).
   * For each neighbor `(next, weight)` of `node`: if `currentDist + weight < dist[next]`, update `dist[next]` and push `(dist[next], next)`.
5. Find the maximum value among `dist[1..n]`. If any node is still `infinity`, return `-1`; otherwise return that maximum.

## Complexity

* **Time:** `O(E log V)`

  * Each edge can trigger at most one heap push (`O(log V)` each), and the heap holds at most `O(E)` entries across the whole run.
* **Space:** `O(V + E)`

  * For the adjacency list, the `dist` array, and the heap.

## Notes / Tips

* The `if currentDist > dist[node]: skip` check is what handles "stale" heap entries — since a node can be pushed onto the heap multiple times with different distances before being finalized, this check ensures only the first (smallest) pop for any node actually does the relaxation work, and later, worse entries are safely ignored.
* Dijkstra's algorithm requires non-negative edge weights to guarantee correctness — this problem's constraints (`wi >= 1`) make it a clean fit; if negative weights were possible, Bellman-Ford (Approach 1) would be the only correct choice.
* This is the standard single-source shortest path template — the only problem-specific addition here is taking the **maximum** of all shortest distances at the end (since the signal isn't "done" until it reaches the *last* node), rather than looking up a single target distance as in a typical shortest-path query.

## Code

```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<vector<pair<int,int>>> graph(n + 1);
        for (auto& t : times) {
            graph[t[0]].push_back({t[1], t[2]});
        }

        vector<int> dist(n + 1, INT_MAX);
        dist[k] = 0;

        priority_queue<pair<int,int>, vector<pair<int,int>>, greater<>> pq;
        pq.push({0, k});

        while (!pq.empty()) {
            auto [currentDist, node] = pq.top();
            pq.pop();

            if (currentDist > dist[node]) {
                continue;
            }

            for (auto& [next, weight] : graph[node]) {
                if (currentDist + weight < dist[next]) {
                    dist[next] = currentDist + weight;
                    pq.push({dist[next], next});
                }
            }
        }

        int maxDist = 0;
        for (int i = 1; i <= n; i++) {
            if (dist[i] == INT_MAX) {
                return -1;
            }
            maxDist = max(maxDist, dist[i]);
        }

        return maxDist;
    }
};
```

---

## Key Template

```text
graph = adjacency list from times
dist = array of size (n+1), all infinity
dist[k] = 0

minHeap = [(0, k)]

while minHeap not empty:
    (currentDist, node) = minHeap.pop()
    if currentDist > dist[node]: continue

    for (next, weight) in graph[node]:
        if currentDist + weight < dist[next]:
            dist[next] = currentDist + weight
            minHeap.push((dist[next], next))

maxDist = 0
for i in 1..n:
    if dist[i] == infinity: return -1
    maxDist = max(maxDist, dist[i])

return maxDist
```