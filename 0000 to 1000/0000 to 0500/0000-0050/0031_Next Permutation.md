# LeetCode 31 — Next Permutation

## Metadata

* **LeetCode:** 31
* **Problem:** Next Permutation
* **Difficulty:** Medium
* **Topics:** Array, Two Pointers
* **Pattern:** Lexicographical Permutation
* **Key Technique:** Find Pivot → Find Successor → Reverse Suffix
* **Key Pattern:** Greedy + Two Pointers
* **Optimal Complexity:** `O(n)` time, `O(1)` space

---

## Problem

Given an array of integers `nums`, rearrange it into the **next lexicographically greater permutation**.

If the current permutation is already the largest possible permutation, rearrange it into the **smallest possible permutation**.

The modification must be done **in-place**.

### Example

```text
nums = [1, 2, 3]

Permutations in lexicographical order:

[1, 2, 3]
[1, 3, 2]  ← next permutation
[2, 1, 3]
[2, 3, 1]
[3, 1, 2]
[3, 2, 1]
```

Therefore:

```text
[1, 2, 3] → [1, 3, 2]
```

---

# Approach 1 — Generate All Permutations

## Idea

One possible approach is to generate every permutation, sort them lexicographically, and find the permutation immediately after the current one.

However, this is extremely inefficient because there can be `n!` permutations.

This approach is mainly useful for understanding why we need a smarter solution.

## Dry Run

For:

```text
nums = [1, 2, 3]
```

Generate:

```text
[1, 2, 3]
[1, 3, 2]
[2, 1, 3]
[2, 3, 1]
[3, 1, 2]
[3, 2, 1]
```

Current permutation:

```text
[1, 2, 3]
```

Next permutation:

```text
[1, 3, 2]
```

## Algorithm

1. Generate all permutations.
2. Sort them lexicographically.
3. Find the current permutation.
4. Return the permutation immediately after it.
5. If it is the last permutation, return the first permutation.

## Complexity

* **Time:** `O(n! × n)`
* **Space:** `O(n! × n)`

## Notes / Tips

* This is not a practical solution.
* The actual problem can be solved in `O(n)` time.
* We need to identify what changes between two consecutive lexicographical permutations.

## Code

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        // Not suitable for the actual constraints.
        // Brute-force permutation generation would be inefficient.
    }
};
```

---

# Approach 2 — Find the Pivot, Swap, Reverse

## Idea

The key observation is that the next permutation should be **only slightly larger** than the current permutation.

Consider:

```text
[1, 2, 3, 6, 5, 4]
```

The suffix:

```text
[6, 5, 4]
```

is in decreasing order.

There is no way to make this suffix larger by rearranging it because it is already the largest permutation of those elements.

So we move from right to left and find the first position where:

```text
nums[i] < nums[i + 1]
```

This position is called the **pivot**.

For:

```text
[1, 2, 3, 6, 5, 4]
       ↑
     pivot
```

The pivot is `3`.

Now we need to:

1. Find the smallest element greater than `3` in the suffix.
2. Swap it with `3`.
3. Reverse the suffix to make it as small as possible.

This gives the smallest permutation that is still greater than the original one.

## Dry Run

Consider:

```text
nums = [1, 2, 3, 6, 5, 4]
```

### Step 1 — Find Pivot

Start from the right:

```text
6 > 5
5 > 4
```

The suffix is decreasing.

Now:

```text
3 < 6
```

Therefore:

```text
pivot = index 2
pivot value = 3
```

Array:

```text
[1, 2, 3, 6, 5, 4]
       ↑
```

### Step 2 — Find Successor

Find the first element from the right that is greater than `3`.

```text
4 > 3
```

So swap `3` and `4`:

```text
[1, 2, 4, 6, 5, 3]
```

### Step 3 — Reverse the Suffix

The suffix after the pivot is:

```text
[6, 5, 3]
```

Reverse it:

```text
[3, 5, 6]
```

Final result:

```text
[1, 2, 4, 3, 5, 6]
```

This is the next lexicographically greater permutation.

## Algorithm

1. Start from the second-last element.
2. Move from right to left until finding:

   ```cpp
   nums[i] < nums[i + 1]
   ```
3. Store this index as `pivot`.
4. If no pivot exists:

   * The entire array is in decreasing order.
   * Reverse the complete array.
   * Return.
5. Starting from the end, find the first element greater than `nums[pivot]`.
6. Swap the pivot with this element.
7. Reverse everything after the pivot.
8. The array is now the next permutation.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

### Why do we find the pivot from the right?

We want the **smallest possible increase**.

The suffix after the pivot is in decreasing order, meaning it is already the largest arrangement of those elements.

### Why find the successor from the right?

Because the suffix is decreasing.

The first element from the right that is greater than the pivot is the **smallest element greater than the pivot**.

Therefore, swapping with it produces the smallest possible increase.

### Why reverse the suffix?

After the swap, the suffix is still in decreasing order.

We want the smallest possible arrangement after increasing the pivot.

The smallest arrangement is ascending order, so we reverse the decreasing suffix.

### Important Edge Case

If:

```text
nums = [3, 2, 1]
```

there is no index where:

```text
nums[i] < nums[i + 1]
```

The array is already the largest permutation.

Therefore, reverse the entire array:

```text
[3, 2, 1]
     ↓
[1, 2, 3]
```

## Code

```cpp
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int n = nums.size();

        // Step 1: Find the pivot
        int pivot = -1;

        for (int i = n - 2; i >= 0; i--) {
            if (nums[i] < nums[i + 1]) {
                pivot = i;
                break;
            }
        }

        // No pivot means the array is the largest permutation
        if (pivot == -1) {
            reverse(nums.begin(), nums.end());
            return;
        }

        // Step 2: Find the smallest element greater than pivot
        for (int i = n - 1; i > pivot; i--) {
            if (nums[i] > nums[pivot]) {
                swap(nums[i], nums[pivot]);
                break;
            }
        }

        // Step 3: Reverse the suffix
        reverse(nums.begin() + pivot + 1, nums.end());
    }
};
```

---

# Key Takeaway

The entire problem can be remembered as:

```text
1. Find Pivot
       ↓
   nums[i] < nums[i + 1]

2. Find Successor
       ↓
   Smallest element > pivot

3. Swap
       ↓

4. Reverse Suffix
       ↓
   Make suffix smallest
```

### Key Template

```cpp
int pivot = -1;

for (int i = n - 2; i >= 0; i--) {
    if (nums[i] < nums[i + 1]) {
        pivot = i;
        break;
    }
}

if (pivot == -1) {
    reverse(nums.begin(), nums.end());
    return;
}

for (int i = n - 1; i > pivot; i--) {
    if (nums[i] > nums[pivot]) {
        swap(nums[i], nums[pivot]);
        break;
    }
}

reverse(nums.begin() + pivot + 1, nums.end());
```

**Key Pattern:** Find the longest decreasing suffix → increase the element just before it → minimize the suffix.

**Remember:**
**Pivot → Successor → Swap → Reverse.**

This is a classic **greedy + two-pointer/in-place array manipulation** pattern.
