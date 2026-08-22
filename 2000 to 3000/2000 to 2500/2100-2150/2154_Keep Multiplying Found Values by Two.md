# Keep Multiplying Found Values by Two

## Problem

Given an integer array `nums` and an integer `original`.

While `original` exists in `nums`, multiply `original` by `2`.

Return the final value of `original`.

Example:

```text
nums = [5,3,6,1,12]
original = 3

3 exists → 6
6 exists → 12
12 exists → 24
24 does not exist

Output = 24
```

---

## Approach 1: Hash Set

### Idea

Store all elements of `nums` in a hash set for fast lookup.

Then repeatedly:

* Check whether `original` exists.
* If it exists, multiply it by `2`.
* Otherwise, stop.

Because we only need to check whether a value exists, a set is sufficient; frequency does not matter.

### Dry Run

```text
nums = [5,3,6,1,12]
original = 3

set = {1,3,5,6,12}

3 exists → original = 6
6 exists → original = 12
12 exists → original = 24
24 not found

Answer = 24
```

### Algorithm

1. Insert every element of `nums` into an `unordered_set`.
2. While `original` exists in the set:

   * Multiply `original` by `2`.
3. Return `original`.

### Complexity

* Time: `O(n)` average
* Space: `O(n)`

### Code

```cpp
class Solution {
public:
    int findFinalValue(vector<int>& nums, int original) {
        unordered_set<int> seen(nums.begin(), nums.end());

        while (seen.count(original)) {
            original *= 2;
        }

        return original;
    }
};
```

### Notes / Tips

* This is a **Hashing + Simulation** problem.
* We only care about **existence**, so use a set instead of a frequency map.
* The important pattern is:

  ```text
  while current value exists:
      update current value
  ```
* Sorting is also possible, but hashing gives direct average `O(1)` lookup.
* The array does not need to be modified.

### Key Template

```text
set = all elements

while current exists in set:
    current *= 2

return current
```
