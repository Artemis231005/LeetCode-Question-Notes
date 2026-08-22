# 338. Counting Bits

## Metadata

* **Topic:** Dynamic Programming, Bit Manipulation
* **Difficulty:** Easy
* **Key Pattern:** DP using `n & (n - 1)`
* **Key Template:** `bits[i] = bits[i >> 1] + (i & 1)`
* **Goal:** Return an array where `ans[i]` is the number of `1`s in the binary representation of `i`.

---

## Approach 1: DP using Right Shift

### Idea

For every number `i`:

```text
i >> 1
```

removes the last binary bit.

So the number of set bits in `i` is:

```text
bits[i] = bits[i >> 1] + (i & 1)
```

* `i >> 1` → removes the last bit.
* `i & 1` → tells whether the last bit was `1`.

### Dry Run

For `n = 5`:

```text
0 → 000 → 0
1 → 001 → 1
2 → 010 → 1
3 → 011 → 2
4 → 100 → 1
5 → 101 → 2
```

For `5`:

```text
bits[5]
= bits[5 >> 1] + (5 & 1)
= bits[2] + 1
= 1 + 1
= 2
```

Answer:

```text
[0, 1, 1, 2, 1, 2]
```

### Algorithm

1. Create an array `bits` of size `n + 1`.
2. Initialize `bits[0] = 0`.
3. For every `i` from `1` to `n`:

   * Take the number of set bits in `i >> 1`.
   * Add the last bit using `i & 1`.
4. Store the result in `bits[i]`.
5. Return `bits`.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`

### Notes / Tips

* `i & 1` checks whether `i` is odd:

  * `1` → last bit is `1`
  * `0` → last bit is `0`
* `i >> 1` is equivalent to integer division by `2`.
* This is the most important formula to remember:

  ```text
  bits[i] = bits[i >> 1] + (i & 1)
  ```
* The problem is a good example of using a **smaller subproblem** to build the current answer.

### Code

```cpp
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> bits(n + 1, 0);

        for (int i = 1; i <= n; i++) {
            bits[i] = bits[i >> 1] + (i & 1);
        }

        return bits;
    }
};
```

---

## Approach 2: DP using `i & (i - 1)`

### Idea

The operation:

```text
i & (i - 1)
```

removes the **rightmost set bit** of `i`.

Therefore:

```text
bits[i] = bits[i & (i - 1)] + 1
```

We already know the number of set bits in the number after removing one `1`.

### Dry Run

For `i = 6`:

```text
6 = 110
5 = 101

6 & 5 = 100 = 4
```

So:

```text
bits[6] = bits[4] + 1
         = 1 + 1
         = 2
```

Since:

```text
6 = 110
```

it contains two `1`s.

### Algorithm

1. Create an array `bits` of size `n + 1`.
2. Initialize `bits[0] = 0`.
3. For each `i` from `1` to `n`:

   * Remove the rightmost set bit using `i & (i - 1)`.
   * Add `1` to the stored count.
4. Return the array.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(n)`

### Notes / Tips

* `i & (i - 1)` is a very important bit manipulation trick.
* It removes exactly one set bit.
* Example:

  ```text
  12 = 1100
  11 = 1011

  12 & 11 = 1000
  ```
* This technique is also useful for checking whether a number is a power of two:

  ```cpp
  n > 0 && (n & (n - 1)) == 0
  ```

### Code

```cpp
class Solution {
public:
    vector<int> countBits(int n) {
        vector<int> bits(n + 1, 0);

        for (int i = 1; i <= n; i++) {
            bits[i] = bits[i & (i - 1)] + 1;
        }

        return bits;
    }
};
```

---

## Key Template

```cpp
vector<int> bits(n + 1, 0);

for (int i = 1; i <= n; i++) {
    bits[i] = bits[i >> 1] + (i & 1);
}
```

### Alternative Bit Template

```cpp
bits[i] = bits[i & (i - 1)] + 1;
```

### Pattern to Remember

```text
i >> 1
    ↓
Remove last binary bit

i & 1
    ↓
Check last binary bit

Therefore:

bits[i] = bits[i >> 1] + (i & 1)
```
