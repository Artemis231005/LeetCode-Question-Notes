# 374. Guess Number Higher or Lower

## Metadata

* **Topic:** Binary Search
* **Difficulty:** Easy
* **Key Pattern:** Binary Search on Answer
* **Key Template:** Use the feedback from `guess(mid)` to eliminate half of the search space
* **Goal:** Find the number picked by the API within the range `[1, n]`.

---

## Approach 1: Binary Search

### Idea

The API `guess(num)` tells us whether our guess is:

* `0` → correct number
* `-1` → picked number is smaller
* `1` → picked number is larger

Since the possible answer lies in a sorted range `[1, n]`, we can use binary search.

For every `mid`:

```text
guess(mid) == 0
→ Found the answer

guess(mid) == -1
→ Answer is smaller
→ Search left half

guess(mid) == 1
→ Answer is larger
→ Search right half
```

### Dry Run

Suppose:

```text
n = 10
picked = 6
```

Search range:

```text
[1, 10]
```

First:

```text
mid = 5
guess(5) = 1
```

Picked number is greater:

```text
[6, 10]
```

Next:

```text
mid = 8
guess(8) = -1
```

Picked number is smaller:

```text
[6, 7]
```

Next:

```text
mid = 6
guess(6) = 0
```

Answer:

```text
6
```

### Algorithm

1. Initialize `left = 1` and `right = n`.
2. While `left <= right`:

   * Calculate `mid`.
   * Call `guess(mid)`.
   * If result is `0`, return `mid`.
   * If result is `-1`, move `right` to `mid - 1`.
   * If result is `1`, move `left` to `mid + 1`.
3. Return the found number.

### Complexity

* **Time:** `O(log n)`
* **Space:** `O(1)`

### Notes / Tips

* This is a direct application of **Binary Search on Answer**.
* The `guess()` API tells us which half can be eliminated.
* Use:

  ```cpp
  mid = left + (right - left) / 2;
  ```

  instead of `(left + right) / 2` to avoid overflow.
* The search space is always `[1, n]`.
* Remember the meaning of the API:

  ```text
  -1 → mid is too high
   1 → mid is too low
   0 → correct
  ```

### Code

```cpp
/**
 * Forward declaration of guess API.
 * @param num   your guess
 * @return      -1 if num is higher than the picked number
 *               1 if num is lower than the picked number
 *               otherwise return 0
 * int guess(int num);
 */

class Solution {
public:
    int guessNumber(int n) {
        long long left = 1;
        long long right = n;

        while (left <= right) {
            long long mid = left + (right - left) / 2;

            int result = guess(mid);

            if (result == 0) {
                return mid;
            }
            else if (result == -1) {
                right = mid - 1;
            }
            else {
                left = mid + 1;
            }
        }

        return -1;
    }
};
```

---

## Key Template

```cpp
long long left = 1;
long long right = n;

while (left <= right) {
    long long mid = left + (right - left) / 2;

    int result = guess(mid);

    if (result == 0) {
        return mid;
    }
    else if (result < 0) {
        right = mid - 1;
    }
    else {
        left = mid + 1;
    }
}
```

### Pattern to Remember

```text
Search space: [1, n]

guess(mid)
    ↓
    0  → Answer found
   -1  → Too high → Go left
    1  → Too low  → Go right
```
