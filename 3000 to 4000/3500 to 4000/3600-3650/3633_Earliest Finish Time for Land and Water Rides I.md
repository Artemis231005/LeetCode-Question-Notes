# LeetCode 3633 — Earliest Finish Time for Land and Water Rides I

## Metadata

* **LeetCode:** 3633
* **Problem:** Earliest Finish Time for Land and Water Rides I
* **Difficulty:** Easy
* **Topics:** Array, Greedy
* **Pattern:** Greedy Reduction (Fix the Best "First Ride" Per Category)
* **Key Technique:** For a fixed order (land-then-water or water-then-land), the optimal choice for the *first* ride is always the one with the earliest possible finish time in its category — no other choice for the first ride can ever do better, regardless of which second ride is paired with it
* **Optimal Complexity:** `O(n + m)` Time, `O(1)` Space

---

## Problem Statement

Given land ride start times and durations, and water ride start times and durations, a tourist takes exactly one ride from each category, in either order (waiting if the next ride hasn't opened yet). Return the earliest possible time both rides can be finished.

---

## Approaches

1. **Brute Force — Check Every Pair in Both Orders**
2. **Optimal — Fix the Best First Ride Per Category**

---

# Approach 1 — Brute Force / Check Every Pair in Both Orders

## Idea

For every combination of one land ride and one water ride, compute the finish time for both possible orders (land first, or water first), and track the overall minimum across all combinations and both orders.

## Dry Run

```text
landStartTime = [2,8], landDuration = [4,1]
waterStartTime = [6], waterDuration = [3]
```

Pair `(land 0, water 0)`, land first:

```text
land finishes at 2+4=6
water starts at max(6,6)=6, finishes at 6+3=9
```

Pair `(land 0, water 0)`, water first:

```text
water finishes at 6+3=9
land starts at max(9,2)=9, finishes at 9+4=13
```

Pair `(land 1, water 0)`, land first:

```text
land finishes at 8+1=9
water starts at max(9,6)=9, finishes at 9+3=12
```

Pair `(land 1, water 0)`, water first:

```text
water finishes at 9 (same as before)
land starts at max(9,8)=9, finishes at 9+1=10
```

Minimum across all four: `9` (from the first combination, land-first).

## Algorithm

1. Initialize `best = infinity`.
2. For each land ride `i` and each water ride `j`:

   * **Land first:** `landFinish = landStart[i] + landDuration[i]`; `waterFinish = max(landFinish, waterStart[j]) + waterDuration[j]`. Update `best`.
   * **Water first:** `waterFinish = waterStart[j] + waterDuration[j]`; `landFinish = max(waterFinish, landStart[i]) + landDuration[i]`. Update `best`.
3. Return `best`.

## Complexity

* **Time:** `O(n * m)`

  * Every pair of land and water rides is checked in both possible orders.
* **Space:** `O(1)`

  * Only a running minimum tracked.

## Notes / Tips

* Given the problem's small constraints (`n, m <= 100`), this brute force is already fast enough in practice (`10,000` pairs at most).
* The key realization that unlocks a faster approach: for a fixed order, the best choice for the *first* ride never actually depends on which second ride ends up being paired with it — this is what Approach 2 exploits to avoid checking every pair.

## Code

```cpp
class Solution {
public:
    int earliestFinishTime(vector<int>& landStartTime, vector<int>& landDuration,
                            vector<int>& waterStartTime, vector<int>& waterDuration) {
        int best = INT_MAX;
        int n = landStartTime.size(), m = waterStartTime.size();

        for (int i = 0; i < n; i++) {
            for (int j = 0; j < m; j++) {
                int landFinish = landStartTime[i] + landDuration[i];
                int waterFinishAfterLand = max(landFinish, waterStartTime[j]) + waterDuration[j];
                best = min(best, waterFinishAfterLand);

                int waterFinish = waterStartTime[j] + waterDuration[j];
                int landFinishAfterWater = max(waterFinish, landStartTime[i]) + landDuration[i];
                best = min(best, landFinishAfterWater);
            }
        }

        return best;
    }
};
```

---

# Approach 2 — Optimal / Fix the Best First Ride Per Category

## Idea

For the order "land ride first, then water ride": the total finish time is `max(landFinish, waterStart[j]) + waterDuration[j]`. Splitting on whether `landFinish <= waterStart[j]` or not shows that, for *any* fixed water ride `j`, this expression is minimized by using whichever land ride has the smallest `landFinish` overall — a land ride with an earlier finish can never make the result worse, and can only ever help (either by falling into the case where it doesn't matter at all, or by directly reducing the `max`). So the best land ride to use for this order is fixed in advance: the one with minimum `landStart[i] + landDuration[i]`. The same reasoning applies symmetrically to "water ride first, then land ride," fixing the best water ride as the one with minimum `waterFinish`. This collapses the search from checking every pair down to just two linear scans.

## Dry Run

```text
landStartTime = [2,8], landDuration = [4,1]
waterStartTime = [6], waterDuration = [3]
```

Find best land ride finish (minimum `landStart+landDuration`):

```text
land 0: 2+4=6
land 1: 8+1=9
minLandFinish = 6
```

Find best water ride finish (minimum `waterStart+waterDuration`):

```text
water 0: 6+3=9
minWaterFinish = 9
```

**Land-then-water order:** using `minLandFinish=6`, scan all water rides:

```text
water 0: max(6, 6) + 3 = 9
```

Best for this order: `9`.

**Water-then-land order:** using `minWaterFinish=9`, scan all land rides:

```text
land 0: max(9, 2) + 4 = 13
land 1: max(9, 8) + 1 = 10
```

Best for this order: `10`.

Overall answer: `min(9, 10) = 9`, matching the brute-force result.

## Algorithm

1. Compute `minLandFinish = min(landStart[i] + landDuration[i])` over all land rides.
2. Compute `minWaterFinish = min(waterStart[j] + waterDuration[j])` over all water rides.
3. For the land-then-water order, scan all water rides using the fixed `minLandFinish`:

   * `candidate = max(minLandFinish, waterStart[j]) + waterDuration[j]` for each `j`; track the minimum.
4. For the water-then-land order, scan all land rides using the fixed `minWaterFinish`:

   * `candidate = max(minWaterFinish, landStart[i]) + landDuration[i]` for each `i`; track the minimum.
5. Return the overall minimum across both orders.

## Complexity

* **Time:** `O(n + m)`

  * One pass to find `minLandFinish` (`O(n)`), one pass to find `minWaterFinish` (`O(m)`), then one more `O(m)` pass and one more `O(n)` pass for the two final scans.
* **Space:** `O(1)`

  * Only a handful of running values tracked.

## Notes / Tips

* The correctness argument is a case split: if the fixed "best" first ride's finish time is already `<=` the second ride's start time, the second ride's own finish time is all that matters (independent of which first ride was used, as long as *some* first ride qualifies) — and the ride with the globally minimum finish time is the one most likely to qualify, so using it is never worse. If instead it's `>` the second ride's start time, the total is `firstRideFinish + secondRideDuration`, which is directly minimized by the smallest possible `firstRideFinish`. Either way, the minimum-finish first ride dominates all other choices.
* Given the problem's very small constraints, the `O(n*m)` brute force and this `O(n+m)` optimization perform almost identically in practice — but the reduction itself is a good example of spotting when a seemingly two-dimensional search actually only depends on a single extremal value from each side.
* This same "the best choice for one slot is independent of what fills the other slot" reduction pattern shows up in other pairing/scheduling problems where a `max` or `min` creates a clean case split.

## Code

```cpp
class Solution {
public:
    int earliestFinishTime(vector<int>& landStartTime, vector<int>& landDuration,
                            vector<int>& waterStartTime, vector<int>& waterDuration) {
        int n = landStartTime.size(), m = waterStartTime.size();

        int minLandFinish = INT_MAX;
        for (int i = 0; i < n; i++) {
            minLandFinish = min(minLandFinish, landStartTime[i] + landDuration[i]);
        }

        int minWaterFinish = INT_MAX;
        for (int j = 0; j < m; j++) {
            minWaterFinish = min(minWaterFinish, waterStartTime[j] + waterDuration[j]);
        }

        int best = INT_MAX;

        for (int j = 0; j < m; j++) {
            int candidate = max(minLandFinish, waterStartTime[j]) + waterDuration[j];
            best = min(best, candidate);
        }

        for (int i = 0; i < n; i++) {
            int candidate = max(minWaterFinish, landStartTime[i]) + landDuration[i];
            best = min(best, candidate);
        }

        return best;
    }
};
```

---

## Key Template

```text
minLandFinish = min(landStart[i] + landDuration[i] for all i)
minWaterFinish = min(waterStart[j] + waterDuration[j] for all j)

best = infinity

for j in 0..m-1:
    best = min(best, max(minLandFinish, waterStart[j]) + waterDuration[j])

for i in 0..n-1:
    best = min(best, max(minWaterFinish, landStart[i]) + landDuration[i])

return best
```