# LeetCode 303 — Range Sum Query - Immutable

## Metadata

* **LeetCode:** 303
* **Problem:** Range Sum Query - Immutable
* **Difficulty:** Easy
* **Topics:** Array, Design, Prefix Sum
* **Pattern:** Prefix Sum
* **Key Technique:** Precompute cumulative sums once so any range sum becomes a single subtraction
* **Optimal Complexity:** `O(1)` Time per query, `O(n)` Space

---

## Problem Statement

Given an integer array `nums`, design a class `NumArray` that answers multiple `sumRange(left, right)` queries, each returning the sum of `nums[left..right]` (inclusive). The array itself never changes between queries.

---

## Approaches

1. **Brute Force — Sum on Every Query**
2. **Optimal — Prefix Sum Array**

---

# Approach 1 — Brute Force / Sum on Every Query

## Idea

Store the array as-is. For every `sumRange` call, loop from `left` to `right` and add up the values directly.

## Dry Run

```text
nums = [-2, 0, 3, -5, 2, -1]
```

`sumRange(0, 2)`:

```text
-2 + 0 + 3 = 1
```

`sumRange(2, 5)`:

```text
3 + (-5) + 2 + (-1) = -1
```

Each call rescans its own range from scratch — no reuse between calls.

## Algorithm

1. Constructor: store `nums` as-is.
2. `sumRange(left, right)`:

   * Initialize `sum = 0`.
   * Loop `i` from `left` to `right`, adding `nums[i]` to `sum`.
   * Return `sum`.

## Complexity

* **Time:** `O(1)` for the constructor, `O(n)` per `sumRange` call

  * Each query rescans up to the entire array with no precomputed help.
* **Space:** `O(1)`

  * No extra structure beyond storing the input array reference.

## Notes / Tips

* Fine for a handful of queries, but degrades badly if `sumRange` is called many times — total cost becomes `O(n * q)` for `q` queries.
* The array never changes ("immutable"), which is exactly the signal that repeated work can be precomputed once instead of redone per query.

## Code

```cpp
class NumArray {
public:
    vector<int> nums;

    NumArray(vector<int>& nums) {
        this->nums = nums;
    }

    int sumRange(int left, int right) {
        int sum = 0;
        for (int i = left; i <= right; i++) {
            sum += nums[i];
        }
        return sum;
    }
};
```

---

# Approach 2 — Optimal / Prefix Sum Array

## Idea

Precompute a prefix sum array `prefix` where `prefix[i]` holds the sum of `nums[0..i-1]`. Any range sum `nums[left..right]` can then be computed as `prefix[right + 1] - prefix[left]` — no need to touch the original array again.

## Dry Run

```text
nums = [-2, 0, 3, -5, 2, -1]
```

Build prefix (size `n + 1`, `prefix[0] = 0`):

```text
prefix[0] = 0
prefix[1] = 0 + (-2) = -2
prefix[2] = -2 + 0 = -2
prefix[3] = -2 + 3 = 1
prefix[4] = 1 + (-5) = -4
prefix[5] = -4 + 2 = -2
prefix[6] = -2 + (-1) = -3
```

`sumRange(0, 2)`:

```text
prefix[3] - prefix[0] = 1 - 0 = 1
```

`sumRange(2, 5)`:

```text
prefix[6] - prefix[2] = -3 - (-2) = -1
```

Both match the brute-force results, computed in `O(1)`.

## Algorithm

1. Constructor:

   * Initialize `prefix` of size `n + 1`, with `prefix[0] = 0`.
   * For `i` from `0` to `n-1`: `prefix[i+1] = prefix[i] + nums[i]`.
2. `sumRange(left, right)`:

   * Return `prefix[right + 1] - prefix[left]`.

## Complexity

* **Time:** `O(n)` for the constructor, `O(1)` per `sumRange` call

  * The constructor does one linear pass to build the prefix array; each query afterward is a single subtraction.
* **Space:** `O(n)`

  * For the `prefix` array itself, one element larger than `nums`.

## Notes / Tips

* Using `prefix[i+1] = prefix[i] + nums[i]` and querying with `prefix[right+1] - prefix[left]` avoids any off-by-one special-casing for `left == 0`.
* This is the standard building block for a huge range of problems: subarray sum equals K, range update queries, 2D range sums (extended to a 2D prefix table), etc.
* Best used exactly when hinted by "immutable" + "multiple range queries" — precompute once, answer many times in `O(1)`.

## Code

```cpp
class NumArray {
public:
    vector<int> prefix;

    NumArray(vector<int>& nums) {
        int n = nums.size();
        prefix.assign(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }
    }

    int sumRange(int left, int right) {
        return prefix[right + 1] - prefix[left];
    }
};
```

---

## Key Template

```text
prefix[0] = 0
for i in 0..n-1:
    prefix[i+1] = prefix[i] + nums[i]

sumRange(left, right) = prefix[right + 1] - prefix[left]
```