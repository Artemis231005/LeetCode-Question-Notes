# LeetCode 14 — Longest Common Prefix

## Metadata

* **LeetCode:** 14
* **Problem:** Longest Common Prefix
* **Difficulty:** Easy
* **Topics:** Array, String
* **Pattern:** String Matching, Prefix
* **Key Pattern:** Compare characters position-by-position across all strings
* **Key Template:** Prefix Scanning
* **Key Technique:** Stop at the first position where strings differ
* **Optimal Complexity:** `O(S)` time, `O(1)` auxiliary space

Where `S` is the total number of characters examined across the input strings.

---

# Approaches

1. **Brute Force — Compare Every Prefix**
2. **Better — Horizontal Scanning**
3. **Optimal — Vertical Scanning**

---

# Approach 1 — Brute Force / Compare Every Prefix

## Idea

Generate prefixes of the first string one by one and check whether each prefix exists at the beginning of every other string.

Start with the shortest possible prefix and keep extending it while it remains common.

A simpler implementation is to check every possible prefix length.

## Dry Run

```text
strs = ["flower", "flow", "flight"]
```

Try:

```text
"f"      → common ✓
"fl"     → common ✓
"flo"    → not common
```

Therefore:

```text
"fl"
```

is the longest common prefix.

## Algorithm

1. Take the first string as the reference.
2. Generate prefixes of increasing length.
3. For every prefix:

   * Compare it with every other string.
4. If all strings start with that prefix, continue.
5. Stop at the first invalid prefix.
6. Return the previous valid prefix.

## Complexity

Let:

* `n` = number of strings

* `m` = length of the shortest string

* **Time:** `O(n × m²)` in the straightforward implementation due to prefix comparisons.

* **Space:** `O(m)` for the prefix strings.

## Notes / Tips

* This approach is mainly useful for understanding the problem.
* It performs unnecessary repeated comparisons.

## Code

```cpp
class Solution {
public:
    bool isCommonPrefix(vector<string>& strs, string prefix) {
        for (int i = 1; i < strs.size(); i++) {
            if (strs[i].compare(0, prefix.size(), prefix) != 0) {
                return false;
            }
        }

        return true;
    }

    string longestCommonPrefix(vector<string>& strs) {
        string answer = "";

        for (int len = 1; len <= strs[0].size(); len++) {
            string prefix = strs[0].substr(0, len);

            if (!isCommonPrefix(strs, prefix)) {
                break;
            }

            answer = prefix;
        }

        return answer;
    }
};
```

---

# Approach 2 — Better / Horizontal Scanning

## Idea

Use the first string as the current prefix.

Compare it with every other string.

Whenever the current prefix is not a prefix of the next string, shorten it until it becomes one.

For example:

```text
["flower", "flow", "flight"]
```

Start:

```text
prefix = "flower"
```

Compare with `"flow"`:

```text
prefix → "flow"
```

Compare `"flow"` with `"flight"`:

```text
"flow"
"flig..."
```

Common part:

```text
"fl"
```

Final answer:

```text
"fl"
```

## Dry Run

```text
strs = ["flower", "flow", "flight"]
```

Initial:

```text
prefix = "flower"
```

Compare with:

```text
"flow"
```

Reduce:

```text
"flower" → "flow"
```

Now compare:

```text
"flow"
"flight"
```

Reduce:

```text
"flow" → "flo" → "fl"
```

Final:

```text
"fl"
```

## Algorithm

1. Set:

   ```text
   prefix = strs[0]
   ```
2. For every remaining string:

   * While the current string does not start with `prefix`:

     * Remove the last character from `prefix`.
3. If `prefix` becomes empty, return `""`.
4. Return `prefix`.

## Complexity

Let:

* `n` = number of strings

* `m` = length of the shortest string

* **Time:** `O(n × m)` in the worst case.

* **Space:** `O(1)` auxiliary space, excluding the returned string.

## Notes / Tips

* The prefix can only become shorter as we process more strings.
* Once `prefix == ""`, we can immediately return.
* This is simpler than generating every possible prefix.

## Code

```cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        string prefix = strs[0];

        for (int i = 1; i < strs.size(); i++) {
            while (strs[i].find(prefix) != 0) {
                prefix.pop_back();

                if (prefix.empty()) {
                    return "";
                }
            }
        }

        return prefix;
    }
};
```

---

# Approach 3 — Optimal / Vertical Scanning

## Idea

Instead of comparing complete strings one by one, compare characters at the **same position** across all strings.

For every character position `i`:

```text
strs[0][i]
strs[1][i]
strs[2][i]
...
```

If all characters are equal, they belong to the common prefix.

The moment one character differs, stop.

### Example

```text
flower
flow
flight
```

Compare vertically:

```text
f f f ✓
l l l ✓
o o i ✗
```

Therefore:

```text
"fl"
```

## Dry Run

```text
strs = ["flower", "flow", "flight"]
```

### Position 0

```text
f
f
f
```

All equal.

```text
prefix = "f"
```

### Position 1

```text
l
l
l
```

All equal.

```text
prefix = "fl"
```

### Position 2

```text
o
o
i
```

Mismatch.

Stop.

Answer:

```text
"fl"
```

## Algorithm

1. Use the first string as the reference.
2. Iterate through each character index `i` of the first string.
3. For every other string:

   * Check whether it has a character at index `i`.
   * Check whether `strs[j][i] == strs[0][i]`.
4. If any string:

   * Ends before `i`, or
   * Has a different character,
     stop.
5. Return the characters matched so far.

## Complexity

Let:

* `S` = total number of characters examined

* `m` = length of the shortest string

* `n` = number of strings

* **Time:** `O(S)` in terms of characters actually examined.

* **Space:** `O(1)` auxiliary space, excluding the returned string.

## Notes / Tips

* This directly models the definition of a common prefix.
* We only need to inspect characters until the first mismatch.
* The shortest string automatically limits the maximum possible prefix length.
* This is a clean **vertical scanning** pattern for string problems.
* An empty input array should be handled before accessing `strs[0]`.

## Code

```cpp
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if (strs.empty()) {
            return "";
        }

        for (int i = 0; i < strs[0].size(); i++) {
            char current = strs[0][i];

            for (int j = 1; j < strs.size(); j++) {
                if (i >= strs[j].size() || strs[j][i] != current) {
                    return strs[0].substr(0, i);
                }
            }
        }

        return strs[0];
    }
};
```

---

# Approach Comparison

| Approach             |        Time |  Space | Status      |
| -------------------- | ----------: | -----: | ----------- |
| Compare Every Prefix | `O(n × m²)` | `O(m)` | Brute       |
| Horizontal Scanning  |  `O(n × m)` | `O(1)` | Better      |
| Vertical Scanning    |      `O(S)` | `O(1)` | **Optimal** |

Where:

```text
n = number of strings
m = length of shortest string
S = total number of characters examined
```

---

# Key Template

### Prefix Scanning

```cpp
for (int i = 0; i < strs[0].size(); i++) {
    char current = strs[0][i];

    for (int j = 1; j < strs.size(); j++) {
        if (i >= strs[j].size() || strs[j][i] != current) {
            return strs[0].substr(0, i);
        }
    }
}

return strs[0];
```

### Pattern Recognition

When a problem asks for something common **across multiple strings**, especially a prefix:

```text
Compare the same index across all strings
        ↓
If all match → continue
        ↓
First mismatch → stop
```

The key observation is:

> **A common prefix ends at the first position where any string differs or ends.**
