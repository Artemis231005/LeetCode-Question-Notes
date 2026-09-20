# LeetCode 1631 — Path With Minimum Effort

## Metadata

* **LeetCode:** 1631
* **Problem:** Path With Minimum Effort
* **Difficulty:** Medium
* **Topics:** Array, Binary Search, Depth-First Search, Breadth-First Search, Union Find, Heap (Priority Queue), Matrix
* **Pattern:** Minimax Path (Dijkstra Variant) or Binary Search on Answer + Reachability Check
* **Key Technique:** "Effort" along a path is the **maximum** single-step height difference, not a sum — this changes Dijkstra's relaxation rule to `max(currentEffort, edgeDiff)` instead of addition, and also makes the answer monotonic, enabling binary search as an alternative
* **Optimal Complexity:** `O(rows * cols * log(rows * cols))` Time, `O(rows * cols)` Auxiliary Space

---

## Problem Statement

Given a grid `heights` representing elevations, find a path from the top-left to the bottom-right cell (moving 4-directionally) that minimizes the **maximum absolute difference** in heights between two consecutive cells on the path. Return that minimized maximum difference.

---

## Approaches

1. **Brute Force — DFS Exploring Every Path**
2. **Better — Binary Search on Answer + BFS/DFS Feasibility Check**
3. **Optimal — Dijkstra Variant (Minimax Relaxation)**

---

# Approach 1 — Brute Force / DFS Exploring Every Path

## Idea

Explore every possible path from the top-left to the bottom-right cell via DFS, tracking the maximum step difference seen so far along the current path. Whenever the destination is reached, compare that path's maximum difference against the best found so far.

## Dry Run

```text
heights = [[1,2,2],[3,8,2],[5,3,5]]
```

DFS path `(0,0) -> (0,1) -> (0,2) -> (1,2) -> (2,2)`:

```text
diffs: |1-2|=1, |2-2|=0, |2-2|=0, |2-5|=3
max along this path = 3
```

DFS path `(0,0) -> (1,0) -> (2,0) -> (2,1) -> (2,2)`:

```text
diffs: |1-3|=2, |3-5|=2, |5-3|=2, |3-5|=2
max along this path = 2
```

Continue exploring all paths (with a `visited` set to avoid infinite loops) — the best (smallest maximum) found across all of them is `2`.

## Algorithm

1. Define a recursive `dfs(r, c, visited, currentMax)`:

   * If `(r, c)` is the destination, update `best = min(best, currentMax)` and return.
   * For each of the 4 neighbors not yet visited:

     * Compute `diff = abs(heights[r][c] - heights[nr][nc])`.
     * Recurse with `max(currentMax, diff)`, marking `(nr, nc)` visited, then unmark it after (backtrack).
2. Call `dfs(0, 0, {(0,0)}, 0)`.
3. Return `best`.

## Complexity

* **Time:** `O(4^(rows*cols))` in the worst case

  * Exploring every possible path with backtracking has exponential branching.
* **Space:** `O(rows * cols)`

  * For the `visited` set and recursion stack.

## Notes / Tips

* This blows up extremely quickly even on modest grids, a grid path problem like this almost always needs either a shortest-path algorithm adapted to the "minimize the maximum edge" objective, or a binary search + reachability reformulation.
* Useful only for confirming correctness of approach on tiny grids before moving to a real algorithm.

## Code

```cpp
class Solution {
public:
    int rows, cols;
    int best = INT_MAX;

    void dfs(vector<vector<int>>& heights, int r, int c, vector<vector<bool>>& visited, int currentMax) {
        if (r == rows - 1 && c == cols - 1) {
            best = min(best, currentMax);
            return;
        }

        int dr[] = {1, -1, 0, 0};
        int dc[] = {0, 0, 1, -1};

        for (int d = 0; d < 4; d++) {
            int nr = r + dr[d], nc = c + dc[d];
            if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && !visited[nr][nc]) {
                int diff = abs(heights[r][c] - heights[nr][nc]);
                visited[nr][nc] = true;
                dfs(heights, nr, nc, visited, max(currentMax, diff));
                visited[nr][nc] = false;
            }
        }
    }

    int minimumEffortPath(vector<vector<int>>& heights) {
        rows = heights.size();
        cols = heights[0].size();

        vector<vector<bool>> visited(rows, vector<bool>(cols, false));
        visited[0][0] = true;

        dfs(heights, 0, 0, visited, 0);
        return best;
    }
};
```

---

# Approach 2 — Better / Binary Search on Answer + BFS/DFS Feasibility Check

## Idea

The answer (minimum possible "maximum effort") is monotonic: if a path exists using only steps with difference `<= x`, then a path also exists for any `x' > x` (the same path still qualifies). This monotonic "yes/no" structure means binary search can find the smallest feasible `x`. For each candidate `x`, run a BFS/DFS from the top-left, only moving through edges whose height difference is `<= x`, and check whether the bottom-right cell is reachable.

## Dry Run

```text
heights = [[1,2,2],[3,8,2],[5,3,5]]
```

Binary search bounds: `low = 0`, `high = max height - min height = 8 - 1 = 7`.

```text
mid = 3: BFS allowing only steps with diff <= 3
   (0,0)->(0,1) diff=1 ok, (0,0)->(1,0) diff=2 ok, ...
   destination (2,2) reachable → feasible → high = 3

mid = 1: BFS allowing only steps with diff <= 1
   (0,0)->(0,1) diff=1 ok; (0,0)->(1,0) diff=2 too big, blocked
   ... destination not reachable via any diff<=1 path → infeasible → low = 2

mid = 2: BFS allowing only steps with diff <= 2
   (0,0)->(1,0) diff=2 ok, (1,0)->(2,0) diff=2 ok, (2,0)->(2,1) diff=2 ok, (2,1)->(2,2) diff=2 ok
   destination reachable → feasible → high = 2
```

`low == high == 2` → answer `2`, matching the brute-force result.

## Algorithm

1. Define a helper `feasible(x)`:

   * BFS/DFS from `(0,0)`, only traversing to a neighbor if `abs(heights[r][c] - heights[nr][nc]) <= x`.
   * Return whether `(rows-1, cols-1)` was reached.
2. Binary search `x` between `0` and the maximum possible height difference in the grid:

   * If `feasible(mid)`, the answer is at most `mid` — move `high = mid`.
   * Otherwise, move `low = mid + 1`.
3. Return `low`.

## Complexity

* **Time:** `O(rows * cols * log(maxHeightDiff))`

  * Binary search runs `O(log(maxHeightDiff))` iterations, each feasibility check costing `O(rows * cols)` for a full BFS/DFS.
* **Space:** `O(rows * cols)`

  * For the `visited` structure and BFS queue/DFS stack used in each feasibility check.

## Notes / Tips

* This is the standard "binary search on the answer" pattern (same shape as LC 1292 and LC 3356) applied to a path-finding feasibility question instead of a direct arithmetic one.
* The monotonicity argument is what justifies binary search: a larger allowed threshold `x` can only ever unlock more edges (never remove any that were previously usable), so feasibility never "flips back" to infeasible as `x` increases.
* Approach 3's Dijkstra variant achieves a comparable or better time complexity without needing the extra `log` factor from repeated feasibility checks, making it the more commonly preferred "optimal" solution.

## Code

```cpp
class Solution {
public:
    int rows, cols;

    bool feasible(vector<vector<int>>& heights, int x) {
        vector<vector<bool>> visited(rows, vector<bool>(cols, false));
        queue<pair<int,int>> q;
        q.push({0, 0});
        visited[0][0] = true;

        int dr[] = {1, -1, 0, 0};
        int dc[] = {0, 0, 1, -1};

        while (!q.empty()) {
            auto [r, c] = q.front();
            q.pop();

            if (r == rows - 1 && c == cols - 1) {
                return true;
            }

            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && !visited[nr][nc]) {
                    if (abs(heights[r][c] - heights[nr][nc]) <= x) {
                        visited[nr][nc] = true;
                        q.push({nr, nc});
                    }
                }
            }
        }

        return false;
    }

    int minimumEffortPath(vector<vector<int>>& heights) {
        rows = heights.size();
        cols = heights[0].size();

        int low = 0, high = 1000000;

        while (low < high) {
            int mid = low + (high - low) / 2;

            if (feasible(heights, mid)) {
                high = mid;
            } else {
                low = mid + 1;
            }
        }

        return low;
    }
};
```

---

# Approach 3 — Optimal / Dijkstra Variant (Minimax Relaxation)

## Idea

Adapt Dijkstra's algorithm to this "minimize the maximum edge on the path" objective instead of the usual "minimize the sum of edges." Use a min-heap ordered by each path's current effort. When relaxing an edge, instead of adding the edge weight to the running total, take the **max** of the current path's effort and the new edge's difference — this directly represents "the effort to complete a path is however bad its single worst step was."

## Dry Run

```text
heights = [[1,2,2],[3,8,2],[5,3,5]]
```

Min-heap starts with `(effort=0, r=0, c=0)`.

Pop `(0, 0, 0)`:

```text
neighbor (0,1): diff=1, newEffort=max(0,1)=1 < dist[0][1]=inf → update, push (1,0,1)
neighbor (1,0): diff=2, newEffort=max(0,2)=2 < dist[1][0]=inf → update, push (2,1,0)
```

Pop `(1, 0, 1)`:

```text
neighbor (0,2): diff=0, newEffort=max(1,0)=1 < inf → update, push (1,0,2)
neighbor (1,1): diff=6, newEffort=max(1,6)=6 < inf → update, push (6,1,1)
```

Pop `(1, 0, 2)`:

```text
neighbor (1,2): diff=0, newEffort=max(1,0)=1 < inf → update, push (1,1,2)
```

Pop `(1, 1, 2)`:

```text
neighbor (2,2): diff=3, newEffort=max(1,3)=3 < inf → update, push (3,2,2)
```

Pop `(2, 1, 0)`:

```text
neighbor (2,0): diff=2, newEffort=max(2,2)=2 < inf → update, push (2,2,0)
```

Pop `(2, 2, 0)`:

```text
neighbor (2,1): diff=2, newEffort=max(2,2)=2 < inf → update, push (2,2,1)
```

Pop `(2, 2, 1)`:

```text
neighbor (2,2): diff=2, newEffort=max(2,2)=2 < dist[2][2]=3 → improve! update dist[2][2]=2, push (2,2,2)
```

Pop `(2, 2, 2)`: destination reached with effort `2` → return `2`, matching both earlier approaches.

## Algorithm

1. Initialize a `dist` matrix of size `rows x cols`, all `infinity`, except `dist[0][0] = 0`.
2. Initialize a min-heap with `(0, 0, 0)` (effort, row, col).
3. While the heap is non-empty:

   * Pop the `(currentEffort, r, c)` with smallest effort.
   * If `(r, c)` is the destination, return `currentEffort`.
   * If `currentEffort > dist[r][c]`, skip (stale entry).
   * For each of the 4 neighbors: compute `diff`, then `newEffort = max(currentEffort, diff)`. If `newEffort < dist[nr][nc]`, update and push.
4. Return `dist[rows-1][cols-1]` once the heap is exhausted (or return immediately upon popping the destination, as above).

## Complexity

* **Time:** `O(rows * cols * log(rows * cols))`

  * Each cell can be pushed onto the heap a bounded number of times, and each heap operation costs `O(log(rows * cols))`.
* **Space:** `O(rows * cols)`

  * For the `dist` matrix and the heap.

## Notes / Tips

* The only structural change from standard Dijkstra is the relaxation rule: `max(currentEffort, edgeDiff)` instead of `currentDist + edgeWeight`, which is why this variant is sometimes called a "minimax" or "bottleneck" shortest path.
* Returning immediately upon popping the destination (rather than waiting for the heap to fully empty) is a valid and common optimization, since Dijkstra guarantees the first time a node is popped, its distance is finalized and minimal.
* This minimax-Dijkstra approach and the binary search-plus-feasibility approach avoids the extra logarithmic factor from repeated full-grid feasibility checks, making it marginally more efficient, but both are considered "optimal" in practice for this problem's typical constraints.

## Code

```cpp
class Solution {
public:
    int minimumEffortPath(vector<vector<int>>& heights) {
        int rows = heights.size(), cols = heights[0].size();
        vector<vector<int>> dist(rows, vector<int>(cols, INT_MAX));
        dist[0][0] = 0;

        priority_queue<tuple<int,int,int>, vector<tuple<int,int,int>>, greater<>> pq;
        pq.push({0, 0, 0});

        int dr[] = {1, -1, 0, 0};
        int dc[] = {0, 0, 1, -1};

        while (!pq.empty()) {
            auto [effort, r, c] = pq.top();
            pq.pop();

            if (r == rows - 1 && c == cols - 1) {
                return effort;
            }

            if (effort > dist[r][c]) {
                continue;
            }

            for (int d = 0; d < 4; d++) {
                int nr = r + dr[d], nc = c + dc[d];
                if (nr >= 0 && nr < rows && nc >= 0 && nc < cols) {
                    int diff = abs(heights[r][c] - heights[nr][nc]);
                    int newEffort = max(effort, diff);

                    if (newEffort < dist[nr][nc]) {
                        dist[nr][nc] = newEffort;
                        pq.push({newEffort, nr, nc});
                    }
                }
            }
        }

        return dist[rows - 1][cols - 1];
    }
};
```

---

## Key Template

```text
dist = matrix of size rows x cols, all infinity
dist[0][0] = 0

minHeap = [(0, 0, 0)]  # (effort, r, c)

while minHeap not empty:
    (effort, r, c) = minHeap.pop()
    if (r, c) == destination: return effort
    if effort > dist[r][c]: continue

    for each neighbor (nr, nc):
        diff = abs(heights[r][c] - heights[nr][nc])
        newEffort = max(effort, diff)

        if newEffort < dist[nr][nc]:
            dist[nr][nc] = newEffort
            minHeap.push((newEffort, nr, nc))

return dist[destination]
```