# LeetCode 345 — Reverse Vowels of a String

## Metadata

* **LeetCode:** 345
* **Problem:** Reverse Vowels of a String
* **Difficulty:** Easy
* **Topics:** Two Pointers, String
* **Pattern:** Two Pointers + Conditional Matching
* **Optimal Complexity:** O(n) time, O(1) auxiliary space

---

## Idea

We need to reverse only the **vowels**, while keeping all consonants and other characters in their original positions.

Use two pointers:

* `left` starts from the beginning.
* `right` starts from the end.

Move `left` until it reaches a vowel and `right` until it reaches a vowel. Swap those vowels and move both pointers inward.

---

## Dry Run

### `s = "hello"`

```text
left  → e
right → o

swap e and o

"holle"
```

Result: `"holle"`

---

## Algorithm

1. Initialize `left = 0` and `right = n - 1`.
2. While `left < right`:

   * Move `left` forward until it points to a vowel.
   * Move `right` backward until it points to a vowel.
   * Swap the two vowels.
   * Move both pointers inward.
3. Return the modified string.

---

## Complexity

* **Time:** O(n)
* **Space:** O(1) auxiliary space

---

## Code

```cpp
class Solution {
public:
    bool isVowel(char c) {
        return c == 'a' || c == 'e' || c == 'i' ||
               c == 'o' || c == 'u' ||
               c == 'A' || c == 'E' || c == 'I' ||
               c == 'O' || c == 'U';
    }

    string reverseVowels(string s) {
        int left = 0;
        int right = s.size() - 1;

        while (left < right) {
            while (left < right && !isVowel(s[left])) {
                left++;
            }

            while (left < right && !isVowel(s[right])) {
                right--;
            }

            if (left < right) {
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

## Notes / Tips

* This is a **two-pointer** problem where pointers skip elements that don't satisfy a condition.
* The only characters that participate in swaps are vowels.
* Check both uppercase and lowercase vowels.
* The string is modified in-place, so no extra array is required.

---

## Key Template

```text
left = 0
right = n - 1

while left < right:
    move left until target element
    move right until target element

    swap(left, right)

    move both inward
```

**Pattern:** When only certain elements need to be reversed while all other elements remain fixed, use **two pointers that skip non-target elements**.
