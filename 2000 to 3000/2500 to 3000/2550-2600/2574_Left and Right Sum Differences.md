# LeetCode 2574 — Left and Right Sum Differences

## Metadata

* **LeetCode:** 2574
* **Problem:** Left and Right Sum Differences
* **Difficulty:** Easy
* **Topics:** Array, Prefix Sum
* **Pattern:** Prefix Sum
* **Key Technique:** Track total sum and a running left-sum so the right-sum at any index is derived, not recomputed
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `nums`, build an array `answer` of the same length where `answer[i] = |leftSum[i] - rightSum[i]|`, with `leftSum[i]` being the sum of all elements before index `i` and `rightSum[i]` the sum of all elements after index `i` (both `0` if none exist).

---

## Approaches

1. **Brute Force — Sum Left and Right for Every Index**
2. **Optimal — Total Sum + Running Left Sum**

---

# Approach 1 — Brute Force / Sum Left and Right for Every Index

## Idea

For each index `i`, sum every element to its left and every element to its right independently, then take the absolute difference.

## Dry Run

```text
nums = [10, 4, 8, 3]
```

`i = 2` (value `8`):

```text
left sum = 10 + 4 = 14
right sum = 3
```

```text
|14 - 3| = 11
```

## Algorithm

1. Create a `result` array of the same length as `nums`.
2. For each index `i`:

   * Compute `leftSum` by summing `nums[0..i-1]`.
   * Compute `rightSum` by summing `nums[i+1..n-1]`.
   * `result[i] = abs(leftSum - rightSum)`.
3. Return `result`.

## Complexity

* **Time:** `O(n²)`

  * Each of the `n` indices triggers two fresh scans (left and right) that can each cover up to `n` elements.
* **Space:** `O(1)`

  * No extra structure beyond the required output array — no additional data structures allocated.

## Notes / Tips

* Recomputing both sums from scratch at every index throws away almost all the work done for the previous index.
* Same redundant pattern seen in LC 724 and LC 1991 — this problem is effectively those two combined into one output array instead of a single boolean/index answer.

## Code

```cpp
class Solution {
public:
    vector<int> leftRigthDifference(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n);

        for (int i = 0; i < n; i++) {
            int leftSum = 0, rightSum = 0;

            for (int j = 0; j < i; j++) {
                leftSum += nums[j];
            }
            for (int j = i + 1; j < n; j++) {
                rightSum += nums[j];
            }

            result[i] = abs(leftSum - rightSum);
        }

        return result;
    }
};
```

---

# Approach 2 — Optimal / Total Sum + Running Left Sum

## Idea

Compute the total sum of the array once. Then scan left to right, keeping a running `leftSum`. At each index, the right sum is derived directly as `total - leftSum - nums[i]` — no separate right-side scan needed.

## Dry Run

```text
nums = [10, 4, 8, 3]
total = 10+4+8+3 = 25
```

Process:

```text
i=0: leftSum=0, rightSum = 25 - 0 - 10 = 15 → result[0] = |0-15| = 15
     leftSum += 10 → leftSum = 10

i=1: rightSum = 25 - 10 - 4 = 11 → result[1] = |10-11| = 1
     leftSum += 4 → leftSum = 14

i=2: rightSum = 25 - 14 - 8 = 3 → result[2] = |14-3| = 11
     leftSum += 8 → leftSum = 22

i=3: rightSum = 25 - 22 - 3 = 0 → result[3] = |22-0| = 22
```

Result:

```text
[15, 1, 11, 22]
```

## Algorithm

1. Compute `total = sum(nums)`.
2. Initialize `leftSum = 0`.
3. For each index `i` from `0` to `n-1`:

   * Compute `rightSum = total - leftSum - nums[i]`.
   * `result[i] = abs(leftSum - rightSum)`.
   * `leftSum += nums[i]`.
4. Return `result`.

## Complexity

* **Time:** `O(n)`

  * A single pass computes `total`, and a second single pass computes every `result[i]` — both linear, so overall linear.
* **Space:** `O(1)`

  * Only `total` and `leftSum` are tracked beyond the required output array — no extra structures needed.

## Notes / Tips

* Same underlying identity as LC 724 (Find Pivot Index) and LC 1991 (Find the Middle Index) — this problem just asks for the difference at **every** index instead of searching for where it hits `0`.
* Common mistake: updating `leftSum` before computing `rightSum` for the current index — `leftSum` must represent everything strictly before `i`, not including `nums[i]`.

## Code

```cpp
class Solution {
public:
    vector<int> leftRigthDifference(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n);

        int total = 0;
        for (int num : nums) {
            total += num;
        }

        int leftSum = 0;
        for (int i = 0; i < n; i++) {
            int rightSum = total - leftSum - nums[i];
            result[i] = abs(leftSum - rightSum);
            leftSum += nums[i];
        }

        return result;
    }
};
```

---

## Key Template

```text
total = sum(nums)
leftSum = 0
result = []

for i in 0..n-1:
    rightSum = total - leftSum - nums[i]
    result[i] = abs(leftSum - rightSum)
    leftSum += nums[i]

return result
```