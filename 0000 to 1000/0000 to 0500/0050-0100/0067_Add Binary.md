# LeetCode 67 — Add Binary

## Metadata

* **LeetCode:** 67
* **Problem:** Add Binary
* **Difficulty:** Easy
* **Topics:** Math, String, Bit Manipulation
* **Pattern:** Carry Propagation
* **Key Pattern:** Add binary digits from right to left with a carry
* **Key Technique:** `sum = bitA + bitB + carry`
* **Key Template:** Right-to-Left Addition with Carry
* **Optimal Complexity:** `O(max(n, m))` time, `O(max(n, m))` space

---

## Problem

Given two binary strings `a` and `b`, return their sum as a **binary string**.

Example:

```text id="g5k3h1"
a = "11"
b = "1"

  11
+ 01
----
 100
```

Answer:

```text id="qz7h2p"
"100"
```

---

## Approach — Binary Addition with Carry

### Idea

This works exactly like normal decimal addition, except each digit is either `0` or `1`.

Start from the **rightmost digits** and move left.

For every position:

```text id="n2w6s8"
sum = bitA + bitB + carry
```

The resulting digit is:

```text id="z3x1b7"
digit = sum % 2
```

And the carry is:

```text id="f8m4k2"
carry = sum / 2
```

Since binary digits are only `0` or `1`, `sum` can only be:

```text id="8d4jqp"
0 → digit 0, carry 0
1 → digit 1, carry 0
2 → digit 0, carry 1
3 → digit 1, carry 1
```

Build the answer from right to left, then reverse it.

---

### Dry Run

For:

```text id="w5q8rs"
a = "1010"
b = "1011"
```

Addition:

```text id="3y7k1p"
  1010
+ 1011
------
 10101
```

Process from right to left:

| `a` | `b` | Carry | Sum | Digit | New Carry |
| --: | --: | ----: | --: | ----: | --------: |
|   0 |   1 |     0 |   1 |     1 |         0 |
|   1 |   1 |     0 |   2 |     0 |         1 |
|   0 |   0 |     1 |   1 |     1 |         0 |
|   1 |   1 |     0 |   2 |     0 |         1 |
|   - |   - |     1 |   1 |     1 |         0 |

The digits are generated from right to left:

```text id="6m1q8s"
10101
```

Therefore:

```text id="4p7n2c"
Answer = "10101"
```

---

### Algorithm

1. Set:

   ```text
   i = a.length() - 1
   j = b.length() - 1
   carry = 0
   ```
2. While `i >= 0`, `j >= 0`, or `carry != 0`:

   * Initialize `sum = carry`.
   * If `i >= 0`, add `a[i] - '0'` to `sum`.
   * If `j >= 0`, add `b[j] - '0'` to `sum`.
   * Append:

     ```text
     sum % 2
     ```

     to the result.
   * Update:

     ```text
     carry = sum / 2
     ```
   * Move `i` and `j` left.
3. Reverse the result because digits were generated from right to left.
4. Return the result.

---

### Complexity

Let:

```text id="p3r7vn"
n = length of a
m = length of b
```

* **Time:** `O(max(n, m))`
* **Space:** `O(max(n, m))` for the result string.

---

### Notes / Tips

* Treat characters as digits using:

```cpp
a[i] - '0'
```

* The most important formula is:

```text id="9b3h4k"
sum = bitA + bitB + carry

digit = sum % 2
carry = sum / 2
```

* Unlike **Plus One (66)**, here there are **two input numbers**, so both strings need to be traversed.
* The strings can have different lengths.

Example:

```text id="x4m7qs"
  101
+   11
------
  1000
```

The missing positions are treated as `0`.

* The loop condition must include `carry`:

```text id="8w2k5r"
while (i >= 0 || j >= 0 || carry)
```

This handles cases like:

```text id="j9s3kx"
"1" + "1" = "10"
```

After both strings are exhausted, the remaining carry creates the leading `1`.

* Don't convert the entire binary strings into integers because they may be too large for standard integer types.

---

### Code

```cpp id="k5v8z2"
class Solution {
public:
    string addBinary(string a, string b) {
        int i = a.size() - 1;
        int j = b.size() - 1;
        int carry = 0;

        string result;

        while (i >= 0 || j >= 0 || carry) {
            int sum = carry;

            if (i >= 0) {
                sum += a[i] - '0';
                i--;
            }

            if (j >= 0) {
                sum += b[j] - '0';
                j--;
            }

            result += char('0' + (sum % 2));
            carry = sum / 2;
        }

        reverse(result.begin(), result.end());

        return result;
    }
};
```

---

## Quick Revision

```text id="r7m2x9"
Add Binary
    ↓
Start from right
    ↓
sum = bitA + bitB + carry
    ↓
digit = sum % 2
carry = sum / 2
    ↓
Move left
    ↓
Reverse result
```

### Core Template

```text id="u4n8cw"
i = a.length - 1
j = b.length - 1
carry = 0

while i >= 0 OR j >= 0 OR carry:
    sum = carry

    if i >= 0:
        sum += a[i] - '0'
        i--

    if j >= 0:
        sum += b[j] - '0'
        j--

    result += sum % 2
    carry = sum / 2

reverse(result)
```

### Key Insight

```text id="h6q2vt"
Binary addition:

0 + 0 + carry
0 + 1 + carry
1 + 0 + carry
1 + 1 + carry

Only possible sums:
0, 1, 2, 3
```

**Pattern to remember:**
**Add Binary → Right-to-left traversal → `sum = a + b + carry` → `digit = sum % 2` → `carry = sum / 2`**
