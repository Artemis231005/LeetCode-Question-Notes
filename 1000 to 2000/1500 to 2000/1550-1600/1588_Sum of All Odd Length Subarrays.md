# LeetCode 1588 — Sum of All Odd Length Subarrays

## Metadata

* **LeetCode:** 1588
* **Problem:** Sum of All Odd Length Subarrays
* **Difficulty:** Easy
* **Topics:** Array, Math, Prefix Sum
* **Pattern:** Contribution Counting
* **Key Technique:** Instead of summing every subarray directly, compute how many odd-length subarrays each individual element belongs to, and multiply that count by the element's value — summing these contributions gives the same total in one pass
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `arr`, return the sum of all possible odd-length subarrays' sums.

---

## Approaches

1. **Brute Force — Sum Every Odd-Length Subarray Directly**
2. **Better — Prefix Sum + Sum Every Odd-Length Subarray**
3. **Optimal — Contribution Counting (No Subarray Enumeration)**

---

# Approach 1 — Brute Force / Sum Every Odd-Length Subarray Directly

## Idea

For every possible starting index and every odd length, extract that subarray and sum it directly, accumulating the total across all of them.

## Dry Run

```text
arr = [1, 4, 2, 5, 3]
```

Length `1` subarrays: `[1],[4],[2],[5],[3]` → sums `1,4,2,5,3` → contributes `15`.

Length `3` subarrays: `[1,4,2]=7, [4,2,5]=11, [2,5,3]=10` → contributes `28`.

Length `5` subarray: `[1,4,2,5,3]=15` → contributes `15`.

Total: `15 + 28 + 15 = 58`, matching the expected output.

## Algorithm

1. Initialize `total = 0`.
2. For each odd length `len` from `1` to `n`, stepping by `2`:

   * For each start index `i` from `0` to `n - len`:

     * Sum `arr[i .. i+len-1]` directly.
     * Add that sum to `total`.
3. Return `total`.

## Complexity

* **Time:** `O(n³)`

  * For each of `O(n)` odd lengths, up to `O(n)` starting positions, each requiring an `O(n)` sum of the subarray directly.
* **Space:** `O(1)`

  * Only a running total and loop counters.

## Notes / Tips

* Summing each subarray from scratch is the main inefficiency here — a prefix sum array (Approach 2) removes the innermost summation, and recognizing each element's total contribution across all subarrays (Approach 3) removes the need to enumerate subarrays at all.

## Code

```cpp
class Solution {
public:
    int sumOddLengthSubarrays(vector<int>& arr) {
        int n = arr.size();
        int total = 0;

        for (int len = 1; len <= n; len += 2) {
            for (int i = 0; i + len <= n; i++) {
                int sum = 0;
                for (int j = i; j < i + len; j++) {
                    sum += arr[j];
                }
                total += sum;
            }
        }

        return total;
    }
};
```

---

# Approach 2 — Better / Prefix Sum + Sum Every Odd-Length Subarray

## Idea

Precompute a prefix sum array once. Any subarray's sum can then be looked up in `O(1)` via `prefix[end+1] - prefix[start]`, removing the innermost summation loop from Approach 1.

## Dry Run

```text
arr = [1, 4, 2, 5, 3]
```

Prefix sums (`prefix[0] = 0`):

```text
prefix = [0, 1, 5, 7, 12, 15]
```

Length `3`, start `0`: `prefix[3] - prefix[0] = 7`.
Length `3`, start `1`: `prefix[4] - prefix[1] = 11`.
Length `3`, start `2`: `prefix[5] - prefix[2] = 10`.

Continuing across all odd lengths and start positions gives the same total: `58`.

## Algorithm

1. Build `prefix` array of size `n + 1`, with `prefix[0] = 0` and `prefix[i+1] = prefix[i] + arr[i]`.
2. Initialize `total = 0`.
3. For each odd length `len` from `1` to `n`, stepping by `2`:

   * For each start index `i` from `0` to `n - len`:

     * `total += prefix[i + len] - prefix[i]`.
4. Return `total`.

## Complexity

* **Time:** `O(n²)`

  * Building the prefix array is `O(n)`, but summing over all odd lengths and starting positions is still `O(n²)`, each lookup now `O(1)`.
* **Space:** `O(n)`

  * For the `prefix` array.

## Notes / Tips

* This removes the cubic blowup from Approach 1, but still enumerates every odd-length subarray explicitly — Approach 3 skips this enumeration entirely by reframing the problem around each element's individual contribution.
* This is the same "precompute prefix sums to speed up repeated range sum queries" idea as LC 303, just applied across every subarray length instead of a fixed set of queries.

## Code

```cpp
class Solution {
public:
    int sumOddLengthSubarrays(vector<int>& arr) {
        int n = arr.size();
        vector<int> prefix(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + arr[i];
        }

        int total = 0;
        for (int len = 1; len <= n; len += 2) {
            for (int i = 0; i + len <= n; i++) {
                total += prefix[i + len] - prefix[i];
            }
        }

        return total;
    }
};
```

---

# Approach 3 — Optimal / Contribution Counting (No Subarray Enumeration)

## Idea

Instead of summing subarrays, ask a different question: for each individual element `arr[i]`, in how many **odd-length** subarrays does it appear at all? That count, multiplied by `arr[i]`, is exactly `arr[i]`'s total contribution to the final answer (since it gets added once for every subarray containing it). Summing every element's contribution this way gives the same total, without ever building or summing an actual subarray.

For element at index `i` (0-indexed), the number of choices for where a subarray containing it can start is `i + 1` (positions `0` through `i`), and the number of choices for where it can end is `n - i` (positions `i` through `n-1`). Multiplying gives the **total** number of subarrays (any length) containing `arr[i]`: `(i+1) * (n-i)`. Exactly half of these (rounded appropriately) have odd length — computed directly as `ceil(totalSubarrays / 2)`.

## Dry Run

```text
arr = [1, 4, 2, 5, 3]   (n = 5)
```

For `i = 0` (value `1`):

```text
startChoices = 0+1 = 1, endChoices = 5-0 = 5
totalSubarrays = 1*5 = 5
oddCount = ceil(5/2) = 3
contribution = 1 * 3 = 3
```

For `i = 1` (value `4`):

```text
startChoices = 2, endChoices = 4
totalSubarrays = 8
oddCount = ceil(8/2) = 4
contribution = 4 * 4 = 16
```

For `i = 2` (value `2`):

```text
startChoices = 3, endChoices = 3
totalSubarrays = 9
oddCount = ceil(9/2) = 5
contribution = 2 * 5 = 10
```

For `i = 3` (value `5`):

```text
startChoices = 4, endChoices = 2
totalSubarrays = 8
oddCount = 4
contribution = 5 * 4 = 20
```

For `i = 4` (value `3`):

```text
startChoices = 5, endChoices = 1
totalSubarrays = 5
oddCount = 3
contribution = 3 * 3 = 9
```

Total: `3 + 16 + 10 + 20 + 9 = 58`, matching both earlier approaches.

## Algorithm

1. Initialize `total = 0`.
2. For each index `i` from `0` to `n-1`:

   * `startChoices = i + 1`, `endChoices = n - i`.
   * `totalSubarrays = startChoices * endChoices`.
   * `oddCount = (totalSubarrays + 1) / 2` (integer division rounding up).
   * `total += arr[i] * oddCount`.
3. Return `total`.

## Complexity

* **Time:** `O(n)`

  * A single pass over the array, constant work per element.
* **Space:** `O(1)`

  * Only a running total — no prefix array or extra structures.

## Notes / Tips

* This "contribution counting" reframing — asking how many times each element is counted across all valid structures, rather than enumerating the structures themselves — is a powerful general technique whenever a problem sums over many overlapping subarrays/subsets and the per-element contribution has a clean closed form.
* The `(totalSubarrays + 1) / 2` formula for "half rounded up" works because subarray counts alternate in a specific way, and this arithmetic trick avoids needing a separate `if totalSubarrays is odd/even` branch.
* This is a genuinely different algorithmic idea from Approaches 1 and 2 (which both explicitly build or sum subarrays) — it's worth recognizing as a distinct technique rather than just a further optimization of the prefix-sum approach.

## Code

```cpp
class Solution {
public:
    int sumOddLengthSubarrays(vector<int>& arr) {
        int n = arr.size();
        long long total = 0;

        for (int i = 0; i < n; i++) {
            int startChoices = i + 1;
            int endChoices = n - i;
            int totalSubarrays = startChoices * endChoices;
            int oddCount = (totalSubarrays + 1) / 2;

            total += (long long)arr[i] * oddCount;
        }

        return (int)total;
    }
};
```

---

## Key Template

```text
total = 0

for i in 0..n-1:
    startChoices = i + 1
    endChoices = n - i
    totalSubarrays = startChoices * endChoices
    oddCount = (totalSubarrays + 1) / 2

    total += arr[i] * oddCount

return total
```