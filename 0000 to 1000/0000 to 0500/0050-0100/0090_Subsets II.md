# LeetCode 90 — Subsets II

## Metadata

* **LeetCode:** 90
* **Problem:** Subsets II
* **Difficulty:** Medium
* **Topics:** Array, Backtracking, Bit Manipulation
* **Pattern:** Backtracking with Duplicate Skipping
* **Key Technique:** Sort + Skip duplicates at the same recursion level
* **Key Pattern:** Subsets + Duplicate Handling
* **Key Template:** Backtracking
* **Optimal Complexity:** `O(n × 2^n)` time, `O(n)` auxiliary space (excluding output)

---

## Problem

Given an integer array `nums` that **may contain duplicates**, return all possible subsets.

The solution must **not contain duplicate subsets**.

Example:

```text id="5u4y0e"
nums = [1,2,2]
```

Output:

```text id="n4op6k"
[
  [],
  [1],
  [1,2],
  [1,2,2],
  [2],
  [2,2]
]
```

Notice that `[2]` appears only once even though there are two `2`s in the input.

---

## Approach — Backtracking + Sorting

### Idea

This is almost the same as **LeetCode 78 — Subsets**.

The only new problem is **duplicates**.

First, sort the array:

```text id="h7z4m2"
[1,2,2]
```

Now duplicate elements are adjacent.

During backtracking, if two equal elements appear at the **same recursion level**, skip the later one:

```cpp id="w7k7aa"
if (i > start && nums[i] == nums[i - 1]) {
    continue;
}
```

### Why `i > start`?

This is extremely important.

We want to skip duplicates only when they are being considered as **alternative choices at the same level**.

But we must still allow duplicate values to be selected at deeper levels.

For:

```text id="qv5r4v"
[1,2,2]
```

At the first level:

```text id="r3l4gt"
        []
       /  \
      1    2
           \
            2
```

The second `2` should not create another identical `[2]`.

But after choosing the first `2`, we **must** be allowed to choose the second `2` to create:

```text id="0d5j8k"
[2,2]
```

Therefore:

```cpp id="7d2s0k"
i > start
```

means:

> Skip a duplicate only if it is another choice at the same recursion level.

### Dry Run

Consider:

```text id="fhxk4n"
nums = [1,2,2]
```

After sorting:

```text id="b0w1jx"
[1,2,2]
```

Start with:

```text id="b8q9yx"
path = []
```

Add empty subset:

```text id="6rh6yh"
[]
```

#### Choose `1`

```text id="u7m0q3"
path = [1]
```

Continue.

Choose first `2`:

```text id="4l3v8h"
path = [1,2]
```

Choose second `2`:

```text id="9kj2xp"
path = [1,2,2]
```

Backtrack.

At the same level, the second `2` is skipped where appropriate because:

```text id="e0e7cf"
nums[i] == nums[i-1]
```

Now backtrack to the empty path.

#### Choose first `2`

```text id="w8j2l4"
path = [2]
```

Choose the next `2`:

```text id="7uk1xr"
path = [2,2]
```

This is allowed because the second `2` is at a **deeper recursion level**.

Final subsets:

```text id="3gy0ef"
[]
[1]
[1,2]
[1,2,2]
[2]
[2,2]
```

### Algorithm

1. Sort `nums`.
2. Start backtracking from index `0`.
3. At every recursive call:

   * Add the current `path` to the answer.
4. Iterate from `start` to `n - 1`.
5. If:

   ```cpp
   i > start && nums[i] == nums[i - 1]
   ```

   skip the current element.
6. Otherwise:

   * Add `nums[i]` to `path`.
   * Recursively call:

     ```cpp
     backtrack(i + 1)
     ```
   * Remove `nums[i]` from `path`.
7. Continue until all possibilities have been explored.

### Complexity

There can be at most `2^n` subsets.

Each subset may require up to `O(n)` time to copy.

* **Time:** `O(n × 2^n)`
* **Auxiliary Space:** `O(n)` recursion + current subset
* **Output Space:** `O(n × 2^n)`

Sorting takes:

```text id="4glp8f"
O(n log n)
```

which is dominated by the subset generation for typical analysis.

### Notes / Tips

* **Sort first.** Duplicate skipping depends on equal values being adjacent.
* The most important line is:

  ```cpp
  if (i > start && nums[i] == nums[i - 1])
  ```
* `i > start` means:

  ```text
  same recursion level
  ```
* Do **not** use only:

  ```cpp
  nums[i] == nums[i - 1]
  ```

  because that would incorrectly prevent `[2,2]`.
* Compare this with **LeetCode 78**:

  * 78 → unique elements → no duplicate handling.
  * 90 → duplicates allowed → sort + skip duplicates.
* Duplicate skipping happens **before choosing** the element.
* The standard backtracking structure remains:

  ```text
  Choose
  → Explore
  → Undo
  ```

### Code

```cpp id="zj0f5a"
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;

    void backtrack(vector<int>& nums, int start) {
        ans.push_back(path);

        for (int i = start; i < nums.size(); i++) {
            if (i > start && nums[i] == nums[i - 1]) {
                continue;
            }

            path.push_back(nums[i]);

            backtrack(nums, i + 1);

            path.pop_back();
        }
    }

    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        sort(nums.begin(), nums.end());

        backtrack(nums, 0);

        return ans;
    }
};
```

---

## Key Takeaway

### The One Line to Remember

```cpp
if (i > start && nums[i] == nums[i - 1]) {
    continue;
}
```

Meaning:

```text
Same value
   +
Same recursion level
   ↓
Skip
```

But:

```text
Same value
   +
Different/deeper recursion level
   ↓
Allow
```

### 78 vs 90

```text
LeetCode 78:
Unique elements
→ Normal backtracking

LeetCode 90:
Duplicates allowed
→ Sort
→ Backtracking
→ Skip duplicates at same level
```

**Pattern:**

> For combinations/subsets with duplicate input, sort first and skip equal elements only at the same recursion level.
