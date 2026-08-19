# LeetCode 29 — Divide Two Integers

## Metadata

* **LeetCode:** 29
* **Problem:** Divide Two Integers
* **Difficulty:** Medium
* **Topics:** Math, Bit Manipulation
* **Pattern:** Repeated Subtraction / Binary Decomposition
* **Key Technique:** Bit Manipulation
* **Key Pattern:** Exponential Doubling
* **Optimal Complexity:** `O(log n)` time, `O(1)` space

---

## Problem

Given two integers `dividend` and `divisor`, divide `dividend` by `divisor` **without using multiplication, division, or modulo operators**.

Return the quotient truncated toward zero.

### Important Rules

* Do not use `*`, `/`, or `%`.
* If the result is outside the signed 32-bit integer range:

```text
[-2³¹, 2³¹ - 1]
```

return the appropriate boundary value.

Example:

```text
dividend = 10
divisor = 3

Output = 3
```

Because:

```text
10 / 3 = 3.333...
```

and truncation toward zero gives `3`.

---

# Approach 1 — Repeated Subtraction

## Idea

Division can be viewed as repeatedly subtracting the divisor from the dividend.

For:

```text
10 / 3
```

we do:

```text
10 - 3 = 7
7 - 3 = 4
4 - 3 = 1
```

We subtracted `3` three times, so:

```text
quotient = 3
```

This is simple but inefficient when the numbers are large.

## Dry Run

```text
dividend = 10
divisor = 3

10 >= 3 → subtract
10 - 3 = 7
count = 1

7 >= 3 → subtract
7 - 3 = 4
count = 2

4 >= 3 → subtract
4 - 3 = 1
count = 3

1 < 3 → stop

Answer = 3
```

## Algorithm

1. Handle the special overflow case.
2. Determine whether the answer should be positive or negative.
3. Convert both numbers to positive values.
4. Repeatedly subtract `divisor` from `dividend`.
5. Count how many times subtraction is possible.
6. Apply the correct sign.
7. Return the result.

## Complexity

* **Time:** `O(|dividend / divisor|)`
* **Space:** `O(1)`

## Notes / Tips

* This approach is easy to understand but can take billions of operations.
* For example, dividing `2³¹ - 1` by `1` would require roughly `2³¹` subtractions.
* We need a faster way to subtract large multiples of the divisor at once.

## Code

```cpp
class Solution {
public:
    int divide(int dividend, int divisor) {
        if (dividend == INT_MIN && divisor == -1) {
            return INT_MAX;
        }

        long long a = abs((long long) dividend);
        long long b = abs((long long) divisor);

        int quotient = 0;

        while (a >= b) {
            a -= b;
            quotient++;
        }

        if ((dividend < 0) != (divisor < 0)) {
            quotient = -quotient;
        }

        return quotient;
    }
};
```

---

# Approach 2 — Exponential Doubling / Bit Manipulation

## Idea

Instead of subtracting the divisor one time at a time, repeatedly **double the divisor**.

For:

```text
43 / 3
```

we can generate:

```text
3 × 1  = 3
3 × 2  = 6
3 × 4  = 12
3 × 8  = 24
3 × 16 = 48  ← too large
```

So we can subtract:

```text
24
```

from `43`.

Remaining:

```text
43 - 24 = 19
```

Then:

```text
12 ≤ 19
```

Subtract `12`:

```text
19 - 12 = 7
```

Then:

```text
6 ≤ 7
```

Subtract `6`:

```text
7 - 6 = 1
```

The corresponding multiples were:

```text
8 + 4 + 2 = 14
```

Therefore:

```text
43 / 3 = 14
```

The key is that doubling corresponds to a **left shift**:

```text
3 × 2  → 3 << 1
3 × 4  → 3 << 2
3 × 8  → 3 << 3
```

## Dry Run

```text
dividend = 43
divisor = 3
```

Generate powers:

```text
3 × 1  = 3
3 × 2  = 6
3 × 4  = 12
3 × 8  = 24
3 × 16 = 48 → too large
```

Start from the largest valid multiple:

```text
43 - 24 = 19
quotient = 8
```

Next:

```text
19 - 12 = 7
quotient = 8 + 4 = 12
```

Next:

```text
7 - 6 = 1
quotient = 12 + 2 = 14
```

Now:

```text
1 < 3
```

Stop.

```text
Answer = 14
```

## Algorithm

1. Handle the overflow case:

   ```text
   INT_MIN / -1
   ```

   must return `INT_MAX`.
2. Determine whether the result should be negative.
3. Convert `dividend` and `divisor` to positive `long long` values.
4. Initialize `quotient = 0`.
5. While `dividend >= divisor`:

   * Start with:

     ```text
     current = divisor
     multiple = 1
     ```
   * Keep doubling both:

     ```text
     current <<= 1
     multiple <<= 1
     ```

     while the doubled value still fits inside `dividend`.
   * Subtract the largest valid `current` from `dividend`.
   * Add its corresponding `multiple` to `quotient`.
6. Apply the sign.
7. Return the quotient.

## Complexity

* **Time:** `O(log |dividend|)`
* **Space:** `O(1)`

## Notes / Tips

* **Do not use `abs(INT_MIN)` with an `int`.** `INT_MIN` cannot be represented as a positive `int`.
* Convert to `long long` before taking the absolute value.
* The overflow case is special:

  ```text
  -2³¹ / -1 = 2³¹
  ```

  but `2³¹` is larger than `INT_MAX`.
* Therefore:

  ```cpp
  if (dividend == INT_MIN && divisor == -1) {
      return INT_MAX;
  }
  ```
* `current << 1` means multiplying by `2`.
* `multiple << 1` tracks how many divisors are represented by `current`.
* We repeatedly take the **largest possible power-of-two multiple**.

## Code

```cpp
class Solution {
public:
    int divide(int dividend, int divisor) {
        if (dividend == INT_MIN && divisor == -1) {
            return INT_MAX;
        }

        bool negative = (dividend < 0) != (divisor < 0);

        long long dividendAbs = abs((long long) dividend);
        long long divisorAbs = abs((long long) divisor);

        long long quotient = 0;

        while (dividendAbs >= divisorAbs) {
            long long current = divisorAbs;
            long long multiple = 1;

            while ((current << 1) <= dividendAbs) {
                current <<= 1;
                multiple <<= 1;
            }

            dividendAbs -= current;
            quotient += multiple;
        }

        if (negative) {
            quotient = -quotient;
        }

        return (int) quotient;
    }
};
```

---

# Key Takeaway

The main trick is to replace **repeated subtraction** with **exponential subtraction**.

Instead of:

```text
dividend -= divisor
dividend -= divisor
dividend -= divisor
...
```

use:

```text
divisor
divisor × 2
divisor × 4
divisor × 8
...
```

Using bit shifts:

```cpp
current <<= 1;
multiple <<= 1;
```

So the reusable pattern is:

```text
Repeated operation
       ↓
Can we double the operation each time?
       ↓
Use powers of 2
       ↓
Bit manipulation
       ↓
O(log n)
```

**Key Pattern:** Exponential Doubling / Binary Decomposition.

**Key Template:**

```cpp
while (dividend >= divisor) {
    long long current = divisor;
    long long multiple = 1;

    while ((current << 1) <= dividend) {
        current <<= 1;
        multiple <<= 1;
    }

    dividend -= current;
    quotient += multiple;
}
```

**Remember:** When multiplication/division is forbidden and you need to repeatedly subtract something, think **powers of 2 + bit shifting**.
