# Fibonacci Number

## Problem

Given an integer `n`, calculate the `n`th Fibonacci number.

The Fibonacci sequence is defined as:

```text
F(0) = 0
F(1) = 1
F(n) = F(n - 1) + F(n - 2)
```

Example:

```text
n = 4

F(4) = 3
```

---

## Approach 1: Iterative / Space Optimized

### Idea

Only the previous two Fibonacci numbers are needed to calculate the next one.

Maintain:

* `prev2` → `F(n-2)`
* `prev1` → `F(n-1)`

Calculate the next value and shift the variables forward.

### Dry Run

```text
n = 5

prev2 = 0
prev1 = 1

next = 1
prev2 = 1
prev1 = 1

next = 2
prev2 = 1
prev1 = 2

next = 3
prev2 = 2
prev1 = 3

next = 5

Answer = 5
```

### Algorithm

1. If `n == 0`, return `0`.
2. Initialize `prev2 = 0` and `prev1 = 1`.
3. Repeat from `2` to `n`:

   * `current = prev1 + prev2`
   * `prev2 = prev1`
   * `prev1 = current`
4. Return `prev1`.

### Complexity

* Time: `O(n)`
* Space: `O(1)`

### Code

```cpp
class Solution {
public:
    int fib(int n) {
        if (n == 0) {
            return 0;
        }

        int prev2 = 0;
        int prev1 = 1;

        for (int i = 2; i <= n; i++) {
            int current = prev1 + prev2;
            prev2 = prev1;
            prev1 = current;
        }

        return prev1;
    }
};
```

### Notes / Tips

* Fibonacci has a **two-state dependency**: only the previous two values are required.
* Therefore, an entire DP array is unnecessary.
* Handle `n = 0` separately.
* This is an example of **space optimization in DP**.
* Recursive Fibonacci without memoization takes `O(2^n)` time, so the iterative approach is much better.

### Key Template

```text
if base case:
    return base value

prev2 = first value
prev1 = second value

for i from 2 to n:
    current = prev1 + prev2
    prev2 = prev1
    prev1 = current

return prev1
```
