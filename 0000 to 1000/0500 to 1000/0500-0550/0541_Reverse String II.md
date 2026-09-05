# LeetCode 541 — Reverse String II

## Metadata

* **LeetCode:** 541
* **Problem:** Reverse String II
* **Difficulty:** Easy
* **Topics:** String, Two Pointers
* **Pattern:** Chunked Two-Pointer Reversal
* **Key Technique:** Jump through the string in blocks of `2k`, reversing only the first `k` characters of each block, with bounds handling for the final short block
* **Optimal Complexity:** `O(n)` Time, `O(1)` Auxiliary Space

---

## Problem Statement

Given a string `s` and an integer `k`, reverse the first `k` characters for every `2k` characters starting from the beginning of the string. If fewer than `k` characters remain, reverse all of them. If between `k` and `2k` characters remain, reverse only the first `k` and leave the rest unchanged.

---

## Approaches

1. **Brute Force — Build a New String with Substring Slicing**
2. **Optimal — In-Place Two-Pointer Reversal per Block**

---

# Approach 1 — Brute Force / Build a New String with Substring Slicing

## Idea

Walk through the string in steps of `2k`. For each step, slice out the relevant substring, reverse just the portion that needs reversing (using a helper or built-in reverse), and concatenate everything into a new result string.

## Dry Run

```text
s = "abcdefg", k = 2
```

Step at `i = 0`:

```text
block = "abcdefg"[0..3] = "abcd" (up to 2k=4 chars)
reverse first k=2 chars of "ab" → "ba"
combine: "ba" + "cd" = "bacd"
```

Step at `i = 4`:

```text
remaining = "efg" (3 chars, between k=2 and 2k=4)
reverse first k=2 chars of "ef" → "fe"
combine: "fe" + "g" = "feg"
```

Concatenate all pieces:

```text
"bacd" + "feg" = "bacdfeg"
```

## Algorithm

1. Initialize `result = ""`.
2. For `i` from `0` to `n-1`, stepping by `2k`:

   * Take the substring `s[i .. i+2k-1]` (or to the end if shorter).
   * Reverse the first `min(k, length of this substring)` characters of it.
   * Append the modified substring to `result`.
3. Return `result`.

## Complexity

* **Time:** `O(n)`

  * Every character is visited and copied a constant number of times across all the substring/reverse/concatenate operations.
* **Space:** `O(n)`

  * A brand new result string is built, along with temporary substrings for each block.

## Notes / Tips

* Correct and reasonably clean, but string slicing and concatenation typically allocate new memory each time, which is avoidable since the reversal can be done directly on the original string.
* Good for languages where strings are immutable and in-place mutation isn't natural (e.g. Python) — less ideal in C++ where in-place reversal is straightforward.

## Code

```cpp
class Solution {
public:
    string reverseStr(string s, int k) {
        string result = "";
        int n = s.size();

        for (int i = 0; i < n; i += 2 * k) {
            string block = s.substr(i, min(2 * k, n - i));
            int reverseLen = min(k, (int)block.size());

            reverse(block.begin(), block.begin() + reverseLen);
            result += block;
        }

        return result;
    }
};
```

---

# Approach 2 — Optimal / In-Place Two-Pointer Reversal per Block

## Idea

Instead of building substrings, work directly on the original string. Jump through it in steps of `2k`, and at each step, reverse the range `[i, i + k - 1]` in place — but clamp the right boundary of that range to `s.size() - 1` in case fewer than `k` characters remain at the very end.

## Dry Run

```text
s = "abcdefg", k = 2
```

`i = 0`:

```text
range to reverse: [0, min(0+2-1, 6)] = [0, 1] → "ab" → reverse → "ba"
s = "bacdefg"
```

`i = 4`:

```text
range to reverse: [4, min(4+2-1, 6)] = [4, 5] → "ef" → reverse → "fe"
s = "bacdfeg"
```

Loop ends (`i = 8 >= n = 7`).

Final:

```text
"bacdfeg"
```

## Algorithm

1. Let `n = s.size()`.
2. For `i` from `0` to `n-1`, stepping by `2k`:

   * Compute `left = i`, `right = min(i + k - 1, n - 1)`.
   * Reverse `s[left..right]` in place using a standard two-pointer swap.
3. Return `s`.

## Complexity

* **Time:** `O(n)`

  * Each character is touched by at most one reversal swap across the entire string.
* **Space:** `O(1)`

  * The reversal is done directly on the input string (or a mutable copy passed by value) — no extra buffers or substrings created.

## Notes / Tips

* Clamping `right` with `min(i + k - 1, n - 1)` is what correctly handles the final partial block, whether it has fewer than `k` characters (reverse all of it) or between `k` and `2k` (reverse exactly the first `k`, since the leftover beyond `k` is simply never included in the reversal range).
* This is the standard block-processing template — same "jump by fixed stride, act differently within each block" shape shows up in problems that group elements or characters in fixed-size chunks.
* No separate case-check is needed for "fewer than k remain" vs "between k and 2k remain" — the `min()` clamp naturally handles both.

## Code

```cpp
class Solution {
public:
    string reverseStr(string s, int k) {
        int n = s.size();

        for (int i = 0; i < n; i += 2 * k) {
            int left = i;
            int right = min(i + k - 1, n - 1);

            while (left < right) {
                swap(s[left], s[right]);
                left++;
                right--;
            }
        }

        return s;
    }
};
```

---

## Key Template

```text
n = s.size()

for i = 0 to n-1, step 2k:
    left = i
    right = min(i + k - 1, n - 1)

    while left < right:
        swap(s[left], s[right])
        left++
        right--

return s
```