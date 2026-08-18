# LeetCode 3 — Longest Substring Without Repeating Characters

## Metadata

* **LeetCode:** 3
* **Problem:** Longest Substring Without Repeating Characters
* **Difficulty:** Medium
* **Topics:** String, Hash Table, Sliding Window
* **Pattern:** Sliding Window
* **Key Pattern:** Maintain a window containing only unique characters and shrink it when a duplicate appears
* **Key Template:** Sliding Window — Variable Size
* **Key Technique:** Track the last occurrence of each character to jump the left pointer forward
* **Optimal Complexity:** `O(n)` time, `O(k)` space

---

# Approaches

1. **Brute Force — Generate Every Substring**
2. **Better — Sliding Window + Set**
3. **Optimal — Sliding Window + Last Occurrence Map**

---

# Approach 1 — Brute Force / Generate Every Substring

## Idea

Generate every possible substring and check whether it contains duplicate characters.

For every starting index `i`, extend the substring one character at a time.

If a duplicate is encountered, stop extending that substring.

## Dry Run

```text
s = "abcabcbb"
```

Starting from index `0`:

```text
"a"    → unique
"ab"   → unique
"abc"  → unique
"abca" → duplicate 'a' → stop
```

Current maximum:

```text
3
```

Starting from index `1`:

```text
"b"
"bc"
"bca"
"bcab" → duplicate 'b'
```

Again:

```text
3
```

Final answer:

```text
3
```

The longest substrings are `"abc"` and `"bca"`.

## Algorithm

1. Initialize `maxLength = 0`.
2. For every starting index `i`:

   * Create an empty set.
   * Extend the substring using `j`.
3. If `s[j]` is already in the set:

   * Stop this substring.
4. Otherwise:

   * Add `s[j]` to the set.
   * Update `maxLength`.
5. Return `maxLength`.

## Complexity

* **Time:** `O(n²)`
* **Space:** `O(k)`

where `k` is the character set size.

## Notes / Tips

* This is better than explicitly generating and storing all substrings because we stop immediately when a duplicate appears.
* Still too slow for large strings.

## Code

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        int maxLength = 0;

        for (int i = 0; i < s.size(); i++) {
            unordered_set<char> st;

            for (int j = i; j < s.size(); j++) {
                if (st.find(s[j]) != st.end()) {
                    break;
                }

                st.insert(s[j]);
                maxLength = max(maxLength, j - i + 1);
            }
        }

        return maxLength;
    }
};
```

---

# Approach 2 — Better / Sliding Window + Set

## Idea

Instead of starting over for every index, maintain one sliding window.

The window represents:

```text
[left ... right]
```

and must always contain **unique characters**.

Expand the window by moving `right`.

If `s[right]` is already present:

* Remove characters from the left.
* Move `left` forward until the duplicate is removed.

Then update the maximum window length.

## Dry Run

```text
s = "abcabcbb"
```

Start:

```text
left = 0
right = 0
```

Window progression:

```text
"a"       → length 1
"ab"      → length 2
"abc"     → length 3
```

Next character:

```text
"a"
```

Duplicate found.

Remove from the left:

```text
"abc"
 ↓
"bc"
```

Then add the new `a`:

```text
"bca"
```

Length:

```text
3
```

Continue similarly.

Final answer:

```text
3
```

## Algorithm

1. Initialize:

   ```text
   left = 0
   maxLength = 0
   ```
2. Create a set containing the characters currently inside the window.
3. Move `right` from left to right.
4. While `s[right]` already exists in the set:

   * Remove `s[left]`.
   * Increment `left`.
5. Add `s[right]` to the set.
6. Update:

   ```text
   maxLength = max(maxLength, right - left + 1)
   ```
7. Return `maxLength`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(k)`

Each character is inserted and removed at most once.

## Notes / Tips

* This is the standard variable-size sliding-window approach.
* The window invariant is:

  > **There are no duplicate characters inside `[left, right]`.**
* The `while` loop is important because multiple characters may need to be removed.

## Code

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        unordered_set<char> st;

        int left = 0;
        int maxLength = 0;

        for (int right = 0; right < s.size(); right++) {
            while (st.find(s[right]) != st.end()) {
                st.erase(s[left]);
                left++;
            }

            st.insert(s[right]);

            maxLength = max(maxLength, right - left + 1);
        }

        return maxLength;
    }
};
```

---

# Approach 3 — Optimal / Sliding Window + Last Occurrence

## Idea

The previous approach may move `left` one position at a time.

We can do better conceptually by storing the **last index where each character appeared**.

When we encounter a duplicate, instead of repeatedly removing characters, directly jump `left` past the previous occurrence.

For a character `s[right]` whose previous occurrence was at index `last[s[right]]`:

```text
left = last[s[right]] + 1
```

However, `left` must never move backward.

Therefore:

```text
left = max(left, last[s[right]] + 1)
```

## Dry Run

```text
s = "abcabcbb"
```

Start:

```text
left = 0
```

### `right = 0`

```text
a
```

Last occurrence:

```text
a → 0
```

Window:

```text
" a "
```

Length:

```text
1
```

### `right = 1`

```text
b
```

Window:

```text
"ab"
```

Length:

```text
2
```

### `right = 2`

```text
c
```

Window:

```text
"abc"
```

Length:

```text
3
```

### `right = 3`

Character:

```text
a
```

Previous `a` was at index `0`.

Therefore:

```text
left = max(0, 0 + 1)
     = 1
```

New window:

```text
"bca"
```

Length:

```text
3
```

### `right = 4`

```text
b
```

Previous `b` was at index `1`.

```text
left = max(1, 1 + 1)
     = 2
```

Window:

```text
"cab"
```

Length:

```text
3
```

Continue similarly.

Final answer:

```text
3
```

## Algorithm

1. Create a map/array storing the last occurrence of each character.
2. Initialize:

   ```text
   left = 0
   maxLength = 0
   ```
3. Traverse the string using `right`.
4. If the current character has appeared before:

   * Move `left` to:

     ```text
     max(left, lastOccurrence + 1)
     ```
5. Update the character's last occurrence:

   ```text
   lastOccurrence[s[right]] = right
   ```
6. Calculate the current window length:

   ```text
   right - left + 1
   ```
7. Update `maxLength`.
8. Return `maxLength`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(k)`

where `k` is the number of possible distinct characters.

## Notes / Tips

* This avoids repeatedly erasing characters from the left.
* The key invariant is:

  ```text
  [left ... right]
  ```

  always contains unique characters.
* `max(left, last[c] + 1)` is essential.
* Without `max`, `left` could move backward.

### Example of Why `max` Is Needed

Consider:

```text
s = "abba"
```

After processing the second `b`:

```text
left = 2
```

When the final `a` is encountered, its previous occurrence is at index `0`.

We must **not** move:

```text
left = 0 + 1 = 1
```

because that would move `left` backward.

Instead:

```text
left = max(2, 1)
     = 2
```

## Code

```cpp
class Solution {
public:
    int lengthOfLongestSubstring(string s) {
        vector<int> last(256, -1);

        int left = 0;
        int maxLength = 0;

        for (int right = 0; right < s.size(); right++) {
            if (last[s[right]] >= left) {
                left = last[s[right]] + 1;
            }

            last[s[right]] = right;

            maxLength = max(maxLength, right - left + 1);
        }

        return maxLength;
    }
};
```

---

# Approach Comparison

| Approach                         |    Time |  Space | Status      |
| -------------------------------- | ------: | -----: | ----------- |
| Brute Force + Set                | `O(n²)` | `O(k)` | Brute       |
| Sliding Window + Set             |  `O(n)` | `O(k)` | Better      |
| Sliding Window + Last Occurrence |  `O(n)` | `O(k)` | **Optimal** |

---

# Key Template

### Variable-Size Sliding Window

```cpp
int left = 0;

for (int right = 0; right < n; right++) {
    // Add/process nums[right]

    while (window_is_invalid) {
        // Remove/process nums[left]
        left++;
    }

    // Update answer using [left, right]
}
```

### Last Occurrence Optimization

```cpp
vector<int> last(256, -1);

int left = 0;
int answer = 0;

for (int right = 0; right < s.size(); right++) {
    if (last[s[right]] >= left) {
        left = last[s[right]] + 1;
    }

    last[s[right]] = right;

    answer = max(answer, right - left + 1);
}
```

## Pattern Recognition

When you see:

```text
Longest / shortest
+
Contiguous substring/subarray
+
Constraint on elements inside the window
```

Think:

```text
Sliding Window
```

For this problem:

```text
Need longest window
+
No repeated characters
        ↓
Variable-size sliding window
        ↓
Track character occurrences
```

The key observation is:

> **When a duplicate appears, the left boundary only needs to jump past the previous occurrence of that character.**
