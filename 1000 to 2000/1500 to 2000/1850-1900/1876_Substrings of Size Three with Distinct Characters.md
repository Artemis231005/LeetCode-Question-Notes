# LeetCode 1876 — Substrings of Size Three with Distinct Characters

## Metadata

* **LeetCode:** 1876
* **Problem:** Substrings of Size Three with Distinct Characters
* **Difficulty:** Easy
* **Topics:** Hash Table, String, Sliding Window, Counting
* **Pattern:** Fixed-Size Sliding Window (Size 3)
* **Key Technique:** Slide a window of length 3 across the string and check whether its three characters are pairwise distinct — since the window size is a fixed constant, three direct comparisons are enough, with no hash set or frequency structure needed
* **Optimal Complexity:** `O(n)` Time, `O(1)` Space

---

## Problem Statement

Given a string `s`, return the number of "good substrings" of length `3` — substrings where all three characters are distinct. If the same good substring appears at multiple positions, each occurrence counts separately.

---

## Approaches

1. **Brute Force — Hash Set Check Per Window**
2. **Optimal — Direct Pairwise Comparison**

---

# Approach 1 — Brute Force / Hash Set Check Per Window

## Idea

For every window of length `3`, insert its three characters into a hash set. If the set's final size is `3`, all characters are distinct and the window counts as good.

## Dry Run

```text
s = "xyzzaz"
```

Window `"xyz"` (indices 0-2):

```text
set = {x, y, z} → size 3 → good, count = 1
```

Window `"yzz"` (indices 1-3):

```text
set = {y, z} → size 2 → not good
```

Window `"zza"` (indices 2-4):

```text
set = {z, a} → size 2 → not good
```

Window `"zaz"` (indices 3-5):

```text
set = {z, a} → size 2 → not good
```

Final count: `1`.

## Algorithm

1. Initialize `count = 0`.
2. For each start index `i` from `0` to `n - 3`:

   * Insert `s[i]`, `s[i+1]`, `s[i+2]` into a fresh hash set.
   * If the set's size is `3`, increment `count`.
3. Return `count`.

## Complexity

* **Time:** `O(n)`

  * Each of the `n - 2` windows does a constant amount of work (inserting 3 characters into a set).
* **Space:** `O(1)`

  * The set never holds more than 3 characters at a time, regardless of input length.

## Notes / Tips

* Building a hash set for every window is correct but heavier than necessary — with a fixed window size of only `3`, three direct character comparisons accomplish the same thing without any container overhead.
* Both approaches have the same asymptotic complexity here; the difference is purely constant-factor overhead from allocating and populating a set per window.

## Code

```cpp
class Solution {
public:
    int countGoodSubstrings(string s) {
        int count = 0;

        for (int i = 0; i + 2 < (int)s.size(); i++) {
            unordered_set<char> window;
            window.insert(s[i]);
            window.insert(s[i + 1]);
            window.insert(s[i + 2]);

            if (window.size() == 3) {
                count++;
            }
        }

        return count;
    }
};
```

---

# Approach 2 — Optimal / Direct Pairwise Comparison

## Idea

Since the window size is fixed at exactly `3`, there are only three pairs of characters to compare: `(s[i], s[i+1])`, `(s[i], s[i+2])`, and `(s[i+1], s[i+2])`. The window is good exactly when all three pairs differ — no hash set, array, or other structure is needed at all.

## Dry Run

```text
s = "xyzzaz"
```

Window `i=0` (`"xyz"`):

```text
x != y ✓, x != z ✓, y != z ✓ → good, count = 1
```

Window `i=1` (`"yzz"`):

```text
y != z ✓, y != z ✓, z != z ✗ → not good
```

Window `i=2` (`"zza"`):

```text
z != z ✗ → not good
```

Window `i=3` (`"zaz"`):

```text
z != a ✓, z != z ✗ → not good
```

Final count: `1`, matching the brute-force result.

## Algorithm

1. Initialize `count = 0`.
2. For each start index `i` from `0` to `n - 3`:

   * If `s[i] != s[i+1]` and `s[i] != s[i+2]` and `s[i+1] != s[i+2]`, increment `count`.
3. Return `count`.

## Complexity

* **Time:** `O(n)`

  * One pass, with three constant-time comparisons per window.
* **Space:** `O(1)`

  * No auxiliary structures at all — just a counter.

## Notes / Tips

* This only works as cleanly as it does because the window size is a small fixed constant — for a general window size `k`, comparing every pair would cost `O(k²)` per window, and a sliding frequency structure (like in LC 3 — Longest Substring Without Repeating Characters) would be the right tool instead.
* This is the simplest possible instance of the fixed-size sliding window pattern — the window "slides" implicitly just by incrementing `i`, with no add/remove bookkeeping needed since the check is recomputed directly from the three characters at each position.
* Since only `n - 2` windows exist, the loop bound `i + 2 < n` (or equivalently `i <= n - 3`) is what prevents reading past the end of the string.

## Code

```cpp
class Solution {
public:
    int countGoodSubstrings(string s) {
        int count = 0;

        for (int i = 0; i + 2 < (int)s.size(); i++) {
            if (s[i] != s[i + 1] && s[i] != s[i + 2] && s[i + 1] != s[i + 2]) {
                count++;
            }
        }

        return count;
    }
};
```

---

## Key Template

```text
count = 0

for i in 0..n-3:
    if s[i] != s[i+1] and s[i] != s[i+2] and s[i+1] != s[i+2]:
        count += 1

return count
```