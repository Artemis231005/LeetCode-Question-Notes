# LeetCode 136 — Single Number

## Metadata

* **LeetCode:** 136
* **Problem:** Single Number
* **Difficulty:** Easy
* **Topics:** Array, Bit Manipulation
* **Pattern:** XOR
* **Key Technique:** XOR cancellation
* **Key Pattern:** Duplicate cancellation using XOR
* **Key Template:** `ans ^= num`
* **Optimal Complexity:** `O(n)`

---

## Problem

Given a **non-empty** array of integers `nums`, every element appears **twice** except for one element which appears exactly once.

Find the element that appears only once.

The solution must:

* Run in `O(n)` time.
* Use `O(1)` extra space.

Example:

```text
nums = [2, 2, 1]
```

Answer:

```text
1
```

Another example:

```text
nums = [4, 1, 2, 1, 2]
```

Answer:

```text
4
```

---

## Idea

Use the **XOR (`^`)** operator.

The important XOR properties are:

```text
a ^ a = 0
a ^ 0 = a
a ^ b ^ a = b
```

Since every number except one appears exactly twice, the duplicate numbers cancel each other:

```text
2 ^ 2 = 0
1 ^ 1 = 0
```

The only number left is the number that appears once.

For:

```text
[4, 1, 2, 1, 2]
```

we calculate:

```text
0 ^ 4 ^ 1 ^ 2 ^ 1 ^ 2
```

Rearranging XOR:

```text
(1 ^ 1) ^ (2 ^ 2) ^ 4
```

Therefore:

```text
0 ^ 0 ^ 0 ^ 4 = 4
```

Answer:

```text
4
```

---

## Dry Run

Consider:

```text
nums = [4, 1, 2, 1, 2]
```

Initialize:

```text
ans = 0
```

Process each element:

```text
ans = 0 ^ 4 = 4

ans = 4 ^ 1 = 5

ans = 5 ^ 2 = 7

ans = 7 ^ 1 = 6

ans = 6 ^ 2 = 4
```

Final:

```text
ans = 4
```

Therefore:

```text
4
```

is the single number.

---

## Algorithm

1. Initialize `ans = 0`.
2. Traverse every number in the array.
3. XOR the current number with `ans`:

   ```cpp
   ans ^= num;
   ```
4. Duplicate numbers cancel each other.
5. The remaining value is the number appearing once.
6. Return `ans`.

---

## Complexity

* **Time:** `O(n)`

  * Traverse the array once.
* **Space:** `O(1)`

  * Only one variable is used.

---

## Notes / Tips

* XOR is **commutative**:

  ```text
  a ^ b = b ^ a
  ```
* XOR is **associative**:

  ```text
  (a ^ b) ^ c = a ^ (b ^ c)
  ```
* Therefore, the order of elements does not matter.
* Every duplicate pair becomes `0`.
* XOR with `0` leaves the number unchanged.
* This avoids using a hash map or sorting.
* This solution works with negative integers as well.

### Why Not Use a Hash Map?

A frequency map would work:

```text
number → frequency
```

but would require `O(n)` extra space.

The XOR approach satisfies the required `O(1)` space.

### Key Trick

Whenever a problem has:

```text
Every element appears twice
+
One element appears once
```

think:

```text
XOR
```

---

## Code

```cpp
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int ans = 0;

        for (int num : nums) {
            ans ^= num;
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

    for (int num : nums) {
        ans ^= num;
    }

    return ans;
}
```

### Reusable Pattern

```text
Initialize ans = 0
        ↓
Traverse every element
        ↓
ans = ans XOR current
        ↓
Duplicates cancel
        ↓
Unique element remains
```

### XOR Properties to Remember

```text
x ^ x = 0
x ^ 0 = x
x ^ y ^ x = y
```
