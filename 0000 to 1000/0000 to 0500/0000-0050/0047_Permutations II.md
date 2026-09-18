# LeetCode 47 — Permutations II

## Metadata

* **LeetCode:** 47
* **Problem:** Permutations II
* **Difficulty:** Medium
* **Topics:** Array, Backtracking
* **Pattern:** Backtracking with Duplicate Pruning
* **Key Technique:** Sort the array first, then during backtracking skip a duplicate value at the same recursion depth unless its identical predecessor has already been used — this prevents generating the same permutation twice without needing a hash set
* **Optimal Complexity:** `O(n * n!)` Time, `O(n)` Auxiliary Space (excluding the output)

---

## Problem Statement

Given an array `nums` that may contain duplicate values, return all possible unique permutations in any order.

---

## Approaches

1. **Brute Force — Generate All Permutations, Deduplicate with a Set**
2. **Optimal — Backtracking with Sorted-Array Duplicate Skipping**

---

# Approach 1 — Brute Force / Generate All Permutations, Deduplicate with a Set

## Idea

Treat every element as distinct by its index (ignoring that some values repeat), generate every possible permutation using standard backtracking (swap-based or used-array-based), and insert each resulting permutation into a hash set to automatically discard duplicates that arise from swapping equal values into the same positions.

## Dry Run

```text
nums = [1, 1, 2]
```

Generate all permutations treating each `1` as distinct (by position):

```text
[1,1,2] (from original order)
[1,1,2] (swapping the two 1's — same values, different indices)
[1,2,1]
[1,2,1]
[2,1,1]
[2,1,1]
```

Insert each into a set:

```text
{[1,1,2], [1,2,1], [2,1,1]}
```

Final unique permutations: `[1,1,2], [1,2,1], [2,1,1]`.

## Algorithm

1. Initialize an empty hash set of permutations (e.g. using vectors as keys).
2. Use standard backtracking (swap each position with every later position, recurse, then swap back) to generate every permutation of `nums`, ignoring value duplication.
3. Whenever a complete permutation is formed, insert it into the set.
4. Return the set's contents as a list.

## Complexity

* **Time:** `O(n * n!)`

  * All `n!` permutations are generated (even duplicate ones), each taking `O(n)` to build/copy, plus hashing overhead for deduplication.
* **Space:** `O(n * n!)`

  * For the hash set, which in the worst case (no duplicates) still needs to hold all `n!` permutations, plus wasted work on discarded duplicates when duplicates do exist.

## Notes / Tips

* Generating every permutation and then discarding duplicates afterward is wasteful. With many repeated values, a large fraction of the `n!` generated permutations can be exact duplicates that get thrown away.
* Sorting the array first and pruning duplicate branches **during** generation (Approach 2) avoids ever constructing the duplicate permutations in the first place, rather than constructing and then filtering them.

## Code

```cpp
class Solution {
public:
    void generateAll(vector<int>& nums, int start, set<vector<int>>& results) {
        if (start == nums.size()) {
            results.insert(nums);
            return;
        }

        for (int i = start; i < nums.size(); i++) {
            swap(nums[start], nums[i]);
            generateAll(nums, start + 1, results);
            swap(nums[start], nums[i]);
        }
    }

    vector<vector<int>> permuteUnique(vector<int>& nums) {
        set<vector<int>> results;
        generateAll(nums, 0, results);

        return vector<vector<int>>(results.begin(), results.end());
    }
};
```

---

# Approach 2 — Optimal / Backtracking with Sorted-Array Duplicate Skipping

## Idea

Sort `nums` first so that equal values sit next to each other. Build permutations by choosing one unused element at a time. At each recursive step, skip a value if it's the same as the previous value in the sorted array **and** that previous occurrence hasn't been used yet — this specific condition ensures duplicate values are only ever placed in a fixed relative order relative to each other, which eliminates duplicate permutations without needing to compare or hash full permutations.

## Dry Run

```text
nums = [1, 1, 2]  (already sorted)
```

Start backtracking with `used = [false, false, false]`, `path = []`.

Position 0: try index `0` (value `1`):

```text
used[0]=false, nums[0] has no identical predecessor (index -1 doesn't exist) → allowed
choose it → path=[1], used=[true,false,false]
```

Position 1 (inside path=[1]): try index `1` (value `1`):

```text
nums[1]==nums[0]==1, and used[0]=true (already used) → allowed (predecessor was used, so this is a valid "next" occurrence)
choose it → path=[1,1], used=[true,true,false]
```

Position 2: try index `2` (value `2`):

```text
choose it → path=[1,1,2] → complete permutation → record [1,1,2]
backtrack: used=[true,true,false], path=[1,1]
```

Back at position 1: try index `2` (value `2`) instead of index `1`:

```text
choose it → path=[1,2], used=[true,false,true]
```

Position 2: try index `1` (value `1`):

```text
nums[1]==nums[0]==1, used[0]=true → allowed
choose it → path=[1,2,1] → complete permutation → record [1,2,1]
backtrack
```

Back at position 0: try index `1` (value `1`):

```text
nums[1]==nums[0]==1, but used[0]=false (not used) → SKIP (would create a duplicate branch)
```

Try index `2` (value `2`) at position 0:

```text
choose it → path=[2], used=[false,false,true]
```

Continue similarly to eventually record `[2,1,1]`.

Final unique permutations: `[1,1,2], [1,2,1], [2,1,1]` — no duplicates generated, no deduplication step needed.

## Algorithm

1. Sort `nums` in ascending order.
2. Initialize a `used` boolean array of size `n`, all `false`, and an empty `path`.
3. Define a recursive `backtrack()`:

   * If `path.size() == n`, record a copy of `path` as a valid permutation and return.
   * For each index `i` from `0` to `n-1`:

     * If `used[i]`, skip (already placed in this path).
     * If `i > 0` and `nums[i] == nums[i-1]` and `!used[i-1]`, skip (duplicate-avoidance rule).
     * Mark `used[i] = true`, append `nums[i]` to `path`, recurse.
     * Backtrack: remove the last element from `path`, set `used[i] = false`.
4. Call `backtrack()` and return all recorded permutations.

## Complexity

* **Time:** `O(n * n!)`

  * In the worst case (all distinct elements), this still explores all `n!` permutations, each taking `O(n)` to build; with duplicates present, many branches are pruned early, making this significantly faster in practice than the brute force despite sharing the same worst-case bound.
* **Space:** `O(n)` auxiliary (excluding the output)

  * For the `used` array and the recursion stack/`path`, both bounded by `n`.

## Notes / Tips

* The duplicate-skipping condition — skip `nums[i]` if it equals `nums[i-1]` **and** `nums[i-1]` hasn't been used yet — is subtle but essential: it enforces that among any group of equal values, they're always placed in the same relative left-to-right order across all generated permutations, which is exactly what prevents generating the same permutation multiple times via different index choices.
* Sorting first is a prerequisite for the skip condition to work at all, without adjacent equal values, `nums[i] == nums[i-1]` can't reliably detect duplicates.

## Code

```cpp
class Solution {
public:
    vector<vector<int>> results;
    vector<int> path;

    void backtrack(vector<int>& nums, vector<bool>& used) {
        if (path.size() == nums.size()) {
            results.push_back(path);
            return;
        }

        for (int i = 0; i < nums.size(); i++) {
            if (used[i]) continue;
            if (i > 0 && nums[i] == nums[i - 1] && !used[i - 1]) continue;

            used[i] = true;
            path.push_back(nums[i]);

            backtrack(nums, used);

            path.pop_back();
            used[i] = false;
        }
    }

    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<bool> used(nums.size(), false);

        backtrack(nums, used);

        return results;
    }
};
```

---

## Key Template

```text
sort(nums)
used = array of size n, all false
path = []
results = []

function backtrack():
    if path.size() == n:
        results.append(copy of path)
        return

    for i in 0..n-1:
        if used[i]: continue
        if i > 0 and nums[i] == nums[i-1] and not used[i-1]: continue

        used[i] = true
        path.append(nums[i])

        backtrack()

        path.pop()
        used[i] = false

backtrack()
return results
```