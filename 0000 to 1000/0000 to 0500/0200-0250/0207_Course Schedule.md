# LeetCode 207 — Course Schedule

## Metadata

* **LeetCode:** 207
* **Problem:** Course Schedule
* **Difficulty:** Medium
* **Topics:** Depth-First Search, Breadth-First Search, Graph, Topological Sort
* **Pattern:** Cycle Detection in a Directed Graph (DFS Coloring or Kahn's BFS)
* **Key Technique:** A valid course order exists exactly when the prerequisite graph has no cycle — detect this either via DFS with a "currently in recursion stack" marker, or via Kahn's algorithm (repeatedly removing zero-indegree nodes)
* **Optimal Complexity:** `O(V + E)` Time, `O(V + E)` Auxiliary Space

---

## Problem Statement

Given `numCourses` and a list of prerequisite pairs `[a, b]` (meaning course `b` must be taken before course `a`), return `true` if it's possible to finish all courses (i.e. the prerequisite graph has no cycle).

---

## Approaches

1. **Brute Force — Try Every Course Order via Backtracking**
2. **Better — DFS Cycle Detection with Three-State Coloring**
3. **Optimal — Kahn's Algorithm (BFS Topological Sort)**

---

# Approach 1 — Brute Force / Try Every Course Order via Backtracking

## Idea

Try to build a valid course order one course at a time: at each step, pick any course whose prerequisites have all already been taken, mark it taken, and recurse. Backtrack if a choice leads to a dead end (no remaining course has its prerequisites satisfied, but courses remain). If any full ordering succeeds, all courses can be finished.

## Dry Run

```text
numCourses = 3, prerequisites = [[1,0],[2,1]]
```

Try taking course `0` first (no prerequisites):

```text
taken = {0}
```

Try taking course `1` (prerequisite `0` satisfied):

```text
taken = {0, 1}
```

Try taking course `2` (prerequisite `1` satisfied):

```text
taken = {0, 1, 2}
```

All 3 courses taken → return `true`.

## Algorithm

1. Build an adjacency structure mapping each course to its list of prerequisites.
2. Define a recursive `tryOrder(taken)`:

   * If `taken.size() == numCourses`, return `true`.
   * For each untaken course whose prerequisites are all in `taken`:

     * Add it to `taken`, recurse.
     * If the recursive call succeeds, return `true`; otherwise, backtrack (remove it from `taken`).
   * If no course can be added, return `false`.
3. Return the result of `tryOrder({})`.

## Complexity

* **Time:** `O(V!)` in the worst case

  * Exploring every possible valid ordering of courses is factorial in the number of courses without any pruning beyond prerequisite satisfaction.
* **Space:** `O(V)`

  * For the recursion stack and the `taken` set.

## Notes / Tips

* This approach massively overcomplicates the problem — the question only asks *whether* all courses can be finished, not to produce every possible valid ordering, so there's no need to explore multiple orderings once any cycle-free structure is confirmed.
* Detecting a cycle directly (Approach 2) or repeatedly removing free-standing nodes (Approach 3) both answer the yes/no question in linear time, without ever needing to enumerate orderings.

## Code

```cpp
class Solution {
public:
    bool tryOrder(int numCourses, vector<vector<int>>& prereqOf, vector<bool>& taken, int count) {
        if (count == numCourses) {
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
                if (tryOrder(numCourses, prereqOf, taken, count + 1)) {
                    return true;
                }
                taken[course] = false;
            }
        }

        return false;
    }

    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        vector<vector<int>> prereqOf(numCourses);
        for (auto& p : prerequisites) {
            prereqOf[p[0]].push_back(p[1]);
        }

        vector<bool> taken(numCourses, false);
        return tryOrder(numCourses, prereqOf, taken, 0);
    }
};
```

---

# Approach 2 — Better / DFS Cycle Detection with Three-State Coloring

## Idea

Build a directed graph from each course to the courses that depend on it (or from each course to its prerequisites — direction just needs to be consistent). Run DFS from every unvisited course, tracking each node's state as **unvisited**, **visiting** (currently on the current DFS path), or **visited** (fully processed, confirmed safe). If DFS ever reaches a node that's already **visiting**, a cycle has been found — courses can't be finished.

## Dry Run

```text
numCourses = 3, prerequisites = [[1,0],[2,1]]
```

Graph (course → its prerequisites): `1 → [0]`, `2 → [1]`.

DFS from course `0`:

```text
state[0] = visiting
0 has no prerequisites → state[0] = visited
```

DFS from course `1`:

```text
state[1] = visiting
1's prerequisite 0: state[0] = visited → no cycle → continue
state[1] = visited
```

DFS from course `2`:

```text
state[2] = visiting
2's prerequisite 1: state[1] = visited → no cycle → continue
state[2] = visited
```

No cycle found across any DFS → return `true`.

### Cycle example

```text
prerequisites = [[1,0],[0,1]]
```

DFS from course `0`:

```text
state[0] = visiting
0's prerequisite 1: state[1] = unvisited → recurse into 1
    state[1] = visiting
    1's prerequisite 0: state[0] = visiting → CYCLE DETECTED → return false
```

## Algorithm

1. Build an adjacency list mapping each course to its list of prerequisites.
2. Initialize a `state` array of size `numCourses`, all `0` (unvisited).
3. Define a recursive `hasCycle(course)`:

   * If `state[course] == 1` (visiting), return `true` (cycle found).
   * If `state[course] == 2` (visited), return `false` (already confirmed safe).
   * Set `state[course] = 1`.
   * For each prerequisite of `course`, if `hasCycle(prerequisite)` returns `true`, propagate `true`.
   * Set `state[course] = 2`.
   * Return `false`.
4. For each course, if `hasCycle(course)` returns `true`, return `false` overall.
5. If no cycle is found across any course, return `true`.

## Complexity

* **Time:** `O(V + E)`

  * Each node is fully processed once (transitioning through all three states), and each edge is examined once.
* **Space:** `O(V + E)`

  * For the adjacency list and the `state` array, plus the recursion stack (up to `O(V)` deep).

## Notes / Tips

* The three-state coloring (unvisited / visiting / visited) is what distinguishes a genuine cycle from simply revisiting a node through a different path, a plain two-state (visited/unvisited) marker would incorrectly flag any diamond-shaped dependency structure (where two courses share a common prerequisite) as a cycle.
* This is the standard cycle-detection template for directed graphs — the same "visiting" marker technique detects cycles in dependency graphs, build systems, and any other "must happen before" ordering problem.
* Recursive DFS can risk a stack overflow on very deep prerequisite chains, an iterative version with an explicit stack avoids that risk on large inputs.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> graph;
    vector<int> state;

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
        return false;
    }

    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
        graph.assign(numCourses, {});
        for (auto& p : prerequisites) {
            graph[p[0]].push_back(p[1]);
        }

        state.assign(numCourses, 0);

        for (int course = 0; course < numCourses; course++) {
            if (hasCycle(course)) {
                return false;
            }
        }

        return true;
    }
};
```

---

# Approach 3 — Optimal / Kahn's Algorithm (BFS Topological Sort)

## Idea

Build the graph in the "prerequisite → dependent" direction, and compute each course's **indegree** (number of prerequisites it still needs). Start a BFS from every course with indegree `0` (no prerequisites — safe to take immediately). Each time a course is "taken," decrement the indegree of every course that depends on it; whenever a dependent's indegree drops to `0`, it becomes takeable and is enqueued. If every course eventually gets taken this way, there's no cycle; if some courses are never reached (indegree never drops to `0`), a cycle exists among them.

## Dry Run

```text
numCourses = 3, prerequisites = [[1,0],[2,1]]
```

Graph (prerequisite → dependent): `0 → [1]`, `1 → [2]`.
Indegree: `indegree[0]=0, indegree[1]=1, indegree[2]=1`.

Start queue with all indegree-`0` courses: `[0]`.

Process `0`:

```text
taken count = 1
neighbor 1: indegree[1] -= 1 → 0 → enqueue 1
queue = [1]
```

Process `1`:

```text
taken count = 2
neighbor 2: indegree[2] -= 1 → 0 → enqueue 2
queue = [2]
```

Process `2`:

```text
taken count = 3
no neighbors
queue = []
```

`taken count (3) == numCourses (3)` → return `true`.

### Cycle example

```text
prerequisites = [[1,0],[0,1]]
```

Indegree: `indegree[0]=1, indegree[1]=1`. No course starts at indegree `0` → queue starts empty → `taken count = 0 != numCourses (2)` → return `false`.

## Algorithm

1. Build an adjacency list mapping each course to the courses that depend on it (`prerequisite → dependent`), and compute each course's indegree (number of prerequisites).
2. Initialize a queue with every course whose indegree is `0`.
3. Initialize `taken = 0`.
4. While the queue is non-empty:

   * Pop a course, increment `taken`.
   * For each course that depends on it, decrement that course's indegree; if it drops to `0`, enqueue it.
5. Return `taken == numCourses`.

## Complexity

* **Time:** `O(V + E)`

  * Every node is enqueued and processed once, and every edge is examined once when decrementing indegrees.
* **Space:** `O(V + E)`

  * For the adjacency list, the indegree array, and the BFS queue.

## Notes / Tips

* This is Kahn's algorithm for topological sorting,beyond just detecting a cycle (as this problem asks), it naturally produces a valid course order as a side effect (the sequence in which courses are dequeued), which is exactly what LC 210 (Course Schedule II) asks for directly.
* No recursion is used here, which avoids any stack-overflow risk on deep dependency chains that Approach 2's recursive DFS could face, an  advantage even though both approaches share the same asymptotic complexity.
* The core insight — "a cycle exists if and only if some nodes never reach indegree 0" — is what makes the final `taken == numCourses` check sufficient to detect cycles without any separate cycle-specific logic.

## Code

```cpp
class Solution {
public:
    bool canFinish(int numCourses, vector<vector<int>>& prerequisites) {
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

        int taken = 0;
        while (!q.empty()) {
            int course = q.front();
            q.pop();
            taken++;

            for (int dependent : graph[course]) {
                indegree[dependent]--;
                if (indegree[dependent] == 0) {
                    q.push(dependent);
                }
            }
        }

        return taken == numCourses;
    }
};
```

---

## Key Template

```text
graph = adjacency list, prereq -> dependent
indegree = count of prerequisites per course

queue = all courses with indegree 0
taken = 0

while queue not empty:
    course = queue.pop()
    taken += 1

    for dependent in graph[course]:
        indegree[dependent] -= 1
        if indegree[dependent] == 0:
            queue.push(dependent)

return taken == numCourses
```