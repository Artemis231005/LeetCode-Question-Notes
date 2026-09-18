# LeetCode 3356 — Zero Array Transformation II

## Metadata

* **LeetCode:** 3356
* **Problem:** Zero Array Transformation II
* **Difficulty:** Medium
* **Topics:** Array, Binary Search, Prefix Sum
* **Pattern:** Binary Search on Answer + Difference Array Feasibility Check
* **Key Technique:** Feasibility (can `nums` be zeroed using the first `k` queries) is monotonic in `k` — more queries can only add decrement capacity, never remove it — so binary search for the smallest feasible `k`, checking each candidate with a difference array sweep
* **Optimal Complexity:** `O((n + q) log q)` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given an array `nums` and queries `[li, ri, vali]` (each allowing, for every index in `[li, ri]`, an independently-chosen decrement of **up to** `vali`), return the minimum number of queries (taken as a prefix, in order) needed to reduce `nums` entirely to zero, or `-1` if impossible even using all queries.

---

## Approaches

1. **Brute Force — Linear Scan Over k, Direct Range Increment**
2. **Better — Linear Scan Over k, Incremental Difference Array**
3. **Optimal — Binary Search on k + Difference Array Feasibility Check**

---

# Approach 1 — Brute Force / Linear Scan Over k, Direct Range Increment

## Idea

Try `k = 1, 2, 3, ...` in order. For each candidate `k`, rebuild a "capacity" array completely from scratch using the first `k` queries: for every query, loop directly over its entire range and add `val` to each covered index. Check whether every index's capacity now meets or exceeds `nums[i]`. Stop at the first `k` that works.

## Dry Run

```text
nums = [2, 0, 2], queries = [[0,2,1],[0,2,1],[1,1,3]]
```

`k = 1`: apply query `[0,2,1]` directly (add `1` to indices `0,1,2`):

```text
capacity = [1, 1, 1]
check: nums[0]=2 > 1 → infeasible
```

`k = 2`: rebuild from scratch using queries `0` and `1` (both `[0,2,1]`):

```text
capacity = [2, 2, 2]
check: nums[0]=2<=2, nums[1]=0<=2, nums[2]=2<=2 → feasible!
```

Return `k = 2`.

## Algorithm

1. For `k` from `1` to `queries.size()`:

   * Rebuild `capacity` array of size `n`, all zeros.
   * For each of the first `k` queries `[l, r, val]`:

     * For `i` from `l` to `r`, `capacity[i] += val`.
   * If `nums[i] <= capacity[i]` for every `i`, return `k`.
2. If no `k` works even using all queries, return `-1`.

## Complexity

* **Time:** `O(q² * n)`

  * In the worst case, up to `q` candidate values of `k` are tried, each requiring a full rebuild that touches up to `q * n` cells (each of up to `k <= q` queries directly incrementing up to `n` indices).
* **Space:** `O(n)`

  * For the `capacity` array, rebuilt fresh each iteration.

## Notes / Tips

* Rebuilding the entire capacity array from scratch for every candidate `k` is very wasteful — almost all of the work from the previous `k` is thrown away and redone.
* This also directly increments every index in each range, repeating the same inefficiency.

## Code

```cpp
class Solution {
public:
    int minZeroArray(vector<int>& nums, vector<vector<int>>& queries) {
        int n = nums.size();

        for (int k = 1; k <= (int)queries.size(); k++) {
            vector<long long> capacity(n, 0);

            for (int i = 0; i < k; i++) {
                int l = queries[i][0], r = queries[i][1], val = queries[i][2];
                for (int idx = l; idx <= r; idx++) {
                    capacity[idx] += val;
                }
            }

            bool feasible = true;
            for (int i = 0; i < n; i++) {
                if (nums[i] > capacity[i]) {
                    feasible = false;
                    break;
                }
            }

            if (feasible) {
                return k;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Better / Linear Scan Over k, Incremental Difference Array

## Idea

Still try `k = 1, 2, 3, ...` in order, but stop rebuilding the capacity structure from scratch each time. Instead, maintain a difference array across iterations: each time `k` grows by one, apply just the newest query's `+val`/`-val` markers to the difference array in `O(1)`, then run a single `O(n)` prefix sum sweep to check feasibility with the updated array.

## Dry Run

```text
nums = [2, 0, 2], queries = [[0,2,1],[0,2,1],[1,1,3]]
```

`k = 1`: apply query `0` to the difference array:

```text
diff[0] += 1, diff[3] -= 1 → diff = [1, 0, 0, -1]
```

Sweep to check feasibility:

```text
capacity = [1, 1, 1]
nums[0]=2 > 1 → infeasible
```

`k = 2`: apply query `1` (also `[0,2,1]`) to the *same* difference array (no rebuild):

```text
diff[0] += 1, diff[3] -= 1 → diff = [2, 0, 0, -2]
```

Sweep:

```text
capacity = [2, 2, 2]
nums[0]=2<=2, nums[1]=0<=2, nums[2]=2<=2 → feasible!
```

Return `k = 2`.

## Algorithm

1. Initialize a difference array `diff` of size `n + 1`, all zeros.
2. For `k` from `1` to `queries.size()`:

   * Apply the `k`-th query `[l, r, val]` to `diff`: `diff[l] += val`, `diff[r + 1] -= val`.
   * Sweep `diff` with a running prefix sum to build `capacity`, and check `nums[i] <= capacity[i]` for every `i`.
   * If feasible, return `k`.
3. If no `k` works, return `-1`.

## Complexity

* **Time:** `O(q * n)`

  * Each of the `q` iterations does `O(1)` work to extend the difference array and `O(n)` work for the feasibility sweep.
* **Space:** `O(n)`

  * For the difference array, updated in place across iterations rather than rebuilt.

## Notes / Tips

* This removes the wasteful full-rebuild cost of Approach 1 — each new query is folded into the existing difference array in constant time — but a full `O(n)` feasibility sweep is still performed after every single query, even when it's obvious most candidate `k` values will fail early in the sequence.
* This is a good intermediate step: applying the difference-array technique (from LC 1109 / LC 2848) already helps, but scanning `k` linearly from `1` upward still doesn't exploit any higher-level structure in the problem.

## Code

```cpp
class Solution {
public:
    int minZeroArray(vector<int>& nums, vector<vector<int>>& queries) {
        int n = nums.size();
        vector<long long> diff(n + 1, 0);

        for (int k = 1; k <= (int)queries.size(); k++) {
            int l = queries[k-1][0], r = queries[k-1][1], val = queries[k-1][2];
            diff[l] += val;
            diff[r + 1] -= val;

            long long running = 0;
            bool feasible = true;
            for (int i = 0; i < n; i++) {
                running += diff[i];
                if (nums[i] > running) {
                    feasible = false;
                    break;
                }
            }

            if (feasible) {
                return k;
            }
        }

        return -1;
    }
};
```

---

# Approach 3 — Optimal / Binary Search on k + Difference Array Feasibility Check

## Idea

Feasibility as a function of `k` is **monotonic**: using more queries can only ever add more decrement capacity (every `val >= 1`), never remove any — so if some `k` is feasible, every larger `k` is also feasible. This monotonic "yes/no" structure is exactly what binary search on the answer needs. Instead of checking every `k` from `1` upward, binary search directly for the smallest feasible `k`, using a difference array (built fresh for just the first `mid` queries) to check feasibility at each candidate in `O(n + mid)`.

## Dry Run

```text
nums = [2, 0, 2], queries = [[0,2,1],[0,2,1],[1,1,3]]
```

Binary search bounds: `low = 0`, `high = 3` (total queries).

```text
mid = 1: build diff using first 1 query → capacity = [1,1,1]
   nums[0]=2 > 1 → infeasible → low = 2

mid = (2+3+1)/2 = 3: build diff using first 3 queries
   query0 [0,2,1], query1 [0,2,1], query2 [1,1,3]
   diff: [0]+=1,[3]-=1 (q0); [0]+=1,[3]-=1 (q1); [1]+=3,[2]-=3 (q2)
   capacity = [2, 5, 2] → nums[0]=2<=2, nums[1]=0<=5, nums[2]=2<=2 → feasible → high = 3
```

Wait, binary search needs to narrow toward the smallest feasible value — re-tracing properly:

```text
low = 0, high = 3

mid = (0+3+1)/2 = 2: build diff using first 2 queries (both [0,2,1])
   capacity = [2,2,2] → all satisfied → feasible → high = 2

mid = (0+2+1)/2 = 1: build diff using first 1 query
   capacity = [1,1,1] → nums[0]=2 > 1 → infeasible → low = 2
```

`low == high == 2` → answer `k = 2`, matching the earlier approaches.

## Algorithm

1. Define a helper `feasible(k)`:

   * Build a difference array using only the first `k` queries.
   * Sweep it to get `capacity`, and check `nums[i] <= capacity[i]` for every `i`.
   * Return whether all indices pass.
2. If `feasible(queries.size())` is `false`, return `-1` (even all queries aren't enough).
3. Binary search `k` between `0` and `queries.size()`:

   * If `feasible(mid)`, the answer is at most `mid` — move `high = mid`.
   * Otherwise, move `low = mid + 1`.
4. Return `low` (the smallest feasible `k`), or `0` if `nums` is already all zeros to begin with (feasibility check naturally handles this at `k=0`).

## Complexity

* **Time:** `O((n + q) log q)`

  * Binary search runs `O(log q)` iterations, each feasibility check costing `O(n + mid)` to build the difference array and sweep it (bounded by `O(n + q)`).
* **Space:** `O(n)`

  * For the difference array rebuilt fresh inside each feasibility check.

## Notes / Tips

* The monotonicity argument — "more queries only ever help, never hurt, since every `val >= 1`" — is what justifies binary search here; this is the same "binary search on the answer" pattern used in LC 1292 (Maximum Side Length of a Square...), just with feasibility defined over a count of queries instead of a side length.
* Rebuilding the difference array fresh inside each `feasible(k)` call (rather than reusing state across binary search steps, which isn't straightforward since binary search jumps around non-sequentially) keeps each check simple and correct, at the cost of `O(n + mid)` per check rather than `O(1)` — still far fewer total checks than Approach 2's linear scan.
* Checking `feasible(queries.size())` first is important — if even using every query still can't zero out `nums`, no smaller `k` can either, and binary search should be skipped entirely amad it should return `-1` directly.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> queries;
    vector<int> nums;
    int n;

    bool feasible(int k) {
        vector<long long> diff(n + 1, 0);

        for (int i = 0; i < k; i++) {
            int l = queries[i][0], r = queries[i][1], val = queries[i][2];
            diff[l] += val;
            diff[r + 1] -= val;
        }

        long long running = 0;
        for (int i = 0; i < n; i++) {
            running += diff[i];
            if (nums[i] > running) {
                return false;
            }
        }

        return true;
    }

    int minZeroArray(vector<int>& nums, vector<vector<int>>& queries) {
        this->nums = nums;
        this->queries = queries;
        n = nums.size();

        if (!feasible(queries.size())) {
            return -1;
        }

        int low = 0, high = queries.size();

        while (low < high) {
            int mid = low + (high - low) / 2;

            if (feasible(mid)) {
                high = mid;
            } else {
                low = mid + 1;
            }
        }

        return low;
    }
};
```

---

## Key Template

```text
function feasible(k):
    diff = array of size (n + 1), all 0
    for i in 0..k-1:
        [l, r, val] = queries[i]
        diff[l] += val
        diff[r + 1] -= val

    running = 0
    for i in 0..n-1:
        running += diff[i]
        if nums[i] > running:
            return false
    return true

if not feasible(queries.size()): return -1

low = 0, high = queries.size()
while low < high:
    mid = low + (high - low) / 2
    if feasible(mid): high = mid
    else: low = mid + 1

return low
```