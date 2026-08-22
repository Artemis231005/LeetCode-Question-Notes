# Max Consecutive Ones

## Problem

Given a binary array `nums`, return the maximum number of consecutive `1`s in the array.

Example:

```text
nums = [1,1,0,1,1,1]
Output: 3
```

---

## Approach 1: Counting Consecutive Ones

### Idea

Maintain:

* `count` → current number of consecutive `1`s
* `maxCount` → maximum consecutive `1`s found so far

Whenever we see:

* `1` → increment `count`
* `0` → reset `count` to `0`

Update `maxCount` after each element.

### Dry Run

```text
nums = [1,1,0,1,1,1]

1 → count = 1, max = 1
1 → count = 2, max = 2
0 → count = 0
1 → count = 1, max = 2
1 → count = 2, max = 2
1 → count = 3, max = 3

Answer = 3
```

### Algorithm

1. Initialize `count = 0` and `maxCount = 0`.
2. Traverse every element in `nums`.
3. If the element is `1`, increment `count`.
4. Otherwise reset `count = 0`.
5. Update `maxCount = max(maxCount, count)`.
6. Return `maxCount`.

### Complexity

* Time: `O(n)`
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    int findMaxConsecutiveOnes(vector<int>& nums) {
        int count = 0;
        int maxCount = 0;

        for (int num : nums) {
            if (num == 1) {
                count++;
                maxCount = max(maxCount, count);
            }
            else {
                count = 0;
            }
        }

        return maxCount;
    }
};
```

### Notes / Tips

* This is a simple **one-pass counting** problem.
* `0` acts as a separator between groups of consecutive `1`s.
* Reset the current count whenever `0` appears.
* Always maintain a separate maximum because the final group may not end with `0`.
* Pattern: **Count current streak + maintain maximum.**

### Key Template

```text
count = 0
maxCount = 0

for each element:
    if element satisfies condition:
        count++
        maxCount = max(maxCount, count)
    else:
        count = 0

return maxCount
```
