# LeetCode 3362 — Zero Array Transformation III

## Metadata

* **LeetCode:** 3362
* **Problem:** Zero Array Transformation III
* **Difficulty:** Medium
* **Topics:** Array, Greedy, Heap (Priority Queue), Prefix Sum
* **Pattern:** Greedy Max-Heap Interval Selection + Difference Array
* **Key Technique:** Sweep left to right; whenever the current index still needs more decrement coverage, greedily activate the **available** query that reaches the **furthest right** (from a max-heap ordered by end point) — this maximizes future usefulness of every query actually used
* **Optimal Complexity:** `O((n + q) log q)` Time, `O(n + q)` Auxiliary Space

---

## Problem Statement

Given an array `nums` and queries `[li, ri]` (each allowing, for every index in `[li, ri]`, an independently-chosen decrement of **at most 1**), return the maximum number of queries that can be **removed** such that the *remaining* queries can still reduce `nums` entirely to zero — using any subset of the remaining queries, in any way, not restricted to a prefix. Return `-1` if even all queries together can't zero `nums`.

---

## Approaches

1. **Brute Force — Try Every Subset of Queries**
2. **Optimal — Greedy Max-Heap + Difference Array**

---

# Approach 1 — Brute Force / Try Every Subset of Queries

## Idea

Since any subset of the queries can be used (not just a prefix), directly try every possible subset, from smallest to largest, checking each one's feasibility with a difference array sweep. The first subset size that has *any* feasible subset gives the minimum number of queries needed to keep; the answer is `total queries - that minimum`.

## Dry Run

```text
nums = [2, 0, 2], queries = [[0,2],[0,2],[1,1]]
```

Try subset size `0`: only the empty subset — feasible only if `nums` is already all zero → not the case here.

Try subset size `1`: check each single query alone:

```text
{query0}: capacity = [1,1,1] → nums[0]=2 > 1 → infeasible
{query1}: same as query0 → infeasible
{query2}: capacity = [0,1,0] → nums[0]=2 > 0 → infeasible
```

Try subset size `2`: check each pair:

```text
{query0, query1}: capacity = [2,2,2] → nums[0]=2<=2, nums[1]=0<=2, nums[2]=2<=2 → feasible!
```

Minimum kept = `2`. Total queries = `3`. Answer = `3 - 2 = 1`.

## Algorithm

1. For `size` from `0` to `queries.size()`:

   * For every subset of `queries` of exactly this `size`:

     * Build a difference array from just this subset, sweep it, and check `nums[i] <= capacity[i]` for every `i`.
     * If feasible, return `queries.size() - size`.
2. If no subset (including the full set) is feasible, return `-1`.

## Complexity

* **Time:** `O(2^q * n)`

  * Every possible subset of the `q` queries is potentially checked, each requiring an `O(n)` feasibility sweep.
* **Space:** `O(n + q)`

  * For the difference array and the recursion/subset-generation stack.

## Notes / Tips

* This is very slow even for modest `q`,  but it's the most direct translation of the problem statement ("any subset of remaining queries" implies no assumed ordering or structure to exploit without further insight).
* The key realization that unlocks Approach 2: even though queries can be used in *any* combination, a **greedy left-to-right sweep that always prefers the query reaching furthest right when a decrement is needed** turns out to be optimal: this removes the need to search subsets at all.

## Code

```cpp
class Solution {
public:
    int n;
    vector<int> nums;

    bool feasible(vector<vector<int>>& queries, vector<int>& subsetIndices) {
        vector<long long> diff(n + 1, 0);

        for (int idx : subsetIndices) {
            diff[queries[idx][0]] += 1;
            diff[queries[idx][1] + 1] -= 1;
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

    bool tryAllSubsetsOfSize(vector<vector<int>>& queries, int size, int start,
                              vector<int>& current) {
        if ((int)current.size() == size) {
            return feasible(queries, current);
        }

        for (int i = start; i < (int)queries.size(); i++) {
            current.push_back(i);
            if (tryAllSubsetsOfSize(queries, size, i + 1, current)) {
                return true;
            }
            current.pop_back();
        }

        return false;
    }

    int maxRemoval(vector<int>& nums, vector<vector<int>>& queries) {
        this->nums = nums;
        n = nums.size();

        for (int size = 0; size <= (int)queries.size(); size++) {
            vector<int> current;
            if (tryAllSubsetsOfSize(queries, size, 0, current)) {
                return queries.size() - size;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Optimal / Greedy Max-Heap + Difference Array

## Idea

Sweep left to right across `nums`. Maintain a max-heap of currently *available* queries (those whose `l <= current index`), ordered by their **right endpoint** `r`. At each index `i`, track how much decrement coverage is already committed via a difference array. Whenever the committed coverage is less than `nums[i]`, greedily activate the available query with the **largest** `r` (popping and discarding any expired queries — those with `r < i` — along the way): choosing the furthest-reaching query first maximizes how much future demand that single query can also help satisfy, which is what makes this greedy choice optimal. If no available (non-expired) query remains but more coverage is still needed, it's impossible — return `-1`.

## Dry Run

```text
nums = [2, 0, 2], queries = [[0,2],[0,2],[1,1]]
```

Sort/process queries by their `l` value so they can be added to the heap exactly when reached. Initialize `diff` array, `running = 0`, `usedCount = 0`, heap empty.

`i = 0`: add queries with `l = 0`: `query0(r=2)`, `query1(r=2)` → heap has both.

```text
running += diff[0] = 0
need nums[0] = 2, running(0) < 2:
   pop query with max r (r=2) → use it: diff[0] += 1, diff[3] -= 1, running += 1 → 1, usedCount=1
   running(1) still < 2:
   pop next max r (r=2, the other one) → use it: diff[0] += 1, diff[3] -= 1, running += 1 → 2, usedCount=2
   running(2) >= 2 → satisfied
```

`i = 1`: add queries with `l = 1`: `query2(r=1)` → heap now also has query2.

```text
running += diff[1] = 0 → running stays 2
need nums[1] = 0, running(2) >= 0 → nothing to do (query2 never gets used)
```

`i = 2`: no new queries to add (`query2`'s `l=1` already passed).

```text
running += diff[2] = 0 → running stays 2
need nums[2] = 2, running(2) >= 2 → satisfied
```

Final `usedCount = 2`. Answer = `queries.size() - usedCount = 3 - 2 = 1`, matching the expected output.

## Algorithm

1. Sort queries by their `l` value (or process them via a pointer as `i` increases, if already convenient).
2. Initialize a difference array `diff` of size `n + 1`, `running = 0`, `usedCount = 0`, and a max-heap ordered by `r`.
3. For each index `i` from `0` to `n - 1`:

   * Add every query with `l == i` to the heap.
   * `running += diff[i]`.
   * While `running < nums[i]`:

     * While the heap's top query has `r < i` (expired), pop and discard it.
     * If the heap is empty, return `-1`.
     * Pop the query with the largest `r`; mark it used: `diff[i] += 1`, `diff[r + 1] -= 1`, `running += 1`, `usedCount += 1`.
4. Return `queries.size() - usedCount`.

## Complexity

* **Time:** `O((n + q) log q)`

  * Sorting queries by `l` is `O(q log q)`; each query is pushed and popped from the heap at most once (`O(log q)` each), and the main sweep over `n` indices does `O(1)` amortized work per index outside of heap operations.
* **Space:** `O(n + q)`

  * For the difference array and the heap, which can hold up to `q` queries.

## Notes / Tips

* Choosing the query with the **largest `r`** (not just any available query) when a decrement is needed is the crux of the greedy proof: among all queries currently available, the one reaching furthest right is guaranteed to be at least as useful for satisfying *future* indices as any other available query, so there's never a reason to prefer a shorter-reaching one.
* Discarding expired queries (`r < i`) from the top of the heap lazily — only when they'd otherwise be selected, avoids needing to actively remove them the moment they expire, which a plain array-based structure would require scanning for.
* This is a classic greedy-with-max-heap interval covering pattern, structurally similar to "minimum number of intervals to cover a range" problems — the twist here is that each covering interval only contributes `1` unit of capacity, so a position needing `nums[i]` units of coverage requires that many *distinct* covering intervals to be actively used simultaneously.

## Code

```cpp
class Solution {
public:
    int maxRemoval(vector<int>& nums, vector<vector<int>>& queries) {
        int n = nums.size();
        sort(queries.begin(), queries.end());

        priority_queue<int> maxHeap; // stores r values of available queries
        vector<long long> diff(n + 1, 0);
        long long running = 0;
        int usedCount = 0;
        int qIdx = 0;

        for (int i = 0; i < n; i++) {
            while (qIdx < (int)queries.size() && queries[qIdx][0] == i) {
                maxHeap.push(queries[qIdx][1]);
                qIdx++;
            }

            running += diff[i];

            while (running < nums[i]) {
                while (!maxHeap.empty() && maxHeap.top() < i) {
                    maxHeap.pop();
                }

                if (maxHeap.empty()) {
                    return -1;
                }

                int r = maxHeap.top();
                maxHeap.pop();

                diff[i] += 1;
                diff[r + 1] -= 1;
                running += 1;
                usedCount++;
            }
        }

        return (int)queries.size() - usedCount;
    }
};
```

---

## Key Template

```text
sort queries by l
maxHeap = empty (ordered by r)
diff = array of size (n + 1), all 0
running = 0
usedCount = 0
qIdx = 0

for i in 0..n-1:
    while qIdx < queries.size() and queries[qIdx].l == i:
        maxHeap.push(queries[qIdx].r)
        qIdx += 1

    running += diff[i]

    while running < nums[i]:
        while maxHeap not empty and maxHeap.top() < i:
            maxHeap.pop()   # expired, discard

        if maxHeap.empty():
            return -1

        r = maxHeap.pop()
        diff[i] += 1
        diff[r + 1] -= 1
        running += 1
        usedCount += 1

return queries.size() - usedCount
```