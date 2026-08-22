# N-Repeated Element in Size 2N Array

## Problem

You are given an integer array `nums` containing `2N` elements.

Exactly one element is repeated **N times**, while every other element appears exactly once.

Return the element that is repeated `N` times.

Example:

```text
nums = [1,2,3,3]
Output = 3
```

---

## Approach 1: Hash Set

### Idea

Traverse the array and keep track of elements that have already been seen.

* If the current element is already in the set, it is the repeated element.
* Otherwise, add it to the set.

Since only one element is repeated, the **first duplicate** we encounter is the answer.

### Dry Run

```text
nums = [1,2,3,3]

1 → not seen → add
2 → not seen → add
3 → not seen → add
3 → already seen → answer = 3
```

### Algorithm

1. Create an empty set.
2. Traverse every element of `nums`.
3. If the element already exists in the set, return it.
4. Otherwise, insert it into the set.
5. Return the repeated element.

### Complexity

* Time: `O(n)`
* Space: `O(n)`

### Code

```cpp
class Solution {
public:
    int repeatedNTimes(vector<int>& nums) {
        unordered_set<int> seen;

        for (int num : nums) {
            if (seen.count(num)) {
                return num;
            }

            seen.insert(num);
        }

        return -1;
    }
};
```

### Notes / Tips

* This is a simple **Hashing** problem.
* The first duplicate is guaranteed to be the answer.
* `unordered_set` provides average `O(1)` insertion and lookup.
* Since only one value can occur more than once, we don't need to count frequencies.

### Key Template

```text
set = empty

for each num:
    if num is already in set:
        return num

    add num to set
```
