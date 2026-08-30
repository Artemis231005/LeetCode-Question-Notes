# LeetCode 4034 — Minimum Bishop Moves to Reach Target

## Metadata

- **LeetCode:** 4034
- **Problem:** Minimum Bishop Moves to Reach Target
- **Difficulty:** Medium
- **Topics:** Math, Chess Geometry
- **Pattern:** Diagonal / Parity Analysis
- **Key Technique:** A bishop stays on one color forever; same diagonal → 1 move, same color → 2 moves, different color → impossible

---

# Approaches

1. **Brute Force — BFS over the 8x8 Board**
2. **Optimal — Direct Diagonal/Parity Check**

---

# Approach 1 — Brute Force / BFS

## Idea

Treat each of the 64 squares as a graph node. From any square, a bishop can move to any square along its 4 diagonal rays (any distance, board is empty so no blocking). Run BFS from `source` and find shortest distance to `target`.

## Dry Run

```text
source = [4,2], target = [1,3]
```

BFS layer 0:
```text
(4,2)
```

Layer 1 (all diagonal squares from (4,2)):
```text
(1,5),(2,4),(3,3),(5,1),(5,3),(6,4),(7,5),(8,6),(3,1)
```

`(1,3)` not found yet.

Layer 2 (expand from layer 1, e.g. from (3,1)):
```text
(3,1) → (1,3) ✓ found
```

Answer:
```text
2
```

## Algorithm

1. If `source == target`, return `0`.
2. Build/traverse a graph where each square connects to every square on its 4 diagonals.
3. BFS from `source`, tracking visited squares and distance.
4. Return distance when `target` is dequeued, or `-1` if BFS exhausts without finding it.

## Complexity

- **Time:** `O(64^2)` — 64 squares, each with up to ~28 diagonal neighbors
- **Space:** `O(64)`

## Notes / Tips

- Works but massively overkill since the board is fixed size and empty.
- Useful only as a sanity-check / brute-force reference for the math-based approach.

## Code

```cpp
class Solution {
public:
    int minimumMoves(vector<int>& source, vector<int>& target) {
        int sr = source[0], sc = source[1];
        int tr = target[0], tc = target[1];

        if (sr == tr && sc == tc) return 0;

        queue<pair<int,int>> q;
        vector<vector<bool>> visited(9, vector<bool>(9, false));
        q.push({sr, sc});
        visited[sr][sc] = true;
        int dist = 0;

        int dr[] = {1, 1, -1, -1};
        int dc[] = {1, -1, 1, -1};

        while (!q.empty()) {
            int size = q.size();
            dist++;
            for (int i = 0; i < size; i++) {
                auto [r, c] = q.front(); q.pop();

                for (int d = 0; d < 4; d++) {
                    int nr = r, nc = c;
                    while (true) {
                        nr += dr[d];
                        nc += dc[d];
                        if (nr < 1 || nr > 8 || nc < 1 || nc > 8) {
                            break;
                        }

                        if (nr == tr && nc == tc) {
                            return dist;
                        }

                        if (!visited[nr][nc]) {
                            visited[nr][nc] = true;
                            q.push({nr, nc});
                        }
                    }
                }
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Optimal / Direct Diagonal & Parity Check

## Idea

A bishop only ever lands on squares of the same color as its start. Color is determined by `(r + c) % 2`.

- Different color → unreachable, `-1`.
- Same square → `0` moves.
- Same diagonal (`r - c` equal, or `r + c` equal) → `1` move.
- Same color but not aligned → always `2` moves (pick any square that shares a diagonal with both source and target).

## Dry Run

```text
source = [4,2], target = [1,3]
```

Color check:
```text
(4+2)%2 = 0
(1+3)%2 = 0 → same color
```

Same diagonal check:
```text
sr - sc = 2, tr - tc = -2 → not equal
sr + sc = 6, tr + sc = 4 → not equal
```

Not aligned but same color:
```text
answer = 2
```

## Algorithm

1. If `source == target`, return `0`.
2. If `(sr + sc) % 2 != (tr + tc) % 2`, return `-1`.
3. If `sr - sc == tr - tc` or `sr + sc == tr + tc`, return `1`.
4. Otherwise, return `2`.

## Complexity

- **Time:** `O(1)`
- **Space:** `O(1)`

## Notes / Tips

- Board is empty and 8x8, so 2-move reachability always holds for any same-color square not on a shared diagonal — no need to actually find the intermediate square.
- Common mistake: checking only `r - c` or only `r + c` for the 1-move case — need both diagonals.
- Same core idea as color-based reachability puzzles (checkerboard parity arguments).

## Code

```cpp
class Solution {
public:
    int minimumMoves(vector<int>& source, vector<int>& target) {
        int sr = source[0], sc = source[1];
        int tr = target[0], tc = target[1];

        if (sr == tr && sc == tc) {
            return 0;
        }

        if ((sr + sc) % 2 != (tr + tc) % 2) {
            return -1;
        }

        if (sr - sc == tr - tc || sr + sc == tr + tc) {
            return 1;
        }

        return 2;
    }
};
```

---

# Key Template

### Bishop Reachability (Empty Board)

```text
if source == target: return 0
if (sr+sc)%2 != (tr+tc)%2: return -1
if sr-sc == tr-tc or sr+sc == tr+tc: return 1
return 2
```

## Pattern Recognition
```text
Bishop = fixed color per diagonal
        ↓
Check color parity first
        ↓
Same diagonal → 1, same color → 2, else → -1
```

The key observation is:

> **A bishop's reachable squares are entirely determined by diagonal alignment and color parity — pathfinding not needed.**