# LeetCode 187 — Repeated DNA Sequences

## Metadata

* **LeetCode:** 187
* **Problem:** Repeated DNA Sequences
* **Difficulty:** Medium
* **Topics:** Hash Table, String, Bit Manipulation, Sliding Window, Rolling Hash
* **Pattern:** Fixed-Size Sliding Window + Bitmask Encoding (Rolling Hash)
* **Key Technique:** Encode each DNA letter in 2 bits so a window of 10 letters fits in a 20-bit integer, then update that integer in `O(1)` as the window slides — no substring is ever built or hashed
* **Optimal Complexity:** `O(n)` Time, `O(n)` Space (bounded by the `2^20` possible masks)

---

## Problem Statement

Given a DNA string `s` made of the letters `A`, `C`, `G`, `T`, return all 10-letter-long substrings that occur **more than once** in `s`. The answer may be returned in any order.

---

## Approaches

1. **Brute Force — Compare Every Pair of Windows**
2. **Better — Hash Map of Substrings**
3. **Optimal — Rolling Bitmask Encoding**

---

# Approach 1 — Brute Force / Compare Every Pair of Windows

## Idea

For every 10-letter window, compare it against every later window. If two windows are identical, that sequence is repeated, so record it in a set (the set avoids reporting the same sequence more than once).

## Dry Run

```text
s = "AAAAAAAAAAAAA"   (13 A's)
```

Windows start at indices `0, 1, 2, 3`, and every one is `"AAAAAAAAAA"`.

```text
compare window 0 with window 1 → equal → add "AAAAAAAAAA" to set
compare window 0 with window 2 → equal → already in set
compare window 0 with window 3 → equal → already in set
compare window 1 with window 2 → equal → already in set
...
```

Result: `["AAAAAAAAAA"]`.

## Algorithm

1. Initialize an empty set `result`.
2. For each start index `i` from `0` to `n - 10`:

   * For each later start index `j` from `i + 1` to `n - 10`:

     * If `s[i..i+9] == s[j..j+9]`, insert that substring into `result`.
3. Return the contents of `result`.

## Complexity

* **Time:** `O(n² * L)`

  * `O(n²)` pairs of windows, and each comparison takes up to `L = 10` character checks.
* **Space:** `O(1)` (beyond the output)

  * Only loop counters, plus the result set that holds the answer.

## Notes / Tips

* Comparing every pair of windows is far too slow for `n` up to `10^5`, which makes this useful only as a correctness baseline.
* The set is required because a sequence that appears 3 or more times would otherwise be reported once per matching pair.

## Code

```cpp
class Solution {
public:
    vector<string> findRepeatedDnaSequences(string s) {
        int n = s.size();
        set<string> result;

        for (int i = 0; i + 10 <= n; i++) {
            for (int j = i + 1; j + 10 <= n; j++) {
                if (s.compare(i, 10, s, j, 10) == 0) {
                    result.insert(s.substr(i, 10));
                }
            }
        }

        return vector<string>(result.begin(), result.end());
    }
};
```

---

# Approach 2 — Better / Hash Map of Substrings

## Idea

Slide a window of length 10 across the string once. Use a hash map from substring to its occurrence count. The moment a substring's count reaches exactly `2`, add it to the result — reaching `2` happens once per distinct repeated sequence, so no duplicates are added.

## Dry Run

```text
s = "AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT"
```

```text
start 0:  "AAAAACCCCC" → count 1
start 5:  "CCCCCAAAAA" → count 1
start 10: "AAAAACCCCC" → count 2 → add to result
start 16: "CCCCCAAAAA" → count 2 → add to result
(every other window appears only once)
```

Result: `["AAAAACCCCC", "CCCCCAAAAA"]`.

## Algorithm

1. Initialize an empty hash map `freq` and an empty list `result`.
2. For each start index `i` from `0` to `n - 10`:

   * `sub = s.substr(i, 10)`.
   * Increment `freq[sub]`.
   * If `freq[sub] == 2`, append `sub` to `result`.
3. Return `result`.

## Complexity

* **Time:** `O(n * L)`

  * `O(n)` windows, and each substring extraction and hash computation costs `O(L)` with `L = 10`.
* **Space:** `O(n * L)`

  * The map can hold up to `n` distinct 10-character strings.

## Notes / Tips

* Checking `== 2` rather than `>= 2` is what keeps each repeated sequence in the answer exactly once, even if it occurs many times.
* This is already linear in `n`, but it still creates and hashes a fresh 10-character string for every window, which the bitmask approach avoids.

## Code

```cpp
class Solution {
public:
    vector<string> findRepeatedDnaSequences(string s) {
        int n = s.size();
        unordered_map<string, int> freq;
        vector<string> result;

        for (int i = 0; i + 10 <= n; i++) {
            string sub = s.substr(i, 10);

            if (++freq[sub] == 2) {
                result.push_back(sub);
            }
        }

        return result;
    }
};
```

---

# Approach 3 — Optimal / Rolling Bitmask Encoding

## Idea

There are only 4 letters, so each one fits in 2 bits: `A = 00`, `C = 01`, `G = 10`, `T = 11`. Ten letters therefore fit in exactly 20 bits, so a window can be stored as a plain integer. Sliding the window by one letter is a constant-time update: shift the mask left by 2 bits, OR in the new letter's code, and mask with `(1 << 20) - 1` to drop the bits of the letter that left. Count how often each integer has appeared, and add the substring to the result when its count first reaches `2`.

## Dry Run

```text
s = "AAAAACCCCCAAAAACCCCCCAAAAAGGGTTT"
```

Encoding of the two repeated windows:

```text
"AAAAACCCCC" → 00 00 00 00 00 01 01 01 01 01 = 341
"CCCCCAAAAA" → 01 01 01 01 01 00 00 00 00 00 = 349184
```

Rolling through the string (a window is complete once index `i >= 9`):

```text
i=9:   window "AAAAACCCCC" (start 0)  → mask 341     → count 1
i=14:  window "CCCCCAAAAA" (start 5)  → mask 349184  → count 1
i=19:  window "AAAAACCCCC" (start 10) → mask 341     → count 2 → add s.substr(10, 10)
i=25:  window "CCCCCAAAAA" (start 16) → mask 349184  → count 2 → add s.substr(16, 10)
```

Result: `["AAAAACCCCC", "CCCCCAAAAA"]`, matching the hash map approach.

## Algorithm

1. If `n < 10`, return an empty list.
2. Initialize `mask = 0`, an empty count map (or a count array of size `2^20`), and an empty `result`.
3. For each index `i` from `0` to `n - 1`:

   * `mask = ((mask << 2) | code(s[i])) & ((1 << 20) - 1)`.
   * If `i >= 9` (a full window has been read):

     * Increment `count[mask]`.
     * If `count[mask] == 2`, append `s.substr(i - 9, 10)` to `result`.
4. Return `result`.

## Complexity

* **Time:** `O(n)`

  * One pass; every mask update and map operation is `O(1)`, and a substring is only built when a sequence is added to the answer.
* **Space:** `O(n)`

  * The count map stores one small integer key per distinct window, and there can never be more than `2^20` distinct keys.

## Notes / Tips

* The `& ((1 << 20) - 1)` step is what slides the window: it discards the 2 bits of the letter that just fell out of the front of the window, keeping exactly 10 letters' worth of bits.
* Comparing and hashing a single integer is cheaper than hashing a 10-character string, and no substring is created for windows that never become part of the answer.
* This is the same rolling window idea as LC 2269 (rolling digit window), except the "digits" are 2-bit letters in base 4.
* The 2-bit encoding only works because the alphabet has exactly 4 symbols. For a larger alphabet, a polynomial rolling hash (Rabin-Karp) would be needed, which can have collisions.
* Common mistake: using `count >= 2` instead of `== 2` when adding to the result, which would append the same sequence multiple times.

## Code

```cpp
class Solution {
public:
    vector<string> findRepeatedDnaSequences(string s) {
        int n = s.size();
        if (n < 10) {
            return {};
        }

        auto code = [](char c) {
            switch (c) {
                case 'A': return 0;
                case 'C': return 1;
                case 'G': return 2;
                default:  return 3; // 'T'
            }
        };

        const int MASK = (1 << 20) - 1;
        unordered_map<int, int> freq;
        vector<string> result;
        int mask = 0;

        for (int i = 0; i < n; i++) {
            mask = ((mask << 2) | code(s[i])) & MASK;

            if (i >= 9) {
                if (++freq[mask] == 2) {
                    result.push_back(s.substr(i - 9, 10));
                }
            }
        }

        return result;
    }
};
```

---

## Key Template

```text
if n < L: return []

code = {A: 0, C: 1, G: 2, T: 3}
MASK = (1 << (2 * L)) - 1     # L = 10 → 20 bits
mask = 0
freq = {}
result = []

for i in 0..n-1:
    mask = ((mask << 2) | code[s[i]]) & MASK

    if i >= L - 1:
        freq[mask] += 1
        if freq[mask] == 2:
            result.append(s[i - L + 1 .. i])

return result
```