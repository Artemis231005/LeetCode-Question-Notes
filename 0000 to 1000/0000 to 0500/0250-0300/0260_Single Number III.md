# 260. Single Number III

## Metadata

* **Topic:** Bit Manipulation
* **Difficulty:** Medium
* **Pattern:** XOR + Bit Partitioning
* **Key Pattern:** XOR gives `a ^ b`; use a differing bit to separate `a` and `b`.

---

## Idea

Every number appears **twice**, except for exactly two numbers that appear once.

Example:

```text
nums = [1, 2, 1, 3, 2, 5]
```

The unique numbers are:

```text
3 and 5
```

### Step 1: XOR everything

Pairs cancel because:

```text
x ^ x = 0
```

So:

```text
1 ^ 2 ^ 1 ^ 3 ^ 2 ^ 5
= 3 ^ 5
```

Let:

```text
xorAll = a ^ b
```

Since `a != b`, `xorAll` must contain at least one set bit.

### Step 2: Find a differing bit

Use:

```text
xorAll & -xorAll
```

This extracts the **rightmost set bit**.

This bit is:

* `1` in one of `a` or `b`
* `0` in the other

So it can separate the two unique numbers into different groups.

### Step 3: XOR each group

Partition numbers based on that bit:

```text
bit = 1 → group 1
bit = 0 → group 2
```

All duplicate numbers still cancel within their group.

The two remaining values are `a` and `b`.

---

## Dry Run

```text
nums = [1, 2, 1, 3, 2, 5]
```

XOR everything:

```text
xorAll = 3 ^ 5
       = 6
       = 110
```

Find rightmost set bit:

```text
110 & 010
= 010
```

So:

```text
mask = 010
```

Partition using this bit.

Numbers with bit `1`:

```text
2, 3, 2, 5
```

Numbers with bit `0`:

```text
1, 1
```

XOR each group:

```text
2 ^ 3 ^ 2 ^ 5 = 3 ^ 5 = 6
```

This simplified grouping explanation isn't sufficient by itself because the chosen bit must separate the two unique numbers; here `3` and `5` are both `1` at bit `1`. Instead, use the actual rightmost set bit of `3 ^ 5 = 6`, which is `2`, and note:

```text
3 = 011
5 = 101
      ↑
```

They differ at the `2` bit, so they fall into different groups:

```text
3 → bit is 1
5 → bit is 0
```

After XORing each group, we get:

```text
3 and 5
```

---

## Algorithm

1. XOR all numbers to get:

   ```text
   xorAll = a ^ b
   ```
2. Extract a differing bit:

   ```text
   mask = xorAll & (-xorAll)
   ```
3. XOR numbers into two groups based on `mask`.
4. Return the two resulting XOR values.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

---

## Code

```cpp
class Solution {
public:
    vector<int> singleNumber(vector<int>& nums) {
        int xorAll = 0;

        for (int num : nums) {
            xorAll ^= num;
        }

        int mask = xorAll & (-xorAll);

        int a = 0;
        int b = 0;

        for (int num : nums) {
            if (num & mask) {
                a ^= num;
            }
            else {
                b ^= num;
            }
        }

        return {a, b};
    }
};
```

---

## Notes / Tips

### Core XOR Properties

```text
x ^ x = 0
x ^ 0 = x
```

Therefore, duplicate numbers cancel each other.

### Why Partition?

After XORing everything:

```text
xorAll = a ^ b
```

We need to separate `a` and `b`.

A set bit in `a ^ b` represents a position where `a` and `b` differ.

```text
mask = xorAll & (-xorAll)
```

extracts one such position.

### Important

Do **not** simply XOR everything and return it.

That only gives:

```text
a ^ b
```

We need to use a differing bit to recover `a` and `b` individually.

---

## Key Template

```text
xorAll = XOR of all numbers

mask = xorAll & (-xorAll)

a = 0
b = 0

for each num:
    if num & mask:
        a ^= num
    else:
        b ^= num

return {a, b}
```
