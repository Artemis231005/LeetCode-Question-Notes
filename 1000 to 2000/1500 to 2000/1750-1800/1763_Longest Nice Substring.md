# LeetCode 1763 — Longest Nice Substring

## Metadata

* **LeetCode:** 1763
* **Problem:** Longest Nice Substring
* **Difficulty:** Easy
* **Topics:** Hash Table, String, Divide and Conquer, Bit Manipulation, Sliding Window
* **Pattern:** Divide and Conquer on Invalid Characters
* **Key Technique:** Any character whose opposite-case counterpart is missing from the string can never be part of a nice substring — split the string at every such character and recurse on the pieces, instead of checking substrings one by one
* **Optimal Complexity:** `O(n * 26)` Time (roughly `O(n)` per recursion level, bounded depth), `O(n)` Space

---

## Problem Statement

A string is "nice" if, for every letter it contains, both its uppercase and lowercase forms appear in the string. Given a string `s`, return the longest nice substring (the earliest one if there's a tie), or an empty string if none exists.

---

## Approaches

1. **Brute Force — Check Every Substring**
2. **Optimal — Divide and Conquer on Invalid Characters**

---

# Approach 1 — Brute Force / Check Every Substring

## Idea

For every possible substring, check directly whether it's "nice" by confirming every letter in it appears in both cases. Track the longest nice substring found (preferring the earliest on ties).

## Dry Run

```text
s = "YazaAay"
```

Check substring `"Y"`: contains `Y` but no `y` → not nice.

Check substring `"Ya"`: contains `Y`, `a` — neither has its opposite case → not nice.

Continue checking all substrings... the substring `"aAa"` (indices `2..4`, i.e. `"zaA"` — rechecking indices: `s = Y,a,z,a,A,a,y`, so `"aAa"` is at indices `3..5`):

```text
"aAa": contains 'a' (lowercase) and 'A' (uppercase) → every letter has both cases → nice, length 3
```

No longer nice substring exists. Return `"aAa"`.

## Algorithm

1. Initialize `best = ""`.
2. For each start index `i`, for each end index `j >= i`:

   * Extract the substring `s[i..j]`.
   * Check whether every distinct letter in it appears in both lowercase and uppercase forms (e.g. via a set).
   * If nice and longer than `best`, update `best`.
3. Return `best`.

## Complexity

* **Time:** `O(n³)`

  * `O(n²)` substrings, each requiring an `O(n)` check of its contents to verify niceness.
* **Space:** `O(n)`

  * For the temporary substring and set used in each check.

## Notes / Tips

* Checking every substring independently ignores structure: if a single character in a substring has no counterpart anywhere in the string, no substring containing that character can ever be nice, so large portions of the search are provably wasted.
* The "tie goes to the earliest" requirement is naturally satisfied by only updating `best` when a **strictly** longer nice substring is found, scanning start indices in increasing order.

## Code

```cpp
class Solution {
public:
    bool isNice(const string& s) {
        set<char> chars(s.begin(), s.end());

        for (char c : chars) {
            if (!chars.count(tolower(c)) || !chars.count(toupper(c))) {
                return false;
            }
        }

        return true;
    }

    string longestNiceSubstring(string s) {
        string best = "";
        int n = s.size();

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                string sub = s.substr(i, j - i + 1);

                if (sub.size() > best.size() && isNice(sub)) {
                    best = sub;
                }
            }
        }

        return best;
    }
};
```

---

# Approach 2 — Optimal / Divide and Conquer on Invalid Characters

## Idea

If a character in the current string has no opposite-case counterpart anywhere in the string, no nice substring can contain it — so it acts as a "wall." Split the string at every such invalid character into separate pieces, and recursively find the longest nice substring in each piece. If the string has **no** invalid characters at all, the entire string is nice and is returned directly. The final answer is the longest result across all pieces (preferring the earliest on ties, since pieces are processed left to right).

## Dry Run

```text
s = "YazaAay"
```

Check which characters have no counterpart in `s`:

```text
'Y' (uppercase): is 'y' in s? yes (last character) → ok
'a': is 'A' in s? yes → ok
'z': is 'Z' in s? no → INVALID
'A': is 'a' in s? yes → ok
'y': is 'Y' in s? yes → ok
```

Split at the invalid character `'z'` (index `2`) into pieces: `"Ya"` (left) and `"aAay"` (right).

Recurse on `"Ya"`:

```text
'Y': is 'y' in "Ya"? no → INVALID
'a': is 'A' in "Ya"? no → INVALID
all characters invalid → splits into empty pieces → result: ""
```

Recurse on `"aAay"`:

```text
'a': is 'A' present? yes → ok
'A': is 'a' present? yes → ok
'a': ok
'y': is 'Y' in "aAay"? no → INVALID
```

Split at `'y'` (index `3` of this piece): pieces `"aAa"` (left) and `""` (right).

Recurse on `"aAa"`:

```text
every character has its opposite case present → no invalid characters → entire string is nice → return "aAa"
```

Overall best across all pieces: `"aAa"` (length `3`), matching the brute-force result.

## Algorithm

1. If the string is shorter than `2`, return `""` (a single character can never be nice).
2. Build a set of all characters present in `s`.
3. Scan `s` for the first character `c` whose opposite-case counterpart isn't in the set:

   * If none is found, the whole string is nice — return `s`.
   * Otherwise, split `s` at that character's index into a left piece and a right piece (excluding the invalid character itself).
   * Recursively compute the longest nice substring of each piece.
   * Return the longer of the two results (preferring the left one on ties, to get the earliest).

## Complexity

* **Time:** `O(n * 26)` roughly (bounded recursion depth)

  * Each recursion level scans its piece in `O(n)` work total; since each split removes at least one invalid character *type* from consideration in subsequent pieces, the recursion depth is bounded by the number of distinct letters (at most `52` including both cases), giving roughly `O(52 * n)` overall.
* **Space:** `O(n)`

  * For the recursion stack and temporary substrings created during splitting.

## Notes / Tips

* The correctness argument: a character with no counterpart *anywhere in the current string* can never be part of a nice substring of that string (since niceness requires its counterpart to also be present inside the substring, which is only possible if the counterpart exists somewhere in the string to begin with), so splitting around it never discards a valid answer.
* Preferring the left piece on ties (using `>=` when comparing lengths, or equivalently only replacing when strictly longer) is what correctly returns the **earliest** longest nice substring, as the problem requires.
* This divide-and-conquer-on-invalid-elements pattern also appears in problems like "Longest Substring with At Least K Repeating Characters" (LC 395) — whenever a global property rules out specific characters entirely, splitting around them lets the recursion handle only the portions that could still be valid.

## Code

```cpp
class Solution {
public:
    string longestNiceSubstring(string s) {
        if (s.size() < 2) {
            return "";
        }

        set<char> chars(s.begin(), s.end());

        for (int i = 0; i < s.size(); i++) {
            char c = s[i];

            if (!chars.count(tolower(c)) || !chars.count(toupper(c))) {
                string left = longestNiceSubstring(s.substr(0, i));
                string right = longestNiceSubstring(s.substr(i + 1));

                return left.size() >= right.size() ? left : right;
            }
        }

        return s;
    }
};
```

---

## Key Template

```text
function longestNice(s):
    if len(s) < 2: return ""

    chars = set of characters in s

    for i, c in enumerate(s):
        if swapcase(c) not in chars:
            left = longestNice(s[0:i])
            right = longestNice(s[i+1:])
            return left if len(left) >= len(right) else right

    return s
```