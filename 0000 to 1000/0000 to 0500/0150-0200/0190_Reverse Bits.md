# 190. Reverse Bits

## Metadata

* **Topic:** Bit Manipulation
* **Difficulty:** Easy
* **Pattern:** Bit Extraction + Bit Shifting
* **Key Pattern:** Extract the last bit of `n` and build the reversed number from left to right.

---

## Idea

We are given a **32-bit unsigned integer** and need to reverse all 32 bits.

Example:

```text id="a8qk71"
n = 00000010100101000001111010011100

reverse = 00111001011110000010100101000000
```

For every bit:

1. Extract the last bit using:

   ```text
   n & 1
   ```
2. Shift the answer left.
3. Add the extracted bit.
4. Shift `n` right to process the next bit.

We must perform this exactly **32 times** because leading zeroes also need to be reversed.

---

## Dry Run

Consider a smaller 4-bit example:

```text id="r5uhw8"
n = 1011
```

Process from right to left:

```text
bit = 1 → ans = 0001
bit = 1 → ans = 0011
bit = 0 → ans = 0110
bit = 1 → ans = 1101
```

So:

```text id="x5w1b3"
1011 → 1101
```

The same process is repeated **32 times** for the actual problem.

---

## Algorithm

1. Initialize `ans = 0`.
2. Repeat 32 times:

   * Extract the last bit:

     ```text
     bit = n & 1
     ```
   * Shift `ans` left by 1.
   * Add `bit` to `ans`.
   * Shift `n` right by 1.
3. Return `ans`.

---

## Complexity

* **Time:** `O(32)` → `O(1)`
* **Space:** `O(1)`

---

## Notes / Tips

* `n & 1` extracts the **rightmost bit**.
* `n >> 1` removes the rightmost bit.
* `ans << 1` creates space for the next reversed bit.
* Do exactly **32 iterations**, including leading zeroes.
* Since the input is a 32-bit integer, the number of iterations is fixed.

### Key Template

```text id="ef7l7x"
ans = 0

repeat 32 times:
    bit = n & 1
    ans = (ans << 1) | bit
    n = n >> 1

return ans
```

---

## Code

```cpp
class Solution {
public:
    uint32_t reverseBits(uint32_t n) {
        uint32_t ans = 0;

        for (int i = 0; i < 32; i++) {
            ans = (ans << 1) | (n & 1);
            n >>= 1;
        }

        return ans;
    }
};
```
