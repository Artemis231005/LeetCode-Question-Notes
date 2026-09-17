# LeetCode 836 — Rectangle Overlap

## Metadata

* **LeetCode:** 836
* **Problem:** Rectangle Overlap
* **Difficulty:** Easy
* **Topics:** Math, Geometry
* **Pattern:** Axis-Separation Check
* **Key Technique:** Two axis-aligned rectangles overlap (with positive area) exactly when their x-ranges overlap AND their y-ranges overlap — check each axis independently instead of reasoning about the 2D shapes directly
* **Optimal Complexity:** `O(1)` Time, `O(1)` Space

---

## Problem Statement

Given two axis-aligned rectangles `rec1` and `rec2`, each represented as `[x1, y1, x2, y2]` (bottom-left and top-right corners), return `true` if they overlap with a **positive** area (touching at an edge or corner doesn't count).

---

## Approaches

1. **Brute Force — Check Corner/Point Containment**
2. **Optimal — Axis-Separation Check**

---

# Approach 1 — Brute Force / Check Corner/Point Containment

## Idea

Check whether any corner of one rectangle lies strictly inside the other rectangle, and vice versa. If any corner containment check succeeds, the rectangles overlap.

## Dry Run

```text
rec1 = [0,0,2,2], rec2 = [1,1,3,3]
```

Check rec1's corners against rec2:

```text
(0,0): not inside rec2 (rec2 spans x:1-3, y:1-3)
(2,0): not inside rec2
(0,2): not inside rec2
(2,2): inside rec2? 1<2<3 and 1<2<3 → yes → overlap found
```

Return `true`.

## Algorithm

1. Define a helper `pointInsideRect(x, y, rect)` that checks if `(x, y)` lies strictly inside `rect` (i.e. `rect[0] < x < rect[2]` and `rect[1] < y < rect[3]`).
2. Check all 4 corners of `rec1` against `rec2`, and all 4 corners of `rec2` against `rec1`.
3. If any check succeeds, return `true`.
4. Otherwise, return `false`.

## Complexity

* **Time:** `O(1)`

  * A fixed number of corner checks (8 total) — but this approach is subtly **incorrect** in some cases (see Notes below), unlike the other `O(1)` approaches in this set of notes, which are all correct.
* **Space:** `O(1)`

  * No extra structures used.

## Notes / Tips

* This approach is actually **broken** for certain valid overlaps — e.g. two rectangles forming a "plus sign" cross shape (one wide and short, one narrow and tall) can overlap with positive area in the middle without either rectangle's corners ever landing inside the other. It's included here as a first instinct, not a reliable brute force.
* Because of this correctness gap, a real brute force for this problem would need a different (more exhaustive) formulation — checking corner containment alone is not a valid substitute for the axis-separation logic in Approach 2, which should be treated as the primary correct solution.

## Code

```cpp
class Solution {
public:
    bool pointInsideRect(int x, int y, vector<int>& rect) {
        return rect[0] < x && x < rect[2] && rect[1] < y && y < rect[3];
    }

    bool isRectangleOverlap(vector<int>& rec1, vector<int>& rec2) {
        // NOTE: this check is incomplete — it misses "cross" overlaps
        // where neither rectangle's corners land inside the other.
        vector<pair<int,int>> corners1 = {
            {rec1[0], rec1[1]}, {rec1[2], rec1[1]},
            {rec1[0], rec1[3]}, {rec1[2], rec1[3]}
        };
        vector<pair<int,int>> corners2 = {
            {rec2[0], rec2[1]}, {rec2[2], rec2[1]},
            {rec2[0], rec2[3]}, {rec2[2], rec2[3]}
        };

        for (auto& [x, y] : corners1) {
            if (pointInsideRect(x, y, rec2)) return true;
        }
        for (auto& [x, y] : corners2) {
            if (pointInsideRect(x, y, rec1)) return true;
        }

        return false;
    }
};
```

---

# Approach 2 — Optimal / Axis-Separation Check

## Idea

Two axis-aligned rectangles overlap with positive area if and only if their projections onto **both** the x-axis and the y-axis overlap. Instead of reasoning about the 2D shapes directly (which is what made Approach 1 fragile), check each axis independently: the x-ranges `[x1, x2]` overlap if `rec1's left < rec2's right` AND `rec2's left < rec1's right` (strict inequalities, since touching edges don't count as positive-area overlap) — and the same check applies to the y-ranges. Both must hold simultaneously.

## Dry Run

```text
rec1 = [0,0,2,2], rec2 = [1,1,3,3]
```

X-axis check:

```text
rec1's x-range: [0,2], rec2's x-range: [1,3]
rec1.x1 < rec2.x2? 0 < 3 → yes
rec2.x1 < rec1.x2? 1 < 2 → yes
→ x-ranges overlap
```

Y-axis check:

```text
rec1's y-range: [0,2], rec2's y-range: [1,3]
rec1.y1 < rec2.y2? 0 < 3 → yes
rec2.y1 < rec1.y2? 1 < 2 → yes
→ y-ranges overlap
```

Both axes overlap → return `true`.

### Cross-shape example (where Approach 1 would fail)

```text
rec1 = [0,2,5,3]  (wide, short — horizontal bar)
rec2 = [2,0,3,5]  (narrow, tall — vertical bar)
```

X-axis check: `rec1.x1=0 < rec2.x2=3` and `rec2.x1=2 < rec1.x2=5` → overlap.
Y-axis check: `rec1.y1=2 < rec2.y2=5` and `rec2.y1=0 < rec1.y2=3` → overlap.

Both hold → return `true` (correctly detects the cross-shaped overlap in the middle, which no corner of either rectangle touches).

## Algorithm

1. Check x-axis overlap: `rec1[0] < rec2[2]` AND `rec2[0] < rec1[2]`.
2. Check y-axis overlap: `rec1[1] < rec2[3]` AND `rec2[1] < rec1[3]`.
3. Return `true` only if both checks hold; otherwise return `false`.

## Complexity

* **Time:** `O(1)`

  * A fixed, small number of comparisons.
* **Space:** `O(1)`

  * No extra structures used.

## Notes / Tips

* Using **strict** inequalities (`<`, not `<=`) is what correctly excludes edge-touching or corner-touching rectangles from counting as overlapping, per the problem's "positive area" requirement.
* This axis-separation idea (checking each dimension independently and requiring overlap on *every* axis) generalizes directly to axis-aligned bounding box (AABB) collision detection in any number of dimensions — a very common technique in computational geometry and game physics.
* This is the actually-correct approach for this problem — Approach 1's corner-containment idea is a natural first instinct but fails on cross-shaped overlaps, which is exactly why axis separation is preferred here rather than as just a "faster" alternative.

## Code

```cpp
class Solution {
public:
    bool isRectangleOverlap(vector<int>& rec1, vector<int>& rec2) {
        bool xOverlap = rec1[0] < rec2[2] && rec2[0] < rec1[2];
        bool yOverlap = rec1[1] < rec2[3] && rec2[1] < rec1[3];

        return xOverlap && yOverlap;
    }
};
```

---

## Key Template

```text
xOverlap = rec1.x1 < rec2.x2 and rec2.x1 < rec1.x2
yOverlap = rec1.y1 < rec2.y2 and rec2.y1 < rec1.y2

return xOverlap and yOverlap
```