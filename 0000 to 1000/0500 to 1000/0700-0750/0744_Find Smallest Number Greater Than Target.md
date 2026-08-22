# Find Smallest Letter Greater Than Target

## Problem

Given a sorted array of characters `letters` and a character `target`, return the **smallest character that is lexicographically greater than `target`**.

The array is sorted in non-decreasing order.

If no character is greater than `target`, return the **first character** in the array.

Example:

```text
letters = ['c','f','j']
target = 'd'

Output = 'f'
```

---

## Approach 1: Binary Search

### Idea

We need to find the **first character strictly greater than `target`**.

This is essentially a **lower bound / upper bound** type binary search.

For every `mid`:

* If `letters[mid] <= target`, it cannot be the answer, so move right.
* If `letters[mid] > target`, it could be the answer, so store it and continue searching left.

If no such character exists, return `letters[0]`.

### Dry Run

```text
letters = ['c','f','j']
target = 'd'

left = 0, right = 2

mid = 1
letters[1] = 'f'

'f' > 'd'
→ possible answer
→ search left

right = 0

mid = 0
letters[0] = 'c'

'c' <= 'd'
→ search right

left = 1

Answer = 'f'
```

### Algorithm

1. Initialize `left = 0` and `right = letters.size() - 1`.
2. While `left <= right`:

   * Calculate `mid`.
   * If `letters[mid] <= target`, move `left = mid + 1`.
   * Otherwise:

     * Store `letters[mid]` as a possible answer.
     * Move `right = mid - 1`.
3. If `left` reaches `letters.size()`, no greater character exists, so return `letters[0]`.
4. Otherwise return `letters[left]`.

### Complexity

* Time: `O(log n)`
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    char nextGreatestLetter(vector<char>& letters, char target) {
        int left = 0;
        int right = letters.size() - 1;

        while (left <= right) {
            int mid = left + (right - left) / 2;

            if (letters[mid] <= target) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }

        return letters[left % letters.size()];
    }
};
```

### Notes / Tips

* We need a character **strictly greater** than `target`, so use `<=` when eliminating the left half.
* This is essentially finding the **first element greater than target**.
* If the target is greater than or equal to every character, the answer **wraps around** to the first character.
* The expression `left % letters.size()` handles this wrap-around neatly.
* Example:

  * `['c','f','j']`, target `'j'` → `'c'`
  * `['c','f','j']`, target `'a'` → `'c'`

### Key Template

```text
left = 0
right = n - 1

while left <= right:
    mid = left + (right - left) / 2

    if arr[mid] <= target:
        left = mid + 1
    else:
        right = mid - 1

return arr[left % n]
```
