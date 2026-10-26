# LeetCode 3090 — Maximum Length Substring With Two Occurrences

## Metadata

* **LeetCode:** 3090
* **Problem:** Maximum Length Substring With Two Occurrences
* **Difficulty:** Easy
* **Topics:** Hash Table, String, Sliding Window
* **Pattern:** Variable-Size Sliding Window with Frequency Count
* **Key Technique:** Expand the window to the right, and whenever any character's count exceeds `2`, shrink from the left until that character's count drops back to `2` — the window is always valid after shrinking, so its length can be tracked directly
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space (fixed 26-letter alphabet)

---

## Problem Statement

Given a string `s`, return the maximum length of a substring such that it contains **at most two occurrences** of each character.

---

## Approaches

1. **Brute Force — Check Every Substring**
2. **Optimal — Sliding Window with Frequency Array**

---

# Approach 1 — Brute Force / Check Every Substring

## Idea

For every possible substring, count the frequency of each character inside it. If no character appears more than twice, the substring is valid — track the longest valid one.

## Dry Run

```text
s = "bcbbbcba"
```

Check substring `"bcbb"` (indices 0-3):

```text
b appears 3 times → exceeds 2 → invalid
```

Check substring `"bcb"` (indices 0-2):

```text
b: 2, c: 1 → valid, length 3
```

Check substring `"bbcb"` (indices 3-6... exact indices: `s = b,c,b,b,b,c,b,a`; substring `"bbcb"` at indices 4-7 would be `b,c,b,a` — rechecking: the valid longest is `"bcbb"`? No, that has three b's.)

Continuing to check all substrings, the longest valid one found has length `4` (e.g. `"bcbb"` is invalid, but `"bbcb"` at indices `3..6` = `b,b,c,b` has three b's — invalid; the valid length-4 substring is `"bcba"`... exact validity depends on character counts, and the longest valid substring across all candidates has length `4`).

Final answer: `4`.

## Algorithm

1. Initialize `best = 0`.
2. For each start index `i`, for each end index `j >= i`:

   * Count each character's frequency in `s[i..j]`.
   * If every character's count is `<= 2`, update `best = max(best, j - i + 1)`.
3. Return `best`.

## Complexity

* **Time:** `O(n³)`

  * `O(n²)` substrings, each requiring an `O(n)` frequency count to validate.
* **Space:** `O(1)`

  * A fixed-size frequency array of `26` per check.

## Notes / Tips

* Recounting characters from scratch for every substring ignores that overlapping substrings share almost all of their content — a sliding window maintains the counts incrementally instead.
* Even a smarter brute force that breaks early (once a substring becomes invalid, any longer substring from the same start will also be invalid) still re-derives window contents repeatedly across different start indices.

## Code

```cpp
class Solution {
public:
    int maximumLengthSubstring(string s) {
        int n = s.size();
        int best = 0;

        for (int i = 0; i < n; i++) {
            for (int j = i; j < n; j++) {
                vector<int> freq(26, 0);
                bool valid = true;

                for (int k = i; k <= j; k++) {
                    freq[s[k] - 'a']++;
                    if (freq[s[k] - 'a'] > 2) {
                        valid = false;
                        break;
                    }
                }

                if (valid) {
                    best = max(best, j - i + 1);
                }
            }
        }

        return best;
    }
};
```

---

# Approach 2 — Optimal / Sliding Window with Frequency Array

## Idea

Maintain a window `[left, right]` with a frequency array tracking how many times each character appears inside it. Expand `right` one character at a time, incrementing that character's count. If the new count exceeds `2`, shrink the window from the left — decrementing counts as characters leave — until the offending character's count is back to `2`. After this adjustment, the window is always valid, so its length `right - left + 1` is a candidate for the answer.

## Dry Run

```text
s = "bcbbbcba"
```

Initialize `left = 0`, `freq` all zero, `best = 0`.

```text
right=0, 'b': freq[b]=1 → ok → window "b" length 1 → best=1
right=1, 'c': freq[c]=1 → ok → window "bc" length 2 → best=2
right=2, 'b': freq[b]=2 → ok → window "bcb" length 3 → best=3
right=3, 'b': freq[b]=3 → exceeds 2! shrink:
    remove s[0]='b' → freq[b]=2, left=1 → now valid
    window "cbb" (indices 1-3) length 3 → best stays 3
right=4, 'b': freq[b]=3 → exceeds 2! shrink:
    remove s[1]='c' → freq[c]=0, left=2
    still freq[b]=3 → remove s[2]='b' → freq[b]=2, left=3 → now valid
    window "bb" (indices 3-4) length 2 → best stays 3
right=5, 'c': freq[c]=1 → ok → window "bbc" (indices 3-5) length 3 → best stays 3
right=6, 'b': freq[b]=3 → exceeds 2! shrink:
    remove s[3]='b' → freq[b]=2, left=4 → now valid
    window "bcb" (indices 4-6) length 3 → best stays 3
right=7, 'a': freq[a]=1 → ok → window "bcba" (indices 4-7) length 4 → best=4
```

Final answer: `4`, matching the brute-force result.

## Algorithm

1. Initialize `left = 0`, `freq` array of size `26`, all zeros, and `best = 0`.
2. For each `right` from `0` to `n - 1`:

   * Increment `freq[s[right] - 'a']`.
   * While `freq[s[right] - 'a'] > 2`: decrement `freq[s[left] - 'a']` and increment `left`.
   * Update `best = max(best, right - left + 1)`.
3. Return `best`.

## Complexity

* **Time:** `O(n)`

  * `left` and `right` each move forward at most `n` times total, giving amortized linear time despite the nested-looking `while` loop.
* **Space:** `O(1)`

  * The frequency array has a fixed size of `26` regardless of input length.

## Notes / Tips

* Only the character that was just added (`s[right]`) can possibly cause a violation, since every other character's count was already `<= 2` before this step — so the shrink loop only needs to check that single character's count, not scan the whole frequency array.
* Shrinking continues until `s[right]`'s count is `2` again, which may require removing characters other than `s[right]` along the way (like removing `'c'` in the dry run above) — this is fine, since those removals only reduce counts, never causing new violations.
* This is the standard variable-size sliding window template (shrink-while-invalid, then record) — the same shape as LC 3 (Longest Substring Without Repeating Characters), just with the allowed occurrence count raised from `1` to `2`.

## Code

```cpp
class Solution {
public:
    int maximumLengthSubstring(string s) {
        int n = s.size();
        vector<int> freq(26, 0);
        int left = 0, best = 0;

        for (int right = 0; right < n; right++) {
            freq[s[right] - 'a']++;

            while (freq[s[right] - 'a'] > 2) {
                freq[s[left] - 'a']--;
                left++;
            }

            best = max(best, right - left + 1);
        }

        return best;
    }
};
```

---

## Key Template

```text
freq = array of size 26, all 0
left = 0
best = 0

for right in 0..n-1:
    freq[s[right]] += 1

    while freq[s[right]] > 2:
        freq[s[left]] -= 1
        left += 1

    best = max(best, right - left + 1)

return best
```