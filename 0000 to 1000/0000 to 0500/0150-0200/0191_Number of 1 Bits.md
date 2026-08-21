# 191. Number of 1 Bits

## Metadata

* **Topic:** Bit Manipulation
* **Difficulty:** Easy
* **Pattern:** Brian Kernighan's Algorithm
* **Key Pattern:** `n & (n - 1)` removes the lowest set bit.

---

## Idea

We need to count the number of `1` bits in a binary number.

The key operation is:

```text
n & (n - 1)
```

It removes the **rightmost `1` bit** from `n`.

Example:

```text
n     = 101100
n - 1 = 101011

n & (n - 1)
      = 101000
```

One `1` bit was removed.

Therefore, keep applying this operation and increment the count until `n` becomes `0`.

---

## Dry Run

```text
n = 101100
```

| Step | `n`      | Count |
| ---- | -------- | ----: |
| 1    | `101000` |     1 |
| 2    | `100000` |     2 |
| 3    | `000000` |     3 |

Therefore:

```text
Number of 1 bits = 3
```

---

## Algorithm

1. Initialize `count = 0`.
2. While `n != 0`:

   * Remove the lowest set bit:

     ```text
     n = n & (n - 1)
     ```
   * Increment `count`.
3. Return `count`.

---

## Complexity

* **Time:** `O(k)`, where `k` = number of set bits.
* **Space:** `O(1)`

Worst case:

```text
O(32) = O(1)
```

---

## Notes / Tips

### Most Important Trick

```text
n & (n - 1)
```

removes exactly **one `1` bit**.

Example:

```text
10010000
&
10001111
-----------
10000000
```

So the number of iterations depends on the number of `1`s, not the total number of bits.

### Key Template

```text
count = 0

while n != 0:
    n = n & (n - 1)
    count++

return count
```

---

## Code

```cpp
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int count = 0;

        while (n != 0) {
            n = n & (n - 1);
            count++;
        }

        return count;
    }
};
```
