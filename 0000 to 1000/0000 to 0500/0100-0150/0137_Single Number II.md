# LeetCode 137 — Single Number II

## Metadata

* **LeetCode:** 137
* **Problem:** Single Number II
* **Difficulty:** Medium
* **Topics:** Array, Bit Manipulation
* **Pattern:** Bit Manipulation, Bit Counting
* **Key Technique:** Count each bit modulo `3`
* **Key Pattern:** Bitwise frequency counting
* **Key Template:** `bitCount[i] % 3`
* **Optimal Complexity:** `O(n)`

---

## Problem

Given an integer array `nums` where:

* Every element appears **three times**.
* Exactly one element appears **once**.

Find the element that appears only once.

Example:

```text
nums = [2, 2, 3, 2]
```

Answer:

```text
3
```

Another example:

```text
nums = [0, 1, 0, 1, 0, 1, 99]
```

Answer:

```text
99
```

The solution must run in:

```text
Time  = O(n)
Space = O(1)
```

---

## Idea

For LeetCode 136, XOR works because:

```text
x ^ x = 0
```

But here every number appears **three times**, so XOR cannot cancel them completely:

```text
x ^ x ^ x = x
```

Therefore, we use **bit counting**.

### Bit Counting

Every integer has a fixed number of bits.

For each bit position:

1. Count how many numbers have that bit set.
2. Take the count modulo `3`.
3. If the remainder is `1`, that bit belongs to the unique number.

Why?

If a number appears three times, every bit contributed by it appears a multiple of `3` times.

So:

```text
count % 3 = 0
```

The unique number appears once, so its set bits produce:

```text
count % 3 = 1
```

Reconstruct the answer from those bits.

---

## Dry Run

Consider:

```text
nums = [2, 2, 3, 2]
```

Binary representation:

```text
2 = 010
3 = 011
2 = 010
2 = 010
```

Count each bit:

```text
Bit 2   Bit 1   Bit 0
  0       4       1
```

Now take modulo `3`:

```text
0 % 3 = 0
4 % 3 = 1
1 % 3 = 1
```

So the resulting bits are:

```text
011
```

which is:

```text
3
```

Therefore:

```text
answer = 3
```

---

## Algorithm

1. Initialize an integer `ans = 0`.
2. For every bit position from `0` to `31`:

   * Count how many numbers have this bit set.
3. Take the count modulo `3`.
4. If the remainder is `1`, set that bit in `ans`.
5. After checking all 32 bits, return `ans`.

---

## Complexity

* **Time:** `O(32n)` = `O(n)`

  * 32 is constant.
* **Space:** `O(1)`

  * Only a few integer variables are used.

---

## Notes / Tips

* The key idea is:

  ```text
  Count bits → modulo 3 → reconstruct answer
  ```
* XOR alone does **not** solve this problem.
* `32` bits are checked for a standard C++ `int`.
* This method works for negative numbers as well when iterating through all 32 bits.
* The important difference between LeetCode 136 and 137:

### LeetCode 136

Every number appears twice:

```text
x ^ x = 0
```

So use:

```text
XOR
```

### LeetCode 137

Every number appears three times:

```text
bit count % 3
```

So use:

```text
Bit Counting
```

### Common Mistake

Do not use:

```cpp
ans ^= num;
```

because:

```text
x ^ x ^ x = x
```

The triplicated number would still remain.

---

## Code

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int ans = 0;

        for (int bit = 0; bit < 32; bit++) {
            int count = 0;

            for (int num : nums) {
                if ((num >> bit) & 1) {
                    count++;
                }
            }

            if (count % 3 != 0) {
                ans |= (1 << bit);
            }
        }

        return ans;
    }
};
```

---

## Basic Template

```cpp
int singleNumber(vector<int>& nums) {
    int ans = 0;

    for (int bit = 0; bit < 32; bit++) {
        int count = 0;

        for (int num : nums) {
            if ((num >> bit) & 1) {
                count++;
            }
        }

        if (count % 3 != 0) {
            ans |= (1 << bit);
        }
    }

    return ans;
}
```

### Reusable Pattern

```text
For each bit position
        ↓
Count set bits across all numbers
        ↓
count % 3
        ↓
Remainder = 1?
   ↓             ↓
  Yes            No
   ↓              ↓
Set bit         Ignore
        ↓
Reconstruct answer
```

### Core Formula

```text
answer_bit = bit_count % 3
```

This works because all repeated numbers contribute bits in multiples of `3`, leaving only the unique number's bits.
