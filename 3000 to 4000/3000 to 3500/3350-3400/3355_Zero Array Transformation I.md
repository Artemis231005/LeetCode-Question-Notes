# LeetCode 3355 — Zero Array Transformation I

## Metadata

* **LeetCode:** 3355
* **Problem:** Zero Array Transformation I
* **Difficulty:** Medium
* **Topics:** Array, Prefix Sum
* **Pattern:** Difference Array (Range Update, Point Query)
* **Key Technique:** Mark `+1` at each query's start and `-1` just after its end, then a running prefix sum gives the total number of queries covering each index — check that it's always at least as large as that index's value
* **Optimal Complexity:** `O(n + q)` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given an integer array `nums` and a list of queries `[li, ri]` (each allowing at most one decrement of any single element within `nums[li..ri]`), return `true` if it's possible to apply queries so that every element of `nums` becomes `0`. Each query, if used, decrements exactly one chosen index within its range by `1`; queries don't need to be used, and using a query still only decrements one index by `1` (not the whole range).

---

## Approaches

1. **Brute Force — Count Queries Covering Each Index Directly**
2. **Optimal — Difference Array**

---

# Approach 1 — Brute Force / Count Queries Covering Each Index Directly

## Idea

For every index, directly count how many queries' ranges include it — that count is the maximum number of times that index could possibly be decremented (since each covering query can contribute at most one decrement to it, and every query can be assigned freely to whichever index needs it most within its range). If every index's `nums[i]` is at most this covering count, decrements can always be distributed appropriately to zero out the array.

## Dry Run

```text
nums = [1, 0, 1], queries = [[0,2]]
```

Count queries covering each index:

```text
index 0: covered by [0,2] → count = 1
index 1: covered by [0,2] → count = 1
index 2: covered by [0,2] → count = 1
```

Check feasibility:

```text
nums[0]=1 <= count[0]=1 → ok
nums[1]=0 <= count[1]=1 → ok
nums[2]=1 <= count[2]=1 → ok
```

All indices satisfied → return `true`.

## Algorithm

1. Initialize a `count` array of size `n`, all zeros.
2. For each query `[l, r]`:

   * For `i` from `l` to `r`:

     * `count[i] += 1`.
3. For each index `i`, if `nums[i] > count[i]`, return `false` (not enough covering queries to zero it out).
4. If every index passes, return `true`.

## Complexity

* **Time:** `O(n * q)`

  * Each of the `q` queries can directly touch up to `n` indices.
* **Space:** `O(n)`

  * For the `count` array.

## Notes / Tips

* Directly incrementing every index a query covers is redundant, a query's effect only needs to be marked once, at its boundaries, rather than applied to every index inside it, using the difference array technique.
* The core correctness insight — "an index can be zeroed exactly when the number of queries covering it is at least its own value" — holds regardless of how the counting is computed; this is what makes the difference array useful here rather than a different algorithm.

## Code

```cpp
class Solution {
public:
    bool isZeroArray(vector<int>& nums, vector<vector<int>>& queries) {
        int n = nums.size();
        vector<int> count(n, 0);

        for (auto& q : queries) {
            for (int i = q[0]; i <= q[1]; i++) {
                count[i]++;
            }
        }

        for (int i = 0; i < n; i++) {
            if (nums[i] > count[i]) {
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

Instead of incrementing every index a query covers, mark the query's *effect* directly: add `+1` at the query's start, and subtract `1` just after its end. Then sweep through the array once, accumulating a running sum — at each index, that running sum is exactly the number of queries covering it, which can then be directly compared against `nums[i]`.

## Dry Run

```text
nums = [4, 3, 2, 1], queries = [[1,3],[0,2]]
```

Build difference array `diff` of size `n + 1`:

```text
[1,3] → diff[1] += 1, diff[4] -= 1
[0,2] → diff[0] += 1, diff[3] -= 1
```

```text
diff = [1, 1, 0, -1, -1]
```

Prefix sum sweep, checking feasibility as we go:

```text
i=0: running=1 → count[0]=1 → nums[0]=4 > 1 → infeasible → return false
```

(In this example the answer is `false` since index `0` needs `4` decrements but only `1` query covers it.)

### A feasible example

```text
nums = [1, 0, 1], queries = [[0,2]]
```

```text
diff[0] += 1, diff[3] -= 1 → diff = [1, 0, 0, -1]
```

Sweep:

```text
i=0: running=1 → nums[0]=1 <= 1 → ok
i=1: running=1 → nums[1]=0 <= 1 → ok
i=2: running=1 → nums[2]=1 <= 1 → ok
```

All indices pass → return `true`.

## Algorithm

1. Create a difference array `diff` of size `n + 1`, all zeros.
2. For each query `[l, r]`:

   * `diff[l] += 1`.
   * `diff[r + 1] -= 1`.
3. Sweep `i` from `0` to `n - 1`, maintaining a running sum:

   * `running += diff[i]`.
   * If `nums[i] > running`, return `false` immediately.
4. If the sweep completes without any index failing, return `true`.

## Complexity

* **Time:** `O(n + q)`

  * `O(q)` to apply all query markers, `O(n)` to sweep through and check each index.
* **Space:** `O(n)`

  * For the difference array, sized independently of `q`.

## Notes / Tips

* Checking `nums[i] > running` during the sweep (rather than building the full `count` array first and checking afterward) allows early exit the moment infeasibility is detected, without needing a separate pass.
* The correctness of comparing `nums[i]` directly against the covering-query count relies on the fact that decrements can be freely assigned to *any* single index within a used query's range — so as long as each index has at least as many covering queries as its required decrements, a valid assignment (giving each query's one decrement to whichever index still needs it) is always achievable; this is essentially a bipartite-matching feasibility argument simplified down to a per-index counting check.

## Code

```cpp
class Solution {
public:
    bool isZeroArray(vector<int>& nums, vector<vector<int>>& queries) {
        int n = nums.size();
        vector<int> diff(n + 1, 0);

        for (auto& q : queries) {
            diff[q[0]]++;
            diff[q[1] + 1]--;
        }

        int running = 0;
        for (int i = 0; i < n; i++) {
            running += diff[i];
            if (nums[i] > running) {
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
diff = array of size (n + 1), all 0

for [l, r] in queries:
    diff[l] += 1
    diff[r + 1] -= 1

running = 0
for i = 0 to n-1:
    running += diff[i]
    if nums[i] > running:
        return false

return true
```