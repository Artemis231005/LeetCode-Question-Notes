# LeetCode 383 — Ransom Note

## Metadata

* **LeetCode:** 383
* **Problem:** Ransom Note
* **Difficulty:** Easy
* **Topics:** String, Hash Map
* **Pattern:** Character Frequency Counting
* **Key Technique:** Count characters available in `magazine`, then decrement while scanning `ransomNote` — running out of any character means it can't be constructed
* **Optimal Complexity:** `O(n + m)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given two strings `ransomNote` and `magazine`, return `true` if `ransomNote` can be constructed using the letters from `magazine`, where each letter in `magazine` can only be used once.

---

## Approaches

1. **Brute Force — Remove Matched Characters One at a Time**
2. **Optimal — Fixed-Size Frequency Array**

---

# Approach 1 — Brute Force / Remove Matched Characters One at a Time

## Idea

For every character in `ransomNote`, search through `magazine` to find that character. If found, "use it up" by removing it (e.g. marking its position as used) so it can't be reused for a later character. If any character can't be found, `ransomNote` can't be built.

## Dry Run

```text
ransomNote = "aab", magazine = "baa"
```

Process:

```text
'a' → found at magazine[1] → mark used → magazine = "b_a"
'a' → found at magazine[2] → mark used → magazine = "b__"
'b' → found at magazine[0] → mark used → magazine = "___"
```

All characters matched → return `true`.

```text
ransomNote = "aa", magazine = "ab"
```

```text
'a' → found at magazine[0] → mark used → magazine = "_b"
'a' → not found (only 'b' remains) → return false
```

## Algorithm

1. Convert `magazine` into a mutable structure (e.g. a character array with a "used" marker, or erase matched characters directly).
2. For each character `c` in `ransomNote`:

   * Search `magazine` for an unused occurrence of `c`.
   * If found, mark it used (or erase it).
   * If not found, return `false`.
3. If every character was matched, return `true`.

## Complexity

* **Time:** `O(n * m)`

  * For each of the `n` characters in `ransomNote`, a linear search of up to `m` characters in `magazine` may be needed.
* **Space:** `O(m)`

  * For the mutable copy of `magazine` (or its used-marker array).

## Notes / Tips

* Searching linearly through `magazine` for every character in `ransomNote` is the main bottleneck — a frequency count sidesteps repeated searching entirely.
* Erasing from a string/vector during a scan can also be error-prone (shifting indices) — using a separate "used" boolean array avoids that pitfall if this approach is kept.

## Code

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        vector<bool> used(magazine.size(), false);

        for (char c : ransomNote) {
            bool found = false;

            for (int i = 0; i < magazine.size(); i++) {
                if (!used[i] && magazine[i] == c) {
                    used[i] = true;
                    found = true;
                    break;
                }
            }

            if (!found) {
                return false;
            }
        }

        return true;
    }
};
```

---

# Approach 2 — Optimal / Fixed-Size Frequency Array

## Idea

Since the problem is restricted to lowercase English letters, count how many of each letter `magazine` has using a fixed-size array of `26` counters. Then scan `ransomNote`, decrementing the corresponding counter for each character used. If any counter goes negative, `magazine` didn't have enough of that letter.

## Dry Run

```text
ransomNote = "aab", magazine = "baa"
```

Count `magazine`:

```text
a: 2, b: 1
```

Scan `ransomNote`, decrementing:

```text
'a' → count[a] = 1
'a' → count[a] = 0
'b' → count[b] = 0
```

No counter went negative → return `true`.

```text
ransomNote = "aa", magazine = "ab"
```

Count `magazine`:

```text
a: 1, b: 1
```

Scan `ransomNote`:

```text
'a' → count[a] = 0
'a' → count[a] = -1 → not enough 'a' → return false
```

## Algorithm

1. Initialize a `count` array of size `26`, all zeros.
2. For each character in `magazine`, increment `count[c - 'a']`.
3. For each character in `ransomNote`:

   * Decrement `count[c - 'a']`.
   * If it drops below `0`, return `false` immediately.
4. If the entire scan completes without going negative, return `true`.

## Complexity

* **Time:** `O(n + m)`

  * One linear pass over `magazine` to build counts, one linear pass over `ransomNote` to consume them.
* **Space:** `O(1)`

  * The `count` array is a fixed size of `26` regardless of input length — doesn't scale with `n` or `m`.

## Notes / Tips

* Returning early the moment a counter goes negative avoids scanning the rest of `ransomNote` unnecessarily once failure is already certain.
* Same "increment for one side, decrement for the other" shape as LC 242 (Valid Anagram) — the key difference here is `magazine` can have leftover, unused characters, so only a negative count (not a nonzero one) signals failure.
* For non-lowercase-only inputs, a hash map keyed by character replaces the fixed 26-slot array, trading `O(1)` space for `O(charset size)`.

## Code

```cpp
class Solution {
public:
    bool canConstruct(string ransomNote, string magazine) {
        vector<int> count(26, 0);

        for (char c : magazine) {
            count[c - 'a']++;
        }

        for (char c : ransomNote) {
            count[c - 'a']--;
            if (count[c - 'a'] < 0) {
                return false;
            }
        }

        return true;
    }
};
```

---

## Key Template

```text
count = array of size 26, all 0

for c in magazine: count[c - 'a'] += 1

for c in ransomNote:
    count[c - 'a'] -= 1
    if count[c - 'a'] < 0: return false

return true
```