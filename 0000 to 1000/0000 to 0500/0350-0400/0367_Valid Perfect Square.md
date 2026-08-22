# 367. Valid Perfect Square

## Metadata

* **Topic:** Math, Binary Search
* **Difficulty:** Easy
* **Key Pattern:** Binary Search on Answer
* **Key Template:** Search for `mid * mid == num`
* **Goal:** Determine whether a given positive integer is a perfect square without using built-in square-root functions.

---

## Approach 1: Binary Search

### Idea

For a positive number `num`, its square root lies between `1` and `num`.

Instead of checking every number, use binary search to find a number whose square equals `num`.

For a midpoint `mid`:

* If `mid * mid == num` → perfect square.
* If `mid * mid < num` → answer must be on the right.
* If `mid * mid > num` → answer must be on the left.

### Dry Run

```text
num = 16
```

Search range:

```text
left = 1, right = 16
```

Take `mid = 8`:

```text
8 × 8 = 64 > 16
→ search left
```

Take `mid = 4`:

```text
4 × 4 = 16
→ found
```

Answer:

```text
true
```

For `num = 14`:

```text
mid = 7 → 49 > 14
mid = 3 → 9 < 14
mid = 5 → 25 > 14
mid = 4 → 16 > 14
```

No value satisfies `mid * mid == 14`.

Answer:

```text
false
```

### Algorithm

1. Set `left = 1` and `right = num`.
2. While `left <= right`:

   * Calculate `mid`.
   * If `mid * mid == num`, return `true`.
   * If `mid * mid < num`, search the right half.
   * Otherwise, search the left half.
3. If the loop finishes, return `false`.

### Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

### Notes / Tips

* This is a classic **Binary Search on Answer** problem.
* Do not use `sqrt()` because the problem explicitly asks you not to use built-in square-root functions.
* Use `long long` for `mid * mid` to avoid integer overflow.
* The key comparison is:

  ```text
  mid² vs num
  ```
* Binary search is much faster than checking every number from `1` to `num`.

### Code

```cpp
class Solution {
public:
    bool isPerfectSquare(int num) {
        long long left = 1;
        long long right = num;

        while (left <= right) {
            long long mid = left + (right - left) / 2;
            long long square = mid * mid;

            if (square == num) {
                return true;
            }
            else if (square < num) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }

        return false;
    }
};
```

---

## Approach 2: Binary Search with Reduced Range

### Idea

For `num > 1`, the square root of `num` can never be greater than `num / 2`.

For example:

```text
num = 16
sqrt(16) = 4
```

So we can search between `2` and `num / 2`.

Handle `num == 1` separately.

### Dry Run

For:

```text
num = 25
```

Search:

```text
left = 2
right = 12
```

Eventually:

```text
mid = 5
5 × 5 = 25
```

Therefore:

```text
true
```

### Algorithm

1. If `num == 1`, return `true`.
2. Set `left = 2`, `right = num / 2`.
3. Apply binary search.
4. Compare `mid * mid` with `num`.
5. Return `true` if an exact square is found.
6. Otherwise return `false`.

### Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

### Notes / Tips

* Reducing the search space is an optional optimization.
* The first approach is simpler and works directly for all positive integers.
* Always use `long long` when calculating `mid * mid`.

### Code

```cpp
class Solution {
public:
    bool isPerfectSquare(int num) {
        if (num == 1) {
            return true;
        }

        long long left = 2;
        long long right = num / 2;

        while (left <= right) {
            long long mid = left + (right - left) / 2;
            long long square = mid * mid;

            if (square == num) {
                return true;
            }
            else if (square < num) {
                left = mid + 1;
            }
            else {
                right = mid - 1;
            }
        }

        return false;
    }
};
```

---

## Key Template

```cpp
long long left = 1;
long long right = num;

while (left <= right) {
    long long mid = left + (right - left) / 2;
    long long value = mid * mid;

    if (value == num) {
        return true;
    }
    else if (value < num) {
        left = mid + 1;
    }
    else {
        right = mid - 1;
    }
}

return false;
```

### Pattern to Remember

```text
Need to find x such that:

f(x) == target

If f(x) < target:
    move right

If f(x) > target:
    move left

Here:

f(x) = x²
```
