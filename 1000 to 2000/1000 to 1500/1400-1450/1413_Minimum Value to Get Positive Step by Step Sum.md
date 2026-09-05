# LeetCode 1413 — Minimum Value to Get Positive Step by Step Sum

## Metadata

* **LeetCode:** 1413
* **Problem:** Minimum Value to Get Positive Step by Step Sum
* **Difficulty:** Easy
* **Topics:** Array, Prefix Sum
* **Pattern:** Prefix Sum
* **Key Technique:** Track the minimum running prefix sum, then derive the required starting value from it directly
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `nums`, find the smallest positive `startValue` such that the running sum (`startValue + nums[0] + nums[1] + ... + nums[i]`) never drops below `1` at any step `i`.

---

## Approaches

1. **Brute Force — Try Increasing Start Values**
2. **Optimal — Minimum Running Prefix Sum**

---

# Approach 1 — Brute Force / Try Increasing Start Values

## Idea

Starting from `startValue = 1`, simulate the entire running sum for the whole array. If it ever dips below `1`, increment `startValue` and try the whole simulation again. Repeat until a value works.

## Dry Run

```text
nums = [-3, 2, -3, 4, 2]
```

Try `startValue = 1`:

```text
1 + (-3) = -2 → fails (below 1)
```

Try `startValue = 2`:

```text
2 + (-3) = -1 → fails
```

Try `startValue = 3`:

```text
3 + (-3) = 0 → fails
```

Try `startValue = 4`:

```text
4 + (-3) = 1 → ok
1 + 2 = 3 → ok
3 + (-3) = 0 → fails
```

Try `startValue = 5`:

```text
5 + (-3) = 2 → ok
2 + 2 = 4 → ok
4 + (-3) = 1 → ok
1 + 4 = 5 → ok
5 + 2 = 7 → ok
```

All steps pass → `startValue = 5`.

## Algorithm

1. Set `startValue = 1`.
2. Simulate the running sum across the whole array with this `startValue`.
3. If any step drops below `1`, increment `startValue` and restart the simulation.
4. Once a full simulation passes, return `startValue`.

## Complexity

* **Time:** `O(n * maxStart)`

  * Each candidate `startValue` triggers a fresh full simulation of length `n`, and `maxStart` can be as large as the total magnitude of negative dips.
* **Space:** `O(1)`

  * Only the running sum and candidate `startValue` are tracked — no extra structures allocated.

## Notes / Tips

* Extremely wasteful — every failed attempt re-simulates the entire array from scratch just to find where it failed.
* The only thing that actually matters is *how far* the running sum dips below `1` at its worst point — no need to search value by value at all.

## Code

```cpp
class Solution {
public:
    int minStartValue(vector<int>& nums) {
        int startValue = 1;

        while (true) {
            int sum = startValue;
            bool ok = true;

            for (int num : nums) {
                sum += num;
                if (sum < 1) {
                    ok = false;
                    break;
                }
            }

            if (ok) {
                return startValue;
            }

            startValue++;
        }
    }
};
```

---

# Approach 2 — Optimal / Minimum Running Prefix Sum

## Idea

The running sum starting from `0` (ignoring `startValue` entirely) will dip to some minimum value at its worst point. To keep the actual running sum (with `startValue` added) at least `1` everywhere, `startValue` just needs to cancel out that worst dip: `startValue = 1 - minPrefixSum`. If the minimum prefix sum is already `>= 1`, no boost is needed and `startValue = 1`.

## Dry Run

```text
nums = [-3, 2, -3, 4, 2]
```

Compute running sum from `0` and track the minimum:

```text
sum = 0
-3 → sum = -3 → min = -3
2  → sum = -1 → min = -3
-3 → sum = -4 → min = -4
4  → sum = 0  → min = -4
2  → sum = 2  → min = -4
```

Minimum prefix sum: `-4`.

```text
startValue = 1 - (-4) = 5
```

Matches the brute-force result.

## Algorithm

1. Initialize `sum = 0` and `minSum = 0`.
2. For each value in `nums`:

   * `sum += num`.
   * `minSum = min(minSum, sum)`.
3. Return `1 - minSum` (this naturally returns `1` when `minSum` is already `>= 0`).

## Complexity

* **Time:** `O(n)`

  * Single pass over the array.
* **Space:** `O(1)`

  * Only two running variables (`sum`, `minSum`) — no extra structures needed.

## Notes / Tips

* `minSum` is initialized to `0`, not the first element — this correctly represents the running sum's value *before* any elements are added (i.e. the empty prefix), matching how the problem defines the starting point.
* The formula `startValue = 1 - minSum` works uniformly whether the array ever dips negative or not — no need to special-case the "always positive" scenario.
* Same "track the running minimum" idea shows up in problems like "Minimum Subarray Sum" or "Buy and Sell Stock" — a running extreme value replaces re-scanning.

## Code

```cpp
class Solution {
public:
    int minStartValue(vector<int>& nums) {
        int sum = 0, minSum = 0;

        for (int num : nums) {
            sum += num;
            minSum = min(minSum, sum);
        }

        return 1 - minSum;
    }
};
```

---

## Key Template

```text
sum = 0
minSum = 0

for num in nums:
    sum += num
    minSum = min(minSum, sum)

return 1 - minSum
```