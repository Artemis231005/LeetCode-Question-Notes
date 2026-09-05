# LeetCode 238 — Product of Array Except Self

## Metadata

* **LeetCode:** 238
* **Problem:** Product of Array Except Self
* **Difficulty:** Medium
* **Topics:** Array, Prefix Sum
* **Pattern:** Prefix / Suffix Product
* **Key Technique:** For each index, the answer is the product of everything to its left times everything to its right — computed as two passes, then merged into one
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given an array `nums`, return an array `answer` such that `answer[i]` equals the product of all elements of `nums` except `nums[i]`. Must run in `O(n)` time without using division, and the output array doesn't count toward space complexity.

---

## Approaches

1. **Brute Force — Multiply Every Other Element**
2. **Better — Division-Based (Disallowed but Instructive)**
3. **Optimal — Prefix and Suffix Products**

---

# Approach 1 — Brute Force / Multiply Every Other Element

## Idea

For each index `i`, loop through the entire array and multiply together every element except `nums[i]`.

## Dry Run

```text
nums = [1, 2, 3, 4]
```

`i = 0`:

```text
2 * 3 * 4 = 24
```

`i = 1`:

```text
1 * 3 * 4 = 12
```

`i = 2`:

```text
1 * 2 * 4 = 8
```

`i = 3`:

```text
1 * 2 * 3 = 6
```

Result:

```text
[24, 12, 8, 6]
```

## Algorithm

1. Create a `result` array of the same length as `nums`.
2. For each index `i`:

   * Initialize `product = 1`.
   * Loop `j` from `0` to `n-1`, skipping `j == i`, multiplying `nums[j]` into `product`.
   * Store `product` in `result[i]`.
3. Return `result`.

## Complexity

* **Time:** `O(n²)`

  * Each of the `n` indices triggers a fresh inner loop over up to `n` elements.
* **Space:** `O(1)`

  * No extra structure beyond the required output array — no additional data structures allocated.

## Notes / Tips

* Recomputes almost the entire product from scratch at every index — massive redundant work.
* Doesn't risk overflow issues from division, but scales the worst of all three approaches.

## Code

```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n);

        for (int i = 0; i < n; i++) {
            int product = 1;
            for (int j = 0; j < n; j++) {
                if (j != i) {
                    product *= nums[j];
                }
            }
            result[i] = product;
        }

        return result;
    }
};
```

---

# Approach 2 — Better / Division-Based (Disallowed but Instructive)

## Idea

Compute the product of the entire array once. For each index, the answer is just `totalProduct / nums[i]`. This is fast but breaks down when the array contains a `0`, and the problem explicitly disallows division as the intended solution.

## Dry Run

```text
nums = [1, 2, 3, 4]
total = 1*2*3*4 = 24
```

```text
result[0] = 24 / 1 = 24
result[1] = 24 / 2 = 12
result[2] = 24 / 3 = 8
result[3] = 24 / 4 = 6
```

Result:

```text
[24, 12, 8, 6]
```

With a zero in the array:

```text
nums = [1, 0, 3, 4]
total = 0
```

Every division by a nonzero `nums[i]` gives `0`, but `result[1]` (the actual zero's position) needs the product of the other three (`1*3*4 = 12`), which `total / 0` cannot compute — this case needs special handling, which defeats the simplicity of the division trick.

## Algorithm

1. Compute `total = product of all nums`.
2. If no zero is present, `result[i] = total / nums[i]` for every `i`.
3. If exactly one zero is present, every `result[i]` is `0` except at the zero's index, which becomes the product of all other (nonzero) elements.
4. If more than one zero is present, every `result[i]` is `0`.

## Complexity

* **Time:** `O(n)`

  * One pass to compute the total product, one pass to divide (plus extra handling for zero cases).
* **Space:** `O(1)`

  * Only the total product and zero-count are tracked.

## Notes / Tips

* Technically `O(n)`, but the problem statement explicitly forbids division as the intended solution, and the zero-handling makes it messier than the prefix/suffix approach.
* Also risky with very large arrays — the running product can overflow standard integer types well before the final answer is reached.

## Code

```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n);

        int zeroCount = 0;
        long long product = 1;

        for (int num : nums) {
            if (num == 0) {
                zeroCount++;
            } else {
                product *= num;
            }
        }

        for (int i = 0; i < n; i++) {
            if (zeroCount > 1) {
                result[i] = 0;
            } else if (zeroCount == 1) {
                result[i] = (nums[i] == 0) ? (int)product : 0;
            } else {
                result[i] = (int)(product / nums[i]);
            }
        }

        return result;
    }
};
```

---

# Approach 3 — Optimal / Prefix and Suffix Products

## Idea

`result[i]` is exactly `(product of everything left of i) * (product of everything right of i)`. Build a prefix-product array where `prefix[i]` holds the product of `nums[0..i-1]`, and a suffix-product array where `suffix[i]` holds the product of `nums[i+1..n-1]`. Then `result[i] = prefix[i] * suffix[i]`. No division needed anywhere.

## Dry Run

```text
nums = [1, 2, 3, 4]
```

Prefix products (`prefix[i]` = product of everything before `i`):

```text
prefix[0] = 1
prefix[1] = 1 * 1 = 1
prefix[2] = 1 * 2 = 2
prefix[3] = 2 * 3 = 6
```

Suffix products (`suffix[i]` = product of everything after `i`):

```text
suffix[3] = 1
suffix[2] = 1 * 4 = 4
suffix[1] = 4 * 3 = 12
suffix[0] = 12 * 2 = 24
```

Combine:

```text
result[0] = prefix[0] * suffix[0] = 1 * 24 = 24
result[1] = prefix[1] * suffix[1] = 1 * 12 = 12
result[2] = prefix[2] * suffix[2] = 2 * 4 = 8
result[3] = prefix[3] * suffix[3] = 6 * 1 = 6
```

Result:

```text
[24, 12, 8, 6]
```

## Algorithm

1. Initialize `result` array of size `n`, all set to `1`.
2. Left-to-right pass: keep a running `prefix` product, set `result[i] = prefix` before updating, then `prefix *= nums[i]`.
3. Right-to-left pass: keep a running `suffix` product, multiply `result[i] *= suffix` before updating, then `suffix *= nums[i]`.
4. Return `result`.

## Complexity

* **Time:** `O(n)`

  * Two linear passes over the array — one left to right, one right to left.
* **Space:** `O(1)`

  * The output array doesn't count toward space complexity per the problem's constraints; beyond that, only two running product variables (`prefix`, `suffix`) are used — no separate prefix/suffix arrays needed.

## Notes / Tips

* The two-pass version can be fused into a single output array by writing prefix products in the first pass and multiplying suffix products into the same array in the second pass — this is what gets space down to `O(1)` auxiliary.
* No division anywhere, so zeros in the array are handled naturally without special-casing.
* Same prefix/suffix accumulation idea as LC 42 (Trapping Rain Water), just with product instead of max.

## Code

```cpp
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n, 1);

        int prefix = 1;
        for (int i = 0; i < n; i++) {
            result[i] = prefix;
            prefix *= nums[i];
        }

        int suffix = 1;
        for (int i = n - 1; i >= 0; i--) {
            result[i] *= suffix;
            suffix *= nums[i];
        }

        return result;
    }
};
```

---

## Key Template

```text
result = [1] * n

prefix = 1
for i in 0..n-1:
    result[i] = prefix
    prefix *= nums[i]

suffix = 1
for i in n-1..0:
    result[i] *= suffix
    suffix *= nums[i]

return result
```