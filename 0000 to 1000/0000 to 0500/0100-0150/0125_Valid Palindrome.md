# LeetCode 125 — Valid Palindrome

## Metadata

* **LeetCode:** 125
* **Problem:** Valid Palindrome
* **Difficulty:** Easy
* **Topics:** String, Two Pointers
* **Pattern:** Two Pointers
* **Key Technique:** Compare characters from both ends
* **Key Pattern:** Two-pointer string traversal
* **Key Template:** `left` and `right` pointers moving toward the center
* **Optimal Complexity:** `O(n)`

---

## Problem

Given a string `s`, determine whether it is a **palindrome**.

Rules:

* Ignore all non-alphanumeric characters.
* Ignore case differences.
* Return `true` if the resulting string reads the same forward and backward.

Example:

```text
"A man, a plan, a canal: Panama"
```

After ignoring spaces, punctuation, and case:

```text
"amanaplanacanalpanama"
```

This is a palindrome.

Answer:

```text
true
```

---

## Idea

Use the **Two Pointer** technique.

Place:

```text
left  → beginning
right → end
```

At every step:

1. Skip non-alphanumeric characters from the left.
2. Skip non-alphanumeric characters from the right.
3. Convert both characters to the same case.
4. Compare them.
5. If they are different, return `false`.
6. Otherwise move both pointers toward the center.

If all characters match, return `true`.

### Why Two Pointers?

We only need to compare:

```text
first ↔ last
second ↔ second-last
third ↔ third-last
...
```

There is no need to create another cleaned string.

---

## Dry Run

Consider:

```text
s = "A man, a plan, a canal: Panama"
```

Relevant characters:

```text
A ↔ a
m ↔ m
a ↔ a
n ↔ n
a ↔ a
p ↔ p
...
```

Ignoring case, every corresponding pair matches.

Therefore:

```text
true
```

### Example Where It Fails

```text
s = "race a car"
```

Compare:

```text
r ↔ r   ✓
a ↔ a   ✓
c ↔ c   ✓
e ↔ a   ✗
```

Characters do not match.

Therefore:

```text
false
```

---

## Algorithm

1. Set `left = 0` and `right = s.length() - 1`.
2. While `left < right`:

   * While `left < right` and `s[left]` is not alphanumeric, increment `left`.
   * While `left < right` and `s[right]` is not alphanumeric, decrement `right`.
   * Convert both characters to lowercase.
   * If they are different, return `false`.
   * Increment `left`.
   * Decrement `right`.
3. Return `true`.

---

## Complexity

* **Time:** `O(n)`

  * Each character is processed at most once.
* **Space:** `O(1)`

  * No extra string is created.

---

## Notes / Tips

* Use `isalnum()` to check whether a character is a letter or digit.
* Use `tolower()` to compare characters without considering case.
* Always skip invalid characters **before comparing**.
* The condition `left < right` prevents the pointers from crossing.
* Digits are also considered valid alphanumeric characters.
* Creating a cleaned string and reversing it works, but uses `O(n)` extra space.

### Common Mistake

Do not compare characters before skipping punctuation/spaces.

For example:

```text
"A man, a plan..."
```

The comma and spaces must be ignored.

### Alternative Approach

You can first create a cleaned lowercase string and then check whether it equals its reverse.

That approach:

* Time: `O(n)`
* Space: `O(n)`

The two-pointer approach is preferable because it uses **`O(1)` extra space**.

---

## Code

```cpp
class Solution {
public:
    bool isPalindrome(string s) {
        int left = 0;
        int right = s.length() - 1;

        while (left < right) {
            while (left < right && !isalnum(s[left])) {
                left++;
            }

            while (left < right && !isalnum(s[right])) {
                right--;
            }

            if (tolower(s[left]) != tolower(s[right])) {
                return false;
            }

            left++;
            right--;
        }

        return true;
    }
};
```

---

## Basic Template

```cpp
bool twoPointer(string s) {
    int left = 0;
    int right = s.length() - 1;

    while (left < right) {
        while (left < right && invalid(s[left])) {
            left++;
        }

        while (left < right && invalid(s[right])) {
            right--;
        }

        if (process(s[left]) != process(s[right])) {
            return false;
        }

        left++;
        right--;
    }

    return true;
}
```
