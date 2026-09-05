# LeetCode 724 — Find Pivot Index

## Metadata

* **LeetCode:** 724
* **Problem:** Find Pivot Index
* **Difficulty:** Easy
* **Topics:** Array, Prefix Sum
* **Pattern:** Prefix Sum
* **Key Technique:** Track total sum and a running left-sum so the right-sum at any index is derived, not recomputed
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `nums`, find the leftmost "pivot index" where the sum of all elements to its left equals the sum of all elements to its right. Return `-1` if no such index exists.

---

## Approaches

1. **Brute Force — Sum Left and Right for Every Index**
2. **Optimal — Total Sum + Running Left Sum**

---

# Approach 1 — Brute Force / Sum Left and Right for Every Index

## Idea

For each candidate index `i`, sum every element to its left and every element to its right independently, then compare the two sums.

## Dry Run

```text
nums = [1, 7, 3, 6, 5, 6]
```

`i = 3` (value `6`):

```text
left sum = 1 + 7 + 3 = 11
right sum = 5 + 6 = 11
```

`11 == 11` → pivot found at index `3`.

## Algorithm

1. For each index `i` from `0` to `n-1`:

   * Compute `leftSum` by summing `nums[0..i-1]`.
   * Compute `rightSum` by summing `nums[i+1..n-1]`.
   * If `leftSum == rightSum`, return `i`.
2. If no index qualifies, return `-1`.

## Complexity

* **Time:** `O(n²)`

  * Each of the `n` indices triggers two fresh scans (left and right) that can each cover up to `n` elements.
* **Space:** `O(1)`

  * Only a couple of running total variables per iteration — no extra structures allocated.

## Notes / Tips

* Recomputing both sums from scratch at every index throws away almost all the work done for the previous index — left and right sums shift predictably as `i` moves by one.
* Correct but unnecessary once the relationship `rightSum = total - leftSum - nums[i]` is recognized.

## Code

```cpp
class Solution {
public:
    int pivotIndex(vector<int>& nums) {
        int n = nums.size();

        for (int i = 0; i < n; i++) {
            int leftSum = 0, rightSum = 0;

            for (int j = 0; j < i; j++) {
                leftSum += nums[j];
            }
            for (int j = i + 1; j < n; j++) {
                rightSum += nums[j];
            }

            if (leftSum == rightSum) {
                return i;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Optimal / Total Sum + Running Left Sum

## Idea

Compute the total sum of the array once. Then scan left to right, keeping a running `leftSum`. At each index, the right sum can be derived directly as `total - leftSum - nums[i]` — no separate right-side scan needed.

## Dry Run

```text
nums = [1, 7, 3, 6, 5, 6]
total = 1+7+3+6+5+6 = 28
```

Process:

```text
i=0: leftSum=0, rightSum = 28 - 0 - 1 = 27 → 0 != 27
     leftSum += 1 → leftSum = 1

i=1: rightSum = 28 - 1 - 7 = 20 → 1 != 20
     leftSum += 7 → leftSum = 8

i=2: rightSum = 28 - 8 - 3 = 17 → 8 != 17
     leftSum += 3 → leftSum = 11

i=3: rightSum = 28 - 11 - 6 = 11 → 11 == 11 ✓
```

Pivot found at index `3`.

## Algorithm

1. Compute `total = sum(nums)`.
2. Initialize `leftSum = 0`.
3. For each index `i` from `0` to `n-1`:

   * Compute `rightSum = total - leftSum - nums[i]`.
   * If `leftSum == rightSum`, return `i`.
   * Otherwise, `leftSum += nums[i]`.
4. If no index qualifies, return `-1`.

## Complexity

* **Time:** `O(n)`

  * One pass to compute `total`, one pass to check each index — both linear, so overall linear.
* **Space:** `O(1)`

  * Only `total` and `leftSum` are tracked — no arrays or extra structures.

## Notes / Tips

* The identity `rightSum = total - leftSum - nums[i]` is what removes the need for a second scan entirely — this is the same prefix-sum idea as LC 303 and LC 1480, just applied to avoid explicitly storing a prefix array.
* Common mistake: updating `leftSum` before comparing at the current index — `leftSum` should represent everything strictly before `i`, not including `nums[i]`.
* Works correctly even with negative numbers, since it never assumes sums are monotonically increasing.

## Code

```cpp
class Solution {
public:
    int pivotIndex(vector<int>& nums) {
        int total = 0;
        for (int num : nums) {
            total += num;
        }

        int leftSum = 0;
        for (int i = 0; i < nums.size(); i++) {
            int rightSum = total - leftSum - nums[i];
            if (leftSum == rightSum) {
                return i;
            }
            leftSum += nums[i];
        }

        return -1;
    }
};
```

---

## Key Template

```text
total = sum(nums)
leftSum = 0

for i in 0..n-1:
    rightSum = total - leftSum - nums[i]
    if leftSum == rightSum:
        return i
    leftSum += nums[i]

return -1
```