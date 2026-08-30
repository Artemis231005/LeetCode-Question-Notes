# LeetCode 168 — Excel Sheet Column Title

## Metadata

- **LeetCode:** 168
- **Problem:** Excel Sheet Column Title
- **Difficulty:** Easy
- **Topics:** Math, String
- **Pattern:** Base-26 Conversion (Bijective / 1-Indexed)
- **Key Technique:** Subtract 1 before taking `% 26` to correctly handle a base where digits run 1-26 instead of 0-25

---

# Approaches

1. **Brute Force — Recursive String Prepending**
2. **Optimal — Iterative with Off-By-One Correction**

---

# Approach 1 — Brute Force / Recursive String Prepending

## Idea

Treat this like converting to base 26, peeling off one digit at a time from the least significant end, and prepending it to a growing string. Since there's no "0" digit in this base, subtract `1` before the mod/divide so the range `1-26` maps correctly onto `0-25`.

## Dry Run

```text
columnNumber = 701
```

Step 1:
```text
n = 701 - 1 = 700
digit = 700 % 26 = 24 → 'Y'
n = 700 / 26 = 26
result = "Y"
```

Step 2:
```text
n = 26 - 1 = 25
digit = 25 % 26 = 25 → 'Z'
n = 25 / 26 = 0
result = "ZY"
```

`n == 0` → stop.

Final:
```text
"ZY"
```

## Algorithm

1. If `columnNumber == 0`, return `""` (base case for recursion).
2. Subtract `1` from `columnNumber`.
3. Compute `digit = columnNumber % 26`, convert to character `'A' + digit`.
4. Recurse on `columnNumber / 26`, and **prepend** the current character to the recursive result.
5. Return the combined string.

## Complexity

- **Time:** `O(log_26(n))` — number of digits, but prepending to a string is `O(k)` each time, so effectively `O((log n)^2)`
- **Space:** `O(log n)` — recursion stack

## Notes / Tips

- Prepending (`c + recurse(...)`) is what makes this slightly wasteful — each prepend can shift the whole string.
- Useful for seeing the recursive structure clearly before flattening into an iterative loop.

## Code

```cpp
class Solution {
public:
    string convertToTitle(int columnNumber) {
        if (columnNumber == 0) {
            return "";
        }

        columnNumber--;
        char digit = 'A' + (columnNumber % 26);

        return convertToTitle(columnNumber / 26) + digit;
    }
};
```

---

# Approach 2 — Optimal / Iterative with Off-By-One Correction

## Idea

Same base-26 peeling logic, but iterate instead of recursing, appending each digit to the **end** of a string (cheap operation), then reverse the string once at the end.

## Dry Run

```text
columnNumber = 701
```

Process:
```text
n = 701
n-- → 700
digit = 700 % 26 = 24 → 'Y' → append → result = "Y"
n = 700 / 26 = 26

n-- → 25
digit = 25 % 26 = 25 → 'Z' → append → result = "YZ"
n = 25 / 26 = 0

n == 0 → stop
```

Reverse:
```text
"YZ" → "ZY"
```

Final:
```text
"ZY"
```

## Algorithm

1. Initialize an empty string `result`.
2. While `columnNumber > 0`:
   - `columnNumber--`.
   - `digit = columnNumber % 26`, append character `'A' + digit`.
   - `columnNumber /= 26`.
3. Reverse `result` and return it.

## Complexity

- **Time:** `O(log_26(n))`
- **Space:** `O(log n)` for the output string (unavoidable, it's the answer)

## Notes / Tips

- This is the exact reverse process of LC 171 (Excel Sheet Column Number) — same base, opposite direction.
- The `columnNumber--` before the mod/divide is the entire trick — without it, values like `26` (`"Z"`) would break since normal base-26 has no representation that maps directly onto a 1-26 digit range.
- Appending + reversing at the end is standard and avoids the repeated-prepend cost from Approach 1.

## Code

```cpp
class Solution {
public:
    string convertToTitle(int columnNumber) {
        string result = "";

        while (columnNumber > 0) {
            columnNumber--;
            result += ('A' + columnNumber % 26);
            columnNumber /= 26;
        }

        reverse(result.begin(), result.end());
        return result;
    }
};
```

---

# Key Template

### Number to Base-K String (Bijective / 1-Indexed Base)

```text
result = ""

while n > 0:
    n--
    digit = n % K
    result += char_of(digit)
    n /= K

reverse(result)
return result
```

## Pattern Recognition

The key observation is:

> **A bijective base (digits 1-26 instead of 0-25) just needs a `n--` before the usual mod/divide — everything else is standard base conversion.**