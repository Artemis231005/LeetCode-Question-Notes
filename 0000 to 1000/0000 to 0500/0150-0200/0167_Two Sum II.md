# LeetCode 167 — Two Sum II - Input Array Is Sorted

## Metadata
* **LeetCode:** 167
* **Problem:** Two Sum II - Input Array Is Sorted
* **Difficulty:** Medium
* **Topics:** Array, Two Pointers, Binary Search
* **Pattern:** Opposite-Direction Two Pointers
* **Key Technique:** Left and Right Pointers
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Problem Statement
Find two numbers in a **sorted array** whose sum equals the target and return their **1-indexed positions**.

---

## Idea
Since the array is already **sorted**, use two pointers:
```text
left  → beginning
right → end
```

Calculate:
```text
sum = numbers[left] + numbers[right]
```

Then:
* If `sum == target` → answer found.
* If `sum < target` → increase `left` to make the sum larger.
* If `sum > target` → decrease `right` to make the sum smaller.

This works because the array is sorted.

---

## Dry Run
```text
numbers = [2, 7, 11, 15]
target = 9
```

Initially:
```text
2  7  11  15
L           R
```

### Step 1
```text
2 + 15 = 17
```

`17 > 9`, so decrease `right`.

```text
2  7  11  15
L       R
```

### Step 2
```text
2 + 11 = 13
```

`13 > 9`, so decrease `right`.

```text
2  7  11  15
L   R
```

### Step 3
```text
2 + 7 = 9
```

Found.

Positions are:
```text
[1, 2]
```

---

## Algorithm

1. Set `left = 0`.
2. Set `right = n - 1`.
3. While `left < right`:

   * Calculate `sum = numbers[left] + numbers[right]`.
   * If `sum == target`, return `{left + 1, right + 1}`.
   * If `sum < target`, increment `left`.
   * Otherwise, decrement `right`.
4. Return an empty vector if no pair is found.

---

## Why Does Two Pointers Work?
The sorted order gives us a way to **eliminate possibilities**.

Suppose:
```text
[1, 3, 5, 7, 9]
 L           R
```

If:
```text
1 + 9 < target
```

then `1` cannot form the target with **any element ≤ 9**, because those elements would produce an even smaller sum.

Therefore, we can safely eliminate `1`:
```text
L++
```

Similarly, if:
```text
1 + 9 > target
```

then `9` cannot form the target with **any element ≥ 1**, because those elements would produce an even larger sum.

Therefore:
```text
R--
```

This is the key reason the sorted property allows us to solve the problem in `O(n)` time with `O(1)` space.

---

## Complexity
* **TC:** `O(n)`

  * `left` only moves forward.
  * `right` only moves backward.
  * Together they make at most `n` pointer movements.
* **SC:** `O(1)`
  * Only two pointers are used.

---

## Code

```cpp
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int left = 0;
        int right = numbers.size() - 1;

        while (left < right) {
            int sum = numbers[left] + numbers[right];

            if (sum == target) {
                return {left + 1, right + 1};
            }
            else if (sum < target) {
                left++;
            }
            else {
                right--;
            }
        }

        return {};
    }
};
```

---

## Notes / Tips

* The **sorted array** is the entire reason two pointers work.
* This is the classic **opposite-direction two-pointer** pattern.
* `left++` makes the sum **larger**.
* `right--` makes the sum **smaller**.
* The problem asks for **1-indexed** positions, so return:

  ```cpp
  left + 1
  right + 1
  ```
* Unlike LeetCode 1, no hash map is required.
* Both LeetCode 1 and 167 have `O(n)` time, but:

  * **Two Sum:** `O(n)` extra space.
  * **Two Sum II:** `O(1)` extra space.

---

## Key Template
```text
left = 0
right = n - 1

while (left < right):

    sum = arr[left] + arr[right]

    if sum == target:
        found

    else if sum < target:
        left++

    else:
        right--
```
