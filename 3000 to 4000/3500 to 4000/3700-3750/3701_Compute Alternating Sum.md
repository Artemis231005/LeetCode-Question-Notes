# Compute Alternating Sum

## Problem

Given an integer array `nums`, return its **alternating sum**.

The alternating sum is calculated by:

```text
nums[0] - nums[1] + nums[2] - nums[3] + ...
```

So:

* Elements at **even indices** are added.
* Elements at **odd indices** are subtracted.

Example:

```text
nums = [1,3,5,7]

1 - 3 + 5 - 7 = -4

Output = -4
```

---

## Approach 1: Simulation

### Idea

Traverse the array once.

For every index:

* If the index is even → add `nums[i]`.
* If the index is odd → subtract `nums[i]`.

This directly follows the definition of alternating sum.

### Dry Run

```text
nums = [1,3,5,7]

i = 0 → +1 → sum = 1
i = 1 → -3 → sum = -2
i = 2 → +5 → sum = 3
i = 3 → -7 → sum = -4

Answer = -4
```

### Algorithm

1. Initialize `sum = 0`.
2. Traverse the array using index `i`.
3. If `i` is even, add `nums[i]` to `sum`.
4. Otherwise, subtract `nums[i]`.
5. Return `sum`.

### Complexity

* Time: `O(n)`
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    int alternatingSum(vector<int>& nums) {
        int sum = 0;

        for (int i = 0; i < nums.size(); i++) {
            if (i % 2 == 0) {
                sum += nums[i];
            }
            else {
                sum -= nums[i];
            }
        }

        return sum;
    }
};
```

### Notes / Tips

* This is a straightforward **Array + Simulation** problem.
* The important thing is that the sign depends on the **index**, not the value.
* Index pattern:

  ```text
  0 → +
  1 → -
  2 → +
  3 → -
  ```
* A sign-toggle variable can also be used instead of checking `i % 2`.

### Key Template

```text
sum = 0

for i from 0 to n-1:
    if i is even:
        sum += nums[i]
    else:
        sum -= nums[i]

return sum
```
