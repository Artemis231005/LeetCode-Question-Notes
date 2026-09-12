# LeetCode 200 — Number of Islands

## Metadata

* **LeetCode:** 200
* **Problem:** Number of Islands
* **Difficulty:** Medium
* **Topics:** Array, Depth-First Search, Breadth-First Search, Union Find, Matrix
* **Pattern:** Connected Components on a Grid (DFS/BFS or Union-Find)
* **Key Technique:** Treat each land cell as a graph node connected to its 4 neighbors, then count connected components by sinking/marking an entire island each time an unvisited land cell is found
* **Optimal Complexity:** `O(rows * cols)` Time, `O(rows * cols)` Auxiliary Space

---

## Problem Statement

Given an `m x n` 2D binary grid `grid` representing a map of `'1'`s (land) and `'0'`s (water), return the number of islands, where an island is formed by connecting adjacent lands horizontally or vertically.

---

## Approaches

1. **Brute Force — DFS/BFS Flood Fill to Mark Each Island**
2. **Optimal — Union-Find (Disjoint Set Union)**

---

# Approach 1 — Brute Force / DFS/BFS Flood Fill to Mark Each Island

## Idea

Scan every cell in the grid. Whenever an unvisited land cell (`'1'`) is found, it marks the start of a new island — run a DFS (or BFS) "flood fill" from that cell, marking every connected land cell (up/down/left/right) as visited so it's never counted again. Each time a fresh flood fill is started, increment the island count.

## Dry Run

```text
grid = [["1","1","0","0"],
        ["1","1","0","0"],
        ["0","0","1","0"],
        ["0","0","0","1"]]
```

Scan `(0,0)` — land, unvisited:

```text
flood fill: visits (0,0),(0,1),(1,0),(1,1) — all connected land
islands = 1
```

Scan `(0,1)`, `(1,0)`, `(1,1)` — already visited, skip.

Scan `(2,2)` — land, unvisited:

```text
flood fill: visits only (2,2), no connected land neighbors
islands = 2
```

Scan `(3,3)` — land, unvisited:

```text
flood fill: visits only (3,3)
islands = 3
```

Final: `3` islands.

## Algorithm

1. Initialize `islands = 0`.
2. For each cell `(r, c)` in the grid:

   * If `grid[r][c] == '1'` (unvisited land):

     * Increment `islands`.
     * Run DFS (or BFS) from `(r, c)`, marking every connected land cell as visited — either using a separate `visited` matrix, or by mutating the grid directly (e.g. setting visited land to `'0'`) to avoid extra space.
3. Return `islands`.

## Complexity

* **Time:** `O(rows * cols)`

  * Every cell is visited a constant number of times across all flood fills combined.
* **Space:** `O(rows * cols)`

  * For the recursion stack (DFS) or queue (BFS) in the worst case (e.g. the entire grid is one connected island), plus a `visited` matrix if the grid itself isn't mutated in place.

## Notes / Tips

* Mutating the grid in place (flipping visited `'1'`s to `'0'`s) avoids needing a separate `visited` matrix, trading input mutation for space savings — only do this if modifying the input is acceptable.
* Recursive DFS can risk a stack overflow on very large grids that form one giant connected island; an iterative BFS with an explicit queue (or an iterative DFS with an explicit stack) avoids that risk entirely.
* This flood-fill technique is the standard approach for "count connected regions in a grid" problems — the same shape solves "Max Area of Island," "Surrounded Regions," and similar grid-traversal problems.

## Code

```cpp
class Solution {
public:
    int rows, cols;

    void dfs(vector<vector<char>>& grid, int r, int c) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || grid[r][c] != '1') {
            return;
        }

        grid[r][c] = '0';

        dfs(grid, r + 1, c);
        dfs(grid, r - 1, c);
        dfs(grid, r, c + 1);
        dfs(grid, r, c - 1);
    }

    int numIslands(vector<vector<char>>& grid) {
        rows = grid.size();
        cols = grid[0].size();
        int islands = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    islands++;
                    dfs(grid, r, c);
                }
            }
        }

        return islands;
    }
};
```

---

# Approach 2 — Optimal / Union-Find (Disjoint Set Union)

## Idea

Treat each land cell as its own group initially, using its flattened 1D index (`row * cols + col`) as its identity. Scan every land cell, and for each one, union it with its land neighbors to the right and below (checking only these two directions is enough to eventually connect the whole grid, since scanning left-to-right, top-to-bottom already covers every adjacent pair exactly once). After processing all cells, count the number of distinct roots among land cells.

## Dry Run

```text
grid = [["1","1","0","0"],
        ["1","1","0","0"],
        ["0","0","1","0"],
        ["0","0","0","1"]]
```

Initialize each land cell as its own parent (water cells ignored).

Process `(0,0)`:

```text
right neighbor (0,1) is land → union(0*4+0, 0*4+1) = union(0, 1)
below neighbor (1,0) is land → union(0, 1*4+0) = union(0, 4)
```

Process `(0,1)`:

```text
right neighbor (0,2) is water → skip
below neighbor (1,1) is land → union(1, 1*4+1) = union(1, 5)
```

Process `(1,0)`:

```text
right neighbor (1,1) is land → union(4, 5)
below neighbor (2,0) is water → skip
```

Process `(1,1)`:

```text
neighbors (1,2) and (2,1) are water → no unions
```

Process `(2,2)`:

```text
neighbors (2,3) and (3,2) are water → no unions, stays its own group
```

Process `(3,3)`:

```text
no valid right/below neighbors (edge of grid) → stays its own group
```

Distinct roots among land cells: one root for `{(0,0),(0,1),(1,0),(1,1)}`, one for `{(2,2)}`, one for `{(3,3)}` → `3` islands.

## Algorithm

1. Initialize a `parent` array sized `rows * cols`, where each land cell's flattened index is its own parent, and water cells are ignored (or marked invalid).
2. For each cell `(r, c)`:

   * If `grid[r][c] == '1'`:

     * If the cell to the right is land, union their flattened indices.
     * If the cell below is land, union their flattened indices.
3. Count the number of distinct roots among all land cells (calling `find` on each land cell's index and counting unique results).
4. Return that count.

## Complexity

* **Time:** `O(rows * cols * α(rows * cols))`

  * Every cell is processed once, and each union/find operation is nearly `O(1)` on average with path compression and union by rank (`α` is the inverse Ackermann function, effectively constant).
* **Space:** `O(rows * cols)`

  * For the `parent` (and optional `rank`) arrays, sized to the total number of cells.

## Notes / Tips

* Only checking the right and below neighbors (not all 4 directions) during the union step is sufficient and avoids redundant union calls — every adjacent pair in the grid gets covered exactly once this way as the scan proceeds left-to-right, top-to-bottom.
* For this specific problem, Union-Find doesn't  beat DFS/BFS flood fill — both are `O(rows * cols)`. Union-Find becomes more clearly useful in problems where the grid changes dynamically over time (e.g. land being added incrementally, as in "Number of Islands II") and connectivity needs to be tracked incrementally rather than recomputed from scratch.
* Flattening 2D coordinates into a 1D index (`row * cols + col`) is the standard trick for applying Union-Find (which is inherently 1D) to a grid-based problem.

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

    int numIslands(vector<vector<char>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        parent.resize(rows * cols);

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                parent[r * cols + c] = r * cols + c;
            }
        }

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    if (c + 1 < cols && grid[r][c + 1] == '1') {
                        unite(r * cols + c, r * cols + (c + 1));
                    }
                    if (r + 1 < rows && grid[r + 1][c] == '1') {
                        unite(r * cols + c, (r + 1) * cols + c);
                    }
                }
            }
        }

        set<int> roots;
        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == '1') {
                    roots.insert(find(r * cols + c));
                }
            }
        }

        return roots.size();
    }
};
```

---

## Key Template

```text
parent[i] = i for every land cell's flattened index

function find(x):
    if parent[x] != x:
        parent[x] = find(parent[x])
    return parent[x]

function unite(x, y):
    rootX = find(x)
    rootY = find(y)
    if rootX != rootY:
        parent[rootX] = rootY

for each land cell (r, c):
    if right neighbor is land: unite(index(r,c), index(r,c+1))
    if below neighbor is land: unite(index(r,c), index(r+1,c))

count distinct find(index(r,c)) over all land cells
return count
```