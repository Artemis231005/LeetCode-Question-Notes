# 189. Rotate Array

## Metadata

* **Topic:** Array
* **Difficulty:** Medium
* **Pattern:** Array Reversal
* **Key Pattern:** Reverse the whole array, then reverse the two parts.

---

## Idea

We need to rotate the array to the **right by `k` positions**.

Example:

```text
nums = [1,2,3,4,5,6,7], k = 3
```

Answer:

```text
[5,6,7,1,2,3,4]
```

Use the **reversal technique**:

1. Reverse the entire array.
2. Reverse the first `k` elements.
3. Reverse the remaining elements.

First:

```text
[7,6,5,4,3,2,1]
```

Reverse first `3`:

```text
[5,6,7,4,3,2,1]
```

Reverse remaining:

```text
[5,6,7,1,2,3,4]
```

---

## Dry Run

```text
nums = [1,2,3,4,5,6,7]
k = 3
```

```text
Reverse all
→ [7,6,5,4,3,2,1]

Reverse first k
→ [5,6,7,4,3,2,1]

Reverse remaining
→ [5,6,7,1,2,3,4]
```

---

## Algorithm

1. Set:

   ```text
   k = k % n
   ```

   to handle `k > n`.
2. Reverse the entire array.
3. Reverse indices `[0, k-1]`.
4. Reverse indices `[k, n-1]`.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

---

## Notes / Tips

* Always do:

  ```text
  k %= n
  ```
* For **right rotation**:

  ```text
  Reverse all
  Reverse first k
  Reverse remaining
  ```
* This achieves the required **in-place** rotation.
* Alternative approaches using an extra array take `O(n)` space.

### Key Template

```text
reverse(0, n-1)
reverse(0, k-1)
reverse(k, n-1)
```

---

## Code

```cpp
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k %= n;

        reverse(nums.begin(), nums.end());
        reverse(nums.begin(), nums.begin() + k);
        reverse(nums.begin() + k, nums.end());
    }
};
```
