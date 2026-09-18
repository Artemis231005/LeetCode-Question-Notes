# LeetCode 2772 — Apply Operations to Make All Array Elements Equal to Zero

## Metadata

* **LeetCode:** 2772
* **Problem:** Apply Operations to Make All Array Elements Equal to Zero
* **Difficulty:** Medium
* **Topics:** Array, Greedy, Prefix Sum, Queue, Sliding Window
* **Pattern:** Greedy Left-to-Right + Difference Array (Bulk Range Decrement)
* **Key Technique:** Process indices left to right; whenever the current element still needs decrementing, it must be the start of a size-`k` decrement window (since nothing to its left can help anymore) — apply the whole needed amount at once using a difference array instead of one unit at a time
* **Optimal Complexity:** `O(n)` Time, `O(n)` Auxiliary Space (or `O(k)` with a sliding-window sum instead of a full diff array)

---

## Problem Statement

Given an integer array `nums` and an integer `k`, in one operation you choose a subarray of size exactly `k` and decrease every element in it by `1`. Return `true` if it's possible to make every element of `nums` equal to `0` using any number of such operations.

---

## Approaches

1. **Brute Force — Literal Unit-by-Unit Simulation**
2. **Optimal — Greedy Left-to-Right with Difference Array**

---

# Approach 1 — Brute Force / Literal Unit-by-Unit Simulation

## Idea

Simulate the problem exactly as stated: repeatedly find the leftmost nonzero element, apply a single decrement-by-1 operation to the size-`k` window starting there (since a leftmost nonzero element can never be helped by a window starting to its left, this greedy choice of window position is always necessary), and repeat until either the whole array is zero or a needed window would run off the end of the array.

## Dry Run

```text
nums = [2, 2, 3, 1, 1, 0], k = 3
```

Leftmost nonzero: index `0`. Window `[0,2]`, subtract `1`:

```text
[1, 1, 2, 1, 1, 0]
```

Leftmost nonzero: index `0` again. Window `[0,2]`, subtract `1`:

```text
[0, 0, 1, 1, 1, 0]
```

Leftmost nonzero: index `2`. Window `[2,4]`, subtract `1`:

```text
[0, 0, 0, 0, 0, 0]
```

All zero → return `true`.

## Algorithm

1. While any element of `nums` is nonzero:

   * Find the leftmost index `i` with `nums[i] != 0`.
   * If `i + k > n`, return `false` (can't form a valid window here).
   * Subtract `1` from every element in `nums[i .. i+k-1]`.
2. Return `true`.

## Complexity

* **Time:** `O(maxVal * n)`

  * In the worst case, one unit is decremented per operation, requiring up to `maxVal` (the largest value in `nums`) operations, each scanning/modifying up to `n` elements.
* **Space:** `O(1)`

  * Modifies the array in place, only a few index variables tracked.

## Notes / Tips

* Applying decrements one unit at a time is extremely wasteful — the moment the leftmost nonzero element's required window is determined, the **entire** needed decrement amount (`nums[i]`, not just `1`) can be applied to that window in one shot, since every operation on that window would have started at the same position anyway.
* This literal simulation is mainly useful for confirming the greedy window-selection logic (always pick the leftmost nonzero element as the window start) before optimizing the actual decrement application.

## Code

```cpp
class Solution {
public:
    bool checkArray(vector<int>& nums, int k) {
        int n = nums.size();

        while (true) {
            int i = -1;
            for (int idx = 0; idx < n; idx++) {
                if (nums[idx] != 0) {
                    i = idx;
                    break;
                }
            }

            if (i == -1) {
                return true;
            }

            if (i + k > n) {
                return false;
            }

            for (int j = i; j < i + k; j++) {
                nums[j]--;
            }
        }
    }
};
```

---

# Approach 2 — Optimal / Greedy Left-to-Right with Difference Array

## Idea

Scan left to right, tracking the total decrement already applied to the current index via a difference array (the same range-update technique as LC 1109 and LC 2848). At each index `i`, compute how much more it still needs to be decremented (`nums[i] - alreadyApplied`). If that's negative, it's impossible (over-decremented, meaning an earlier window pushed this index below zero). If positive, that entire remaining amount must be applied as a single bulk operation to the window `[i, i+k-1]` (since, just as in the brute force, nothing to the left of `i` can ever help `i` again) — mark this with the difference array instead of looping over the window.

## Dry Run

```text
nums = [2, 2, 3, 1, 1, 0], k = 3
```

Initialize `diff` array of size `n+1`, `running = 0`.

```text
i=0: running += diff[0] = 0 → applied=0, need = 2-0 = 2
     need > 0 → apply 2 to window [0,2]: diff[0] += 2, diff[3] -= 2
     diff = [2,0,0,-2,0,0,0]

i=1: running += diff[1] = 0 → running stays 2, applied=2, need = 2-2 = 0 → nothing to do

i=2: running += diff[2] = 0 → running stays 2, applied=2, need = 3-2 = 1
     need > 0 → apply 1 to window [2,4]: diff[2] += 1, diff[5] -= 1
     diff = [2,0,1,-2,0,-1,0]

i=3: running += diff[3] = -2 → running = 2-2=0, applied=0... 
```

Wait — running sum must accumulate across all previous `diff` entries properly; re-tracing carefully:

```text
running starts at 0
i=0: running += diff[0]=2 → running=2 → applied=2, need=2-2=0 (already satisfied from the update just made)
```

Actually the update at `i=0` happens *before* reading `applied` for that same index (apply first, then read), so:

```text
i=0: running (before any diff applied) = 0
     need = nums[0] - running = 2 - 0 = 2 → apply: diff[0]+=2, diff[3]-=2
     running += diff[0] → running = 0+2 = 2 (now reflects this index's applied amount)

i=1: running += diff[1] = 0 → running = 2
     need = nums[1] - running = 2 - 2 = 0 → nothing to do

i=2: running += diff[2] = 0 → running = 2
     need = nums[2] - running = 3 - 2 = 1 → apply: diff[2]+=1, diff[5]-=1
     running += 1 (the just-added diff[2]) → running = 3

i=3: running += diff[3] = -2 → running = 3-2 = 1
     need = nums[3] - running = 1 - 1 = 0 → nothing to do

i=4: running += diff[4] = 0 → running = 1
     need = nums[4] - running = 1 - 1 = 0 → nothing to do

i=5: running += diff[5] = -1 → running = 1-1 = 0
     need = nums[5] - running = 0 - 0 = 0 → nothing to do
```

All needs satisfied, no negative encountered, and every window fit within bounds → return `true`.

## Algorithm

1. Create a difference array `diff` of size `n + 1`, all zeros.
2. Initialize `running = 0`.
3. For each index `i` from `0` to `n-1`:

   * `running += diff[i]` (accumulate all bulk decrements applied so far that reach this index).
   * `need = nums[i] - running`.
   * If `need < 0`, return `false` (over-decremented — impossible).
   * If `need > 0`:

     * If `i + k > n`, return `false` (not enough room for a full window here).
     * Apply the bulk decrement: `diff[i] += need`, `diff[i + k] -= need`.
     * `running += need` (reflect this new decrement immediately, since it applies starting at `i`).
4. Return `true`.

## Complexity

* **Time:** `O(n)`

  * A single left-to-right pass, with `O(1)` work per index.
* **Space:** `O(n)`

  * For the difference array (can be reduced to `O(k)` using a sliding-window sum of just the last `k` diff entries instead of a full-length array, since only the most recent `k` bulk operations can still be affecting the current index).

## Notes / Tips

* The greedy choice — always start a needed decrement window at the current leftmost deficient index — is provably optimal and forced, not just convenient: since indices are processed left to right and nothing before the current index can ever be revisited, any decrement needed at `i` *must* come from a window starting exactly at `i` (starting later would miss `i` entirely, and starting earlier already would have been accounted for by previous iterations).
* The `need < 0` check is what catches impossible cases where an earlier bulk decrement (intended to fix an earlier index) accidentally over-decremented a later index below what it actually needed — this can happen when a window's right portion overshoots relative to the current index's own requirement.

## Code

```cpp
class Solution {
public:
    bool checkArray(vector<int>& nums, int k) {
        int n = nums.size();
        vector<long long> diff(n + 1, 0);
        long long running = 0;

        for (int i = 0; i < n; i++) {
            running += diff[i];
            long long need = nums[i] - running;

            if (need < 0) {
                return false;
            }

            if (need > 0) {
                if (i + k > n) {
                    return false;
                }

                diff[i] += need;
                diff[i + k] -= need;
                running += need;
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
running = 0

for i in 0..n-1:
    running += diff[i]
    need = nums[i] - running

    if need < 0:
        return false

    if need > 0:
        if i + k > n:
            return false

        diff[i] += need
        diff[i + k] -= need
        running += need

return true
```