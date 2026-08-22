# Smallest Missing Integer Greater Than Sequential Prefix Sum

## Problem

Given an array `nums`, first find the **longest sequential prefix** starting from `nums[0]`.

A sequential prefix means:

```text
nums[i] = nums[i - 1] + 1
```

for consecutive elements.

Calculate the sum of this sequential prefix.

Then return the **smallest positive integer greater than or equal to this sum that does not exist in `nums`**.

Example:

```text
nums = [1,2,3,2,5]

Sequential prefix = [1,2,3]
Sum = 6

6 is not present in nums

Output = 6
```

---

## Approach 1: Hash Set + Sequential Prefix

### Idea

First calculate the sum of the sequential prefix.

Then put all elements into a hash set so we can quickly check whether the sum exists.

Start from the prefix sum:

* If the value exists in the set, increment it.
* Continue until we find a value that does not exist.

That value is the answer.

### Dry Run

```text
nums = [1,2,3,2,5]

Sequential prefix:
1 → 2 → 3
2 breaks the sequence

sum = 1 + 2 + 3 = 6

set = {1,2,3,5}

6 not present
→ answer = 6
```

Another example:

```text
nums = [3,4,5,6,7]

Sequential prefix = [3,4,5,6,7]
sum = 25

25 not present
→ answer = 25
```

### Algorithm

1. Initialize `sum = nums[0]`.
2. Traverse the array from index `1`.
3. Continue adding elements while:

   ```text
   nums[i] == nums[i - 1] + 1
   ```
4. Stop when the sequential prefix ends.
5. Insert all elements into an `unordered_set`.
6. While `sum` exists in the set:

   * Increment `sum`.
7. Return `sum`.

### Complexity

* Time: `O(n)` average.
* Space: `O(n)`.

### Code

```cpp
class Solution {
public:
    int missingInteger(vector<int>& nums) {
        int sum = nums[0];

        for (int i = 1; i < nums.size(); i++) {
            if (nums[i] == nums[i - 1] + 1) {
                sum += nums[i];
            }
            else {
                break;
            }
        }

        unordered_set<int> seen(nums.begin(), nums.end());

        while (seen.count(sum)) {
            sum++;
        }

        return sum;
    }
};
```

### Notes / Tips

* There are **two separate tasks**:

  1. Find the sequential prefix sum.
  2. Find the smallest missing value starting from that sum.
* The prefix is based on **consecutive values**, not simply the first increasing portion.
* Once the sequential pattern breaks, stop calculating the prefix.
* A hash set gives average `O(1)` existence checks.
* The answer starts at the prefix sum and only increases when that value already exists.

### Key Template

```text
sum = nums[0]

for i from 1:
    if nums[i] == nums[i - 1] + 1:
        sum += nums[i]
    else:
        break

set = all elements

while sum exists in set:
    sum++

return sum
```
