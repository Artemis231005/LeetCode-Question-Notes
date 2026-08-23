# LeetCode 344 — Reverse String

## Metadata

* **LeetCode:** 344
* **Problem:** Reverse String
* **Difficulty:** Easy
* **Topics:** Two Pointers, String
* **Pattern:** Two Pointers
* **Optimal Complexity:** O(n) time, O(1) space

---

## Idea

Reverse the string **in-place** using two pointers:

* `left` starts at the beginning.
* `right` starts at the end.
* Swap the characters at `left` and `right`.
* Move both pointers toward the center.

Stop when `left >= right`.

This avoids creating another string.

---

## Dry Run

### `s = ['h', 'e', 'l', 'l', 'o']`

```text
left = 0, right = 4
swap h and o
→ ['o', 'e', 'l', 'l', 'h']

left = 1, right = 3
swap e and l
→ ['o', 'l', 'l', 'e', 'h']

left = 2, right = 2
stop
```

Result:

```text
['o', 'l', 'l', 'e', 'h']
```

---

## Algorithm

1. Initialize `left = 0`.
2. Initialize `right = s.size() - 1`.
3. While `left < right`:

   * Swap `s[left]` and `s[right]`.
   * Increment `left`.
   * Decrement `right`.
4. The string is now reversed in-place.

---

## Complexity

* **Time:** `O(n)`
* **Space:** `O(1)`

---

## Code

```cpp
class Solution {
public:
    void reverseString(vector<char>& s) {
        int left = 0;
        int right = s.size() - 1;

        while (left < right) {
            swap(s[left], s[right]);
            left++;
            right--;
        }
    }
};
```

---

## Notes / Tips

* The key pattern is **two pointers moving toward each other**.
* Since the problem requires **in-place** reversal, avoid creating another array/string.
* Only `n / 2` swaps are needed.
* This same pattern appears in reversing arrays, checking palindromes, and reversing parts of a string.

---

## Key Template

```text
left = 0
right = n - 1

while left < right:
    swap(left, right)
    left++
    right--
```

**Pattern:** For operations involving symmetric elements from both ends, use **two pointers moving inward**.
