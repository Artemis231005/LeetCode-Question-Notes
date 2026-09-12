# LeetCode 844 — Backspace String Compare

## Metadata

* **LeetCode:** 844
* **Problem:** Backspace String Compare
* **Difficulty:** Easy
* **Topics:** String, Stack, Two Pointers, Simulation
* **Pattern:** Stack Simulation / Two Pointers from the End
* **Key Technique:** Treat `'#'` as a pop operation, either via an explicit stack or by scanning both strings from the back while skipping backspace-cancelled characters
* **Optimal Complexity:** `O(n + m)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given two strings `s` and `t`, each possibly containing `'#'` characters representing a backspace (deleting the previous character, if any), return `true` if the two strings are equal after processing all backspaces.

---

## Approaches

1. **Brute Force — Build Both Strings Using a Stack, Then Compare**
2. **Optimal — Two Pointers from the End, Skipping Backspaces**

---

# Approach 1 — Brute Force / Build Both Strings Using a Stack, Then Compare

## Idea

Process each string independently: scan left to right, pushing normal characters onto a stack, and popping the stack whenever a `'#'` is seen (if the stack isn't already empty). The final stack contents represent the string after all backspaces are applied. Compare both resulting stacks/strings for equality.

## Dry Run

```text
s = "ab#c", t = "ad#c"
```

Process `s`:

```text
'a' → push → stack = [a]
'b' → push → stack = [a, b]
'#' → pop  → stack = [a]
'c' → push → stack = [a, c]
```

Result: `"ac"`

Process `t`:

```text
'a' → push → stack = [a]
'd' → push → stack = [a, d]
'#' → pop  → stack = [a]
'c' → push → stack = [a, c]
```

Result: `"ac"`

Both equal `"ac"` → return `true`.

## Algorithm

1. Define a helper `buildFinalString(str)`:

   * Initialize an empty stack (or string used as a stack).
   * For each character in `str`:

     * If it's `'#'`, pop the stack if non-empty.
     * Otherwise, push the character.
   * Return the resulting stack contents as a string.
2. Compute `buildFinalString(s)` and `buildFinalString(t)`.
3. Return whether the two results are equal.

## Complexity

* **Time:** `O(n + m)`

  * One linear pass to build each string's final form.
* **Space:** `O(n + m)`

  * For the two stacks (or strings) holding each processed result.

## Notes / Tips

* Clean and easy to reason about, but uses extra space proportional to both input strings — the two-pointer approach reduces this to constant space by working backward through the original strings directly, without building new ones.
* This is the same "push normal, pop on special character" template as LC 71 (Simplify Path) and LC 682 (Baseball Game), just applied to backspace characters.

## Code

```cpp
class Solution {
public:
    string buildFinalString(string str) {
        string stack = "";

        for (char c : str) {
            if (c == '#') {
                if (!stack.empty()) {
                    stack.pop_back();
                }
            } else {
                stack.push_back(c);
            }
        }

        return stack;
    }

    bool backspaceCompare(string s, string t) {
        return buildFinalString(s) == buildFinalString(t);
    }
};
```

---

# Approach 2 — Optimal / Two Pointers from the End, Skipping Backspaces

## Idea

Instead of building the full processed string, walk both strings **backward** simultaneously. For each string, skip over any character that gets cancelled by a backspace: when a `'#'` is seen, increment a skip counter and move left; when a normal character is seen while the skip counter is positive, decrement the counter and skip that character too (it's been deleted). Once a "real" character is found in both strings (or one runs out), compare them directly — no full reconstruction needed.

## Dry Run

```text
s = "ab#c", t = "ad#c"
```

Start both pointers at the last index.

```text
s: index 3 = 'c' (real character, skip=0) → compare
t: index 3 = 'c' (real character, skip=0) → compare
'c' == 'c' → continue
```

Move both pointers left:

```text
s: index 2 = '#' → skip++ (skip=1), move left
   index 1 = 'b' → skip>0, so cancel it (skip--, skip=0), move left
   index 0 = 'a' (real character) → compare
t: index 2 = '#' → skip++ (skip=1), move left
   index 1 = 'd' → skip>0, so cancel it (skip--, skip=0), move left
   index 0 = 'a' (real character) → compare
'a' == 'a' → continue
```

Both pointers exhausted → strings match → return `true`.

## Algorithm

1. Initialize `i = s.size() - 1`, `j = t.size() - 1`.
2. Define a helper `nextValidIndex(str, index)` that, starting from `index`, skips backward over any backspace-cancelled characters:

   * Maintain a `skip` counter.
   * While `index >= 0`:

     * If `str[index] == '#'`, increment `skip` and decrement `index`.
     * Else if `skip > 0`, decrement `skip` and decrement `index` (this character is cancelled).
     * Else, break (found a valid character, or `index < 0`).
   * Return the final `index`.
3. While `i >= 0` or `j >= 0`:

   * Advance `i` and `j` to their next valid (non-cancelled) positions using the helper.
   * If one string is exhausted (`index < 0`) and the other isn't, return `false`.
   * If both are exhausted, return `true`.
   * If `s[i] != t[j]`, return `false`.
   * Decrement both `i` and `j`, and continue.
4. Return `true` if the loop completes without mismatch.

## Complexity

* **Time:** `O(n + m)`

  * Each pointer moves backward through its string at most once in total, even accounting for the skip logic.
* **Space:** `O(1)`

  * Only pointers and skip counters are used — no stacks or new strings created.

## Notes / Tips

* Processing from the **end** is the key insight — a backspace only ever affects characters that come *before* it, so scanning backward means every character's property (kept or cancelled) is already fully determined by the time it's reached, without needing to look ahead.
* Common mistake: forgetting to handle the case where one string still has characters left after the other is fully exhausted — mismatched lengths after full backspace processing should return `false`, not be treated as trivially equal.

## Code

```cpp
class Solution {
public:
    bool backspaceCompare(string s, string t) {
        int i = s.size() - 1, j = t.size() - 1;

        while (i >= 0 || j >= 0) {
            int skip = 0;
            while (i >= 0 && (s[i] == '#' || skip > 0)) {
                skip += (s[i] == '#') ? 1 : -1;
                i--;
            }

            skip = 0;
            while (j >= 0 && (t[j] == '#' || skip > 0)) {
                skip += (t[j] == '#') ? 1 : -1;
                j--;
            }

            if (i >= 0 && j >= 0) {
                if (s[i] != t[j]) {
                    return false;
                }
            } else if (i >= 0 || j >= 0) {
                return false;
            }

            i--;
            j--;
        }

        return true;
    }
};
```

---

## Key Template

```text
i = s.size() - 1
j = t.size() - 1

while i >= 0 or j >= 0:
    skip = 0
    while i >= 0 and (s[i] == '#' or skip > 0):
        skip += 1 if s[i] == '#' else -1
        i -= 1

    skip = 0
    while j >= 0 and (t[j] == '#' or skip > 0):
        skip += 1 if t[j] == '#' else -1
        j -= 1

    if i >= 0 and j >= 0:
        if s[i] != t[j]: return false
    elif i >= 0 or j >= 0:
        return false

    i -= 1
    j -= 1

return true
```