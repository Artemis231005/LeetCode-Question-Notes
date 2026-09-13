# LeetCode 994 — Rotting Oranges

## Metadata

* **LeetCode:** 994
* **Problem:** Rotting Oranges
* **Difficulty:** Medium
* **Topics:** Array, Breadth-First Search, Matrix
* **Pattern:** Multi-Source BFS
* **Key Technique:** Start BFS from all rotten oranges simultaneously (not one at a time), so each level of the BFS naturally corresponds to one minute passing
* **Optimal Complexity:** `O(rows * cols)` Time, `O(rows * cols)` Auxiliary Space

---

## Problem Statement

Given a grid where each cell is `0` (empty), `1` (fresh orange), or `2` (rotten orange), every minute any fresh orange adjacent (4-directionally) to a rotten orange also becomes rotten. Return the minimum number of minutes until no cell has a fresh orange, or `-1` if that's impossible.

---

## Approaches

1. **Brute Force — Simulate Minute by Minute with Full Grid Scans**
2. **Optimal — Multi-Source BFS**

---

# Approach 1 — Brute Force / Simulate Minute by Minute with Full Grid Scans

## Idea

Literally simulate the process described in the problem: each minute, scan the entire grid to find every fresh orange adjacent to a rotten one, and mark all of them as "to be rotted" (using a separate pass so that oranges rotted *this* minute don't also mark others in the same minute). Apply all the changes at once, increment the minute counter, and repeat until a full pass produces no new rotten oranges.

## Dry Run

```text
grid = [[2,1,1],
        [1,1,0],
        [0,1,1]]
```

Minute `0` → `1`: scan for fresh oranges adjacent to rotten ones:

```text
(0,1) adjacent to (0,0)=2 → mark to rot
(1,0) adjacent to (0,0)=2 → mark to rot
```

Apply changes:

```text
[[2,2,1],
 [2,1,0],
 [0,1,1]]
```

Minute `1` → `2`: scan again:

```text
(0,2) adjacent to (0,1)=2 → mark
(1,1) adjacent to (0,1)=2 or (1,0)=2 → mark
```

Apply changes:

```text
[[2,2,2],
 [2,2,0],
 [0,1,1]]
```

Minute `2` → `3`: scan again:

```text
(2,1) adjacent to (1,1)=2 → mark
```

Apply changes:

```text
[[2,2,2],
 [2,2,0],
 [0,2,1]]
```

Minute `3` → `4`: scan again:

```text
(2,2) adjacent to (2,1)=2 → mark
```

Apply changes:

```text
[[2,2,2],
 [2,2,0],
 [0,2,2]]
```

No fresh oranges remain → stop. Total minutes: `4`.

## Algorithm

1. Initialize `minutes = 0`.
2. Repeat:

   * Scan the grid for fresh oranges (`1`) adjacent to a rotten orange (`2`), collecting them into a separate list (don't mutate mid-scan).
   * If the list is empty, stop.
   * Mark all collected oranges as rotten in the grid.
   * Increment `minutes`.
3. Scan the final grid for any remaining fresh oranges — if any exist, return `-1`.
4. Otherwise, return `minutes`.

## Complexity

* **Time:** `O((rows * cols)²)`

  * In the worst case (a long snake-like path of fresh oranges), each minute only rots one new orange, requiring up to `rows * cols` minutes, each costing a full `O(rows * cols)` grid scan.
* **Space:** `O(rows * cols)`

  * For the temporary list of oranges to rot each minute.

## Notes / Tips

* Rescanning the entire grid every single minute is the main inefficiency — a BFS starting from all rotten oranges at once naturally processes the grid in "minute layers" without needing to repeatedly search for what changed.
* Using a separate collection list before applying changes (rather than mutating the grid mid-scan) is essential for correctness — otherwise a freshly-rotted orange could incorrectly cause a chain reaction within the same simulated minute.

## Code

```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        int minutes = 0;

        while (true) {
            vector<pair<int,int>> toRot;

            for (int r = 0; r < rows; r++) {
                for (int c = 0; c < cols; c++) {
                    if (grid[r][c] == 1) {
                        bool adjacentToRotten =
                            (r > 0 && grid[r-1][c] == 2) ||
                            (r < rows-1 && grid[r+1][c] == 2) ||
                            (c > 0 && grid[r][c-1] == 2) ||
                            (c < cols-1 && grid[r][c+1] == 2);

                        if (adjacentToRotten) {
                            toRot.push_back({r, c});
                        }
                    }
                }
            }

            if (toRot.empty()) {
                break;
            }

            for (auto& [r, c] : toRot) {
                grid[r][c] = 2;
            }

            minutes++;
        }

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 1) {
                    return -1;
                }
            }
        }

        return minutes;
    }
};
```

---

# Approach 2 — Optimal / Multi-Source BFS

## Idea

Instead of repeatedly rescanning the whole grid, start a BFS from **every** initially rotten orange at once (a "multi-source" BFS). Process the queue level by level — each full level processed corresponds to exactly one minute passing, since every orange in the current level rots all its fresh neighbors simultaneously. This naturally mirrors the problem's real-world simultaneity without any rescanning.

## Dry Run

```text
grid = [[2,1,1],
        [1,1,0],
        [0,1,1]]
```

Initialize queue with all rotten oranges: `[(0,0)]`. Count fresh oranges: `6`.

Level 1 (minute 1): process `(0,0)`:

```text
neighbors (0,1) and (1,0) are fresh → rot them, add to queue
grid becomes [[2,2,1],[2,1,0],[0,1,1]], freshCount = 4
queue = [(0,1),(1,0)]
```

Level 2 (minute 2): process `(0,1)` and `(1,0)`:

```text
(0,1)'s fresh neighbor (0,2) → rot, add to queue
(1,0)'s fresh neighbor (1,1) → rot, add to queue
grid becomes [[2,2,2],[2,2,0],[0,1,1]], freshCount = 2
queue = [(0,2),(1,1)]
```

Level 3 (minute 3): process `(0,2)` and `(1,1)`:

```text
(0,2) has no fresh neighbors
(1,1)'s fresh neighbor (2,1) → rot, add to queue
grid becomes [[2,2,2],[2,2,0],[0,2,1]], freshCount = 1
queue = [(2,1)]
```

Level 4 (minute 4): process `(2,1)`:

```text
(2,1)'s fresh neighbor (2,2) → rot, add to queue
grid becomes [[2,2,2],[2,2,0],[0,2,2]], freshCount = 0
queue = [(2,2)]
```

Level 5: queue processed but no fresh oranges left to rot, and `freshCount == 0` → stop, don't count this as an extra minute.

Final minutes: `4`. Matches the brute-force result.

## Algorithm

1. Initialize a queue with the coordinates of every initially rotten orange, and count the total number of fresh oranges.
2. If there are no fresh oranges to begin with, return `0` immediately.
3. Initialize `minutes = 0`.
4. While the queue is non-empty and `freshCount > 0`:

   * Record the current queue size (this level's oranges).
   * For each orange in this level:

     * For each of its 4 neighbors: if fresh, rot it, decrement `freshCount`, and add it to the queue.
   * Increment `minutes` (one full level processed = one minute passed).
5. Return `minutes` if `freshCount == 0`, otherwise return `-1` (some fresh oranges were unreachable).

## Complexity

* **Time:** `O(rows * cols)`

  * Every cell is enqueued and processed at most once across the entire BFS.
* **Space:** `O(rows * cols)`

  * For the BFS queue, which in the worst case can hold a large fraction of the grid's cells.

## Notes / Tips

* Starting BFS from **all** rotten oranges simultaneously (rather than running separate BFS traversals per orange) is the key idea — this is what makes each processed "level" correspond directly to one minute, since all oranges rotting at the same time do actually spread simultaneously.
* Tracking `freshCount` directly (rather than rescanning the grid at the end) is a minor but useful simplification — it avoids a second full grid pass just to check whether any fresh oranges remain.
* This is the standard multi-source BFS template — same shape solves "walls and gates," "01 matrix" (distance to nearest 0), and other problems where multiple starting points spread outward simultaneously and time/distance is measured in BFS levels.

## Code

```cpp
class Solution {
public:
    int orangesRotting(vector<vector<int>>& grid) {
        int rows = grid.size(), cols = grid[0].size();
        queue<pair<int,int>> q;
        int freshCount = 0;

        for (int r = 0; r < rows; r++) {
            for (int c = 0; c < cols; c++) {
                if (grid[r][c] == 2) {
                    q.push({r, c});
                } else if (grid[r][c] == 1) {
                    freshCount++;
                }
            }
        }

        if (freshCount == 0) {
            return 0;
        }

        int minutes = 0;
        int dr[] = {1, -1, 0, 0};
        int dc[] = {0, 0, 1, -1};

        while (!q.empty() && freshCount > 0) {
            int size = q.size();

            for (int i = 0; i < size; i++) {
                auto [r, c] = q.front();
                q.pop();

                for (int d = 0; d < 4; d++) {
                    int nr = r + dr[d], nc = c + dc[d];

                    if (nr >= 0 && nr < rows && nc >= 0 && nc < cols && grid[nr][nc] == 1) {
                        grid[nr][nc] = 2;
                        freshCount--;
                        q.push({nr, nc});
                    }
                }
            }

            minutes++;
        }

        return freshCount == 0 ? minutes : -1;
    }
};
```

---

## Key Template

```text
queue = all initially rotten cells
freshCount = count of fresh cells

if freshCount == 0: return 0

minutes = 0
while queue not empty and freshCount > 0:
    levelSize = queue.size()

    for i in 0..levelSize-1:
        (r, c) = queue.pop()
        for each neighbor (nr, nc):
            if neighbor is fresh:
                rot it, freshCount -= 1
                queue.push((nr, nc))

    minutes += 1

return minutes if freshCount == 0 else -1
```