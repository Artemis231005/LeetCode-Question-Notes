# LeetCode 0171 — Excel Sheet Column Number

## Metadata

- **Problem:** Excel Sheet Column Number
- **Problem Statement:** Convert Excel column title to number
- **Difficulty:** Easy
- **Topics:** String, Math
- **Pattern:** Base Conversion
- **Key Technique:** Treat letters as digits in base 26
- **Optimal Complexity:** O(n) Time | O(1) Space

---

## Idea

Excel column titles work similarly to a number system with **base 26**, where:
- `A = 1`, `B = 2`, ..., `Z = 26`
- Unlike normal base systems, there is no `0` digit.

For every character, multiply the current answer by `26` and add the value of the current letter.

**Formula:**
`result = result × 26 + (currentCharacter - 'A' + 1)`

---

## Dry Run

### Example: `"AB"`

- Start: `result = 0`
- `'A' = 1` → `result = 0 × 26 + 1 = 1`
- `'B' = 2` → `result = 1 × 26 + 2 = 28`
Answer: `28`

---

## Algorithm

1. Initialize `result = 0`.
2. Traverse every character in the column title.
3. Convert the character to its value from `1` to `26`.
4. Update `result = result × 26 + value`.
5. Return `result`.

---

## Complexity

- **Time Complexity:** O(n)
- **Space Complexity:** O(1)

---

## Notes / Tips

- This is similar to converting a number from another base to decimal.
- Remember that `A` represents `1`, not `0`.
- For a string like `"ZY"`, process it from left to right using the multiplication formula.

---

## Key Template

```cpp
for (char c : s) {
    result = result * 26 + (c - 'A' + 1);  // - 'A' to convert to No, and + 1 as A is mapped to 1 and not 0
}
```

---

## Code

```cpp
class Solution {
public:
    int titleToNumber(string columnTitle) {
        int result = 0;

        for (char c : columnTitle) {
            result = result * 26 + (c - 'A' + 1);
        }

        return result;
    }
};
```
