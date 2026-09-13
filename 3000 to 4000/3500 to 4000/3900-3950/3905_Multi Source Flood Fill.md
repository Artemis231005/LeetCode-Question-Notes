# LeetCode 3905 — Multi Source Flood Fill

## Metadata

* **LeetCode:** 3905
* **Problem:** Multi Source Flood Fill
* **Difficulty:** Medium
* **Topics:** Array, Breadth-First Search, Matrix
* **Pattern:** Multi-Source BFS with Simultaneous Tie-Breaking
* **Key Technique:** Process the grid level by level like a standard multi-source BFS, but collect **all** candidate colors reaching each uncolored cell within the current level before committing the maximum, so simultaneous arrivals are resolved correctly
* **Optimal Complexity:** `O(rows * cols)` Time, `O(rows * cols)` Auxiliary Space

---

## Problem Statement

Given an `n x m` grid and a list of colored source cells `[r, c, color]`, every colored cell spreads its color to adjacent uncolored cells each time step, simultaneously. If multiple colors reach the same uncolored cell in the same time step, that cell takes the **maximum** color value. Return the final grid once no more cells can be colored.

---

## Approaches

1. **Brute Force — Repeated Full-Grid Scans Until Stable**
2. **Optimal — Multi-Source BFS with Per-Level Candidate Collection**

---

# Approach 1 — Brute Force / Repeated Full-Grid Scans Until Stable

## Idea

Simulate the process time step by time step. Each step, scan the **entire** grid: for every currently uncolored cell, check all 4 neighbors from the *previous* step's grid state, and record the maximum color among any colored neighbors as a candidate. After scanning the whole grid, apply all the candidate colors at once (so a cell colored this step doesn't influence another cell's candidate calculation within the same step). Repeat until a full pass makes no changes.

## Dry Run

```text
n=3, m=3, sources = [[0,0,1],[2,2,2]]
```

Initial grid:

```text
[[1,0,0],
 [0,0,0],
 [0,0,2]]
```

Step 1 — scan for uncolored cells adjacent to colored ones (using the grid above):

```text
(0,1) adjacent to (0,0)=1 → candidate 1
(1,0) adjacent to (0,0)=1 → candidate 1
(2,1) adjacent to (2,2)=2 → candidate 2
(1,2) adjacent to (2,2)=2 → candidate 2
```

Apply all at once:

```text
[[1,1,0],
 [1,0,2],
 [0,2,2]]
```

Step 2 — scan again using this new grid:

```text
(0,2) adjacent to (0,1)=1 and (1,2)=2 → candidate max(1,2)=2
(1,1) adjacent to (0,1)=1, (1,0)=1, (2,1)=2, (1,2)=2 → candidate max=2
(2,0) adjacent to (1,0)=1 and (2,1)=2 → candidate max=2
```

Apply all at once:

```text
[[1,1,2],
 [1,2,2],
 [2,2,2]]
```

Step 3 — scan again: no uncolored cells remain → stop.

Final grid matches the expected output.

## Algorithm

1. Initialize the grid with `0`s, then place each source's color at its cell.
2. Repeat:

   * Build a separate `candidate` grid, all zeros.
   * For every uncolored cell, check its 4 neighbors in the *current* grid; if any are colored, set `candidate[r][c]` to the maximum color among them.
   * If no candidate was set anywhere this pass, stop.
   * Otherwise, copy every nonzero `candidate` value into the real grid.
3. Return the grid.

## Complexity

* **Time:** `O((rows * cols)²)`

  * In the worst case (e.g. a single source in a corner), each step only colors one new "ring" of cells at the frontier, requiring up to `O(rows + cols)` steps for a single source, and up to `O(rows * cols)` steps overall in adversarial layouts — each step costing a full `O(rows * cols)` grid scan.
* **Space:** `O(rows * cols)`

  * For the temporary `candidate` grid built each pass.

## Notes / Tips

* Rescanning the entire grid every step is the main inefficiency — most cells checked each pass are nowhere near the current "frontier" of newly colored cells, and get rechecked pointlessly step after step.
* Using a separate `candidate` grid (rather than mutating the real grid mid-scan) is essential — it's what correctly captures "simultaneous" spreading and prevents a same-step chain reaction from incorrectly extending a color's reach further than one step in a single pass.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> floodFill(int n, int m, vector<vector<int>>& sources) {
        vector<vector<int>> grid(n, vector<int>(m, 0));
        for (auto& src : sources) {
            grid[src[0]][src[1]] = src[2];
        }

        int dr[] = {1, -1, 0, 0};
        int dc[] = {0, 0, 1, -1};

        while (true) {
            vector<vector<int>> candidate(n, vector<int>(m, 0));
            bool changed = false;

            for (int r = 0; r < n; r++) {
                for (int c = 0; c < m; c++) {
                    if (grid[r][c] == 0) {
                        int best = 0;
                        for (int d = 0; d < 4; d++) {
                            int nr = r + dr[d], nc = c + dc[d];
                            if (nr >= 0 && nr < n && nc >= 0 && nc < m && grid[nr][nc] != 0) {
                                best = max(best, grid[nr][nc]);
                            }
                        }
                        if (best != 0) {
                            candidate[r][c] = best;
                            changed = true;
                        }
                    }
                }
            }

            if (!changed) break;

            for (int r = 0; r < n; r++) {
                for (int c = 0; c < m; c++) {
                    if (candidate[r][c] != 0) {
                        grid[r][c] = candidate[r][c];
                    }
                }
            }
        }

        return grid;
    }
};
```

---

# Approach 2 — Optimal / Multi-Source BFS with Per-Level Candidate Collection

## Idea

Start a BFS from every source cell simultaneously. Process the queue level by level (one level equals one time step). Within a single level, instead of coloring a cell the instant it's first reached (which could pick the wrong color if a second, larger color reaches it later in the *same* level), collect **all** candidate colors landing on each uncolored cell during this level into a temporary map, take the maximum for each cell only after the whole level has been scanned, then commit those colors and enqueue the newly colored cells for the next level.

## Dry Run

```text
n=3, m=3, sources = [[0,0,1],[2,2,2]]
```

Initialize queue with both sources: `[(0,0), (2,2)]`.

Level 1 — process `(0,0)` color `1` and `(2,2)` color `2`:

```text
(0,0) spreads to (0,1) and (1,0), both uncolored → candidates: (0,1)=1, (1,0)=1
(2,2) spreads to (2,1) and (1,2), both uncolored → candidates: (2,1)=2, (1,2)=2
```

Commit: grid becomes

```text
[[1,1,0],
 [1,0,2],
 [0,2,2]]
```

Push `(0,1),(1,0),(2,1),(1,2)` into the queue for the next level.

Level 2 — process those four cells:

```text
(0,1) color=1 spreads to (0,2) and (1,1) → candidates: (0,2)=1, (1,1)=1
(1,0) color=1 spreads to (1,1) and (2,0) → candidates: (1,1)=max(1,1)=1, (2,0)=1
(2,1) color=2 spreads to (2,0) and (1,1) → candidates: (2,0)=max(1,2)=2, (1,1)=max(1,2)=2
(1,2) color=2 spreads to (0,2) and (1,1) → candidates: (0,2)=max(1,2)=2, (1,1)=max(2,2)=2
```

Final candidates this level: `(0,2)=2, (1,1)=2, (2,0)=2`. Commit:

```text
[[1,1,2],
 [1,2,2],
 [2,2,2]]
```

No uncolored cells remain → BFS ends. Matches the expected output.

## Algorithm

1. Initialize the grid with `0`s, place each source's color, and push all source coordinates into a queue.
2. While the queue is non-empty:

   * Record the current level's size.
   * Initialize an empty map `candidates` (cell → best color seen this level).
   * For each cell in this level:

     * For each of its 4 neighbors: if still uncolored, update `candidates[neighbor] = max(candidates[neighbor], currentCellColor)` (or just set it if not yet present).
   * After processing the whole level, for each entry in `candidates`: set the grid cell to that color, and push it into the queue for the next level.
3. Return the grid.

## Complexity

* **Time:** `O(rows * cols)`

  * Every cell is enqueued and processed exactly once across the entire BFS; the per-level candidate map operations are amortized `O(1)` each.
* **Space:** `O(rows * cols)`

  * For the BFS queue and the per-level candidate map, each bounded by the total number of cells.

## Notes / Tips

* Deferring the actual color assignment until an entire level has been fully scanned is the most IMP step, without this, a cell reached first by a smaller color would lock in that color and never get overwritten by a larger color arriving in the same time step from a different direction.
* A cell only ever needs to be considered as a "candidate" once (during the single level when it first becomes reachable). Once colored and enqueued, it acts as a source for future levels and is never revisited as a target.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> floodFill(int n, int m, vector<vector<int>>& sources) {
        vector<vector<int>> grid(n, vector<int>(m, 0));
        queue<pair<int,int>> q;

        for (auto& src : sources) {
            int r = src[0], c = src[1], color = src[2];
            grid[r][c] = color;
            q.push({r, c});
        }

        int dr[] = {1, -1, 0, 0};
        int dc[] = {0, 0, 1, -1};

        while (!q.empty()) {
            int size = q.size();
            unordered_map<int, int> candidates;

            for (int i = 0; i < size; i++) {
                auto [r, c] = q.front();
                q.pop();
                int color = grid[r][c];

                for (int d = 0; d < 4; d++) {
                    int nr = r + dr[d], nc = c + dc[d];

                    if (nr >= 0 && nr < n && nc >= 0 && nc < m && grid[nr][nc] == 0) {
                        int key = nr * m + nc;
                        if (candidates.find(key) == candidates.end() || candidates[key] < color) {
                            candidates[key] = color;
                        }
                    }
                }
            }

            for (auto& [key, color] : candidates) {
                int r = key / m, c = key % m;
                grid[r][c] = color;
                q.push({r, c});
            }
        }

        return grid;
    }
};
```

---

## Key Template

```text
grid = all 0
queue = all source cells (with grid colored at their positions)

while queue not empty:
    levelSize = queue.size()
    candidates = {}   # cell -> best color this level

    for i in 0..levelSize-1:
        (r, c) = queue.pop()
        color = grid[r][c]

        for each neighbor (nr, nc):
            if grid[nr][nc] == 0:
                candidates[(nr,nc)] = max(candidates.get((nr,nc), 0), color)

    for (r, c), color in candidates:
        grid[r][c] = color
        queue.push((r, c))

return grid
```