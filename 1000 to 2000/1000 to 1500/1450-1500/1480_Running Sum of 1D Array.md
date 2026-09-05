# LeetCode 1480 — Running Sum of 1d Array

## Metadata

* **LeetCode:** 1480
* **Problem:** Running Sum of 1d Array
* **Difficulty:** Easy
* **Topics:** Array, Prefix Sum
* **Pattern:** Prefix Sum
* **Key Technique:** Accumulate a running total in place as you scan left to right
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `nums`, return its running sum, where `runningSum[i] = nums[0] + nums[1] + ... + nums[i]`.

---

## Approaches

1. **Brute Force — Recompute Each Prefix Sum from Scratch**
2. **Optimal — Single-Pass Accumulation**

---

# Approach 1 — Brute Force / Recompute Each Prefix Sum from Scratch

## Idea

For every index `i`, sum up all elements from `0` to `i` independently, without reusing any work done for earlier indices.

## Dry Run

```text
nums = [1, 2, 3, 4]
```

`i = 0`:

```text
sum(nums[0..0]) = 1
```

`i = 1`:

```text
sum(nums[0..1]) = 1 + 2 = 3
```

`i = 2`:

```text
sum(nums[0..2]) = 1 + 2 + 3 = 6
```

`i = 3`:

```text
sum(nums[0..3]) = 1 + 2 + 3 + 4 = 10
```

Result:

```text
[1, 3, 6, 10]
```

## Algorithm

1. Create a `result` array of the same length as `nums`.
2. For each index `i`:

   * Loop `j` from `0` to `i`, summing `nums[j]`.
   * Store the sum in `result[i]`.
3. Return `result`.

## Complexity

* **Time:** `O(n²)`

  * Each of the `n` indices triggers a fresh inner loop that can scan up to `n` elements.
* **Space:** `O(1)`

  * No extra structure beyond the required output array — no additional data structures allocated.

## Notes / Tips

* Every prefix sum recomputed here fully contains the previous prefix sum's work — recomputing it every time throws away information that's trivial to carry forward.
* Fine for very small arrays, but scales poorly.

## Code

```cpp
class Solution {
public:
    vector<int> runningSum(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n);

        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = 0; j <= i; j++) {
                sum += nums[j];
            }
            result[i] = sum;
        }

        return result;
    }
};
```

---

# Approach 2 — Optimal / Single-Pass Accumulation

## Idea

Keep a running total as you scan left to right. Each new element just gets added to the total from the previous index — no need to re-sum anything already counted.

## Dry Run

```text
nums = [1, 2, 3, 4]
```

Process:

```text
i=0: total = 0 + 1 = 1 → result[0] = 1
i=1: total = 1 + 2 = 3 → result[1] = 3
i=2: total = 3 + 3 = 6 → result[2] = 6
i=3: total = 6 + 4 = 10 → result[3] = 10
```

Result:

```text
[1, 3, 6, 10]
```

## Algorithm

1. Initialize `total = 0`.
2. For each index `i` from `0` to `n-1`:

   * `total += nums[i]`.
   * `nums[i] = total` (or write into a separate result array if the input must stay unmodified).
3. Return the updated array.

## Complexity

* **Time:** `O(n)`

  * Single pass, each element processed exactly once.
* **Space:** `O(1)`

  * Can be done in place on `nums` itself, using only one running total variable — no extra array needed.

## Notes / Tips

* Can be solved in place by overwriting `nums` directly, since each `result[i]` only ever depends on `result[i-1]` and `nums[i]`.
* This is the same accumulate-as-you-go idea behind prefix sum arrays (see LC 303) — the only difference here is the running sum itself **is** the answer, not just an intermediate structure.
* Common mistake: resetting the running total inside the loop instead of carrying it forward — defeats the entire point of the optimization.

## Code

```cpp
class Solution {
public:
    vector<int> runningSum(vector<int>& nums) {
        int total = 0;

        for (int i = 0; i < nums.size(); i++) {
            total += nums[i];
            nums[i] = total;
        }

        return nums;
    }
};
```

---

## Key Template

```text
total = 0

for i in 0..n-1:
    total += nums[i]
    nums[i] = total   // or result[i] = total

return nums   // or result
```