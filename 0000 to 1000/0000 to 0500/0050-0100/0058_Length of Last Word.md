# LeetCode 58 — Length of Last Word

## Metadata

* **LeetCode:** 58
* **Problem:** Length of Last Word
* **Difficulty:** Easy
* **Topics:** String
* **Pattern:** String Traversal
* **Key Pattern:** Traverse from the end and skip trailing spaces
* **Key Technique:** Find the last non-space character, then count backward
* **Key Template:** Reverse Traversal
* **Optimal Complexity:** `O(n)` time, `O(1)` space

---

## Problem

Given a string `s` consisting of words and spaces, return the **length of the last word**.

A word is a maximal sequence of non-space characters.

Example:

```text
s = "Hello World"

Last word = "World"
Answer = 5
```

Trailing spaces may be present:

```text
s = "   fly me   to   the moon   "

Last word = "moon"
Answer = 4
```

---

## Approach — Reverse Traversal

### Idea

The easiest way is to start from the **end of the string**.

There may be spaces after the last word, so:

1. Skip all trailing spaces.
2. Start counting characters.
3. Stop when a space is encountered.

This avoids unnecessary splitting or storing individual words.

---

### Dry Run

For:

```text
s = "Hello World   "
```

Start from the end:

```text
"Hello World   "
            ↑
```

Skip spaces:

```text
"Hello World   "
         ↑
```

Now count:

```text
d → 1
l → 2
r → 3
o → 4
W → 5
```

Next character is a space, so stop.

Answer:

```text
5
```

---

### Algorithm

1. Set:

   ```text
   i = s.length() - 1
   ```
2. Skip all trailing spaces:

   ```text
   while i >= 0 and s[i] == ' ':
       i--
   ```
3. Initialize:

   ```text
   length = 0
   ```
4. Count characters until a space or the beginning of the string:

   ```text
   while i >= 0 and s[i] != ' ':
       length++
       i--
   ```
5. Return `length`.

---

### Complexity

* **Time:** `O(n)` in the worst case.
* **Space:** `O(1)`.

---

### Notes / Tips

* Reverse traversal is ideal because we only need the **last** word.
* Always skip trailing spaces first.
* Do not stop immediately when starting from `s.length() - 1`, because the string can end with spaces.
* `split()` can solve the problem, but it uses extra space and is unnecessary.
* The two important phases are:

```text
Skip trailing spaces
        ↓
Count last word
```

* Example edge case:

```text
s = "a"
```

Answer:

```text
1
```

* Another important case:

```text
s = "Hello "
```

Skip the trailing space, then count `"Hello"` → `5`.

---

### Code

```cpp
class Solution {
public:
    int lengthOfLastWord(string s) {
        int i = s.length() - 1;

        // Skip trailing spaces
        while (i >= 0 && s[i] == ' ') {
            i--;
        }

        int length = 0;

        // Count the last word
        while (i >= 0 && s[i] != ' ') {
            length++;
            i--;
        }

        return length;
    }
};
```

---

## Quick Revision

```text
Length of Last Word
        ↓
Start from end
        ↓
Skip trailing spaces
        ↓
Count non-space characters
        ↓
Stop at space / beginning
        ↓
Return count
```

### Core Template

```text
i = n - 1

while i >= 0 and s[i] == ' ':
    i--

length = 0

while i >= 0 and s[i] != ' ':
    length++
    i--

return length
```

**Pattern to remember:**
**Last word → Start from end → Skip spaces → Count characters**
