# LeetCode 802 — Find Eventual Safe States

## Metadata

* **LeetCode:** 802
* **Problem:** Find Eventual Safe States
* **Difficulty:** Medium
* **Topics:** Depth-First Search, Breadth-First Search, Graph, Topological Sort
* **Pattern:** Cycle Detection via Three-State DFS Coloring, or Reverse-Graph Topological Sort (Kahn's Algorithm)
* **Key Technique:** A node is "safe" exactly when every path from it eventually leads to a terminal node without ever revisiting a node still being processed — this is the same three-state (unvisited/visiting/visited) cycle-detection coloring used in LC 207, or equivalently, safe nodes are exactly those that survive Kahn's algorithm when applied to the **reversed** graph, peeling away from terminal nodes inward
* **Optimal Complexity:** `O(V + E)` Time, `O(V + E)` Auxiliary Space

---

## Problem Statement

Given a directed graph `graph` (as an adjacency list), a node is a "terminal node" if it has no outgoing edges. A node is a "safe node" if every possible path starting from it eventually leads to a terminal node (i.e. it's never part of, or able to reach, a cycle). Return all safe nodes in ascending order.

---

## Approaches

1. **Brute Force — DFS With Depth Limit / Path Re-exploration**
2. **Better — DFS With Three-State Cycle Detection**
3. **Optimal — Kahn's Algorithm on the Reversed Graph**

---

# Approach 1 — Brute Force / DFS With Depth Limit / Path Re-exploration

## Idea

For each node, try to determine safety by exploring paths from it directly, without any memoization or proper cycle tracking — relying only on a bounded exploration depth (e.g. capped at `n` steps) to avoid truly infinite loops, and treating a node as unsafe if any explored path fails to terminate within that bound. This is a naive, unmemoized simulation of "does every path from here eventually stop."

## Dry Run

```text
graph = [[1,2],[2,3],[5],[0],[5],[],[]]
```

Explore from node `0`, capping depth at `n=7`:

```text
0 -> 1 -> 2 -> 3 -> 0 -> 1 -> 2 -> 3 (depth limit reached without termination)
```

Since the depth limit was hit without reaching a terminal node on this path, node `0` is (correctly, in this case) judged unsafe — but this conclusion was reached by exhausting a depth budget, not by actually detecting the cycle `0->1->2->3->0` directly.

## Algorithm

1. For each node `v`, run a DFS up to a fixed depth limit (e.g. `n`), following every outgoing edge.
2. If every explored path from `v` reaches a terminal node (no outgoing edges) within the depth limit, mark `v` safe.
3. If any path exhausts the depth limit without terminating, mark `v` unsafe.
4. Collect and return all safe nodes.

## Complexity

* **Time:** `O(V * (V + E))` in the best framing, but potentially much worse

  * Without memoization, the same sub-paths can be re-explored repeatedly from different starting nodes, and a depth-limited (rather than properly cycle-detecting) search can do redundant work up to the depth bound on every branch.
* **Space:** `O(V)`

  * For the recursion depth bound.

## Notes / Tips

* Using a depth limit instead of proper cycle detection is fragile and inefficient — it works only because the limit happens to be large enough to guarantee catching any cycle, but it does far more exploration than necessary and doesn't cleanly generalize.
* This approach mostly exists to illustrate why a proper "currently on this path" marker (three-state coloring, as in Approach 2) is the correct tool — it replaces "keep exploring until a depth budget runs out" with "immediately recognize a repeated node on the current path as proof of a cycle."

## Code

```cpp
class Solution {
public:
    bool exploreSafe(vector<vector<int>>& graph, int node, int depthLeft) {
        if (graph[node].empty()) {
            return true; // terminal node
        }

        if (depthLeft == 0) {
            return false; // treat as unsafe if we can't confirm termination
        }

        for (int neighbor : graph[node]) {
            if (!exploreSafe(graph, neighbor, depthLeft - 1)) {
                return false;
            }
        }

        return true;
    }

    vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<int> result;

        for (int i = 0; i < n; i++) {
            if (exploreSafe(graph, i, n)) {
                result.push_back(i);
            }
        }

        return result;
    }
};
```

---

# Approach 2 — Better / DFS With Three-State Cycle Detection

## Idea

Use the same three-state coloring technique as LC 207 (unvisited / visiting / visited) to properly and efficiently detect cycles. A node is safe if DFS from it never revisits a node currently marked "visiting" (which would mean a cycle was found). Memoize each node's final safe/unsafe result so it's only computed once, regardless of how many other nodes' explorations pass through it.

## Dry Run

```text
graph = [[1,2],[2,3],[5],[0],[5],[],[]]
```

DFS from node `0`:

```text
state[0] = visiting
explore neighbor 1: state[1] = visiting
    explore neighbor 2: state[2] = visiting
        explore neighbor 3: state[3] = visiting
            explore neighbor 0: state[0] = visiting → CYCLE DETECTED
            → node 3 is unsafe
        → node 2 is unsafe (depends on unsafe node 3)
    → node 1 is unsafe (depends on unsafe node 2)
→ node 0 is unsafe
```

DFS from node `4`:

```text
state[4] = visiting
explore neighbor 5: state[5] = visiting, no outgoing edges → terminal → safe
state[5] = visited (safe)
→ node 4 is safe (all neighbors safe)
```

Nodes `5` and `6` are terminal (no outgoing edges) → automatically safe. Final safe nodes: `[2, 4, 5, 6]`.

## Algorithm

1. Initialize a `state` array of size `n`, all `0` (unvisited), and a `safe` boolean array.
2. Define a recursive `dfs(node)`:

   * If `state[node] == 1` (visiting), return `false` (cycle found, unsafe).
   * If `state[node] == 2` (visited), return `safe[node]` (already resolved).
   * Set `state[node] = 1`.
   * For each neighbor, if `dfs(neighbor)` is `false`, this node is unsafe.
   * Set `state[node] = 2`, `safe[node] = (all neighbors were safe)`.
   * Return `safe[node]`.
3. Run `dfs` on every node, collect all nodes where `safe[node]` is `true`.

## Complexity

* **Time:** `O(V + E)`

  * Each node is fully resolved once (memoized), and each edge is examined once.
* **Space:** `O(V)`

  * For `state`, `safe`, and the recursion stack.

## Notes / Tips

* This is a direct, correct application of the same cycle-detection coloring used in LC 207 — the key addition here is that a node isn't just "cyclic or not," it's "safe or not," which depends recursively on whether *all* of its neighbors are safe, not just on the absence of a cycle through it directly.
* Memoizing each node's result (`state[node] == 2` short-circuits repeated exploration) is essential — without it, this degenerates back toward Approach 1's redundant re-exploration.

## Code

```cpp
class Solution {
public:
    vector<int> state;
    vector<bool> safe;

    bool dfs(vector<vector<int>>& graph, int node) {
        if (state[node] == 1) return false;
        if (state[node] == 2) return safe[node];

        state[node] = 1;

        for (int neighbor : graph[node]) {
            if (!dfs(graph, neighbor)) {
                state[node] = 2;
                safe[node] = false;
                return false;
            }
        }

        state[node] = 2;
        safe[node] = true;
        return true;
    }

    vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
        int n = graph.size();
        state.assign(n, 0);
        safe.assign(n, false);

        vector<int> result;
        for (int i = 0; i < n; i++) {
            if (dfs(graph, i)) {
                result.push_back(i);
            }
        }

        return result;
    }
};
```

---

# Approach 3 — Optimal / Kahn's Algorithm on the Reversed Graph

## Idea

A node is safe exactly when it can't reach a cycle — equivalently, a node is safe if it's part of the graph's "peel-away" structure starting from terminal nodes. Reverse every edge in the graph, and think of "outdegree" in the original graph as "indegree" in the reversed one. Run Kahn's algorithm on the reversed graph: start from nodes with `0` outgoing edges in the *original* graph (terminal nodes — indegree `0` in the reversed graph), and repeatedly peel away nodes whose "original outdegree" count drops to `0` as their dependents get processed. Every node that gets peeled away this way is safe.

## Dry Run

```text
graph = [[1,2],[2,3],[5],[0],[5],[],[]]
```

Reverse the graph (edge `u->v` becomes `v->u`) and compute each node's **outdegree** in the original graph:

```text
outdegree = [2, 1, 1, 1, 1, 0, 0]
reversedGraph: 1->[0], 2->[0,1], 3->[1], 0->[3], 5->[2,4], (6 has no incoming edges)
```

Start queue with nodes having original outdegree `0`: nodes `5` and `6`.

Process `5`:

```text
mark 5 safe
reversed neighbors of 5: [2, 4] → decrement their outdegree
outdegree[2] -= 1 → 0 → enqueue 2
outdegree[4] -= 1 → 0 → enqueue 4
```

Process `6`: no reversed neighbors, nothing to do.

Process `2`:

```text
mark 2 safe
reversed neighbors of 2: [0, 1] → decrement outdegree
outdegree[0] -= 1 → 1 (not yet 0)
outdegree[1] -= 1 → 0 → enqueue 1
```

Process `4`: mark safe, no reversed neighbors.

Process `1`:

```text
mark 1 safe
With `1`'s outdegree being `2` initially (to `2` and `3`), only one of those (`2`) gets resolved when `2` is processed, leaving `outdegree[1] = 1` still, so `1` is correctly **not** enqueued yet — it remains stuck (along with `0` and `3`, which form the actual cycle), never reaching outdegree `0`, and correctly never gets marked safe.

```

Final safe nodes (successfully peeled away): `2, 4, 5, 6` (sorted), matching Approach 2's result.

## Algorithm

1. Build the reversed graph: for each edge `u -> v` in the original graph, add an edge `v -> u` in the reversed graph.
2. Compute `outdegree[i]` for every node as the number of outgoing edges it has in the **original** graph.
3. Initialize a queue with every node whose `outdegree` is `0` (terminal nodes).
4. While the queue is non-empty:

   * Pop a node, mark it safe.
   * For each of its reversed-graph neighbors (i.e. nodes that originally pointed **to** it): decrement their `outdegree`; if it drops to `0`, enqueue them.
5. Collect all nodes marked safe, sorted ascending, and return them.

## Complexity

* **Time:** `O(V + E)`

  * Building the reversed graph and outdegree counts is `O(V + E)`; the BFS peel-away processes every node and edge once.
* **Space:** `O(V + E)`

  * For the reversed adjacency list, the outdegree array, and the BFS queue.

## Notes / Tips

* This is exactly Kahn's algorithm (as used in LC 207/210) but applied to the **reversed** graph with **outdegree** playing the role that indegree normally plays — safe nodes are precisely the ones reachable by "peeling inward" from terminal nodes, since any node stuck in or able to reach a cycle can never have all of its outgoing paths resolved this way.
* This BFS-based approach avoids recursion entirely, sidestepping any stack-overflow risk that Approach 2's recursive DFS could face on very deep or large graphs — a meaningful practical advantage at scale, similar to the DFS-vs-Kahn's trade-off seen in LC 207/210.
* Since the result must be returned in ascending order and nodes are typically labeled `0` to `n-1`, collecting safe nodes by scanning indices in order (rather than relying on BFS processing order) is the simplest way to guarantee correctly sorted output.

## Code

```cpp
class Solution {
public:
    vector<int> eventualSafeNodes(vector<vector<int>>& graph) {
        int n = graph.size();
        vector<vector<int>> reversedGraph(n);
        vector<int> outdegree(n, 0);

        for (int u = 0; u < n; u++) {
            outdegree[u] = graph[u].size();
            for (int v : graph[u]) {
                reversedGraph[v].push_back(u);
            }
        }

        queue<int> q;
        for (int i = 0; i < n; i++) {
            if (outdegree[i] == 0) {
                q.push(i);
            }
        }

        vector<bool> safe(n, false);

        while (!q.empty()) {
            int node = q.front();
            q.pop();
            safe[node] = true;

            for (int prev : reversedGraph[node]) {
                outdegree[prev]--;
                if (outdegree[prev] == 0) {
                    q.push(prev);
                }
            }
        }

        vector<int> result;
        for (int i = 0; i < n; i++) {
            if (safe[i]) {
                result.push_back(i);
            }
        }

        return result;
    }
};
```

---

## Key Template

```text
reversedGraph = reverse every edge of graph
outdegree[i] = graph[i].size() for every node

queue = all nodes with outdegree 0
safe = array of size n, all false

while queue not empty:
    node = queue.pop()
    safe[node] = true

    for prev in reversedGraph[node]:
        outdegree[prev] -= 1
        if outdegree[prev] == 0:
            queue.push(prev)

return sorted list of nodes where safe[node] is true
```