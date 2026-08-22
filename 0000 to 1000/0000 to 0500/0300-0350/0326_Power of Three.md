# 326. Power of Three

## Metadata

* **Topic:** Math, Recursion
* **Difficulty:** Easy
* **Key Pattern:** Repeated Division
* **Key Template:** Keep dividing by the base while divisible, then check whether the result becomes `1`
* **Goal:** Determine whether a number is a power of `3`.

---

## Approach 1: Repeated Division

### Idea

A number is a power of `3` if it can be repeatedly divided by `3` until it becomes exactly `1`.

For example:

```text
27 → 9 → 3 → 1
```

Since `27 = 3³`, it is a power of `3`.

### Dry Run

For `n = 27`:

```text
27 % 3 == 0 → 27 / 3 = 9
9  % 3 == 0 → 9 / 3  = 3
3  % 3 == 0 → 3 / 3  = 1

n == 1 → true
```

For `n = 45`:

```text
45 / 3 = 15
15 / 3 = 5

5 % 3 != 0
```

So `45` is not a power of `3`.

### Algorithm

1. If `n <= 0`, return `false`.
2. While `n` is divisible by `3`, divide it by `3`.
3. After the loop, check whether `n == 1`.
4. If yes, return `true`; otherwise return `false`.

### Complexity

* **Time:** `O(log₃ n)`
* **Space:** `O(1)`

### Notes / Tips

* `1 = 3⁰`, so `1` is considered a power of `3`.
* Always handle `n <= 0`.
* Avoid using logarithms because floating-point precision can cause errors.
* This pattern can be generalized to checking powers of other bases.

### Code

```cpp
class Solution {
public:
    bool isPowerOfThree(int n) {
        if (n <= 0) {
            return false;
        }

        while (n % 3 == 0) {
            n /= 3;
        }

        return n == 1;
    }
};
```

---

## Approach 2: Mathematical Property

### Idea

The largest power of `3` that fits in a signed 32-bit integer is:

```text
3^19 = 1162261467
```

Every positive power of `3` divides this number exactly.

Therefore, `n` is a power of `3` if:

```text
1162261467 % n == 0
```

### Dry Run

For `n = 27`:

```text
1162261467 % 27 == 0
```

So:

```text
true
```

For `n = 45`:

```text
1162261467 % 45 != 0
```

So:

```text
false
```

### Algorithm

1. Check that `n > 0`.
2. Take the largest power of `3` within the integer range.
3. Check whether it is divisible by `n`.
4. Return the result.

### Complexity

* **Time:** `O(1)`
* **Space:** `O(1)`

### Notes / Tips

* This solution relies on the given 32-bit integer constraint.
* It is mathematically clever but less general.
* The repeated-division approach is easier to remember and explain in interviews.

### Code

```cpp
class Solution {
public:
    bool isPowerOfThree(int n) {
        return n > 0 && 1162261467 % n == 0;
    }
};
```

---

## Key Template

```cpp
if (n <= 0) {
    return false;
}

while (n % base == 0) {
    n /= base;
}

return n == 1;
```

### Pattern to Remember

```text
Power of X?
    ↓
Repeatedly divide by X
    ↓
If exactly 1 remains → Power of X
Otherwise → Not a power
```

