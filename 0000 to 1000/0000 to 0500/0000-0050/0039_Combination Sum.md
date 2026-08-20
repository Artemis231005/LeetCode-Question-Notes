# LeetCode 39 — Combination Sum

## Metadata

* **LeetCode:** 39
* **Problem:** Combination Sum
* **Difficulty:** Medium
* **Topics:** Array, Backtracking
* **Pattern:** Backtracking, Subset/Combination Generation
* **Key Pattern:** Choose → Recurse → Undo
* **Key Technique:** Backtracking with `start` index
* **Optimal Complexity:** `O(2^T)` approximately, where `T = target / min(candidates)`
* **Key Template:** Backtracking with Reuse of Current Element

---

## Problem

Given an array of **distinct positive integers** `candidates` and a target integer `target`, return all unique combinations where the chosen numbers sum to `target`.

Rules:

* The same number can be chosen **unlimited times**.
* Different combinations should not be duplicated.
* The order of numbers inside a combination does not matter.

Example:

```text
candidates = [2, 3, 6, 7]
target = 7

Output:
[
    [2, 2, 3],
    [7]
]
```

`[2, 3, 2]` is not considered a different combination from `[2, 2, 3]`.

---

## Approach — Backtracking

### Idea

At every position, we have two possibilities:

```text
Choose the current number
        OR
Skip to the next number
```

The important detail is that **we can reuse the same number unlimited times**.

Therefore, when we choose `candidates[i]`, the recursive call should use:

```cpp
backtrack(i, ...)
```

instead of:

```cpp
backtrack(i + 1, ...)
```

because we are allowed to choose the same candidate again.

However, when we skip the current candidate, we move forward:

```cpp
backtrack(i + 1, ...)
```

The `start` index also prevents duplicate combinations.

For example:

```text
[2, 3]
```

and

```text
[3, 2]
```

represent the same combination.

By only allowing the recursion to move forward through the candidate array, we generate only one ordering.

---

## Dry Run

Consider:

```text
candidates = [2, 3, 6, 7]
target = 7
```

Start:

```text
[]
target = 7
```

Choose `2`:

```text
[2]
target = 5
```

Since `2` can be reused, choose `2` again:

```text
[2, 2]
target = 3
```

Choose `2`:

```text
[2, 2, 2]
target = 1
```

No candidate can be used because all candidates are greater than `1`.

Backtrack:

```text
[2, 2]
target = 3
```

Try `3`:

```text
[2, 2, 3]
target = 0
```

Target becomes `0`, so we found a valid combination:

```text
[2, 2, 3]
```

Backtrack and explore other possibilities.

Eventually:

```text
[7]
target = 0
```

So:

```text
[7]
```

is also added.

Final result:

```text
[
    [2, 2, 3],
    [7]
]
```

---

## Why `start` Is Important

Suppose we did not use a `start` index.

After choosing `2`, we could later choose `3`, and then choose `2` again:

```text
[2, 3, 2]
```

This would produce the same combination as:

```text
[2, 2, 3]
```

We would also generate:

```text
[3, 2, 2]
```

These are duplicates.

Using `start` ensures:

```text
Once we move past an element,
we never go back to it.
```

So combinations are generated in non-decreasing candidate-index order.

---

## Why We Pass `i` Instead of `i + 1`

This is the most important detail in this problem.

### If repetition is allowed

```cpp
backtrack(i, remaining);
```

Example:

```text
[2]
 ↓
[2, 2]
 ↓
[2, 2, 2]
```

The same candidate can be selected again.

### If repetition were NOT allowed

We would use:

```cpp
backtrack(i + 1, remaining);
```

Example:

```text
[2]
 ↓
move to next candidate
```

This distinction is extremely important for backtracking problems.

---

### Algorithm

1. Create an empty `current` combination.
2. Start backtracking from index `0`.
3. If `target == 0`:

   * Add `current` to the answer.
   * Return.
4. Iterate from `start` to the end of `candidates`.
5. If the current candidate is greater than the remaining target, stop exploring further candidates.
6. Add the candidate to `current`.
7. Recursively call:

   ```cpp
   backtrack(i, target - candidates[i])
   ```

   using `i`, not `i + 1`, because reuse is allowed.
8. Remove the candidate from `current` to backtrack.
9. Continue trying other candidates.

---

### Complexity

Let:

```text
T = target
M = minimum candidate value
```

The maximum recursion depth is approximately:

```text
T / M
```

The number of possible combinations can be exponential.

A commonly used upper-bound description is approximately:

```text
Time:  O(2^T)
Space: O(T)
```

More precisely, the actual complexity depends heavily on the candidate values and the number of valid combinations.

* **Recursion depth:** `O(T / minCandidate)`
* **Auxiliary recursion space:** `O(T / minCandidate)`
* **Output space:** depends on the number and size of valid combinations.

---

### Notes / Tips

* The key idea is **backtracking with a `start` index**.
* Repetition is allowed, so recurse using:

  ```cpp
  backtrack(i, ...)
  ```
* Moving to the next candidate uses:

  ```cpp
  backtrack(i + 1, ...)
  ```
* Always remove the chosen element after recursion:

  ```cpp
  current.pop_back();
  ```
* Because candidates are positive, if:

  ```cpp
  candidates[i] > target
  ```

  then later candidates will also be too large **if the array is sorted**.
* Sorting the candidates allows early stopping.
* The `start` index prevents permutations of the same combination from being generated.
* There is no need for a `used` array because elements can be reused unlimited times.

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
            if (candidates[i] > target) {
                break;
            }

            // Choose
            current.push_back(candidates[i]);

            // Reuse the same candidate
            backtrack(candidates, i, target - candidates[i]);

            // Undo
            current.pop_back();
        }
    }

    vector<vector<int>> combinationSum(vector<int>& candidates, int target) {
        sort(candidates.begin(), candidates.end());

        backtrack(candidates, 0, target);

        return ans;
    }
};
```

---

## Key Takeaways

### Core Template

```cpp
void backtrack(int start, int target) {
    if (target == 0) {
        add current combination;
        return;
    }

    for (int i = start; i < candidates.size(); i++) {
        if (candidate[i] > target) {
            break;
        }

        current.push_back(candidate[i]);

        backtrack(i, target - candidate[i]);

        current.pop_back();
    }
}
```

### Most Important Rule

```text
Combination Sum
        |
        +-- Reuse allowed
        |      ↓
        |   recurse with i
        |
        +-- Duplicate combinations avoided
               ↓
            use start
```

Compare:

```text
Combination Sum I:
backtrack(i)       → reuse allowed

Combination Sum II:
backtrack(i + 1)   → each element used once
```

The fundamental pattern is:

```text
Choose
  ↓
Recurse
  ↓
Undo
```

with the **`start` index controlling whether elements can be revisited and preventing duplicate combinations**.
