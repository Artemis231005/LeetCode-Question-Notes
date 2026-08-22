# 342. Power of Four

## Metadata

* **Topic:** Math, Bit Manipulation
* **Difficulty:** Easy
* **Key Pattern:** Power of Two + Bit Position
* **Key Template:** Check power of two, then verify that the set bit is at an even position
* **Goal:** Determine whether a given integer `n` is a power of `4`.

---

## Approach 1: Bit Manipulation

### Idea

Every power of `4` is also a power of `2`.

```text
4⁰ = 1  = 2⁰
4¹ = 4  = 2²
4² = 16 = 2⁴
4³ = 64 = 2⁶
```

So first check whether `n` is a power of `2`.

A power of `2` has exactly one set bit:

```cpp
n & (n - 1) == 0
```

Then we need to make sure that this set bit is at an **even position**.

We can use the mask:

```text
01010101010101010101010101010101
```

which has `1`s at all even bit positions.

### Dry Run

For `n = 16`:

```text
16 = 00010000
```

Power of two:

```text
16 & 15 = 0
```

Set bit is at position `4`, which is even.

Therefore:

```text
true
```

For `n = 8`:

```text
8 = 00001000
```

It is a power of two, but its set bit is at position `3`, which is odd.

Therefore:

```text
false
```

### Algorithm

1. Check that `n > 0`.
2. Check whether `n` is a power of `2` using:

   ```cpp
   (n & (n - 1)) == 0
   ```
3. Check whether its set bit is located at an even position using the mask:

   ```cpp
   0x55555555
   ```
4. Return `true` only if both conditions are satisfied.

### Complexity

* **Time:** `O(1)`
* **Space:** `O(1)`

### Notes / Tips

* Every power of `4` is a power of `2`, but every power of `2` is **not** a power of `4`.
* `n & (n - 1)` removes the rightmost set bit.
* A power of `2` has exactly one set bit.
* `0x55555555` has `1`s at even bit positions:

  ```text
  01010101010101010101010101010101
  ```
* The mask distinguishes powers of `4` from powers of `2` such as `2, 8, 32, 128`.
* This is a classic **bit manipulation** problem.

### Code

```cpp
class Solution {
public:
    bool isPowerOfFour(int n) {
        return n > 0 &&
               (n & (n - 1)) == 0 &&
               (n & 0x55555555) != 0;
    }
};
```

---

## Approach 2: Repeated Division

### Idea

A number is a power of `4` if we can repeatedly divide it by `4` until it becomes `1`.

For example:

```text
64 → 16 → 4 → 1
```

If at any point it is not divisible by `4`, it cannot be a power of `4`.

### Dry Run

For `n = 64`:

```text
64 % 4 == 0 → 64 / 4 = 16
16 % 4 == 0 → 16 / 4 = 4
4  % 4 == 0 → 4 / 4 = 1
```

Now:

```text
n == 1
```

So the answer is `true`.

For `n = 32`:

```text
32 / 4 = 8
8 / 4 = 2
```

Now `2` is not divisible by `4`.

Therefore:

```text
false
```

### Algorithm

1. If `n <= 0`, return `false`.
2. While `n` is divisible by `4`, divide it by `4`.
3. Check whether the final value is `1`.
4. Return the result.

### Complexity

* **Time:** `O(log₄ n)`
* **Space:** `O(1)`

### Notes / Tips

* This approach is easier to understand than the bit-mask approach.
* `1 = 4⁰`, so `1` is a valid power of `4`.
* Avoid floating-point logarithms because of precision issues.
* Use this approach when clarity is more important than demonstrating bit manipulation.

### Code

```cpp
class Solution {
public:
    bool isPowerOfFour(int n) {
        if (n <= 0) {
            return false;
        }

        while (n % 4 == 0) {
            n /= 4;
        }

        return n == 1;
    }
};
```

---

## Key Template

```cpp
// Power of 2
(n > 0 && (n & (n - 1)) == 0)

// Power of 4
(n > 0 &&
 (n & (n - 1)) == 0 &&
 (n & 0x55555555) != 0)
```

### Pattern to Remember

```text
Power of 4
    ↓
Must be a power of 2
    ↓
Exactly one set bit
    ↓
Set bit must be at an even position
    ↓
Use 0x55555555 mask
```
