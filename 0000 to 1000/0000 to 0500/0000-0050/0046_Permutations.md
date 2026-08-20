# LeetCode 46 — Permutations

## Metadata

* **LeetCode:** 46
* **Problem:** Permutations
* **Difficulty:** Medium
* **Topics:** Array, Backtracking
* **Pattern:** Backtracking, Permutation Generation
* **Key Pattern:** Choose an unused element at every position
* **Key Technique:** `used[]` array + Backtracking
* **Optimal Complexity:** `O(n × n!)` time, `O(n)` auxiliary space
* **Key Template:** Choose → Recurse → Undo

---

## Problem

Given an array `nums` containing **distinct integers**, return all possible permutations.

A permutation is an arrangement of all elements where:

* Every element appears exactly once.
* The order of elements matters.

Example:

```text
nums = [1,2,3]

Output:
[
    [1,2,3],
    [1,3,2],
    [2,1,3],
    [2,3,1],
    [3,1,2],
    [3,2,1]
]
```

For `n` distinct elements, there are:

```text
n!
```

possible permutations.

---

## Approach — Backtracking with `used[]`

### Idea

We construct the permutation **one position at a time**.

At every recursion level:

1. Try every element.
2. If the element has not been used, choose it.
3. Recursively fill the next position.
4. After returning, remove the element so it can be used in another permutation.

For:

```text
nums = [1,2,3]
```

the decision tree starts as:

```text
             []
          /   |   \
        [1]  [2]  [3]
        / \   / \   / \
    [1,2] [1,3] ...
```

Each level represents **one position** in the permutation.

---

## Why Do We Need `used[]`?

Unlike Combination Sum, we cannot simply use a `start` index.

For permutations, after choosing an element, we can choose **any unused element** next.

For example:

```text
[1]
```

The next element can be:

```text
2 or 3
```

Therefore, we need to know which elements are already present in the current permutation.

```cpp
used[i] = true;
```

means:

```text
nums[i] is already in current
```

After backtracking:

```cpp
used[i] = false;
```

so it becomes available again.

---

## Dry Run

Consider:

```text
nums = [1,2,3]
```

Initially:

```text
current = []
used = [false,false,false]
```

### Choose `1`

```text
current = [1]
used = [true,false,false]
```

Now try unused elements.

Choose `2`:

```text
current = [1,2]
used = [true,true,false]
```

Choose `3`:

```text
current = [1,2,3]
```

Size becomes `3`, so we add:

```text
[1,2,3]
```

### Backtrack

Remove `3`:

```text
current = [1,2]
used = [true,true,false]
```

No more choices.

Backtrack again:

```text
current = [1]
used = [true,false,false]
```

Now choose `3`:

```text
current = [1,3]
used = [true,false,true]
```

Then choose `2`:

```text
[1,3,2]
```

Continue similarly.

Final result:

```text
[
    [1,2,3],
    [1,3,2],
    [2,1,3],
    [2,3,1],
    [3,1,2],
    [3,2,1]
]
```

---

## Recursion Tree

For `[1,2,3]`:

```text
                         []
                    /     |     \
                  1       2       3
                /  \     /  \     /  \
              1,2  1,3 2,1  2,3 3,1  3,2
               |    |    |    |    |    |
             1,2,3 1,3,2 2,1,3 2,3,1 3,1,2 3,2,1
```

Every root-to-leaf path contains every element exactly once.

---

### Algorithm

1. Create an empty `current` permutation.
2. Create a `used` array initialized to `false`.
3. Start backtracking.
4. If `current.size() == nums.size()`:

   * Add `current` to the answer.
   * Return.
5. Iterate through every index `i`.
6. If `used[i]` is `true`, skip it.
7. Mark `used[i] = true`.
8. Add `nums[i]` to `current`.
9. Recursively build the remaining positions.
10. Remove `nums[i]` from `current`.
11. Set `used[i] = false`.
12. Continue with the next unused element.

---

### Complexity

There are:

```text
n!
```

permutations.

Each permutation contains `n` elements, so copying each result costs `O(n)`.

Therefore:

* **Time:** `O(n × n!)`
* **Auxiliary Space:** `O(n)` for recursion + `used`
* **Output Space:** `O(n × n!)`

If output storage is included, total space is:

```text
O(n × n!)
```

---

### Notes / Tips

* The key difference from combination problems is that **order matters**.
* There is no `start` index.
* At every level, iterate through **all elements**.
* `used[]` tells us which elements are already in the current permutation.
* The backtracking sequence is always:

  ```text
  choose → recurse → undo
  ```
* Because the input contains distinct integers, we do not need duplicate handling.
* For `[1,2,3]`, the number of permutations is:

  ```text
  3! = 6
  ```
* For `n` elements:

  ```text
  n × (n-1) × (n-2) × ... × 1 = n!
  ```

---

### Code

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> current;

    void backtrack(vector<int>& nums, vector<bool>& used) {
        if (current.size() == nums.size()) {
            ans.push_back(current);
            return;
        }

        for (int i = 0; i < nums.size(); i++) {
            if (used[i]) {
                continue;
            }

            // Choose
            used[i] = true;
            current.push_back(nums[i]);

            // Explore
            backtrack(nums, used);

            // Undo
            current.pop_back();
            used[i] = false;
        }
    }

    vector<vector<int>> permute(vector<int>& nums) {
        vector<bool> used(nums.size(), false);

        backtrack(nums, used);

        return ans;
    }
};
```

---

## Alternative Approach — In-Place Swapping

### Idea

We can generate permutations without a separate `used[]` array by fixing one position at a time.

At recursion level `start`:

* Positions before `start` are already fixed.
* Swap every possible element into position `start`.
* Recurse for the next position.
* Swap back to restore the original array.

For:

```text
[1,2,3]
```

At `start = 0`:

```text
[1,2,3]
[2,1,3]
[3,2,1]
```

Then recursively permute the remaining portion.

This uses the array itself to keep track of which elements have already been placed.

---

### Dry Run

Start:

```text
[1,2,3]
start = 0
```

Keep `1` at position `0`:

```text
[1,2,3]
```

Now `start = 1`.

Swap `2` into position `1`:

```text
[1,2,3]
```

Fix `3`:

```text
[1,2,3]
```

Backtrack and swap:

```text
[1,3,2]
```

Then return to `start = 0`.

Swap `2` into position `0`:

```text
[2,1,3]
```

Continue until every possibility has been explored.

---

### Algorithm

1. Start with `start = 0`.
2. If `start == nums.size()`:

   * Add the current array to the answer.
   * Return.
3. Iterate `i` from `start` to the end.
4. Swap `nums[start]` and `nums[i]`.
5. Recursively permute positions after `start`.
6. Swap back to restore the array.
7. Continue with the next `i`.

---

### Complexity

* **Time:** `O(n × n!)`
* **Auxiliary Space:** `O(n)` recursion stack
* **Output Space:** `O(n × n!)`

Compared with the `used[]` approach, this avoids the extra `O(n)` `used` array.

---

### Notes / Tips

The invariant is:

```text
Positions [0 ... start - 1]
        ↓
Already fixed

Positions [start ... n - 1]
        ↓
Still available
```

The most important line is:

```cpp
swap(nums[start], nums[i]);
```

and the corresponding undo:

```cpp
swap(nums[start], nums[i]);
```

This is another form of:

```text
Choose → Recurse → Undo
```

---

### Code

```cpp
class Solution {
public:
    vector<vector<int>> ans;

    void backtrack(vector<int>& nums, int start) {
        if (start == nums.size()) {
            ans.push_back(nums);
            return;
        }

        for (int i = start; i < nums.size(); i++) {
            // Choose
            swap(nums[start], nums[i]);

            // Explore
            backtrack(nums, start + 1);

            // Undo
            swap(nums[start], nums[i]);
        }
    }

    vector<vector<int>> permute(vector<int>& nums) {
        backtrack(nums, 0);

        return ans;
    }
};
```

---

## Key Takeaways

### Core `used[]` Template

```cpp
void backtrack() {
    if (current.size() == nums.size()) {
        add current;
        return;
    }

    for (int i = 0; i < nums.size(); i++) {
        if (used[i]) {
            continue;
        }

        used[i] = true;
        current.push_back(nums[i]);

        backtrack();

        current.pop_back();
        used[i] = false;
    }
}
```

### Core In-Place Template

```cpp
void backtrack(int start) {
    if (start == nums.size()) {
        add nums;
        return;
    }

    for (int i = start; i < nums.size(); i++) {
        swap(nums[start], nums[i]);

        backtrack(start + 1);

        swap(nums[start], nums[i]);
    }
}
```

---

## Combination vs Permutation

This distinction is extremely important.

### Combination

Order does **not** matter:

```text
[1,2] == [2,1]
```

Therefore we use a `start` index to avoid going backward.

### Permutation

Order **does** matter:

```text
[1,2] != [2,1]
```

Therefore every recursion level can consider **every unused element**.

```text
Combination:
start → move forward

Permutation:
used[] → choose any unused element
```

### Mental Model

> **Permutations fill positions. At each position, choose any element that has not been used yet. Then undo that choice when backtracking.**

The essential pattern is:

```text
Choose unused element
        ↓
Place it
        ↓
Recurse
        ↓
Remove it
        ↓
Try another unused element
```
