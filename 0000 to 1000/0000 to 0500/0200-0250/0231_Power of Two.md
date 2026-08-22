# 231. Power of Two

## Metadata

* **Topic:** Bit Manipulation
* **Difficulty:** Easy
* **Pattern:** Bit Manipulation
* **Key Pattern:** A positive power of `2` has exactly **one set bit**.

---

## Idea

A number is a power of `2` if it can be written as:

```text
2^k
```

Binary representation of powers of `2` contains exactly one `1`:

```text
1    = 2^0
10   = 2^1
100  = 2^2
1000 = 2^3
```

For a number with exactly one set bit:

```text
n & (n - 1) == 0
```

But we also need:

```text
n > 0
```

because `0` also satisfies `0 & (-1) == 0`, but `0` is not a power of `2`.

---

## Dry Run

### Example 1

```text
n = 8
```

Binary:

```text
1000
```

Then:

```text
n - 1 = 0111

1000
&
0111
----
0000
```

Therefore, `8` is a power of `2`.

### Example 2

```text
n = 6
```

Binary:

```text
110
```

```text
110
&
101
---
100
```

Result is not `0`, so `6` is not a power of `2`.

---

## Algorithm

1. Check that `n > 0`.
2. Check:

   ```text
   n & (n - 1) == 0
   ```
3. If both conditions are true, return `true`.
4. Otherwise return `false`.

---

## Complexity

* **Time:** `O(1)`
* **Space:** `O(1)`

---

## Notes / Tips

### Important Bit Trick

```text
n & (n - 1)
```

removes the rightmost `1` bit.

Therefore:

```text
Exactly one 1 bit → power of 2
More than one 1 bit → not a power of 2
```

### Key Template

```text
return n > 0 && (n & (n - 1)) == 0;
```

---

## Code

```cpp
class Solution {
public:
    bool isPowerOfTwo(int n) {
        return n > 0 && (n & (n - 1)) == 0;
    }
};
```
