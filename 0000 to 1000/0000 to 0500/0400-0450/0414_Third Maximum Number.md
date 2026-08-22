# Third Maximum Number

## Problem

Given an integer array `nums`, return the **third distinct maximum number** in the array.

If the third maximum does not exist, return the **maximum number**.

Example:

```text
nums = [3, 2, 1]
Output: 1

nums = [1, 2]
Output: 2

nums = [2, 2, 3, 1]
Output: 1
```

---

## Approach 1: Track Three Maximums

### Idea

Maintain three variables:

* `first` → largest distinct number
* `second` → second largest distinct number
* `third` → third largest distinct number

For every number:

* Ignore it if it is already one of the three maximums.
* Update `first`, `second`, and `third` accordingly.

Use `long long` with `LLONG_MIN` as a marker so that `INT_MIN` can also be a valid array value.

### Dry Run

```text
nums = [2, 2, 3, 1]

first = -∞
second = -∞
third = -∞

2 → first = 2
2 → duplicate, ignore
3 → first = 3, second = 2
1 → third = 1

third = 1
```

### Algorithm

1. Initialize `first`, `second`, and `third` to `LLONG_MIN`.
2. Traverse every number in `nums`.
3. If the number equals `first`, `second`, or `third`, skip it.
4. If it is greater than `first`:

   * `third = second`
   * `second = first`
   * `first = num`
5. Else if it is greater than `second`:

   * `third = second`
   * `second = num`
6. Else if it is greater than `third`:

   * `third = num`
7. If `third` was never updated, return `first`.
8. Otherwise return `third`.

### Complexity

* Time: `O(n)`
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    int thirdMax(vector<int>& nums) {
        long long first = LLONG_MIN;
        long long second = LLONG_MIN;
        long long third = LLONG_MIN;

        for (int num : nums) {
            if (num == first || num == second || num == third) {
                continue;
            }

            if (num > first) {
                third = second;
                second = first;
                first = num;
            }
            else if (num > second) {
                third = second;
                second = num;
            }
            else if (num > third) {
                third = num;
            }
        }

        return third == LLONG_MIN ? first : third;
    }
};
```

### Notes / Tips

* The word **distinct** is important: `[3, 3, 2, 1]` has three distinct maximums `3, 2, 1`.
* Duplicates must not update the three maximums.
* Do not use `INT_MIN` as the "not found" marker because `INT_MIN` itself can appear in the array.
* This is essentially a **top-3 distinct elements** problem.
* No sorting is required.

### Key Template

```text
first = -∞
second = -∞
third = -∞

for each num:
    if num is already first/second/third:
        continue

    if num > first:
        third = second
        second = first
        first = num

    else if num > second:
        third = second
        second = num

    else if num > third:
        third = num

return third if it exists, otherwise first
```
