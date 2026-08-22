# Non-overlapping Intervals

## Problem

Given an array of intervals where `intervals[i] = [start, end]`, return the **minimum number of intervals that must be removed** to make the remaining intervals non-overlapping.

Example:

```text
intervals = [[1,2], [2,3], [3,4], [1,3]]
Output: 1
```

Removing `[1,3]` leaves non-overlapping intervals.

---

## Approach 1: Greedy — Sort by Ending Time

### Idea

To keep the maximum number of non-overlapping intervals, always choose the interval that **ends earliest**.

Why?

An interval with a smaller end leaves more space for future intervals.

Sort intervals by their ending point. Then:

* Keep an interval if its start is `>=` the end of the previously kept interval.
* Otherwise, it overlaps, so remove it.

The answer is the number of overlapping intervals removed.

### Dry Run

```text
intervals = [[1,2], [2,3], [3,4], [1,3]]

Sorted by end:
[1,2], [2,3], [1,3], [3,4]

prevEnd = 2

[2,3] → 2 >= 2 → keep
[1,3] → 1 < 3 → remove
[3,4] → 3 >= 3 → keep

Answer = 1
```

### Algorithm

1. Sort intervals by their ending time.
2. Set `prevEnd` to the end of the first interval.
3. Traverse the remaining intervals.
4. If `start >= prevEnd`:

   * Keep the interval.
   * Update `prevEnd = end`.
5. Otherwise:

   * The interval overlaps.
   * Increment the removal count.
6. Return the removal count.

### Complexity

* Time: `O(n log n)` due to sorting
* Space: `O(1)` excluding sorting space

### Code

```cpp
class Solution {
public:
    int eraseOverlapIntervals(vector<vector<int>>& intervals) {
        sort(intervals.begin(), intervals.end(),
             [](const vector<int>& a, const vector<int>& b) {
                 return a[1] < b[1];
             });

        int removals = 0;
        int prevEnd = intervals[0][1];

        for (int i = 1; i < intervals.size(); i++) {
            if (intervals[i][0] >= prevEnd) {
                prevEnd = intervals[i][1];
            }
            else {
                removals++;
            }
        }

        return removals;
    }
};
```

### Notes / Tips

* This is a classic **Greedy + Interval Scheduling** problem.
* Think: **"Keep intervals that finish earliest."**
* `[1,2]` and `[2,3]` do **not** overlap because `2 >= 2`.
* We are maximizing the number of intervals kept, which is equivalent to minimizing the number removed.
* Do **not** sort by starting time; sorting by ending time is the key greedy choice.
* If intervals are empty, return `0`.

### Key Template

```text
sort intervals by end

prevEnd = end of first interval
removals = 0

for each next interval:
    if start >= prevEnd:
        keep it
        prevEnd = end
    else:
        remove it
        removals++

return removals
```
