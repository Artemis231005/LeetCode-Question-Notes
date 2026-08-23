# LeetCode 3804 — Number of Centered Subarrays

## Metadata

* **LeetCode:** 3804
* **Problem:** Number of Centered Subarrays
* **Difficulty:** Easy
* **Topics:** Array, Prefix Sum, Hash Table
* **Pattern:** Expand Around Center + Prefix Sum
* **Optimal Complexity:** O(n²) time, O(1) extra space

---

## Idea

A subarray is **centered** if its middle element is equal to the sum of all elements on either side of it.

For a subarray from `l` to `r`, choose its center `i`.

We can expand around every possible center and maintain:

* `leftSum` = sum of elements between `l` and `i - 1`
* `rightSum` = sum of elements between `i + 1` and `r`

For each expansion, if:

```text
leftSum == rightSum == nums[i]
```

then the current subarray is centered.

Since the subarray must have a unique center, only subarrays of **odd length** need to be considered.

---

## Dry Run

### `nums = [1, 2, 1]`

Take `2` as the center.

```text
leftSum  = 1
center   = 2
rightSum = 1
```

Since:

```text
leftSum == rightSum
```

and the center satisfies the required condition, `[1, 2, 1]` is counted.

The single-element subarray `[2]` is also checked according to the problem's definition.

---

## Algorithm

1. Initialize `ans = 0`.
2. Treat every index `i` as the center.
3. Initialize:

   * `leftSum = 0`
   * `rightSum = 0`
4. Expand outward from `i`.
5. For every expansion:

   * Add the left element to `leftSum`.
   * Add the right element to `rightSum`.
6. Check the centered-subarray condition.
7. If valid, increment `ans`.
8. Return `ans`.

---

## Complexity

* **Time:** `O(n²)` — each possible center can expand up to `O(n)` positions.
* **Space:** `O(1)` extra space.

---

## Code

```cpp
class Solution {
public:
    long long countSubarrays(vector<int>& nums) {
        int n = nums.size();
        long long ans = 0;

        for (int i = 0; i < n; i++) {
            long long leftSum = 0;
            long long rightSum = 0;

            for (int d = 1; i - d >= 0 && i + d < n; d++) {
                leftSum += nums[i - d];
                rightSum += nums[i + d];

                if (leftSum == rightSum && leftSum == nums[i]) {
                    ans++;
                }
            }
        }

        return ans;
    }
};
```

---

## Notes / Tips

* **Odd-length subarrays** are handled naturally by choosing a center and expanding equally on both sides.
* The two sides must contain the **same number of elements** because they are equidistant from the center.
* Maintain the sums while expanding instead of recomputing them.
* Use `long long` for sums and the answer when array values can make the sum large.
* The key pattern is **expand around center while maintaining aggregate information**.

---

## Key Template

```text
for every possible center:
    leftSum = 0
    rightSum = 0

    expand equally in both directions:
        update leftSum
        update rightSum

        check centered condition
```

**Pattern:** When a subarray property depends on a unique middle element and symmetric left/right portions, try **expanding around every center**.
