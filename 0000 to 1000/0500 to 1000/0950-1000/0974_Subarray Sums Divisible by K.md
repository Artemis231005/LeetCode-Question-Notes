# LeetCode 974 — Subarray Sums Divisible by K

## Metadata

* **LeetCode:** 974
* **Problem:** Subarray Sums Divisible by K
* **Difficulty:** Medium
* **Topics:** Array, Hash Map, Prefix Sum
* **Pattern:** Prefix Sum + Hash Map Frequency Count (Remainder Grouping)
* **Key Technique:** Two prefix sums with the same remainder mod `k` mean everything between them sums to a multiple of `k`
* **Optimal Complexity:** `O(n)` Time, `O(k)` Space

---

## Problem Statement

Given an integer array `nums` and an integer `k`, return the number of contiguous subarrays whose sum is divisible by `k`.

---

## Approaches

1. **Brute Force — Check Every Subarray**
2. **Better — Prefix Sum Array + Nested Check**
3. **Optimal — Prefix Sum Remainders + Hash Map**

---

# Approach 1 — Brute Force / Check Every Subarray

## Idea

For every possible subarray, accumulate its sum incrementally and check whether it's divisible by `k`.

## Dry Run

```text
nums = [4, 5, 0, -2, -3, 1], k = 5
```

Start `i = 0`:

```text
[4] = 4 → 4 % 5 != 0
[4,5] = 9 → 9 % 5 != 0
[4,5,0] = 9 → no
[4,5,0,-2] = 7 → no
[4,5,0,-2,-3] = 4 → no
[4,5,0,-2,-3,1] = 5 → 5 % 5 == 0 → count = 1
```

Continue for every other start index the same way, checking each running sum's remainder mod `k`. Doing this for all starts gives the final count.

## Algorithm

1. Initialize `count = 0`.
2. For each start index `i` from `0` to `n-1`:

   * Initialize `sum = 0`.
   * For each end index `j` from `i` to `n-1`:

     * `sum += nums[j]`.
     * If `sum % k == 0` (careful with negative sums), increment `count`.
3. Return `count`.

## Complexity

* **Time:** `O(n²)`

  * Every pair of `(start, end)` indices is checked, with the sum accumulated incrementally in the inner loop rather than recomputed from scratch.
* **Space:** `O(1)`

  * Only a running sum and counter are tracked — no extra structures allocated.

## Notes / Tips

* Watch out for negative sums with C++'s `%` operator — it can return a negative remainder, so `((sum % k) + k) % k` is needed to normalize it to `[0, k-1]` before comparing to `0`.
* Correct but doesn't scale — the same redundant re-derivation problem seen in LC 560.

## Code

```cpp
class Solution {
public:
    int subarraysDivByK(vector<int>& nums, int k) {
        int n = nums.size();
        int count = 0;

        for (int i = 0; i < n; i++) {
            int sum = 0;
            for (int j = i; j < n; j++) {
                sum += nums[j];
                int rem = ((sum % k) + k) % k;
                if (rem == 0) {
                    count++;
                }
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Better / Prefix Sum Array + Nested Check

## Idea

Precompute the prefix sum array once. Any subarray sum `nums[i..j]` is then `prefix[j+1] - prefix[i]`. Check every pair `(i, j)` for divisibility by `k` using this O(1) lookup instead of accumulating manually.

## Dry Run

```text
nums = [4, 5, 0, -2, -3, 1], k = 5
```

Prefix sums (`prefix[0] = 0`):

```text
prefix = [0, 4, 9, 9, 7, 4, 5]
```

Check pair `(i=0, j=6)` (whole array):

```text
prefix[6] - prefix[0] = 5 - 0 = 5 → 5 % 5 == 0 → count += 1
```

Check pair `(i=1, j=2)`:

```text
prefix[2] - prefix[1] = 9 - 4 = 5 → divisible → count += 1
```

Continue across all `(i, j)` pairs the same way to reach the final count.

## Algorithm

1. Build `prefix` array of size `n + 1`, with `prefix[0] = 0` and `prefix[i+1] = prefix[i] + nums[i]`.
2. For each pair `i < j` (0-indexed into `prefix`, representing subarray `nums[i..j-1]`):

   * If `(prefix[j] - prefix[i]) % k == 0` (normalized for negatives), increment `count`.
3. Return `count`.

## Complexity

* **Time:** `O(n²)`

  * Building the prefix array is `O(n)`, but checking every pair of prefix indices is still `O(n²)`.
* **Space:** `O(n)`

  * For the `prefix` array.

## Notes / Tips

* Same intermediate step seen in LC 560 — precomputing prefix sums removes redundant summation but not the quadratic pair-checking cost.
* The insight that unlocks Approach 3: two prefix sums with the *same remainder mod k* automatically satisfy `(prefix[j] - prefix[i]) % k == 0`, so grouping by remainder replaces the need to check every pair explicitly.

## Code

```cpp
class Solution {
public:
    int subarraysDivByK(vector<int>& nums, int k) {
        int n = nums.size();
        vector<int> prefix(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] + nums[i];
        }

        int count = 0;
        for (int i = 0; i < n; i++) {
            for (int j = i + 1; j <= n; j++) {
                int diff = prefix[j] - prefix[i];
                int rem = ((diff % k) + k) % k;
                if (rem == 0) {
                    count++;
                }
            }
        }

        return count;
    }
};
```

---

# Approach 3 — Optimal / Prefix Sum Remainders + Hash Map

## Idea

A subarray `nums[i..j]` is divisible by `k` exactly when `prefix[j] % k == prefix[i] % k` — the two prefix sums land in the same remainder class. Instead of comparing every pair, keep a hash map (or fixed-size array of size `k`) counting how many times each remainder has occurred so far. Every time the current remainder repeats, all previous occurrences of that same remainder pair up with it to form a valid subarray.

## Dry Run

```text
nums = [4, 5, 0, -2, -3, 1], k = 5
```

Initialize `remainderCount = {0: 1}` (empty prefix has remainder `0`), `sum = 0`, `count = 0`.

```text
i=0, num=4: sum=4, rem=4 → count += remainderCount[4]=0 → count=0
   remainderCount[4] = 1

i=1, num=5: sum=9, rem=4 → count += remainderCount[4]=1 → count=1
   remainderCount[4] = 2

i=2, num=0: sum=9, rem=4 → count += remainderCount[4]=2 → count=3
   remainderCount[4] = 3

i=3, num=-2: sum=7, rem=2 → count += remainderCount[2]=0 → count=3
   remainderCount[2] = 1

i=4, num=-3: sum=4, rem=4 → count += remainderCount[4]=3 → count=6
   remainderCount[4] = 4

i=5, num=1: sum=5, rem=0 → count += remainderCount[0]=1 → count=7
   remainderCount[0] = 2
```

Final `count = 7`, matching the expected answer.

## Algorithm

1. Initialize a frequency array/map `remainderCount` of size `k`, with `remainderCount[0] = 1`.
2. Initialize `sum = 0` and `count = 0`.
3. For each value in `nums`:

   * `sum += num`.
   * `rem = ((sum % k) + k) % k` (normalize for negative sums).
   * `count += remainderCount[rem]`.
   * `remainderCount[rem]++`.
4. Return `count`.

## Complexity

* **Time:** `O(n)`

  * Single pass, each remainder lookup/update is `O(1)`.
* **Space:** `O(k)`

  * The remainder counts only ever take `k` distinct values (`0` to `k-1`), so a fixed-size array of length `k` works instead of an unbounded hash map.

## Notes / Tips

* Normalizing the remainder with `((sum % k) + k) % k` is essential in C++ since `%` can return negative values for negative sums — skipping this breaks the remainder grouping entirely.
* Same overall shape as LC 560 (prefix sum + hash map counting), but grouped by **remainder mod k** instead of exact prefix sum value, and using `count += frequency` instead of a direct "difference lookup," since any two prefix sums sharing a remainder automatically work — no specific target value needs to be searched for.
* Seeding with `remainderCount[0] = 1` handles subarrays starting at index `0` that are already divisible by `k`.

## Code

```cpp
class Solution {
public:
    int subarraysDivByK(vector<int>& nums, int k) {
        vector<int> remainderCount(k, 0);
        remainderCount[0] = 1;

        int sum = 0, count = 0;

        for (int num : nums) {
            sum += num;
            int rem = ((sum % k) + k) % k;

            count += remainderCount[rem];
            remainderCount[rem]++;
        }

        return count;
    }
};
```

---

## Key Template

```text
remainderCount = array of size k, all 0
remainderCount[0] = 1
sum = 0
count = 0

for num in nums:
    sum += num
    rem = ((sum % k) + k) % k
    count += remainderCount[rem]
    remainderCount[rem] += 1

return count
```