# LeetCode 13 — Roman to Integer

## Metadata

* **LeetCode:** 13
* **Problem:** Roman to Integer
* **Difficulty:** Easy
* **Topics:** Hash Table, String, Math
* **Pattern:** Greedy, String Traversal
* **Key Pattern:** Subtract a Roman numeral when it is smaller than the numeral immediately after it; otherwise add it
* **Key Template:** Adjacent Comparison
* **Key Technique:** `if (value[i] < value[i + 1]) subtract, else add`
* **Optimal Complexity:** `O(n)` time, `O(1)` auxiliary space

---

# Approaches

1. **Brute Force — Handle Subtractive Pairs Explicitly**
2. **Better — Hash Map + Subtractive Pair Detection**
3. **Optimal — Adjacent Comparison**

---

# Approach 1 — Brute Force / Explicit Subtractive Pairs

## Idea

Roman numerals normally add their values:

```text
VI = 5 + 1 = 6
```

But there are six subtractive combinations:

```text
IV = 4
IX = 9
XL = 40
XC = 90
CD = 400
CM = 900
```

We can scan the string and explicitly recognize these pairs.

If a subtractive pair occurs, add its combined value and skip both characters.

Otherwise, add the value of the current character.

## Dry Run

```text
s = "MCMIV"
```

Process:

```text
M    → 1000
CM   → 900
IV   → 4
```

Therefore:

```text
1000 + 900 + 4 = 1904
```

## Algorithm

1. Initialize `answer = 0`.
2. Traverse the string from left to right.
3. Check whether the current and next characters form one of:

   ```text
   IV, IX, XL, XC, CD, CM
   ```
4. If they form a subtractive pair:

   * Add its corresponding value.
   * Skip the next character.
5. Otherwise, add the value of the current character.
6. Return `answer`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* This works because the valid Roman numeral subtractive cases are fixed.
* It is straightforward but requires explicitly maintaining all six special cases.
* The next approach removes much of this special-case handling.

## Code

```cpp
class Solution {
public:
    int romanToInt(string s) {
        int answer = 0;

        for (int i = 0; i < s.size(); i++) {
            if (i + 1 < s.size()) {
                string pair = s.substr(i, 2);

                if (pair == "IV") {
                    answer += 4;
                    i++;
                    continue;
                }

                if (pair == "IX") {
                    answer += 9;
                    i++;
                    continue;
                }

                if (pair == "XL") {
                    answer += 40;
                    i++;
                    continue;
                }

                if (pair == "XC") {
                    answer += 90;
                    i++;
                    continue;
                }

                if (pair == "CD") {
                    answer += 400;
                    i++;
                    continue;
                }

                if (pair == "CM") {
                    answer += 900;
                    i++;
                    continue;
                }
            }

            if (s[i] == 'I') {
                answer += 1;
            }
            else if (s[i] == 'V') {
                answer += 5;
            }
            else if (s[i] == 'X') {
                answer += 10;
            }
            else if (s[i] == 'L') {
                answer += 50;
            }
            else if (s[i] == 'C') {
                answer += 100;
            }
            else if (s[i] == 'D') {
                answer += 500;
            }
            else if (s[i] == 'M') {
                answer += 1000;
            }
        }

        return answer;
    }
};
```

---

# Approach 2 — Better / Hash Map + Subtractive Pair Detection

## Idea

Instead of using a long `if-else` chain for the Roman numeral values, use a hash map:

```text
I → 1
V → 5
X → 10
L → 50
C → 100
D → 500
M → 1000
```

Then use the same basic idea:

* If the current value is smaller than the next value → subtract it.
* Otherwise → add it.

This removes the need to explicitly check:

```text
IV, IX, XL, XC, CD, CM
```

## Dry Run

```text
s = "MCMIV"
```

Values:

```text
M C M I V
100 100 1000 1 5
```

Process:

```text
M < C ? No  → +1000
C < M ? Yes → -100
M < I ? No  → +1000
I < V ? Yes → -1
V             → +5
```

Total:

```text
1000 - 100 + 1000 - 1 + 5 = 1904
```

## Algorithm

1. Create a map containing the value of each Roman symbol.
2. Initialize `answer = 0`.
3. Traverse the string.
4. For each character:

   * Get its value.
   * If it is smaller than the next character's value, subtract it.
   * Otherwise, add it.
5. Return `answer`.

## Complexity

* **Time:** `O(n)` average
* **Space:** `O(1)`

The map contains only seven fixed characters.

## Notes / Tips

* The important comparison is between **adjacent values**.
* This approach already achieves optimal asymptotic complexity.
* The final approach simply removes the repeated hash-map lookups by using a fixed array/table.

## Code

```cpp
class Solution {
public:
    int romanToInt(string s) {
        unordered_map<char, int> value = {
            {'I', 1},
            {'V', 5},
            {'X', 10},
            {'L', 50},
            {'C', 100},
            {'D', 500},
            {'M', 1000}
        };

        int answer = 0;

        for (int i = 0; i < s.size(); i++) {
            if (i + 1 < s.size() && value[s[i]] < value[s[i + 1]]) {
                answer -= value[s[i]];
            }
            else {
                answer += value[s[i]];
            }
        }

        return answer;
    }
};
```

---

# Approach 3 — Optimal / Adjacent Comparison with Lookup Table

## Idea

The key mathematical observation is:

> A Roman numeral is normally additive. A symbol is subtracted only when its value is smaller than the symbol immediately after it.

Therefore, we do not need to recognize the six special pairs individually.

For example:

```text
VI
```

```text
5 >= 1
```

So:

```text
+5 +1 = 6
```

But:

```text
IV
```

```text
1 < 5
```

So:

```text
-1 +5 = 4
```

The same rule handles all subtractive cases.

## Dry Run

```text
s = "MCMIV"
```

### `M`

Next is `C`:

```text
1000 > 100
```

Add:

```text
answer = 1000
```

### First `C`

Next is `M`:

```text
100 < 1000
```

Subtract:

```text
answer = 900
```

### Second `M`

Next is `I`:

```text
1000 > 1
```

Add:

```text
answer = 1900
```

### `I`

Next is `V`:

```text
1 < 5
```

Subtract:

```text
answer = 1899
```

### `V`

No next character.

Add:

```text
answer = 1904
```

Final answer:

```text
1904
```

## Algorithm

1. Create a fixed lookup table for Roman numeral values.
2. Initialize `answer = 0`.
3. Traverse every character in the string.
4. If the current value is smaller than the next value:

   * Subtract the current value.
5. Otherwise:

   * Add the current value.
6. Return `answer`.

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

## Notes / Tips

* The entire problem reduces to one rule:

  ```text
  current < next → subtract
  current >= next → add
  ```
* This automatically handles:

  ```text
  IV, IX, XL, XC, CD, CM
  ```
* No special-case checking is necessary.
* A fixed array is preferable to `unordered_map` here because the input alphabet contains only seven known characters.

## Code

```cpp
class Solution {
public:
    int romanToInt(string s) {
        int value[256] = {};

        value['I'] = 1;
        value['V'] = 5;
        value['X'] = 10;
        value['L'] = 50;
        value['C'] = 100;
        value['D'] = 500;
        value['M'] = 1000;

        int answer = 0;

        for (int i = 0; i < s.size(); i++) {
            if (i + 1 < s.size() && value[s[i]] < value[s[i + 1]]) {
                answer -= value[s[i]];
            }
            else {
                answer += value[s[i]];
            }
        }

        return answer;
    }
};
```

---

# Approach Comparison

| Approach                   |   Time |  Space | Status      |
| -------------------------- | -----: | -----: | ----------- |
| Explicit Subtractive Pairs | `O(n)` | `O(1)` | Brute       |
| Hash Map + Comparison      | `O(n)` | `O(1)` | Better      |
| Lookup Table + Comparison  | `O(n)` | `O(1)` | **Optimal** |

---

# Key Template

### Adjacent Comparison

```cpp
for (int i = 0; i < s.size(); i++) {
    if (i + 1 < s.size() && value(s[i]) < value(s[i + 1])) {
        answer -= value(s[i]);
    }
    else {
        answer += value(s[i]);
    }
}
```

### Pattern Recognition

When a problem has:

```text
Symbols with numerical values
+
Normally additive
+
Special case when current value < next value
```

Think:

```text
Compare current with next
        ↓
current < next → subtract
current >= next → add
```

The key observation is:

> **Roman numeral subtraction can be represented entirely through adjacent value comparison instead of explicitly checking every subtractive pair.**
