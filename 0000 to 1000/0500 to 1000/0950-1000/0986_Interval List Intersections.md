# LeetCode 986 — Interval List Intersections

## Metadata

* **LeetCode:** 986
* **Problem:** Interval List Intersections
* **Difficulty:** Medium
* **Topics:** Array, Two Pointers
* **Pattern:** Two Pointers on Two Sorted Interval Lists
* **Key Technique:** Since both lists are already sorted and disjoint within themselves, walk two pointers simultaneously — compute the overlap of the two current intervals directly, then advance whichever interval ends first (it can never overlap with anything further ahead)
* **Optimal Complexity:** `O(n + m)` Time, `O(1)` Auxiliary Space (excluding the output)

---

## Problem Statement

Given two lists of closed intervals `firstList` and `secondList`, each pairwise disjoint and sorted, return the intersection of the two interval lists (a list of closed intervals representing overlapping ranges).

---

## Approaches

1. **Brute Force — Check Every Pair of Intervals**
2. **Optimal — Two Pointers on Both Lists**

---

# Approach 1 — Brute Force / Check Every Pair of Intervals

## Idea

For every interval in `firstList`, check it against every interval in `secondList`, computing their overlap directly whenever one exists.

## Dry Run

```text
firstList = [[0,2],[5,10],[13,23],[24,25]]
secondList = [[1,5],[8,12],[15,24],[25,26]]
```

Check `[0,2]` against `[1,5]`:

```text
overlap: max(0,1)=1, min(2,5)=2 → 1<=2 → valid overlap [1,2]
```

Check `[0,2]` against `[8,12]`:

```text
overlap: max(0,8)=8, min(2,12)=2 → 8>2 → no overlap
```

Continue checking every pair from both lists (`4 * 4 = 16` pairs total) to collect all valid overlaps.

Final intersections: `[1,2],[5,5],[8,10],[15,23],[24,24],[25,25]`.

## Algorithm

1. Initialize an empty result list.
2. For each interval `a` in `firstList`:
3. For each interval `b` in `secondList`:

   * Compute `start = max(a[0], b[0])`, `end = min(a[1], b[1])`.
   * If `start <= end`, append `[start, end]` to the result.
4. Return the result.

## Complexity

* **Time:** `O(n * m)`

  * Every pair of intervals from the two lists is checked directly.
* **Space:** `O(1)` (beyond the required output)

  * Only a couple of temporary variables per comparison.

## Notes / Tips

* Checking every pair ignores the fact that both lists are already sorted. Once an interval from one list ends before the current interval from the other list even starts, no interval further back in either list needs to be reconsidered.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> intervalIntersection(vector<vector<int>>& firstList, vector<vector<int>>& secondList) {
        vector<vector<int>> result;

        for (auto& a : firstList) {
            for (auto& b : secondList) {
                int start = max(a[0], b[0]);
                int end = min(a[1], b[1]);

                if (start <= end) {
                    result.push_back({start, end});
                }
            }
        }

        return result;
    }
};
```

---

# Approach 2 — Optimal / Two Pointers on Both Lists

## Idea

Since both `firstList` and `secondList` are already sorted and internally non-overlapping, walk through them simultaneously with two pointers `i` and `j`. At each step, compute the overlap (if any) between `firstList[i]` and `secondList[j]` directly — no need to compare against any other interval, since sorted order guarantees nothing earlier could still be relevant and nothing later could overlap yet. After checking, advance whichever interval ends first: that interval can no longer overlap with *anything* remaining in the other list, since all future intervals in that list start even later.

## Dry Run

```text
firstList = [[0,2],[5,10],[13,23],[24,25]]
secondList = [[1,5],[8,12],[15,24],[25,26]]
```

`i=0` (`[0,2]`), `j=0` (`[1,5]`):

```text
overlap: max(0,1)=1, min(2,5)=2 → [1,2] → record
[0,2] ends at 2, [1,5] ends at 5 → 2 < 5 → advance i
```

`i=1` (`[5,10]`), `j=0` (`[1,5]`):

```text
overlap: max(5,1)=5, min(10,5)=5 → [5,5] → record
[5,10] ends at 10, [1,5] ends at 5 → 5 < 10 → advance j
```

`i=1` (`[5,10]`), `j=1` (`[8,12]`):

```text
overlap: max(5,8)=8, min(10,12)=10 → [8,10] → record
[5,10] ends at 10, [8,12] ends at 12 → 10 < 12 → advance i
```

`i=2` (`[13,23]`), `j=1` (`[8,12]`):

```text
overlap: max(13,8)=13, min(23,12)=12 → 13 > 12 → no overlap
[13,23] ends at 23, [8,12] ends at 12 → 12 < 23 → advance j
```

`i=2` (`[13,23]`), `j=2` (`[15,24]`):

```text
overlap: max(13,15)=15, min(23,24)=23 → [15,23] → record
[13,23] ends at 23, [15,24] ends at 24 → 23 < 24 → advance i
```

`i=3` (`[24,25]`), `j=2` (`[15,24]`):

```text
overlap: max(24,15)=24, min(25,24)=24 → [24,24] → record
[24,25] ends at 25, [15,24] ends at 24 → 24 < 25 → advance j
```

`i=3` (`[24,25]`), `j=3` (`[25,26]`):

```text
overlap: max(24,25)=25, min(25,26)=25 → [25,25] → record
[24,25] ends at 25, [25,26] ends at 26 → 25 < 26 → advance i
```

`i=4` reaches end of `firstList` → stop.

Final: `[1,2],[5,5],[8,10],[15,23],[24,24],[25,25]`, matching the brute-force result.

## Algorithm

1. Initialize `i = 0`, `j = 0`, and an empty result list.
2. While `i < firstList.size()` and `j < secondList.size()`:

   * Compute `start = max(firstList[i][0], secondList[j][0])`, `end = min(firstList[i][1], secondList[j][1])`.
   * If `start <= end`, append `[start, end]` to the result.
   * If `firstList[i][1] < secondList[j][1]`, advance `i`; otherwise advance `j`.
3. Return the result.

## Complexity

* **Time:** `O(n + m)`

  * Each pointer advances forward through its own list at most once, for a combined total of `n + m` steps.
* **Space:** `O(1)` (beyond the required output)

  * Only two pointers and a couple of temporary variables — no extra structures.

## Notes / Tips

* Advancing based on whichever interval **ends first** (and not, say, whichever starts first) is the key correctness detail. An interval that ends earlier genuinely can't overlap with any future interval in the other list, while the one that ends later might still overlap with what comes next in the other list.
* Both lists must be sorted and internally non-overlapping for this two-pointer technique to work correctly — the problem's guarantees are what make it valid, not something the algorithm needs to check or enforce itself.
* This is the standard "merge/intersect two sorted sequences" two-pointer shape — closely related to the merge step of merge sort, but comparing and combining ranges instead of single values.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> intervalIntersection(vector<vector<int>>& firstList, vector<vector<int>>& secondList) {
        vector<vector<int>> result;
        int i = 0, j = 0;

        while (i < (int)firstList.size() && j < (int)secondList.size()) {
            int start = max(firstList[i][0], secondList[j][0]);
            int end = min(firstList[i][1], secondList[j][1]);

            if (start <= end) {
                result.push_back({start, end});
            }

            if (firstList[i][1] < secondList[j][1]) {
                i++;
            } else {
                j++;
            }
        }

        return result;
    }
};
```

---

## Key Template

```text
i = 0, j = 0
result = []

while i < firstList.size() and j < secondList.size():
    start = max(firstList[i][0], secondList[j][0])
    end = min(firstList[i][1], secondList[j][1])

    if start <= end:
        result.append([start, end])

    if firstList[i][1] < secondList[j][1]:
        i += 1
    else:
        j += 1

return result
```