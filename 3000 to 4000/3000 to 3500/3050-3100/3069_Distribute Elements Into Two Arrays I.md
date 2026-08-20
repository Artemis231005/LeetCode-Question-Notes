# LeetCode 3069 — Distribute Elements Into Two Arrays I

## Metadata

* **LeetCode:** 3069
* **Problem:** Distribute Elements Into Two Arrays I
* **Difficulty:** Easy
* **Topics:** Array, Simulation
* **Pattern:** Sequential Distribution
* **Key Technique:** Compare the last elements of the two arrays
* **Key Pattern:** Greedy / Simulation
* **Optimal Complexity:** `O(n)` time, `O(n)` space

---

## Problem

You are given a **1-indexed array** `nums` of distinct integers.

You need to distribute all elements of `nums` between two arrays `arr1` and `arr2` using `n` operations.

Rules:

1. In the first operation, append `nums[1]` to `arr1`.
2. In the second operation, append `nums[2]` to `arr2`.
3. For every operation from `3` to `n`:

   * If the last element of `arr1` is **greater than** the last element of `arr2`, append the current element to `arr1`.
   * Otherwise, append it to `arr2`.
4. Finally, return:

   ```text
   arr1 + arr2
   ```

---

## Approach — Simulation

### Idea

The problem directly tells us what to do.

* First element → `arr1`
* Second element → `arr2`
* For every remaining element:

  * Compare the **last elements** of `arr1` and `arr2`.
  * Put the current element into the array whose last element is smaller.
* Concatenate `arr1` and `arr2`.

Since the elements are distinct, there is no equality case between the last elements.

### Dry Run

Consider:

```text
nums = [2, 1, 3, 4, 5]
```

#### Initial

```text
arr1 = [2]
arr2 = [1]
```

#### Element `3`

Compare:

```text
last(arr1) = 2
last(arr2) = 1
```

Since:

```text
2 > 1
```

append `3` to `arr1`.

```text
arr1 = [2, 3]
arr2 = [1]
```

#### Element `4`

Compare:

```text
3 > 1
```

So append `4` to `arr1`.

```text
arr1 = [2, 3, 4]
arr2 = [1]
```

#### Element `5`

Compare:

```text
4 > 1
```

So append `5` to `arr1`.

```text
arr1 = [2, 3, 4, 5]
arr2 = [1]
```

Final answer:

```text
[2, 3, 4, 5, 1]
```

### Algorithm

1. Create empty arrays `arr1` and `arr2`.
2. Put `nums[0]` into `arr1`.
3. Put `nums[1]` into `arr2`.
4. For every `i` from `2` to `n - 1`:

   * If `arr1.back() > arr2.back()`:

     * Append `nums[i]` to `arr1`.
   * Otherwise:

     * Append `nums[i]` to `arr2`.
5. Append all elements of `arr2` to `arr1`.
6. Return `arr1`.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(n)` for the two arrays and final result.

### Notes / Tips

* This is primarily a **simulation** problem.
* Don't compare the maximum elements of the arrays; compare only their **last elements**.
* `back()` gives the last element of a vector:

  ```cpp
  arr1.back()
  arr2.back()
  ```
* The final answer is **`arr1` followed by `arr2`**, not an interleaving of the two arrays.
* The first two elements must be handled separately because `arr1` and `arr2` are initially empty.
* The most important condition is:

  ```cpp
  if (arr1.back() > arr2.back())
  ```

### Code

```cpp
class Solution {
public:
    vector<int> resultArray(vector<int>& nums) {
        vector<int> arr1;
        vector<int> arr2;

        arr1.push_back(nums[0]);
        arr2.push_back(nums[1]);

        for (int i = 2; i < nums.size(); i++) {
            if (arr1.back() > arr2.back()) {
                arr1.push_back(nums[i]);
            }
            else {
                arr2.push_back(nums[i]);
            }
        }

        for (int num : arr2) {
            arr1.push_back(num);
        }

        return arr1;
    }
};
```

---

## Key Takeaway

The entire problem can be reduced to:

```text
First → arr1
Second → arr2

For every remaining element:
    if last(arr1) > last(arr2)
        → arr1
    else
        → arr2

Answer = arr1 + arr2
```

**Pattern:**

> When the problem gives a sequence of explicit rules for placing elements, simulate those rules directly.
