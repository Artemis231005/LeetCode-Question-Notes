# LeetCode 3550 — Smallest Index With Digit Sum Equal to Index

## Metadata

* **LeetCode:** 3550
* **Problem:** Smallest Index With Digit Sum Equal to Index
* **Difficulty:** Easy
* **Topics:** Array, Math
* **Pattern:** Direct Scan with Digit Sum Computation
* **Key Technique:** Compute each number's digit sum via arithmetic (`% 10` and `/ 10`) directly, avoiding string conversion overhead, while scanning left to right and returning the first matching index
* **Optimal Complexity:** `O(n * log(maxVal))` Time, `O(1)` Space

---

## Problem Statement

Given an integer array `nums`, return the smallest index `i` such that the digit sum of `nums[i]` equals `i`. Return `-1` if no such index exists.

---

## Approaches

1. **Brute Force — String Conversion for Digit Sum**
2. **Optimal — Arithmetic Digit Sum (No String Conversion)**

---

# Approach 1 — Brute Force / String Conversion for Digit Sum

## Idea

For each index, convert the number at that index to a string, then sum the numeric value of each character in the string to get its digit sum. Compare that sum against the index and return the first match.

## Dry Run

```text
nums = [1, 3, 2]
```

`i = 0`: convert `nums[0]=1` to string `"1"`, sum digits: `1`.

```text
digitSum=1, i=0 → 1 != 0 → no match
```

`i = 1`: convert `nums[1]=3` to string `"3"`, sum digits: `3`.

```text
digitSum=3, i=1 → 3 != 1 → no match
```

`i = 2`: convert `nums[2]=2` to string `"2"`, sum digits: `2`.

```text
digitSum=2, i=2 → 2 != 2 → match! return 2
```

## Algorithm

1. For each index `i` from `0` to `n-1`:

   * Convert `nums[i]` to a string.
   * Sum the numeric value of each character in that string.
   * If the sum equals `i`, return `i`.
2. If no index matches, return `-1`.

## Complexity

* **Time:** `O(n * log(maxVal))`

  * For each of the `n` indices, converting to a string and summing its characters both take time proportional to the number of digits.
* **Space:** `O(log(maxVal))`

  * For the temporary string representation of each number.

## Notes / Tips

* Converting to a string just to sum its digits is unnecessary overhead — the digit sum can be computed directly with arithmetic (`% 10` to peel off the last digit, `/ 10` to remove it), avoiding any string allocation entirely.
* Given this problem's typically small constraints, the string conversion overhead is negligible in practice, but the arithmetic approach (Approach 2) is the more direct and idiomatic way to compute a digit sum regardless of scale.

## Code

```cpp
class Solution {
public:
    int smallestIndex(vector<int>& nums) {
        for (int i = 0; i < nums.size(); i++) {
            string s = to_string(nums[i]);
            int digitSum = 0;

            for (char c : s) {
                digitSum += c - '0';
            }

            if (digitSum == i) {
                return i;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Optimal / Arithmetic Digit Sum (No String Conversion)

## Idea

Compute each number's digit sum directly using arithmetic: repeatedly take the last digit with `% 10`, add it to a running sum, and remove it with `/ 10`, until the number reaches `0`. This avoids ever building a string representation, working purely with integer operations.

## Dry Run

```text
nums = [1, 3, 2]
```

`i = 0`, `nums[0] = 1`:

```text
digit = 1 % 10 = 1, sum = 1
num = 1 / 10 = 0 → stop
digitSum = 1 → 1 != 0 → no match
```

`i = 1`, `nums[1] = 3`:

```text
digit = 3, sum = 3
digitSum = 3 → 3 != 1 → no match
```

`i = 2`, `nums[2] = 2`:

```text
digit = 2, sum = 2
digitSum = 2 → 2 == 2 → match! return 2
```

## Algorithm

1. For each index `i` from `0` to `n-1`:

   * Initialize `num = nums[i]`, `digitSum = 0`.
   * While `num > 0`: `digitSum += num % 10`, `num /= 10`.
   * If `digitSum == i`, return `i`.
2. If no index matches, return `-1`.

## Complexity

* **Time:** `O(n * log(maxVal))`

  * Same asymptotic bound as Approach 1, but with lower constant overhead since no string allocation or character-to-integer conversion is needed.
* **Space:** `O(1)`

  * Only a couple of integer variables per index — no string or extra structures.

## Notes / Tips

* This is the standard way to compute a digit sum in any language — peeling digits off with `% 10` and `/ 10` is faster and more direct than converting to a string first, and generalizes cleanly (e.g. to other bases by swapping `10` for the base).
* A special case worth noting: if `nums[i] == 0`, the `while` loop never executes and `digitSum` stays `0` — this is correct, since the digit sum of `0` is indeed `0`.
* Returning immediately upon finding the first match (rather than scanning further) is what guarantees the *smallest* qualifying index is returned, matching the problem's requirement.

## Code

```cpp
class Solution {
public:
    int smallestIndex(vector<int>& nums) {
        for (int i = 0; i < nums.size(); i++) {
            int num = nums[i];
            int digitSum = 0;

            while (num > 0) {
                digitSum += num % 10;
                num /= 10;
            }

            if (digitSum == i) {
                return i;
            }
        }

        return -1;
    }
};
```

---

## Key Template

```text
for i in 0..n-1:
    num = nums[i]
    digitSum = 0

    while num > 0:
        digitSum += num % 10
        num /= 10

    if digitSum == i:
        return i

return -1
```