# LeetCode 50 — Pow(x, n)

## Metadata

* **LeetCode:** 50
* **Problem:** Pow(x, n)
* **Difficulty:** Medium
* **Topics:** Math, Recursion, Binary Exponentiation
* **Pattern:** Divide and Conquer
* **Key Pattern:** Binary Exponentiation / Exponentiation by Squaring
* **Key Technique:** Repeatedly square the base and halve the exponent
* **Optimal Complexity:** `O(log n)` time, `O(1)` space for iterative approach
* **Key Template:** Binary Exponentiation

---

## Problem

Implement:

```cpp
double myPow(double x, int n)
```

which calculates:

```text
x^n
```

Examples:

```text
x = 2.00000, n = 10
→ 1024.00000
```

```text
x = 2.10000, n = 3
→ 9.26100
```

```text
x = 2.00000, n = -2
→ 0.25000
```

The challenge is to calculate the result efficiently.

A simple loop would take `O(n)` time, which is too slow for large exponents.

---

## Key Observation

Instead of multiplying `x` by itself `n` times, repeatedly **square the base and halve the exponent**.

For example:

```text
x^10
```

can be written as:

```text
x^10
= (x^5)^2
```

and:

```text
x^5
= x × x^4
```

Then:

```text
x^4
= (x^2)^2
```

and:

```text
x^2
= x × x
```

This reduces the number of operations from `O(n)` to `O(log n)`.

---

## Approach 1 — Iterative Binary Exponentiation

### Idea

Use the binary representation of the exponent.

For every bit of `n`:

* If the current exponent is odd, multiply the answer by the current base.
* Square the base.
* Divide the exponent by `2`.

Maintain:

```cpp
double ans = 1;
```

At every step:

```text
ans = result accumulated so far
x   = current power of x
n   = remaining exponent
```

The key operations are:

```cpp
if (n % 2 == 1) {
    ans *= x;
}

x *= x;
n /= 2;
```

---

## Dry Run

Calculate:

```text
2^10
```

Initially:

```text
ans = 1
x = 2
n = 10
```

### Step 1

`n = 10` is even.

```text
ans = 1
x = 2² = 4
n = 5
```

### Step 2

`n = 5` is odd.

Multiply:

```text
ans = 1 × 4 = 4
```

Then:

```text
x = 4² = 16
n = 2
```

### Step 3

`n = 2` is even.

```text
ans = 4
x = 16² = 256
n = 1
```

### Step 4

`n = 1` is odd.

```text
ans = 4 × 256
    = 1024
```

Then:

```text
n = 0
```

Stop.

Final answer:

```text
1024
```

---

## Why Does This Work?

Suppose:

```text
n = 13
```

Binary representation:

```text
13 = 1101₂
```

Therefore:

```text
13 = 8 + 4 + 1
```

So:

```text
x^13
= x^8 × x^4 × x
```

Repeated squaring produces:

```text
x
x²
x⁴
x⁸
```

We only multiply the powers corresponding to the `1` bits:

```text
x^8 × x^4 × x
```

Therefore binary exponentiation effectively calculates the exponent using its binary representation.

---

## Handling Negative Exponents

The definition is:

```text
x^(-n) = 1 / x^n
```

For example:

```text
2^-3
= 1 / 2^3
= 1 / 8
= 0.125
```

So we can convert:

```text
x → 1 / x
n → -n
```

and then calculate the positive exponent.

However, there is an important C++ issue.

---

## Important Edge Case — `INT_MIN`

An `int` has range:

```text
-2³¹ to 2³¹ - 1
```

Therefore:

```text
INT_MIN = -2³¹
```

but:

```text
INT_MAX = 2³¹ - 1
```

So this is dangerous:

```cpp
n = -n;
```

when:

```cpp
n == INT_MIN
```

because `2³¹` cannot be represented by a signed `int`.

### Safe Solution

Convert the exponent to `long long` first:

```cpp
long long power = n;
```

Then:

```cpp
if (power < 0) {
    x = 1 / x;
    power = -power;
}
```

Now `-INT_MIN` fits safely inside `long long`.

---

### Algorithm

1. Convert `n` to `long long`.
2. If the exponent is negative:

   * Replace `x` with `1 / x`.
   * Make the exponent positive.
3. Initialize:

   ```cpp
   ans = 1
   ```
4. While `n > 0`:

   * If `n` is odd, multiply `ans` by `x`.
   * Square `x`.
   * Divide `n` by `2`.
5. Return `ans`.

---

### Complexity

* **Time:** `O(log |n|)`
* **Space:** `O(1)`

Each iteration approximately halves the exponent:

```text
n
n/2
n/4
n/8
...
0
```

Therefore there are only `O(log n)` iterations.

---

### Notes / Tips

* Do not use:

  ```cpp
  pow(x, n)
  ```

  because the problem asks you to implement the operation.
* Do not use a simple loop from `1` to `n`; that is `O(n)`.
* Always convert `n` to `long long` before handling negative exponents.
* The key condition is:

  ```cpp
  n % 2 == 1
  ```
* The key optimization is:

  ```cpp
  x *= x;
  n /= 2;
  ```
* For negative exponents:

  ```text
  x^-n = 1 / x^n
  ```
* `ans` stores the powers corresponding to the `1` bits of the exponent.

---

### Code

```cpp
class Solution {
public:
    double myPow(double x, int n) {
        long long power = n;

        if (power < 0) {
            x = 1 / x;
            power = -power;
        }

        double ans = 1.0;

        while (power > 0) {
            if (power % 2 == 1) {
                ans *= x;
            }

            x *= x;
            power /= 2;
        }

        return ans;
    }
};
```

---

## Approach 2 — Recursive Binary Exponentiation

### Idea

Use the mathematical relationship:

For even `n`:

```text
x^n = (x^(n/2))²
```

For odd `n`:

```text
x^n = x × x^(n-1)
```

A cleaner recursive formulation is:

```text
x^n
= x^(n/2) × x^(n/2)
```

and then multiply by `x` if `n` is odd.

Define:

```cpp
power(x, n)
```

as the function that returns `x^n`.

Base case:

```text
n = 0
→ 1
```

---

## Dry Run

Calculate:

```text
2^10
```

We recursively divide the exponent:

```text
2^10
 ↓
2^5
 ↓
2^2
 ↓
2^1
 ↓
2^0
```

Base case:

```text
2^0 = 1
```

Return upward:

```text
2^1 = 1 × 1 × 2 = 2
```

```text
2^2 = 2 × 2 = 4
```

```text
2^5 = 4 × 4 × 2 = 32
```

```text
2^10 = 32 × 32 = 1024
```

So:

```text
2^10 = 1024
```

---

### Algorithm

1. If `n == 0`, return `1`.
2. Recursively calculate:

   ```cpp
   half = power(x, n / 2)
   ```
3. Square `half`.
4. If `n` is odd, multiply the result by `x`.
5. Return the result.
6. Handle negative exponents by calculating the reciprocal.

---

### Complexity

* **Time:** `O(log |n|)`
* **Space:** `O(log |n|)` due to recursion stack

The recursive approach has the same time complexity as the iterative approach but uses additional stack space.

---

### Notes / Tips

The important mathematical identities are:

```text
n is even:
x^n = (x^(n/2))²
```

and:

```text
n is odd:
x^n = (x^(n/2))² × x
```

C++ integer division automatically gives:

```text
5 / 2 = 2
```

which works correctly here.

Again, convert `n` to `long long` before taking its absolute value.

---

### Code

```cpp
class Solution {
public:
    double fastPower(double x, long long n) {
        if (n == 0) {
            return 1.0;
        }

        double half = fastPower(x, n / 2);
        double result = half * half;

        if (n % 2 == 1) {
            result *= x;
        }

        return result;
    }

    double myPow(double x, int n) {
        long long power = n;

        if (power < 0) {
            return 1.0 / fastPower(x, -power);
        }

        return fastPower(x, power);
    }
};
```

---

## Key Takeaways

### Core Binary Exponentiation Template

```cpp
double ans = 1.0;

while (power > 0) {
    if (power % 2 == 1) {
        ans *= x;
    }

    x *= x;
    power /= 2;
}
```

The entire optimization comes from:

```text
Exponent
   ↓
Halve exponent
   ↓
Square base
   ↓
Repeat
```

Instead of:

```text
x × x × x × x × ...   n times
```

we calculate:

```text
x
x²
x⁴
x⁸
x¹⁶
...
```

This changes:

```text
O(n)
```

to:

```text
O(log n)
```

### Most Important Edge Case

```cpp
long long power = n;
```

must be done **before**:

```cpp
power = -power;
```

because `INT_MIN` cannot be negated safely as an `int`.

### Mental Model

> **Binary exponentiation repeatedly halves the exponent and squares the base, multiplying the answer only when the current exponent is odd.**
