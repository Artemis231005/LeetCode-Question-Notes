# LeetCode 66 — Plus One

## Metadata

* **LeetCode:** 66
* **Problem:** Plus One
* **Difficulty:** Easy
* **Topics:** Array, Math
* **Pattern:** Carry Propagation
* **Key Pattern:** Traverse from right to left and handle carry
* **Key Technique:** If digit is `9`, make it `0` and carry to the left
* **Key Template:** Right-to-Left Carry
* **Optimal Complexity:** `O(n)` time, `O(1)` auxiliary space

---

## Problem

You are given a large integer represented as an array of digits.

The digits are stored such that:

* The most significant digit is at index `0`.
* The least significant digit is at index `n - 1`.
* There are no leading zeros.

Increment the integer by **one** and return the resulting array of digits.

Example:

```text
digits = [1,2,3]

123 + 1 = 124

Answer = [1,2,4]
```

---

## Approach — Right-to-Left Carry

### Idea

Addition starts from the **rightmost digit**, because that is where the `+1` is applied.

For each digit from right to left:

* If the digit is less than `9`, simply increment it and return.
* If the digit is `9`, it becomes `0` and the carry continues to the left.

For example:

```text
[1,2,9]
     ↑
     9 + 1 = 10
```

So:

```text
[1,2,9]
    ↓
[1,3,0]
```

If every digit is `9`, we need an extra digit:

```text
[9,9,9]
   ↓
[1,0,0,0]
```

---

### Dry Run

#### Example 1

```text
digits = [1,2,3]
```

Start from the right:

```text
3 + 1 = 4
```

Since `3 != 9`, no carry is needed.

```text
[1,2,4]
```

Answer:

```text
[1,2,4]
```

#### Example 2

```text
digits = [1,2,9]
```

Start from the right:

```text
9 → 0
carry continues
```

Next:

```text
2 + 1 = 3
```

Result:

```text
[1,3,0]
```

#### Example 3

```text
digits = [9,9,9]
```

Process from right to left:

```text
9 → 0
9 → 0
9 → 0
```

All digits were `9`, so the carry remains.

Add `1` at the beginning:

```text
[1,0,0,0]
```

---

### Algorithm

1. Start from the last digit.
2. Traverse from right to left.
3. For each digit:

   * If it is `9`:

     ```text
     digit = 0
     ```

     and continue carrying.
   * Otherwise:

     ```text
     digit++
     ```

     and return the array immediately.
4. If the loop finishes, every digit was `9`.
5. Insert `1` at the beginning.
6. Return the resulting array.

---

### Complexity

* **Time:** `O(n)` in the worst case.
* **Auxiliary Space:** `O(1)` excluding the space required for the returned array.
* In the worst case, inserting `1` at the beginning may require `O(n)` movement, but the overall complexity remains `O(n)`.

---

### Notes / Tips

* The key pattern is **carry propagation**.
* You only need to continue moving left when the current digit is `9`.
* As soon as you encounter a digit `< 9`, increment it and **return immediately**.
* Important cases:

```text
[1,2,3] → [1,2,4]

[1,2,9] → [1,3,0]

[9,9] → [1,0,0]

[9] → [1,0]
```

* Do not convert the array into an integer because the number can be too large to fit into standard integer types.
* The most important observation:

```text
digit < 9
    ↓
increment and stop

digit == 9
    ↓
make 0 and carry left
```

---

### Code

```cpp
class Solution {
public:
    vector<int> plusOne(vector<int>& digits) {
        for (int i = digits.size() - 1; i >= 0; i--) {
            if (digits[i] < 9) {
                digits[i]++;
                return digits;
            }

            digits[i] = 0;
        }

        digits.insert(digits.begin(), 1);

        return digits;
    }
};
```

---

## Quick Revision

```text
Plus One
    ↓
Start from right
    ↓
Is digit < 9?
    ├── YES → increment → return
    │
    └── NO → make 0 → carry left
                    ↓
              continue
```

### Core Template

```text
for i = n - 1 → 0:
    if digits[i] < 9:
        digits[i]++
        return digits

    digits[i] = 0

insert 1 at beginning
return digits
```

### Key Insight

```text
[9,9,9]
   ↓
all digits become 0
   ↓
carry remains
   ↓
add 1 at front
   ↓
[1,0,0,0]
```

**Pattern to remember:**
**Plus One → Right-to-left traversal → Carry propagation → `9 → 0`, otherwise increment and stop**
