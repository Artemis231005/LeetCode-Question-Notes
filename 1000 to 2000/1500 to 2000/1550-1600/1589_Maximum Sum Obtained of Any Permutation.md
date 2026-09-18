# LeetCode 1589 — Maximum Sum Obtained of Any Permutation

## Metadata

* **LeetCode:** 1589
* **Problem:** Maximum Sum Obtained of Any Permutation
* **Difficulty:** Medium
* **Topics:** Array, Greedy, Sorting, Prefix Sum
* **Pattern:** Difference Array (Frequency Counting) + Greedy Sorted Pairing
* **Key Technique:** Count how many requests cover each index using a difference array, then pair the largest counts with the largest values to maximize total sum
* **Optimal Complexity:** `O(n + m + n log n)` Time, `O(n)` Auxiliary Space

---

## Problem Statement

Given an array `nums` and a list of `requests[i] = [starti, endi]` (each meaning "sum `nums[starti..endi]` inclusive"), you may permute `nums` in any order. Return the maximum total sum across all requests achievable by some permutation, modulo `10^9 + 7`.

---

## Approaches

1. **Brute Force — Direct Range Increment for Counting**
2. **Optimal — Difference Array + Greedy Sorted Pairing**

---

# Approach 1 — Brute Force / Direct Range Increment for Counting

## Idea

The key insight (needed for both approaches) is that the total sum over all requests only depends on **how many requests cover each index**, not the specific requests themselves. Once that "coverage count" per index is known, the largest counts should be paired with the largest values in `nums` to maximize the total (since a value gets added once for every request covering its assigned position). This approach computes the coverage count the direct way: for every request, loop through its entire range and increment a counter at each index.

## Dry Run

```text
nums = [1, 2, 3, 4, 5], requests = [[1,3],[0,1]]
```

Process request `[1,3]`: increment indices `1, 2, 3`:

```text
count = [0, 1, 1, 1, 0]
```

Process request `[0,1]`: increment indices `0, 1`:

```text
count = [1, 2, 1, 1, 0]
```

Sort `count` descending: `[2, 1, 1, 1, 0]`.
Sort `nums` descending: `[5, 4, 3, 2, 1]`.

Pair largest with largest:

```text
5*2 + 4*1 + 3*1 + 2*1 + 1*0 = 10+4+3+2+0 = 19
```

## Algorithm

1. Initialize a `count` array of size `n`, all zeros.
2. For each request `[start, end]`:

   * For `i` from `start` to `end`, increment `count[i]`.
3. Sort `count` in descending order, and sort `nums` in descending order.
4. Sum `nums[i] * count[i]` for every `i`, taking the result modulo `10^9 + 7`.
5. Return the sum.

## Complexity

* **Time:** `O(n * m + n log n)`

  * Each of the `m` requests can directly touch up to `n` indices; sorting both arrays afterward costs `O(n log n)`.
* **Space:** `O(n)`

  * For the `count` array.

## Notes / Tips

* Directly incrementing every index in a request's range is redundant — a range's effect on the count array only needs to be marked once, at its boundaries, rather than applied to every index inside it (same insight as LC 1109 and LC 2848).
* The greedy pairing step (largest count with largest value) itself is already correct and optimal in both approaches — the only difference between brute force and optimal here is how the coverage counts are computed, not the pairing logic.

## Code

```cpp
class Solution {
public:
    int maxSumRangeQuery(vector<int>& nums, vector<vector<int>>& requests) {
        int n = nums.size();
        const int MOD = 1e9 + 7;
        vector<long long> count(n, 0);

        for (auto& req : requests) {
            for (int i = req[0]; i <= req[1]; i++) {
                count[i]++;
            }
        }

        sort(count.rbegin(), count.rend());
        sort(nums.rbegin(), nums.rend());

        long long total = 0;
        for (int i = 0; i < n; i++) {
            total = (total + (long long)nums[i] * count[i]) % MOD;
        }

        return (int)total;
    }
};
```

---

# Approach 2 — Optimal / Difference Array + Greedy Sorted Pairing

## Idea

Instead of incrementing every index a request covers, mark the request's *effect* directly using a difference array: add `+1` at the request's start, and subtract `1` just after its end. A single prefix sum sweep then turns this into the exact same coverage count as Approach 1, but built in linear time. From there, the greedy pairing (sort both `count` and `nums` descending, multiply pairwise) is unchanged.

## Dry Run

```text
nums = [1, 2, 3, 4, 5], requests = [[1,3],[0,1]]
```

Build difference array `diff` of size `n + 1`:

```text
[1,3] → diff[1] += 1, diff[4] -= 1
[0,1] → diff[0] += 1, diff[2] -= 1
```

```text
diff = [1, 1, -1, 0, -1, 0]
```

Prefix sum sweep to get `count`:

```text
i=0: running=1 → count[0]=1
i=1: running=1+1=2 → count[1]=2
i=2: running=2-1=1 → count[2]=1
i=3: running=1+0=1 → count[3]=1
i=4: running=1-1=0 → count[4]=0
```

```text
count = [1, 2, 1, 1, 0]
```

Matches Approach 1's result. Sort `count` descending: `[2,1,1,1,0]`. Sort `nums` descending: `[5,4,3,2,1]`.

Pair and sum:

```text
5*2 + 4*1 + 3*1 + 2*1 + 1*0 = 19
```

## Algorithm

1. Create a difference array `diff` of size `n + 1`, all zeros.
2. For each request `[start, end]`:

   * `diff[start] += 1`.
   * `diff[end + 1] -= 1`.
3. Sweep `diff` with a running sum to build the `count` array (one entry per index of `nums`).
4. Sort `count` in descending order, and sort `nums` in descending order.
5. Sum `nums[i] * count[i]` for every `i`, taking the result modulo `10^9 + 7`.
6. Return the sum.

## Complexity

* **Time:** `O(n + m + n log n)`

  * `O(m)` to apply all request markers, `O(n)` to sweep and build `count`, `O(n log n)` to sort both arrays.
* **Space:** `O(n)`

  * For the difference array and the `count` array.

## Notes / Tips

* This is the same difference array technique as LC 1109 (Corporate Flight Bookings) and LC 2848 (Points That Intersect With Cars), the only new idea specific to this problem is what happens *after* the counts are computed: pairing sorted counts with sorted values to maximize total sum.
* The greedy pairing itself follows the rearrangement inequality: to maximize the sum of pairwise products between two sequences, pair the largest with the largest, second-largest with second-largest, and so on.
* Since request ranges can overlap arbitrarily and permuting `nums` doesn't change which "slots" get how much coverage, only the **multiset of coverage counts** (not which specific index has which count) actually matters for the final answer — this is what allows freely sorting both arrays independently before pairing.

## Code

```cpp
class Solution {
public:
    int maxSumRangeQuery(vector<int>& nums, vector<vector<int>>& requests) {
        int n = nums.size();
        const int MOD = 1e9 + 7;
        vector<long long> diff(n + 1, 0);

        for (auto& req : requests) {
            diff[req[0]]++;
            diff[req[1] + 1]--;
        }

        vector<long long> count(n);
        long long running = 0;
        for (int i = 0; i < n; i++) {
            running += diff[i];
            count[i] = running;
        }

        sort(count.rbegin(), count.rend());
        sort(nums.rbegin(), nums.rend());

        long long total = 0;
        for (int i = 0; i < n; i++) {
            total = (total + (long long)nums[i] * count[i]) % MOD;
        }

        return (int)total;
    }
};
```

---

## Key Template

```text
diff = array of size (n + 1), all 0

for [start, end] in requests:
    diff[start] += 1
    diff[end + 1] -= 1

count = array of size n
running = 0
for i = 0 to n-1:
    running += diff[i]
    count[i] = running

sort(count, descending)
sort(nums, descending)

total = 0
for i = 0 to n-1:
    total = (total + nums[i] * count[i]) % MOD

return total
```