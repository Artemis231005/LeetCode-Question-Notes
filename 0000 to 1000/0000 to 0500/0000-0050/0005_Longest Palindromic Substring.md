# LeetCode 5 — Longest Palindromic Substring

## Metadata

* **LeetCode:** 5
* **Problem:** Longest Palindromic Substring
* **Difficulty:** Medium
* **Topics:** String, Dynamic Programming
* **Pattern:** Palindrome, Expand Around Center
* **Key Technique:** Expand Around Center
* **Optimal Complexity:** `O(n²)` Time, `O(1)` Space
* **Advanced Technique:** Manacher's Algorithm

---

## Problem Statement

Find the **longest palindromic substring** in the given string.

A palindrome reads the same forward and backward.

Example:

```text
Input:  "babad"
Output: "bab"
```

`"aba"` is also a valid answer.

---

# Brute Approach

## Idea

Generate **every possible substring** and check whether each substring is a palindrome.

For every pair of indices `(i, j)`:

1. Consider `s[i...j]`.
2. Check whether it is a palindrome.
3. If it is, update the longest palindrome.

---

## Dry Run

For:

```text
s = "babad"
```

Some possible substrings are:

```text
"b"
"ba"
"bab"    ← palindrome
"baba"
"babad"
"aba"    ← palindrome
...
```

The longest palindrome has length `3`.

Therefore, `"bab"` is a valid answer.

---

## Algorithm

1. Initialize `start = 0` and `maxLen = 1`.
2. Generate every possible substring using two indices `i` and `j`.
3. Check whether `s[i...j]` is a palindrome.
4. If it is longer than the current answer:

   * Update `start`.
   * Update `maxLen`.
5. Return `s.substr(start, maxLen)`.

---

## Complexity
* **TC:** `O(n³)`

  * There are `O(n²)` substrings.
  * Each palindrome check can take `O(n)`.
  * Gives TLE.
* **SC:** `O(1)` auxiliary space.

---

## Code

```cpp
class Solution {
public:
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

    string longestPalindrome(string s) {
        int n = s.size();

        int start = 0;
        int maxLen = 1;

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                if (isPalindrome(s, i, j)) {
                    int len = j - i + 1;

                    if (len > maxLen) {
                        maxLen = len;
                        start = i;
                    }
                }
            }
        }

        return s.substr(start, maxLen);
    }
};
```

---

# Better Approach

## Idea

Use **Dynamic Programming** to remember whether smaller substrings are palindromes.

Define:
```text
dp[i][j] = true if s[i...j] is a palindrome
```

A substring `s[i...j]` is a palindrome if:
1. `s[i] == s[j]`, and
2. The inside substring `s[i+1...j-1]` is also a palindrome.

Therefore:
```text
dp[i][j] = (s[i] == s[j]) && dp[i + 1][j - 1]
```

### Base Cases

```text
Length 1 → always palindrome
Length 2 → palindrome if s[i] == s[j]
```

---

## Dry Run

For:
```text
s = "aba"
```

Single characters are palindromes:
```text
dp[0][0] = true
dp[1][1] = true
dp[2][2] = true
```

For `"aba"`:
```text
s[0] == s[2]
```

and:
```text
dp[1][1] = true
```

Therefore:
```text
dp[0][2] = true
```

So `"aba"` is a palindrome.

---

## Algorithm

1. Create an `n × n` DP table initialized to `false`.
2. Mark every single character as a palindrome.
3. Check substrings by increasing length.
4. For each substring `s[i...j]`:

   * Check whether `s[i] == s[j]`.
   * If length is `2`, it is a palindrome.
   * Otherwise, check `dp[i + 1][j - 1]`.
5. Whenever a palindrome is found, update the longest one.
6. Return the longest palindrome.

---

## Complexity

* **TC:** `O(n²)`
* **SC:** `O(n²)`

---

## Code

```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size();

        vector<vector<bool>> dp(n, vector<bool>(n, false));

        int start = 0;
        int maxLen = 1;

        // Every single character is a palindrome
        for (int i = 0; i < n; i++) {
            dp[i][i] = true;
        }

        // Check substrings by increasing length
        for (int len = 2; len <= n; len++) {
            for (int i = 0; i + len <= n; i++) {
                int j = i + len - 1;

                if (s[i] == s[j]) {
                    if (len == 2 || dp[i + 1][j - 1]) {
                        dp[i][j] = true;

                        if (len > maxLen) {
                            maxLen = len;
                            start = i;
                        }
                    }
                }
            }
        }

        return s.substr(start, maxLen);
    }
};
```

---

# Optimal Approach

## Idea
Every palindrome has a **center**.
There are two types of palindrome centers:

### Odd-Length Palindrome

Example:
```text
aba
 ↑
center
```
The center is one character.

### Even-Length Palindrome

Example:
```text
abba
  ↑
center
```
The center lies **between two characters**.

Therefore, for every possible center, expand outward while the characters match.
This gives `O(n²)` time with `O(1)` space.

---

## Dry Run

For:
```text
s = "babad"
```

Consider index `2`:
```text
b a b a d
    ↑
```

Expand outward:
```text
b a b
```

Since:
```text
s[1] == s[3]
```

the palindrome expands to:
```text
"bab"
```

Now check the center between two characters for even-length palindromes as well.

Continue for every possible center and keep the longest palindrome.

---

## Algorithm

1. Initialize `start = 0` and `maxLen = 1`.
2. For every index `i`:

   * Expand around `(i, i)` for an odd-length palindrome.
   * Expand around `(i, i + 1)` for an even-length palindrome.
3. During expansion:

   * Compare the left and right characters.
   * If equal, expand outward.
   * Stop when they differ or go out of bounds.
4. Calculate the palindrome length.
5. Update `start` and `maxLen` if necessary.
6. Return `s.substr(start, maxLen)`.

---

## Complexity
* **TC:** `O(n²)`

  * `O(n)` possible centers.
  * Each center can expand up to `O(n)`.
* **SC:** `O(1)` auxiliary space.

---

## Code

```cpp
class Solution {
public:
    int expand(string& s, int left, int right) {
        while (left >= 0 && right < s.size() &&
               s[left] == s[right]) {
            left--;
            right++;
        }

        return right - left - 1;
    }

    string longestPalindrome(string s) {
        int n = s.size();

        int start = 0;
        int maxLen = 1;

        for (int i = 0; i < n; i++) {
            // Odd-length palindrome
            int len1 = expand(s, i, i);

            // Even-length palindrome
            int len2 = expand(s, i, i + 1);

            int len = max(len1, len2);

            if (len > maxLen) {
                maxLen = len;
                start = i - (len - 1) / 2;
            }
        }

        return s.substr(start, maxLen);
    }
};
```

---

# Advanced Approach — Manacher's Algorithm

## Idea

**Manacher's Algorithm** finds the longest palindromic substring in **linear `O(n)` time**.

The main problem with Expand Around Center is that the same characters can be compared repeatedly while expanding around different centers.
Manacher's Algorithm avoids this repeated work by storing the **palindrome radius** around each center and using information from previously processed palindromes.

The key idea is:

> If a new center lies inside an already-known palindrome, we can use the palindrome information from its **mirror position** instead of starting the expansion from scratch.

---

## Preprocessing
Odd and even-length palindromes are easier to handle if we transform the string.

For example:
```text
Original:
abba
```

Transform it into:
```text
^#a#b#b#a#$
```

Here:
* `#` separates characters.
* `^` and `$` are boundary sentinels.
* Both odd and even-length palindromes become odd-length palindromes in the transformed string.

---

## Core Variables

Maintain:
```text
center = center of the rightmost palindrome
right  = right boundary of that palindrome
```

For the current position `i`, its mirror around `center` is:
```text
mirror = 2 * center - i
```

If `i < right`, we already know something about its palindrome radius:
```text
p[i] = min(right - i, p[mirror])
```

Then we try to expand further.

If the palindrome extends beyond `right`, update:
```text
center = i
right = i + p[i]
```

---

## Dry Run

For:
```text
s = "babad"
```

Transform:
```text
^#b#a#b#a#d$
```

Manacher's algorithm calculates the palindrome radius around every center.

For the center corresponding to the middle `b`:
```text
b a b
```

the radius expands to cover:
```text
"bab"
```

The largest radius found determines the longest palindromic substring.

---

## Algorithm

1. Transform the original string by inserting `#` between characters and adding boundary sentinels.
2. Create an array `p` where:

   ```text
   p[i] = palindrome radius around i
   ```
3. Maintain:

   ```text
   center
   right
   ```
4. For each position `i`:

   * Find its mirror:

     ```text
     mirror = 2 * center - i
     ```
   * If `i < right`, initialize:

     ```text
     p[i] = min(right - i, p[mirror])
     ```
   * Expand while the characters on both sides match.
   * If the palindrome extends beyond `right`, update `center` and `right`.
5. Find the position with the largest `p[i]`.
6. Convert that position back to the original string indices.
7. Return the corresponding substring.

---

## Complexity

* **TC:** `O(n)`
* **SC:** `O(n)`

The algorithm is linear because previously computed palindrome information prevents repeated expansion work.

---

## Code

```cpp
class Solution {
public:
    string longestPalindrome(string s) {
        int n = s.size();

        if (n <= 1) {
            return s;
        }

        string t = "^";

        for (char c : s) {
            t += "#";
            t += c;
        }

        t += "#$";

        int m = t.size();

        vector<int> p(m, 0);

        int center = 0;
        int right = 0;

        int maxLen = 0;
        int maxCenter = 0;

        for (int i = 1; i < m - 1; i++) {
            int mirror = 2 * center - i;

            if (i < right) {
                p[i] = min(right - i, p[mirror]);
            }

            // Expand around center i
            while (t[i + 1 + p[i]] == t[i - 1 - p[i]]) {
                p[i]++;
            }

            // Update rightmost palindrome
            if (i + p[i] > right) {
                center = i;
                right = i + p[i];
            }

            // Track longest palindrome
            if (p[i] > maxLen) {
                maxLen = p[i];
                maxCenter = i;
            }
        }

        int start = (maxCenter - maxLen) / 2;

        return s.substr(start, maxLen);
    }
};
```

---

## Notes / Tips

* **Brute:** Generate every substring + check palindrome → `O(n³)`.
* **Better:** DP remembers whether smaller substrings are palindromes → `O(n²)` time, `O(n²)` space.
* **Optimal practical:** Expand Around Center → `O(n²)` time, `O(1)` space.
* **Advanced:** Manacher's Algorithm → `O(n)` time, `O(n)` space.
* Every palindrome has a **center**.
* Always check both:

  * `(i, i)` → odd-length palindrome.
  * `(i, i + 1)` → even-length palindrome.
* **Expand Around Center** is a reusable palindrome technique, but it is more of a **problem-specific technique** than a broad pattern like Sliding Window or Two Pointers.
* Manacher's is an advanced specialized algorithm. It is generally **not needed for normal questions** unless the problem specifically demands `O(n)`.
* The problem asks for a **substring**, so the characters must be contiguous.
* If multiple longest palindromes exist, returning any one of them is valid.

---

## Key Template

### Standard Practical Template

```text
Palindrome
    ↓
Find Every Center
    ↓
Odd Center:  (i, i)
Even Center: (i, i + 1)
    ↓
Expand Outward
    ↓
Update Longest
```

### Advanced Template

```text
Transform String
       ↓
Manacher's Array P
       ↓
Use Mirror Information
       ↓
Expand Only When Necessary
       ↓
Find Maximum Radius
       ↓
Recover Longest Palindrome
```

**Core Pattern to Remember:**

```text
Palindrome → Think About Its Center
```
