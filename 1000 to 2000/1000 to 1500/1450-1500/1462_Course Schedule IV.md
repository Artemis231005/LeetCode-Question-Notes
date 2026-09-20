# LeetCode 1462 — Course Schedule IV

## Metadata

* **LeetCode:** 1462
* **Problem:** Course Schedule IV
* **Difficulty:** Medium
* **Topics:** Graph, Topological Sort, Depth-First Search, Breadth-First Search, Dynamic Programming
* **Pattern:** Transitive Closure Precomputation (Floyd-Warshall)
* **Key Technique:** Precompute, once, whether every course is reachable from every other course (the full transitive closure of the prerequisite graph), so each query afterward is a single `O(1)` lookup instead of a fresh graph search
* **Optimal Complexity:** `O(n³)` Time (precomputation), `O(n²)` Space

---

## Problem Statement

Given `numCourses` and a list of direct prerequisite pairs `[a, b]` (course `a` must be taken before course `b`), answer a list of queries `[u, v]`: is `u` a prerequisite of `v`, directly or indirectly (transitively)?

---

## Approaches

1. **Brute Force — BFS/DFS Per Query**
2. **Optimal — Precompute Full Transitive Closure (Floyd-Warshall)**

---

# Approach 1 — Brute Force / BFS/DFS Per Query

## Idea

For each query `[u, v]`, run a fresh BFS (or DFS) starting from `u`, following prerequisite edges forward, and check whether `v` is ever reached.

## Dry Run

```text
numCourses = 5, prerequisites = [[0,1],[1,2],[2,3],[3,4]]
query = [0,4]
```

BFS from `0`:

```text
visit 0 → neighbors: 1
visit 1 → neighbors: 2
visit 2 → neighbors: 3
visit 3 → neighbors: 4
visit 4 → target found!
```

Query `[0,4]` → `true`.

## Algorithm

1. Build an adjacency list from `prerequisites`.
2. For each query `[u, v]`:

   * Run BFS (or DFS) from `u`, marking visited nodes.
   * If `v` is reached during the traversal, the answer is `true`; otherwise `false`.
3. Return the list of answers.

## Complexity

* **Time:** `O(q * (V + E))`

  * Each of the `q` queries triggers a fresh graph traversal that can visit up to every node and edge.
* **Space:** `O(V + E)`

  * For the adjacency list and the `visited` structure per query.

## Notes / Tips

* Re-traversing the graph from scratch for every query is wasteful when the same underlying reachability structure is being checked repeatedly — precomputing all pairwise reachability once (Approach 2) turns every query into an `O(1)` lookup.
* This approach becomes especially costly when `q` (number of queries) is large relative to the graph size.

## Code

```cpp
class Solution {
public:
    vector<bool> checkIfPrerequisite(int numCourses, vector<vector<int>>& prerequisites, vector<vector<int>>& queries) {
        vector<vector<int>> graph(numCourses);
        for (auto& p : prerequisites) {
            graph[p[0]].push_back(p[1]);
        }

        vector<bool> results;

        for (auto& q : queries) {
            int u = q[0], v = q[1];

            vector<bool> visited(numCourses, false);
            queue<int> bfsQueue;
            bfsQueue.push(u);
            visited[u] = true;
            bool found = false;

            while (!bfsQueue.empty() && !found) {
                int node = bfsQueue.front();
                bfsQueue.pop();

                if (node == v) {
                    found = true;
                    break;
                }

                for (int neighbor : graph[node]) {
                    if (!visited[neighbor]) {
                        visited[neighbor] = true;
                        bfsQueue.push(neighbor);
                    }
                }
            }

            results.push_back(found);
        }

        return results;
    }
};
```

---

# Approach 2 — Optimal / Precompute Full Transitive Closure (Floyd-Warshall)

## Idea

Instead of answering each query with a fresh search, precompute a full `numCourses x numCourses` reachability matrix once: `reachable[i][j] = true` if `i` is a prerequisite of `j` (directly or indirectly). Initialize it with the direct edges, then use a Floyd-Warshall-style triple loop: for every intermediate course `k`, if `i` can reach `k` and `k` can reach `j`, then `i` can reach `j` too. After this precomputation, every query is answered with a single array lookup.

## Dry Run

```text
numCourses = 4, prerequisites = [[0,1],[1,2],[2,3]]
```

Initialize `reachable` from direct edges:

```text
reachable[0][1] = true
reachable[1][2] = true
reachable[2][3] = true
(all other entries false)
```

Floyd-Warshall closure, trying each `k` as an intermediate:

```text
k=0: for i,j where reachable[i][0] and reachable[0][j] — none found (nothing reaches 0)
k=1: reachable[0][1]=true, reachable[1][2]=true → reachable[0][2] = true
k=2: reachable[0][2]=true (just set), reachable[2][3]=true → reachable[0][3] = true
     reachable[1][2]=true, reachable[2][3]=true → reachable[1][3] = true
k=3: no further extensions (nothing reaches 3 except what's already covered... actually reachable[i][3] combined with reachable[3][j] — 3 reaches nothing further)
```

Final `reachable`:

```text
0 reaches: 1, 2, 3
1 reaches: 2, 3
2 reaches: 3
3 reaches: (nothing)
```

Query `[0,3]` → `reachable[0][3] = true`. Query `[3,0]` → `reachable[3][0] = false`.

## Algorithm

1. Initialize a `reachable` matrix of size `numCourses x numCourses`, all `false`.
2. For each direct prerequisite `[a, b]`, set `reachable[a][b] = true`.
3. For each intermediate course `k` from `0` to `numCourses - 1`:

   * For each `i` from `0` to `numCourses - 1`:

     * For each `j` from `0` to `numCourses - 1`:

       * If `reachable[i][k]` and `reachable[k][j]`, set `reachable[i][j] = true`.
4. For each query `[u, v]`, look up `reachable[u][v]` directly.
5. Return the list of answers.

## Complexity

* **Time:** `O(n³)` for the precomputation, `O(1)` per query afterward

  * The triple nested loop over all `(i, j, k)` triples is the standard Floyd-Warshall transitive closure cost; each of the `q` queries is then a single array lookup.
* **Space:** `O(n²)`

  * For the `reachable` matrix.

## Notes / Tips

* This is the Floyd-Warshall algorithm applied to reachability (boolean OR/AND) instead of its more familiar use for shortest-path distances (min/plus), the same triple-loop structure, just with a different combining operation.
* This approach is a clean fit specifically because `numCourses` is small in this problem's constraints (`<= 100`), making `O(n³)` very manageable; for a much larger course count, a per-source BFS/DFS precomputation (`O(V * (V + E))` total, run once per starting node rather than per query) would scale better while still keeping each query at `O(1)`.
* The order of the triple loop matters — `k` must be the **outermost** loop for Floyd-Warshall-style closure to be correct, since `reachable[i][j]` may depend on paths through `k` that themselves depend on earlier values of `reachable[i][k]` and `reachable[k][j]` having already been finalized for smaller `k`.

## Code

```cpp
class Solution {
public:
    vector<bool> checkIfPrerequisite(int numCourses, vector<vector<int>>& prerequisites, vector<vector<int>>& queries) {
        vector<vector<bool>> reachable(numCourses, vector<bool>(numCourses, false));

        for (auto& p : prerequisites) {
            reachable[p[0]][p[1]] = true;
        }

        for (int k = 0; k < numCourses; k++) {
            for (int i = 0; i < numCourses; i++) {
                for (int j = 0; j < numCourses; j++) {
                    if (reachable[i][k] && reachable[k][j]) {
                        reachable[i][j] = true;
                    }
                }
            }
        }

        vector<bool> results;
        for (auto& q : queries) {
            results.push_back(reachable[q[0]][q[1]]);
        }

        return results;
    }
};
```

---

## Key Template

```text
reachable = matrix of size n x n, all false

for [a, b] in prerequisites:
    reachable[a][b] = true

for k in 0..n-1:
    for i in 0..n-1:
        for j in 0..n-1:
            if reachable[i][k] and reachable[k][j]:
                reachable[i][j] = true

for [u, v] in queries:
    answer = reachable[u][v]

return answers
```