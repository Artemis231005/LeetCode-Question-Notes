# LeetCode 40 — Combination Sum II

## Metadata

* **LeetCode:** 40
* **Problem:** Combination Sum II
* **Difficulty:** Medium
* **Topics:** Array, Backtracking
* **Pattern:** Backtracking, Combination Generation, Duplicate Handling
* **Key Pattern:** Sort + Skip Duplicates + Backtrack
* **Key Technique:** `start` index with duplicate skipping
* **Optimal Complexity:** `O(2^n)` time, excluding output cost
* **Key Template:** Backtracking Without Reuse

---

## Problem

Given an array `candidates` of positive integers and a target integer `target`, return all unique combinations where the chosen numbers sum to `target`.

Rules:

1. Each number in `candidates` can be used **at most once**.
2. The solution must not contain duplicate combinations.
3. The order of numbers inside a combination does not matter.
4. `candidates` may contain duplicate values.

Example:

```text
candidates = [10,1,2,7,6,1,5]
target = 8

Output:
[
    [1,1,6],
    [1,2,5],
    [1,7],
    [2,6]
]
```

---

## Approach — Backtracking + Sorting

### Idea

This is very similar to **Combination Sum (LeetCode 39)**, but there are two critical differences:

### LeetCode 39

```text
Same candidate can be reused
        ↓
backtrack(i, ...)
```

### LeetCode 40

```text
Each element can be used only once
        ↓
backtrack(i + 1, ...)
```

The second important difference is that the input itself can contain duplicate values.

For example:

```text
[1, 1, 2]
```

If we blindly choose both `1`s as starting points, we could generate the same combination multiple times.

Therefore:

1. **Sort the array.**
2. At the same recursion level, skip duplicate values:

   ```cpp
   if (i > start && candidates[i] == candidates[i - 1]) {
       continue;
   }
   ```

The condition `i > start` is extremely important.

It means:

> Skip a duplicate only when it is being considered as another choice at the **same recursion level**.

We do **not** skip duplicates that are used at different depths because those may be valid combinations.

---

## Dry Run

Consider:

```text
candidates = [1,1,2,5,6,7,10]
target = 8
```

Start:

```text
[]
target = 8
```

Choose first `1`:

```text
[1]
target = 7
```

At the next level, we can choose the **second `1`**:

```text
[1,1]
target = 6
```

Then choose `6`:

```text
[1,1,6]
target = 0
```

Valid combination:

```text
[1,1,6]
```

Now backtrack.

At the original level, suppose we have:

```text
i = 1
candidates[i] = 1
candidates[i - 1] = 1
```

Since:

```cpp
i > start
```

and:

```cpp
candidates[i] == candidates[i - 1]
```

we skip this `1`.

This prevents generating:

```text
[1,1,6]
```

again from the second `1`.

Then we try:

```text
[1,2,5]
[1,7]
[2,6]
```

giving:

```text
[
    [1,1,6],
    [1,2,5],
    [1,7],
    [2,6]
]
```

---

## Why `i > start` Is Important

Consider:

```text
candidates = [1,1,6]
target = 8
```

At the first level:

```text
start = 0
```

We choose the first `1`.

Now:

```text
[1]
```

At the next recursion level:

```text
start = 1
```

The second `1` is allowed:

```text
[1,1]
```

because it is a **different array element** and can be used once.

The duplicate condition is:

```cpp
if (i > start && candidates[i] == candidates[i - 1])
```

At this level:

```text
i = 1
start = 1
```

So:

```text
i > start
1 > 1 → false
```

Therefore, the second `1` is allowed.

But at the original recursion level:

```text
start = 0
i = 1
```

Now:

```text
i > start
1 > 0 → true
```

So the second `1` is skipped.

### Remember

```text
Same recursion level
    duplicate → SKIP

Different recursion level
    duplicate → ALLOW
```

This is the core trick of Combination Sum II.

---

## Why We Sort

Sorting gives:

```text
[10,1,2,7,6,1,5]
        ↓
[1,1,2,5,6,7,10]
```

This is necessary for two reasons.

### 1. Duplicate Detection

We can easily detect adjacent duplicates:

```cpp
candidates[i] == candidates[i - 1]
```

### 2. Early Stopping

Because the array is sorted:

```cpp
if (candidates[i] > target) {
    break;
}
```

If the current candidate is already too large, every candidate after it will also be too large.

---

### Algorithm

1. Sort `candidates`.
2. Start backtracking from index `0`.
3. If `target == 0`:

   * Add the current combination to the answer.
   * Return.
4. Iterate from `start` to the end of the array.
5. If the current candidate is a duplicate of the previous candidate **at the same recursion level**, skip it:

   ```cpp
   if (i > start && candidates[i] == candidates[i - 1]) {
       continue;
   }
   ```
6. If the candidate is greater than the remaining target, stop the loop.
7. Add the candidate to the current combination.
8. Recursively call:

   ```cpp
   backtrack(i + 1, target - candidates[i])
   ```

   because each array element can be used only once.
9. Remove the candidate from the current combination.
10. Continue with the next candidate.

---

### Complexity

Let `n` be the number of candidates.

In the worst case, every element can either be:

```text
chosen
OR
not chosen
```

giving approximately:

```text
2^n
```

possible subsets.

* **Time:** `O(2^n)` excluding the cost of returning the output
* **Sorting:** `O(n log n)`
* **Auxiliary recursion space:** `O(n)`
* **Output space:** depends on the number of valid combinations and their sizes

The actual runtime is usually much better because duplicate skipping and pruning reduce the search space.

---

### Notes / Tips

* **Sort first.**
* Use:

  ```cpp
  i > start
  ```

  when skipping duplicates.
* Do **not** use:

  ```cpp
  i > 0
  ```

  because that would incorrectly prevent using duplicate values at different recursion levels.
* Since every element can be used only once:

  ```cpp
  backtrack(i + 1, ...)
  ```
* Since candidates are sorted and positive:

  ```cpp
  if (candidates[i] > target) {
      break;
  }
  ```

  is valid pruning.
* `pop_back()` is the undo operation.
* The duplicate rule is about **recursion levels**, not about whether a value has appeared anywhere in the current combination.

---

### Code

```cpp
class Solution {
public:
    vector<vector<int>> ans;
    vector<int> current;

    void backtrack(vector<int>& candidates, int start, int target) {
        if (target == 0) {
            ans.push_back(current);
            return;
        }

        for (int i = start; i < candidates.size(); i++) {
            // Skip duplicate choices at the same recursion level
            if (i > start && candidates[i] == candidates[i - 1]) {
                continue;
            }

            // Since candidates is sorted
            if (candidates[i] > target) {
                break;
            }

            // Choose
            current.push_back(candidates[i]);

            // Move to i + 1 because each element can be used once
            backtrack(candidates, i + 1, target - candidates[i]);

            // Undo
            current.pop_back();
        }
    }

    vector<vector<int>> combinationSum2(
        vector<int>& candidates,
        int target
    ) {
        sort(candidates.begin(), candidates.end());

        backtrack(candidates, 0, target);

        return ans;
    }
};
```

---

## LeetCode 39 vs LeetCode 40

|                     | Combination Sum I   | Combination Sum II                |
| ------------------- | ------------------- | --------------------------------- |
| Reuse element?      | Yes                 | No                                |
| Input duplicates?   | No                  | Yes                               |
| Sort required?      | Helpful             | **Yes**                           |
| Recursive call      | `backtrack(i, ...)` | `backtrack(i + 1, ...)`           |
| Duplicate skipping? | No                  | **Yes**                           |
| Main pattern        | Backtracking        | Backtracking + Duplicate Handling |

### Most Important Difference

```text
LeetCode 39:
Choose candidate
    ↓
Can choose it again
    ↓
backtrack(i, ...)


LeetCode 40:
Choose candidate
    ↓
Cannot choose same array element again
    ↓
backtrack(i + 1, ...)
```

---

## Key Takeaways

The core template for **Combination Sum II** is:

```cpp
void backtrack(int start, int target) {
    if (target == 0) {
        add current;
        return;
    }

    for (int i = start; i < candidates.size(); i++) {

        if (i > start && candidates[i] == candidates[i - 1]) {
            continue;
        }

        if (candidates[i] > target) {
            break;
        }

        current.push_back(candidates[i]);

        backtrack(i + 1, target - candidates[i]);

        current.pop_back();
    }
}
```

The three things to remember are:

```text
1. Sort
      ↓
2. Skip duplicates at the same level
      ↓
3. Recurse with i + 1
```

### Backtracking Pattern

```text
Choose
  ↓
Recurse
  ↓
Undo
```

### Duplicate Pattern

```cpp
if (i > start && candidates[i] == candidates[i - 1]) {
    continue;
}
```

**`i > start` is the key line.** It prevents duplicate combinations while still allowing duplicate values to be selected when they come from different array positions.
