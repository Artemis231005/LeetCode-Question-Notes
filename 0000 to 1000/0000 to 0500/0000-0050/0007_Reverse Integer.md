# LeetCode 7 — Reverse Integer

## Metadata

* **LeetCode:** 7
* **Problem:** Reverse Integer
* **Difficulty:** Medium
* **Topics:** Math
* **Pattern:** Digit Extraction with Overflow Guard
* **Key Technique:** Build the reversed number digit by digit, checking for 32-bit overflow before each multiply-and-add step instead of after
* **Optimal Complexity:** `O(log n)` Time, `O(1)` Space

---

## Problem Statement

Given a signed 32-bit integer `x`, return `x` with its digits reversed. If reversing causes the value to go outside the signed 32-bit integer range `[-2^31, 2^31 - 1]`, return `0`.

---

## Approaches

1. **Brute Force — String Conversion**
2. **Optimal — Digit-by-Digit with Overflow Check**

---

# Approach 1 — Brute Force / String Conversion

## Idea

Convert the integer to a string, handle the sign separately, reverse the digit characters, and convert back to an integer — checking the result against the 32-bit range using a wider type (like `long long`) to detect overflow safely.

## Dry Run

```text
x = -123
```

Extract sign and digits:

```text
sign = negative
digits = "123"
```

Reverse:

```text
"321"
```

Reapply sign and parse:

```text
-321
```

Check range: `-321` fits within `[-2^31, 2^31 - 1]` → return `-321`.

## Algorithm

1. Record the sign of `x`, then work with its absolute value.
2. Convert the absolute value to a string.
3. Reverse the string.
4. Convert the reversed string back to a number using a 64-bit type.
5. Reapply the sign.
6. If the result is outside `[-2^31, 2^31 - 1]`, return `0`; otherwise return the result.

## Complexity

* **Time:** `O(log n)`

  * Proportional to the number of digits in `x`, from the string conversion and reversal.
* **Space:** `O(log n)`

  * For the string representation of the number.

## Notes / Tips

* Simple and readable, but uses string conversion overhead that's unnecessary — the reversal can be done with pure arithmetic just as easily.
* Using `long long` (or `stoll`) for the parse step is essential — parsing directly into a 32-bit `int` risks undefined behavior on overflow before the range check even happens.

## Code

```cpp
class Solution {
public:
    int reverse(int x) {
        bool negative = x < 0;
        string s = to_string(negative ? -(long long)x : x);

        std::reverse(s.begin(), s.end());

        long long result = stoll(s);
        if (negative) result = -result;

        if (result < INT_MIN || result > INT_MAX) {
            return 0;
        }

        return (int)result;
    }
};
```

---

# Approach 2 — Optimal / Digit-by-Digit with Overflow Check

## Idea

Extract and append digits arithmetically, the same way as reversing any integer (`digit = x % 10`, `result = result * 10 + digit`, `x /= 10`), but check for overflow **before** each multiply-and-add step rather than after, since the overflow itself would already be undefined behavior on a 32-bit `int` if allowed to happen.

## Dry Run

```text
x = -123
```

Process (working with the sign preserved throughout, since `%` and `/` truncate toward zero in C++ for negative numbers):

```text
x=-123: digit = -123 % 10 = -3
        result = 0*10 + (-3) = -3
        x = -123/10 = -12

x=-12:  digit = -12 % 10 = -2
        result = -3*10 + (-2) = -32
        x = -12/10 = -1

x=-1:   digit = -1 % 10 = -1
        result = -32*10 + (-1) = -321
        x = -1/10 = 0
```

`x == 0` → stop. Result `-321` is within range → return `-321`.

## Algorithm

1. Initialize `result = 0`.
2. While `x != 0`:

   * `digit = x % 10`.
   * Before updating, check if `result > INT_MAX / 10` or `result < INT_MIN / 10` (or the edge case where `result` equals the boundary quotient but `digit` would push it over) — if so, return `0` immediately (overflow would occur).
   * `result = result * 10 + digit`.
   * `x /= 10`.
3. Return `result`.

## Complexity

* **Time:** `O(log n)`

  * One iteration per digit of `x`.
* **Space:** `O(1)`

  * Only a running `result` and the shrinking `x` — no strings or extra structures.

## Notes / Tips

* C++'s `%` and `/` operators truncate toward zero for negative numbers (e.g. `-123 % 10 = -3`, not `7`), which conveniently means the same digit-extraction logic works for both positive and negative `x` without separate sign-handling code.
* The overflow check must happen **before** the multiply-and-add, not after — by the time `result * 10 + digit` overflows a 32-bit `int`, the behavior is already undefined in C++, so the value can't be trusted to safely detect the overflow post-hoc.
* This exact "check before multiply-add" overflow guard is the standard pattern for any digit-reversal or digit-accumulation problem that must stay within a fixed integer width (e.g. LC 9 — Palindrome Number's reversal step).

## Code

```cpp
class Solution {
public:
    int reverse(int x) {
        int result = 0;

        while (x != 0) {
            int digit = x % 10;

            if (result > INT_MAX / 10 || (result == INT_MAX / 10 && digit > 7)) {
                return 0;
            }
            if (result < INT_MIN / 10 || (result == INT_MIN / 10 && digit < -8)) {
                return 0;
            }

            result = result * 10 + digit;
            x /= 10;
        }

        return result;
    }
};
```

---

## Key Template

```text
result = 0

while x != 0:
    digit = x % 10

    if result > INT_MAX/10 or (result == INT_MAX/10 and digit > 7): return 0
    if result < INT_MIN/10 or (result == INT_MIN/10 and digit < -8): return 0

    result = result * 10 + digit
    x /= 10

return result
```