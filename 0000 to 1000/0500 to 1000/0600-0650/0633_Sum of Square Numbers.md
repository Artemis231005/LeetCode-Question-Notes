# Sum of Square Numbers

## Problem

Given a non-negative integer `c`, determine whether there exist two integers `a` and `b` such that:

```text
a² + b² = c
```

Return `true` if such integers exist, otherwise return `false`.

Example:

```text
c = 5
Output = true

5 = 1² + 2²
```

---

## Approach 1: Two Pointers

### Idea

Since:

```text
a² + b² = c
```

Both `a` and `b` can be between `0` and `sqrt(c)`.

Use two pointers:

* `left = 0`
* `right = sqrt(c)`

Calculate:

```text
sum = left² + right²
```

* If `sum == c` → found the answer.
* If `sum < c` → increase `left` to make the sum larger.
* If `sum > c` → decrease `right` to make the sum smaller.

### Dry Run

```text
c = 5

left = 0
right = 2

0² + 2² = 4 < 5
→ left++

1² + 2² = 5
→ found

Answer = true
```

### Algorithm

1. Set `left = 0`.
2. Set `right = sqrt(c)`.
3. While `left <= right`:

   * Calculate `sum = left² + right²`.
   * If `sum == c`, return `true`.
   * If `sum < c`, increment `left`.
   * Otherwise decrement `right`.
4. Return `false`.

### Complexity

* Time: `O(sqrt(c))`
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    bool judgeSquareSum(int c) {
        long long left = 0;
        long long right = sqrt(c);

        while (left <= right) {
            long long sum = left * left + right * right;

            if (sum == c) {
                return true;
            }
            else if (sum < c) {
                left++;
            }
            else {
                right--;
            }
        }

        return false;
    }
};
```

### Notes / Tips

* The search space is only `0` to `sqrt(c)`.
* Use `long long` for `left * left` and `right * right` to avoid integer overflow.
* This is a **two-pointer problem on a mathematical search space**.
* If the sum is too small, increase the smaller value.
* If the sum is too large, decrease the larger value.
* Don't use a nested loop; that would take `O(c)` time.

### Key Template

```text
left = 0
right = sqrt(c)

while left <= right:
    sum = left² + right²

    if sum == c:
        return true

    if sum < c:
        left++
    else:
        right--

return false
```
