# Minimum Bit Flips to Convert Number

## Problem

Given two integers `start` and `goal`, return the **minimum number of bit flips** needed to convert `start` into `goal`.

A bit flip changes:

* `0 → 1`
* `1 → 0`

Example:

```text
start = 10  → 1010
goal  =  7  → 0111

Different bits = 3

Output = 3
```

---

## Approach 1: XOR + Count Set Bits

### Idea

Use XOR between `start` and `goal`.

```text
start ^ goal
```

Properties of XOR:

* Same bits → `0`
* Different bits → `1`

Therefore, the number of `1`s in `start ^ goal` is exactly the number of bits that need to be flipped.

Then count the set bits.

### Dry Run

```text
start = 10  → 1010
goal  =  7  → 0111

1010
0111
----
1101

Number of 1s = 3

Answer = 3
```

### Algorithm

1. Compute `x = start ^ goal`.
2. Count the number of set bits in `x`.
3. Return the count.

### Complexity

* Time: `O(log n)` where `n` is the larger number.
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    int minBitFlips(int start, int goal) {
        int x = start ^ goal;
        int count = 0;

        while (x > 0) {
            count += x & 1;
            x >>= 1;
        }

        return count;
    }
};
```

### Notes / Tips

* This is a classic **Bit Manipulation + XOR** problem.
* XOR directly identifies the positions where the two numbers differ.
* Remember:

  ```text
  a ^ a = 0
  a ^ b = 1 when bits differ
  ```
* `x & 1` checks whether the last bit is `1`.
* `x >>= 1` removes the last bit.
* An even shorter solution can use C++'s built-in `__builtin_popcount(start ^ goal)`.

### Key Template

```text
x = a ^ b

count = 0

while x > 0:
    count += x & 1
    x >>= 1

return count
```
