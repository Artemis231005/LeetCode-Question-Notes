# LeetCode 3898 — Find the Degree of Each Vertex

## Metadata

* **LeetCode:** 3898
* **Problem:** Find the Degree of Each Vertex
* **Difficulty:** Easy
* **Topics:** Array, Matrix, Graph
* **Pattern:** Row Sum on an Adjacency Matrix
* **Key Technique:** A vertex's degree is exactly the count of `1`s in its own row of the adjacency matrix — no need to build an edge list or scan anything beyond that one row
* **Optimal Complexity:** `O(n²)` Time, `O(1)` Auxiliary Space (excluding the output)

---

## Problem Statement

Given an `n x n` adjacency matrix `matrix` for an undirected graph (`matrix[i][j] = 1` if an edge exists between `i` and `j`), return an array `ans` where `ans[i]` is the degree of vertex `i` (the number of edges connected to it).

---

## Approaches

1. **Brute Force — Build an Edge List, Then Count Per Vertex**
2. **Optimal — Direct Row Sum**

---

# Approach 1 — Brute Force / Build an Edge List, Then Count Per Vertex

## Idea

First scan the entire matrix once to extract every edge `(i, j)` where `matrix[i][j] == 1` (only counting each undirected edge once, e.g. when `i < j`), storing them in a list. Then, for each vertex, scan through this entire edge list and count how many edges touch it.

## Dry Run

```text
matrix = [[0,1,1],[1,0,1],[1,1,0]]
```

Build edge list (scanning upper triangle, `i < j`):

```text
(0,1), (0,2), (1,2)
```

Count edges touching vertex `0`:

```text
(0,1) touches 0 → count
(0,2) touches 0 → count
(1,2) doesn't touch 0
degree[0] = 2
```

Count edges touching vertex `1`:

```text
(0,1) touches 1 → count
(1,2) touches 1 → count
degree[1] = 2
```

Count edges touching vertex `2`:

```text
(0,2) touches 2 → count
(1,2) touches 2 → count
degree[2] = 2
```

Final: `[2, 2, 2]`, matching the expected output.

## Algorithm

1. Scan the matrix's upper triangle (`i < j`) to build a list of edges where `matrix[i][j] == 1`.
2. For each vertex `v` from `0` to `n-1`:

   * Scan the entire edge list, counting how many edges have `v` as either endpoint.
3. Return the resulting degree array.

## Complexity

* **Time:** `O(n³)`

  * Building the edge list takes `O(n²)`; for each of the `n` vertices, scanning the full edge list (which can hold up to `O(n²)` edges) takes `O(n²)`, giving `O(n³)` overall.
* **Space:** `O(n²)`

  * For storing the edge list.

## Notes / Tips

* Building an intermediate edge list and then re-scanning it per vertex is unnecessary work — the adjacency matrix already stores, in row `i`, exactly the information needed to compute vertex `i`'s degree directly.
* This detour through an edge list is a common instinct when thinking of the input as "a graph" rather than "a matrix," but the matrix's own structure already answers the question without needing that intermediate representation.

## Code

```cpp
class Solution {
public:
    vector<int> findDegree(vector<vector<int>>& matrix) {
        int n = matrix.size();
        vector<pair<int,int>> edges;

        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j < n; j++) {
                if (matrix[i][j] == 1) {
                    edges.push_back({i, j});
                }
            }
        }

        vector<int> ans(n, 0);
        for (int v = 0; v < n; v++) {
            for (auto& [a, b] : edges) {
                if (a == v || b == v) {
                    ans[v]++;
                }
            }
        }

        return ans;
    }
};
```

---

# Approach 2 — Optimal / Direct Row Sum

## Idea

The degree of vertex `i` is, by definition, the number of `1`s in row `i` of the adjacency matrix — each `1` in that row directly represents one edge incident to `i`. No edge list, graph structure, or column checks are needed; the matrix already stores exactly this count, one row per vertex.

## Dry Run

```text
matrix = [[0,1,0],[1,0,0],[0,0,0]]
```

Row `0`: `[0,1,0]` → sum = `1` → `ans[0] = 1`.
Row `1`: `[1,0,0]` → sum = `1` → `ans[1] = 1`.
Row `2`: `[0,0,0]` → sum = `0` → `ans[2] = 0`.

Final: `[1, 1, 0]`, matching the expected output.

## Algorithm

1. Initialize `ans` array of size `n`.
2. For each row `i` from `0` to `n-1`:

   * Sum all values in `matrix[i]`.
   * Store that sum in `ans[i]`.
3. Return `ans`.

## Complexity

* **Time:** `O(n²)`

  * Each of the `n` rows is summed in `O(n)`.
* **Space:** `O(1)` auxiliary (beyond the required output)

  * No extra structures beyond the output array — the sum is accumulated directly from the input matrix.

## Notes / Tips

* Since the matrix is guaranteed symmetric (`matrix[i][j] == matrix[j][i]`) and has zero diagonal (`matrix[i][i] == 0`), summing row `i` alone is both correct and sufficient — there's no need to also check column `i`, since any edge `(i,j)` is already captured by `matrix[i][j]` in row `i`.
* This is about as direct a translation of the problem's definition as possible: "degree = count of edges touching a vertex" maps exactly onto "count of `1`s in that vertex's row," with no intermediate graph representation needed.
* Given the matrix format already encodes the graph completely, this problem is a good reminder to look for whether an input's existing structure directly answers the question before reaching for a more general graph-processing technique (adjacency lists, edge lists, traversals, etc.).

## Code

```cpp
class Solution {
public:
    vector<int> findDegree(vector<vector<int>>& matrix) {
        int n = matrix.size();
        vector<int> ans(n, 0);

        for (int i = 0; i < n; i++) {
            int degree = 0;
            for (int j = 0; j < n; j++) {
                degree += matrix[i][j];
            }
            ans[i] = degree;
        }

        return ans;
    }
};
```

---

## Key Template

```text
ans = array of size n

for i in 0..n-1:
    degree = 0
    for j in 0..n-1:
        degree += matrix[i][j]
    ans[i] = degree

return ans
```