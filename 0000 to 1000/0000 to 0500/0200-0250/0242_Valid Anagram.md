# LeetCode 242 — Valid Anagram

## Metadata

* **LeetCode:** 242
* **Problem:** Valid Anagram
* **Difficulty:** Easy
* **Topics:** String, Hash Map, Sorting
* **Pattern:** Character Frequency Counting
* **Key Technique:** Count character occurrences in one string, then decrement using the other — a mismatch or leftover count means not an anagram
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given two strings `s` and `t`, return `true` if `t` is an anagram of `s` (uses exactly the same characters with the same frequencies), otherwise return `false`.

---

## Approaches

1. **Brute Force — Sort Both Strings and Compare**
2. **Optimal — Fixed-Size Frequency Array**

---

# Approach 1 — Brute Force / Sort Both Strings and Compare

## Idea

Two strings are anagrams exactly when they contain the same multiset of characters. Sorting both strings puts identical characters in identical positions if (and only if) they're anagrams, so a direct equality check after sorting settles it.

## Dry Run

```text
s = "anagram", t = "nagaram"
```

Sort both:

```text
sorted(s) = "aaagmnr"
sorted(t) = "aaagmnr"
```

Equal → anagram → return `true`.

```text
s = "rat", t = "car"
```

Sort both:

```text
sorted(s) = "art"
sorted(t) = "acr"
```

Not equal → return `false`.

## Algorithm

1. If `s.length() != t.length()`, return `false` immediately.
2. Sort both `s` and `t`.
3. Return whether the sorted strings are equal.

## Complexity

* **Time:** `O(n log n)`

  * Dominated by sorting both strings.
* **Space:** `O(n)`

  * Sorting typically requires or produces a copy of the string (or `O(log n)` for the sort's own recursion stack, depending on implementation — but a returned/copied sorted string means `O(n)` overall).

## Notes / Tips

* Simple and easy to reason about, but sorting is unnecessary work when only character counts matter, not their order.
* Requires only a length check as an early exit — if the lengths differ, they can never be anagrams regardless of content.

## Code

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        if (s.size() != t.size()) {
            return false;
        }

        sort(s.begin(), s.end());
        sort(t.begin(), t.end());

        return s == t;
    }
};
```

---

# Approach 2 — Optimal / Fixed-Size Frequency Array

## Idea

Since the problem is restricted to lowercase English letters, a fixed-size array of `26` counters is enough to track character frequency. Increment the count for each character in `s`, decrement for each character in `t`. If they're truly anagrams, every counter ends at exactly `0`.

## Dry Run

```text
s = "anagram", t = "nagaram"
```

Increment for `s`:

```text
a: 3, n: 1, g: 1, r: 1, m: 1
```

Decrement for `t`:

```text
n → a:3, n:0
a → a:2, n:0
g → a:2, n:0, g:0
a → a:1, n:0, g:0
r → a:1, n:0, g:0, r:0
a → a:0, n:0, g:0, r:0
m → a:0, n:0, g:0, r:0, m:0
```

All counts end at `0` → return `true`.

## Algorithm

1. If `s.length() != t.length()`, return `false` immediately.
2. Initialize a `count` array of size `26`, all zeros.
3. For each character in `s`, increment `count[c - 'a']`.
4. For each character in `t`, decrement `count[c - 'a']`.
5. If any value in `count` is nonzero, return `false`.
6. Otherwise, return `true`.

## Complexity

* **Time:** `O(n)`

  * Two linear passes (one over `s`, one over `t`), plus a final constant-size scan of the 26-element count array.
* **Space:** `O(1)`

  * The `count` array is a fixed size of `26` regardless of input length — doesn't scale with `n`.

## Notes / Tips

* The length check upfront is what makes the single increment/decrement pass safe — without it, mismatched lengths could still land on all-zero counts if one string is a truncated prefix pattern of repeated characters (though in practice differing lengths always fail the anagram condition, checking early avoids ambiguity and extra work).
* For Unicode or non-lowercase-only inputs, a fixed 26-slot array doesn't work — a hash map keyed by character is the natural fallback, trading `O(1)` space for `O(charset size)`.
* This exact "increment for one, decrement for the other, check all zero" trick generalizes to any "do two collections have the same multiset of items" problem.

## Code

```cpp
class Solution {
public:
    bool isAnagram(string s, string t) {
        if (s.size() != t.size()) {
            return false;
        }

        vector<int> count(26, 0);

        for (char c : s) {
            count[c - 'a']++;
        }
        for (char c : t) {
            count[c - 'a']--;
        }

        for (int c : count) {
            if (c != 0) {
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
if len(s) != len(t): return false

count = array of size 26, all 0

for c in s: count[c - 'a'] += 1
for c in t: count[c - 'a'] -= 1

for value in count:
    if value != 0: return false

return true
```