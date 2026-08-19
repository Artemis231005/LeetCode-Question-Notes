# LeetCode 28 — Find the Index of the First Occurrence in a String

## Metadata

* **LeetCode:** 28
* **Problem:** Find the Index of the First Occurrence in a String
* **Difficulty:** Easy
* **Topics:** String, String Matching
* **Pattern:** Substring Search
* **Key Technique:** Sliding Window / Pattern Matching
* **Key Pattern:** Fixed-Size Window
* **Optimal Complexity:** `O(n × m)` time, `O(1)` space

---

## Problem

Given two strings `haystack` and `needle`, return the index of the **first occurrence** of `needle` in `haystack`.

Return `-1` if `needle` does not occur in `haystack`.

Example:

```text
haystack = "sadbutsad"
needle = "sad"

Output = 0
```

---

# Approach 1 — Brute Force / Character-by-Character Matching

## Idea

Try every possible starting position in `haystack`.

For each position:

1. Check whether the substring starting there matches `needle`.
2. Compare characters one by one.
3. If all characters match, return that starting index.
4. If no position works, return `-1`.

This is essentially manual substring matching.

## Dry Run

```text
haystack = "sadbutsad"
needle = "but"

Start at index 0:
"sad" ≠ "but"

Start at index 1:
"adb" ≠ "but"

Start at index 2:
"dbu" ≠ "but"

Start at index 3:
"but" = "but"

Return 3
```

## Algorithm

1. Let `n = haystack.size()` and `m = needle.size()`.
2. If `m > n`, return `-1`.
3. Try every starting index `i` from `0` to `n - m`.
4. For each `i`, compare:

   * `haystack[i + j]`
   * `needle[j]`
5. If every character matches, return `i`.
6. If no match is found, return `-1`.

## Complexity

* **Time:** `O(n × m)`
* **Space:** `O(1)`

## Notes / Tips

* We only need to check starting positions up to `n - m`.
* There is no point starting after `n - m` because there would not be enough characters left to contain `needle`.
* This is the basic string-matching technique and is important to understand before learning optimized algorithms such as KMP.

## Code

```cpp
class Solution {
public:
    int strStr(string haystack, string needle) {
        int n = haystack.size();
        int m = needle.size();

        if (m > n) {
            return -1;
        }

        for (int i = 0; i <= n - m; i++) {
            int j = 0;

            while (j < m && haystack[i + j] == needle[j]) {
                j++;
            }

            if (j == m) {
                return i;
            }
        }

        return -1;
    }
};
```

---

# Approach 2 — Sliding Window

## Idea

Since `needle` has a fixed length `m`, look at every window of size `m` in `haystack`.

For each window, compare it with `needle`.

For example:

```text
haystack = "sadbutsad"
needle   = "but"

Windows of size 3:

"sad"
"adb"
"dbu"
"but"  ← match
"uts"
"tsa"
"sad"
```

The first matching window gives the answer.

## Dry Run

```text
haystack = "hello"
needle = "ll"

Window size = 2

"he" → no
"el" → no
"ll" → match

Return index 2
```

## Algorithm

1. Store the length of `haystack` as `n`.
2. Store the length of `needle` as `m`.
3. If `m > n`, return `-1`.
4. Start a window at every index from `0` to `n - m`.
5. Compare the characters inside the current window with `needle`.
6. If all characters match, return the window's starting index.
7. If no window matches, return `-1`.

## Complexity

* **Time:** `O(n × m)`
* **Space:** `O(1)`

## Notes / Tips

* This is conceptually the same as Approach 1.
* The **window perspective** makes the pattern easier to recognize:

  * Text = `haystack`
  * Pattern = `needle`
  * Window size = `needle.length()`
* Avoid creating a new substring for every window because that can introduce unnecessary memory/time overhead.

## Code

```cpp
class Solution {
public:
    int strStr(string haystack, string needle) {
        int n = haystack.size();
        int m = needle.size();

        if (m > n) {
            return -1;
        }

        for (int i = 0; i <= n - m; i++) {
            bool match = true;

            for (int j = 0; j < m; j++) {
                if (haystack[i + j] != needle[j]) {
                    match = false;
                    break;
                }
            }

            if (match) {
                return i;
            }
        }

        return -1;
    }
};
```

---

# Approach 3 — KMP (Knuth-Morris-Pratt)

## Idea

The brute-force approach may repeatedly compare characters that we already know about.

KMP avoids these unnecessary comparisons by preprocessing `needle`.

It builds an **LPS array**:

> **LPS = Longest Proper Prefix which is also a Suffix**

The LPS array tells us how far we can move within `needle` after a mismatch instead of starting over.

For example:

```text
needle = "ababaca"

LPS = [0, 0, 1, 2, 3, 0, 1]
```

If a mismatch occurs after some matching characters, LPS tells us where in `needle` to continue.

## Dry Run

Consider:

```text
haystack = "ababcabcabababd"
needle   = "ababd"
```

The LPS array for `needle` is:

```text
needle:  a b a b d
LPS:     0 0 1 2 0
```

While matching:

```text
a b a b
```

we have already matched 4 characters.

Next:

```text
haystack → c
needle   → d
```

Mismatch.

Instead of restarting `needle` from index `0`, LPS tells us:

```text
LPS[3] = 2
```

So we continue matching from:

```text
needle[2]
```

This avoids rechecking characters unnecessarily.

## Algorithm

### Build LPS Array

1. Create an LPS array of size `m`.
2. Set `lps[0] = 0`.
3. Maintain:

   * `i` → current position in `needle`
   * `len` → length of the current matching prefix
4. If `needle[i] == needle[len]`:

   * Increment `len`.
   * Set `lps[i] = len`.
   * Increment `i`.
5. Otherwise:

   * If `len > 0`, set `len = lps[len - 1]`.
   * Otherwise set `lps[i] = 0` and increment `i`.

### Search Using LPS

1. Maintain:

   * `i` → index in `haystack`
   * `j` → index in `needle`
2. If `haystack[i] == needle[j]`:

   * Increment both pointers.
3. If `j == m`, the entire pattern has matched:

   * Return `i - j`.
4. If a mismatch occurs:

   * If `j > 0`, set `j = lps[j - 1]`.
   * Otherwise increment `i`.
5. If the entire `haystack` is processed without a match, return `-1`.

## Complexity

* **Time:** `O(n + m)`
* **Space:** `O(m)`

## Notes / Tips

* KMP is the optimized string-matching algorithm.
* The key idea is **do not throw away information from previous matches**.
* `LPS` tells us how much of the already-matched pattern can still be useful.
* This is useful when:

  * The strings are large.
  * Efficient pattern matching is required.
  * You need guaranteed `O(n + m)` matching.
* For this LeetCode problem, the simple `O(n × m)` approach is usually sufficient, but **KMP is the theoretically optimal standard pattern-matching solution**.

## Code

```cpp
class Solution {
public:
    int strStr(string haystack, string needle) {
        int n = haystack.size();
        int m = needle.size();

        if (m == 0) {
            return 0;
        }

        vector<int> lps(m, 0);

        // Build LPS array
        int len = 0;
        int i = 1;

        while (i < m) {
            if (needle[i] == needle[len]) {
                len++;
                lps[i] = len;
                i++;
            }
            else {
                if (len > 0) {
                    len = lps[len - 1];
                }
                else {
                    lps[i] = 0;
                    i++;
                }
            }
        }

        // Search for needle in haystack
        i = 0;
        int j = 0;

        while (i < n) {
            if (haystack[i] == needle[j]) {
                i++;
                j++;

                if (j == m) {
                    return i - j;
                }
            }
            else {
                if (j > 0) {
                    j = lps[j - 1];
                }
                else {
                    i++;
                }
            }
        }

        return -1;
    }
};
```

---

# Key Takeaway

The basic reusable pattern is:

```text
Try every possible starting position
        ↓
Compare needle with the substring
        ↓
Return first complete match
```

For normal LeetCode usage:

```cpp
for (int i = 0; i <= n - m; i++) {
    // Check whether needle matches starting at i
}
```

For advanced string matching:

```text
KMP
 ↓
Build LPS
 ↓
Use LPS to skip unnecessary comparisons
 ↓
O(n + m)
```

**Key Pattern:** Fixed-size substring search.

**Remember:** If you see **"find the first occurrence of one string inside another"**, think **String Matching / Sliding Window**. For advanced optimization, think **KMP**.
