# LeetCode 733 — Flood Fill

## Metadata

* **LeetCode:** 733
* **Problem:** Flood Fill
* **Difficulty:** Easy
* **Topics:** Array, Depth-First Search, Breadth-First Search, Matrix
* **Pattern:** Grid Traversal (DFS/BFS) with a Boundary/Color Condition
* **Key Technique:** Starting from the given pixel, spread to connected pixels of the same original color, recoloring each one as it's visited to avoid reprocessing
* **Optimal Complexity:** `O(rows * cols)` Time, `O(rows * cols)` Auxiliary Space

---

## Problem Statement

Given an `m x n` grid `image` representing pixel colors, a starting pixel `(sr, sc)`, and a new color, perform a flood fill: change the color of the starting pixel and every pixel connected to it (4-directionally) that shares the starting pixel's original color, to the new color.

---

## Approaches

1. **Brute Force — Repeated Full-Grid Scans Until Stable**
2. **Optimal — DFS/BFS from the Starting Pixel**

---

# Approach 1 — Brute Force / Repeated Full-Grid Scans Until Stable

## Idea

Instead of tracing connectivity directly, repeatedly scan the entire grid looking for any pixel that (a) matches the original starting color and (b) is adjacent to a pixel already recolored to the new color. Recolor it, and keep repeating full scans until an entire pass finds nothing left to change.

## Dry Run

```text
image = [[1,1,1],
         [1,1,0],
         [1,0,1]], sr=1, sc=1, color=2

original color = image[1][1] = 1
```

Recolor the starting pixel directly: `image[1][1] = 2`.

Full scan 1: find pixels matching original color `1` adjacent to a `2`:

```text
(0,1) adjacent to (1,1)=2 → recolor → image[0][1]=2
(1,0) adjacent to (1,1)=2 → recolor → image[1][0]=2
```

Full scan 2: find pixels matching `1` adjacent to a `2`:

```text
(0,0) adjacent to (0,1)=2 or (1,0)=2 → recolor → image[0][0]=2
(0,2) adjacent to (0,1)=2 → recolor → image[0][2]=2
```

Full scan 3: no more matching pixels found → stop.

Final:

```text
[[2,2,2],
 [2,2,0],
 [2,0,1]]
```

## Algorithm

1. Record `originalColor = image[sr][sc]`.
2. If `originalColor == color`, return `image` unchanged (avoids infinite loop from re-matching already-recolored pixels).
3. Set `image[sr][sc] = color`.
4. Repeat:

   * Scan every cell in the grid.
   * If a cell equals `originalColor` and is 4-directionally adjacent to a cell already equal to `color`, recolor it to `color`.
   * Track whether any change was made this pass.
   * Stop once a full pass makes no changes.
5. Return `image`.

## Complexity

* **Time:** `O((rows * cols)²)`

  * In the worst case, each full scan only recolors one new pixel (e.g. a long snake-like region), requiring up to `rows * cols` full scans, each costing `O(rows * cols)`.
* **Space:** `O(1)`

  * Only a boolean "changed" flag — modifies the grid in place with no extra structures.

## Notes / Tips

* Rescanning the entire grid repeatedly is extremely wasteful — a direct traversal (DFS/BFS) that only visits cells actually connected to the starting pixel is far more efficient and is the natural way to think about "flood fill."
* The `originalColor == color` early return is essential regardless of approach — without it, a same-color fill would cause the traversal (or repeated scanning) to consider already-"recolored" pixels as still matching, leading to incorrect behavior or infinite loops in some formulations.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int color) {
        int originalColor = image[sr][sc];
        if (originalColor == color) {
            return image;
        }

        int rows = image.size(), cols = image[0].size();
        image[sr][sc] = color;

        bool changed = true;
        while (changed) {
            changed = false;

            for (int r = 0; r < rows; r++) {
                for (int c = 0; c < cols; c++) {
                    if (image[r][c] == originalColor) {
                        bool adjacentToNew =
                            (r > 0 && image[r-1][c] == color) ||
                            (r < rows-1 && image[r+1][c] == color) ||
                            (c > 0 && image[r][c-1] == color) ||
                            (c < cols-1 && image[r][c+1] == color);

                        if (adjacentToNew) {
                            image[r][c] = color;
                            changed = true;
                        }
                    }
                }
            }
        }

        return image;
    }
};
```

---

# Approach 2 — Optimal / DFS/BFS from the Starting Pixel

## Idea

Directly trace connectivity from the starting pixel outward using DFS (or BFS): recolor the starting pixel, then recursively visit each of its 4 neighbors — if a neighbor still has the *original* color, recolor it and recurse from there too. This only ever touches pixels that are actually connected to the start, instead of repeatedly rescanning the whole grid.

## Dry Run

```text
image = [[1,1,1],
         [1,1,0],
         [1,0,1]], sr=1, sc=1, color=2

original color = 1
```

DFS from `(1,1)`:

```text
image[1][1] = 2 (recolored)
check (0,1): color=1 (matches original) → recolor to 2, recurse from (0,1)
    check (0,0): color=1 → recolor to 2, recurse
        no further matching neighbors from (0,0)
    check (0,2): color=1 → recolor to 2, recurse
        no further matching neighbors from (0,2)
check (2,1): color=0 (doesn't match original) → skip
check (1,0): color=1 → recolor to 2, recurse
    no further matching neighbors from (1,0)
check (1,2): color=0 → skip
```

Final:

```text
[[2,2,2],
 [2,2,0],
 [2,0,1]]
```

Matches the brute-force result.

## Algorithm

1. Record `originalColor = image[sr][sc]`.
2. If `originalColor == color`, return `image` unchanged (prevents infinite recursion on a same-color fill).
3. Define a recursive `dfs(r, c)`:

   * If `(r, c)` is out of bounds, or `image[r][c] != originalColor`, return.
   * Set `image[r][c] = color`.
   * Recursively call `dfs` on all 4 neighbors: `(r+1,c)`, `(r-1,c)`, `(r,c+1)`, `(r,c-1)`.
4. Call `dfs(sr, sc)`.
5. Return `image`.

## Complexity

* **Time:** `O(rows * cols)`

  * Each pixel is visited and recolored at most once, since recoloring immediately prevents it from matching `originalColor` again.
* **Space:** `O(rows * cols)`

  * For the recursion call stack in the worst case (e.g. the entire grid is one connected region of the original color).

## Notes / Tips

* Recoloring a pixel *before* recursing into its neighbors (rather than after) is what naturally prevents revisiting it — once changed, it no longer matches `originalColor`, so the base case catches it immediately on any future visit attempt.
* The `originalColor == color` guard is critical: without it, since the start pixel gets "recolored" to the same value it already has, every neighboring pixel still matches and the recursion could loop back through already-visited pixels indefinitely (or at least redundantly).
* This is the exact same flood-fill template used in LC 200 (Number of Islands) and similar grid connectivity problems — the shape is identical, only the "match" condition (specific color vs. simply being land) and the "action taken" (recolor vs. mark visited) differ.

## Code

```cpp
class Solution {
public:
    int rows, cols;

    void dfs(vector<vector<int>>& image, int r, int c, int originalColor, int newColor) {
        if (r < 0 || r >= rows || c < 0 || c >= cols || image[r][c] != originalColor) {
            return;
        }

        image[r][c] = newColor;

        dfs(image, r + 1, c, originalColor, newColor);
        dfs(image, r - 1, c, originalColor, newColor);
        dfs(image, r, c + 1, originalColor, newColor);
        dfs(image, r, c - 1, originalColor, newColor);
    }

    vector<vector<int>> floodFill(vector<vector<int>>& image, int sr, int sc, int color) {
        int originalColor = image[sr][sc];
        if (originalColor == color) {
            return image;
        }

        rows = image.size();
        cols = image[0].size();

        dfs(image, sr, sc, originalColor, color);

        return image;
    }
};
```

---

## Key Template

```text
function dfs(r, c, originalColor, newColor):
    if out of bounds or image[r][c] != originalColor:
        return

    image[r][c] = newColor

    dfs(r+1, c, ...)
    dfs(r-1, c, ...)
    dfs(r, c+1, ...)
    dfs(r, c-1, ...)

originalColor = image[sr][sc]
if originalColor == newColor: return image

dfs(sr, sc, originalColor, newColor)
return image
```