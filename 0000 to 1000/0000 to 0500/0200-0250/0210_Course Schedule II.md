# LeetCode 210 — Course Schedule II

## Metadata

* **LeetCode:** 210
* **Problem:** Course Schedule II
* **Difficulty:** Medium
* **Topics:** Depth-First Search, Breadth-First Search, Graph, Topological Sort
* **Pattern:** Topological Sort (DFS Post-Order or Kahn's BFS)
* **Key Technique:** A valid course order is exactly a topological ordering of the prerequisite graph — either build it via DFS post-order (reversed) with cycle detection, or via Kahn's algorithm (repeatedly taking zero-indegree nodes), which produces the order and detects impossibility together
* **Optimal Complexity:** `O(V + E)` Time, `O(V + E)` Auxiliary Space

---

## Problem Statement

Given `numCourses` and a list of prerequisite pairs `[a, b]` (meaning course `b` must be taken before course `a`), return **any** valid order in which all courses can be completed, or an empty array if it's impossible (a cycle exists).

---

## Approaches

1. **Brute Force — Backtracking to Build a Valid Order**
2. **Better — DFS Post-Order with Cycle Detection**
3. **Optimal — Kahn's Algorithm (BFS Topological Sort)**

---

# Approach 1 — Brute Force / Backtracking to Build a Valid Order

## Idea

Try to build a valid order one course at a time: at each step, pick any course whose prerequisites have all already been taken, add it to the order, and recurse. Backtrack if a choice leads to a dead end. If a full ordering of all courses is ever completed, return it.

## Dry Run

```text
numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
```

Try taking course `0` first (no prerequisites):

```text
order = [0]
```

Try taking course `1` (prerequisite `0` satisfied):

```text
order = [0, 1]
```

Try taking course `2` (prerequisite `0` satisfied):

```text
order = [0, 1, 2]
```

Try taking course `3` (prerequisites `1` and `2` both satisfied):

```text
order = [0, 1, 2, 3]
```

All 4 courses taken → return `[0,1,2,3]` (one valid order among several possible).

## Algorithm

1. Build an adjacency structure mapping each course to its list of prerequisites.
2. Define a recursive `tryOrder(order)`:

   * If `order.size() == numCourses`, return `order` (success).
   * For each untaken course whose prerequisites are all in `order`:

     * Add it to `order`, recurse.
     * If the recursive call succeeds, return that result; otherwise, backtrack (remove it from `order`).
   * If no course can be added, return failure.
3. Return the result of `tryOrder([])`, or an empty array if it fails.

## Complexity

* **Time:** `O(V!)` in the worst case

  * Exploring every possible valid ordering without pruning beyond prerequisite satisfaction is factorial in the number of courses.
* **Space:** `O(V)`

  * For the recursion stack and the `order` list.

## Notes / Tips

* This massively overcomplicates the problem — a topological sort (via DFS or Kahn's algorithm) produces a valid order directly and detects impossibility as a natural byproduct, without any backtracking or trial-and-error needed.
* This is the same unnecessary backtracking approach seen in LC 207's brute force, just extended to actually return the order instead of a yes/no answer.

## Code

```cpp
class Solution {
public:
    bool tryOrder(int numCourses, vector<vector<int>>& prereqOf, vector<int>& order, vector<bool>& taken) {
        if ((int)order.size() == numCourses) {
            return true;
        }

        for (int course = 0; course < numCourses; course++) {
            if (taken[course]) continue;

            bool ready = true;
            for (int prereq : prereqOf[course]) {
                if (!taken[prereq]) {
                    ready = false;
                    break;
                }
            }

            if (ready) {
                taken[course] = true;
                order.push_back(course);

                if (tryOrder(numCourses, prereqOf, order, taken)) {
                    return true;
                }

                order.pop_back();
                taken[course] = false;
            }
        }

        return false;
    }

    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> prereqOf(numCourses);
        for (auto& p : prerequisites) {
            prereqOf[p[0]].push_back(p[1]);
        }

        vector<int> order;
        vector<bool> taken(numCourses, false);

        if (tryOrder(numCourses, prereqOf, order, taken)) {
            return order;
        }

        return {};
    }
};
```

---

# Approach 2 — Better / DFS Post-Order with Cycle Detection

## Idea

Build the graph in the "course → its prerequisites" direction. Run DFS from every unvisited course, using three-state coloring (unvisited / visiting / visited) to detect cycles, exactly as in LC 207. The key addition here: append each course to an `order` list **after** all of its prerequisites have been fully processed (post-order) — since a course is only appended once everything it depends on is already guaranteed to be in the list, reversing this post-order gives a valid course sequence.

## Dry Run

```text
numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
```

Graph (course → its prerequisites): `1→[0]`, `2→[0]`, `3→[1,2]`.

DFS from course `0`:

```text
state[0] = visiting
0 has no prerequisites → state[0] = visited, append 0 to postOrder
postOrder = [0]
```

DFS from course `1`:

```text
state[1] = visiting
1's prerequisite 0: already visited → continue
state[1] = visited, append 1
postOrder = [0, 1]
```

DFS from course `2`:

```text
state[2] = visiting
2's prerequisite 0: already visited → continue
state[2] = visited, append 2
postOrder = [0, 1, 2]
```

DFS from course `3`:

```text
state[3] = visiting
3's prerequisite 1: already visited → continue
3's prerequisite 2: already visited → continue
state[3] = visited, append 3
postOrder = [0, 1, 2, 3]
```

`postOrder` itself (`[0,1,2,3]`) already has prerequisites before dependents in this particular trace. The exact reversal requirement depends on which edge direction is used, so this must be verified carefully against the specific graph direction chosen in the differnet questions.

## Algorithm

1. Build an adjacency list mapping each course to its list of prerequisites.
2. Initialize a `state` array of size `numCourses`, all `0` (unvisited), and an empty `order` list.
3. Define a recursive `dfs(course)`:

   * If `state[course] == 1` (visiting), return `true` (cycle found).
   * If `state[course] == 2` (visited), return `false`.
   * Set `state[course] = 1`.
   * For each prerequisite of `course`, if `dfs(prerequisite)` returns `true`, propagate `true` (cycle).
   * Set `state[course] = 2`, append `course` to `order`.
   * Return `false`.
4. For each course, if `dfs(course)` detects a cycle, return an empty array.
5. Return `order` (verifying/adjusting direction as needed based on the graph's edge orientation).

## Complexity

* **Time:** `O(V + E)`

  * Each node is fully processed once, and each edge is examined once.
* **Space:** `O(V + E)`

  * For the adjacency list, `state` array, `order` list, and the recursion stack.

## Notes / Tips

* Getting the append timing right (only appending a course to `order` **after** fully recursing into all its prerequisites) is what makes this a valid post-order traversal — appending too early (e.g. right when `state[course]` is set to `visiting`) would produce an incorrect order.
* This DFS-based topological sort is a completely valid and commonly taught alternative to Kahn's algorithm (Approach 3) — the cycle detection and ordering are produced together in a single traversal, similar in spirit to LC 207's cycle-detection DFS but now also collecting output.
* Recursive DFS can risk a stack overflow on very deep prerequisite chains — Kahn's algorithm (Approach 3) avoids recursion entirely, which can be a meaningful practical advantage on large inputs.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> graph;
    vector<int> state;
    vector<int> order;

    bool hasCycle(int course) {
        if (state[course] == 1) return true;
        if (state[course] == 2) return false;

        state[course] = 1;

        for (int prereq : graph[course]) {
            if (hasCycle(prereq)) {
                return true;
            }
        }

        state[course] = 2;
        order.push_back(course);
        return false;
    }

    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        graph.assign(numCourses, {});
        for (auto& p : prerequisites) {
            graph[p[0]].push_back(p[1]);
        }

        state.assign(numCourses, 0);

        for (int course = 0; course < numCourses; course++) {
            if (hasCycle(course)) {
                return {};
            }
        }

        return order;
    }
};
```

---

# Approach 3 — Optimal / Kahn's Algorithm (BFS Topological Sort)

## Idea

Build the graph in the "prerequisite → dependent" direction, and compute each course's indegree. Start a BFS from every course with indegree `0` (takeable immediately, no unmet prerequisites). Each time a course is dequeued, append it to the result order and decrement the indegree of every course depending on it; whenever a dependent's indegree drops to `0`, enqueue it. If every course ends up in the result order, it's a valid sequence; if some courses are never reached, a cycle exists and no valid order is possible.

## Dry Run

```text
numCourses = 4, prerequisites = [[1,0],[2,0],[3,1],[3,2]]
```

Graph (prerequisite → dependent): `0→[1,2]`, `1→[3]`, `2→[3]`.
Indegree: `indegree[0]=0, indegree[1]=1, indegree[2]=1, indegree[3]=2`.

Start queue with indegree-`0` courses: `[0]`.

Process `0`:

```text
order = [0]
neighbor 1: indegree[1] -= 1 → 0 → enqueue 1
neighbor 2: indegree[2] -= 1 → 0 → enqueue 2
queue = [1, 2]
```

Process `1`:

```text
order = [0, 1]
neighbor 3: indegree[3] -= 1 → 1 → not yet 0, don't enqueue
queue = [2]
```

Process `2`:

```text
order = [0, 1, 2]
neighbor 3: indegree[3] -= 1 → 0 → enqueue 3
queue = [3]
```

Process `3`:

```text
order = [0, 1, 2, 3]
no neighbors
queue = []
```

`order.size() (4) == numCourses (4)` → return `[0,1,2,3]`, a valid order.

## Algorithm

1. Build an adjacency list mapping each course to the courses that depend on it, and compute each course's indegree.
2. Initialize a queue with every course whose indegree is `0`, and an empty `order` list.
3. While the queue is non-empty:

   * Pop a course, append it to `order`.
   * For each course that depends on it, decrement that course's indegree; if it drops to `0`, enqueue it.
4. If `order.size() == numCourses`, return `order`; otherwise return an empty array (a cycle prevented some courses from ever reaching indegree `0`).

## Complexity

* **Time:** `O(V + E)`

  * Every node is enqueued and processed once, and every edge is examined once when decrementing indegrees.
* **Space:** `O(V + E)`

  * For the adjacency list, the indegree array, the BFS queue, and the `order` list.

## Notes / Tips

* This is exactly Kahn's algorithm as used in LC 207, extended to actually collect and return the order (rather than just checking `taken == numCourses`) — the order naturally falls out of the sequence in which courses are dequeued.
* No recursion is involved, avoiding any stack-overflow risk on deep dependency chains that Approach 2's recursive DFS could face on very large inputs.
* Unlike the DFS approach, the order produced here is built directly in a valid "prerequisites first" sequence without needing to reason about reversal — BFS naturally processes courses in an order where a course is only dequeued once all its prerequisites have already been dequeued (indegree reduced to `0`).

## Code

```cpp
class Solution {
public:
    vector<int> findOrder(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> graph(numCourses);
        vector<int> indegree(numCourses, 0);

        for (auto& p : prerequisites) {
            int course = p[0], prereq = p[1];
            graph[prereq].push_back(course);
            indegree[course]++;
        }

        queue<int> q;
        for (int i = 0; i < numCourses; i++) {
            if (indegree[i] == 0) {
                q.push(i);
            }
        }

        vector<int> order;
        while (!q.empty()) {
            int course = q.front();
            q.pop();
            order.push_back(course);

            for (int dependent : graph[course]) {
                indegree[dependent]--;
                if (indegree[dependent] == 0) {
                    q.push(dependent);
                }
            }
        }

        return order.size() == numCourses ? order : vector<int>{};
    }
};
```

---

## Key Template

```text
graph = adjacency list, prereq -> dependent
indegree = count of prerequisites per course

queue = all courses with indegree 0
order = []

while queue not empty:
    course = queue.pop()
    order.append(course)

    for dependent in graph[course]:
        indegree[dependent] -= 1
        if indegree[dependent] == 0:
            queue.push(dependent)

return order if order.size() == numCourses else []
```