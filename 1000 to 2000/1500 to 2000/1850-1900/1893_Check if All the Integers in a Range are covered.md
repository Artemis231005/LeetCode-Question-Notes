# LeetCode 1893 — Check if All the Integers in a Range Are Covered

## Metadata

* **LeetCode:** 1893
* **Problem:** Check if All the Integers in a Range Are Covered
* **Difficulty:** Easy
* **Topics:** Array, Hash Table, Prefix Sum, Sorting
* **Pattern:** Difference Array (Range Update, Point Query)
* **Key Technique:** Mark `+1` at each interval's start and `-1` just after its end, then a running prefix sum tells you how many intervals cover each point
* **Optimal Complexity:** `O(range + n)` Time, `O(range)` Space

---

## Problem Statement

Given a 2D array `ranges` where `ranges[i] = [starti, endi]` represents an inclusive interval, and two integers `left` and `right`, return `true` if every integer in `[left, right]` is covered by at least one interval in `ranges`.

---

## Approaches

1. **Brute Force — Check Every Integer Against Every Interval**
2. **Optimal — Difference Array**

---

# Approach 1 — Brute Force / Check Every Integer Against Every Interval

## Idea

For every integer `x` from `left` to `right`, scan through all the intervals in `ranges` to see if any of them contains `x`. If any integer has no covering interval, the answer is `false`.

## Dry Run

```text
ranges = [[1,2],[3,4],[5,6]], left = 2, right = 5
```

`x = 2`:

```text
[1,2] contains 2 → covered
```

`x = 3`:

```text
[3,4] contains 3 → covered
```

`x = 4`:

```text
[3,4] contains 4 → covered
```

`x = 5`:

```text
[5,6] contains 5 → covered
```

All covered → return `true`.

## Algorithm

1. For each integer `x` from `left` to `right`:

   * Set `covered = false`.
   * For each interval `[start, end]` in `ranges`:

     * If `start <= x <= end`, set `covered = true` and break.
   * If `covered` is `false`, return `false`.
2. If every integer was covered, return `true`.

## Complexity

* **Time:** `O((right - left) * n)`

  * For each integer in `[left, right]`, all `n` intervals may need to be checked.
* **Space:** `O(1)`

  * Only a boolean flag per integer — no extra structures allocated.

## Notes / Tips

* Rechecking every interval for every integer is redundant — each interval's coverage only needs to be "applied" once, not re-derived for every point it touches.
* Fine for small ranges/interval counts, but scales poorly if both grow.

## Code

```cpp
class Solution {
public:
    bool isCovered(vector<vector<int>>& ranges, int left, int right) {
        for (int x = left; x <= right; x++) {
            bool covered = false;

            for (auto& range : ranges) {
                if (range[0] <= x && x <= range[1]) {
                    covered = true;
                    break;
                }
            }

            if (!covered) {
                return false;
            }
        }

        return true;
    }
};
```

---

# Approach 2 — Optimal / Difference Array

## Idea

Instead of checking each point against every interval, mark each interval's *effect* directly: increment a counter at the interval's start, and decrement it just after the interval's end. Then sweep through the array once, accumulating a running sum — at any point, that running sum tells you exactly how many intervals cover that point. A point with a running sum of `0` means no interval covers it.

## Dry Run

```text
ranges = [[1,2],[3,4],[5,6]], left = 2, right = 5
```

Constraints guarantee values stay within `1..50`, so use a difference array of size `52` (to safely handle `end + 1` up to `51`).

Apply each interval:

```text
[1,2] → diff[1] += 1, diff[3] -= 1
[3,4] → diff[3] += 1, diff[5] -= 1
[5,6] → diff[5] += 1, diff[7] -= 1
```

```text
diff = [.., 1(at 1), 0, 1(at 3)-1(at 3)=0, ..., ]
```

More precisely, tracking just the nonzero deltas:

```text
diff[1] = 1
diff[3] = -1 + 1 = 0
diff[5] = -1 + 1 = 0
diff[7] = -1
```

Prefix sum sweep (running coverage count):

```text
i=1: running = 1
i=2: running = 1
i=3: running = 1 (diff[3]=0, no change)
i=4: running = 1
i=5: running = 1 (diff[5]=0, no change)
i=6: running = 1
i=7: running = 0 (diff[7]=-1)
```

Check `left=2` to `right=5`: running coverage at `2,3,4,5` are all `1` (i.e. `>= 1`) → return `true`.

## Algorithm

1. Create a difference array `diff` of size `52` (or `maxRange + 2`), all zeros.
2. For each interval `[start, end]` in `ranges`:

   * `diff[start] += 1`.
   * `diff[end + 1] -= 1`.
3. Sweep from `1` to `right`, maintaining a running sum `covered = 0`:

   * `covered += diff[i]`.
   * If `i >= left` and `covered == 0`, return `false`.
4. If the sweep completes without finding an uncovered point, return `true`.

## Complexity

* **Time:** `O(range + n)`

  * `O(n)` to apply all interval markers, `O(range)` to sweep through and accumulate the running sum, where `range` is the fixed bound on possible values (here, up to `50`).
* **Space:** `O(range)`

  * For the fixed-size difference array, independent of `n`.

## Notes / Tips

* This is the standard "difference array" (a.k.a. range-update-point-query) technique — mark the start and end+1 of each range once, then let a prefix sum sweep do the work of "applying" every interval's effect across all the points it touches, instead of checking each point against every interval individually.
* The `diff[end + 1] -= 1` marker is what makes coverage "turn off" exactly one step past the interval's end — this is why the array needs one extra slot beyond the maximum possible value.
* Given this problem's constraints (values bounded to `1..50`), the difference array can be a small fixed-size array instead of a general hash map — same idea as bucket-based counting when the value range is known and small.

## Code

```cpp
class Solution {
public:
    bool isCovered(vector<vector<int>>& ranges, int left, int right) {
        vector<int> diff(52, 0);

        for (auto& range : ranges) {
            diff[range[0]]++;
            diff[range[1] + 1]--;
        }

        int covered = 0;
        for (int i = 1; i <= right; i++) {
            covered += diff[i];

            if (i >= left && covered == 0) {
                return false;
            }
        }

        return true;
    }
};
```

---

## Key Template

```text
diff = array of size (maxValue + 2), all 0

for [start, end] in ranges:
    diff[start] += 1
    diff[end + 1] -= 1

covered = 0
for i = 1 to right:
    covered += diff[i]
    if i >= left and covered == 0:
        return false

return true
```