# Add Two Integers

## Problem

Given two integers `num1` and `num2`, return their sum.

The problem asks for the sum without directly using the `+` operator.

Example:

```text
num1 = 2
num2 = 3

Output = 5
```

---

## Approach 1: Bit Manipulation

### Idea

Addition can be performed using bit operations.

For two numbers `a` and `b`:

* `a ^ b` gives the sum **without carry**.
* `a & b` identifies positions where both bits are `1`, meaning a carry is generated.
* Shift the carry left by one position:

  ```text
  (a & b) << 1
  ```

Repeat until there is no carry.

Formula:

```text
sum = a ^ b
carry = (a & b) << 1
```

Then:

```text
a = sum
b = carry
```

### Dry Run

```text
num1 = 5  → 0101
num2 = 3  → 0011

Without carry:
0101 ^ 0011 = 0110

Carry:
0101 & 0011 = 0001
0001 << 1   = 0010

Now:
0110 + 0010

Without carry:
0110 ^ 0010 = 0100

Carry:
0110 & 0010 = 0010
0010 << 1   = 0100

Now:
0100 + 0100

Without carry:
0100 ^ 0100 = 0000

Carry:
0100 & 0100 = 0100
0100 << 1   = 1000

Now:
0000 ^ 1000 = 1000
carry = 0

Answer = 8
```

### Algorithm

1. While `num2` is not `0`:

   * Calculate the sum without carry using XOR.
   * Calculate the carry using AND followed by left shift.
   * Store the sum in `num1`.
   * Store the carry in `num2`.
2. Return `num1`.

### Complexity

* Time: `O(log n)` in the number of bits.
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    int sum(int num1, int num2) {
        while (num2 != 0) {
            int sumWithoutCarry = num1 ^ num2;
            int carry = (num1 & num2) << 1;

            num1 = sumWithoutCarry;
            num2 = carry;
        }

        return num1;
    }
};
```

### Notes / Tips

* This is a classic **Bit Manipulation** problem.
* XOR performs addition without carrying.
* AND finds the carry positions.
* Left shift moves the carry to the correct position.
* Core formula:

  ```text
  a + b = (a ^ b) + ((a & b) << 1)
  ```
* Repeat until the carry becomes `0`.

### Key Template

```text
while b != 0:
    sum = a ^ b
    carry = (a & b) << 1

    a = sum
    b = carry

return a
```
