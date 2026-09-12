# LeetCode 16 — 3Sum Closest

## Metadata

* **LeetCode:** 16
* **Problem:** 3Sum Closest
* **Difficulty:** Medium
* **Topics:** Array, Two Pointers, Sorting
* **Pattern:** Sort + Fix One Element + Two Pointers
* **Key Technique:** Sort the array, fix each element as the "first" of the triplet, then use two pointers to search for the pair that brings the triplet sum as close as possible to `target`, tracking the closest sum seen
* **Optimal Complexity:** `O(n²)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an integer array `nums` and an integer `target`, find three integers in `nums` such that their sum is closest to `target`. Return that sum. Assume exactly one solution exists.

---

## Approaches

1. **Brute Force — Check Every Triplet Directly**
2. **Optimal — Sort + Fix One Element + Two Pointers**

---

# Approach 1 — Brute Force / Check Every Triplet Directly

## Idea

Check every possible triplet `(i, j, k)`, compute its sum, and keep track of whichever triplet's sum has the smallest absolute difference from `target`.

## Dry Run

```text
nums = [-1, 2, 1, -4], target = 1
```

Check triplet `(-1, 2, 1)`:

```text
sum = -1+2+1 = 2 → |2-1| = 1 → best so far = 2
```

Check triplet `(-1, 2, -4)`:

```text
sum = -1+2-4 = -3 → |-3-1| = 4 → worse, keep best = 2
```

Check triplet `(-1, 1, -4)`:

```text
sum = -1+1-4 = -4 → |-4-1| = 5 → worse, keep best = 2
```

Check triplet `(2, 1, -4)`:

```text
sum = 2+1-4 = -1 → |-1-1| = 2 → worse than 1, keep best = 2
```

Final closest sum: `2`.

## Algorithm

1. Initialize `closestSum` to the sum of the first three elements (or `+infinity` as a placeholder).
2. For each `i` from `0` to `n-3`:
3. For each `j` from `i+1` to `n-2`:
4. For each `k` from `j+1` to `n-1`:

   * Compute `sum = nums[i] + nums[j] + nums[k]`.
   * If `abs(sum - target) < abs(closestSum - target)`, update `closestSum = sum`.
5. Return `closestSum`.

## Complexity

* **Time:** `O(n³)`

  * Three nested loops check every possible triplet directly.
* **Space:** `O(1)`

  * Only a running closest-sum tracker — no extra structures allocated.

## Notes / Tips

* Straightforward but doesn't exploit any structure in the data — sorting first (as in Approach 2) allows a much faster search per fixed element using two pointers instead of a full nested scan.
* Since this problem guarantees a unique answer and doesn't ask for all triplets, there's no need for the duplicate-handling logic seen in LC 15 (3Sum) — this approach and the optimal one both just track a single best value.

## Code

```cpp
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        int n = nums.size();
        int closestSum = nums[0] + nums[1] + nums[2];

        for (int i = 0; i < n - 2; i++) {
            for (int j = i + 1; j < n - 1; j++) {
                for (int k = j + 1; k < n; k++) {
                    int sum = nums[i] + nums[j] + nums[k];

                    if (abs(sum - target) < abs(closestSum - target)) {
                        closestSum = sum;
                    }
                }
            }
        }

        return closestSum;
    }
};
```

---

# Approach 2 — Optimal / Sort + Fix One Element + Two Pointers

## Idea

Sort the array first. Fix each element `nums[i]`, then use two pointers (`left` starting right after `i`, `right` at the end) to search the remaining sorted subarray for the pair that brings the total sum closest to `target`. Move `left` right when the sum is too small, or `right` left when the sum is too large — this works because the array is sorted, so adjusting in the "wrong" direction can only make the sum move further from `target` in a predictable way.

## Dry Run

```text
nums = [-4, -1, 1, 2]  (sorted), target = 1
```

`i = 0` (value `-4`):

```text
left=1 (-1), right=3 (2): sum = -4-1+2 = -3 → |-3-1|=4 → closest so far = -3
   sum < target → move left right
left=2 (1), right=3 (2): sum = -4+1+2 = -1 → |-1-1|=2 → better, closest = -1
   sum < target → move left right
left=3, right=3: left == right, stop
```

`i = 1` (value `-1`):

```text
left=2 (1), right=3 (2): sum = -1+1+2 = 2 → |2-1|=1 → better, closest = 2
   sum > target → move right left
left=2, right=2: left == right, stop
```

`i = 2`: not enough elements left for `left < right`, loop ends.

Final closest sum: `2`.

## Algorithm

1. Sort `nums` in ascending order.
2. Initialize `closestSum` to the sum of the first three elements.
3. For each `i` from `0` to `n-3`:

   * Set `left = i + 1`, `right = n - 1`.
   * While `left < right`:

     * `sum = nums[i] + nums[left] + nums[right]`.
     * If `abs(sum - target) < abs(closestSum - target)`, update `closestSum = sum`.
     * If `sum == target`, return `sum` immediately (can't get any closer than exact).
     * If `sum < target`, increment `left`.
     * Else, decrement `right`.
4. Return `closestSum`.

## Complexity

* **Time:** `O(n²)`

  * Sorting is `O(n log n)`; the outer loop runs `O(n)` times, and each iteration's two-pointer scan is `O(n)`.
* **Space:** `O(1)` auxiliary (beyond the sort's own space)

  * Only a running closest-sum tracker and two pointers — no extra structures.

## Notes / Tips

* Unlike LC 15 (3Sum), there's no need to skip duplicate values here — since the goal is a single closest sum (not a list of unique triplets), revisiting an equal value can never produce a wrong answer, just redundant extra comparisons.
* The early return when `sum == target` is a valid optimization — once the exact target is hit, no other triplet can possibly be closer.
* This is the same sort + fix-one-element + two-pointers skeleton as LC 15, just swapping "collect all zero-sum triplets" for "track the single sum closest to a target" — recognizing this shared structure makes both problems approachable with the same core technique.

## Code

```cpp
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target) {
        sort(nums.begin(), nums.end());
        int n = nums.size();
        int closestSum = nums[0] + nums[1] + nums[2];

        for (int i = 0; i < n - 2; i++) {
            int left = i + 1, right = n - 1;

            while (left < right) {
                int sum = nums[i] + nums[left] + nums[right];

                if (abs(sum - target) < abs(closestSum - target)) {
                    closestSum = sum;
                }

                if (sum == target) {
                    return sum;
                } else if (sum < target) {
                    left++;
                } else {
                    right--;
                }
            }
        }

        return closestSum;
    }
};
```

---

## Key Template

```text
sort(nums)
closestSum = nums[0] + nums[1] + nums[2]

for i in 0..n-3:
    left = i + 1
    right = n - 1

    while left < right:
        sum = nums[i] + nums[left] + nums[right]

        if abs(sum - target) < abs(closestSum - target):
            closestSum = sum

        if sum == target:
            return sum
        elif sum < target:
            left += 1
        else:
            right -= 1

return closestSum
```