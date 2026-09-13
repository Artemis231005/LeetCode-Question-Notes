# LeetCode 1310 — XOR Queries of a Subarray

## Metadata

* **LeetCode:** 1310
* **Problem:** XOR Queries of a Subarray
* **Difficulty:** Medium
* **Topics:** Array, Bit Manipulation, Prefix Sum
* **Pattern:** Prefix XOR
* **Key Technique:** Precompute a running XOR array once so any range XOR becomes a single XOR of two prefix values, exploiting that XOR-ing a value with itself cancels it out
* **Optimal Complexity:** `O(n + q)` Time, `O(n)` Space

---

## Problem Statement

Given an array `arr` and a list of queries `[left, right]`, return an array where each answer is the XOR of all elements in `arr[left..right]` (inclusive).

---

## Approaches

1. **Brute Force — XOR Each Range Directly**
2. **Optimal — Prefix XOR Array**

---

# Approach 1 — Brute Force / XOR Each Range Directly

## Idea

For each query, loop from `left` to `right` and XOR all the values together directly.

## Dry Run

```text
arr = [1, 3, 4, 8], queries = [[0,1],[1,2],[0,3],[3,3]]
```

Query `[0,1]`:

```text
1 ^ 3 = 2
```

Query `[1,2]`:

```text
3 ^ 4 = 7
```

Query `[0,3]`:

```text
1 ^ 3 ^ 4 ^ 8 = 14
```

Query `[3,3]`:

```text
8
```

Result: `[2, 7, 14, 8]`.

## Algorithm

1. For each query `[left, right]`:

   * Initialize `result = 0`.
   * Loop `i` from `left` to `right`, XOR-ing `arr[i]` into `result`.
   * Store `result` as the answer for this query.
2. Return all answers.

## Complexity

* **Time:** `O(n * q)`

  * Each of the `q` queries can scan up to `n` elements directly.
* **Space:** `O(1)`

  * Only a running XOR accumulator per query — no extra structures allocated (beyond the required output).

## Notes / Tips

* Every query recomputes its range's XOR from scratch, but XOR ranges can be derived from cumulative XOR values, exactly like prefix sums can be derived for addition.
* This is the direct analog of LC 303's brute force, just with XOR instead of `+`.

## Code

```cpp
class Solution {
public:
    vector<int> xorQueries(vector<int>& arr, vector<vector<int>>& queries) {
        vector<int> result;

        for (auto& q : queries) {
            int left = q[0], right = q[1];
            int xorVal = 0;

            for (int i = left; i <= right; i++) {
                xorVal ^= arr[i];
            }

            result.push_back(xorVal);
        }

        return result;
    }
};
```

---

# Approach 2 — Optimal / Prefix XOR Array

## Idea

Build a prefix XOR array where `prefix[i]` holds the XOR of `arr[0..i-1]`. Since XOR-ing a value with itself cancels it out to `0` (`x ^ x = 0`), the range XOR `arr[left..right]` can be computed as `prefix[right+1] ^ prefix[left]` — the portion `arr[0..left-1]` appears in both prefix values and cancels out, leaving exactly the XOR of `arr[left..right]`.

## Dry Run

```text
arr = [1, 3, 4, 8]
```

Build prefix (size `n+1`, `prefix[0] = 0`):

```text
prefix[0] = 0
prefix[1] = 0 ^ 1 = 1
prefix[2] = 1 ^ 3 = 2
prefix[3] = 2 ^ 4 = 6
prefix[4] = 6 ^ 8 = 14
```

Query `[0,1]`:

```text
prefix[2] ^ prefix[0] = 2 ^ 0 = 2
```

Query `[1,2]`:

```text
prefix[3] ^ prefix[1] = 6 ^ 1 = 7
```

Query `[0,3]`:

```text
prefix[4] ^ prefix[0] = 14 ^ 0 = 14
```

Query `[3,3]`:

```text
prefix[4] ^ prefix[3] = 14 ^ 6 = 8
```

Result: `[2, 7, 14, 8]`, matching the brute-force output.

## Algorithm

1. Build `prefix` array of size `n + 1`, with `prefix[0] = 0` and `prefix[i+1] = prefix[i] ^ arr[i]`.
2. For each query `[left, right]`:

   * Compute the answer as `prefix[right + 1] ^ prefix[left]`.
3. Return all answers.

## Complexity

* **Time:** `O(n + q)`

  * `O(n)` to build the prefix XOR array, `O(1)` per query afterward.
* **Space:** `O(n)`

  * For the `prefix` array itself.

## Notes / Tips

* This is the exact same structural idea as LC 303's prefix sum, substituting XOR for addition — both operations support the "prefix combine then cancel the overlap" trick because they're associative and have an inverse operation (subtraction for `+`, XOR-with-itself for `^`).
* No special handling is needed for `left == 0`, since `prefix[0] = 0` naturally acts as the identity element for XOR — `prefix[right+1] ^ 0` just returns `prefix[right+1]` unchanged.

## Code

```cpp
class Solution {
public:
    vector<int> xorQueries(vector<int>& arr, vector<vector<int>>& queries) {
        int n = arr.size();
        vector<int> prefix(n + 1, 0);

        for (int i = 0; i < n; i++) {
            prefix[i + 1] = prefix[i] ^ arr[i];
        }

        vector<int> result;
        for (auto& q : queries) {
            int left = q[0], right = q[1];
            result.push_back(prefix[right + 1] ^ prefix[left]);
        }

        return result;
    }
};
```

---

## Key Template

```text
prefix[0] = 0
for i in 0..n-1:
    prefix[i+1] = prefix[i] ^ arr[i]

for each query [left, right]:
    answer = prefix[right + 1] ^ prefix[left]

return answers
```