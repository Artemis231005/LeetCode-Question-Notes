# 201. Bitwise AND of Numbers Range

## Metadata

* **Topic:** Bit Manipulation
* **Difficulty:** Medium
* **Pattern:** Common Prefix of Binary Numbers
* **Key Pattern:** Remove the differing rightmost bits until `left == right`.

---

## Idea

We need to find:

```text
left & (left + 1) & ... & right
```

Doing AND for every number is inefficient.

### Key Observation

In a range of numbers, any bit that **changes** within the range will eventually become `0` after performing AND.

Therefore, only the **common binary prefix** of `left` and `right` remains.

Example:

```text
left  = 5  → 101
right = 7  → 111
```

Common prefix:

```text
10
```

So:

```text
101
110
111
---
100
```

Answer = `4`.

We can repeatedly remove the lowest set bit from `right` until:

```text
left == right
```

---

## Dry Run

```text
left = 5
right = 7
```

Binary:

```text
left  = 101
right = 111
```

### Step 1

```text
right = right & (right - 1)

111 & 110
= 110
```

Now:

```text
left  = 101
right = 110
```

### Step 2

```text
110 & 101
= 100
```

Now:

```text
left  = 100
right = 100
```

Therefore:

```text
answer = 100 = 4
```

---

## Algorithm

1. While `left != right`:

   * Remove the lowest set bit of `right`:

     ```text
     right = right & (right - 1)
     ```
2. When `left == right`, return `right`.

---

## Complexity

* **Time:** `O(log n)` in the worst case.
* **Space:** `O(1)`

---

## Notes / Tips

### Most Important Trick

```text
n & (n - 1)
```

removes the **rightmost `1` bit**.

Here, we use it to eliminate bits from `right` that cannot survive the AND across the entire range.

Another way to think about the problem:

```text
Only the common binary prefix survives.
Everything after the first differing bit becomes 0.
```

### Key Template

```text
while (left != right):
    right = right & (right - 1)

return right
```

---

## Code

```cpp
class Solution {
public:
    int rangeBitwiseAnd(int left, int right) {
        while (left != right) {
            right = right & (right - 1);
        }

        return right;
    }
};
```
