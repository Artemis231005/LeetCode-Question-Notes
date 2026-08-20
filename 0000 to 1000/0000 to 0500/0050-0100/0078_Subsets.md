# LeetCode 78 — Subsets

## Metadata

* **LeetCode:** 78
* **Problem:** Subsets
* **Difficulty:** Medium
* **Topics:** Array, Backtracking, Bit Manipulation
* **Pattern:** Subsets / Power Set
* **Key Technique:** Backtracking — Choose / Explore / Undo
* **Key Pattern:** Include or exclude each element
* **Key Template:** Backtracking
* **Optimal Complexity:** `O(n × 2^n)` time, `O(n)` auxiliary space (excluding output)

---

## Problem

Given an integer array `nums` of **unique elements**, return all possible subsets.

The solution set must not contain duplicate subsets.

For an array of size `n`, there are:

```text
2^n
```

possible subsets, including the empty subset.

Example:

```text
nums = [1, 2, 3]

Output:
[
  [],
  [1],
  [2],
  [1,2],
  [3],
  [1,3],
  [2,3],
  [1,2,3]
]
```

---

## Approach 1 — Backtracking

### Idea

For every element, we have exactly **two choices**:

```text
1. Include the element
2. Exclude the element
```

This creates a binary decision tree.

For:

```text
[1, 2, 3]
```

we can visualize it as:

```text
                    []
                 /      \
              [1]        []
             /   \      /   \
         [1,2]   [1]  [2]   []
          / \     / \   / \  / \
      [1,2,3] [1,2] [1,3] [1] [2,3] [2] [3] []
```

Every node represents one valid subset.

### Dry Run

For:

```text
nums = [1, 2]
```

Start:

```text
path = []
```

#### Choose `1`

```text
path = [1]
```

Choose `2`:

```text
path = [1,2]
```

Add it.

Undo `2`:

```text
path = [1]
```

Skip `2`:

```text
path = [1]
```

Add it.

Undo `1`:

```text
path = []
```

Skip `1`.

Choose `2`:

```text
path = [2]
```

Add it.

Skip `2`:

```text
path = []
```

Add empty subset.

Final:

```text
[
  [1,2],
  [1],
  [2],
  []
]
```

The order of subsets does not matter.

### Algorithm

1. Create an empty `path` to store the current subset.
2. Start backtracking from index `0`.
3. At every recursive call:

   * Add the current `path` to the answer.
4. Iterate from `start` to `n - 1`.
5. Choose `nums[i]` by adding it to `path`.
6. Recursively generate subsets starting from `i + 1`.
7. Remove `nums[i]` from `path` to undo the choice.
8. Continue with the next element.

### Complexity

There are `2^n` subsets.

Copying each subset can take up to `O(n)`:

* **Time:** `O(n × 2^n)`
* **Auxiliary Space:** `O(n)` recursion + current subset
* **Output Space:** `O(n × 2^n)`

### Notes / Tips

* The empty subset is automatically included because we add `path` before making any choices.
* `start` prevents using the same element multiple times and maintains the original order.
* The key backtracking pattern is:

  ```text
  choose
  → recurse
  → undo
  ```
* Since every element is unique, no duplicate-checking is required.
* For **Subsets II (LeetCode 90)**, duplicates exist, so an additional duplicate-skipping condition is required.

### Code

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> path;

    void backtrack(vector<int>& nums, int start) {
        ans.push_back(path);

        for (int i = start; i < nums.size(); i++) {
            path.push_back(nums[i]);

            backtrack(nums, i + 1);

            path.pop_back();
        }
    }

    vector<vector<int>> subsets(vector<int>& nums) {
        backtrack(nums, 0);
        return ans;
    }
};
```

---

## Approach 2 — Bit Manipulation

### Idea

Each element has two possibilities:

```text
0 → don't include
1 → include
```

So each subset can be represented by an `n`-bit number.

For:

```text
nums = [1, 2, 3]
```

we have `n = 3`, so masks range from:

```text
000 → []
001 → [1]
010 → [2]
011 → [1,2]
100 → [3]
101 → [1,3]
110 → [2,3]
111 → [1,2,3]
```

There are exactly:

```text
2^n
```

masks.

### Dry Run

For:

```text
nums = [1, 2, 3]
```

Take mask:

```text
101
```

Check every bit:

```text
bit 0 → 1 → include nums[0] = 1
bit 1 → 0 → skip nums[1] = 2
bit 2 → 1 → include nums[2] = 3
```

Therefore:

```text
101 → [1,3]
```

Another example:

```text
mask = 110
```

```text
bit 0 → 0 → skip 1
bit 1 → 1 → include 2
bit 2 → 1 → include 3
```

Therefore:

```text
110 → [2,3]
```

### Algorithm

1. Let `n = nums.size()`.
2. There are `2^n` possible subsets.
3. Iterate through every mask from `0` to `(1 << n) - 1`.
4. Create an empty subset.
5. For every bit position `i`:

   * Check whether bit `i` is set:

     ```cpp
     mask & (1 << i)
     ```
   * If set, include `nums[i]`.
6. Add the generated subset to the answer.

### Complexity

* **Time:** `O(n × 2^n)`
* **Auxiliary Space:** `O(n)` for the current subset
* **Output Space:** `O(n × 2^n)`

### Notes / Tips

* `1 << n` means `2^n`.
* Bit `i` represents whether `nums[i]` is included.
* The key expression is:

  ```cpp
  mask & (1 << i)
  ```
* This approach is iterative and avoids recursion.
* It is especially useful when you want to understand the connection between **subsets and binary representation**.

### Code

```cpp
class Solution {
public:
    vector<vector<int>> subsets(vector<int>& nums) {
        int n = nums.size();
        vector<vector<int>> ans;

        for (int mask = 0; mask < (1 << n); mask++) {
            vector<int> subset;

            for (int i = 0; i < n; i++) {
                if (mask & (1 << i)) {
                    subset.push_back(nums[i]);
                }
            }

            ans.push_back(subset);
        }

        return ans;
    }
};
```

---

## Key Takeaway

For subsets, remember:

```text
Every element → 2 choices
               ↓
        Include / Exclude
               ↓
          2^n subsets
```

### Backtracking Template

```cpp
path.push_back(nums[i]);

backtrack(i + 1);

path.pop_back();
```

The most important backtracking pattern is:

```text
Choose
  ↓
Explore
  ↓
Undo
```

**Pattern:**

> Subsets = binary decisions / choose-or-skip + backtracking.

**Quick recognition:**
If a problem asks for **all possible combinations where each element can either be selected or not selected**, think **Subsets → Backtracking or Bitmasking**.
