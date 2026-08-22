# 268. Missing Number

## Metadata

* **Topic:** Array, Bit Manipulation
* **Difficulty:** Easy
* **Pattern:** XOR
* **Key Pattern:** XOR all numbers with `0...n`; equal numbers cancel.

---

## Idea

We are given `n` distinct numbers from the range:

```text
0 to n
```

Exactly one number is missing.

Use XOR because:

```text
x ^ x = 0
x ^ 0 = x
```

XOR all numbers from `0` to `n` and all elements of the array.

Every present number appears twice and cancels, leaving only the missing number.

Example:

```text
nums = [3, 0, 1]
n = 3
```

```text
(0 ^ 1 ^ 2 ^ 3) ^ (3 ^ 0 ^ 1)
```

Everything cancels except:

```text
2
```

---

## Dry Run

```text
nums = [3, 0, 1]
n = 3
```

Initialize:

```text
ans = n = 3
```

Process indices and values:

```text
i = 0 → ans = 3 ^ 0 ^ 3
i = 1 → ans = 3 ^ 0 ^ 3 ^ 1 ^ 0
i = 2 → ans = ...
```

After cancellation:

```text
ans = 2
```

Therefore, the missing number is:

```text
2
```

---

## Algorithm

1. Initialize `ans = n`.
2. For every index `i`:

   * XOR `ans` with `i`.
   * XOR `ans` with `nums[i]`.
3. Return `ans`.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

---

## Code

```cpp
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        int n = nums.size();
        int ans = n;

        for (int i = 0; i < n; i++) {
            ans ^= i;
            ans ^= nums[i];
        }

        return ans;
    }
};
```

---

## Notes / Tips

* The array contains numbers from `0` to `n`, so there are `n + 1` possible values.
* Exactly one value is missing.
* XOR avoids the overflow problem that can occur with the sum approach.
* The key cancellation property is:

```text
a ^ a = 0
```

* Another valid approach is using the formula:

```text
sum(0...n) = n * (n + 1) / 2
```

but XOR is safer because it avoids integer overflow.

---

## Key Template

```text
ans = n

for i = 0 to n - 1:
    ans ^= i
    ans ^= nums[i]

return ans
```
