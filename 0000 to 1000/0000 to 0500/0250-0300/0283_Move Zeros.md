# 283. Move Zeroes

## Metadata

* **Topic:** Array, Two Pointer
* **Difficulty:** Easy
* **Key Pattern:** Two Pointer — In-place Compaction
* **Key Template:** Move valid elements forward, then fill the remaining positions with zeroes
* **Goal:** Move all `0`s to the end while maintaining the relative order of non-zero elements.

---

## Approach 1: Two Pointer — In-place

### Idea

Use a pointer `j` to track the position where the next non-zero element should be placed.

Traverse the array using `i`:

* If `nums[i]` is non-zero, swap it with `nums[j]`.
* Increment `j`.

This automatically moves zeroes toward the end while preserving the order of non-zero elements.

### Dry Run

`nums = [0, 1, 0, 3, 12]`

| `i` | Array          | `j` |
| --- | -------------- | --- |
| 0   | `[0,1,0,3,12]` | 0   |
| 1   | `[1,0,0,3,12]` | 1   |
| 2   | `[1,0,0,3,12]` | 1   |
| 3   | `[1,3,0,0,12]` | 2   |
| 4   | `[1,3,12,0,0]` | 3   |

Final: `[1,3,12,0,0]`

### Algorithm

1. Initialize `j = 0`.
2. Traverse the array using `i`.
3. If `nums[i] != 0`:

   * Swap `nums[i]` and `nums[j]`.
   * Increment `j`.
4. Continue until the array ends.
5. The non-zero elements are now at the beginning and all zeroes are at the end.

### Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

### Notes / Tips

* This is an **in-place** problem, so do not create another array.
* `j` represents the position where the next non-zero element belongs.
* Relative order of non-zero elements is preserved.
* Swapping is safe even when `i == j`.
* This is essentially an **in-place stable compaction** technique.
* Similar pattern appears in problems where unwanted elements must be pushed to the end.

### Code

```cpp
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int j = 0;

        for (int i = 0; i < nums.size(); i++) {
            if (nums[i] != 0) {
                swap(nums[i], nums[j]);
                j++;
            }
        }
    }
};
```

---

## Key Template

```cpp
int j = 0;

for (int i = 0; i < nums.size(); i++) {
    if (nums[i] != 0) {
        swap(nums[i], nums[j]);
        j++;
    }
}
```

### Pattern to Remember

```text
i → scans every element
j → position for the next valid/non-zero element

if current element is valid:
    put it at j
    move j forward
```
