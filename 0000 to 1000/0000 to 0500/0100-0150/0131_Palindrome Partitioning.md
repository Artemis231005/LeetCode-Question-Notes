# LeetCode 131 — Palindrome Partitioning

## Metadata

* **LeetCode:** 131
* **Problem:** Palindrome Partitioning
* **Difficulty:** Medium
* **Topics:** String, Backtracking, Dynamic Programming
* **Pattern:** Backtracking, Partitioning
* **Key Technique:** Try every possible substring and recurse only if it is a palindrome
* **Key Pattern:** Backtracking + palindrome checking
* **Key Template:** Choose → Explore → Undo
* **Optimal Complexity:** `O(n · 2^n)` excluding output construction details

---

## Problem

Given a string `s`, partition it such that **every substring of the partition is a palindrome**.

Return all possible palindrome partitions.

Example:

```text
s = "aab"
```

Possible partitions:

```text
["a", "a", "b"]
["aa", "b"]
```

Answer:

```text
[
    ["a", "a", "b"],
    ["aa", "b"]
]
```

---

## Idea

This is a **Backtracking** problem.

At every position, try every possible substring starting from that position.

For:

```text
"aab"
```

Starting from index `0`, try:

```text
"a"
"aa"
"aab"
```

Only choose the substring if it is a palindrome.

For each valid choice:

1. Add the substring to the current partition.
2. Recursively solve the remaining string.
3. Remove the substring to backtrack.

### Decision Tree

```text
                    ""
              /      |       \
             a      aa       aab
            ✓       ✓         ✗
           /         \
         "a"         "b"
        ✓             ✓

Results:
[a, a, b]
[aa, b]
```

The key idea is:

> **At each index, choose a palindromic substring and recursively partition the remaining suffix.**

---

## Dry Run

Consider:

```text
s = "aab"
```

Start:

```text
start = 0
current = []
```

### Choose `"a"`

`"a"` is a palindrome.

```text
current = ["a"]
```

Recurse from index `1`.

Choose `"a"`:

```text
current = ["a", "a"]
```

Recurse from index `2`.

Choose `"b"`:

```text
current = ["a", "a", "b"]
```

We reached the end of the string.

Add:

```text
["a", "a", "b"]
```

Backtrack:

```text
["a", "a"]
→ ["a"]
```

### Back at index `1`

Try `"ab"`.

```text
"ab" → not palindrome
```

Skip it.

### Back at index `0`

Remove `"a"`.

Try `"aa"`.

`"aa"` is a palindrome.

```text
current = ["aa"]
```

Recurse from index `2`.

Choose `"b"`:

```text
current = ["aa", "b"]
```

Reached the end.

Add:

```text
["aa", "b"]
```

Final result:

```text
[
    ["a", "a", "b"],
    ["aa", "b"]
]
```

---

## Algorithm

1. Create an empty `current` partition.
2. Start backtracking from index `0`.
3. At each recursive call:

   * If `start == s.length()`, add the current partition to the answer.
   * Otherwise, try every ending index `end` from `start` to `n - 1`.
4. For each substring `s[start...end]`:

   * Check whether it is a palindrome.
   * If it is not, skip it.
   * If it is:

     1. Add it to `current`.
     2. Recursively partition from `end + 1`.
     3. Remove it from `current`.
5. Return all generated partitions.

---

## Complexity

Let `n` be the length of the string.

* **Time:** `O(n · 2^n)`

  * There can be `O(2^n)` possible partitions.
  * Checking/copying substrings adds up to `O(n)` work.
* **Space:** `O(n)`

  * Recursion depth and current partition.
  * Excluding the output.
* **Output Space:** Can be `O(n · 2^n)` in the worst case.

### Worst Case

For:

```text
"aaaa"
```

every substring is a palindrome, so many partitions are generated.

---

## Notes / Tips

* This is a classic **Backtracking + Partitioning** problem.
* The recursion parameter `start` represents where the next partition should begin.
* The loop chooses the **end** of the next substring.
* Only add a substring if it is a palindrome.
* Always **undo the choice** after recursion:

  ```cpp
  current.pop_back();
  ```
* The base case is:

  ```cpp
  start == s.length()
  ```
* Every character by itself is always a palindrome.
* The order of partitions naturally follows the order in which substrings are tried.

### Common Mistake

Do not recursively continue with every substring.

Wrong:

```text
Choose substring
→ recurse
```

Correct:

```text
Choose substring
→ check palindrome
→ recurse only if palindrome
```

### Optimization

Instead of checking every substring with a separate `O(n)` palindrome check, palindrome information can be precomputed using DP.

Define:

```text
dp[i][j] = true
```

if `s[i...j]` is a palindrome.

Then palindrome checking becomes `O(1)` during backtracking.

---

## Code

```cpp
class Solution {
public:
    vector<vector<string>> result;
    vector<string> current;

    bool isPalindrome(string& s, int left, int right) {
        while (left < right) {
            if (s[left] != s[right]) {
                return false;
            }

            left++;
            right--;
        }

        return true;
    }

    void backtrack(string& s, int start) {
        if (start == s.length()) {
            result.push_back(current);
            return;
        }

        for (int end = start; end < s.length(); end++) {
            if (!isPalindrome(s, start, end)) {
                continue;
            }

            current.push_back(s.substr(start, end - start + 1));

            backtrack(s, end + 1);

            current.pop_back();
        }
    }

    vector<vector<string>> partition(string s) {
        backtrack(s, 0);
        return result;
    }
};
```

---

## Basic Template

```cpp
void backtrack(string& s, int start) {
    if (start == s.length()) {
        result.push_back(current);
        return;
    }

    for (int end = start; end < s.length(); end++) {
        if (!isValid(s, start, end)) {
            continue;
        }

        current.push_back(s.substr(start, end - start + 1));

        backtrack(s, end + 1);

        current.pop_back();
    }
}
```

