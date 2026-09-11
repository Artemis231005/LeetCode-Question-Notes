# LeetCode 2848 — Points That Intersect With Cars

## Metadata

* **LeetCode:** 2848
* **Problem:** Points That Intersect With Cars
* **Difficulty:** Easy
* **Topics:** Array, Hash Table, Prefix Sum, Sorting
* **Pattern:** Difference Array (Range Update, Point Query)
* **Key Technique:** Mark `+1` at each car's start and `-1` just after its end, then a running prefix sum tells you whether each point is covered by any car
* **Optimal Complexity:** `O(range + n)` Time, `O(range)` Space

---

## Problem Statement

Given a 2D array `nums` where `nums[i] = [starti, endi]` represents the inclusive road segment occupied by the `i`th car, return the number of integer points on the road that are covered by at least one car.

---

## Approaches

1. **Brute Force — Mark Every Point Covered by Each Car**
2. **Optimal — Difference Array**

---

# Approach 1 — Brute Force / Mark Every Point Covered by Each Car

## Idea

Use a boolean/set structure sized to the maximum possible coordinate. For each car, directly mark every point from `start` to `end` as covered. At the end, count how many points were marked.

## Dry Run

```text
nums = [[3,6],[1,5],[4,7]]
```

Mark `[3,6]`:

```text
covered = {3,4,5,6}
```

Mark `[1,5]`:

```text
covered = {1,2,3,4,5,6}
```

Mark `[4,7]`:

```text
covered = {1,2,3,4,5,6,7}
```

Count: `7` points covered.

## Algorithm

1. Create a boolean array `covered` of size `maxCoordinate + 1`, all `false`.
2. For each car `[start, end]`:

   * For `i` from `start` to `end`:

     * Set `covered[i] = true`.
3. Count and return the number of `true` entries.

## Complexity

* **Time:** `O(n * range)`

  * Each of the `n` cars can directly mark up to `range` points, where `range` is the maximum possible road length.
* **Space:** `O(range)`

  * For the `covered` boolean array.

## Notes / Tips

* Directly marking every point for every car is redundant — a range's coverage only needs to be recorded once, at its boundaries, rather than written to every single point inside it.
* Given the problem's small constraints (coordinates capped at `100`), this brute force is actually fast enough in practice, but doesn't scale to larger ranges.

## Code

```cpp
class Solution {
public:
    int numberOfPoints(vector<vector<int>>& nums) {
        vector<bool> covered(102, false);

        for (auto& car : nums) {
            for (int i = car[0]; i <= car[1]; i++) {
                covered[i] = true;
            }
        }

        int count = 0;
        for (bool c : covered) {
            if (c) count++;
        }

        return count;
    }
};
```

---

# Approach 2 — Optimal / Difference Array

## Idea

Instead of marking every point a car covers, mark the car's *effect* directly: add `+1` at the car's start, and subtract `1` just after its end. Then sweep through the array once, accumulating a running sum — at any point, a running sum greater than `0` means at least one car covers that point.

## Dry Run

```text
nums = [[3,6],[1,5],[4,7]]
```

Build difference array `diff` of size `102` (safely covering `end + 1` up to `101`):

```text
[3,6] → diff[3] += 1, diff[7] -= 1
[1,5] → diff[1] += 1, diff[6] -= 1
[4,7] → diff[4] += 1, diff[8] -= 1
```

```text
diff[1] = 1
diff[3] = 1
diff[4] = 1
diff[6] = -1
diff[7] = -1
diff[8] = -1
```

Prefix sum sweep, counting points where the running sum is `> 0`:

```text
i=1: running=1 → covered → count=1
i=2: running=1 → covered → count=2
i=3: running=2 → covered → count=3
i=4: running=3 → covered → count=4
i=5: running=3 → covered → count=5
i=6: running=2 → covered → count=6
i=7: running=1 → covered → count=7
i=8: running=0 → not covered
```

Final count: `7`, matching the brute-force result.

## Algorithm

1. Create a difference array `diff` of size `102` (or `maxCoordinate + 2`), all zeros.
2. For each car `[start, end]`:

   * `diff[start] += 1`.
   * `diff[end + 1] -= 1`.
3. Sweep `i` from `0` to `100`, maintaining a running sum:

   * `running += diff[i]`.
   * If `running > 0`, increment `count`.
4. Return `count`.

## Complexity

* **Time:** `O(range + n)`

  * `O(n)` to apply all car markers, `O(range)` to sweep through and accumulate the running sum, where `range` is the fixed bound on coordinates (here, up to `100`).
* **Space:** `O(range)`

  * For the fixed-size difference array, independent of `n`.

## Notes / Tips

* Same difference array technique as LC 1893 and LC 1109 — this problem is essentially LC 1893's coverage check, but counting covered points instead of verifying full coverage over a fixed range.
* The `diff[end + 1] -= 1` marker is what makes coverage "turn off" exactly one step past a car's end — the array needs one extra slot beyond the maximum possible coordinate to accommodate this safely.
* Given the problem's tiny fixed coordinate range (`0` to `100`), the difference array can be a small fixed-size array instead of a general hash map, keeping both approaches fast in practice despite the brute force's worse theoretical complexity.

## Code

```cpp
class Solution {
public:
    int numberOfPoints(vector<vector<int>>& nums) {
        vector<int> diff(102, 0);

        for (auto& car : nums) {
            diff[car[0]]++;
            diff[car[1] + 1]--;
        }

        int running = 0, count = 0;
        for (int i = 0; i <= 100; i++) {
            running += diff[i];
            if (running > 0) {
                count++;
            }
        }

        return count;
    }
};
```

---

## Key Template

```text
diff = array of size (maxCoordinate + 2), all 0

for [start, end] in nums:
    diff[start] += 1
    diff[end + 1] -= 1

running = 0
count = 0
for i = 0 to maxCoordinate:
    running += diff[i]
    if running > 0:
        count += 1

return count
```