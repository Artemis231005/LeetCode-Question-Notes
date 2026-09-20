# LeetCode 796 — Rotate String

## Metadata

* **LeetCode:** 796
* **Problem:** Rotate String
* **Difficulty:** Easy
* **Topics:** String, String Matching
* **Pattern:** Doubled String Substring Check
* **Key Technique:** A string `goal` is some rotation of `s` if and only if `goal` appears as a substring of `s + s` — concatenating `s` with itself lets every possible rotation appear as a contiguous window
* **Optimal Complexity:** `O(n)` Time (average, using an efficient substring search), `O(n)` Space

---

## Problem Statement

Given two strings `s` and `goal`, return `true` if `goal` can be obtained by rotating `s` some number of positions (moving a prefix of `s` to its end).

---

## Approaches

1. **Brute Force — Try Every Rotation Directly**
2. **Optimal — Doubled String Substring Check**

---

# Approach 1 — Brute Force / Try Every Rotation Directly

## Idea

Generate every possible rotation of `s` by slicing and concatenating, and check whether any of them equals `goal`.

## Dry Run

```text
s = "abcde", goal = "cdeab"
```

Rotation `r = 0`:

```text
"abcde" == "cdeab"? no
```

Rotation `r = 1`:

```text
"bcdea" == "cdeab"? no
```

Rotation `r = 2`:

```text
"cdeab" == "cdeab"? yes → return true
```

## Algorithm

1. If `s.length() != goal.length()`, return `false` immediately.
2. For each rotation amount `r` from `0` to `n-1`:

   * Build `rotated = s.substr(r) + s.substr(0, r)`.
   * If `rotated == goal`, return `true`.
3. If no rotation matches, return `false`.

## Complexity

* **Time:** `O(n²)`

  * For each of the `n` possible rotations, building the rotated string and comparing it to `goal` both take `O(n)`.
* **Space:** `O(n)`

  * For the temporary rotated string built at each step.

## Notes / Tips

* Rebuilding and comparing a full rotated string for every possible rotation amount is redundant — every rotation of `s` is already sitting somewhere inside `s + s` as a contiguous substring, which the optimal approach exploits directly instead of generating rotations one at a time.
* This is the same "physically rotate and check" inefficiency seen in LC 503 and LC 4043 before applying the doubled-array/string trick.

## Code

```cpp
class Solution {
public:
    bool rotateString(string s, string goal) {
        int n = s.size();
        if (n != (int)goal.size()) {
            return false;
        }

        for (int r = 0; r < n; r++) {
            string rotated = s.substr(r) + s.substr(0, r);
            if (rotated == goal) {
                return true;
            }
        }

        return false;
    }
};
```

---

# Approach 2 — Optimal / Doubled String Substring Check

## Idea

Every rotation of `s` corresponds exactly to a length-`n` window somewhere inside `s + s` (concatenating `s` with itself once). So instead of generating and comparing each rotation individually, just check whether `goal` appears anywhere as a substring of `s + s` — if it does, some rotation of `s` equals `goal`; if not, none does.

## Dry Run

```text
s = "abcde", goal = "cdeab"
```

Build the doubled string:

```text
s + s = "abcdeabcde"
```

Search for `goal = "cdeab"` as a substring:

```text
"abcdeabcde"
    ^^^^^
found starting at index 2 → "cdeab" matches
```

Return `true`.

### Non-matching example

```text
s = "abcde", goal = "abced"
```

```text
s + s = "abcdeabcde"
```

`"abced"` does not appear anywhere in `"abcdeabcde"` as a contiguous substring → return `false`.

## Algorithm

1. If `s.length() != goal.length()`, return `false` immediately (a rotation can never change a string's length).
2. Build `doubled = s + s`.
3. Return whether `doubled` contains `goal` as a substring.

## Complexity

* **Time:** `O(n)` on average

  * Using an efficient substring search (like C++'s `std::string::find`, which typically performs well in practice, or a linear-time algorithm like KMP for a guaranteed worst-case bound), searching a string of length `2n` for a pattern of length `n` takes roughly linear time.
* **Space:** `O(n)`

  * For the doubled string `s + s`.

## Notes / Tips

* The length check upfront (`s.length() != goal.length()`) is essential as without it, a shorter or longer `goal` could still coincidentally appear as a substring of `s + s` (e.g. a short `goal` matching part of one rotation), which would incorrectly return `true`.
* This exact "double the string to represent all rotations as fixed-length windows" trick is the same core idea used in LC 503 (Next Greater Element II, applied to an array) and LC 4043 (Count Rotations With Exactly K Equal Adjacent Pairs) — recognizing that circular/rotational structure collapses into a simple substring or subarray check once the sequence is doubled.
* `std::string::find` in C++ has a worst-case complexity of `O(n * m)` for naive implementations, though typical standard library implementations perform well in practice; for a guaranteed linear worst case, a proper substring-matching algorithm like KMP or Z-function would be needed — usually unnecessary at this problem's typical constraint sizes.

## Code

```cpp
class Solution {
public:
    bool rotateString(string s, string goal) {
        if (s.size() != goal.size()) {
            return false;
        }

        string doubled = s + s;
        return doubled.find(goal) != string::npos;
    }
};
```

---

## Key Template

```text
if len(s) != len(goal): return false

doubled = s + s
return goal in doubled
```