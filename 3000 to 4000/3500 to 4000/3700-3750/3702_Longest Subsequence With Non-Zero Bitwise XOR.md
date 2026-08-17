# LeetCode 3702 — Longest Subsequence With Non-Zero Bitwise XOR

## Metadata

* **LeetCode:** 3702
* **Problem:** Longest Subsequence With Non-Zero Bitwise XOR
* **Difficulty:** Easy
* **Topics:** Array, Bit Manipulation, XOR, Greedy
* **Pattern:** XOR Properties
* **Key Technique:** Total XOR + Remove One Element
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Problem

Given an integer array `nums`, find the length of the **longest subsequence** whose bitwise XOR is non-zero.

A subsequence can be obtained by deleting zero or more elements while maintaining the relative order of the remaining elements.

Return the maximum possible length.

---

# Approach 1 — Brute Force

## Idea

A subsequence can contain any subset of the elements while preserving their order.

Therefore, enumerate every possible subsequence, calculate its XOR, and keep the maximum length whose XOR is non-zero.

For an array of size `n`, there are:

```text
2^n
```

possible subsequences.

## Dry Run

```text
nums = [1, 2, 3]

Subsequences:

[]       → XOR = 0
[1]      → XOR = 1    ✓ length 1
[2]      → XOR = 2    ✓ length 1
[3]      → XOR = 3    ✓ length 1
[1,2]    → XOR = 3    ✓ length 2
[1,3]    → XOR = 2    ✓ length 2
[2,3]    → XOR = 1    ✓ length 2
[1,2,3]  → XOR = 0    ✗

Answer = 2
```

## Notes / Tips

* This directly checks every possible subsequence.
* The number of subsequences grows exponentially.
* It is useful only as a conceptual starting point.

## Complexity

* **Time:** `O(2^n × n)`
* **Space:** `O(n)` for recursion/subsequence construction

## Code

```cpp
class Solution {
public:
    int ans = 0;

    void solve(vector<int>& nums, int i, int len, int xr) {
        if (i == nums.size()) {
            if (xr != 0)
                ans = max(ans, len);
            return;
        }

        // Take nums[i]
        solve(nums, i + 1, len + 1, xr ^ nums[i]);

        // Skip nums[i]
        solve(nums, i + 1, len, xr);
    }

    int longestSubsequence(vector<int>& nums) {
        solve(nums, 0, 0, 0);
        return ans;
    }
};
```

---

# Approach 2 — Total XOR Observation

## Idea

First calculate the XOR of the entire array:

```text
X = nums[0] ^ nums[1] ^ ... ^ nums[n-1]
```

There are two main cases.

### Case 1 — Total XOR is Non-Zero

If:

```text
X != 0
```

then the entire array itself is a valid subsequence.

Since no subsequence can contain more than `n` elements:

```text
answer = n
```

### Case 2 — Total XOR is Zero

If:

```text
X == 0
```

the entire array is invalid.

Now remove exactly one element `x`.

The XOR of the remaining elements is:

```text
X ^ x
```

Since `X = 0`:

```text
0 ^ x = x
```

Therefore, if we remove any **non-zero** element, the remaining XOR becomes non-zero.

So:

```text
answer = n - 1
```

provided at least one non-zero element exists.

If every element is zero, removing elements can never produce a non-zero XOR.

Therefore:

```text
answer = 0
```

## Dry Run

### Example 1

```text
nums = [1, 2, 4]

Total XOR:
1 ^ 2 ^ 4 = 7
```

Since:

```text
7 != 0
```

the entire array is valid.

```text
answer = 3
```

### Example 2

```text
nums = [1, 2, 3]

Total XOR:
1 ^ 2 ^ 3 = 0
```

The entire array is invalid.

Remove `1`:

```text
2 ^ 3 = 1
```

which is non-zero.

Therefore:

```text
answer = 2
```

### Example 3

```text
nums = [0, 0, 0]
```

Total XOR:

```text
0 ^ 0 ^ 0 = 0
```

Removing any element still leaves:

```text
0 ^ 0 = 0
```

No valid subsequence exists.

```text
answer = 0
```

## Notes / Tips

* Important XOR properties:

```text
x ^ x = 0
x ^ 0 = x
0 ^ x = x
```

* If the total XOR is non-zero, take the entire array.
* If the total XOR is zero and at least one element is non-zero, remove exactly one non-zero element.
* If all elements are zero, no non-zero XOR is possible.
* The specific non-zero element removed does not matter.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Code

```cpp
class Solution {
public:
    int longestSubsequence(vector<int>& nums) {
        int xr = 0;
        bool hasNonZero = false;

        for (int x : nums) {
            xr ^= x;

            if (x != 0)
                hasNonZero = true;
        }

        if (xr != 0)
            return nums.size();

        if (hasNonZero)
            return nums.size() - 1;

        return 0;
    }
};
```

---

# Approach 3 — Simplified Case Analysis

## Idea

The entire problem can be reduced to two pieces of information:

1. Is the XOR of the entire array non-zero?
2. Does the array contain at least one non-zero element?

This gives three possible outcomes:

```text
Total XOR != 0
        ↓
answer = n
```

```text
Total XOR == 0
        ↓
Contains non-zero element?
      /       \
    Yes        No
     ↓          ↓
answer = n-1  answer = 0
```

## Dry Run

```text
nums = [5, 1, 4, 0]
```

Calculate:

```text
5 ^ 1 ^ 4 ^ 0 = 0
```

The total XOR is zero, but the array contains non-zero elements.

Remove `5`:

```text
1 ^ 4 ^ 0 = 5
```

The resulting XOR is non-zero.

Therefore:

```text
answer = 4 - 1 = 3
```

## Why Is Removing One Element Enough?

Suppose:

```text
total XOR = 0
```

and choose any non-zero element `x`.

The XOR of all elements except `x` is:

```text
remaining XOR
= total XOR ^ x
= 0 ^ x
= x
```

Since:

```text
x != 0
```

the remaining XOR is guaranteed to be non-zero.

Therefore, whenever a non-zero element exists, a valid subsequence of length `n - 1` always exists.

There is no need to remove two or more elements.

## Notes / Tips

* The ordering of elements does not affect XOR.
* A subsequence of length `n - 1` is obtained simply by deleting one element.
* When total XOR is zero, remove **one non-zero element**.
* The all-zero array is the only case where the answer is `0`.
* The final solution only needs one pass to calculate the XOR and determine whether a non-zero element exists.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Code

```cpp
class Solution {
public:
    int longestSubsequence(vector<int>& nums) {
        int xr = 0;

        for (int x : nums) {
            xr ^= x;
        }

        if (xr != 0)
            return nums.size();

        for (int x : nums) {
            if (x != 0)
                return nums.size() - 1;
        }

        return 0;
    }
};
```

---

# Key Pattern

```text
Calculate total XOR
        ↓
Is total XOR non-zero?
       /        \
     Yes         No
      ↓           ↓
   answer = n   Any non-zero?
                /        \
              Yes         No
               ↓           ↓
          answer = n-1   answer = 0
```

## Core XOR Property

```text
If total XOR = 0:

Remove x

Remaining XOR
= 0 ^ x
= x

Therefore, if x != 0,
the remaining XOR is non-zero.
```
