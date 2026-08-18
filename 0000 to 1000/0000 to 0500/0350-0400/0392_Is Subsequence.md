# LeetCode 392 — Is Subsequence

## Metadata

* **LeetCode:** 392
* **Problem:** Is Subsequence
* **Difficulty:** Easy
* **Topics:** String, Two Pointers, Binary Search
* **Pattern:** Subsequence, Two Pointers, Preprocessing
* **Key Technique:** Match characters in order
* **Optimal Complexity:** `O(n)` for the standard problem

---

# Approach 1 — Brute Force / Generate Subsequences

## Idea

Generate all possible subsequences of `t` and check whether `s` is one of them.

A string of length `n` has `2^n` possible subsequences, making this approach extremely inefficient.

## Dry Run

```text
s = "ab"
t = "abc"
```

Possible subsequences include:

```text
""
"a"
"b"
"c"
"ab"  ← found
"ac"
"bc"
"abc"
```

Therefore, return `true`.

## Algorithm

1. Generate every possible subsequence of `t`.
2. Compare each generated subsequence with `s`.
3. If a match is found, return `true`.
4. Otherwise, return `false`.

## Complexity

* **Time:** `O(2^n * n)`
* **Space:** `O(n)` recursion stack, excluding generated subsequences.

## Notes / Tips

* This approach is useful for understanding the definition of a subsequence.
* It is not suitable for the actual constraints.

## Code

```cpp
class Solution {
public:
    bool generate(string &s, string &t, int index, string &current) {
        if (index == t.size()) {
            return current == s;
        }

        current.push_back(t[index]);

        if (generate(s, t, index + 1, current)) {
            return true;
        }

        current.pop_back();

        if (generate(s, t, index + 1, current)) {
            return true;
        }

        return false;
    }

    bool isSubsequence(string s, string t) {
        string current;
        return generate(s, t, 0, current);
    }
};
```

---

# Approach 2 — Two Pointers

## Idea

We do not need to generate subsequences.

We simply check whether every character of `s` can be found in `t` **in the same order**.

Use:

* `i` → current character of `s`
* `j` → current character of `t`

If:

```text
s[i] == t[j]
```

we have found the next required character, so increment `i`.

Regardless of whether characters match, increment `j`.

## Dry Run

```text
s = "abc"
t = "ahbgdc"
```

```text
a == a → match
b != h → skip h
b == b → match
c != g → skip g
c != d → skip d
c == c → match
```

All characters of `s` were matched.

```text
true
```

## Algorithm

1. Initialize `i = 0`, `j = 0`.
2. Traverse `t` using `j`.
3. If `s[i] == t[j]`, increment `i`.
4. Increment `j` after every comparison.
5. If `i == s.length()`, all characters of `s` have been matched.
6. Return `i == s.length()`.

## Complexity

Let:

* `m = s.length()`

* `n = t.length()`

* **Time:** `O(n)`

* **Space:** `O(1)`

## Notes / Tips

* Characters of `s` must appear in the **same relative order**.
* They do **not** need to be contiguous.
* Characters can be skipped from `t`.
* An empty `s` is always a subsequence.

## Code

```cpp
class Solution {
public:
    bool isSubsequence(string s, string t) {
        int i = 0;
        int j = 0;

        while (i < s.size() && j < t.size()) {
            if (s[i] == t[j]) {
                i++;
            }

            j++;
        }

        return i == s.size();
    }
};
```

---

# Approach 3 — Preprocessing + Binary Search

> **Important:** This is **NOT the same as the original LeetCode 392 problem**.
> It is an **advanced modified version / follow-up** where `t` is fixed and we need to check **many different strings `s`** against the same `t`.

## Modified Problem

Given a fixed string `t` and multiple strings `s`, determine whether each `s` is a subsequence of `t`.

For example:

```text
t = "ahbgdc"

s1 = "abc" → true
s2 = "axc" → false
s3 = "bg"  → true
s4 = "hd"  → true
```

Running the normal two-pointer solution separately for every `s` repeatedly scans `t`.

We can preprocess `t` once to make each query faster.

## Idea

For every character, store all positions where it occurs in `t`.

Example:

```text
t = "ahbgdc"
     012345
```

Position lists:

```text
a → [0]
b → [2]
c → [5]
d → [4]
g → [3]
h → [1]
```

Now process `s` from left to right.

For every character of `s`, find the **first occurrence after the previously matched position**.

Because the positions are sorted, use **binary search (`upper_bound`)**.

### Why `upper_bound`?

Suppose we just matched a character at position `2`.

The next character must occur at:

```text
position > 2
```

So:

```cpp
upper_bound(positions.begin(), positions.end(), 2)
```

gives the first position greater than `2`.

## Dry Run

```text
t = "ahbgdc"
s = "abc"
```

Preprocessed:

```text
a → [0]
b → [2]
c → [5]
```

Start:

```text
previous = -1
```

### Character `a`

Find first position `> -1`:

```text
a → 0
```

So:

```text
previous = 0
```

### Character `b`

Find first position `> 0`:

```text
b → 2
```

So:

```text
previous = 2
```

### Character `c`

Find first position `> 2`:

```text
c → 5
```

So:

```text
previous = 5
```

Every character was found in increasing positions.

Therefore:

```text
true
```

### Failing Example

```text
t = "ahbgdc"
s = "axc"
```

For `a`:

```text
a → 0
```

For `x`:

```text
x → no positions
```

Therefore:

```text
false
```

## Algorithm

### Preprocessing

1. Create an array/vector of positions for each possible character.
2. Traverse `t`.
3. Store the index of every character in its corresponding position list.

### For Each Query `s`

1. Set `previous = -1`.
2. For every character `c` in `s`:

   * Retrieve the position list of `c`.
   * Use binary search to find the first position greater than `previous`.
3. If no such position exists, return `false`.
4. Otherwise, update `previous` to the found position.
5. If every character is successfully matched, return `true`.

## Complexity

Let:

* `n = t.length()`
* `m = s.length()`

### Preprocessing

* **Time:** `O(n)`
* **Space:** `O(n)`

### Each Query

* **Time:** `O(m log n)`
* **Space:** `O(1)` additional space

### Multiple Queries

If there are `q` different strings `s`:

```text
Preprocessing: O(n)

Queries: O(q × m log n)
```

This can be much better than repeatedly scanning all of `t`.

## Notes / Tips

* This approach is useful when **`t` remains fixed** and there are many queries.
* The key idea is:
  **preprocess positions → find next valid position using binary search**.
* `upper_bound()` is used because the next character must appear **strictly after** the previous matched position.
* For the original single-query LeetCode 392, the normal **two-pointer solution is simpler and better**.
* This approach becomes valuable specifically in the **modified multiple-query version**.

## Code

```cpp
class SubsequenceChecker {
private:
    vector<vector<int>> positions;

public:
    SubsequenceChecker(string t) {
        positions.resize(256);

        for (int i = 0; i < t.size(); i++) {
            positions[t[i]].push_back(i);
        }
    }

    bool isSubsequence(string s) {
        int previous = -1;

        for (char c : s) {
            vector<int> &pos = positions[c];

            auto it = upper_bound(pos.begin(), pos.end(), previous);

            if (it == pos.end()) {
                return false;
            }

            previous = *it;
        }

        return true;
    }
};
```

---

# Approach Comparison

| Approach                      |                                          Time |  Space | Use Case                            |
| ----------------------------- | --------------------------------------------: | -----: | ----------------------------------- |
| Generate Subsequences         |                                  `O(2^n · n)` | `O(n)` | Learning / brute force              |
| Two Pointers                  |                                        `O(n)` | `O(1)` | **Original LeetCode 392**           |
| Preprocessing + Binary Search | `O(n)` preprocessing + `O(m log n)` per query | `O(n)` | **Modified multiple-query version** |

### Final Takeaway

For **LeetCode 392**, use **Two Pointers**.
For the **modified version with many different `s` strings and one fixed `t`**, use **Preprocessing + Binary Search**.
